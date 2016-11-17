<properties
   pageTitle="在 Windows VM 上配置多个 NIC | Azure"
   description="了解如何使用 Azure PowerShell 或 Resource Manager 模板创建附有多个 NIC 的 VM。"
   services="virtual-machines-windows"
   documentationCenter=""
   authors="iainfoulds"
   manager="timlt"
   editor=""/>  


<tags
	ms.service="virtual-machines-windows"
	ms.date="08/04/2016"
	wacn.date="09/05/2016"/>

# 创建具有多个 NIC 的 VM
可以在 Azure 中创建附有多个虚拟网络接口 (NIC) 的虚拟机 (VM)。一种常见方案是为前端和后端连接使用不同的子网，或者为监视或备份解决方案使用一个专用网络。本文提供用于创建附有多个 NIC 的 VM 的快速命令。有关详细信息，包括如何在自己的 PowerShell 脚本中创建多个 NIC，请阅读 [Deploying multi-NIC VMs](/documentation/articles/virtual-network-deploy-multinic-arm-ps/)（部署具有多个 NIC 的 VM）。不同的 [VM 大小](/documentation/articles/virtual-machines-windows-sizes/)支持不同数目的 NIC，因此请相应地调整 VM 的大小。

>[AZURE.WARNING] 必须在创建 VM 时附加多个 NIC - 不能将 NIC 添加到现有 VM。可以[基于原始虚拟磁盘创建新 VM](/documentation/articles/virtual-machines-windows-specialized-image/)，并在部署 VM 时创建多个 NIC。

## 创建核心资源
确保[已安装并配置最新的 Azure PowerShell](/documentation/articles/powershell-install-configure/)。

首先，创建一个资源组：

    New-AzureRmResourceGroup -Name TestRG -Location ChinaNorth

创建一个存储帐户用于存放 VM：

    $storageAcc = New-AzureRmStorageAccount -Name teststorage `
        -ResourceGroupName TestRG -Kind Storage -SkuName Premium_LRS -Location ChinaNorth

## 创建虚拟网络和子网
定义两个虚拟网络子网 - 一个用于前端流量，一个用于后端流量。创建包含以下子网的虚拟网络：

    $frontEndSubnet = New-AzureRmVirtualNetworkSubnetConfig -Name "FrontEnd" `
        -AddressPrefix 192.168.1.0/24
    $backEndSubnet = New-AzureRmVirtualNetworkSubnetConfig -Name "BackEnd" `
        -AddressPrefix 192.168.2.0/24


    $vnet = New-AzureRmVirtualNetwork -ResourceGroupName TestRG -Name TestVNet `
        -AddressPrefix 192.168.0.0/16 -Location ChinaNorth `
        -Subnet $frontEndSubnet,$backEndSubnet

## 创建多个 NIC
创建两个 NIC，并将其中一个 NIC 附加到前端子网，将另一个 NIC 附加到后端子网：

    $frontEnd = $vnet.Subnets|?{$_.Name -eq 'FrontEnd'}
    $NIC1 = New-AzureRmNetworkInterface -Name NIC1 -ResourceGroupName TestRG `
            -Location ChinaNorth -SubnetId $FrontEnd.Id

    $backEnd = $vnet.Subnets|?{$_.Name -eq 'BackEnd'}
    $NIC2 = New-AzureRmNetworkInterface -Name NIC2 -ResourceGroupName TestRG `
            -Location ChinaNorth -SubnetId $BackEnd.Id

通常，你还会创建[网络安全组](/documentation/articles/virtual-networks-nsg/)或[负载均衡器](/documentation/articles/load-balancer-overview/)来帮助管理流量以及跨 VM 分布流量。[更详细的多 NIC VM](/documentation/articles/virtual-network-deploy-multinic-arm-ps/) 文章将指导你创建网络安全组和分配 NIC。


## 创建虚拟机
立即开始构建 VM 配置。每种 VM 大小限制了可添加到 VM 的 NIC 数目。有关详细信息，请阅读 [Windows VM sizes](/documentation/articles/virtual-machines-windows-sizes/)（Windows VM 大小）。以下示例使用最多支持两个 NIC 的 VM 大小 (`Standard_DS2_v2`)：

    $cred = Get-Credential

    $vmConfig = New-AzureRmVMConfig -VMName TestVM -VMSize "Standard_DS2_v2"

    $vmConfig = Set-AzureRmVMOperatingSystem -VM $vmConfig -Windows -ComputerName TestVM `
        -Credential $cred -ProvisionVMAgent -EnableAutoUpdate
    $vmConfig = Set-AzureRmVMSourceImage -VM $vmConfig -PublisherName MicrosoftWindowsServer `
        -Offer WindowsServer -Skus 2012-R2-Datacenter -Version "latest"

附加前面创建的两个 NIC：

    $vmConfig = Add-AzureRmVMNetworkInterface -VM $vmConfig -Id $NIC1.Id -Primary
    $vmConfig = Add-AzureRmVMNetworkInterface -VM $vmConfig -Id $NIC2.Id

为新 VM 配置存储和虚拟磁盘：

    $blobPath = "vhds/WindowsVMosDisk.vhd"
    $osDiskUri = $storageAcc.PrimaryEndpoints.Blob.ToString() + $blobPath
    $diskName = "windowsvmosdisk"
    $vmConfig = Set-AzureRmVMOSDisk -VM $vmConfig -Name $diskName -VhdUri $osDiskUri `
        -CreateOption fromImage

最后，创建 VM：

    New-AzureRmVM -VM $vmConfig -ResourceGroupName TestRG -Location ChinaNorth

## 使用 Resource Manager 模板创建多个 NIC
Azure Resource Manager 模板使用声明性 JSON 文件来定义环境。阅读 [Azure Resource Manager 概述](/documentation/articles/resource-group-overview/)。Resource Manager 模板可让你在部署期间创建资源的多个实例，例如，创建多个 NIC。使用 *copy* 指定要创建的实例数：

    "copy": {
        "name": "multiplenics"
        "count": "[parameters('count')]"
    }

阅读有关[使用 *copy* 创建多个实例](/documentation/articles/resource-group-create-multiple/)的详细信息。

也可以使用 `copyIndex()` 并在资源名称中追加一个数字，来创建 `NIC1`、`NIC2`，等等。下面显示了追加索引值的示例：

    "name": "[concat('NIC-', copyIndex())]", 

阅读[使用 Resource Manager 模板创建多个 NIC](/documentation/articles/virtual-network-deploy-multinic-arm-template/) 的完整示例。

## 后续步骤
尝试创建具有多个 NIC 的 VM 时，请务必查看 [Windows VM sizes](/documentation/articles/virtual-machines-windows-sizes/)（Windows VM 大小）。注意每个 VM 大小支持的 NIC 数目上限。

请记住，不能将其他 NIC 添加到现有 VM，而必须在部署 VM 时创建所有 NIC。仔细规划部署，确保从一开始就建立了全部所需的网络连接。

<!---HONumber=Mooncake_0829_2016-->