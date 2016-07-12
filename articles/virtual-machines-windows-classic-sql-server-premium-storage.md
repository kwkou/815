<properties
	pageTitle="在 SQL Server 上使用 Azure 高级存储 | Azure"
	description="本文使用通过经典部署模型创建的资源并介绍如何对 Azure 虚拟机上运行的 SQL Server 使用 Azure 高级存储。"
	services="virtual-machines-windows"
	documentationCenter=""
	authors="danielsollondon"
	manager="jeffreyg"
	editor="monicar"    
	tags="azure-service-management"/>

<tags
	ms.service="virtual-machines-windows"
	ms.date="05/04/2016"
	wacn.date="06/29/2016"/>

# 将 Azure 高级存储用于虚拟机上的 SQL Server


## 概述

[Azure 高级存储](/documentation/articles/storage-premium-storage/)是下一代提供低延迟和高吞吐量 IO 的存储。它最适用于关键 IO 密集型工作负荷，例如 IaaS [虚拟机](/home/features/virtual-machines/)上的 SQL Server。

> [AZURE.IMPORTANT]Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model/)。本文介绍使用经典部署模型。Azure 建议大多数新部署使用资源管理器模型。

本文提供迁移运行 SQL Server 的虚拟机以使用高级存储的规划和指南。这包括 Azure 基础结构（网络、存储）以及来宾 Windows VM 步骤。[附录](#appendix-migrating-a-multisite-alwayson-cluster-to-premium-storage)中的示例显示如何移动较大的 VM 以通过 PowerShell 利用改进的本地 SSD 存储的完整全面的端到端迁移。

请务必了解将 Azure 高级存储用于 IAAS VM 上的 SQL Server 的端到端过程。这包括：

- 确定使用高级存储的先决条件。
- 作为新部署，将 IaaS 上的 SQL Server 部署到高级存储的示例。
- 迁移现有部署（包括独立服务器和使用 SQL AlwaysOn 可用性组的部署）的示例。
- 可能的迁移方法。
- 演示迁移现有 AlwaysOn 实现的 Azure、Windows 和 SQL Server 步骤的完整端到端示例。

有关 Azure 虚拟机中的 SQL Server 的更多背景信息，请参阅 [Azure 虚拟机中的 SQL Server](/documentation/articles/virtual-machines-windows-sql-server-iaas-overview/)。

**作者：**Daniel Sol
**技术审阅人员：**Luis Carlos Vargas Herring、Sanjay Mishra、Pravin Mital、Juergen Thomas、Gonzalo Ruiz。

##<a name="prerequisites-for-premium-storage"></a> 高级存储的先决条件

使用高级存储有多个先决条件。

### 虚拟机大小

要使用高级存储，需要使用 DS 系列虚拟机 (VM)。如果你以前尚未在云服务中使用 DS 系列虚拟机，则必须在重新创建 VM 作为 DS* 角色大小之前，删除现有 VM、保留附加磁盘，然后创建新的云服务。有关虚拟机大小的详细信息，请参阅 [Azure 的虚拟机和云服务大小](/documentation/articles/virtual-machines-windows-sizes/)。

### 云服务

在新的云服务中创建 VM 时，只能将 DS* VM 用于高级存储。如果你在 Azure 中使用 SQL Server AlwaysOn，则 AlwaysOn 侦听器将引用与云服务关联的 Azure 内部或外部负载平衡器 IP 地址。本文重点介绍如何在此方案中迁移，同时保持可用性。

> [AZURE.NOTE] DS* 系列必须是部署到新的云服务的第一个 VM。

### 区域 VNET

对于 DS* VM，必须将托管你的 VM 的虚拟网络 (VNET) 配置为区域虚拟网络。这将“加宽”虚拟网络，目的是允许在其他群集中设置更大的 VM，并允许这些 VM 之间进行通信。在以下屏幕截图中，突出显示的位置显示的是区域 VNET，而第一个结果则显示的是“窄”VNET。

![RegionalVNET][1]

你可以提供 Microsoft 支持票证以迁移到区域 VNET，Microsoft 将进行更改，然后为完成迁移到区域 VNET，更改网络配置中的 AffinityGroup 属性。首先导出 PowerShell 中的网络配置，然后将 **VirtualNetworkSite** 元素中的 **AffinityGroup** 属性替换为 **Location** 属性。指定 `Location = XXXX`，其中 `XXXX` 是 Azure 区域。然后导入新配置。

例如，考虑以下 VNET 配置：

    <VirtualNetworkSite name="danAzureSQLnet" AffinityGroup="AzureSQLNetwork">
    <AddressSpace>
      <AddressPrefix>10.0.0.0/8</AddressPrefix>
      <AddressPrefix>172.16.0.0/12</AddressPrefix>
    </AddressSpace>
    <Subnets>
    ...
    </VirtualNetworkSite>

若要将此项移到在欧洲西部的区域 VNET，请将配置更改为以下内容：

    <VirtualNetworkSite name="danAzureSQLnet" Location="West Europe">
    <AddressSpace>
      <AddressPrefix>10.0.0.0/8</AddressPrefix>
      <AddressPrefix>172.16.0.0/12</AddressPrefix>
    </AddressSpace>
    <Subnets>
    ...
    </VirtualNetworkSite>

### 存储帐户

你将需要创建专为高级存储配置的新存储帐户。请注意，在存储帐户（而不是单个 VHD）上设置使用高级存储，但使用 DS* 系列 VM 时，则可以从高级和标准存储帐户附加 VHD。如果不想将操作系统 VHD 放到高级存储帐户上，则可以考虑此项。

以下使用“Premium\_LRS”**类型**的 **New-AzureStorageAccountPowerShell** 命令将创建高级存储帐户：

    $newstorageaccountname = "danpremstor"
    New-AzureStorageAccount -StorageAccountName $newstorageaccountname -Location "West Europe" -Type "Premium_LRS"   

### VHD 缓存设置

创建属于高级存储帐户的磁盘之间的主要区别是磁盘缓存设置。对于 SQL Server 数据卷磁盘，建议你使用“**读取缓存**”。对于事务日志卷，磁盘缓存设置应设为“**无**”。这不同于针对标准存储帐户的建议。

附加 VHD 后，便不能变更缓存设置。你将需要分离 VHD，然后再使用已更新的缓存设置重新附加该 VHD。

### Windows 存储空间

你可以像使用以前的标准存储一样使用 [Windows 存储空间](https://technet.microsoft.com/zh-cn/library/hh831739.aspx)，这将允许你迁移已在利用存储空间的 VM。[附录](#appendix-migrating-a-multisite-alwayson-cluster-to-premium-storage)中的示例（步骤 9 及前面的步骤）演示了提取并导入附加了多个 VHD 的 VM 的 Powershell 代码。

存储池已用于标准 Azure 存储帐户以提高吞吐量并减少延迟。在使用新部署的高级存储测试存储池时，你可能会发现价值，但这样做会为存储设置添加额外的复杂性。

#### 如何查明哪些 Azure 虚拟磁盘映射到存储池

因为针对附加 VHD 有不同的缓存设置建议，你可能会决定将 VHD 复制到高级存储帐户。但是，当你将它们重新附加到新的 DS 系列 VM 时，可能需要变更缓存设置。当你对 SQL 数据文件和日志文件使用单独的 VHD（而不是同时包含这两种文件的单个 VHD）时，应用高级存储建议的缓存设置将更为简单。

> [AZURE.NOTE] 如果你在同一卷上有 SQL Server 数据和日志文件，则所选的缓存选项将取决于你的数据库工作负荷的 IO 访问模式。只有测试可演示哪个缓存选项最适用于这种情况。

但是，如果你使用的是由多个 VHD 构成的 Windows 存储空间，则你将需要查看原始脚本，以确定哪些附加 VHD 在哪个特定池中，这样便可以为每个磁盘相应地设置缓存设置。

如果你没有可向你显示哪些 VHD 映射到存储池的原始脚本，则可以使用以下步骤来确定磁盘/存储池映射。

对于每个磁盘，使用以下步骤：

1. 使用 **Get-AzureVM** 命令获取附加到 VM 的磁盘列表：

    Get-AzureVM -ServiceName <servicename> -Name <vmname> | Get-AzureDataDisk

1. 记下 Diskname 和 LUN。

	![DisknameAndLUN][2]

1. 通过远程桌面连接到 VM。然后转到**计算机管理** | **设备管理器** | **磁盘驱动器**。查看每个 Microsoft 虚拟磁盘的属性

	![VirtualDiskProperties][3]

1. 此处的 LUN 号是对你在将 VHD 附加到 VM 时指定的 LUN 号的引用。
1. 对于 Microsoft 虚拟磁盘，转到“详细信息”选项卡，然后在“属性”列表中转到“驱动程序键”。在**“值”**中，注意**“偏移量”**，该项在下面的屏幕截图中为 0002。0002 表示存储池引用的 PhysicalDisk2。

	![VirtualDiskPropertyDetails][4]

2. 对于每个存储池，转储关联的磁盘：

    Get-StoragePool -FriendlyName AMS1pooldata | Get-PhysicalDisk

	![GetStoragePool][5]

现在，你可以使用此信息将附加 VHD 关联到存储池中的物理磁盘。

将 VHD 映射到存储池中的物理磁盘后，可以将其分离并复制到高级存储帐户，然后使用正确的缓存设置附加它们。请参阅[附录](#appendix-migrating-a-multisite-alwayson-cluster-to-premium-storage)中的示例，步骤 8 到步骤 12。以下步骤显示如何将 VM 附加的 VHD 磁盘配置提取到 CSV 文件、复制 VHD、变更磁盘配置缓存设置，最后将 VM 重新部署为带所有附加磁盘的 DS 系列 VM。

### VM 存储带宽和 VHD 存储吞吐量

存储量性能取决于指定的 DS* VM 大小和 VHD 大小。VM 针对可附加的 VHD 数量以及它们支持的最大带宽（MB/秒）提供不同限额。有关特定带宽数字，请参阅 [Azure 的虚拟机和云服务大小](/documentation/articles/virtual-machines-windows-sizes/)。

较大的磁盘大小可提高 IOPS。当你考虑迁移路径时，应考虑这一点。有关详细信息，[请参阅 IOPS 和磁盘类型的表](/documentation/articles/storage-premium-storage/#scalability-and-performance-targets-when-using-premium-storage)。

最后，考虑到 VM 具有不同的最大磁盘带宽，它们将支持所有附加磁盘。在高负载下可使可供该 VM 角色大小使用的最大磁盘带宽饱和。例如，Standard\_DS14 将最多支持 512MB/秒；因此，使用三个 P30 磁盘可使 VM 的磁盘带宽饱和。但在此示例中，可以超出吞吐量限制，具体取决于读取和写入 IO 的组合。

## 新建部署

接下来的两部分将演示如何将 SQL Server VM 部署到高级存储。如前所述，不一定需要将操作系统磁盘放到高级存储上。如果你打算将任何密集型 IO 工作负荷放置在操作系统 VHD 上，则可以选择这样做。

第一个示例演示如何利用现有 Azure 库映像。第二个示例演示如何使用你在现有标准存储帐户中拥有的自定义 VM 映像。

> [AZURE.NOTE] 这些示例假定你已创建区域 VNET。

### 使用库映像创建带高级存储的新 VM

下面的示例演示如何将操作系统 VHD 放置到高级存储上并附加高级存储 VHD。但是，也可以将操作系统磁盘放置在标准存储帐户中，然后附加驻留在高级存储帐户中的 VHD。将演示这两种方案。

    $mysubscription = "DansSubscription"
    $location = "West Europe"

    #Set up subscription
    Set-AzureSubscription -Environment AzureChinaCloud -SubscriptionName $mysubscription
    Select-AzureSubscription -Environment AzureChinaCloud -SubscriptionName $mysubscription -Current  

#### 步骤 1：创建高级存储帐户


    #Create Premium Storage account, note Type
    $newxiostorageaccountname = "danspremsams"
    New-AzureStorageAccount -StorageAccountName $newxiostorageaccountname -Location $location -Type "Premium_LRS"  


#### 步骤 2：创建新的云服务

    $destcloudsvc = "danNewSvcAms"
    New-AzureService $destcloudsvc -Location $location


#### 步骤 3：保留云服务 VIP（可选）
    #check exisitng reserved VIP
    Get-AzureReservedIP

    $reservedVIPName = "sqlcloudVIP"
    New-AzureReservedIP -ReservedIPName $reservedVIPName -Label $reservedVIPName -Location $location

#### 步骤 4：创建 VM 容器
    #Generate storage keys for later
    $xiostorage = Get-AzureStorageKey -StorageAccountName $newxiostorageaccountname

    ##Generate storage acc contexts
    $xioContext = New-AzureStorageContext -Environment AzureChinaCloud -StorageAccountName $newxiostorageaccountname -StorageAccountKey $xiostorage.Primary   

    #Create container
    $containerName = 'vhds'
    New-AzureStorageContainer -Name $containerName -Context $xioContext

#### 步骤 5：将操作系统 VHD 放置在标准或高级存储上
    #NOTE: Set up subscription and default storage account which will be used to place the OS VHD in

    #If you want to place the OS VHD Premium Storage Account
    Set-AzureSubscription -Environment AzureChinaCloud -SubscriptionName $mysubscription -CurrentStorageAccount  $newxiostorageaccountname  

    #If you wanted to place the OS VHD Standard Storage Account but attach Premium Storage VHDs then you would run this instead:
    $standardstorageaccountname = "danstdams"

    Set-AzureSubscription -Environment AzureChinaCloud -SubscriptionName $mysubscription -CurrentStorageAccount  $standardstorageaccountname

#### 步骤 6：创建 VM
    #Get list of available SQL Server Images from the Azure Image Gallery.
    $galleryImage = Get-AzureVMImage | where-object {$_.ImageName -like "*SQL*2014*Enterprise*"}
    $image = $galleryImage.ImageName

    #Set up Machine Specific Information
    $vmName = "dsDan1"
    $vnet = "dansvnetwesteur"
    $subnet = "SQL"
    $ipaddr = "192.168.0.8"

    #Remember to change to DS series VM
    $newInstanceSize = "Standard_DS1"

    #create new Avaiability Set
    $availabilitySet = "cloudmigAVAMS"

    #Machine User Credentials
    $userName = "myadmin"
    $pass = "mycomplexpwd4*"

    #Create VM Config
    $vmConfigsl = New-AzureVMConfig -Name $vmName -InstanceSize $newInstanceSize -ImageName $image  -AvailabilitySetName $availabilitySet  ` | Add-AzureProvisioningConfig -Windows ` -AdminUserName $userName -Password $pass | Set-AzureSubnet -SubnetNames $subnet | Set-AzureStaticVNetIP -IPAddress $ipaddr

    #Add Data and Log Disks to VM Config
    #Note the size specified '-DiskSizeInGB 1023', this will attach 2 x P30 Premium Storage Disk Type
    #Utilising the Premium Storage enabled Storage account

    $vmConfigsl | Add-AzureDataDisk -CreateNew -DiskSizeInGB 1023 -LUN 0 -HostCaching "ReadOnly"  -DiskLabel "DataDisk1" -MediaLocation "https://$newxiostorageaccountname.blob.core.chinacloudapi.cn/vhds/$vmName-data1.vhd"
    $vmConfigsl | Add-AzureDataDisk -CreateNew -DiskSizeInGB 1023 -LUN 1 -HostCaching "None"  -DiskLabel "logDisk1" -MediaLocation "https://$newxiostorageaccountname.blob.core.chinacloudapi.cn/vhds/$vmName-log1.vhd"

    #Create VM
    $vmConfigsl  | New-AzureVM -ServiceName $destcloudsvc -VNetName $vnet ## Optional (-ReservedIPName $reservedVIPName)  

    #Add RDP Endpoint
    $EndpointNameRDPInt = "3389"
    Get-AzureVM -ServiceName $destcloudsvc -Name $vmName | Add-AzureEndpoint -Name "EndpointNameRDP" -Protocol "TCP" -PublicPort "53385" -LocalPort $EndpointNameRDPInt  | Update-AzureVM

    #Check VHD storage account, these should be in $newxiostorageaccountname
    Get-AzureVM -ServiceName $destcloudsvc -Name $vmName | Get-AzureDataDisk
    Get-AzureVM -ServiceName $destcloudsvc -Name $vmName |Get-AzureOSDisk


### 使用自定义映像创建新的 VM 以使用高级存储

此方案演示拥有驻留在标准存储帐户中的现有自定义映像的位置。如前所述，如果要将操作系统 VHD 放置在高级存储上，则需要复制标准存储帐户中存在的映像，并将它们传输到高级存储中，然后才能使用它。如果你在本地有一个映像，也可以使用此方法将该映像直接复制到高级存储帐户。

#### 步骤 1：创建存储帐户
    $mysubscription = "DansSubscription"
    $location = "West Europe"

    #Create Premium Storage account
    $newxiostorageaccountname = "danspremsams"
    New-AzureStorageAccount -StorageAccountName $newxiostorageaccountname -Location $location -Type "Premium_LRS"  

    #Standard Storage account
    $origstorageaccountname = "danstdams"

#### 步骤 2：创建云服务
    $destcloudsvc = "danNewSvcAms"
    New-AzureService $destcloudsvc -Location $location


#### 步骤 3：使用现有映像
可以使用现有映像，也可以[创建现有虚拟机的映像](/documentation/articles/virtual-machines-windows-classic-capture-image/)。请注意，你要创建映像的虚拟机不一定是 DS* 虚拟机。获得映像后，以下步骤演示如何使用 **Start-AzureStorageBlobCopy** PowerShell commandlet 将其复制到高级存储帐户。

    #Get storage account keys:
    #Standard Storage account
    $originalstorage =  Get-AzureStorageKey -StorageAccountName $origstorageaccountname
    #Premium Storage account
    $xiostorage = Get-AzureStorageKey -StorageAccountName $newxiostorageaccountname

    #Set up contexts for the storage accounts:
    $origContext = New-AzureStorageContext -Environment AzureChinaCloud -StorageAccountName $origstorageaccountname -StorageAccountKey $originalstorage.Primary
    $destContext = New-AzureStorageContext -Environment AzureChinaCloud -StorageAccountName $newxiostorageaccountname -StorageAccountKey $xiostorage.Primary  

#### 步骤 4：在存储帐户之间复制 Blob
    #Get Image VHD
    $myImageVHD = "dansoldonorsql2k14-os-2015-04-15.vhd"
    $containerName = 'vhds'

    #Copy Blob between accounts
    $blob = Start-AzureStorageBlobCopy -SrcBlob $myImageVHD -SrcContainer $containerName `
    -DestContainer vhds -Destblob "prem-$myImageVHD" `
    -Context $origContext -DestContext $destContext  

#### 步骤 5：定期检查复制状态：
    $blob | Get-AzureStorageBlobCopyState

#### 步骤 6：将映像磁盘添加到订阅中的 Azure 磁盘存储库
    $imageMediaLocation = $destContext.BlobEndPoint+"/"+$myImageVHD
    $newimageName = "prem"+"dansoldonorsql2k14"

    Add-AzureVMImage -ImageName $newimageName -MediaLocation $imageMediaLocation

> [AZURE.NOTE] 你可能会发现即使状态报告为成功，你也仍会收到磁盘租约错误。在这种情况下，请等待大约 10 分钟。

#### 步骤 7：生成 VM
在此处你将基于映像生成 VM 并附加两个高级存储 VHD：

    $newimageName = "prem"+"dansoldonorsql2k14"
    #Set up Machine Specific Information
    $vmName = "dansolchild"
    $vnet = "westeur"
    $subnet = "Clients"
    $ipaddr = "192.168.0.41"

    #This will need to be a new cloud service
    $destcloudsvc = "danregsvcamsxio2"

    #Use to DS Series VM
    $newInstanceSize = "Standard_DS1"

    #create new Avaiability Set
    $availabilitySet = "cloudmigAVAMS3"

    #Machine User Credentials
    $userName = "myadmin"
    $pass = "theM)stC0mplexP@ssw0rd!"


    #Create VM Config
    $vmConfigsl2 = New-AzureVMConfig -Name $vmName -InstanceSize $newInstanceSize -ImageName $newimageName  -AvailabilitySetName $availabilitySet  ` | Add-AzureProvisioningConfig -Windows ` -AdminUserName $userName -Password $pass | Set-AzureSubnet -SubnetNames $subnet | Set-AzureStaticVNetIP -IPAddress $ipaddr

    $vmConfigsl2 | Add-AzureDataDisk -CreateNew -DiskSizeInGB 1023 -LUN 0 -HostCaching "ReadOnly"  -DiskLabel "DataDisk1" -MediaLocation "https://$newxiostorageaccountname.blob.core.chinacloudapi.cn/vhds/$vmName-Datadisk-1.vhd"
    $vmConfigsl2 | Add-AzureDataDisk -CreateNew -DiskSizeInGB 1023 -LUN 1 -HostCaching "None"  -DiskLabel "LogDisk1" -MediaLocation "https://$newxiostorageaccountname.blob.core.chinacloudapi.cn/vhds/$vmName-logdisk-1.vhd"



    $vmConfigsl2 | New-AzureVM -ServiceName $destcloudsvc -VNetName $vnet

## 未使用 AlwaysOn 可用性组的现有部署

> [AZURE.NOTE] 对于现有部署，请首先参阅本主题的[先决条件](#prerequisites-for-premium-storage)部分。

未使用 AlwaysOn 可用性组的 SQL Server 部署和使用这些组的 SQL Server 部署有不同的注意事项。如果你未使用 AlwaysOn 并且有现有的独立 SQL Server，则可以通过使用新的云服务和存储帐户升级到高级存储。请考虑以下选项：

- **创建新的 SQL Server VM**。你可以创建使用高级存储帐户的新 SQL Server VM，如“新建部署”中所述。然后备份并还原 SQL Server 配置和用户数据库。将需要更新应用程序以引用新的 SQL Server（如果正在从内部或外部访问它）。你将需要复制所有“数据库外”对象，就像执行并排 (SxS) SQL Server 迁移一样。这包括登录名、证书和链接服务器等对象。
- **迁移现有 SQL Server VM**。这将需要使 SQL Server VM 脱机，然后将其传输到新的云服务，包括将其所有附加 VHD 复制到高级存储帐户。当 VM 联机时，应用程序将像以前一样引用服务器主机名。请注意，现有磁盘的大小会影响性能特性。例如，400 GB 磁盘将向上舍入到 P20。如果你知道你不需要该磁盘性能，则可以将 VM 重新创建为 DS 系列 VM，并附加具有所需大小/性能指标的高级存储 VHD。然后，可以分离并重新附加 SQL DB 文件。

> [AZURE.NOTE] 复制 VHD 磁盘时应注意大小，取决于大小意味着这些磁盘将归入什么高级存储磁盘类型，此类型将确定磁盘性能指标。Azure 将向上舍入到最接近的磁盘大小，因此，如果你有一个 400GB 磁盘，则此磁盘将向上舍入到 P20。根据操作系统 VHD 的现有 IO 要求，你可能不需要将此 VHD 迁移到高级存储帐户。

如果从外部访问你的 SQL Server，则云服务 VIP 将更改。你还将需要更新终结点、ACL 和 DNS 设置。

## 使用 AlwaysOn 可用性组的现有部署

> [AZURE.NOTE] 对于现有部署，请首先参阅本主题的[先决条件](#prerequisites-for-premium-storage)部分。

在本部分开始，我们将了解 AlwaysOn 如何与 Azure 网络交互。然后，我们会将迁移细分为两种方案：可以容忍停机一段时间的迁移和必须实现最短停机时间的迁移。

本地 SQL Server AlwaysOn 可用性组在本地使用这样的侦听器：它注册一个虚拟 DNS 名称，以及在一个或多个 SQL Server 之间共享的一个 IP 地址。当客户端连接时，会将它们通过侦听器 IP 路由到主 SQL Server。这是在该时间拥有 AlwaysOn IP 资源的服务器。

![DeploymentsUseAlwaysOn][6]

在 Azure 中，只能将一个 IP 地址分配给 VM 上的 NIC，因此，为了实现与本地相同的抽象层，Azure 将利用分配给内部/外部负载平衡器 (ILB/ELB) 的 IP 地址。在服务器间共享的 IP 资源将设置为与 ILB/ELB 相同的 IP。此 IP 在 DNS 中发布，客户端流量将通过 ILB/ELB 传递到主 SQL Server 副本。ILB/ELB 知道哪个 SQL Server 为主，因为它使用探测器来探测 AlwaysOn IP 资源。在前面的示例中，它会探测包含 ELB/ILB 引用的终结点的每个节点，做出响应的则是主 SQL Server。

> [AZURE.NOTE] ILB 和 ELB 都分配给特定 Azure 云服务，因此 Azure 中的任何云迁移都很可能意味着负载平衡器 IP 将更改。

### 迁移可以允许停机一段时间的 AlwaysOn 部署

有两种策略可以迁移允许停机一段时间的 AlwaysOn 部署：

1. **将更多辅助副本添加到现有 AlwaysOn 群集**
1. **迁移到新的 AlwaysOn 群集**

#### 1\.将更多辅助副本添加到现有 AlwaysOn 群集

一种策略是将更多辅助副本添加到 AlwaysOn 可用性组。你需要将这些辅助副本添加到新的云服务中，并使用新的负载平衡器 IP 更新侦听器。

##### 停机时间点：

- 群集验证。
- 对新的辅助副本测试 AlwaysOn 故障转移。

如果为提高 IO 吞吐量你使用的是 VM 中的 Windows 存储池，则在进行完整的群集验证过程中，会将这些存储池脱机。向群集添加节点时需要进行验证测试。运行测试所需的时间可能会有所不同，因此你应在有代表性的测试环境中对此进行测试，以获得这将需要的大致时间。

你应设置可以对新添加的节点执行手动故障转移和混沌测试的时间，以确保 AlwaysOn 高可用性功能正常工作。

![DeploymentUseAlwaysOn2][7]

> [AZURE.NOTE] 运行验证前，你应停止使用存储池的 SQL Server 的所有实例。
##### 大致步骤

1. 在使用附加高级存储的新云服务中创建两个新的 SQL Server。
1. 使用 **NORECOVERY** 复制完整备份并进行还原。
1. 复制“用户数据库外”依赖对象，例如登录名等。
1. 新建内部负载平衡器 (ILB) 或使用外部负载平衡器 (ELB)，然后在这两个新节点上设置负载平衡终结点。
> [AZURE.NOTE] 继续下一步之前，检查所有节点的终结点配置是否正确

1. 禁止用户/应用程序访问 SQL Server（如果使用存储池）。
1. 停止所有节点上的 SQL Server 引擎服务（如果使用存储池）。
1. 将新节点添加到群集并运行完整验证。
1. 成功验证后，启动所有 SQL Server 服务。
1. 备份事务日志并还原用户数据库。
1. 将新节点添加到 AlwaysOn 可用性组中，并将复制置为**“同步”**。
1. 根据[附录](#appendix-migrating-a-multisite-alwayson-cluster-to-premium-storage)中的多站点示例，通过 PowerShell 为 AlwaysOn 添加新云服务 ILB/ELB 的 IP 地址资源。在 Windows 群集中，将 **IP 地址**资源的**可能所有者**设置为新节点 old。请参阅[附录](#appendix-migrating-a-multisite-alwayson-cluster-to-premium-storage)的“在同一子网中添加 IP 地址资源”部分。
1. 故障转移到新节点之一。
1. 将新节点设为“自动故障转移伙伴”并测试故障转移。
1. 从可用性组中删除原始节点。

##### 优点

- 在将新 SQL Server（SQL Server 和应用程序）添加到 AlwaysOn 之前，可以对其进行测试。
- 你可以根据你的确切需求更改 VM 大小和自定义存储。但是，保持所有 SQL 文件路径不变将会非常有益。
- 你可以控制何时开始将数据库备份传输到辅助副本。这不同于使用 Azure **Start-AzureStorageBlobCopy** commandlet 复制 VHD，因为这是异步复制。

##### 缺点
- 使用 Windows 存储池时，在对新的附加节点进行完整群集验证的过程中，将会存在群集停机时间。
- 根据 SQL Server 版本和现有的辅助副本数，在不删除现有辅助副本的情况下，你可能不能添加更多辅助副本。
- 设置辅助副本时，SQL 数据传输时间可能会很长。
- 在迁移期间，当你并行运行新计算机时，会产生额外的费用。

#### 2\.迁移到新的 AlwaysOn 群集

另一种策略是在新的云服务中创建包含全新节点的全新 AlwaysOn 群集，然后重定向客户端以使用它。

##### 停机时间点

将应用程序和用户转移到新的 AlwaysOn 侦听器时，会出现停机时间。停机时间取决于：

- 将最后一个事务日志备份还原到新服务器上的数据库时所用的时间。
- 更新客户端应用程序以使用新的 AlwaysOn 侦听器时所用的时间。

##### 优点

- 你可以测试实际生产环境、SQL Server 和操作系统生成更改。
- 你可以选择自定义存储并可能减小 VM 的大小。这可以导致成本降低。
- 你可以在此过程中更新 SQL Server 版本。还可以升级操作系统。
- 以前的 AlwaysOn 群集可以充当稳定的回滚目标。

##### 缺点

- 如果希望同时运行两个 AlwaysOn 群集，则需要更改侦听器的 DNS 名称。这会增加迁移过程中的管理开销，因为客户端应用程序字符串必须反映新的侦听器名称。
- 你必须实现两种环境之间的同步机制，使它们尽可能接近，以最大程度地降低在迁移之前执行最终同步的要求。
- 在迁移期间运行新环境会增加成本。

### 停机时间最短的迁移 AlwaysOn 部署

有两种迁移 AlwaysOn 部署的策略可实现最短停机时间：

1. **利用现有的辅助副本：单站点**
1. **利用现有的辅助副本：多站点**

#### 1\.利用现有的辅助副本：单站点

停机时间最短的一个策略是获取现有的云辅助版本并将其从当前云服务中删除。然后将 VHD 复制到新的高级存储帐户，并在新的云服务中创建 VM。然后，在群集和故障转移中更新侦听器。

##### 停机时间点

- 使用负载平衡终结点更新最后一个节点时，会出现停机时间。
- 你的客户端重新连接可能会延迟，具体取决于你的客户端/DNS 配置。
- 如果你选择将 AlwaysOn 群集组脱机来换出 IP 地址，则会增加停机时间。可以通过对添加的 IP 地址资源使用 OR 依赖关系和可能的所有者来避免出现这种情况。请参阅[附录](#appendix-migrating-a-multisite-alwayson-cluster-to-premium-storage)的“在同一子网中添加 IP 地址资源”部分。

> [AZURE.NOTE] 如果你想让添加的节点作为 AlwaysOn 故障转移伙伴参与其中，则需要为 Azure 终结点添加对负载平衡集的引用。当你通过运行 **Add-AzureEndpoint** 命令来执行此操作时，当前连接将保持打开，但在更新负载平衡器之前，将无法与侦听器建立新连接。在测试时，看到此现像持续 90 到 120 秒，应该对此进行测试。

##### 优点

- 在迁移期间不会产生额外的费用。
- 一对一迁移。
- 降低了复杂性。
- 可提高高级存储 SKU 的 IOPS。将磁盘与 VM 分离并复制到新的云服务中时，可以使用第三方工具来增加 VHD 大小，以便提供更高的吞吐量。有关增加 VHD 大小，请参阅此[论坛讨论](https://social.msdn.microsoft.com/Forums/azure/4a9bcc9e-e5bf-4125-9994-7c154c9b0d52/resizing-azure-data-disk?forum=WAVirtualMachinesforWindows)。

##### 缺点

- 在迁移期间，会暂时失去高可用性和灾难恢复。
- 由于这是 1 对 1 迁移，你必须使用支持你的 VHD 数量的最小 VM 大小，因此你可能不能减小 VM 大小。
- 此方案会使用 Azure **Start-AzureStorageBlobCopy** commandlet，它是异步的。复制完成后没有 SLA。复制时间各不相同，这取决于在队列中等待的时间，还取决于要传输的数据量。如果传输转到另一个在其他区域中支持高级存储的 Azure 数据中心，则复制时间将增加。如果你只有 2 个节点，请考虑在复制所用时间长于测试时的可能的缓解措施。这可以包括以下建议。
	- 在使用商定的停机时间进行迁移之前，添加临时的第 3 个 SQL Server 节点以实现 HA。
	- 在 Azure 计划的维护之外运行迁移。
	- 确保已正确配置群集仲裁。  

##### 大致步骤

本文档并不演示完整的端到端示例，但是[附录](#appendix-migrating-a-multisite-alwayson-cluster-to-premium-storage)提供了可用来执行此操作的详细信息。

![MinimalDowntime][8]

- 收集磁盘配置并删除节点（不删除附加的 VHD）。
- 创建高级存储帐户并从标准存储帐户复制 VHD
- 创建新的云服务并在该云服务中重新部署 SQL2 VM。使用复制的原始操作系统 VHD 创建 VM 并附加复制的 VHD。
- 配置 ILB/ELB 并添加终结点。
- 通过以下任一方法更新侦听器：
	- 将 AlwaysOn 组脱机并使用新的 ILB/ELB IP 地址更新 AlwaysOn 侦听器。
	- 或者通过 PowerShell 将新云服务 ILB/ELB 的 IP 地址资源添加到 Windows 群集。然后，将 IP 地址资源的可能所有者设置为已迁移节点 SQL2，并在网络名称中将此项设置为 OR 依赖关系。请参阅[附录](#appendix-migrating-a-multisite-alwayson-cluster-to-premium-storage)的“在同一子网中添加 IP 地址资源”部分。
- 检查客户端的 DNS 配置/传播。
- 迁移 SQL1 VM，并完成步骤 2 - 4。
- 如果使用步骤 5ii，则将 SQL1 添加为已添加的 IP 地址资源的可能所有者
- 测试故障转移。

#### 2\.利用现有的辅助副本：多站点

如果你在多个 Azure 数据中心 (DC) 中有节点，或者你拥有混合环境，则可以在该环境中使用 AlwaysOn 配置最大程度地减少停机时间。

此方法的目的是将本地或辅助 Azure DC 的 AlwaysOn 同步更改为同步，然后故障转移到该 SQL Server。然后将 VHD 复制到一个高级存储帐户，并将计算机重新部署到新的云服务中。更新侦听器，然后故障恢复。

##### 停机时间点

停机时间包含故障转移到备用 DC 并返回的时间。它还取决于你的客户端/DNS 配置，并且你的客户端重新连接可能会延迟。请考虑以下混合 AlwaysOn 配置的示例：

![MultiSite1][9]

##### 优点

- 你可以利用现有基础结构。
- 可以选择先在 DR Azure DC 上预升级 Azure 存储。
- 可以重新配置 DR Azure DC 存储。
- 在迁移期间，至少进行两次故障转移，不包括测试故障转移。
- 无法使用备份和还原即可移动 SQL Server 数据。

##### 缺点

- 根据客户端对 SQL Server 的访问权限，当 SQL Server 在应用程序的备用 DC 中运行时，延迟时间可能会增加。
- VHD 到高级存储的复制时间可能会很长。这可能会影响是否要在可用性组中保留节点的决策。在所需的迁移过程中，运行日志密集型工作负载时请考虑这一点，因为主节点将必须在其事务日志中保留未复制的事务。因此，此日志可能会显著增长。
- 此方案会使用 Azure **Start-AzureStorageBlobCopy** commandlet，它是异步的。完成后没有 SLA。复制时间各不相同，这取决于在队列中等待的时间，还取决于要传输的数据量。因此，你在第 2 个数据中心中只有一个节点，在复制所用时间长于测试的情况下，你应该采取缓解措施。这可以包括以下建议。
	- 在使用商定的停机时间进行迁移之前，添加临时的第 2 个 SQL 节点以实现 HA。
	- 在 Azure 计划的维护之外运行迁移。
	- 确保已正确配置群集仲裁。

此方案假定你已记录安装，并知道如何存储映射以便对最佳磁盘缓存设置进行更改。

##### 大致步骤
![Multisite2][10]

- 将本地/备用 Azure DC 设为主 SQL Server，并将设为其他自动故障转移伙伴 (AFP)。
- 从 SQL2 收集磁盘配置信息并删除该节点（不删除附加的 VHD）。
- 创建高级存储帐户并从标准存储帐户复制 VHD。
- 创建新的云服务，并使用其附加高级存储磁盘创建 SQL2 VM。
- 配置 ILB/ELB 并添加终结点。
- 使用新的 ILB/ELB IP 地址更新 AlwaysOn 侦听器并测试故障转移。
- 检查 DNS 配置。
- 将 AFP 更改为 SQL2，然后迁移 SQL1 并完成步骤 2 - 5。
- 测试故障转移。
- 将 AFP 切换回 SQL1 和 SQL2

##<a name="appendix-migrating-a-multisite-alwayson-cluster-to-premium-storage"></a> 附录：将多站点 AlwaysOn 群集迁移到高级存储

本主题的剩余部分提供将多站点 AlwaysOn 群集转换为高级存储的详细示例。它还将侦听器从使用外部负载平衡器 (ELB) 转换为使用内部负载平衡器 (ILB)。

### 环境

- Windows 2k12/SQL 2k12
- SP 上的 1 个 DB 文件
- 2 x 每个节点的存储池

![Appendix1][11]

### VM：

在此示例中，我们将演示从 ELB 移到 ILB。ELB 是在 ILB 以前提供的，因此此示例演示如何在迁移期间切换到 ILB。

![Appendix2][12]

### 预先步骤：连接到订阅

    Add-AzureAccount -Environment AzureChinaCloud

    #Set up subscription
    Get-AzureSubscription -Environment AzureChinaCloud

#### 步骤 1：创建新的存储帐户和云服务
    $mysubscription = "DansSubscription"
    $location = "West Europe"

    #Storage accounts
    #current storage account where the vm to migrate resides
    $origstorageaccountname = "danstdams"

    #Create Premium Storage account
    $newxiostorageaccountname = "danspremsams"
    New-AzureStorageAccount -StorageAccountName $newxiostorageaccountname -Location $location -Type "Premium_LRS"  

    #Generate storage keys for later
    $originalstorage =  Get-AzureStorageKey -StorageAccountName $origstorageaccountname
    $xiostorage = Get-AzureStorageKey -StorageAccountName $newxiostorageaccountname

    #Generate storage acc contexts
    $origContext = New-AzureStorageContext -Environment AzureChinaCloud -StorageAccountName $origstorageaccountname -StorageAccountKey $originalstorage.Primary
    $xioContext = New-AzureStorageContext -Environment AzureChinaCloud -StorageAccountName $newxiostorageaccountname -StorageAccountKey $xiostorage.Primary  

    #Set up subscription and default storage account
    Set-AzureSubscription -Environment AzureChinaCloud -SubscriptionName $mysubscription -CurrentStorageAccount $origstorageaccountname
    Select-AzureSubscription -Environment AzureChinaCloud -SubscriptionName $mysubscription -Current

    #CREATE NEW CLOUD SVC
    $vnet = "dansvnetwesteur"

    ##Existing cloud service
    $sourceSvc="dansolSrcAms"

    ##Create new cloud service
    $destcloudsvc = "danNewSvcAms"
    New-AzureService $destcloudsvc -Location $location

#### 步骤 2：在资源上增加允许的故障 <Optional>
在某些属于 AlwaysOn 可用性组的资源上，对于在某一期间可以发生的故障数有限制，超过该限制后，群集服务将尝试重新启动该资源组。在完成此过程时，建议你增加此限制，因为如果你未手动故障转移或通过关闭计算机来触发故障转移，则可能会接近此限制。

明智的做法是，使允许的故障数加倍，为此，请在故障转移群集管理器中，转到 AlwaysOn 资源组的属性：

![Appendix3][13]

将最大故障数更改为 6。

#### 步骤 3：为群集组添加 IP 地址资源 <Optional>

如果你为群集组只提供一个 IP 地址，并且已根据云子网调整此地址，请注意，如果你意外使云中在该网络上的所有群集节点脱机，则该群集 IP 资源和群集网络名称将无法联机。如果发生这种情况，将阻止对其他群集资源进行更新。

#### 步骤 4：DNS 配置

能否实现平稳过渡取决于如何利用和更新 DNS。安装 AlwaysOn 时，它将创建一个 Windows 群集资源组，如果你打开故障转移群集管理器，你将看到它将至少包含三个资源，其中本文档引用的两个资源是：

- 虚拟网络名称 (VNN) - 这是客户端想要通过 AlwaysOn 连接到 SQL Server 时要连接的 DNS 名称。
- IP 地址资源 - 这是与 VNN 关联的 IP 地址，你可以具有多个 IP 地址，在多站点配置中，你将针对每个站点/子网有一个 IP 地址。

连接到 SQL Sevrer 时，SQL Server 客户端驱动程序将检索与侦听器关联的 DNS 记录，并尝试连接到每个 AlwaysOn 关联的 IP 地址，下面我们将讨论可能会影响此项的一些因素。

与侦听器名称关联的并发 DNS 记录数不仅取决于关联的 IP 地址数，而且还取决于 AlwaysON VNN 资源的故障转移群集的“RegisterAllIpProviders”设置。

在 Azure 中部署 AlwaysOn 时有不同的步骤可用于创建侦听器和 IP 地址，你必须手动将“RegisterAllIpProviders”配置为 1，这与本地 AlwaysOn 部署不同，后者已设置为 1。

如果“RegisterAllIpProviders”为 0，则你将在与侦听器关联的 DNS 中只看到一条 DNS 记录：

![Appendix4][14]

如果“RegisterAllIpProviders”为 1：

![Appendix5][15]

以下代码将转储 VNN 设置并为你设置它，请注意，要使更改生效，需要将 VNN 脱机，然后重新联机，这种使侦听器脱机会导致客户端连接中断。

    ##Always On Listener Name
    $ListenerName = "Mylistener"
    ##Get AlwaysOn Network Name Settings
    Get-ClusterResource $ListenerName| Get-ClusterParameter
    ##Set RegisterAllProvidersIP
    Get-ClusterResource $ListenerName| Set-ClusterParameter RegisterAllProvidersIP  1

在后面的迁移步骤中，你将需要使用引用负载平衡器的已更新 IP 地址更新 AlwaysOn 侦听器，这将涉及删除和添加 IP 地址资源。更新 IP 之后，你需要确保已在 DNS 区域中更新新的 IP 地址并且客户端将更新其本地 DNS 缓存。

如果你的客户端驻留在不同网络段，并引用不同的 DNS 服务器，则你需要考虑在迁移期间将发生哪些与 DNS 区域传送相关的事件，因为应用程序重新连接时间将至少受到侦听器的任何新 IP 地址的区域传送时间的约束。如果你在此处受到时间约束，则应与 Windows 团队讨论并测试强制增量区域传送，同时还应将 DNS 主机记录设为较小的生存时间 (TTL)，以使客户端更新。有关详细信息，请参阅[增量区域传送](https://technet.microsoft.com/zh-cn/library/cc958973.aspx)和 [Start-DnsServerZoneTransfer](https://technet.microsoft.com/zh-cn/library/jj649917.aspx)。

默认情况下，与 Azure 中 AlwaysOn 的侦听器关联的 DNS 记录的 TTL 为 1200 秒。如果你在迁移期间受时间约束，你可能希望减少此时间，以确保客户端使用侦听器更新后的 IP 地址更新其 DNS。你可以通过转储 VNN 的配置来查看并修改该配置：

    $AGName = "myProductionAG"
    $ListenerName = "Mylistener"
    #Look at HostRecordTTL
    Get-ClusterResource $ListenerName| Get-ClusterParameter

    #Set HostRecordTTL Examples
    Get-ClusterResource $ListenerName| Set-ClusterParameter -Name "HostRecordTTL" 120

请注意，“HostRecordTTL”越低，将发生的 DNS 通信量越高。

##### 客户端应用程序设置

如果 SQL 客户端应用程序支持 .Net 4.5 SQLClient，则可以使用“MULTISUBNETFAILOVER=TRUE”关键字，建议你应用此项，因为这样可以在故障转移期间更快地连接到 SQL AlwaysOn 可用性组。它在故障转移过程中枚举与 AlwaysOn 侦听器并行关联的所有 IP 地址，并执行更激进的 TCP 连接重试速度。

有关上述设置的详细信息，请参阅 [MultiSubnetFailover 关键字和关联功能](https://msdn.microsoft.com/zh-cn/library/hh213080.aspx#MultiSubnetFailover)。另请参阅 [SqlClient 对高可用性、灾难恢复的支持](https://msdn.microsoft.com/zh-cn/library/hh205662(v=vs.110).aspx)。

#### 步骤 5：群集仲裁设置

由于你一次将要让至少一个 SQL Server 关闭，你应修改群集仲裁设置，如果要对 2 个节点使用文件共享见证 (FSW)，则应将仲裁设置为允许多数节点和利用动态投票，这样做是为了允许单个节点保持运转。


    Set-ClusterQuorum -NodeMajority  

有关管理和配置群集仲裁的详细信息，请参阅[配置和管理 Windows Server 2012 故障转移群集中的仲裁](https://technet.microsoft.com/zh-cn/library/jj612870.aspx)。

#### 步骤 6：提取现有终结点和 ACL
    #GET Endpoint info
    Get-AzureVM -ServiceName $destcloudsvc -Name $vmNameToMigrate | Get-AzureEndpoint
    #GET ACL Rules for Each EP, this example is for the AlwaysOn Endpoint
    Get-AzureVM -ServiceName $destcloudsvc -Name $vmNameToMigrate | Get-AzureAclConfig -EndpointName "myAOEndPoint-LB"  

将这些内容保存到文本文件中。

#### 步骤 7：更改故障转移伙伴和复制模式

如果你使用 2 个以上的 SQL Server，则应将另一个 DC 或本地中的另一个辅助副本的故障转移更改为“同步”，并将其设为“自动故障转移伙伴”(AFP)，这样你便可以在进行更改的同时维护高可用性。你可以通过修改 TSQL 来执行此操作（虽然 SSMS 也可以）：

![Appendix6][16]

#### 步骤 8：从云服务中删除辅助 VM

你应计划先迁移云辅助节点，如果这当前是主节点，则应启动手动故障转移。

    $vmNameToMigrate="dansqlams2"

    #Check machine status
    Get-AzureVM -ServiceName $sourceSvc -Name $vmNameToMigrate

    #Shutdown secondary VM
    Get-AzureVM -ServiceName $sourceSvc -Name $vmNameToMigrate | stop-AzureVM


    #Extract disk configuration

    ##Building Existing Data Disk Configuration
    $file = "C:\Azure Storage Testing\mydiskconfig_$vmNameToMigrate.csv"
    $datadisks = @(Get-AzureVM -ServiceName $sourceSvc -Name $vmNameToMigrate | Get-AzureDataDisk )
    Add-Content $file "lun, vhdname, hostcaching, disklabel, diskName"
    foreach ($disk in $datadisks)
    {
      $vhdname = $disk.MediaLink.AbsolutePath -creplace  "/vhds/"
      $disk.Lun, , $disk.HostCaching, $vhdname, $disk.DiskLabel,$disks.DiskName
    # Write-Host "copying disk $disk"
    $adddisk = "{0},{1},{2},{3},{4}" -f $disk.Lun,$vhdname, $disk.HostCaching, $disk.DiskLabel, $disk.DiskName
    $adddisk | add-content -path $file
    }

    #Get OS Disk
    $osdisks = Get-AzureVM -ServiceName $sourceSvc -Name $vmNameToMigrate | Get-AzureOSDisk ## | select -ExpandProperty MediaLink
    $osvhdname = $osdisks.MediaLink.AbsolutePath -creplace  "/vhds/"
    $osdisks.OS, $osdisks.HostCaching, $osvhdname, $osdisks.DiskLabel, $osdisks.DiskName
    $addosdisk = "{0},{1},{2},{3},{4}" -f $osdisks.OS,$osvhdname, $osdisks.HostCaching, $osdisks.Disklabel , $osdisks.DiskName
    $addosdisk | add-content -path $file

    #Import disk config
    $diskobjects  = Import-CSV $file

    #Check disk config, make sure below returns the disks associated with the VM
    $diskobjects

    #Identify OS Disk
    $osdiskimport = $diskobjects | where {$_.lun -eq "Windows"}
    $osdiskforbuild = $osdiskimport.diskName

    #Check machibe is off
    Get-AzureVM -ServiceName $sourceSvc -Name  $vmNameToMigrate

    #Drop machine and rebuild to new cls
    Remove-AzureVM -ServiceName $sourceSvc -Name $vmNameToMigrate

#### 步骤 9：更改 CSV 文件中的磁盘缓存设置并保存

对于数据卷，这些应设置为 READONLY。

对于 TLOG 卷，这些应设置为 NONE。

![Appendix7][17]

#### 步骤 10：复制 VHD
    #Ensure you have created the container for these:
    $containerName = 'vhds'

    #Create container
    New-AzureStorageContainer -Name $containerName -Context $xioContext

    ####DISK COPYING####
    #Get disks from csv, get settings for each VHDs and copy to Premium Storage accoun
    ForEach ($disk in $diskobjects)
       {
       $lun = $disk.Lun
       $vhdname = $disk.vhdname
       $cacheoption = $disk.HostCaching
       $disklabel = $disk.DiskLabel
       $diskName = $disk.DiskName
       Write-Host "Copying Disk Lun $lun, Label : $disklabel, VHD : $vhdname has cache setting : $cacheoption"

       #Start async copy
       Start-AzureStorageBlobCopy -srcUri "https://$origstorageaccountname.blob.core.chinacloudapi.cn/vhds/$vhdname" `
    -SrcContext $origContext `
    -DestContainer $containerName `
    -DestBlob $vhdname `
    -DestContext $xioContext
       }



可以检查高级存储帐户的 VHD 的复制状态：

    ForEach ($disk in $diskobjects)
       {
       $lun = $disk.Lun
       $vhdname = $disk.vhdname
       $cacheoption = $disk.HostCaching
       $disklabel = $disk.DiskLabel
       $diskName = $disk.DiskName

       $copystate = Get-AzureStorageBlobCopyState -Blob $vhdname -Container $containerName -Context $xioContext
    Write-Host "Copying Disk Lun $lun, Label : $disklabel, VHD : $vhdname, STATUS = " $copystate.Status
       }

![Appendix8][18]

等到所有这些状态都记录为成功。

如需单个 blob 的信息：

    Get-AzureStorageBlobCopyState -Blob "blobname.vhd" -Container $containerName -Context $xioContext

#### 步骤 11：注册 OS 磁盘

    #Change storage account
    Set-AzureSubscription -Environment AzureChinaCloud -SubscriptionName $mysubscription -CurrentStorageAccount $newxiostorageaccountname
    Select-AzureSubscription -Environment AzureChinaCloud -SubscriptionName $mysubscription -Current

    #Register OS disk
    $osdiskimport = $diskobjects | where {$_.lun -eq "Windows"}
    $osvhd = $osdiskimport.vhdname
    $osdiskforbuild = $osdiskimport.diskName

    #Registering OS disk, but as XIO disk
    $xioDiskName = $osdiskforbuild + "xio"
    Add-AzureDisk -DiskName $xioDiskName -MediaLocation  "https://$newxiostorageaccountname.blob.core.chinacloudapi.cn/vhds/$osvhd"  -Label "BootDisk" -OS "Windows"

#### 步骤 12：将辅助副本导入到新的云服务

以下代码还使用此处添加的选项，你可以导入计算机，并使用 可保留的 VIP。

    #Build VM Config
    $ipaddr = "192.168.0.5"
    #Remember to change to XIO
    $newInstanceSize = "Standard_DS13"
    $subnet = "SQL"

    #Create new Avaiability Set
    $availabilitySet = "cloudmigAVAMS"

    #build machine config into object
    $vmConfig = New-AzureVMConfig -Name $vmNameToMigrate -InstanceSize $newInstanceSize -DiskName $xioDiskName -AvailabilitySetName $availabilitySet  ` | Add-AzureProvisioningConfig -Windows ` | Set-AzureSubnet -SubnetNames $subnet | Set-AzureStaticVNetIP -IPAddress $ipaddr

    #Reload disk config
    $diskobjects  = Import-CSV $file
    $datadiskimport = $diskobjects | where {$_.lun -ne "Windows"}

    ForEach ( $attachdatadisk in $datadiskimport)
       {
    $label = $attachdatadisk.disklabel
    $lunNo = $attachdatadisk.lun
    $hostcach = $attachdatadisk.hostcaching
    $datadiskforbuild = $attachdatadisk.diskName
    $vhdname = $attachdatadisk.vhdname

    ###Attaching disks to a VM during a deploy to a new cloud service and new storage account is different from just attaching VHDs to just a redeploy in a new cloud service
    $vmConfig | Add-AzureDataDisk -ImportFrom -MediaLocation "https://$newxiostorageaccountname.blob.core.chinacloudapi.cn/vhds/$vhdname" -LUN $lunNo -HostCaching $hostcach -DiskLabel $label

    }

    #Create VM
    $vmConfig  | New-AzureVM -ServiceName $destcloudsvc -Location $location -VNetName $vnet ## Optional (-ReservedIPName $reservedVIPName)

#### 步骤 13：在新的云服务上创建 ILB，添加负载平衡终结点和 ACL
    #Check for existing ILB
    GET-AzureInternalLoadBalancer -ServiceName $destcloudsvc

    $ilb="sqlIntIlbDest"
    $subnet = "SQL"
    $IP="192.168.0.25"
    Add-AzureInternalLoadBalancer -ServiceName $destcloudsvc -InternalLoadBalancerName $ilb -SubnetName $subnet -StaticVNetIPAddress $IP

    #Endpoints
    $epname="sqlIntEP"
    $prot="tcp"
    $locport=1433
    $pubport=1433
    Get-AzureVM -ServiceName $destcloudsvc -Name $vmNameToMigrate  | Add-AzureEndpoint -Name $epname -Protocol $prot -LocalPort $locport -PublicPort $pubport -ProbePort 59999 -ProbeIntervalInSeconds 5 -ProbeTimeoutInSeconds 11  -ProbeProtocol "TCP" -InternalLoadBalancerName $ilb -LBSetName $ilb -DirectServerReturn $true | Update-AzureVM

    #SET Azure ACLs or Network Security Groups & Windows FWs

    #http://msdn.microsoft.com/zh-cn/library/azure/dn495192.aspx

    ####WAIT FOR FULL AlwaysOn RESYNCRONISATION!!!!!!!!!#####

####步骤 14：更新 AlwaysOn

    #Code to be executed on a Cluster Node
    $ClusterNetworkNameAmsterdam = "Cluster Network 2" # the azure cluster subnet network name
    $newCloudServiceIPAmsterdam = "192.168.0.25" # IP address of your cloud service

    $AGName = "myProductionAG"
    $ListenerName = "Mylistener"


    Add-ClusterResource "IP Address $newCloudServiceIPAmsterdam" -ResourceType "IP Address" -Group $AGName -ErrorAction Stop |  Set-ClusterParameter -Multiple @{"Address"="$newCloudServiceIPAmsterdam";"ProbePort"="59999";SubnetMask="255.255.255.255";"Network"=$ClusterNetworkNameAmsterdam;"OverrideAddressMatch"=1;"EnableDhcp"=0} -ErrorAction Stop

    #set dependancy and NETBIOS, then remove old IP address

    #set NETBIOS, then remove old IP address
    Get-ClusterGroup $AGName | Get-ClusterResource -Name "IP Address $newCloudServiceIPAmsterdam" | Set-ClusterParameter -Name EnableNetBIOS -Value 0

    #set dependency to Listener (OR Dependency) and delete previous IP Address resource that references:

    #Make sure no static records in DNS

![Appendix9][19]

现在，删除旧的云服务 IP 地址。

![Appendix10][20]

#### 步骤 15：DNS 更新检查

现在，你应检查 SQL Server 客户端网络上的 DNS 服务器，并确保群集已为添加的 IP 地址添加额外的主机记录。如果这些 DNS 服务器未更新，请考虑强制进行 DNS 区域复制并确保该子网中的客户端能够解析为这两个 AlwaysOn IP 地址，这就是为什么无需等待自动 DNS 复制的原因。

#### 步骤 16：重新配置 AlwaysOn

此时，你将等待已迁移的辅助节点与本地节点完全重新同步并切换到同步复制节点，然后将其设为 AFP。

#### 步骤 17：迁移辅助节点
    $vmNameToMigrate="dansqlams1"

    Get-AzureVM -ServiceName $sourceSvc -Name $vmNameToMigrate

    #Get endpoint information
    $endpoint = Get-AzureVM -ServiceName $sourceSvc  -Name $vmNameToMigrate | Get-AzureEndpoint

    #Shutdown VM
    Get-AzureVM -ServiceName $sourceSvc -Name $vmNameToMigrate | stop-AzureVM

    #Get disk config

    #Building Existing Data Disk Configuration
    $file = "C:\Azure Storage Testing\mydiskconfig_$vmNameToMigrate.csv"
    $datadisks = @(Get-AzureVM -ServiceName $sourceSvc -Name $vmNameToMigrate | Get-AzureDataDisk )
    Add-Content $file "lun, vhdname, hostcaching, disklabel, diskName"
    foreach ($disk in $datadisks)
    {
      $vhdname = $disk.MediaLink.AbsolutePath -creplace  "/vhds/"
      $disk.Lun, , $disk.HostCaching, $vhdname, $disk.DiskLabel,$disks.DiskName
    # Write-Host "copying disk $disk"
    $adddisk = "{0},{1},{2},{3},{4}" -f $disk.Lun,$vhdname, $disk.HostCaching, $disk.DiskLabel, $disk.DiskName
    $adddisk | add-content -path $file
    }

    #Get OS Disk
    $osdisks = Get-AzureVM -ServiceName $sourceSvc -Name $vmNameToMigrate | Get-AzureOSDisk ## | select -ExpandProperty MediaLink
    $osvhdname = $osdisks.MediaLink.AbsolutePath -creplace  "/vhds/"
    $osdisks.OS, $osdisks.HostCaching, $osvhdname, $osdisks.DiskLabel, $osdisks.DiskName
    $addosdisk = "{0},{1},{2},{3},{4}" -f $osdisks.OS,$osvhdname, $osdisks.HostCaching, $osdisks.Disklabel , $osdisks.DiskName
    $addosdisk | add-content -path $file

    #Import disk config
    $diskobjects  = Import-CSV $file

    #Check disk configuration
    $diskobjects

    #Identify OS Disk
    $osdiskimport = $diskobjects | where {$_.lun -eq "Windows"}
    $osdiskforbuild = $osdiskimport.diskName

    #Check machine is off
    Get-AzureVM -ServiceName $sourceSvc -Name  $vmNameToMigrate

    #Drop machine and rebuild to new cls
    Remove-AzureVM -ServiceName $sourceSvc -Name $vmNameToMigrate

#### 步骤 18：更改 CSV 文件中的磁盘缓存设置并保存

对于数据卷，这些应设置为 READONLY。

对于 TLOG 卷，这些应设置为 NONE。

![Appendix11][21]

#### 步骤 19：为辅助节点创建新的独立存储帐户
    $newxiostorageaccountnamenode2 = "danspremsams2"
    New-AzureStorageAccount -StorageAccountName $newxiostorageaccountnamenode2 -Location $location -Type "Premium_LRS"  

    #Reset the storage account src if node 1 in a different storage account
    $origstorageaccountname2nd = "danstdams2"

    #Generate storage keys for later
    $xiostoragenode2 = Get-AzureStorageKey -StorageAccountName $newxiostorageaccountnamenode2

    #Generate storage acc contexts
    $xioContextnode2 = New-AzureStorageContext -Environment AzureChinaCloud -StorageAccountName $newxiostorageaccountnamenode2 -StorageAccountKey $xiostoragenode2.Primary  

    #Set up subscription and default storage account
    Set-AzureSubscription -Environment AzureChinaCloud -SubscriptionName $mysubscription -CurrentStorageAccount $newxiostorageaccountnamenode2
    Select-AzureSubscription -Environment AzureChinaCloud -SubscriptionName $mysubscription -Current

#### 步骤 20：复制 VHD
    #Ensure you have created the container for these:
    $containerName = 'vhds'

    #Create container
    New-AzureStorageContainer -Name $containerName -Context $xioContextnode2  

    ####DISK COPYING####
    ##get disks from csv, get settings for each VHDs and copy to Premium Storage accoun
    ForEach ($disk in $diskobjects)
       {
       $lun = $disk.Lun
       $vhdname = $disk.vhdname
       $cacheoption = $disk.HostCaching
       $disklabel = $disk.DiskLabel
       $diskName = $disk.DiskName
       Write-Host "Copying Disk Lun $lun, Label : $disklabel, VHD : $vhdname has cache setting : $cacheoption"

       #Start async copy
       Start-AzureStorageBlobCopy -srcUri "https://$origstorageaccountname2nd.blob.core.chinacloudapi.cn/vhds/$vhdname" `
	    -SrcContext $origContext `
	    -DestContainer $containerName `
	    -DestBlob $vhdname `
	    -DestContext $xioContextnode2
       }

    #Check for copy progress

    #check induvidual blob status
    Get-AzureStorageBlobCopyState -Blob "danRegSvcAms-dansqlams1-2014-07-03.vhd" -Container $containerName -Context $xioContext


你可以检查所有 VHD 的 VHD 复制状态：
    ForEach ($disk in $diskobjects)
       {
       $lun = $disk.Lun
       $vhdname = $disk.vhdname
       $cacheoption = $disk.HostCaching
       $disklabel = $disk.DiskLabel
       $diskName = $disk.DiskName

       $copystate = Get-AzureStorageBlobCopyState -Blob $vhdname -Container $containerName -Context $xioContextnode2
    Write-Host "Copying Disk Lun $lun, Label : $disklabel, VHD : $vhdname, STATUS = " $copystate.Status
       }

![Appendix12][22]

等到所有这些状态都记录为成功。

如需单个 blob 的信息：
    #Check induvidual blob status
    Get-AzureStorageBlobCopyState -Blob "danRegSvcAms-dansqlams1-2014-07-03.vhd" -Container $containerName -Context $xioContextnode2

#### 步骤 21：注册 OS 磁盘
    #change storage account to the new XIO storage account
    Set-AzureSubscription -Environment AzureChinaCloud -SubscriptionName $mysubscription -CurrentStorageAccount $newxiostorageaccountnamenode2
    Select-AzureSubscription -Environment AzureChinaCloud -SubscriptionName $mysubscription -Current

    #Register OS disk
    $osdiskimport = $diskobjects | where {$_.lun -eq "Windows"}
    $osvhd = $osdiskimport.vhdname
    $osdiskforbuild = $osdiskimport.diskName

    #Registering OS disk, but as XIO disk
    $xioDiskName = $osdiskforbuild + "xio"
    Add-AzureDisk -DiskName $xioDiskName -MediaLocation  "https://$newxiostorageaccountnamenode2.blob.core.chinacloudapi.cn/vhds/$osvhd"  -Label "BootDisk" -OS "Windows"

    #Build VM Config
    $ipaddr = "192.168.0.4"
    $newInstanceSize = "Standard_DS13"

    #Join to existing Avaiability Set

    #Build machine config into object
    $vmConfig = New-AzureVMConfig -Name $vmNameToMigrate -InstanceSize $newInstanceSize -DiskName $xioDiskName -AvailabilitySetName $availabilitySet  ` | Add-AzureProvisioningConfig -Windows ` | Set-AzureSubnet -SubnetNames $subnet | Set-AzureStaticVNetIP -IPAddress $ipaddr

    #Reload disk config
    $diskobjects  = Import-CSV $file
    $datadiskimport = $diskobjects | where {$_.lun -ne "Windows"}

    ForEach ( $attachdatadisk in $datadiskimport)
       {
    $label = $attachdatadisk.disklabel
    $lunNo = $attachdatadisk.lun
    $hostcach = $attachdatadisk.hostcaching
    $datadiskforbuild = $attachdatadisk.diskName
    $vhdname = $attachdatadisk.vhdname

    ###This is different to just a straight cloud service change
    #note if you do not have a disk label the command below will fail, populate as required.
    $vmConfig | Add-AzureDataDisk -ImportFrom -MediaLocation "https://$newxiostorageaccountnamenode2.blob.core.chinacloudapi.cn/vhds/$vhdname" -LUN $lunNo -HostCaching $hostcach -DiskLabel $label

    }

    #Create VM
    $vmConfig  | New-AzureVM -ServiceName $destcloudsvc -Location $location -VNetName $vnet -Verbose

#### 步骤 22：添加负载平衡终结点和 ACL
    #Endpoints
    $epname="sqlIntEP"
    $prot="tcp"
    $locport=1433
    $pubport=1433
    Get-AzureVM -ServiceName $destcloudsvc -Name $vmNameToMigrate  | Add-AzureEndpoint -Name $epname -Protocol $prot -LocalPort $locport -PublicPort $pubport -ProbePort 59999 -ProbeIntervalInSeconds 5 -ProbeTimeoutInSeconds 11  -ProbeProtocol "TCP" -InternalLoadBalancerName $ilb -LBSetName $ilb -DirectServerReturn $true | Update-AzureVM


    #STOP!!! CHECK in the Azure Management Portal or Machine Endpoints through powershell that these Endpoints are created!

    #SET ACLs or Azure Network Security Groups & Windows FWs

    #http://msdn.microsoft.com/zh-cn/library/azure/dn495192.aspx

#### 步骤 23：测试故障转移

现在，应让已迁移的节点与本地 AlwaysOn 节点同步，将其置于同步复制模式，并等到它已同步。然后从本地故障转移到第一个已迁移的节点（即 AFP）。故障转移完成后，将最后一个已迁移的节点更改为 AFP。

你应测试所有节点之间的故障转移，并运行所有混沌测试，以确保故障转移及时地正常工作。

#### 步骤 24：置回群集仲裁设置/DNS TTL/故障转移伙伴/同步设置
##### 在同一子网上添加 IP 地址资源

如果你只有 2 个 SQL Server 并且想要将它们迁移到新的云服务，但想要将它们保留在同一子网中，则可以避免使侦听器脱机即可删除原始 AlwaysOn IP 地址，然后添加新的 IP 地址。如果要将 VM 迁移到另一个子网，则不需要执行此操作，因为将会有一个附加群集网络会引用该子网。

启动已迁移的辅助节点并在故障转移现有的主节点之前将其添加到新云服务的新 IP 地址资源中后，你应在群集故障转移管理器执行以下步骤：

若要添加 IP 地址，请参阅[附录](#appendix-migrating-a-multisite-alwayson-cluster-to-premium-storage)步骤 14。

1. 对于当前的 IP 地址资源，将可能的所有者更改为“现有的主 SQL Server”（在以下示例中为“dansqlams4”）：

	![Appendix13][23]

1. 对于新的 IP 地址资源，将可能的所有者更改为“已迁移的辅助 SQL Server”（在以下示例中为“dansqlams5”）：

	![Appendix14][24]

1. 设置了此项后便可以进行故障转移了，迁移了最后一个节点后，应编辑“可能的所有者”，以便将该节点添加为可能的所有者：

	![Appendix15][25]

## 其他资源
- [Azure 高级存储](/documentation/articles/storage-premium-storage/)
- [虚拟机](/home/features/virtual-machines/)
- [Azure 虚拟机中的 SQL Server](/documentation/articles/virtual-machines-windows-sql-server-iaas-overview/)

<!-- IMAGES -->
[1]: ./media/virtual-machines-windows-classic-sql-server-premium-storage/1_VNET_Portal.png
[2]: ./media/virtual-machines-windows-classic-sql-server-premium-storage/2_Diskname_Lun.png
[3]: ./media/virtual-machines-windows-classic-sql-server-premium-storage/3_Virtual_Disk_Properties.png
[4]: ./media/virtual-machines-windows-classic-sql-server-premium-storage/4_Virtual_Disk_Properties_Details.png
[5]: ./media/virtual-machines-windows-classic-sql-server-premium-storage/5_Get_Storage_Pool.png
[6]: ./media/virtual-machines-windows-classic-sql-server-premium-storage/6_Deployments_Always_On.png
[7]: ./media/virtual-machines-windows-classic-sql-server-premium-storage/7_Add_More_Secondaries.png
[8]: ./media/virtual-machines-windows-classic-sql-server-premium-storage/8_Use_Existing_Secondary_Single_Site.png
[9]: ./media/virtual-machines-windows-classic-sql-server-premium-storage/9_Use_Existing_Secondary_Multi_Site.png
[10]: ./media/virtual-machines-windows-classic-sql-server-premium-storage/9_Use_Existing_Secondary_Multi_Site_b.png
[11]: ./media/virtual-machines-windows-classic-sql-server-premium-storage/10_Appendix_01.png
[12]: ./media/virtual-machines-windows-classic-sql-server-premium-storage/10_Appendix_02.png
[13]: ./media/virtual-machines-windows-classic-sql-server-premium-storage/10_Appendix_03.png
[14]: ./media/virtual-machines-windows-classic-sql-server-premium-storage/10_Appendix_04.png
[15]: ./media/virtual-machines-windows-classic-sql-server-premium-storage/10_Appendix_05.png
[16]: ./media/virtual-machines-windows-classic-sql-server-premium-storage/10_Appendix_06.png
[17]: ./media/virtual-machines-windows-classic-sql-server-premium-storage/10_Appendix_07.png
[18]: ./media/virtual-machines-windows-classic-sql-server-premium-storage/10_Appendix_08.png
[19]: ./media/virtual-machines-windows-classic-sql-server-premium-storage/10_Appendix_09.png
[20]: ./media/virtual-machines-windows-classic-sql-server-premium-storage/10_Appendix_10.png
[21]: ./media/virtual-machines-windows-classic-sql-server-premium-storage/10_Appendix_11.png
[22]: ./media/virtual-machines-windows-classic-sql-server-premium-storage/10_Appendix_12.png
[23]: ./media/virtual-machines-windows-classic-sql-server-premium-storage/10_Appendix_13.png
[24]: ./media/virtual-machines-windows-classic-sql-server-premium-storage/10_Appendix_14.png
[25]: ./media/virtual-machines-windows-classic-sql-server-premium-storage/10_Appendix_15.png

<!---HONumber=Mooncake_0321_2016-->