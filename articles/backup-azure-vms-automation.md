<properties
	pageTitle="通过 PowerShell 为 Azure VM 部署和管理备份 | Azure"
	description="了解如何使用 PowerShell 部署和管理 Azure 备份"
	services="backup"
	documentationCenter=""
	authors="markgalioto"
	manager="jwhit"
	editor=""/>

<tags
	ms.service="backup"
	ms.date="05/09/2016"
	wacn.date="06/06/2016" />


# 通过 PowerShell 为 Azure VM 部署和管理备份
本文说明如何使用 Azure PowerShell 从恢复服务保管库备份和恢复 Azure 虚拟机 (VM)。恢复服务保管库是一种 Azure Resource Manager (ARM) 资源，用于保护 Azure 备份和 Azure Site Recovery 服务中的数据与资产。在 ARM 部署中操作时，请使用恢复服务保管库。可以使用恢复服务保管库来保护 Azure 服务管理器 ASM 部署的 VM 以及 ARM VM。

>[AZURE.NOTE] Azure 有两种用于创建和使用资源的部署模型：[Resource Manager 和经典部署模型](/documentation/articles/resource-manager-deployment-model/)。本文的内容与使用 Resource Manager 模型创建的 VM 有关。

本文将逐步指导你使用 PowerShell 来保护 VM，以及从恢复点还原数据。

## 概念

如果你不熟悉 Azure 备份服务，请查看 [What is Azure Backup?（什么是 Azure 备份）](/documentation/articles/backup-introduction-to-azure-backup/)，以获取有关该服务的概述 在开始之前，请确保你已掌握与系统必备组件相关的基础知识（这些必备组件是使用 Azure 备份所必需的），并了解当前 VM 备份解决方案的限制。

为了提高 PowerShell 使用效率，必须了解对象的层次结构以及从何处开始。

![恢复服务对象层次结构](./media/backup-azure-vms-arm-automation/recovery-services-object-hierarchy.png)

## 设置和注册

开始时，请执行以下操作：

1. [下载最新版本的 PowerShell](https://github.com/Azure/azure-powershell/releases)（所需的最低版本：1.0.0）

2. 键入以下命令查找可用的 Azure 备份 PowerShell cmdlet：

		
		PS C:\WINDOWS\system32> Get-Command *azurermrecoveryservices*
		
		CommandType     Name                                               Version    Source
		-----------     ----                                               -------    ------
		Cmdlet          Backup-AzureRmRecoveryServicesBackupItem           1.0.0      AzureRM.RecoveryServices.Backup
		Cmdlet          Disable-AzureRmRecoveryServicesBackupProtection    1.0.0      AzureRM.RecoveryServices.Backup
		Cmdlet          Enable-AzureRmRecoveryServicesBackupProtection     1.0.0      AzureRM.RecoveryServices.Backup
		Cmdlet          Get-AzureRmRecoveryServicesBackupContainer         1.0.0      AzureRM.RecoveryServices.Backup
		Cmdlet          Get-AzureRmRecoveryServicesBackupItem              1.0.0      AzureRM.RecoveryServices.Backup
		Cmdlet          Get-AzureRmRecoveryServicesBackupJob               1.0.0      AzureRM.RecoveryServices.Backup
		Cmdlet          Get-AzureRmRecoveryServicesBackupJobDetails        1.0.0      AzureRM.RecoveryServices.Backup
		Cmdlet          Get-AzureRmRecoveryServicesBackupManagementServer  1.0.0      AzureRM.RecoveryServices.Backup
		Cmdlet          Get-AzureRmRecoveryServicesBackupProperties        1.0.7      AzureRM.RecoveryServices
		Cmdlet          Get-AzureRmRecoveryServicesBackupProtectionPolicy  1.0.0      AzureRM.RecoveryServices.Backup
		Cmdlet          Get-AzureRMRecoveryServicesBackupRecoveryPoint     1.0.0      AzureRM.RecoveryServices.Backup
		Cmdlet          Get-AzureRmRecoveryServicesBackupRetentionPolic... 1.0.0      AzureRM.RecoveryServices.Backup
		Cmdlet          Get-AzureRmRecoveryServicesBackupSchedulePolicy... 1.0.0      AzureRM.RecoveryServices.Backup
		Cmdlet          Get-AzureRmRecoveryServicesVault                   1.0.7      AzureRM.RecoveryServices
		Cmdlet          Get-AzureRmRecoveryServicesVaultSettingsFile       1.0.7      AzureRM.RecoveryServices
		Cmdlet          New-AzureRmRecoveryServicesBackupProtectionPolicy  1.0.0      AzureRM.RecoveryServices.Backup
		Cmdlet          New-AzureRmRecoveryServicesVault                   1.0.7      AzureRM.RecoveryServices
		Cmdlet          Remove-AzureRmRecoveryServicesProtectionPolicy     1.0.0      AzureRM.RecoveryServices.Backup
		Cmdlet          Remove-AzureRmRecoveryServicesVault                1.0.7      AzureRM.RecoveryServices
		Cmdlet          Restore-AzureRMRecoveryServicesBackupItem          1.0.0      AzureRM.RecoveryServices.Backup
		Cmdlet          Set-AzureRmRecoveryServicesBackupProperties        1.0.7      AzureRM.RecoveryServices
		Cmdlet          Set-AzureRmRecoveryServicesBackupProtectionPolicy  1.0.0      AzureRM.RecoveryServices.Backup
		Cmdlet          Set-AzureRmRecoveryServicesVaultContext            1.0.7      AzureRM.RecoveryServices
		Cmdlet          Stop-AzureRmRecoveryServicesBackupJob              1.0.0      AzureRM.RecoveryServices.Backup
		Cmdlet          Unregister-AzureRmRecoveryServicesBackupContainer  1.0.0      AzureRM.RecoveryServices.Backup
		Cmdlet          Unregister-AzureRmRecoveryServicesBackupManagem... 1.0.0      AzureRM.RecoveryServices.Backup
		Cmdlet          Wait-AzureRmRecoveryServicesBackupJob              1.0.0      AzureRM.RecoveryServices.Backup


使用 PowerShell 可以自动化以下设置和注册任务：

- 创建恢复服务保管库
- 备份或保护 Azure VM
- 触发备份作业
- 监视备份作业
- 还原 Azure VM

## 创建恢复服务保管库

> [AZURE.TIP] 对于首次使用 Azure 备份的客户，必须注册用于订阅的 Azure 恢复服务提供程序。可通过运行以下命令来执行此操作：Register-AzureRmResourceProvider -ProviderNamespace "Microsoft.RecoveryServices"

可以使用 **New-AzureRmRecoveryServicesVault** cmdlet 新建一个恢复服务保管库。恢复服务保管库是一种 ARM 资源，因此需要将它放在资源组中。在权限提升的 Azure PowerShell 控制台中运行以下命令：

		
		PS C:\> New-AzureRmResourceGroup –Name "test-rg" –Location "West US"
		PS C:\> New-AzureRmRecoveryServicesVault -Name "testvault" -ResourceGroupName " test-rg" -Location "West US"


若要查看订阅中所有恢复服务保管库的列表，请使用 **Get-AzureRmRecoveryServicesVault** cmdlet。

### 设置存储冗余
创建恢复服务保管库时，请指定要使用的存储冗余类型 - 本地冗余存储 (LRS) 或异地冗余存储 (GRS)。以下示例显示了 testVault 设置为 GeoRedundant 的 BackupStorageRedundancy 选项。

		
		PS C:\> $vault1 = Get-AzureRmRecoveryServicesVault –Name "testVault"
		PS C:\> Set-AzureRmRecoveryServicesBackupProperties  -vault $vault1 -BackupStorageRedundancy GeoRedundant



> [AZURE.TIP] 许多 Azure 备份 cmdlet 要求使用恢复服务保管库对象作为输入。出于此原因，在变量中存储备份恢复服务保管库对象可提供方便。


## 备份 Azure VM

在 VM 上启用保护之前，必须设置保管库上下文。该上下文将应用到所有后续 cmdlet。


		PS C:\> Get-AzureRmRecoveryServicesVault -Name testvault | Set-AzureRmRecoveryServicesVaultContext


### 创建保护策略

当你创建新保管库时，它附带了一个默认策略。此策略会在每天的下午 9:30 触发备份作业。备份快照将保留 30 天。可以使用默认策略快速保护你的 VM，以后再使用不同的详细信息编辑该策略。

若要查看保管库中的可用策略列表，请使用 Get-AzureRmRecoveryServicesBackupProtectionPolicy cmdlet：

		
		PS C:\WINDOWS\system32> get-AzureRMRecoveryServicesBackupProtectionPolicy -WorkloadType AzureVM
		Name                 WorkloadType       BackupManagementType BackupTime                DaysOfWeek
		----                 ------------       -------------------- ----------                ----------
		DefaultPolicy        AzureVM            AzureVM              4/14/2016 5:00:00 PM


> [AZURE.NOTE] PowerShell 中 BackupTime 字段的时区是 UTC。但是，在 Azure 经典管理门户中显示备份时间时，该时间将会根据你的本地时区调整。

一个备份保护策略至少与一个保留策略相关联。保留策略定义在 Azure 备份中保留恢复点的时限。使用 Get-AzureRmRecoveryServicesBackupRetentionPolicyObject 可以查看默认保留策略。同理，可以使用 Get-AzureRmRecoveryServicesBackupSchedulePolicyObject 获取默认计划策略。计划和保留策略对象将用作 New-AzureRmRecoveryServicesBackupProtectionPolicy cmdlet 的输入。

备份保护策略定义对某个项目进行备份的时间和频率。New-AzureRmRecoveryServicesBackupProtectionPolicy cmdlet 创建用于保存备份策略信息的 PowerShell 对象。该备份策略用作 Enable-AzureRmRecoveryServicesBackupProtection cmdlet 的输入。


		PS C:\> $schPol = Get-AzureRmRecoveryServicesBackupSchedulePolicyObject -WorkloadType "AzureVM"
		PS C:\>  $retPol = Get-AzureRmRecoveryServicesBackupRetentionPolicyObject -WorkloadType "AzureVM"
		PS C:\>  New-AzureRmRecoveryServicesBackupProtectionPolicy -Name "NewPolicy" -WorkloadType AzureVM -RetentionPolicy $retPol -SchedulePolicy $schPol
		Name                 WorkloadType       BackupManagementType BackupTime                DaysOfWeek
		----                 ------------       -------------------- ----------                ----------
		NewPolicy           AzureVM            AzureVM              4/24/2016 1:30:00 AM


### 启用保护

启用保护涉及两个对象 - 项和策略。需要提供这两个对象才能为保管库启用保护。将策略与项关联之后，将在策略计划中定义的时间触发备份工作流。


		PS C:\> $pol=Get-AzureRmRecoveryServicesBackupProtectionPolicy -Name "NewPolicy"
		PS C:\> Enable-AzureRmRecoveryServicesBackupProtection -Policy $pol -Name "V2VM" -ResourceGroupName "RGName1"


对于基于 ASM 的 VM


		PS C:\>  $pol=Get-AzureRmRecoveryServicesBackupProtectionPolicy -Name "NewPolicy"
		PS C:\>  Enable-AzureRmRecoveryServicesBackupProtection -Policy $pol -Name "V1VM" -ServiceName "ServiceName1"


### 修改保护策略

若要修改策略，请修改 BackupSchedulePolicyObject 或 BackupRetentionPolicy 对象，并使用 Set-AzureRmRecoveryServicesBackupProtectionPolicy 修改策略

以下示例将保留计数更改为 365。


		PS C:\> $retPol = Get-AzureRmRecoveryServicesBackupRetentionPolicyObject -WorkloadType "AzureVM"
		PS C:\> $retPol.DailySchedule.DurationCountInDays = 365
		PS C:\> $pol= Get-AzureRMRecoveryServicesBackupProtectionPolicy -Name NewPolicy
		PS C:\> Set-AzureRmRecoveryServicesBackupProtectionPolicy -Policy $pol  -RetentionPolicy  $RetPol


## 运行初始备份

在首次备份项时，备份计划将触发完整备份。对于后续备份，备份形式为增量复制。如果你想要强制初始备份在某个时间发生甚至立刻发生，可以使用 Backup-AzureRmRecoveryServicesBackupItem cmdlet：


		PS C:\> $namedContainer = Get-AzureRmRecoveryServicesBackupContainer -ContainerType "AzureVM" -Status "Registered" -Name "V2VM";
		PS C:\> $item = Get-AzureRmRecoveryServicesBackupItem -Container $namedContainer -WorkloadType "AzureVM";
		PS C:\> $job = Backup-AzureRmRecoveryServicesBackupItem -Item $item;
		WorkloadName     Operation            Status               StartTime                 EndTime                   JobID
		------------     ---------            ------               ---------                 -------                   ----------
		V2VM        Backup               InProgress            4/23/2016 5:00:30 PM            cf4b3ef5-2fac-4c8e-a215-d2eba4124f27


> [AZURE.NOTE]：PowerShell 中的 StartTime 和 EndTime 字段时区为 UTC。但是，在 Azure 经典管理门户中显示时间时，该时间将会根据你的本地时区调整。

## 监视备份作业

在 Azure 备份中，大多数长时间运行的操作都是作为作业来建模的。这样可以轻松地跟踪相关进度，不必让 Azure 经典管理门户始终打开。

若要获取正在进行的作业的最新状态，请使用 Get-AzureRMRecoveryservicesBackupJob cmdlet。


		PS C:\ > $joblist = Get-AzureRMRecoveryservicesBackupJob –Status InProgress
		PS C:\ > $joblist[0]
		WorkloadName     Operation            Status               StartTime                 EndTime                   JobID
		------------     ---------            ------               ---------                 -------                   ----------
		V2VM        Backup               InProgress            4/23/2016 5:00:30 PM           cf4b3ef5-2fac-4c8e-a215-d2eba4124f27
		

与其使用额外的不必要的代码来轮询这些作业的完成情况，不如使用更简单的方式：Wait-AzureRmRecoveryServicesBackupJob cmdlet。在脚本中使用时，该 cmdlet 会暂停操作的执行，直到作业完成或达到了指定的超时值。


		PS C:\> Wait-AzureRmRecoveryServicesBackupJob -Job $joblist[0] -Timeout 43200



## <a name="restore-an-azure-vm"></a>还原 Azure VM

若要还原备份数据，需要确定已备份项以及保留了时间点数据的恢复点。此信息将提供给 Restore-AzureRMRecoveryServicesBackupItem cmdlet，以启动还原过程，将数据从保管库还原到客户的帐户。

### 选择 VM

若要获取用于标识正确备份项的 PowerShell 对象，请从保管库中的容器开始，按对象层次结构进行操作。若要选择代表 VM 的容器，请使用 Get-AzureRmRecoveryServicesBackupContainer cmdlet，然后通过管道将其传递给 Get-AzureRmRecoveryServicesBackupItem cmdlet。


		PS C:\> $namedContainer = Get-AzureRmRecoveryServicesBackupContainer  -ContainerType AzureVM –Status Registered -Name 'V2VM'
		PS C:\> $backupitem=Get-AzureRmRecoveryServicesBackupItem –Container $namedContainer  –WorkloadType "AzureVM"


### 选择恢复点

现在可以使用 Get-AzureRMRecoveryServicesBackupRecoveryPoint cmdlet 列出备份项的所有恢复点，然后选择要还原的恢复点。通常情况下，用户会选取列表中在时间上最近的 AppConsistent 点。


		PS C:\> $startDate = (Get-Date).AddDays(-7)
		PS C:\> $endDate = Get-Date
		PS C:\> $rp = Get-AzureRMRecoveryServicesBackupRecoveryPoint -Item $backupitem -StartDate $startdate.ToUniversalTime() -EndDate $enddate.ToUniversalTime()
		PS C:\> $rp[0]
		RecoveryPointAdditionalInfo :
		SourceVMStorageType         : NormalStorage
		Name                        : 15260861925810
		ItemName                    : VM;iaasvmcontainer;RGName1;V2VM
		RecoveryPointId             : /subscriptions/XX/resourceGroups/ RGName1/providers/Microsoft.RecoveryServices/vaults/testvault/backupFabrics/Azure/protectionContainers/IaasVMContainer;iaasvmcontainer;RGName1;V2VM/protectedItems/VM;iaasvmcontainer; RGName1;V2VM
		                              /recoveryPoints/15260861925810
		RecoveryPointType           : AppConsistent
		RecoveryPointTime           : 4/23/2016 5:02:04 PM
		WorkloadType                : AzureVM
		ContainerName               : IaasVMContainer;iaasvmcontainer; RGName1;V2VM
		ContainerType               : AzureVM
		BackupManagementType        : AzureVM


变量 $rp 是选定备份项的恢复点数组，已按时间逆序排序 - 最新的恢复点位于索引 0 处。使用标准 PowerShell 数组索引选取恢复点。例如：$rp[0] 将选择最新的恢复点。

### 还原磁盘

通过 Azure 经典管理门户执行还原操作与通过 Azure PowerShell 执行还原操作存在很大的不同。如果使用 PowerShell，还原操作将在从恢复点还原磁盘和配置信息时停止。它不会创建虚拟机。

> [AZURE.WARNING] Restore-AzureRMRecoveryServicesBackupItem 不会创建 VM，而只是将磁盘还原到指定的存储帐户。这不同于 Azure 经典管理门户中发生的情况。


		PS C:\> $restorejob = Restore-AzureRMRecoveryServicesBackupItem -RecoveryPoint $rp[0] -StorageAccountName DestAccount
		 -StorageAccountResourceGroupName DestRG
		PS C:\> $restorejob
		WorkloadName     Operation            Status               StartTime                 EndTime                   JobID
		------------     ---------            ------               ---------                 -------                   ----------
		V2VM        Restore               InProgress            4/23/2016 5:00:30 PM           cf4b3ef5-2fac-4c8e-a215-d2eba4124f27
		

还原作业完成后，可以使用 Get-AzureRmRecoveryServicesBackupJobDetails cmdlet 获取还原操作的详细信息。JobDetails 属性将提供重建 VM 所需的信息。


		PS C:\> $restorejob = Get-AzureRmRecoveryServicesBackupJob -Job $restorejob
		PS C:\> $details = Get-AzureRmRecoveryServicesBackupJobDetails


## 将 Windows Server 或 DPM 注册到恢复服务保管库

创建恢复服务保管库后，请下载最新的代理和保管库凭据，并将其存储在一个方便访问的位置（如 C:\\Downloads）。


		PS C:\> $credspath = "C:\downloads"
		PS C:\> $credsfilename = Get-AzureRmRecoveryServicesVaultSettingsFile -Backup -Vault $vault1 -Path  $credspath
		PS C:\> $credsfilename
		C:\downloads\testvault\_Sun Apr 10 2016.VaultCredentials


在 Windows Server 或 DPM 服务器上，运行 [Start-OBRegistration](https://technet.microsoft.com/library/hh770398%28v=wps.630%29.aspx) cmdlet 以将计算机注册到保管库。


	PS C:\> $cred = $credspath + $credsfilename
	PS C:\> Start-OBRegistration-VaultCredentials $cred -Confirm:$false
	CertThumbprint      :7a2ef2caa2e74b6ed1222a5e89288ddad438df2
	SubscriptionID      : ef4ab577-c2c0-43e4-af80-af49f485f3d1
	ServiceResourceName: testvault
	Region              :West US
	Machine registration succeeded.


<!---HONumber=Mooncake_0530_2016-->