<properties
	pageTitle="通过 PowerShell 为 Azure VM 部署和管理备份 | Azure"
	description="了解如何使用 PowerShell 部署和管理 Azure 备份"
	services="backup"
	documentationCenter=""
	authors="markgalioto"
	manager="cfreeman"
	editor=""/>

<tags
	ms.service="backup"
	ms.workload="storage-backup-recovery"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="08/08/2016"
	wacn.date="01/19/2017" 
	ms.author="markgal;trinadhk;jimpark" />


# 通过 PowerShell 为 Azure VM 部署和管理备份

> [AZURE.SELECTOR]
- [资源管理器](/documentation/articles/backup-azure-vms-automation/)
- [经典](/documentation/articles/backup-azure-vms-classic-automation/)

本文演示如何使用 Azure PowerShell 来备份和恢复 Azure VM。Azure 有两个用于创建和处理资源的不同部署模型：Resouce Manager 部署模型和经典部署模型。本文介绍如何使用经典部署模型。Azure 建议大多数新部署使用资源管理器模型。

## 概念


本文专门介绍用于备份虚拟机的 PowerShell cmdlet。有关如何保护 Azure VM 的介绍，请参阅[在 Azure 中计划 VM 备份基础结构](/documentation/articles/backup-azure-vms-introduction/)。

> [AZURE.NOTE] 在开始之前，请阅读使用 Azure 备份所需的[系统必备组件](/documentation/articles/backup-azure-vms-prepare/)，以及当前 VM 备份解决方案的[限制](/documentation/articles/backup-azure-vms-prepare/#limitations/)。

为了提高 PowerShell 使用效率，请抽时间了解对象的层次结构以及从何处开始。

![对象层次结构](./media/backup-azure-vms-classic-automation/object-hierarchy.png)

两大流程包括启用对 VM 的保护，以及从恢复点还原数据。本文的重点是帮助你熟练使用这两项方案所需的 PowerShell cmdlet。


## 设置和注册
开始时，请执行以下操作：

1. [下载最新 PowerShell](https://github.com/Azure/azure-powershell/releases)（要求的最低版本：1.0.0）

2. 键入以下命令查找可用的 Azure 备份 PowerShell cmdlet：

		PS C:\> Get-Command *azurermbackup*
		
		CommandType     Name                                               Version    Source
		-----------     ----                                               -------    ------
		Cmdlet          Backup-AzureRmBackupItem                           1.0.1      AzureRM.Backup
		Cmdlet          Disable-AzureRmBackupProtection                    1.0.1      AzureRM.Backup
		Cmdlet          Enable-AzureRmBackupContainerReregistration        1.0.1      AzureRM.Backup
		Cmdlet          Enable-AzureRmBackupProtection                     1.0.1      AzureRM.Backup
		Cmdlet          Get-AzureRmBackupContainer                         1.0.1      AzureRM.Backup
		Cmdlet          Get-AzureRmBackupItem                              1.0.1      AzureRM.Backup
		Cmdlet          Get-AzureRmBackupJob                               1.0.1      AzureRM.Backup
		Cmdlet          Get-AzureRmBackupJobDetails                        1.0.1      AzureRM.Backup
		Cmdlet          Get-AzureRmBackupProtectionPolicy                  1.0.1      AzureRM.Backup
		Cmdlet          Get-AzureRmBackupRecoveryPoint                     1.0.1      AzureRM.Backup
		Cmdlet          Get-AzureRmBackupVault                             1.0.1      AzureRM.Backup
		Cmdlet          Get-AzureRmBackupVaultCredentials                  1.0.1      AzureRM.Backup
		Cmdlet          New-AzureRmBackupProtectionPolicy                  1.0.1      AzureRM.Backup
		Cmdlet          New-AzureRmBackupRetentionPolicyObject             1.0.1      AzureRM.Backup
		Cmdlet          New-AzureRmBackupVault                             1.0.1      AzureRM.Backup
		Cmdlet          Register-AzureRmBackupContainer                    1.0.1      AzureRM.Backup
		Cmdlet          Remove-AzureRmBackupProtectionPolicy               1.0.1      AzureRM.Backup
		Cmdlet          Remove-AzureRmBackupVault                          1.0.1      AzureRM.Backup
		Cmdlet          Restore-AzureRmBackupItem                          1.0.1      AzureRM.Backup
		Cmdlet          Set-AzureRmBackupProtectionPolicy                  1.0.1      AzureRM.Backup
		Cmdlet          Set-AzureRmBackupVault                             1.0.1      AzureRM.Backup
		Cmdlet          Stop-AzureRmBackupJob                              1.0.1      AzureRM.Backup
		Cmdlet          Unregister-AzureRmBackupContainer                  1.0.1      AzureRM.Backup
		Cmdlet          Wait-AzureRmBackupJob                              1.0.1      AzureRM.Backup

使用 PowerShell 可以自动化以下设置和注册任务：

- 创建备份保管库
- 将 VM 注册到 Azure 备份服务

### 创建备份保管库

> [AZURE.WARNING] 对于第一次使用 Azure 备份的客户，你需要注册用于订阅的 Azure 备份提供程序。可通过运行以下命令来执行此操作：Register-AzureRmResourceProvider -ProviderNamespace "Microsoft.Backup"

可以使用 **New-AzureRmBackupVault** cmdlet 创建新的备份保管库。备份保管库是一种 ARM 资源，因此需要将它放置在资源组中。在权限提升的 Azure PowerShell 控制台中运行以下命令：

	PS C:\> New-AzureRmResourceGroup -Name "test-rg" -Location "China North"
	PS C:\> $backupvault = New-AzureRmBackupVault -ResourceGroupName "test-rg" -Name "test-vault" -Region "China North" -Storage GeoRedundant

可以使用 **Get-AzureRmBackupVault** cmdlet 获取给定订阅中所有备份保管库的列表。

> [AZURE.NOTE] 可以方便地将备份保管库对象保存到一个变量中。许多 Azure 备份 cmdlet 需要输入保管库对象。


### 注册 VM
若要使用 Azure 备份配置备份，第一步是将你的计算机或 VM 注册到 Azure 备份保管库。**Register-AzureRmBackupContainer** cmdlet 采用 Azure IaaS 虚拟机的输入信息，并将其注册到指定保管库。注册操作将 Azure 虚拟机与备份保管库关联在一起，并跟踪备份生命周期中 VM 的活动。

将 VM 注册到 Azure 备份服务会创建顶级容器对象。一个容器通常保护多个可以备份的项，但在使用 VM 的情况下，容器将只有一个备份项。

	PS C:\> $registerjob = Register-AzureRmBackupContainer -Vault $backupvault -Name "testvm" -ServiceName "testvm"

## 备份 Azure VM <a name="restore-an-azure-vm"></a>

### 创建保护策略
不必创建新的保护策略即可开始 VM 的备份。使用保管库附带的“默认策略”，你可以快速启用保护功能，稍后再使用适当的详细信息对该策略进行编辑。你可以使用 **Get-AzureRmBackupProtectionPolicy cmdlet** 获取保管库中提供的策略的列表：

	PS C:\> Get-AzureRmBackupProtectionPolicy -Vault $backupvault
	
	Name                      Type               ScheduleType       BackupTime
	----                      ----               ------------       ----------
	DefaultPolicy             AzureVM            Daily              26-Aug-15 12:30:00 AM

> [AZURE.NOTE] PowerShell 中 BackupTime 字段的时区是 UTC。

一个备份策略至少与一个保留策略相关联。保留策略定义在 Azure 备份中保留恢复点的时限。**New-AzureRmBackupRetentionPolicy** cmdlet 创建的 PowerShell 对象用于存储保留策略信息。这些保留策略对象可以用作 *New-AzureRmBackupProtectionPolicy* cmdlet 的输入，也可以直接用于 *Enable-AzureRmBackupProtection* cmdlet。

备份策略定义对某个项目进行备份的时间和频率。**New-AzureRmBackupProtectionPolicy** cmdlet 创建的 PowerShell 对象用于存储备份策略信息。该备份策略用作 *Enable-AzureRmBackupProtection* cmdlet 的输入。

	PS C:\> $Daily = New-AzureRmBackupRetentionPolicyObject -DailyRetention -Retention 30
	PS C:\> $newpolicy = New-AzureRmBackupProtectionPolicy -Name DailyBackup01 -Type AzureVM -Daily -BackupTime ([datetime]"3:30 PM") -RetentionPolicy $Daily -Vault $backupvault
	
	Name                      Type               ScheduleType       BackupTime
	----                      ----               ------------       ----------
	DailyBackup01             AzureVM            Daily              01-Sep-15 3:30:00 PM

### 启用保护
启用保护涉及两个对象 - 项目和策略，二者必须属于同一保管库。将策略与项目关联以后，即可按定义的计划执行备份工作流。

	PS C:\> Get-AzureRmBackupContainer -Type AzureVM -Status Registered -Vault $backupvault | Get-AzureRmBackupItem | Enable-AzureRmBackupProtection -Policy $newpolicy

### 初始备份
备份计划需要考虑的是如何为项目执行完整的初始复制，以及如何为后续备份执行增量复制。不过，如果你想要强制初始备份在某个时间发生或者甚至是立刻发生，则可使用 **Backup-AzureRmBackupItem** cmdlet：

	PS C:\> $container = Get-AzureRmBackupContainer -Vault $backupvault -Type AzureVM -Name "testvm"
	PS C:\> $backupjob = Get-AzureRmBackupItem -Container $container | Backup-AzureRmBackupItem
	PS C:\> $backupjob
	
	WorkloadName    Operation       Status          StartTime              EndTime
	------------    ---------       ------          ---------              -------
	testvm          Backup          InProgress      01-Sep-15 12:24:01 PM  01-Jan-01 12:00:00 AM

> [AZURE.NOTE] PowerShell 中显示的 StartTime 和 EndTime 字段的时区是 UTC。

### 监视备份作业
在 Azure 备份中，大多数长时间运行的操作都是作为作业来建模的。

若要获取正在进行的作业的最新状态，请使用 **Get-AzureRmBackupJob** cmdlet。

	PS C:\> $joblist = Get-AzureRmBackupJob -Vault $backupvault -Status InProgress
	PS C:\> $joblist[0]
	
	WorkloadName    Operation       Status          StartTime              EndTime
	------------    ---------       ------          ---------              -------
	testvm          Backup          InProgress      01-Sep-15 12:24:01 PM  01-Jan-01 12:00:00 AM

与其使用额外的不必要的代码来轮询这些作业的完成情况，不如使用更简单的方式：**Wait-AzureRmBackupJob** cmdlet。在脚本中使用时，该 cmdlet 会暂停操作的执行，直到作业完成或达到了指定的超时值。

	PS C:\> Wait-AzureRmBackupJob -Job $joblist[0] -Timeout 43200


## 还原 Azure VM

若要还原备份数据，你需要确定已备份项目以及保留了时间点数据的恢复点。此信息将提供给 Restore-AzureRmBackupItem cmdlet，以便启动还原过程，将数据从保管库还原到客户的帐户。

### 选择 VM

若要获取用于标识正确备份项目的 PowerShell 对象，你需要从保管库中的容器开始，按对象层次结构进行操作。若要选择代表 VM 的容器，可使用 **Get-AzureRmBackupContainer** cmdlet，然后通过管道将其传输给 **Get-AzureRmBackupItem** cmdlet。

	PS C:\> $backupitem = Get-AzureRmBackupContainer -Vault $backupvault -Type AzureVM -name "testvm" | Get-AzureRmBackupItem

### 选择恢复点

你现在可以使用 **Get-AzureRmBackupRecoveryPoint** cmdlet 列出备份项目的所有恢复点，然后选择要还原的恢复点。通常情况下，用户会选取列表中在时间上最近的 *AppConsistent* 点。

	PS C:\> $rp =  Get-AzureRmBackupRecoveryPoint -Item $backupitem
	PS C:\> $rp
	
	RecoveryPointId    RecoveryPointType  RecoveryPointTime      ContainerName
	---------------    -----------------  -----------------      -------------
	15273496567119     AppConsistent      01-Sep-15 12:27:38 PM  iaasvmcontainer;testvm;testv...

变量 ```$rp``` 是选定备份项的恢复点数组，已按时间逆序排序 - 最新的恢复点位于索引 0 处。使用标准 PowerShell 数组索引选取恢复点。例如：```$rp[0]``` 将选择最新的恢复点。

### 还原磁盘

如果使用 PowerShell，还原操作将在从恢复点还原磁盘和配置信息时停止。它不会创建虚拟机。

> [AZURE.WARNING] Restore-AzureRmBackupItem 不创建 VM。它仅将磁盘还原到指定的存储帐户。

	PS C:\> $restorejob = Restore-AzureRmBackupItem -StorageAccountName "DestAccount" -RecoveryPoint $rp[0]
	PS C:\> $restorejob
	
	WorkloadName    Operation       Status          StartTime              EndTime
	------------    ---------       ------          ---------              -------
	testvm          Restore         InProgress      01-Sep-15 1:14:01 PM   01-Jan-01 12:00:00 AM

还原作业完成后，你可以使用 **Get-AzureRmBackupJobDetails** cmdlet 获取还原操作的详细信息。*ErrorDetails* 属性将提供重建 VM 所需的信息。

	PS C:\> $restorejob = Get-AzureRmBackupJob -Job $restorejob
	PS C:\> $details = Get-AzureRmBackupJobDetails -Job $restorejob

### 构建 VM

从还原的磁盘构建 VM 时，可以使用旧版 Azure 服务管理 PowerShell cmdlet、新的 Azure Resource Manager 模板。在快速示例中，我们将演示如何使用 Azure 服务管理 cmdlet 来实现此目的。

	 $properties  = $details.Properties
	
	 $storageAccountName = $properties["Target Storage Account Name"]
	 $containerName = $properties["Config Blob Container Name"]
	 $blobName = $properties["Config Blob Name"]
	
	 $keys = Get-AzureStorageKey -StorageAccountName $storageAccountName
	 $storageAccountKey = $keys.Primary
	 $storageContext = New-AzureStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $storageAccountKey
	
	
	 $destination_path = "C:\Users\admin\Desktop\vmconfig.xml"
	 Get-AzureStorageBlobContent -Container $containerName -Blob $blobName -Destination $destination_path -Context $storageContext
	
	
	$obj = [xml](((Get-Content -Path $destination_path -Encoding UniCode)).TrimEnd([char]0x00))
	 $pvr = $obj.PersistentVMRole
	 $os = $pvr.OSVirtualHardDisk
	 $dds = $pvr.DataVirtualHardDisks
	 $osDisk = Add-AzureDisk -MediaLocation $os.MediaLink -OS $os.OS -DiskName "panbhaosdisk"
	 $vm = New-AzureVMConfig -Name $pvr.RoleName -InstanceSize $pvr.RoleSize -DiskName $osDisk.DiskName
	
	 if (!($dds -eq $null))
	 {
		 foreach($d in $dds.DataVirtualHardDisk)
	 	 {
			 $lun = 0
			 if(!($d.Lun -eq $null))
			 {
		 		 $lun = $d.Lun
			 }
			 $name = "panbhadataDisk" + $lun
	     Add-AzureDisk -DiskName $name -MediaLocation $d.MediaLink
	     $vm | Add-AzureDataDisk -Import -DiskName $name -LUN $lun
		}
	}
	
	New-AzureVM -ServiceName "panbhasample" -Location "China East" -VM $vm

有关如何从还原的磁盘构建 VM 的详细信息，请阅读与下述 cmdlet 相关的内容：

- [Add-AzureDisk](https://msdn.microsoft.com/zh-cn/library/azure/dn495252.aspx)
- [New-AzureVMConfig](https://msdn.microsoft.com/zh-cn/library/azure/dn495159.aspx)
- [New-AzureVM](https://msdn.microsoft.com/zh-cn/library/azure/dn495254.aspx)

## 代码示例

### 1\.获取作业子任务的完成状态

若要跟踪单个子任务的完成状态，可以使用 **Get-AzureRmBackupJobDetails** cmdlet：

	PS C:\> $details = Get-AzureRmBackupJobDetails -JobId $backupjob.InstanceId -Vault $backupvault
	PS C:\> $details.SubTasks
	
	Name                                                        Status
	----                                                        ------
	Take Snapshot                                               Completed
	Transfer data to Backup vault                               InProgress

### 2\.创建备份作业的每日/每周报告

管理员通常想要知道过去 24 小时运行了哪些备份作业以及这些备份作业的状态。此外，管理员可以借助数据传输量来估算每月数据使用量。以下脚本将从 Azure 备份服务提取原始数据，并在 PowerShell 控制台上显示信息。

	param(  [Parameter(Mandatory=$True,Position=1)]
	        [string]$backupvaultname,
	
	        [Parameter(Mandatory=$False,Position=2)]
	        [int]$numberofdays = 7)
	
	
	#Initialize variables
	$DAILYBACKUPSTATS = @()
	$backupvault = Get-AzureRmBackupVault -Name $backupvaultname
	$enddate = ([datetime]::Today).AddDays(1)
	$startdate = ([datetime]::Today)
	
	for( $i = 1; $i -le $numberofdays; $i++ )
	{
	    # We query one day at a time because pulling 7 days of data might be too much
	    $dailyjoblist = Get-AzureRmBackupJob -Vault $backupvault -From $startdate -To $enddate -Type AzureVM -Operation Backup
	    Write-Progress -Activity "Getting job information for the last $numberofdays days" -Status "Day -$i" -PercentComplete ([int]([decimal]$i*100/$numberofdays))
	
	    foreach( $job in $dailyjoblist )
	    {
	        #Extract the information for the reports
	        $newstatsobj = New-Object System.Object
	        $newstatsobj | Add-Member -Type NoteProperty -Name Date -Value $startdate
	        $newstatsobj | Add-Member -Type NoteProperty -Name VMName -Value $job.WorkloadName
	        $newstatsobj | Add-Member -Type NoteProperty -Name Duration -Value $job.Duration
	        $newstatsobj | Add-Member -Type NoteProperty -Name Status -Value $job.Status
	
	        $details = Get-AzureRmBackupJobDetails -Job $job
	        $newstatsobj | Add-Member -Type NoteProperty -Name BackupSize -Value $details.Properties["Backup Size"]
	        $DAILYBACKUPSTATS += $newstatsobj
	    }
	
	    $enddate = $enddate.AddDays(-1)
	    $startdate = $startdate.AddDays(-1)
	}
	
	$DAILYBACKUPSTATS | Out-GridView

如果你想要为此报告输出添加图表功能，可通过 TechNet 博客[使用 PowerShell 绘制图表](http://blogs.technet.com/b/richard_macdonald/archive/2009/04/28/3231887.aspx)来了解相关信息

## 后续步骤

如果你更愿意使用 PowerShell 来处理 Azure 资源，则请查看有关如何保护 Windows Server 的 PowerShell 文章：[为 Windows Server 部署和管理备份](/documentation/articles/backup-client-automation-classic/)。此外还有一篇有关如何管理 DPM 备份的 PowerShell 文章：[为 DPM 部署和管理备份](/documentation/articles/backup-dpm-automation-classic/)。这两篇文章都为资源管理器部署和经典部署提供了一个版本。

<!---HONumber=Mooncake_0829_2016-->
