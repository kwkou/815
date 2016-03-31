<properties
	pageTitle="通过 PowerShell 为 Azure VM 部署和管理备份 | Azure"
	description="了解如何使用 PowerShell 部署和管理 Azure 备份"
	services="backup"
	documentationCenter=""
	authors="aashishr"
	manager="shreeshd"
	editor=""/>

<tags
	ms.service="backup"
	ms.date="01/06/2016"
	wacn.date="01/26/2016" />


# 通过 PowerShell 为 Azure VM 部署和管理备份
本文演示如何使用 Azure PowerShell 来备份和恢复 Azure IaaS VM。

## 概念
Azure 备份文档中的 [Azure IaaS VM 备份简介](/documentation/articles/backup-azure-vms-introduction)。

> [AZURE.WARNING]在开始之前，请确保你已掌握与[系统必备组件](/documentation/articles/backup-azure-vms-prepare)相关的基础知识（这些必备组件是使用 Azure 备份所必需的），并了解当前 VM 备份解决方案的[限制](/documentation/articles/backup-azure-vms-prepare#limitations)。

为了提高 PowerShell 使用效率，必须了解对象的层次结构以及从何处开始。

![对象层次结构](./media/backup-azure-vms-automation/object-hierarchy.png)

两大流程包括启用对 VM 的保护，以及从恢复点还原数据。本文的重点是帮助你熟练使用这两项方案所需的 PowerShell cmdlet。


## 设置和注册
开始时，请执行以下操作：

1. [下载最新 PowerShell](https://github.com/Azure/azure-powershell/releases)（要求的最低版本：1.0.0）

2. 键入以下命令查找可用的 Azure 备份 PowerShell cmdlet：

```
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
```

使用 PowerShell 可以自动化以下设置和注册任务：

- 创建备份保管库
- 将 VM 注册到 Azure 备份服务

### 创建备份保管库

> [AZURE.WARNING]对于第一次使用 Azure 备份的客户，你需要注册用于订阅的 Azure 备份提供程序。可通过运行以下命令来执行此操作：Register-AzureProvider -ProviderNamespace "Microsoft.Backup"

可以使用 **New-AzureRMBackupVault** cmdlet 创建新的备份保管库。备份保管库是一种 ARM 资源，因此需要将它放置在资源组中。在权限提升的 Azure PowerShell 控制台中运行以下命令：

```
PS C:\> New-AzureRMResourceGroup –Name “test-rg” –Region “China North”
PS C:\> $backupvault = New-AzureRMBackupVault –ResourceGroupName “test-rg” –Name “test-vault” –Region “China North” –Storage GeoRedundant
```

可以使用 **Get-AzureRMBackupVault** cmdlet 获取给定订阅中所有备份保管库的列表。

> [AZURE.NOTE]可以方便地将备份保管库对象保存到一个变量中。许多 Azure 备份 cmdlet 需要输入保管库对象。


### 注册 VM
若要使用 Azure 备份配置备份，第一步是将你的计算机或 VM 注册到 Azure 备份保管库。**Register-AzureRMBackupContainer** cmdlet 采用 Azure IaaS 虚拟机的输入信息，并将其注册到指定保管库。注册操作将 Azure 虚拟机与备份保管库关联在一起，并跟踪备份生命周期中 VM 的活动。

将 VM 注册到 Azure 备份服务会创建顶级容器对象。一个容器通常保护多个可以备份的项，但在使用 VM 的情况下，容器将只有一个备份项。

```
PS C:\> $registerjob = Register-AzureRMBackupContainer -Vault $backupvault -Name "testvm" -ServiceName "testvm"
```

## 备份 Azure VM

### 创建保护策略
不必创建新的保护策略即可开始 VM 的备份。使用保管库附带的“默认策略”，你可以快速启用保护功能，稍后再使用适当的详细信息对该策略进行编辑。你可以使用 **Get-AzureRMBackupProtectionPolicy** cmdlet 获取保管库中提供的策略的列表：

```
	PS C:\> Get-AzureRMBackupProtectionPolicy -Vault $backupvault	
	Name                      Type               ScheduleType       BackupTime
	----                      ----               ------------       ----------
	DefaultPolicy             AzureVM            Daily              26-Aug-15 12:30:00 AM
```

> [AZURE.NOTE]PowerShell 中 BackupTime 字段的时区是 UTC。但是，在 Azure 门户中显示备份时间时，时区将会调整为你的本地系统并附带 UTC 时差。

一个备份策略至少与一个保留策略相关联。保留策略定义在 Azure 备份中保留恢复点的时限。**New-AzureRMBackupRetentionPolicy** cmdlet 创建的 PowerShell 对象用于存储保留策略信息。这些保留策略对象可以用作 *New-AzureRMBackupProtectionPolicy* cmdlet 的输入，也可以直接用于 *Enable-AzureRMBackupProtection* cmdlet。

备份策略定义对某个项目进行备份的时间和频率。**New-AzureRMBackupProtectionPolicy** cmdlet 创建的 PowerShell 对象用于存储备份策略信息。该备份策略用作 *Enable-AzureRMBackupProtection* cmdlet 的输入。

```
	PS C:\> $Daily = New-AzureRMBackupRetentionPolicyObject -DailyRetention -Retention 30
	PS C:\> $newpolicy = New-AzureRMBackupProtectionPolicy -Name DailyBackup01 -Type AzureVM -Daily -BackupTime ([datetime]"3:30 PM") -RetentionPolicy ($Daily) -Vault $backupvault
	Name                      Type               ScheduleType       BackupTime
	----                      ----               ------------       ----------
	DailyBackup01             AzureVM            Daily              01-Sep-15 3:30:00 PM
```

### 启用保护
启用保护涉及两个对象 - 项目和策略，二者必须属于同一保管库。将策略与项目关联以后，即可按定义的计划执行备份工作流。

```
PS C:\> Get-AzureRMBackupContainer -Type AzureVM -Status Registered -Vault $backupvault | Get-AzureRMBackupItem | Enable-AzureRMBackupProtection -Policy $newpolicy
```

### 初始备份
备份计划需要考虑的是如何为项目执行完整的初始复制，以及如何为后续备份执行增量复制。不过，如果你想要强制初始备份在某个时间发生或者甚至是立刻发生，则可使用 **Backup-AzureRMBackupItem** cmdlet：

```
	PS C:\> $container = Get-AzureRMBackupContainer -Vault $backupvault -type AzureVM -name "testvm"
	PS C:\> $backupjob = Get-AzureRMBackupItem -Container $container | Backup-AzureRMBackupItem
	PS C:\> $backupjob
	WorkloadName    Operation       Status          StartTime              EndTime
	------------    ---------       ------          ---------              -------
	testvm          Backup          InProgress      01-Sep-15 12:24:01 PM  01-Jan-01 12:00:00 AM
```

> [AZURE.NOTE]PowerShell 中显示的 StartTime 和 EndTime 字段的时区是 UTC。但是，在 Azure 门户中显示类似信息时，时区将会调整为你的本地系统时钟。

### 监视备份作业
在 Azure 备份中，大多数长时间运行的操作都是作为作业来建模的。这样可以轻松地跟踪相关进度，不必让 Azure 门户始终打开。

若要获取正在进行的作业的最新状态，请使用 **Get-AzureRMBackupJob** cmdlet。

```
	PS C:\> $joblist = Get-AzureRMBackupJob -Vault $backupvault -Status InProgress
	PS C:\> $joblist[0]
	WorkloadName    Operation       Status          StartTime              EndTime
	------------    ---------       ------          ---------              -------
	testvm          Backup          InProgress      01-Sep-15 12:24:01 PM  01-Jan-01 12:00:00 AM
```

与其使用额外的不必要的代码来轮询这些作业的完成情况，不如使用更简单的方式：**Wait-AzureRMBackupJob** cmdlet。在脚本中使用时，该 cmdlet 会暂停操作的执行，直到作业完成或达到了指定的超时值。

```
PS C:\> Wait-AzureRMBackupJob -Job $joblist[0] -Timeout 43200
```


## <a name="restore-an-azure-vm"></a>还原 Azure VM

若要还原备份数据，你需要确定已备份项目以及保留了时间点数据的恢复点。此信息将提供给 Restore-AzureRMBackupItem cmdlet，以便启动还原过程，将数据从保管库还原到客户的帐户。

### 选择 VM

若要获取用于标识正确备份项目的 PowerShell 对象，你需要从保管库中的容器开始，按对象层次结构进行操作。若要选择代表 VM 的容器，可使用 **Get-AzureRMBackupContainer** cmdlet，然后通过管道将其传输给 **Get-AzureRMBackupItem** cmdlet。

```
PS C:\> $backupitem = Get-AzureRMBackupContainer -Vault $backupvault -Type AzureVM -name "testvm" | Get-AzureRMBackupItem
```

### 选择恢复点

你现在可以使用 **Get-AzureRMBackupRecoveryPoint** cmdlet 列出备份项目的所有恢复点，然后选择要还原的恢复点。通常情况下，用户会选取列表中在时间上最近的 *AppConsistent* 点。

```
	PS C:\> $rp =  Get-AzureRMBackupRecoveryPoint -Item $backupitem
	PS C:\> $rp
	RecoveryPointId    RecoveryPointType  RecoveryPointTime      ContainerName
	---------------    -----------------  -----------------      -------------
	15273496567119     AppConsistent      01-Sep-15 12:27:38 PM  iaasvmcontainer;testvm;testv...
```

变量 ```$rp``` 是选定备份项的恢复点数组，已按时间逆序排序 - 最新的恢复点位于索引 0 处。使用标准 PowerShell 数组索引选取恢复点。例如：```$rp[0]``` 将选择最新的恢复点。

### 还原磁盘

通过 Azure 门户执行还原操作与通过 Azure PowerShell 执行还原操作存在很大的不同。如果使用 PowerShell，还原操作将在从恢复点还原磁盘和配置信息时停止。它不会创建虚拟机。

> [AZURE.WARNING]Restore-AzureRMBackupItem 不创建 VM。它仅将磁盘还原到指定的存储帐户。这种行为不同于你在 Azure 门户中将会体验到的行为。

```
	PS C:\> $restorejob = Restore-AzureRMBackupItem -StorageAccountName "DestAccount" -RecoveryPoint $rp[0]
	PS C:\> $restorejob
	WorkloadName    Operation       Status          StartTime              EndTime
	------------    ---------       ------          ---------              -------
	testvm          Restore         InProgress      01-Sep-15 1:14:01 PM   01-Jan-01 12:00:00 AM
```

还原作业完成后，你可以使用 **Get-AzureRMBackupJobDetails** cmdlet 获取还原操作的详细信息。*ErrorDetails* 属性将提供重建 VM 所需的信息。

```
PS C:\> $restorejob = Get-AzureRMBackupJob -Job $restorejob
PS C:\> $details = Get-AzureRMBackupJobDetails -Job $restorejob
```

### 构建 VM

从还原的磁盘构建 VM 时，可以使用旧版 Azure ServiceManager PowerShell cmdlet、新的 Azure ResourceManager 模板甚至 Azure 门户。在快速示例中，我们将演示如何使用 Azure ServiceManager cmdlet 来达成此目标。

```
	 $properties  = $details.Properties
	 $storageAccountName = $properties["TargetStorageAccountName"]
	 $containerName = $properties["TargetContainerName"]
	 $blobName = $properties["TargetBlobName"]
	 $keys = Get-AzureStorageKey -StorageAccountName $storageAccountName
	 $storageAccountKey = $keys.Primary
	 $storageContext = New-AzureStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $storageAccountKey
	 $destination_path = "C:\Users\admin\Desktop\vmconfig.xml"
	 Get-AzureStorageBlobContent -Container $containerName -Blob $blobName -Destination $destination_path -Context $storageContext
	 $obj = [xml](Get-Content $destination_path)
	 $pvr = $obj.PersistentVMRole
	 $os = $pvr.OSVirtualHardDisk
	 $dds = $pvr.DataVirtualHardDisks
	 $osDisk = Add-AzureDisk -MediaLocation $os.MediaLink -OS $os.OS -DiskName "panbhaosdisk"
	 $vm = New-AzureVMConfig -Name $pvr.RoleName -InstanceSize $pvr.RoleSize -DiskName $osDisk.DiskName
	 if (!($dds -eq $null))
	 {
		 foreach($d in $dds.DataVirtualHardDisk)
	 	 {
			 $lun = 0;
			 if(!($d.Lun -eq $null))
			 {
		 		 $lun = $d.Lun
			 }
			 $name = "panbhadataDisk" + $lun
	     Add-AzureDisk -DiskName $name -MediaLocation $d.MediaLink
	     $vm | Add-AzureDataDisk -Import -DiskName $name -LUN $lun
		}
	}
	New-AzureVM -ServiceName "panbhasample" -Location "SouthEast Asia" -VM $vm
```

有关如何从还原的磁盘构建 VM 的详细信息，请阅读与下述 cmdlet 相关的内容：

- [Add-AzureDisk](https://msdn.microsoft.com/library/azure/dn495252.aspx)
- [New-AzureVMConfig](https://msdn.microsoft.com/library/azure/dn495159.aspx)
- [New-AzureVM](https://msdn.microsoft.com/library/azure/dn495254.aspx)

## 代码示例

### 1\.获取作业子任务的完成状态

若要跟踪单个子任务的完成状态，可以使用 **Get-AzureRMBackupJobDetails** cmdlet：

```
	PS C:\> $details = Get-AzureRMBackupJobDetails -JobId $backupjob.InstanceId -Vault $backupvault
	PS C:\> $details.SubTasks
	Name                                                        Status
	----                                                        ------
	Take Snapshot                                               Completed
	Transfer data to Backup vault                               InProgress
```

### 2\.创建备份作业的每日/每周报告

管理员通常想要知道过去 24 小时运行了哪些备份作业以及这些备份作业的状态。此外，管理员可以借助数据传输量来估算每月数据使用量。以下脚本将从 Azure 备份服务提取原始数据，并在 PowerShell 控制台上显示信息。

```
param(  [Parameter(Mandatory=$True,Position=1)]
	        [string]$backupvaultname,	
	        [Parameter(Mandatory=$False,Position=2)]
	        [int]$numberofdays = 7)
	#Initialize variables
	$DAILYBACKUPSTATS = @()
	$backupvault = Get-AzureRMBackupVault -Name $backupvaultname
	$enddate = ([datetime]::Today).AddDays(1)
	$startdate = ([datetime]::Today)
	for( $i = 1; $i -le $numberofdays; $i++ )
	{
	    # We query one day at a time because pulling 7 days of data might be too much
	    $dailyjoblist = Get-AzureRMBackupJob -Vault $backupvault -From $startdate -To $enddate -Type AzureVM -Operation Backup
	    Write-Progress -Activity "Getting job information for the last $numberofdays days" -Status "Day -$i" -PercentComplete ([int]([decimal]$i*100/$numberofdays))
	    foreach( $job in $dailyjoblist )
	    {
	        #Extract the information for the reports
	        $newstatsobj = New-Object System.Object
	        $newstatsobj | Add-Member -type NoteProperty -name Date -value $startdate
	        $newstatsobj | Add-Member -type NoteProperty -name VMName -value $job.WorkloadName
	        $newstatsobj | Add-Member -type NoteProperty -name Duration -value $job.Duration
	        $newstatsobj | Add-Member -type NoteProperty -name Status -value $job.Status	
	        $details = Get-AzureRMBackupJobDetails -Job $job
	        $newstatsobj | Add-Member -type NoteProperty -name BackupSize -value $details.Properties["Backup Size"]
	        $DAILYBACKUPSTATS += $newstatsobj
	    }
	    $enddate = $enddate.AddDays(-1)
	    $startdate = $startdate.AddDays(-1)
	}
	$DAILYBACKUPSTATS | Out-GridView
```

如果你想要为此报告输出添加图表功能，可通过 TechNet 博客[使用 PowerShell 绘制图表](http://blogs.technet.com/b/richard_macdonald/archive/2009/04/28/3231887.aspx)来了解相关信息

<!---HONumber=Mooncake_0104_2016-->