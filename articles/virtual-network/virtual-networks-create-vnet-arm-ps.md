<properties
    pageTitle="使用 PowerShell 创建虚拟网络 | Azure"
    description="了解如何使用 PowerShell 创建虚拟网络 | Resource Manager。"
    services="virtual-network"
    documentationcenter=""
    author="jimdial"
    manager="carmonm"
    editor=""
    tags="azure-resource-manager" />  

<tags
    ms.assetid="a31f4f12-54ee-4339-b968-1a8097ca77d3"
    ms.service="virtual-network"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="03/15/2016"
    wacn.date="12/26/2016"
    ms.author="jdial" />  


# 使用 PowerShell 创建虚拟网络

[AZURE.INCLUDE [virtual-networks-create-vnet-intro](../../includes/virtual-networks-create-vnet-intro-include.md)]

Azure 有两个部署模型：Azure Resource Manager 和经典模型。Azure 建议通过 Resource Manager 部署模型创建资源。若要深入了解这两个模型之间的差异，请阅读[了解 Azure 部署模型](/documentation/articles/resource-manager-deployment-model/)一文。
 
本文介绍如何使用 PowerShell 通过 Resource Manager 部署模型创建 VNet。还可以使用其他工具通过 Resource Manager 创建 VNet，或通过从以下列表中选择不同的选项使用经典部署模型创建 VNet：
> [AZURE.SELECTOR]
- [门户](/documentation/articles/virtual-networks-create-vnet-arm-pportal/)
- [PowerShell](/documentation/articles/virtual-networks-create-vnet-arm-ps/)
- [CLI](/documentation/articles/virtual-networks-create-vnet-arm-cli/)
- [门户（经典）](/documentation/articles/virtual-networks-create-vnet-classic-pportal/)
- [PowerShell（经典）](/documentation/articles/virtual-networks-create-vnet-classic-netcfg-ps/)
- [CLI（经典）](/documentation/articles/virtual-networks-create-vnet-classic-cli/)

[AZURE.INCLUDE [virtual-networks-create-vnet-scenario-include](../../includes/virtual-networks-create-vnet-scenario-include.md)]

## 创建虚拟网络

若要使用 PowerShell 创建虚拟网络，请完成以下步骤：

1. 请按[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/) 一文中的步骤安装和配置 Azure PowerShell。

2. 如有必要，创建一个新的资源组，如下所示。对于此方案，创建一个名为 *TestRG* 的资源组。有关资源组的详细信息，请访问 [Azure 资源管理器概述](/documentation/articles/resource-group-overview/)。

        New-AzureRmResourceGroup -Name TestRG -Location chinaeast

    预期输出：

        ResourceGroupName : TestRG
        Location          : chinaeast
        ProvisioningState : Succeeded
        Tags              :
        ResourceId        : /subscriptions/[Subscription Id]/resourceGroups/TestRG    
3. 创建名为 *TestVNet* 的新 VNet：

        New-AzureRmVirtualNetwork -ResourceGroupName TestRG -Name TestVNet `
        -AddressPrefix 192.168.0.0/16 -Location chinaeast

    预期输出：

        Name                  : TestVNet
        ResourceGroupName     : TestRG
        Location              : chinaeast
        Id                    : /subscriptions/[Subscription Id]/resourceGroups/TestRG/providers/Microsoft.Network/virtualNetworks/TestVNet
        Etag                   : W/"[Id]"
        ProvisioningState          : Succeeded
        Tags                       : 
        AddressSpace               : {
                                   "AddressPrefixes": [
                                     "192.168.0.0/16"
                                   ]
                                  }
        DhcpOptions                : {}
        Subnets                    : []
        VirtualNetworkPeerings     : []
4. 将虚拟网络对象存储在变量中：

        $vnet = Get-AzureRmVirtualNetwork -ResourceGroupName TestRG -Name TestVNet

   > [AZURE.TIP]
   可通过运行 `$vnet = New-AzureRmVirtualNetwork -ResourceGroupName TestRG -Name TestVNet -AddressPrefix 192.168.0.0/16 -Location chinaeast` 合并步骤 3 和步骤 4。
   > 

5. 将子网添加到新的 VNet 变量中：

        Add-AzureRmVirtualNetworkSubnetConfig -Name FrontEnd `
        -VirtualNetwork $vnet -AddressPrefix 192.168.1.0/24

    预期输出：
   
        Name                  : TestVNet
        ResourceGroupName     : TestRG
        Location              : chinaeast
        Id                    : /subscriptions/[Subscription Id]/resourceGroups/TestRG/providers/Microsoft.Network/virtualNetworks/TestVNet
        Etag                  : W/"[Id]"
        ProvisioningState     : Succeeded
        Tags                  :
        AddressSpace          : {
                                  "AddressPrefixes": [
                                    "192.168.0.0/16"
                                  ]
                                }
        DhcpOptions           : {}
        Subnets             : [
                                  {
                                    "Name": "FrontEnd",
                                    "AddressPrefix": "192.168.1.0/24"
                                  }
                                ]
        VirtualNetworkPeerings     : []

6. 对要创建的每个子网重复执行上面的步骤 5。对于此方案，以下命令将创建 *BackEnd* 子网：

        Add-AzureRmVirtualNetworkSubnetConfig -Name BackEnd `
        -VirtualNetwork $vnet -AddressPrefix 192.168.2.0/24

7. 尽管你创建了子网，但它们当前仅存在于用于检索你在上面的步骤 4 中创建的 VNet 的本地变量中。若要将更改保存到 Azure，请运行以下命令：

        Set-AzureRmVirtualNetwork -VirtualNetwork $vnet

    预期输出：
   
        Name                  : TestVNet
        ResourceGroupName     : TestRG
        Location              : chinaeast
        Id                    : /subscriptions/[Subscription Id]/resourceGroups/TestRG/providers/Microsoft.Network/virtualNetworks/TestVNet
        Etag                  : W/"[Id]"
        ProvisioningState     : Succeeded
        Tags                  :
        AddressSpace          : {
                                  "AddressPrefixes": [
                                    "192.168.0.0/16"
                                  ]
                                }
        DhcpOptions           : {
                                  "DnsServers": []
                                }
        Subnets               : [
                                  {
                                    "Name": "FrontEnd",
                                    "Etag": "W/"[Id]"",
                                    "Id": "/subscriptions/[Subscription Id]/resourceGroups/TestRG/providers/Microsoft.Network/virtualNetworks/TestVNet/subnets/FrontEnd",
                                    "AddressPrefix": "192.168.1.0/24",
                                    "IpConfigurations": [],
                                    "ProvisioningState": "Succeeded"
                                  },
                                  {
                                    "Name": "BackEnd",
                                    "Etag": "W/"[Id]"",
                                    "Id": "/subscriptions/[Subscription Id]/resourceGroups/TestRG/providers/Microsoft.Network/virtualNetworks/TestVNet/subnets/BackEnd",
                                    "AddressPrefix": "192.168.2.0/24",
                                    "IpConfigurations": [],
                                    "ProvisioningState": "Succeeded"
                                  }
                                ]
        VirtualNetworkPeerings : []

## 后续步骤

了解如何连接：

- 通过阅读[创建 Windows VM](/documentation/articles/virtual-machines-windows-ps-create/)一文，将虚拟机 (VM) 连接到虚拟网络。可选择将 VM 连接到现有 VNet 和子网，而不按文章中的步骤创建 VNet 和子网。
- 通过阅读[连接 VNet](/documentation/articles/vpn-gateway-vnet-vnet-rm-ps/) 一文将虚拟网络连接到其他虚拟网络。
- 使用站点到站点虚拟专用网络 (VPN) 或 ExpressRoute 线路将虚拟网络连接到本地网络。通过阅读[使用站点到站点 VPN 将 VNet 连接到本地网络](/documentation/articles/vpn-gateway-howto-multi-site-to-site-resource-manager-portal/)和[将 VNet 链接到 ExpressRoute 线路](/documentation/articles/expressroute-howto-linkvnet-arm/)文章，了解操作方法。

<!---HONumber=Mooncake_1219_2016-->