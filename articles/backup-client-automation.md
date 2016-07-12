<properties
	pageTitle="使用 PowerShell 部署和管理 Windows Server/客户端的备份 | Azure"
	description="了解如何使用 PowerShell 部署和管理 Azure 备份"
	services="backup"
	documentationCenter=""
	authors="aashishr"
	manager="shreeshd"
	editor=""/>

<tags
	ms.service="backup" 
	ms.date="01/22/2016"
	wacn.date="05/09/2016"/>


# 使用 PowerShell 部署和管理 Windows Server/Windows 客户端的 Azure 备份

本文说明如何使用 PowerShell 在 Windows Server 或 Windows 客户端上设置 Azure 备份，以及管理备份和恢复。

## 安装 Azure PowerShell

[AZURE.INCLUDE [了解部署模型](../includes/learn-about-deployment-models-include.md)]

Azure PowerShell 1.0 已在 2015 年 10 月发布。此版本在 0.9.8 版本的基础上进行了一些重大的更改，尤其是对 cmdlet 的命名方式进行了更改。1.0 版 cmdlet 遵循命名模式 {谓词}-AzureRm{名词}；而 0.9.8 名称不包括 **Rm**（例如，使用 New-AzureResourceGroup 而不是 New-AzureRmResourceGroup）。在使用 Azure PowerShell 0.9.8 时，首先必须通过运行 **Switch-AzureMode AzureResourceManager** 命令启用资源管理器模式。此命令在 1.0 或更高版中并不需要。

若要将针对 0.9.8 环境编写的脚本用在 1.0 或更高版本的环境中，应在预生产环境中对脚本进行仔细的测试，然后才能将其用在生产中，以免造成意外影响。

[下载最新的 PowerShell 版本](https://github.com/Azure/azure-powershell/releases)（所需最低版本：1.0.0）


[AZURE.INCLUDE [arm-getting-setup-powershell](../includes/arm-getting-setup-powershell.md)]


### 创建备份保管库

> [AZURE.WARNING] 对于第一次使用 Azure 备份的客户，你需要注册用于订阅的 Azure 备份提供程序。可通过运行以下命令来执行此操作：Register-AzureProvider -ProviderNamespace "Microsoft.Backup"

可以使用 **New-AzureRMBackupVault** cmdlet 创建新的备份保管库。备份保管库是一种 ARM 资源，因此需要将它放置在资源组中。在权限提升的 Azure PowerShell 控制台中运行以下命令：


		PS C:\> New-AzureResourceGroup –Name “test-rg” -Region “China North”
		PS C:\> $backupvault = New-AzureRMBackupVault –ResourceGroupName “test-rg” –Name “test-vault” –Region “China North” –Storage GeoRedundant


使用 **Get-AzureRMBackupVault** cmdlet 列出订阅中的备份保管库。


## 安装 Azure 备份代理
在安装 Azure 备份代理之前，必须先将安装程序下载到 Windows Server 上。可以从 [Microsoft 下载中心](http://aka.ms/azurebackup_agent)或者备份保管库的“仪表板”页获取最新版本的安装程序。将安装程序保存到方便访问的位置，例如 *C:\\Downloads*。

若要安装代理，请在已提升权限的 PowerShell 控制台中运行以下命令：


		PS C:\> MARSAgentInstaller.exe /q


这将以所有默认选项安装代理。将在后台执行安装几分钟。如果你没有指定 */nu* 选项，则安装结束时，会打开“Windows Update”窗口，以检查是否有任何更新。安装之后，代理将显示在已安装程序列表中。

若要查看已安装的程序列表，请转到“控制面板”>“程序”>“程序和功能”。

![已安装代理](./media/backup-client-automation/installed-agent-listing.png)

### 安装选项

若要查看所有可通过命令行运行的所有选项，请使用以下命令：


		PS C:\> MARSAgentInstaller.exe /?


可用选项包括：

| 选项 | 详细信息 | 默认 |
| ---- | ----- | ----- |
| /q | 静默安装 | - |
| /p:"location" | Azure 备份代理的安装文件夹路径。| C:\\Program Files\\Microsoft Azure Recovery Services Agent |
| /s:"location" | Azure 备份代理的快取文件夹路径。| C:\\Program Files\\Microsoft Azure Recovery Services Agent\\Scratch |
| /m | 选择启用 Microsoft Update | - |
| /nu | 安装完成后不要检查更新 | - |
| /d | 卸载 Microsoft Azure 恢复服务代理 | - |
| /ph | 代理主机地址 | - |
| /po | 代理主机端口号 | - |
| /pu | 代理主机用户名 | - |
| /pw | 代理密码 | - |


## 注册到 Azure 备份服务
在可注册 Azure 备份服务之前，需要确保符合[先决条件](/documentation/articles/backup-configure-vault/)。你必须：

- 具备有效的 Azure 订阅
- 有一个备份保管库

若要下载保管库凭据，请在 Azure PowerShell 控制台中运行 **Get-AzureRMBackupVaultCredentials** cmdlet，并将其存储在方便的位置，例如 *C:\\Downloads*。


		PS C:\> $credspath = "C:"
		PS C:\> $credsfilename = Get-AzureRMBackupVaultCredentials -Vault $backupvault -TargetLocation $credspath
		PS C:\> $credsfilename
		f5303a0b-fae4-4cdb-b44d-0e4c032dde26_backuprg_backuprn_2015-08-11--06-22-35.VaultCredentials


使用 [Start-OBRegistration](https://technet.microsoft.com/zh-cn/library/hh770398%28v=wps.630%29.aspx) cmdlet 即可向保管库注册计算机：


		PS C:\> $cred = $credspath + $credsfilename
		PS C:\> Start-OBRegistration -VaultCredentials $cred -Confirm:$false
		CertThumbprint      : 7a2ef2caa2e74b6ed1222a5e89288ddad438df2
		SubscriptionID      : ef4ab577-c2c0-43e4-af80-af49f485f3d1
		ServiceResourceName : test-vault
		Region              : China North
		Machine registration succeeded.
		

> [AZURE.IMPORTANT] 请勿使用相对路径来指定保管库凭据文件。必须提供绝对路径作为 cmdlet 的输入。

## 网络设置
如果 Windows 计算机通过代理服务器连接到 Internet，则也可以向代理提供代理设置。此示例未使用代理服务器，因此我们要显式清除任何代理相关的信息。

你也可以针对给定的一组星期日期，使用 ```work hour bandwidth``` 和 ```non-work hour bandwidth``` 选项来控制带宽使用。

使用 [Set-OBMachineSetting](https://technet.microsoft.com/zh-cn/library/hh770409%28v=wps.630%29.aspx) cmdlet 即可设置代理和带宽详细信息：


		PS C:\> Set-OBMachineSetting -NoProxy
		Server properties updated successfully.
		PS C:\> Set-OBMachineSetting -NoThrottle
		Server properties updated successfully.


## 加密设置
发送到 Azure 备份的备份数据将会加密，以保护数据的机密性。加密通行短语是在还原时用于解密数据的“密码”。

		
		PS C:\> ConvertTo-SecureString -String "Complex!123_STRING" -AsPlainText -Force | Set-OBMachineSetting
		Server properties updated successfully
		

> [AZURE.IMPORTANT] 请妥善保管设置好的通行短语，并保证其安全。如果没有此通行短语，你将无法从 Azure 还原数据。

## 备份文件和文件夹
从 Windows Server 和客户端到 Azure 备份的所有备份由策略控制。原则包含三个部分：策略由三个部分组成：

1. 一个“备份计划”，指定何时需要备份以及与服务同步。
2. 一个“保留计划”，指定要在 Azure 中保留恢复点多长时间。
3. 一个**文件包含/排除规范**，指示应备份哪些内容。

在本文档中，由于我们要自动备份，因此假设尚未配置任何选项。首先，我们使用 [New-OBPolicy](https://technet.microsoft.com/zh-cn/library/hh770416.aspx) cmdlet 创建新的备份策略，并使用该策略。


		PS C:\> $newpolicy = New-OBPolicy


该策略暂时是空的，需要使用其他 cmdlet 来定义要包含或排除的项、运行备份的时间，以及备份的存储位置。

### 配置备份计划
在策略的 3 个组成部分中，第 1 个部分是备份计划，它是使用 [New-OBSchedule](https://technet.microsoft.com/zh-cn/library/hh770401) cmdlet 创建的。备份计划将定义何时需要备份。创建计划时，需要指定两个输入参数：

- 应运行备份的“星期日期”。你可以只选一天或选择一周的每天运行备份作业，或选择星期日期的任意组合。
- 应运行备份的**日期时间**。最多可以定义一天的 3 个不同日期时间来触发备份

例如，你可以配置在每个星期六和星期日下午 4 点运行备份策略。


		PS C:\> $sched = New-OBSchedule -DaysofWeek Saturday, Sunday -TimesofDay 16:00


备份计划需要与策略相关联，这可以使用 [Set-OBSchedule](https://technet.microsoft.com/zh-cn/library/hh770407) cmdlet 来实现。


		PS C:\> Set-OBSchedule -Policy $newpolicy -Schedule $sched
		BackupSchedule : 4:00 PM Saturday, Sunday, Every 1 week(s) DsList : PolicyName : RetentionPolicy : State : New PolicyState : Valid

### 配置保留策略
保留策略定义基于备份作业创建的恢复点的保留时间。使用 [New-OBRetentionPolicy](https://technet.microsoft.com/zh-cn/library/hh770425) cmdlet 创建新的保留策略时，可以使用 Azure 备份来指定需要保留备份恢复点的天数。以下示例将保留策略设置为 7 天。


		PS C:\> $retentionpolicy = New-OBRetentionPolicy -RetentionDays 7


必须使用 cmdlet [Set-OBRetentionPolicy](https://technet.microsoft.com/library/hh770405) 将保留策略与主要策略相关联：

				
		PS C:\> Set-OBRetentionPolicy -Policy $newpolicy -RetentionPolicy $retentionpolicy
		BackupSchedule  : 4:00 PM
		                  Saturday, Sunday,
		                  Every 1 week(s)
		DsList          :
		PolicyName      :
		RetentionPolicy : Retention Days : 7
		                  WeeklyLTRSchedule :
		                  Weekly schedule is not set
		                  MonthlyLTRSchedule :
		                  Monthly schedule is not set
		                  YearlyLTRSchedule :
		                  Yearly schedule is not set
		State           : New
		PolicyState     : Valid

### 包含和排除要备份的文件
`OBFileSpec` 对象定义要在备份中包含与排除的文件。这组规则可划分出计算机上要保护的文件和文件夹。你可以设置任意数量的文件包含或排除规则，并将其与策略相关联。创建新的 OBFileSpec 对象时，你可以：

- 指定要包含的文件和文件夹
- 指定要排除的文件和文件夹
- 指定要递归备份文件夹中的数据，或是否仅备份指定文件夹中的顶级文件。

可以在 New-OBFileSpec 命令中使用 -NonRecursive 标志来完成后一种指定。

在以下示例中，我们要备份卷 C: 和 D:，并排除 Windows 文件夹和任何临时文件夹中的操作系统二进制文件。为此，我们将使用 [New-OBFileSpec](https://technet.microsoft.com/library/hh770408) cmdlet 创建两个文件规范 - 一个用于包含，一个用于排除。创建文件规范后，使用 [Add-OBFileSpec](https://technet.microsoft.com/library/hh770424) cmdlet 将它们与策略相关联。


		PS C:\> $inclusions = New-OBFileSpec -FileSpec @("C:", "D:")
		PS C:\> $exclusions = New-OBFileSpec -FileSpec @("C:\windows", "C:\temp") -Exclude
		PS C:\> Add-OBFileSpec -Policy $newpolicy -FileSpec $inclusions
		BackupSchedule  : 4:00 PM
		                  Saturday, Sunday,
		                  Every 1 week(s)
		DsList          : {DataSource
		                  DatasourceId:0
		                  Name:C:\
		                  FileSpec:FileSpec
		                  FileSpec:C:\
		                  IsExclude:False
		                  IsRecursive:True
		                  , DataSource
		                  DatasourceId:0
		                  Name:D:\
		                  FileSpec:FileSpec
		                  FileSpec:D:\
		                  IsExclude:False
		                  IsRecursive:True
		                  }
		PolicyName      :
		RetentionPolicy : Retention Days : 7
		                  WeeklyLTRSchedule :
		                  Weekly schedule is not set
		                  MonthlyLTRSchedule :
		                  Monthly schedule is not set
		                  YearlyLTRSchedule :
		                  Yearly schedule is not set
		State           : New
		PolicyState     : Valid
		PS C:\> Add-OBFileSpec -Policy $newpolicy -FileSpec $exclusions
		BackupSchedule  : 4:00 PM
		                  Saturday, Sunday,
		                  Every 1 week(s)
		DsList          : {DataSource
		                  DatasourceId:0
		                  Name:C:\
		                  FileSpec:FileSpec
		                  FileSpec:C:\
		                  IsExclude:False
		                  IsRecursive:True
		                  ,FileSpec
		                  FileSpec:C:\windows
		                  IsExclude:True
		                  IsRecursive:True
		                  ,FileSpec
		                  FileSpec:C:\temp
		                  IsExclude:True
		                  IsRecursive:True
		                  , DataSource
		                  DatasourceId:0
		                  Name:D:\
		                  FileSpec:FileSpec
		                  FileSpec:D:\
		                  IsExclude:False
		                  IsRecursive:True
		                  }
		PolicyName      :
		RetentionPolicy : Retention Days : 7
		                  WeeklyLTRSchedule :
		                  Weekly schedule is not set
		                  MonthlyLTRSchedule :
		                  Monthly schedule is not set
		                  YearlyLTRSchedule :
		                  Yearly schedule is not set
		State           : New
		PolicyState     : Valid


### 应用策略
现在已完成策略对象，并且具有关联的备份计划、保留策略及文件包含/排除列表。现在可以提交此策略以供 Azure 备份使用。应用新建策略之前，请使用 [Remove-OBPolicy](https://technet.microsoft.com/zh-cn/library/hh770415) cmdlet 确保没有任何现有备份策略与服务器相关联。删除策略时，系统会提示你确认。若要跳过确认，请在 cmdlet 中请使用 `-Confirm:$false` 标志。


		PS C:\> Get-OBPolicy | Remove-OBPolicy
		Microsoft Azure Backup Are you sure you want to remove this backup policy? This will delete all the backed up data. [Y] Yes [A] Yes to All [N] No [L] No to All [S] Suspend [?] Help (default is "Y"):


使用 [Set-OBPolicy](https://technet.microsoft.com/zh-cn/library/hh770421) cmdlet 可以提交策略对象。系统将提示你确认。若要跳过确认，请在 cmdlet 中请使用 `-Confirm:$false` 标志。


		PS C:\> Set-OBPolicy -Policy $newpolicy
		Microsoft Azure Backup Do you want to save this backup policy ? [Y] Yes [A] Yes to All [N] No [L] No to All [S] Suspend [?] Help (default is "Y"):
		BackupSchedule : 4:00 PM Saturday, Sunday, Every 1 week(s)
		DsList : {DataSource
		         DatasourceId:4508156004108672185
		         Name:C:\
		         FileSpec:FileSpec
		         FileSpec:C:\
		         IsExclude:False
		         IsRecursive:True,
		         FileSpec
		         FileSpec:C:\windows
		         IsExclude:True
		         IsRecursive:True,
		         FileSpec
		         FileSpec:C:\temp
		         IsExclude:True
		         IsRecursive:True,
		         DataSource
		         DatasourceId:4508156005178868542
		         Name:D:\
		         FileSpec:FileSpec
		         FileSpec:D:\
		         IsExclude:False
		         IsRecursive:True
			}
		PolicyName : c2eb6568-8a06-49f4-a20e-3019ae411bac
		RetentionPolicy : Retention Days : 7
		              WeeklyLTRSchedule :
		              Weekly schedule is not set
		              MonthlyLTRSchedule :
		              Monthly schedule is not set
		              YearlyLTRSchedule :
		              Yearly schedule is not set
		State : Existing PolicyState : Valid


可以使用 [Get-OBPolicy](https://technet.microsoft.com/zh-cn/library/hh770406) cmdlet 来查看现有备份策略的详细信息。可以使用 [Get-OBSchedule](https://technet.microsoft.com/zh-cn/library/hh770423) cmdlet（适用于备份计划）和 [Get-OBRetentionPolicy](https://technet.microsoft.com/zh-cn/library/hh770427) cmdlet（适用于保留策略）向下钻取

		
		PS C:\> Get-OBPolicy | Get-OBSchedule
		SchedulePolicyName : 71944081-9950-4f7e-841d-32f0a0a1359a
		ScheduleRunDays : {Saturday, Sunday}
		ScheduleRunTimes : {16:00:00}
		State : Existing
		PS C:\> Get-OBPolicy | Get-OBRetentionPolicy
		RetentionDays : 7
		RetentionPolicyName : ca3574ec-8331-46fd-a605-c01743a5265e
		State : Existing
		PS C:\> Get-OBPolicy | Get-OBFileSpec
		FileName : *
		FilePath : \?\Volume{b835d359-a1dd-11e2-be72-2016d8d89f0f}\
		FileSpec : D:\
		IsExclude : False
		IsRecursive : True
		FileName : *
		FilePath : \?\Volume{cdd41007-a22f-11e2-be6c-806e6f6e6963}\
		FileSpec : C:\
		IsExclude : False
		IsRecursive : True
		FileName : *
		FilePath : \?\Volume{cdd41007-a22f-11e2-be6c-806e6f6e6963}\windows
		FileSpec : C:\windows
		IsExclude : True
		IsRecursive : True
		FileName : *
		FilePath : \?\Volume{cdd41007-a22f-11e2-be6c-806e6f6e6963}\temp
		FileSpec : C:\temp
		IsExclude : True
		IsRecursive : True


### 执行即席备份
设置备份策略之后，将会根据计划进行备份。你也可以使用 [Start-OBBackup](https://technet.microsoft.com/zh-cn/library/hh770426) cmdlet 来触发即席备份：

		
		PS C:\> Get-OBPolicy | Start-OBBackup
		Taking snapshot of volumes...
		Preparing storage...
		Estimating size of backup items...
		Estimating size of backup items...
		Transferring data...
		Verifying backup...
		Job completed.
		The backup operation completed successfully.


## 从 Azure 备份还原数据
本部分将引导你完成自动从 Azure 备份恢复数据的步骤。此过程涉及以下步骤：

1. 选取源卷
2. 选择要还原的备份点
3. 选择要还原的项
4. 触发还原过程

### 选取源卷
若要从 Azure 备份还原某个项，需要先识别该项的源。由于我们要在 Windows Server 或 Windows 客户端的上下文中执行命令，因此已识别了计算机。识别源的下一步是识别它所在的卷。运行 [Get-OBRecoverableSource](https://technet.microsoft.com/zh-cn/library/hh770410) cmdlet 可以检索正在从此计算机备份的卷或源的列表。此命令将返回从此服务器/客户端备份的所有源的数组。


		PS C:\> $source = Get-OBRecoverableSource
		PS C:\> $source
		FriendlyName : C:\
		RecoverySourceName : C:\
		ServerName : myserver.microsoft.com
		FriendlyName : D:\
		RecoverySourceName : D:\
		ServerName : myserver.microsoft.com
		

### 选择要还原的备份点
结合适当的参数运行 [Get-OBRecoverableItem](https://technet.microsoft.com/zh-cn/library/hh770399.aspx) cmdlet 可以检索备份点列表。在本示例中，我们将选择源卷 *D:* 的最新备份点，并使用它还原特定的文件。

		
		PS C:\> $rps = Get-OBRecoverableItem -Source $source[1]
		IsDir : False
		ItemNameFriendly : D:\
		ItemNameGuid : \?\Volume{b835d359-a1dd-11e2-be72-2016d8d89f0f}\
		LocalMountPoint : D:\
		MountPointName : D:\
		Name : D:\
		PointInTime : 18-Jun-15 6:41:52 AM
		ServerName : myserver.microsoft.com
		ItemSize :
		ItemLastModifiedTime :
		IsDir : False
		ItemNameFriendly : D:\
		ItemNameGuid : \?\Volume{b835d359-a1dd-11e2-be72-2016d8d89f0f}\
		LocalMountPoint : D:\
		MountPointName : D:\
		Name : D:\
		PointInTime : 17-Jun-15 6:31:31 AM
		ServerName : myserver.microsoft.com
		ItemSize :
		ItemLastModifiedTime :

对象 `$rps` 是备份点数组。第一个元素是最新备份点，第 N 个元素是最旧的备份点。为了选择最新的点，我们将使用 `$rps[0]`。

### 选择要还原的项
为了识别要还原的确切文件或文件夹，请以递归方式使用 [Get-OBRecoverableItem](https://technet.microsoft.com/zh-cn/library/hh770399.aspx) cmdlet。这样，只需使用 `Get-OBRecoverableItem` 便可浏览文件夹层次结构。

在本示例中，如果我们要还原文件 *finances.xls*，可以使用对象 `$filesFolders[1]` 来引用该文件。


		PS C:\> $filesFolders = Get-OBRecoverableItem $rps[0]
		PS C:\> $filesFolders
		IsDir : True
		ItemNameFriendly : D:\MyData\
		ItemNameGuid : \?\Volume{b835d359-a1dd-11e2-be72-2016d8d89f0f}\MyData\
		LocalMountPoint : D:\
		MountPointName : D:\
		Name : MyData
		PointInTime : 18-Jun-15 6:41:52 AM
		ServerName : myserver.microsoft.com
		ItemSize :
		ItemLastModifiedTime : 15-Jun-15 8:49:29 AM
		PS C:\> $filesFolders = Get-OBRecoverableItem $filesFolders[0]
		PS C:\> $filesFolders
		IsDir : False
		ItemNameFriendly : D:\MyData\screenshot.oxps
		ItemNameGuid : \?\Volume{b835d359-a1dd-11e2-be72-2016d8d89f0f}\MyData\screenshot.oxps
		LocalMountPoint : D:\
		MountPointName : D:\
		Name : screenshot.oxps
		PointInTime : 18-Jun-15 6:41:52 AM
		ServerName : myserver.microsoft.com
		ItemSize : 228313
		ItemLastModifiedTime : 21-Jun-14 6:45:09 AM
		IsDir : False
		ItemNameFriendly : D:\MyData\finances.xls
		ItemNameGuid : \?\Volume{b835d359-a1dd-11e2-be72-2016d8d89f0f}\MyData\finances.xls
		LocalMountPoint : D:\
		MountPointName : D:\
		Name : finances.xls
		PointInTime : 18-Jun-15 6:41:52 AM
		ServerName : myserver.microsoft.com
		ItemSize : 96256
		ItemLastModifiedTime : 21-Jun-14 6:43:02 AM


你也可以使用 `Get-OBRecoverableItem` cmdlet 来搜索要还原的项。在本示例中，为了搜索 *finances.xls*，我们可以运行以下命令来获取该文件上的句柄：


		PS C:\> $item = Get-OBRecoverableItem -RecoveryPoint $rps[0] -Location "D:\MyData" -SearchString "finance*"


### 触发还原过程
为了触发还原过程，首先需要指定恢复选项。这可以使用 [New-OBRecoveryOption](https://technet.microsoft.com/zh-cn/library/hh770417.aspx) cmdlet 来完成。在本示例中，我们假设要将文件还原到 *C:\\temp*。此外，我们假设要跳过目标文件夹 *C:\\temp* 中已存在的文件。若要创建此类恢复选项，请使用以下命令：


		PS C:\> $recovery_option = New-OBRecoveryOption -DestinationPath "C:\temp" -OverwriteType Skip


现在，请对 `Get-OBRecoverableItem` cmdlet 输出中的选定 `$item` 使用 [Start-OBRecovery](https://technet.microsoft.com/zh-cn/library/hh770402.aspx) 命令来触发还原：


		PS C:\> Start-OBRecovery -RecoverableItem $item -RecoveryOption $recover_option
		Estimating size of backup items...
		Estimating size of backup items...
		Estimating size of backup items...
		Estimating size of backup items...
		Job completed.
		The recovery operation completed successfully.
		


## 卸载 Azure 备份代理
可以使用以下命令卸载 Azure 备份代理：

		PS C:\> .\MARSAgentInstaller.exe /d /q

若要从计算机中卸载代理二进制文件，请注意以下部分后果：

- 这会从计算机中删除文件筛选器，并停止跟踪更改。
- 将从计算机中删除所有策略信息，但服务中会继续存储这些策略信息。
- 将删除所有备份计划，且不会进一步创建备份。

不过，Azure 中存储的数据将会根据你设置的保留策略继续保留。较旧的恢复点会自动过时。

## 远程管理
围绕 Azure 备份代理、策略和数据源的所有管理工作都可通过 Azure PowerShell 远程完成。要远程管理的计算机需要经过适当的准备。

默认情况下，WinRM 服务已配置为手动启动。你必须将启动类型设置为“自动”，并且应该启动该服务。若要确认 WinRM 服务正在运行，“状态”属性的值应该是“正在运行”。


		PS C:\> Get-Service WinRM
		Status   Name               DisplayName
		------   ----               -----------
		Running  winrm              Windows Remote Management (WS-Manag...
		

应该针对远程管理配置 PowerShell。


		PS C:\> Enable-PSRemoting -force
		WinRM is already set up to receive requests on this computer.
		WinRM has been updated for remote management.
		WinRM firewall exception enabled.
		PS C:\> Set-ExecutionPolicy unrestricted -force


现在可以远程管理计算机 - 从代理的安装开始。例如，以下脚本会将代理复制到远程计算机并安装代理。

		
		PS C:\> $dloc = "\\REMOTESERVER01\c$\Windows\Temp"
		PS C:\> $agent = "\\REMOTESERVER01\c$\Windows\Temp\MARSAgentInstaller.exe"
		PS C:\> $args = "/q"
		PS C:\> Copy-Item "C:\Downloads\MARSAgentInstaller.exe" -Destination $dloc - force
		PS C:\> $s = New-PSSession -ComputerName REMOTESERVER01
		PS C:\> Invoke-Command -Session $s -Script { param($d, $a) Start-Process -FilePath $d $a -Wait } -ArgumentList $agent $args


## 后续步骤
有关适用于 Windows Server/客户端的 Azure 备份的详细信息，请参阅

- [Azure 备份简介](/documentation/articles/backup-configure-vault/)
- [备份 Windows Server](/documentation/articles/backup-azure-backup-windows-server/)

<!---HONumber=Mooncake_0307_2016-->