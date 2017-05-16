<properties
    pageTitle="从通用 VHD 创建 VM | Azure"
    description="了解如何使用 Azure PowerShell 以及存储帐户的通用 VHD 映像创建 Windows 虚拟机。"
    services="virtual-machines-windows"
    documentationcenter=""
    author="cynthn"
    manager="timlt"
    editor=""
    tags="azure-resource-manager" />
<tags
    ms.assetid="b4808871-9ef1-49ea-a617-9154d417abb0"
    ms.service="virtual-machines-windows"
    ms.workload="infrastructure-services"
    ms.tgt_pltfrm="vm-windows"
    ms.devlang="na"
    ms.topic="article"
    ms.date="03/21/2017"
    wacn.date="05/15/2017"
    ms.author="cynthn"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="457fc748a9a2d66d7a2906b988e127b09ee11e18"
    ms.openlocfilehash="a2def8b04e148d6eb95378c987703040fb02a989"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/05/2017" />

# <a name="create-a-vm-from-a-generalized-vhd-image-in-a-storage-account"></a>基于存储帐户中的通用 VHD 映像创建 VM 

本主题介绍如何基于存储帐户中的通用非托管磁盘创建 VM。 通用 VHD 映像包含使用 [Sysprep](/documentation/articles/virtual-machines-windows-generalize-vhd/) 删除的所有个人帐户信息。 若要创建通用 VHD，可在本地 VM 上运行 Sysprep，然后[将 VHD 上传到 Azure](/documentation/articles/virtual-machines-windows-upload-image/)；或者在现有 Azure VM 上运行 Sysprep，然后[复制 VHD](/documentation/articles/virtual-machines-windows-vhd-copy/)。

如果要基于存储帐户中的专用 VHD 创建 VM，请参阅[从专用 VHD 创建 VM](/documentation/articles/virtual-machines-windows-create-vm-specialized/)。

## <a name="prerequisites"></a>先决条件
如果打算使用从本地 VM 上传的 VHD（类似于使用 Hyper-V 创建的 VHD），应确保遵循[准备好要上传到 Azure 的 Windows VHD](/documentation/articles/virtual-machines-windows-prepare-for-upload-vhd-image/) 中的说明。 

在使用此方法创建 VM 之前，必须先通用化上载的 VHD 和现有的 Azure VM VHD。 有关详细信息，请参阅 [Generalize a Windows virtual machine using Sysprep](/documentation/articles/virtual-machines-windows-generalize-vhd/)（使用 Sysprep 通用化 Windows 虚拟机）。 

## <a name="set-the-uri-of-the-vhd"></a>设置 VHD 的 URI

VHD 使用的 URI 采用以下格式：https://**mystorageaccount**.blob.core.chinacloudapi.cn/**mycontainer**/**MyVhdName**.vhd。 在此示例中，名为 **myVHD** 的 VHD 位于存储帐户 **mystorageaccount** 的 **mycontainer** 容器中。

    $imageURI = "https://mystorageaccount.blob.core.chinacloudapi.cn/mycontainer/myVhd.vhd"

## <a name="create-a-virtual-network"></a>创建虚拟网络
创建[虚拟网络](/documentation/articles/virtual-networks-overview/)的 vNet 和子网。

1. 创建子网。 以下示例在资源组 **myResourceGroup** 中创建具有 **10.0.0.0/24** 地址前缀的、名为 **mySubnet** 的子网。  

        $rgName = "myResourceGroup"
        $subnetName = "mySubnet"
        $singleSubnet = New-AzureRmVirtualNetworkSubnetConfig -Name $subnetName -AddressPrefix 10.0.0.0/24

2. 创建虚拟网络。 以下示例在**中国北部**位置创建具有 **10.0.0.0/16** 地址前缀的、名为 **myVnet** 的虚拟网络。  

        $location = "China North"
        $vnetName = "myVnet"
        $vnet = New-AzureRmVirtualNetwork -Name $vnetName -ResourceGroupName $rgName -Location $location `
            -AddressPrefix 10.0.0.0/16 -Subnet $singleSubnet

## <a name="create-a-public-ip-address-and-network-interface"></a>创建公共 IP 地址和网络接口
若要与虚拟网络中的虚拟机通信，需要一个 [公共 IP 地址](/documentation/articles/virtual-network-ip-addresses-overview-arm/) 和网络接口。

1. 创建公共 IP 地址。 此示例创建名为 **myPip**的公共 IP 地址。 

        $ipName = "myPip"
        $pip = New-AzureRmPublicIpAddress -Name $ipName -ResourceGroupName $rgName -Location $location `
            -AllocationMethod Dynamic

2. 创建 NIC。 此示例创建名为 **myNic**的 NIC。 

        $nicName = "myNic"
        $nic = New-AzureRmNetworkInterface -Name $nicName -ResourceGroupName $rgName -Location $location `
            -SubnetId $vnet.Subnets[0].Id -PublicIpAddressId $pip.Id

## <a name="create-the-network-security-group-and-an-rdp-rule"></a>创建网络安全组和 RDP 规则
若要使用 RDP 登录到 VM，需要创建一个允许在端口 3389 上进行 RDP 访问的安全规则。 

此示例创建名为 **myNsg** 的 NSG，其中包含一个允许通过端口 3389 传输 RDP 流量的、名为 **myRdpRule** 的规则。 有关 NSG 的详细信息，请参阅 [Opening ports to a VM in Azure using PowerShell](/documentation/articles/virtual-machines-windows-nsg-quickstart-powershell/)（使用 PowerShell 在 Azure 中打开 VM 端口）。

    $nsgName = "myNsg"

    $rdpRule = New-AzureRmNetworkSecurityRuleConfig -Name myRdpRule -Description "Allow RDP" `
        -Access Allow -Protocol Tcp -Direction Inbound -Priority 110 `
        -SourceAddressPrefix Internet -SourcePortRange * `
        -DestinationAddressPrefix * -DestinationPortRange 3389

    $nsg = New-AzureRmNetworkSecurityGroup -ResourceGroupName $rgName -Location $location `
        -Name $nsgName -SecurityRules $rdpRule

## <a name="create-a-variable-for-the-virtual-network"></a>为虚拟网络创建变量
为完成的虚拟网络创建变量。 

    $vnet = Get-AzureRmVirtualNetwork -ResourceGroupName $rgName -Name $vnetName

## <a name="create-the-vm"></a>创建 VM
以下 PowerShell 脚本演示如何设置虚拟机配置，并使用上载的 VM 映像作为新安装的源。

</br>

        # Enter a new user name and password to use as the local administrator account 
        # for remotely accessing the VM.
        $cred = Get-Credential

        # Name of the storage account where the VHD is located. This example sets the 
        # storage account name as "myStorageAccount"
        $storageAccName = "myStorageAccount"

        # Name of the virtual machine. This example sets the VM name as "myVM".
        $vmName = "myVM"

        # Size of the virtual machine. This example creates "Standard_D2_v2" sized VM. 
        # See the VM sizes documentation for more information: 
        # https://www.azure.cn/documentation/articles/virtual-machines-windows-sizes/
        $vmSize = "Standard_D2_v2"

        # Computer name for the VM. This examples sets the computer name as "myComputer".
        $computerName = "myComputer"

        # Name of the disk that holds the OS. This example sets the 
        # OS disk name as "myOsDisk"
        $osDiskName = "myOsDisk"

        # Assign a SKU name. This example sets the SKU name as "Standard_LRS"
        # Valid values for -SkuName are: Standard_LRS - locally redundant storage, Standard_ZRS - zone redundant
        # storage, Standard_GRS - geo redundant storage, Standard_RAGRS - read access geo redundant storage,
        # Premium_LRS - premium locally redundant storage. 
        $skuName = "Standard_LRS"

        # Get the storage account where the uploaded image is stored
        $storageAcc = Get-AzureRmStorageAccount -ResourceGroupName $rgName -AccountName $storageAccName

        # Set the VM name and size
        $vmConfig = New-AzureRmVMConfig -VMName $vmName -VMSize $vmSize

        #Set the Windows operating system configuration and add the NIC
        $vm = Set-AzureRmVMOperatingSystem -VM $vmConfig -Windows -ComputerName $computerName `
            -Credential $cred -ProvisionVMAgent -EnableAutoUpdate
        $vm = Add-AzureRmVMNetworkInterface -VM $vm -Id $nic.Id

        # Create the OS disk URI
        $osDiskUri = '{0}vhds/{1}-{2}.vhd' `
            -f $storageAcc.PrimaryEndpoints.Blob.ToString(), $vmName.ToLower(), $osDiskName

        # Configure the OS disk to be created from the existing VHD image (-CreateOption fromImage).
        $vm = Set-AzureRmVMOSDisk -VM $vm -Name $osDiskName -VhdUri $osDiskUri `
            -CreateOption fromImage -SourceImageUri $imageURI -Windows

        # Create the new VM
        New-AzureRmVM -ResourceGroupName $rgName -Location $location -VM $vm

## <a name="verify-that-the-vm-was-created"></a>验证是否已创建 VM
完成后，应会在 [Azure 门户预览](https://portal.azure.cn)的“浏览” > “虚拟机”下看到新建的 VM，也可以使用以下 PowerShell 命令查看该 VM：

        $vmList = Get-AzureRmVM -ResourceGroupName $rgName
        $vmList.Name

## <a name="next-steps"></a>后续步骤
若要使用 Azure PowerShell 管理新虚拟机，请参阅[使用 Azure Resource Manager 与 PowerShell 来管理虚拟机](/documentation/articles/virtual-machines-windows-ps-manage/)。

<!--Update_Description: wording update-->