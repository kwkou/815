<properties
    pageTitle="从 Azure 中的专用磁盘创建 VM | Azure"
    description="在 Resource Manager 部署模型中，通过附加专用非托管磁盘来创建新 VM。"
    services="virtual-machines-windows"
    documentationcenter=""
    author="cynthn"
    manager="timlt"
    editor=""
    tags="azure-resource-manager"
    translationtype="Human Translation" />
<tags
    ms.assetid="3b7d3cd5-e3d7-4041-a2a7-0290447458ea"
    ms.service="virtual-machines-windows"
    ms.workload="infrastructure-services"
    ms.tgt_pltfrm="vm-windows"
    ms.devlang="na"
    ms.topic="article"
    ms.date="02/06/2017"
    wacn.date="04/24/2017"
    ms.author="cynthn"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="d73e07821c60f5cebf38746ccc47a32a6c000504"
    ms.lasthandoff="04/14/2017" />

# <a name="create-a-vm-from-a-specialized-disk"></a>从专用磁盘创建 VM

通过使用 Powershell 将专用磁盘附加为 OS 磁盘来创建新 VM。 专用磁盘是保留原始 VM 中的用户帐户、应用程序和其他状态数据的现有 VM 中 VHD 的副本。 可以使用专用非托管磁盘创建新 VM。

## <a name="before-you-begin"></a>开始之前
如果使用 PowerShell，请确保使用的是最新版本的 AzureRM.Compute PowerShell 模块。 运行以下命令来安装该模块。

    Install-Module AzureRM.Compute -RequiredVersion 2.6.0

有关详细信息，请参阅 [Azure PowerShell 版本控制](https://docs.microsoft.com/zh-cn/powershell/azureps-cmdlets-docs/#azure-powershell-versioning)。

## <a name="create-the-subnet-and-vnet"></a>创建子网和 vNet

创建[虚拟网络](/documentation/articles/virtual-networks-overview/)的 vNet 和子网。

1. 创建子网。 本示例在资源组 **myResourceGroup** 中创建名为 **mySubnet** 的子网，并将子网地址前缀设置为 **10.0.0.0/24**。

        $rgName = "myResourceGroup"
        $subnetName = "mySubNet"
        $singleSubnet = New-AzureRmVirtualNetworkSubnetConfig -Name $subnetName -AddressPrefix 10.0.0.0/24

2. 创建 vNet。 本示例将虚拟网络名称设置为 **myVnetName**，将位置设置为“中国北部”，将虚拟网络的地址前缀设置为 **10.0.0.0/16**。 

        $location = "China North"
        $vnetName = "myVnetName"
        $vnet = New-AzureRmVirtualNetwork -Name $vnetName -ResourceGroupName $rgName -Location $location `
            -AddressPrefix 10.0.0.0/16 -Subnet $singleSubnet

## <a name="create-a-public-ip-address-and-nic"></a>创建公共 IP 地址和 NIC
若要与虚拟网络中的虚拟机通信，需要一个 [公共 IP 地址](/documentation/articles/virtual-network-ip-addresses-overview-arm/) 和网络接口。

1. 创建公共 IP。 在此示例中，公共 IP 地址名称设置为 **myIP**。

        $ipName = "myIP"
        $pip = New-AzureRmPublicIpAddress -Name $ipName -ResourceGroupName $rgName -Location $location `
            -AllocationMethod Dynamic

2. 创建 NIC。 在此示例中，NIC 名称设置为 **myNicName**。

        $nicName = "myNicName"
        $nic = New-AzureRmNetworkInterface -Name $nicName -ResourceGroupName $rgName `
        -Location $location -SubnetId $vnet.Subnets[0].Id -PublicIpAddressId $pip.Id

## <a name="create-the-network-security-group-and-an-rdp-rule"></a>创建网络安全组和 RDP 规则
若要使用 RDP 登录到 VM，需要创建一个允许在端口 3389 上进行 RDP 访问的安全规则。 由于新 VM 的 VHD 是从现有专用 VM 创建的，因此在创建 VM 后，可以使用源虚拟机中有权通过 RDP 登录的现有帐户。
本示例将 NSG 名称设置为 **myNsg**，将 RDP 规则名称设置为 **myRdpRule**。

    $nsgName = "myNsg"

    $rdpRule = New-AzureRmNetworkSecurityRuleConfig -Name myRdpRule -Description "Allow RDP" `
        -Access Allow -Protocol Tcp -Direction Inbound -Priority 110 `
        -SourceAddressPrefix Internet -SourcePortRange * `
        -DestinationAddressPrefix * -DestinationPortRange 3389
    $nsg = New-AzureRmNetworkSecurityGroup -ResourceGroupName $rgName -Location $location `
        -Name $nsgName -SecurityRules $rdpRule


有关终结点和 NSG 规则的详细信息，请参阅[使用 PowerShell 在 Azure 中打开 VM 端口](/documentation/articles/virtual-machines-windows-nsg-quickstart-powershell/)。

## <a name="set-the-vm-name-and-size"></a>设置 VM 名称和大小

此示例将 VM 名称设置为“myVM”，将 VM 大小设置为“Standard_A2”。

    $vmName = "myVM"
    $vmConfig = New-AzureRmVMConfig -VMName $vmName -VMSize "Standard_A2"

## <a name="add-the-nic"></a>添加 NIC

    $vm = Add-AzureRmVMNetworkInterface -VM $vmConfig -Id $nic.Id

## <a name="configure-the-os-disk"></a>配置 OS 磁盘

专用 OS 可以是[上传到 Azure](/documentation/articles/virtual-machines-windows-upload-image/) 的 VHD 或者[从现有 Azure VM 复制的 VHD](/documentation/articles/virtual-machines-windows-vhd-copy/)。 

可选择以下两个选项之一：
- **选项 1**：通过现有存储帐户的专用 VHD 创建专用托管磁盘，将其用作 OS 磁盘。Azure 中国区尚无法使用 Azure 托管磁盘。

或 

- **选项 2**：使用存储在自己的存储帐户中的专用 VHD（非托管磁盘）。 

### <a name="option-1-create-a-managed-disk-from-an-unmanaged-specialized-disk"></a>选项 1：通过非托管专用磁盘创建托管磁盘

>[AZURE.NOTE] Azure 中国区尚无法使用 Azure 托管磁盘。

### <a name="option-2-attach-a-vhd-that-is-in-an-existing-storage-account"></a>选项 2：附加现有存储帐户中的 VHD

1. 为想要使用的 VHD 设置 URI。 在此示例中，将名为 **myOsDisk.vhd** 的 VHD 文件保留在容器（名为 **myContainer**）中名为 **myStorageAccount** 的存储帐户中。

        $osDiskUri = "https://myStorageAccount.blob.core.chinacloudapi.cn/myContainer/myOsDisk.vhd"

2. 使用复制的 OS VHD 的 URL 添加 OS 磁盘。 在此示例中，创建 OS 磁盘时，“osDisk”一词将追加到用于创建 OS 磁盘名称的 VM 名称后。 此示例还指定应将此基于 Windows 的 VHD 附加到 VM 作为 OS 磁盘。

        $osDiskName = $vmName + "osDisk"
        $vm = Set-AzureRmVMOSDisk -VM $vm -Name $osDiskName -VhdUri $osDiskUri -CreateOption attach -Windows

可选：如果有需要附加到 VM 的数据磁盘，请使用数据 VHD 的 URL 和相应逻辑单元号 (Lun) 添加数据磁盘。

    $dataDiskName = $vmName + "dataDisk"
    $vm = Add-AzureRmVMDataDisk -VM $vm -Name $dataDiskName -VhdUri $dataDiskUri -Lun 1 -CreateOption attach

使用存储帐户时，数据和操作系统磁盘 URL 如下所示：`https://StorageAccountName.blob.core.chinacloudapi.cn/BlobContainerName/DiskName.vhd`。 可通过以下方法在门户上找到此信息：浏览到目标存储容器，单击复制的操作系统或数据 VHD，然后复制 URL 的内容。

## <a name="create-the-vm"></a>创建 VM

使用刚刚创建的配置创建 VM。

    #Create the new VM
    New-AzureRmVM -ResourceGroupName $rgName -Location $location -VM $vm

如果此命令成功，你将看到类似于下面的输出：

    RequestId IsSuccessStatusCode StatusCode ReasonPhrase
    --------- ------------------- ---------- ------------
                             True         OK OK   


## <a name="verify-that-the-vm-was-created"></a>验证是否已创建 VM
应会在 [Azure 门户预览](https://portal.azure.cn)的“浏览” > “虚拟机”下看到新建的 VM，也可以使用以下 PowerShell 命令查看该 VM：

    $vmList = Get-AzureRmVM -ResourceGroupName $rgName
    $vmList.Name

## <a name="next-steps"></a>后续步骤
若要登录到新虚拟机，请在[门户](https://portal.azure.cn)中浏览到该 VM，单击“连接”，然后打开远程桌面 RDP 文件。 使用原始虚拟机的帐户凭据登录到新虚拟机。 有关详细信息，请参阅[如何连接并登录到运行 Windows 的 Azure 虚拟机](/documentation/articles/virtual-machines-windows-connect-logon/)。
<!--Update_Description: wording update-->