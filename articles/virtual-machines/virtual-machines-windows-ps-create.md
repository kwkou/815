<properties
    pageTitle="使用 PowerShell 创建 Azure VM | Azure"
    description="使用 Azure PowerShell 和 Azure Resource Manager 轻松创建运行 Windows Server 的 VM。"
    services="virtual-machines-windows"
    documentationcenter=""
    author="davidmu1"
    manager="timlt"
    editor=""
    tags="azure-resource-manager" />
<tags 
    ms.assetid="14fe9ca9-e228-4d3b-a5d8-3101e9478f6e"
    ms.service="virtual-machines-windows"
    ms.topic="get-started-article"
    ms.date="02/14/2017"
    wacn.date="03/20/2017"
    ms.author="davidmu" />

# 使用 Resource Manager 和 PowerShell 创建 Windows VM

本文介绍如何使用 [Resource Manager](/documentation/articles/resource-group-overview/) 和 PowerShell 快速创建运行 Windows Server 的 Azure 虚拟机及其所需的资源。创建虚拟机需要执行本文中的所有步骤，需要大约 30 分钟。将命令中的示例参数值替换为对环境有意义的名称。

## 步骤 1：安装 Azure PowerShell

有关安装最新版本的 Azure PowerShell、选择订阅和登录帐户的信息，请参阅[如何安装和配置 Azure PowerShell](https://docs.microsoft.com/powershell/azureps-cmdlets-docs)。

## 步骤 2：创建资源组

所有资源都必须包含在资源组中，因此让我们先创建资源组。

1. 获取可以创建资源的可用位置列表。

        Get-AzureRmLocation | sort Location | Select Location

2. 设置资源的位置。此命令将位置设置为 **chinaeast**。

        $location = "chinaeast"

3. 创建资源组。此命令在你设置的位置创建名为 **myResourceGroup** 的资源组。

        $myResourceGroup = "myResourceGroup"
        New-AzureRmResourceGroup -Name $myResourceGroup -Location $location

## 步骤 3：创建存储帐户

Azure 托管磁盘尚在 Azure 中国区中不可用，因此你必须创建[存储帐户](/documentation/articles/storage-introduction/)以存储创建的虚拟机所用的虚拟硬盘。

1. 测试存储帐户名称是否唯一。此命令测试名称 **myStorageAccount**。

        $myStorageAccountName = "mystorageaccount"
        Get-AzureRmStorageAccountNameAvailability $myStorageAccountName

    如果此命令返回 **True**，表示建议的名称在 Azure 中唯一。

2. 现在，创建存储帐户。

        $myStorageAccount = New-AzureRmStorageAccount -ResourceGroupName $myResourceGroup `
            -Name $myStorageAccountName -SkuName "Standard_LRS" -Kind "Storage" -Location $location

## 步骤 4：创建虚拟网络

所有虚拟机都是[虚拟网络](/documentation/articles/virtual-networks-overview/)的一部分。

1. 为虚拟网络创建子网。此命令创建名为 **mySubnet** 的子网，其地址前缀为 10.0.0.0/24。

        $mySubnet = New-AzureRmVirtualNetworkSubnetConfig -Name "mySubnet" -AddressPrefix 10.0.0.0/24

2. 现在，创建虚拟网络。此命令使用已创建的子网和地址前缀 **10.0.0.0/16** 创建名为 **myVnet** 的虚拟网络。

        $myVnet = New-AzureRmVirtualNetwork -Name "myVnet" -ResourceGroupName $myResourceGroup `
            -Location $location -AddressPrefix 10.0.0.0/16 -Subnet $mySubnet

## 步骤 5：创建公共 IP 地址和网络接口

若要与虚拟网络中的虚拟机通信，需要[公共 IP 地址](/documentation/articles/virtual-network-ip-addresses-overview-arm/)和网络接口。

1. 创建公共 IP 地址。此命令使用**动态**分配方法创建名为 **myPublicIp** 的公共 IP 地址。

        $myPublicIp = New-AzureRmPublicIpAddress -Name "myPublicIp" -ResourceGroupName $myResourceGroup `
            -Location $location -AllocationMethod Dynamic

2. 创建网络接口。此命令创建名为 **myNIC** 的网络接口。

        $myNIC = New-AzureRmNetworkInterface -Name "myNIC" -ResourceGroupName $myResourceGroup `
            -Location $location -SubnetId $myVnet.Subnets[0].Id -PublicIpAddressId $myPublicIp.Id

## 步骤 6：创建虚拟机

现已做好所有的准备，接下来可以开始创建虚拟机。可以使用[应用商店映像](/documentation/articles/virtual-machines-windows-cli-ps-findimage/)、[自定义通用 (sysprepped) 映像](/documentation/articles/virtual-machines-windows-create-vm-generalized/)或[自定义专用（非 sysprepped） 映像](/documentation/articles/virtual-machines-windows-create-vm-specialized/)创建虚拟机。此示例使用应用商店中的 Windows Server 映像。

1. 运行以下命令设置虚拟机的管理员帐户名和密码。

        $cred = Get-Credential -Message "Type the name and password of the local administrator account."

    密码长度必须为 12 到 123 个字符，并且至少必须包含 1 个小写字符、1 个大写字符、1 个数字和 1 个特殊字符。

2. 创建虚拟机配置对象。此命令创建名为 **myVmConfig** 的配置对象，以定义 VM 的名称和大小。

        $myVM = New-AzureRmVMConfig -VMName "myVM" -VMSize "Standard_DS1_v2"

    有关虚拟机的可用大小列表，请参阅 [Sizes for virtual machines in Azure](/documentation/articles/virtual-machines-windows-sizes/)（Azure 中的虚拟机大小）。

3. 配置 VM 的操作系统设置。此命令设置 VM 的计算机名称、操作系统类型和帐户凭据。

        $myVM = Set-AzureRmVMOperatingSystem -VM $myVM -Windows -ComputerName "myVM" -Credential $cred `
            -ProvisionVMAgent -EnableAutoUpdate

4. 定义要用于预配 VM 的映像。此命令定义要用于 VM 的 Windows Server 映像。

        $myVM = Set-AzureRmVMSourceImage -VM $myVM -PublisherName "MicrosoftWindowsServer" `
            -Offer "WindowsServer" -Skus "2012-R2-Datacenter" -Version "latest"

5. 将创建的网络接口添加到配置。

        $myVM = Add-AzureRmVMNetworkInterface -VM $myVM -Id $myNIC.Id

6. 运行此命令以定义 VM 硬盘的名称和位置；否则，请跳过此步骤。非托管磁盘的虚拟硬盘文件将存储在容器中。此命令在创建的存储帐户的名为 **vhds/myOsDisk1.vhd** 的容器中创建磁盘。

        $blobPath = "vhds/myOsDisk1.vhd"
        $osDiskUri = $myStorageAccount.PrimaryEndpoints.Blob.ToString() + $blobPath

7. 将操作系统磁盘信息添加到 VM 配置。此命令创建名为 **myOsDisk1** 的磁盘。
   
    运行此命令可在配置中设置操作系统磁盘：

        $myVM = Set-AzureRmVMOSDisk -VM $myVM -Name "myOsDisk1" -VhdUri $osDiskUri -CreateOption fromImage

8. 最后，创建虚拟机。

        New-AzureRmVM -ResourceGroupName $myResourceGroup -Location $location -VM $myVM

## 后续步骤

* 如果部署出现问题，请参阅[排查使用 Azure Resource Manager 时的 Azure 常见部署错误](/documentation/articles/resource-manager-common-deployment-errors/)
* 若要了解如何管理刚创建的虚拟机，请参阅[使用 Azure Resource Manager 和 PowerShell 管理虚拟机](/documentation/articles/virtual-machines-windows-ps-manage/)。
* 参考 [Create a Windows virtual machine with a Resource Manager template](/documentation/articles/virtual-machines-windows-ps-template/)（使用 Resource Manager 模板创建 Windows 虚拟机）中的信息，使用模板创建虚拟机

<!---HONumber=Mooncake_0313_2017-->
<!--Update_Description: wording update-->