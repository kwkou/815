<properties
    pageTitle="使用 PowerShell 部署和管理 Windows Server/客户端的备份 | Azure"
    description="了解如何使用 PowerShell 部署和管理 Azure 备份"
    services="backup"
    documentationcenter=""
    author="saurabhsensharma"
    manager="shivamg"
    editor="" />
<tags
    ms.assetid="65218095-2996-44d9-917b-8c84fc9ac415"
    ms.service="backup"
    ms.workload="storage-backup-recovery"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="11/28/2016"
    wacn.date="01/24/2017"
    ms.author="saurse;markgal;jimpark;nkolli;trinadhk" />  


# 使用 PowerShell 部署和管理 Windows Server/Windows 客户端的 Azure 备份

> [AZURE.SELECTOR]
- [ARM](/documentation/articles/backup-client-automation/)
- [经典](/documentation/articles/backup-client-automation-classic/)

本文说明如何使用 PowerShell 在 Windows Server 或 Windows 客户端上设置 Azure 备份，以及管理备份和恢复。

## 安装 Azure PowerShell

[AZURE.INCLUDE [了解部署模型](../../includes/learn-about-deployment-models-include.md)]

本文重点介绍可用于在资源组中使用恢复服务保管库的 Azure资源管理器(ARM) PowerShell cmdlet。

Azure PowerShell 1.0 已在 2015 年 10 月发布。此版本在 0.9.8 版本的基础上进行了一些重大的更改，尤其是对 cmdlet 的命名方式进行了更改。1.0 版 cmdlet 遵循命名模式 {谓词}-AzureRm{名词}；而 0.9.8 名称不包括 **Rm**（例如 New-AzureResourceGroup 取代了 New-AzureRmResourceGroup）。使用 Azure PowerShell 0.9.8 时，首先必须通过运行 **Switch-AzureMode AzureResourceManager** 命令启用资源管理器模式。1.0 或更高版中并不需要此命令在。

若要将针对 0.9.8 环境编写的脚本用在 1.0 或更高版本的环境中，应在预生产环境中对脚本进行仔细的测试，然后才能将其用在生产中，以免造成意外影响。

[下载最新的 PowerShell 版本](https://github.com/Azure/azure-powershell/releases)（所需最低版本：1.0.0）

[AZURE.INCLUDE [arm-getting-setup-powershell](../../includes/arm-getting-setup-powershell.md)]

## 创建恢复服务保管库
以下步骤将引导用户创建恢复服务保管库。恢复服务保管库不同于备份保管库。

1. 如果是首次使用 Azure 备份，则必须使用 **Register-AzureRMResourceProvider** cmdlet 将 Azure 恢复服务提供程序注册到订阅。

    	PS C:\> Register-AzureRmResourceProvider -ProviderNamespace "Microsoft.RecoveryServices"

2. 恢复服务保管库是一种 ARM 资源，因此需要将它放在资源组中。可以使用现有资源组，也可以创建新组。创建新的资源组时，请指定资源组的名称和位置。

    	PS C:\> New-AzureRmResourceGroup -Name "test-rg" -Location "China North"

3. 使用 **New-AzureRmRecoveryServicesVault** cmdlet 创建新的保管库。确保为保管库指定的位置与用于资源组的位置是相同的。

    	PS C:\> New-AzureRmRecoveryServicesVault -Name "testvault" -ResourceGroupName " test-rg" -Location "China North"

4. 指定要使用的存储冗余类型；你可以使用[本地冗余存储 (LRS)](/documentation/articles/storage-redundancy/#locally-redundant-storage/) 或[异地冗余存储 (GRS)](/documentation/articles/storage-redundancy/#geo-redundant-storage/)。以下示例显示，testVault 的 -BackupStorageRedundancy 选项设置为 GeoRedundant。
   
   > [AZURE.TIP]
   许多 Azure 备份 cmdlet 要求使用恢复服务保管库对象作为输入。因此，在变量中存储备份恢复服务保管库对象可提供方便。
   > 
   > 
   
	PS C:\> $vault1 = Get-AzureRmRecoveryServicesVault -Name "testVault"
	PS C:\> Set-AzureRmRecoveryServicesBackupProperties  -vault $vault1 -BackupStorageRedundancy GeoRedundant


## 在订阅中查看保管库
使用 **Get-AzureRmRecoveryServicesVault** 查看当前订阅中所有保管库的列表。可以使用此命令来查看是否创建了新的保管库，或者查看订阅中的可用保管库。

运行 Get-AzureRmRecoveryServicesVault 命令即可列出订阅中的所有保管库。

	PS C:\> Get-AzureRmRecoveryServicesVault
	Name              : Contoso-vault
	ID                : /subscriptions/1234
	Type              : Microsoft.RecoveryServices/vaults
	Location          : ChinaNorth
	ResourceGroupName : Contoso-docs-rg
	SubscriptionId    : 1234-567f-8910-abc
	Properties        : Microsoft.Azure.Commands.RecoveryServices.ARSVaultProperties


## 安装 Azure 备份代理
在安装 Azure 备份代理之前，必须先将安装程序下载到 Windows Server 上。可以从 [Microsoft 下载中心](http://aka.ms/azurebackup_agent)或恢复服务保管库的“仪表板”页获取最新版本的安装程序。将安装程序保存到方便访问的位置，例如 *C:\\Downloads*。

若要安装代理，请在已提升权限的 PowerShell 控制台中运行以下命令：

	PS C:\> MARSAgentInstaller.exe /q

这将以所有默认选项安装代理。安装将在几分钟内在后台完成。如果不指定 */nu* 选项，则安装结束时，将打开“Windows 更新”窗口以检查更新。安装之后，代理将显示在已安装程序列表中。

若要查看已安装程序的列表，请转到“控制面板”>“程序”>“程序和功能”。

![已安装代理](./media/backup-client-automation/installed-agent-listing.png)

### 安装选项

若要查看可通过命令行运行的所有选项，请使用以下命令：

	PS C:\> MARSAgentInstaller.exe /?

可用选项包括：

| 选项 | 详细信息 | 默认 |
| ---- | ----- | ----- |
| /q | 静默安装 | - | 
| /p:"location" | Azure 备份代理的安装文件夹路径。| C:\\Program Files\\Microsoft Azure Recovery Services Agent | 
| /s:"location" | Azure 备份代理的缓存文件夹路径。| C:\\Program Files\\Microsoft Azure Recovery Services Agent\\Scratch | 
| /m | 选择启用 Microsoft 更新 | - | 
| /nu | 安装完成后不要检查更新 | - | 
| /d | 卸载 Azure 恢复服务代理 | - | 
| /ph | 代理主机地址 | - | 
| /po | 代理主机端口号 | - | 
| /pu | 代理主机用户名 | - | 
| /pw | 代理密码 | - |


## 将 Windows Server 或 Windows 客户端计算机注册到恢复服务保管库

创建恢复服务保管库后，请下载最新代理和保管库凭据，并将其存储在便于访问的位置（如 C:\\Downloads）。

	PS C:\> $credspath = "C:\downloads"
	PS C:\> $credsfilename = Get-AzureRmRecoveryServicesVaultSettingsFile -Backup -Vault $vault1 -Path  $credspath
	PS C:\> $credsfilename C:\downloads\testvault\_Sun Apr 10 2016.VaultCredentials

在 Windows Server 或 Windows 客户端计算机上，运行 [Start-OBRegistration](https://technet.microsoft.com/zh-cn/library/hh770398%28v=wps.630%29.aspx) cmdlet 以将计算机注册到保管库。

	PS C:\> $cred = $credspath + $credsfilename
	PS C:\> Start-OBRegistration-VaultCredentials $cred -Confirm:$false
	CertThumbprint      :7a2ef2caa2e74b6ed1222a5e89288ddad438df2
	SubscriptionID      : ef4ab577-c2c0-43e4-af80-af49f485f3d1
	ServiceResourceName: testvault
	Region              :China North
	Machine registration succeeded.

> [AZURE.IMPORTANT]
请勿使用相对路径来指定保管库凭据文件。必须提供绝对路径作为 cmdlet 的输入。
> 
> 

## 网络设置
如果 Windows 计算机通过代理服务器连接到 Internet，则也可以向代理提供代理设置。此示例未使用代理服务器，因此我们会显式清除任何与代理相关的信息。

也可针对给定的一组“星期几”，使用 ```work hour bandwidth``` 和 ```non-work hour bandwidth``` 选项来控制带宽使用。

使用 [Set-OBMachineSetting](https://technet.microsoft.com/zh-cn/library/hh770409%28v=wps.630%29.aspx) cmdlet 设置代理和带宽详细信息：

	PS C:\> Set-OBMachineSetting -NoProxy
	Server properties updated successfully.

	PS C:\> Set-OBMachineSetting -NoThrottle
	Server properties updated successfully.

## 加密设置
发送到 Azure 备份的备份数据将会加密，以保护数据的机密性。加密通行短语是在还原时用于解密数据的“密码”。

	PS C:\> ConvertTo-SecureString -String "Complex!123_STRING" -AsPlainText -Force | Set-OBMachineSetting
	Server properties updated successfully

> [AZURE.IMPORTANT]
请妥善保管设置好的通行短语，并保证其安全。如果没有此通行短语，则无法从 Azure 还原数据。
> 
> 

## 备份文件和文件夹
从 Windows Server 和客户端到 Azure 备份的所有备份由策略控制。策略由三个部分组成：

1. **备份计划**，用于指定需要备份并与服务保持同步的时间。
2. **保留计划**，用于指定要在 Azure 中保留恢复点的时长。
3. **文件包含/排除规范**，用于指示应备份的内容。

在本文档中，由于自动备份，因此假设尚未配置任何选项。首先，使用 [New-OBPolicy](https://technet.microsoft.com/zh-cn/library/hh770416.aspx) cmdlet 创建新的备份策略。

	PS C:\> $newpolicy = New-OBPolicy

该策略暂时为空，需要使用其他 cmdlet 来定义要包含或排除的项、运行备份的时间，以及备份的存储位置。

### 配置备份计划
在策略的 3 个组成部分中，第 1 部分是使用 [New-OBSchedule](https://technet.microsoft.com/zh-cn/library/hh770401) cmdlet 创建的备份计划。备份计划将定义何时需要备份。创建计划时，需要指定 2 个输入参数：

- 应在“星期几”运行备份。可以选择只在一周的某一天、一周的每一天或其中任意日期组合运行备份作业。
- 应在“一天中的何时”运行备份。最多可以定义一天的 3 个不同日期时间来触发备份

例如，可以配置在每个星期六和星期日下午 4 点运行备份策略。

	PS C:\> $sched = New-OBSchedule -DaysofWeek Saturday, Sunday -TimesofDay 16:00

备份计划需要与策略相关联，这可以使用 [Set-OBSchedule](https://technet.microsoft.com/zh-cn/library/hh770407) cmdlet 来实现。

	PS C:\> Set-OBSchedule -Policy $newpolicy -Schedule $sched
	BackupSchedule : 4:00 PM Saturday, Sunday, Every 1 week(s) DsList : PolicyName : RetentionPolicy : State : New PolicyState : Valid

### 配置保留策略
保留策略定义基于备份作业创建的恢复点的保留时间。使用 [New-OBRetentionPolicy](https://technet.microsoft.com/zh-cn/library/hh770425) cmdlet 创建新的保留策略时，可以指定 Azure 备份需要保留备份恢复点的天数。以下示例将保留策略设置为 7 天。

	PS C:\> $retentionpolicy = New-OBRetentionPolicy -RetentionDays 7

必须使用 [Set-OBRetentionPolicy](https://technet.microsoft.com/zh-cn/library/hh770405) cmdlet 将保留策略与主要策略相关联：

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
```OBFileSpec``` 对象定义备份中要包含和排除的文件。这组规则可划分出计算机上要保护的文件和文件夹。可设置任意数量的文件包含或排除规则，并将其与策略相关联。创建新的 OBFileSpec 对象时，可执行以下操作：

- 指定要包含的文件和文件夹
- 指定要排除的文件和文件夹
- 指定要递归备份文件夹中的数据，或是否仅备份指定文件夹中的顶级文件。

可以在 New-OBFileSpec 命令中使用 -NonRecursive 标志来完成后一种指定。

在以下示例中，备份卷 C: 和 D:，并排除 Windows 文件夹和任何临时文件夹中的操作系统二进制文件。为此，使用 [New-OBFileSpec](https://technet.microsoft.com/zh-cn/library/hh770408) cmdlet 创建两个文件规范 - 一个用于包含，一个用于排除。创建文件规范后，使用 [Add-OBFileSpec](https://technet.microsoft.com/zh-cn/library/hh770424) cmdlet 将它们与策略相关联。

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
现在已完成策略对象，并且具有关联的备份计划、保留策略及文件包含/排除列表。现在可以提交此策略以供 Azure 备份使用。应用新建策略之前，请使用 [Remove-OBPolicy](https://technet.microsoft.com/zh-cn/library/hh770415) cmdlet 确保没有与服务器相关联的现有备份策略。删除策略时，系统将提示用户确认。若要跳过确认，请在 cmdlet 中请使用 ```-Confirm:$false``` 标志。

	PS C:\> Get-OBPolicy | Remove-OBPolicy
	Azure 备份 确定要删除此备份策略吗？这将删除所有已备份的数据。 [Y] 是 [A] 全部是 [N] 否 [L] 全部否 [S] 挂起 [?] 帮助（默认值为“Y”）：

使用 [Set-OBPolicy](https://technet.microsoft.com/zh-cn/library/hh770421) cmdlet 可以提交策略对象。这也将提示用户确认。若要跳过确认，请在 cmdlet 中请使用 ```-Confirm:$false``` 标志。

	PS C:\> Set-OBPolicy -Policy $newpolicy
	Azure Backup Do you want to save this backup policy ? [Y] Yes [A] Yes to All [N] No [L] No to All [S] Suspend [?] Help (default is "Y"):
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
设置备份策略之后，将会根据计划进行备份。也可使用 [Start-OBBackup](https://technet.microsoft.com/zh-cn/library/hh770426) cmdlet 来触发即席备份：

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
本部分将引导用户完成自动从 Azure 备份恢复数据的步骤。此过程涉及以下步骤：

1. 选取源卷
2. 选择要还原的备份点
3. 选择要还原的项
4. 触发还原过程

### 选取源卷
若要从 Azure 备份还原某个项，需要先识别该项的源。由于我们要在 Windows Server 或 Windows 客户端的上下文中执行命令，因此已识别了计算机。识别源的下一步是识别它所在的卷。可通过运行 [Get-OBRecoverableSource](https://technet.microsoft.com/zh-cn/library/hh770410) cmdlet，检索从此计算机备份的卷或源的列表。此命令将返回从此服务器/客户端备份的所有源的数组。

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

对象 ```$rps``` 是备份点数组。第一个元素是最新备份点，第 N 个元素是最旧的备份点。为了选择最新的点，使用 ```$rps[0]```。

### 选择要还原的项
为了识别要还原的确切文件或文件夹，请以递归方式使用 [Get-OBRecoverableItem](https://technet.microsoft.com/zh-cn/library/hh770399.aspx) cmdlet。这样，只需使用 ```Get-OBRecoverableItem``` 便可浏览文件夹层次结构。

在本示例中，如果要还原文件 *finances.xls*，可以使用对象 ```$filesFolders[1]``` 来引用该文件。

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

也可使用 ```Get-OBRecoverableItem``` cmdlet 来搜索要还原的项。在本示例中，为了搜索 *finances.xls*，可通过运行以下命令来获取该文件上的句柄：

	PS C:\> $item = Get-OBRecoverableItem -RecoveryPoint $rps[0] -Location "D:\MyData" -SearchString "finance*"

### 触发还原过程
为了触发还原过程，首先需要指定恢复选项。可以使用 [New-OBRecoveryOption](https://technet.microsoft.com/zh-cn/library/hh770417.aspx) cmdlet 来完成此操作。在本示例中，假设要将文件还原到 *C:\\temp*。此外，假设要跳过目标文件夹 *C:\\temp* 中已存在的文件。若要创建此类恢复选项，请使用以下命令：

	PS C:\> $recovery_option = New-OBRecoveryOption -DestinationPath "C:\temp" -OverwriteType Skip

现在，请对 ```Get-OBRecoverableItem``` cmdlet 输出中的选定 ```$item``` 使用 [Start-OBRecovery](https://technet.microsoft.com/zh-cn/library/hh770402.aspx) 命令来触发还原：

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
- 所有的策略信息将从计算机中删除，但服务中会继续存储这些策略信息。
- 将删除所有备份计划，且不会进一步创建备份。

不过，将根据设置的保留策略继续保留 Azure 中存储的数据。较旧的恢复点会自动过时。

## 远程管理
围绕 Azure 备份代理、策略和数据源的所有管理工作都可通过 Azure PowerShell 远程完成。要远程管理的计算机需要经过适当的准备。

默认情况下，WinRM 服务已配置为手动启动。必须将启动类型设置为“自动”，并且应启动该服务。若要确认 WinRM 服务正在运行，“状态”属性的值应是“正在运行”。

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

- [Azure 备份简介](/documentation/articles/backup-introduction-to-azure-backup/)
- [备份 Windows Server](/documentation/articles/backup-configure-vault/)

<!---HONumber=Mooncake_0116_2017-->
<!---Update_Description: wording update -->
