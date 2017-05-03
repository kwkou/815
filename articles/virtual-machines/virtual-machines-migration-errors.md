<properties
    pageTitle="从经典迁移到 Azure Resource Manager 期间的常见错误 | Azure"
    description="本文编录了将 IaaS 资源从 Azure 服务管理迁移到 Azure Resource Manager 堆栈期间的最常见错误和缓解措施。"
    services="virtual-machines-windows"
    documentationcenter=""
    author="singhkays"
    manager="timlt"
    editor=""
    tags="azure-resource-manager" />
<tags
    ms.assetid="5bc03a1b-eb1c-438c-83d9-f0e9d61f1b6a"
    ms.service="virtual-machines-windows"
    ms.workload="infrastructure-services"
    ms.tgt_pltfrm="vm-windows"
    ms.devlang="na"
    ms.topic="article"
    ms.date="10/13/2016"
    wacn.date="01/20/2017"
    ms.author="singhkay" />  


# 从经典迁移到 Azure Resource Manager 期间的常见错误
本文编录了将 IaaS 资源从 Azure 经典部署模型迁移到 Azure Resource Manager 堆栈期间的最常见错误和缓解措施。

## 错误列表
| 错误字符串 | 缓解措施 |
| --- | --- |
| 内部服务器错误 |在某些情况下，这是重试时消失的暂时性错误。如果此错误仍然存在， 请[联系 Azure 支持人员](/support/contact/)，因为需要调查平台日志。<br><br> **注意：**由支持团队跟踪事件后，请不要尝试任何自我缓解措施，因为这可能会对你的环境产生意想不到的后果。 |
| 托管服务 {hosted-service-name} 中的部署 {deployment-name} 不支持迁移，因为它是 PaaS 部署（Web/辅助角色）。 |当部署包含 Web/辅助角色时，将发生这种情况。由于仅虚拟机支持迁移，请从部署中删除 Web/辅助角色，然后重试迁移。 |
| 模板 {template-name} 部署失败。CorrelationId={guid} |在迁移服务的后端，我们将使用 Azure Resource Manager 模板在 Azure Resource Manager 堆栈中创建资源。由于模板是幂等的，通常你可以安全地重试迁移操作，以忽略此错误。如果此错误仍然存在，请[联系 Azure 支持人员](/support/contact/)并向他们提供 CorrelationId。<br><br> **注意：**由支持团队跟踪事件后，请不要尝试任何自我缓解措施，因为这可能会对你的环境产生意想不到的后果。 |
| 虚拟网络 {virtual-network-name} 不存在。 |如果在新的 Azure 门户预览中创建虚拟网络，可能会发生这种情况。实际的虚拟网络名称遵循模式“Group * <VNET 名称>” |
| 托管服务 {hosted-service-name} 中的 VM {vm-name} 包含 Azure Resource Manager 不支持的扩展 {extension-name}。建议先从 VM 中卸载该扩展，然后再继续进行迁移。 |Azure Resource Manager 不支持 XML 扩展（例如 BGInfo 1.*）。因此，无法迁移这些扩展。如果将这些扩展保留安装在虚拟机上，则在完成迁移之前会自动将其卸载。 |
| 托管服务 {hosted-service-name} 中的 VM {vm-name} 包含迁移当前不支持的扩展 VMSnapshot/VMSnapshotLinux。请从 VM 中卸载该扩展，在迁移完成后再使用 Azure Resource Manager 重新添加它 |这是为 Azure 备份配置虚拟机的方案。由于这当前是不受支持的方案，请按照 [Azure 备份服务 - 常见问题](/documentation/articles/backup-azure-backup-faq/#i-backed-up-my-classic-vms-in-a-backup-vault-can-i-migrate-my-vms-from-classic-mode-to-resource-manager-mode-and-protect-them-in-a-recovery-services-vault) 中的解决方法操作 |
| 托管服务 {hosted-service-name} 中的 VM {vm-name} 包含 VM 未报告其状态的扩展 {extension-name}。因此，此 VM 无法迁移。确保 VM 报告该扩展状态或从 VM 中卸载该扩展，然后重试迁移。<br><br> 托管服务 {hosted-service-name} 中的 VM {vm-name} 包含报告处理程序状态：{handler-status} 的扩展 {extension-name}。因此，此 VM 无法迁移。确保所报告的扩展处理程序状态为 {handler-status} 或从 VM 中卸载该扩展，然后重试迁移。<br><br> 托管服务 {hosted-service-name} 中 VM {vm-name} 的 VM 代理报告的总体代理状态为“未就绪”。因此，该 VM 可能不会迁移（如果它包含可迁移的扩展）。请确保 VM 代理报告的总体代理状态为“就绪”。请参阅 [常见问题](/documentation/articles/virtual-machines-windows-migration-classic-resource-manager/#frequently-asked-questions)。 |Azure 来宾代理和 VM 扩展需要对 VM 存储帐户进行出站 Internet 访问以填充其状态。状态失败的常见原因包括<li>网络安全组阻止对 Internet 进行出站访问<li>（如果 VNET 有本地 DNS 服务器且 DNS 连接已丢失）<br><br>如果继续看到不支持的状态，则可以卸载该扩展以跳过此检查并继续进行迁移。 |
| 托管服务 {hosted-service-name} 中的部署 {deployment-name} 不支持迁移，因为它具有多个可用性集。 |目前，仅具有 1 个或更少可用性集的托管服务可以迁移。若要解决此问题，请将其他可用性集和这些可用性集中的虚拟机移到其他托管服务。 |
| 托管服务 {hosted-service-name} 中的部署 {deployment-name} 不支持迁移，因为它包含的 VM 不属于可用性集，尽管该托管服务包含一个可用性集。 |这种情况的解决方法是将所有虚拟机都移到单个可用性集中，或者从托管服务的可用性集中删除所有虚拟机。 |
| 存储帐户/托管服务/虚拟网络 {virtual-network-name} 正处于迁移过程中，因此无法更改 |当资源已完成“准备”迁移操作并触发了对资源进行更改的操作时，会发生此错误。由于“准备”操作完成后对管理平面进行了锁定，将阻止对资源进行任何更改。若要解锁管理平面，可以运行“提交”迁移操作以完成迁移，或者运行“中止”迁移操作以回退“准备”操作。 |
| 托管服务 {hosted-service-name} 不允许迁移，因为它包含的 VM {vm-name} 处于状态: RoleStateUnknown。仅当 VM 处于以下状态（“正在运行”、“已停止”、“已停止已解除分配”）之一时，才允许迁移。 |该 VM 可能正在进行状态转换，通常在对托管服务进行更新操作（如重新启动，扩展安装等）期间会发生这种情况。建议先对托管服务完成更新操作，然后再尝试迁移。 |
| HostedService {hosted-service-name} 中的部署 {deployment-name} 包含 VM {vm-name}，该 VM 的数据磁盘 {data-disk-name} 的物理 Blob 大小 {size-of-the-vhd-blob-backing-the-data-disk} 字节与 VM 数据磁盘逻辑大小 {size-of-the-data-disk-specified-in-the-vm-api} 字节不匹配。迁移将会继续，但不会指定 Azure Resource Manager VM 的数据磁盘大小。若要在继续迁移之前更正数据磁盘大小，请访问 [VM 中数据磁盘的物理 Blob 字节大小与 VM 数据磁盘的逻辑字节大小不匹配](#vm-with-data-disk-whose-physical-blob-size-bytes-does-not-match-the-vm-data-disk-logical-size-bytes)。 | 如果调整了 VHD Blob 大小但未更新 VM API 模型的大小，将发生此错误。[下面](#vm-with-data-disk-whose-physical-blob-size-bytes-does-not-match-the-vm-data-disk-logical-size-bytes)介绍了详细缓解步骤。|

## 详细缓解措施

### <a name="vm-with-data-disk-whose-physical-blob-size-bytes-does-not-match-the-vm-data-disk-logical-size-bytes"></a> VM 中数据磁盘的物理 Blob 字节大小与 VM 数据磁盘的逻辑字节大小不匹配。

当数据磁盘逻辑大小与实际 VHD Blob 大小不同步时，将发生这种情况。可以轻松使用以下命令验证这种问题：

#### 验证问题

    # Store the VM details in the VM object
    $vm = Get-AzureVM -ServiceName $servicename -Name $vmname

    # Display the data disk properties
    # NOTE the data disk LogicalDiskSizeInGB below which is 11GB. Also note the MediaLink Uri of the VHD blob as we'll use this in the next step
    $vm.VM.DataVirtualHardDisks

    HostCaching         : None
    DiskLabel           : 
    DiskName            : coreosvm-coreosvm-0-201611230636240687
    Lun                 : 0
    LogicalDiskSizeInGB : 11
    MediaLink           : https://contosostorage.blob.core.chinacloudapi.cn/vhds/coreosvm-dd1.vhd
    SourceMediaLink     : 
    IOType              : Standard
    ExtensionData       : 

    # Now get the properties of the blob backing the data disk above
    # NOTE the size of the blob is about 15 GB which is different from LogicalDiskSizeInGB above
    $blob = Get-AzureStorageblob -Blob "coreosvm-dd1.vhd" -Container vhds 

    $blob

    ICloudBlob        : Microsoft.WindowsAzure.Storage.Blob.CloudPageBlob
    BlobType          : PageBlob
    Length            : 16106127872
    ContentType       : application/octet-stream
    LastModified      : 11/23/2016 7:16:22 AM +00:00
    SnapshotTime      : 
    ContinuationToken : 
    Context           : Microsoft.WindowsAzure.Commands.Common.Storage.AzureStorageContext
    Name              : coreosvm-dd1.vhd

#### 缓解问题

    # Convert the blob size in bytes to GB into a variable which we'll use later
    $newSize = [int]($blob.Length / 1GB)

    # See the calculated size in GB
    $newSize

    15

    # Store the disk name of the data disk as we'll use this to identify the disk to be updated
    $diskName = $vm.VM.DataVirtualHardDisks[0].DiskName

    # Identify the LUN of the data disk to remove
    $lunToRemove = $vm.VM.DataVirtualHardDisks[0].Lun

    # Now remove the data disk from the VM so that the disk isn't leased by the VM and it's size can be updated
    Remove-AzureDataDisk -LUN $lunToRemove -VM $vm | Update-AzureVm -Name $vmname -ServiceName $servicename

    OperationDescription OperationId                          OperationStatus
    -------------------- -----------                          ---------------
    Update-AzureVM       213xx1-b44b-1v6n-23gg-591f2a13cd16   Succeeded  

    # Verify we have the right disk that's going to be updated
    Get-AzureDisk -DiskName $diskName

    AffinityGroup        : 
    AttachedTo           : 
    IsCorrupted          : False
    Label                : 
    Location             : China East
    DiskSizeInGB         : 11
    MediaLink            : https://contosostorage.blob.core.chinacloudapi.cn/vhds/coreosvm-dd1.vhd
    DiskName             : coreosvm-coreosvm-0-201611230636240687
    SourceImageName      : 
    OS                   : 
    IOType               : Standard
    OperationDescription : Get-AzureDisk
    OperationId          : 0c56a2b7-a325-123b-7043-74c27d5a61fd
    OperationStatus      : Succeeded

    # Now update the disk to the new size
    Update-AzureDisk -DiskName $diskName -ResizedSizeInGB $newSize -Label $diskName

    OperationDescription OperationId                          OperationStatus
    -------------------- -----------                          ---------------
    Update-AzureDisk     cv134b65-1b6n-8908-abuo-ce9e395ac3e7 Succeeded 

    # Now verify that the "DiskSizeInGB" property of the disk matches the size of the blob 
    Get-AzureDisk -DiskName $diskName

    AffinityGroup        : 
    AttachedTo           : 
    IsCorrupted          : False
    Label                : coreosvm-coreosvm-0-201611230636240687
    Location             : China East
    DiskSizeInGB         : 15
    MediaLink            : https://contosostorage.blob.core.chinacloudapi.cn/vhds/coreosvm-dd1.vhd
    DiskName             : coreosvm-coreosvm-0-201611230636240687
    SourceImageName      : 
    OS                   : 
    IOType               : Standard
    OperationDescription : Get-AzureDisk
    OperationId          : 1v53bde5-cv56-5621-9078-16b9c8a0bad2
    OperationStatus      : Succeeded

    # Now we'll add the disk back to the VM as a data disk. First we need to get an updated VM object
    $vm = Get-AzureVM -ServiceName $servicename -Name $vmname

    Add-AzureDataDisk -Import -DiskName $diskName -LUN 0 -VM $vm -HostCaching ReadWrite | Update-AzureVm -Name $vmname -ServiceName $servicename

    OperationDescription OperationId                          OperationStatus
    -------------------- -----------                          ---------------
    Update-AzureVM       b0ad3d4c-4v68-45vb-xxc1-134fd010d0f8 Succeeded      

## 后续步骤
下面是说明该过程的迁移文章的列表。

* [平台支持的从经典部署模型到 Azure Resource Manager 的 IaaS 资源迁移](/documentation/articles/virtual-machines-windows-migration-classic-resource-manager/)
* [使用 PowerShell 将 IaaS 资源从经典部署模型迁移到 Azure Resource Manager](/documentation/articles/virtual-machines-windows-ps-migration-classic-resource-manager/)
* [使用 CLI 将 IaaS 资源从经典部署模型迁移到 Azure Resource Manager](/documentation/articles/virtual-machines-linux-cli-migration-classic-resource-manager/)

<!---HONumber=Mooncake_0116_2017-->
<!--Update_Description: add detailed troubleshooting-->