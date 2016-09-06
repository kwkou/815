<properties
	pageTitle="初步了解：使用恢复服务保管库保护 Azure VM | Azure"
	description="使用恢复服务保管库保护 Azure VM。使用 Resource Manager 部署型 VM、经典部署型 VM 和高级存储 VM 的备份来保护数据。创建并注册恢复服务保管库。在 Azure 中注册 VM、创建策略和保护 VM。"
	services="backup"
	documentationCenter=""
	authors="markgalioto"
	manager="cfreeman"
	editor=""
	keyword="backups; vm backup"/>

<tags
	ms.service="backup"
	ms.date="06/03/2016"
	wacn.date="09/06/2016"/>  



# 初步了解：使用恢复服务保管库保护 Azure VM

> [AZURE.SELECTOR]
- [初步了解：使用恢复服务保管库保护 VM](/documentation/articles/backup-azure-vms-first-look-arm/)
- [初步了解：使用备份保管库保护 Azure VM](/documentation/articles/backup-azure-vms-first-look/)

本教程将引导你完成创建恢复服务保管库和备份 Azure 虚拟机 (VM) 的步骤。恢复服务保管库保护：

- Azure Resource Manager 部署型 VM
- 经典 VM
- 标准存储 VM
- 高级存储 VM
- 使用 Azure 磁盘加密进行加密的 VM，以及 BEK 和 KEK（使用 Powershell 时受支持）

有关保护高级存储 VM 的详细信息，请参阅[备份和还原高级存储 VM](/documentation/articles/backup-introduction-to-azure-backup/#back-up-and-restore-premium-storage-vms)

>[AZURE.NOTE] 本教程假设 Azure 订阅中已有 VM，且已采取措施允许备份服务访问 VM。Azure 有两种用于创建和使用资源的部署模型：[Resource Manager 部署模型和经典部署模型](/documentation/articles/resource-manager-deployment-model/)。本文适用于 Resource Manager 和 Resource Manager 部署型 VM。

概括而言，你将要完成以下这些步骤。

1. 为 VM 创建恢复服务保管库。
2. 使用 Azure 门户选择方案、设置策略，并标识要保护的项。
3. 运行初始备份。



## 步骤 1 - 为 VM 创建恢复服务保管库

1. 首先使用以下命令行登录你的 Azure 订阅。

        Login-AzureRmAccount -EnvironmentName AzureChinaCloud

1. 如果你是首次使用 Azure 备份，则必须使用 **[Register-AzureRMResourceProvider](https://msdn.microsoft.com/zh-cn/library/mt603685.aspx)** cmdlet 注册用于订阅的 Azure 恢复服务提供程序。

        Register-AzureRmResourceProvider -ProviderNamespace "Microsoft.RecoveryServices"

2. 恢复服务保管库是一种 Resource Manager 资源，因此需要将它放在资源组中。你可以使用现有的资源组，也可以使用 **[New-AzureRmResourceGroup](https://msdn.microsoft.com/zh-cn/library/mt603739.aspx)** cmdlet 创建新的资源组。创建新的资源组时，请指定资源组的名称和位置。

        New-AzureRmResourceGroup –Name "test-rg" –Location "China North"

3. 使用 **[New-AzureRmRecoveryServicesVault](https://msdn.microsoft.com/zh-cn/library/mt643910.aspx)** cmdlet 创建新的保管库。确保为保管库指定的位置与用于资源组的位置是相同的。

        New-AzureRmRecoveryServicesVault -Name "testvault" -ResourceGroupName " test-rg" -Location "West US"

4. 指定要使用的存储冗余类型；你可以使用[本地冗余存储 (LRS)](/documentation/articles/storage-redundancy/#locally-redundant-storage) 或[异地冗余存储 (GRS)](/documentation/articles/storage-redundancy/#geo-redundant-storage)。以下示例显示，testVault 的 -BackupStorageRedundancy 选项设置为 GeoRedundant。

        $vault1 = Get-AzureRmRecoveryServicesVault –Name "testVault"
        Set-AzureRmRecoveryServicesBackupProperties  -Vault $vault1 -BackupStorageRedundancy GeoRedundant

## 步骤 2 - 选择备份目标、设置策略并定义要保护的项
现在，你已经创建了恢复服务保管库，因此可以使用它来保护虚拟机。但是，在应用保护之前，必须设置保管库上下文，并且需验证保护策略。保管库上下文定义了保管库中受保护的数据类型。保护策略是指对备份作业的运行时间以及每个备份快照的保留时长进行计划。

在 VM 上启用保护之前，必须设置保管库上下文。该上下文将应用到所有后续 cmdlet。

	PS C:\> Get-AzureRmRecoveryServicesVault -Name testvault | Set-AzureRmRecoveryServicesVaultContext

### 创建保护策略

当你创建新保管库时，它附带了一个默认策略。此策略会在每天的指定时间触发备份作业。根据默认策略，备份快照将保留 30 天。可以使用默认策略快速保护你的 VM，以后再使用不同的详细信息编辑该策略。

若要查看保管库中的可用策略列表，请使用 **[Get-AzureRmRecoveryServicesBackupProtectionPolicy](https://msdn.microsoft.com/zh-cn/library/mt723300.aspx)**：

	PS C:\> Get-AzureRmRecoveryServicesBackupProtectionPolicy -WorkloadType AzureVM
	Name                 WorkloadType       BackupManagementType BackupTime                DaysOfWeek
	----                 ------------       -------------------- ----------                ----------
	DefaultPolicy        AzureVM            AzureVM              4/14/2016 5:00:00 PM

> [AZURE.NOTE] PowerShell 中 BackupTime 字段的时区是 UTC。

一个备份保护策略至少与一个保留策略相关联。保留策略定义在 Azure 备份中保留恢复点的时限。使用 **Get-AzureRmRecoveryServicesBackupRetentionPolicyObject** 可以查看默认保留策略。同理，可以使用 **Get-AzureRmRecoveryServicesBackupSchedulePolicyObject** 获取默认计划策略。计划和保留策略对象将用作 **New-AzureRmRecoveryServicesBackupProtectionPolicy** cmdlet 的输入。

备份保护策略定义对某个项目进行备份的时间和频率。New-AzureRmRecoveryServicesBackupProtectionPolicy cmdlet 创建用于保存备份策略信息的 PowerShell 对象。该备份策略用作 Enable-AzureRmRecoveryServicesBackupProtection cmdlet 的输入。

	PS C:\> $schPol = Get-AzureRmRecoveryServicesBackupSchedulePolicyObject -WorkloadType "AzureVM"
	PS C:\>  $retPol = Get-AzureRmRecoveryServicesBackupRetentionPolicyObject -WorkloadType "AzureVM"
	PS C:\>  New-AzureRmRecoveryServicesBackupProtectionPolicy -Name "NewPolicy" -WorkloadType AzureVM -RetentionPolicy $retPol -SchedulePolicy $schPol
	Name                 WorkloadType       BackupManagementType BackupTime                DaysOfWeek
	----                 ------------       -------------------- ----------                ----------
	NewPolicy           AzureVM            AzureVM              4/24/2016 1:30:00 AM

### 启用保护

启用保护涉及两个对象 - 项和策略。需要提供这两个对象才能为保管库启用保护。将策略与保管库关联之后，将在策略计划中定义的时间触发备份工作流。

在非加密型 ARM VM 上启用保护

	PS C:\> $pol=Get-AzureRmRecoveryServicesBackupProtectionPolicy -Name "NewPolicy"
	PS C:\> Enable-AzureRmRecoveryServicesBackupProtection -Policy $pol -Name "V2VM" -ResourceGroupName "RGName1"

若要在加密型 VM 上启用保护[使用 BEK 和 KEK 加密]，需提供相应权限，允许 Azure 备份服务读取密钥保管库中的密钥和机密。

	PS C:\> Set-AzureRmKeyVaultAccessPolicy -VaultName 'KeyVaultName' -ResourceGroupName 'RGNameOfKeyVault' -PermissionsToKeys backup,get,list -PermissionsToSecrets get,list -ServicePrincipalName 262044b1-e2ce-469f-a196-69ab7ada62d3
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
	PS C:\> $pol= Get-AzureRmRecoveryServicesBackupProtectionPolicy -Name NewPolicy
	PS C:\> Set-AzureRmRecoveryServicesBackupProtectionPolicy -Policy $pol  -RetentionPolicy  $RetPol

## 在虚拟机中安装 VM 代理

此信息是根据需要提供的。Azure VM 代理必须安装在 Azure 虚拟机上，备份扩展才能运行。但是，如果 VM 创建自 Azure 资源库，则 VM 代理已存在于虚拟机上。从本地数据中心迁移的 VM 上未安装 VM 代理。在这种情况下，需要安装 VM 代理。如果在备份 Azure VM 时遇到问题，请先检查是否已在虚拟机上正确安装 Azure VM 代理（请参阅下表）。如果你要创建自定义 VM，请先[确保已选中“安装 VM 代理”复选框](/documentation/articles/virtual-machines-windows-classic-agents-and-extensions/)，然后再预配虚拟机。

了解 [VM 代理](/documentation/articles/virtual-machines-windows-extensions-features/)以及[如何安装它](/documentation/articles/virtual-machines-windows-classic-manage-extensions/)。

下表提供了适用于 Windows 和 Linux VM 的 VM 代理的其他信息。

| **操作** | **Windows** | **Linux** |
| --- | --- | --- |
| 安装 VM 代理 | <li>下载并安装[代理 MSI](http://download.microsoft.com/download/3/4/3/3437907D-745F-46EF-8116-7FC025BBEBDB/WindowsAzureVmAgent.2.6.1198.718.rd_art_stable.150415-1739.fre.msi)。你需要有管理员权限才能完成安装。<li>[更新 VM 属性](http://blogs.msdn.com/b/mast/archive/2014/04/08/install-the-vm-agent-on-an-existing-azure-vm.aspx)，指明已安装代理。 | <li>从 GitHub 安装最新的 [Linux 代理](https://github.com/Azure/WALinuxAgent)。你需要有管理员权限才能完成安装。<li>[更新 VM 属性](http://blogs.msdn.com/b/mast/archive/2014/04/08/install-the-vm-agent-on-an-existing-azure-vm.aspx)，指明已安装代理。 |
| 更新 VM 代理 | 更新 VM 代理与重新安装 [VM 代理二进制文件](http://download.microsoft.com/download/3/4/3/3437907D-745F-46EF-8116-7FC025BBEBDB/WindowsAzureVmAgent.2.6.1198.718.rd_art_stable.150415-1739.fre.msi)一样简单。<br>确保在更新 VM 代理时，没有任何正在运行的备份操作。 | 按照[更新 Linux VM 代理](/documentation/articles/virtual-machines-linux-update-agent/)上的说明进行操作。<br>确保在更新 VM 代理时，没有任何正在运行的备份操作。 |
| 验证 VM 代理安装 | <li>导航到 Azure VM 中的 *C:\\WindowsAzure\\Packages* 文件夹。<li>你应会发现 WaAppAgent.exe 文件已存在。<li> 右键单击该文件，转到“属性”，然后选择“详细信息”选项卡。“产品版本”字段应为 2.6.1198.718 或更高。 | 不适用 |


### 备份扩展

在虚拟机上安装 VM 代理后，Azure 备份服务会将备份扩展安装到 VM 代理上。Azure 备份服务会无缝地升级和修补备份扩展，不需用户进行额外的干预。

无论 VM 是否在运行，备份服务都安装备份扩展。VM 运行时，很有可能会获得应用程序一致的恢复点。但是，即使 VM 已关闭并且无法安装扩展，Azure 备份服务也会继续备份 VM。这被称为脱机 VM。在这种情况下，恢复点将是 *崩溃一致性* 恢复点。

## 故障排除信息
如果你在完成本文中的某些任务时遇到问题，请参阅[故障排除指南](/documentation/articles/backup-azure-vms-troubleshoot/)。


## 有疑问？
如果你有疑问，或者希望包含某种功能，请[给我们反馈](https://feedback.azure.com/forums/258995-azure-backup)。

<!---HONumber=Mooncake_0829_2016-->