<properties
    pageTitle="将专用 VHD 上载到 Azure，用于创建新的 VM | Azure"
    description="将专用 VHD 上载到 Azure，用于创建新的 VM。"
    services="virtual-machines-windows"
    documentationcenter=""
    author="cynthn"
    manager="timlt"
    editor=""
    tags="azure-resource-manager" />
<tags
    ms.assetid=""
    ms.service="virtual-machines-windows"
    ms.workload="infrastructure-services"
    ms.tgt_pltfrm="vm-windows"
    ms.devlang="na"
    ms.topic="article"
    ms.date="02/05/2017"
    wacn.date="03/20/2017"
    ms.author="cynthn" />  


# 将专用 VHD 上载到 Azure，用于创建新的 VM

专用 VHD 保留原始 VM 中的用户帐户、应用程序和其他状态数据。可以将专用 VHD 上载到 Azure，用其创建一个 VM，以便使用非托管存储帐户。

>[AZURE.NOTE] Azure 中国区尚无法使用 Azure 托管磁盘。

> [AZURE.IMPORTANT]
将任何 VHD 上载到 Azure 之前，应该遵循[准备要上载到 Azure 的 Windows VHD 或 VHDX](/documentation/articles/virtual-machines-windows-prepare-for-upload-vhd-image/) 中的步骤操作
>
>

* 有关不同 VM 大小的定价信息，请参阅[虚拟机定价](/pricing/details/virtual-machines/#Windows)。
* 有关存储定价的信息，请参阅[存储定价](/pricing/details/storage/blob/)。
* 如需了解 Azure 区域中各种 VM 大小的可用性，请参阅[可用产品（按区域）](https://azure.microsoft.com/regions/services/)。
* 若要查看 Azure VM 的一般限制，请参阅 [Azure 订阅和服务限制、配额和约束](/documentation/articles/azure-subscription-service-limits/)。

## 开始之前
如果使用 PowerShell，请确保使用的是最新版本的 AzureRM.Compute PowerShell 模块。运行以下命令来安装该模块。

    Install-Module AzureRM.Compute -RequiredVersion 2.6.0

有关详细信息，请参阅 [Azure PowerShell Versioning](https://docs.microsoft.com/powershell/azureps-cmdlets-docs/#azure-powershell-versioning)（Azure PowerShell 版本控制）。

## 准备 VM
 
如果想要原封不动地使用专用 VHD 创建新 VM，请确保完成以下步骤。

  * [准备好要上传到 Azure 的 Windows VHD](/documentation/articles/virtual-machines-windows-prepare-for-upload-vhd-image/)。**不要**使用 Sysprep 通用化 VM。
  * 删除安装于 VM 上的任何来宾虚拟化工具和代理，即 VMware 工具。
  * 确保 VM 配置为通过 DHCP 来提取其 IP 地址和 DNS 设置。这确保服务器在启动时在 VNet 中获取 IP 地址。
  * 在继续操作之前，请关闭 VM。

## 登录到 Azure
如果尚未安装 Azure PowerShell 1.4 版或更高版本，请阅读[如何安装和配置 Azure PowerShell](https://docs.microsoft.com/powershell/azureps-cmdlets-docs)。

1. 打开 Azure PowerShell 并登录到 Azure 帐户。将打开一个弹出窗口，可以输入 Azure 帐户凭据。

        Login-AzureRmAccount -EnvironmentName AzureChinaCloud

2. 获取可用订阅的订阅 ID。

        Get-AzureRmSubscription

3. 使用订阅 ID 设置正确的订阅。将 `<subscriptionID>` 替换为正确订阅的 ID。

        Select-AzureRmSubscription -SubscriptionId "<subscriptionID>"

## 获取存储帐户
需要在 Azure 中创建存储帐户来存储上传的 VM 映像。可以使用现有存储帐户，也可以创建新存储帐户。

显示可用的存储帐户，请键入：

    Get-AzureRmStorageAccount

如果要使用现有存储帐户，请转到[上载 VM 映像](#upload-the-vm-vhd-to-your-storage-account)部分。

若要创建存储帐户，请执行以下步骤：

1. 需要应在其中创建存储帐户的资源组的名称。若要查找订阅中的所有资源组，请键入：

        Get-AzureRmResourceGroup

    若要在**中国北部**区域中创建名称为 **myResourceGroup** 的资源组，请键入：

        New-AzureRmResourceGroup -Name myResourceGroup -Location "China North"

2. 使用 [New-AzureRmStorageAccount](https://msdn.microsoft.com/zh-cn/library/mt607148.aspx) cmdlet 在此资源组中创建名称为 **mystorageaccount** 存储帐户：

        New-AzureRmStorageAccount -ResourceGroupName myResourceGroup -Name mystorageaccount -Location "China North" `
            -SkuName "Standard_LRS" -Kind "Storage"

    -SkuName 的有效值为：
   
    * **Standard\_LRS** - 本地冗余存储。
    * **Standard\_ZRS** - 区域冗余存储。
    * **Standard\_GRS** - 异地冗余存储。
    * **Standard\_RAGRS** - 读取访问权限异地冗余存储。
    * **Premium\_LRS** - 高级本地冗余存储。

## <a name="upload-the-vm-vhd-to-your-storage-account"></a> 将 VHD 上传到存储帐户

使用 [Add-AzureRmVhd](https://msdn.microsoft.com/zh-cn/library/mt603554.aspx) cmdlet 将 VHD 上载到存储帐户中的容器。此示例将文件 **myVHD.vhd** 从 `"C:\Users\Public\Documents\Virtual hard disks"` 上传到 **myResourceGroup** 资源组中名为 **mystorageaccount** 的存储帐户。该文件将放到名为 **mycontainer** 的容器，新的文件名将是 **myUploadedVHD.vhd**。

    $rgName = "myResourceGroup"
    $urlOfUploadedImageVhd = "https://mystorageaccount.blob.core.chinacloudapi.cn/mycontainer/myUploadedVHD.vhd"
    Add-AzureRmVhd -ResourceGroupName $rgName -Destination $urlOfUploadedImageVhd `
        -LocalFilePath "C:\Users\Public\Documents\Virtual hard disks\myVHD.vhd"

如果成功，显示如下所示的响应：

    MD5 hash is being calculated for the file C:\Users\Public\Documents\Virtual hard disks\myVHD.vhd.
    MD5 hash calculation is completed.
    Elapsed time for the operation: 00:03:35
    Creating new page blob of size 53687091712...
    Elapsed time for upload: 01:12:49

    LocalFilePath           DestinationUri
    -------------           --------------
    C:\Users\Public\Doc...  https://mystorageaccount.blob.core.chinacloudapi.cn/mycontainer/myUploadedVHD.vhd

完成执行此命令可能需要一段时间，具体取决于网络连接速度和 VHD 文件的大小

### 用于上传 VHD 的其他选项

也可以使用以下方法之一将 VHD 上载到存储帐户：

-   [Azure 存储复制 Blob API](https://msdn.microsoft.com/zh-cn/library/azure/dd894037.aspx)

-   [Azure 存储资源管理器上传 Blob](https://azurestorageexplorer.codeplex.com/)

-   [存储导入/导出服务 REST API 参考](https://docs.microsoft.com/zh-cn/rest/api/storageimportexport/)

	如果预估上传时间大于 7 天，建议使用导入/导出服务。可根据数据大小和传输单位，利用 [DataTransferSpeedCalculator](https://github.com/Azure-Samples/storage-dotnet-import-export-job-management/blob/master/DataTransferSpeedCalculator.html) 预估时间。

	导入/导出可用于复制到标准存储帐户。需要使用 AzCopy 等工具从标准存储复制到高级存储帐户。

## 创建子网和 vNet

创建[虚拟网络](/documentation/articles/virtual-networks-overview/)的 vNet 和子网。

1. 创建子网。此示例在资源组 **myResourceGroup** 中创建名为 **mySubNet** 的子网，并将子网地址前缀设置为 **10.0.0.0/24**。

        $subnetName = "mySubNet"
        $singleSubnet = New-AzureRmVirtualNetworkSubnetConfig -Name $subnetName -AddressPrefix 10.0.0.0/24

2. 创建 vNet。此示例将虚拟网络名称设置为 **myVnetName**，将位置设置为**中国北部**，将虚拟网络的地址前缀设置为 **10.0.0.0/16**。

        $location = "China North"
        $vnetName = "myVnetName"
        $vnet = New-AzureRmVirtualNetwork -Name $vnetName -ResourceGroupName $rgName -Location $location `
            -AddressPrefix 10.0.0.0/16 -Subnet $singleSubnet

## 创建公共 IP 地址和 NIC
若要与虚拟网络中的虚拟机通信，需要一个[公共 IP 地址](/documentation/articles/virtual-network-ip-addresses-overview-arm/)和网络接口。

1. 创建公共 IP。在此示例中，公共 IP 地址名称设置为 **myIP**。

        $ipName = "myIP"
        $pip = New-AzureRmPublicIpAddress -Name $ipName -ResourceGroupName $rgName -Location $location `
            -AllocationMethod Dynamic

2. 创建 NIC。在此示例中，NIC 名称设置为 **myNicName**。

        $nicName = "myNicName"
        $nic = New-AzureRmNetworkInterface -Name $nicName -ResourceGroupName $rgName `
        -Location $location -SubnetId $vnet.Subnets[0].Id -PublicIpAddressId $pip.Id

## 创建网络安全组和 RDP 规则
若要使用 RDP 登录到 VM，需要创建一个允许在端口 3389 上进行 RDP 访问的安全规则。由于新 VM 的 VHD 是从现有专用 VM 创建的，因此在创建 VM 后，可以使用源虚拟机中有权通过 RDP 登录的现有帐户。

此示例将 NSG 名称设置为 **myNsg**，将 RDP 规则名称设置为 **myRdpRule**。

    $nsgName = "myNsg"

    $rdpRule = New-AzureRmNetworkSecurityRuleConfig -Name myRdpRule -Description "Allow RDP" `
        -Access Allow -Protocol Tcp -Direction Inbound -Priority 110 `
        -SourceAddressPrefix Internet -SourcePortRange * `
        -DestinationAddressPrefix * -DestinationPortRange 3389

    $nsg = New-AzureRmNetworkSecurityGroup -ResourceGroupName $rgName -Location $location `
        -Name $nsgName -SecurityRules $rdpRule
    
有关终结点和 NSG 规则的详细信息，请参阅 [Opening ports to a VM in Azure using PowerShell](/documentation/articles/virtual-machines-windows-nsg-quickstart-powershell/)（使用 PowerShell 在 Azure 中打开 VM 端口）。

## 设置 VM 名称和大小

此示例将 VM 名称设置为“myVM”，将 VM 大小设置为“Standard\_A2”。

    $vmName = "myVM"
    $vmConfig = New-AzureRmVMConfig -VMName $vmName -VMSize "Standard_A2"

## 添加 NIC

    $vm = Add-AzureRmVMNetworkInterface -VM $vmConfig -Id $nic.Id

## 配置 OS 磁盘

专用 OS 可以是[上载到 Azure](/documentation/articles/virtual-machines-windows-upload-image/)的 VHD 或[现有 Azure VM 的 VHD 副本](/documentation/articles/virtual-machines-windows-vhd-copy/)。

使用你自己的存储帐户中存储的专用 VHD（非托管磁盘）。

>[AZURE.NOTE] Azure 中国区尚无法使用 Azure 托管磁盘。

### 附加现有存储帐户中的 VHD

1. 设置要使用的 VHD 的 URI。在此示例中，名为 **myOsDisk.vhd** 的 VHD 文件保留在名为 **myContainer** 的容器中名为 **myStorageAccount** 的存储帐户中。

        $osDiskUri = $urlOfUploadedImageVhd

2. 使用复制的 OS VHD 的 URL 添加 OS 磁盘。在此示例中，创建 OS 磁盘时，“osDisk”一词将追加到用于创建 OS 磁盘名称的 VM 名称后。此示例还指定应将此基于 Windows 的 VHD 附加到 VM 作为 OS 磁盘。

        $osDiskName = $vmName + "osDisk"
        $vm = Set-AzureRmVMOSDisk -VM $vm -Name $osDiskName -VhdUri $osDiskUri -CreateOption attach -Windows

可选：如果有需要附加到 VM 的数据磁盘，请使用数据 VHD 的 URL 和相应逻辑单元号 (Lun) 添加数据磁盘。

    $dataDiskName = $vmName + "dataDisk"
    $vm = Add-AzureRmVMDataDisk -VM $vm -Name $dataDiskName -VhdUri $dataDiskUri -Lun 1 -CreateOption attach

使用存储帐户时，数据和操作系统磁盘 URL 如下所示：`https://StorageAccountName.blob.core.chinacloudapi.cn/BlobContainerName/DiskName.vhd`。可通过以下方法在门户上找到此信息：浏览到目标存储容器，单击复制的操作系统或数据 VHD，然后复制 URL 的内容。

## 创建 VM

使用刚刚创建的配置创建 VM。

    New-AzureRmVM -ResourceGroupName $rgName -Location $location -VM $vm

如果此命令成功，你将看到类似于下面的输出：

    RequestId IsSuccessStatusCode StatusCode ReasonPhrase
    --------- ------------------- ---------- ------------
                             True         OK OK   

## 验证是否已创建 VM
应会在 [Azure 门户预览](https://portal.azure.cn)的“浏览”>“虚拟机”下看到新建的 VM，也可以使用以下 PowerShell 命令查看该 VM：

    $vmList = Get-AzureRmVM -ResourceGroupName $rgName
    $vmList.Name

## 后续步骤
若要登录到新虚拟机，请在[门户](https://portal.azure.cn)中浏览到该 VM，单击“连接”，然后打开远程桌面 RDP 文件。使用原始虚拟机的帐户凭据登录到新虚拟机。有关详细信息，请参阅 [How to connect and log on to an Azure virtual machine running Windows](/documentation/articles/virtual-machines-windows-connect-logon/)（如何连接并登录到运行 Windows 的 Azure 虚拟机）。

<!---HONumber=Mooncake_0313_2017-->