<properties
    pageTitle="使用 PowerShell 控制路由和虚拟设备 | Azure"
    description="了解如何使用 PowerShell 控制路由和虚拟设备。"
    services="virtual-network"
    documentationcenter="na"
    author="jimdial"
    manager="carmonm"
    editor=""
    tags="azure-resource-manager" />  

<tags
    ms.assetid="9582fdaa-249c-4c98-9618-8c30d496940f"
    ms.service="virtual-network"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="02/23/2016"
    wacn.date="12/26/2016"
    ms.author="jdial" />  


# 使用 PowerShell 创建用户定义的路由 (UDR)
> [AZURE.SELECTOR]
- [PowerShell](/documentation/articles/virtual-network-create-udr-arm-ps/)
- [Azure CLI](/documentation/articles/virtual-network-create-udr-arm-cli/)
- [模板](/documentation/articles/virtual-network-create-udr-arm-template/)
- [PowerShell（经典）](/documentation/articles/virtual-network-create-udr-classic-ps/)
- [CLI（经典）](/documentation/articles/virtual-network-create-udr-classic-cli/)

[AZURE.INCLUDE [virtual-network-create-udr-intro-include.md](../../includes/virtual-network-create-udr-intro-include.md)]

> [AZURE.IMPORTANT]
在使用 Azure 资源之前，请务必了解 Azure 当前使用两种部署模型：Azure Resource Manager 部署模型和经典部署模型。在使用任何 Azure 资源前，请确保了解[部署模型和工具](/documentation/articles/resource-manager-deployment-model/)。可以通过单击本文顶部的选项卡来查看不同工具的文档。本文介绍 Resource Manager 部署模型。还可[在经典部署模型中创建 UDR](/documentation/articles/virtual-network-create-udr-classic-ps/)。

[AZURE.INCLUDE [virtual-network-create-udr-scenario-include.md](../../includes/virtual-network-create-udr-scenario-include.md)]

下面的示例 PowerShell 命令需要基于上述方案创建的简单环境。若要运行本文档中所显示的命令，请首先通过部署[此模板](http://github.com/telmosampaio/azure-templates/tree/master/IaaS-NSG-UDR-Before)构建测试环境，单击“部署至 Azure”，根据需要替换默认参数值，然后按照门户中的说明进行操作。

[AZURE.INCLUDE [azure-ps-prerequisites-include.md](../../includes/azure-ps-prerequisites-include.md)]

## 为前端子网创建 UDR
若要根据上述方案为前端子网创建所需的路由表和路由，请完成以下步骤：

1. 创建一个路由，用于将流向后端子网 (192.168.2.0/24) 的所有流量路由到 **FW1** 虚拟设备 (192.168.0.4)。

        $route = New-AzureRmRouteConfig -Name RouteToBackEnd `
        -AddressPrefix 192.168.2.0/24 -NextHopType VirtualAppliance `
        -NextHopIpAddress 192.168.0.4

2. 在 **chinanorth** 区域中创建一个名为 **UDR-FrontEnd** 的路由表，其中包含路由。

        $routeTable = New-AzureRmRouteTable -ResourceGroupName TestRG -Location chinanorth `
        -Name UDR-FrontEnd -Route $route

3. 创建一个变量，包含该子网所在的 VNet。在我们的方案中，VNet 名为 **TestVNet**。

        $vnet = Get-AzureRmVirtualNetwork -ResourceGroupName TestRG -Name TestVNet

4. 将上面创建的路由表与 **FrontEnd** 子网关联起来。

        Set-AzureRmVirtualNetworkSubnetConfig -VirtualNetwork $vnet -Name FrontEnd `
        -AddressPrefix 192.168.1.0/24 -RouteTable $routeTable

    > [AZURE.WARNING]
    上述命令的输出显示虚拟网络配置对象的内容，该对象仅存在于运行 PowerShell 的计算机上。若要将这些设置保存到 Azure，需要运行 **Set-AzureVirtualNetwork** cmdlet。
    > 

5. 将新的子网配置保存在 Azure 中。

        Set-AzureRmVirtualNetwork -VirtualNetwork $vnet

    预期输出：
   
        Name              : TestVNet
        ResourceGroupName : TestRG
        Location          : chinanorth
        Id                : /subscriptions/[Subscription Id]/resourceGroups/TestRG/providers/Microsoft.Network/virtualNetworks/TestVNet
        Etag              : W/"[Id]"
        ProvisioningState : Succeeded
        Tags              : 
                            Name         Value
                            ===========  =====
                            displayName  VNet 
   
        AddressSpace      : {
                              "AddressPrefixes": [
                                "192.168.0.0/16"
                              ]
                            }
        DhcpOptions       : {
                              "DnsServers": null
                            }
        NetworkInterfaces : null
        Subnets           : [
                                ...,
                              {
                                "Name": "FrontEnd",
                                "Etag": "W/\"[Id]\"",
                                "Id": "/subscriptions/[Id]/resourceGroups/TestRG/providers/Microsoft.Network/virtualNetworks/TestVNet/subnets/FrontEnd",
                                "AddressPrefix": "192.168.1.0/24",
                                "IpConfigurations": [
                                  {
                                    "Id": "/subscriptions/[Subscription Id]/resourceGroups/TestRG/providers/Microsoft.Network/networkInterfaces/NICWEB2/ipConfigurations/ipconfig1"
                                  },
                                  {
                                    "Id": "/subscriptions/[Subscription Id]/resourceGroups/TestRG/providers/Microsoft.Network/networkInterfaces/NICWEB1/ipConfigurations/ipconfig1"
                                  }
                                ],
                                "NetworkSecurityGroup": {
                                  "Id": "/subscriptions/[Subscription Id]/resourceGroups/TestRG/providers/Microsoft.Network/networkSecurityGroups/NSG-FrontEnd"
                                },
                                "RouteTable": {
                                  "Id": "/subscriptions/[Subscription Id]/resourceGroups/TestRG/providers/Microsoft.Network/routeTables/UDR-FrontEnd"
                                },
                                "ProvisioningState": "Succeeded"
                              },
                                ...
                            ]    

## 为后端子网创建 UDR

若要根据上述方案为后端子网创建所需的路由表和路由，请按照下面的步骤操作。

1. 创建一个路由，用于将流向前端子网 (192.168.1.0/24) 的所有流量路由到 **FW1** 虚拟设备 (192.168.0.4)。

        $route = New-AzureRmRouteConfig -Name RouteToFrontEnd `
        -AddressPrefix 192.168.1.0/24 -NextHopType VirtualAppliance `
        -NextHopIpAddress 192.168.0.4

2. 在 **chinanorth** 区域中创建一个名为 **UDR-BackEnd** 的路由表，其中包含上面创建的路由。

        $routeTable = New-AzureRmRouteTable -ResourceGroupName TestRG -Location chinanorth `
        -Name UDR-BackEnd -Route $route

3. 将上面创建的路由表与 **BackEnd** 子网关联起来。

        Set-AzureRmVirtualNetworkSubnetConfig -VirtualNetwork $vnet -Name BackEnd `
        -AddressPrefix 192.168.2.0/24 -RouteTable $routeTable

4. 将新的子网配置保存在 Azure 中。

        Set-AzureRmVirtualNetwork -VirtualNetwork $vnet

    预期输出：
   
        Name              : TestVNet
        ResourceGroupName : TestRG
        Location          : chinanorth
        Id                : /subscriptions/[Subscription Id]/resourceGroups/TestRG/providers/Microsoft.Network/virtualNetworks/TestVNet
        Etag              : W/"[Id]"
        ProvisioningState : Succeeded
        Tags              : 
                            Name         Value
                            ===========  =====
                            displayName  VNet 
   
        AddressSpace      : {
                              "AddressPrefixes": [
                                "192.168.0.0/16"
                              ]
                            }
        DhcpOptions       : {
                              "DnsServers": null
                            }
        NetworkInterfaces : null
        Subnets           : [
                              ...,
                              {
                                "Name": "BackEnd",
                                "Etag": "W/\"[Id]\"",
                                "Id": "/subscriptions/[Subscription Id]/resourceGroups/TestRG/providers/Microsoft.Network/virtualNetworks/TestVNet/subnets/BackEnd",
                                "AddressPrefix": "192.168.2.0/24",
                                "IpConfigurations": [
                                  {
                                    "Id": "/subscriptions/[Subscription Id]/resourceGroups/TestRG/providers/Microsoft.Network/networkInterfaces/NICSQL2/ipConfigurations/ipconfig1"
                                  },
                                  {
                                    "Id": "/subscriptions/[Subscription Id]/resourceGroups/TestRG/providers/Microsoft.Network/networkInterfaces/NICSQL1/ipConfigurations/ipconfig1"
                                  }
                                ],
                                "NetworkSecurityGroup": {
                                  "Id": "/subscriptions/[Subscription Id]/resourceGroups/TestRG/providers/Microsoft.Network/networkSecurityGroups/NSG-BacEnd"
                                },
                                "RouteTable": {
                                  "Id": "/subscriptions/[Subscription Id]/resourceGroups/TestRG/providers/Microsoft.Network/routeTables/UDR-BackEnd"
                                },
                                "ProvisioningState": "Succeeded"
                              }
                            ]

## 在 FW1 上启用 IP 转发
若要在 **FW1** 使用的 NIC 上启用 IP 转发，请按照下面的步骤操作。

1. 创建一个变量，包含由 FW1 使用的 NIC 的设置。在我们的方案中，NIC 名为 **NICFW1**。

        $nicfw1 = Get-AzureRmNetworkInterface -ResourceGroupName TestRG -Name NICFW1

2. 启用 IP 转发，并保存 NIC 设置。

        $nicfw1.EnableIPForwarding = 1
        Set-AzureRmNetworkInterface -NetworkInterface $nicfw1
   
    预期输出：
   
        Name                 : NICFW1
        ResourceGroupName    : TestRG
        Location             : chinanorth
        Id                   : /subscriptions/[Subscription Id]/resourceGroups/TestRG/providers/Microsoft.Network/networkInterfaces/NICFW1
        Etag                 : W/"[Id]"
        ProvisioningState    : Succeeded
        Tags                 : 
                               Name         Value                  
                               ===========  =======================
                               displayName  NetworkInterfaces - DMZ
   
        VirtualMachine       : {
                                 "Id": "/subscriptions/[Subscription Id]/resourceGroups/TestRG/providers/Microsoft.Compute/virtualMachines/FW1"
                               }
        IpConfigurations     : [
                                 {
                                   "Name": "ipconfig1",
                                   "Etag": "W/\"[Id]\"",
                                   "Id": "/subscriptions/[Subscription Id]/resourceGroups/TestRG/providers/Microsoft.Network/networkInterfaces/NICFW1/ipConfigurations/ipconfig1",
                                   "PrivateIpAddress": "192.168.0.4",
                                   "PrivateIpAllocationMethod": "Static",
                                   "Subnet": {
                                     "Id": "/subscriptions/[Subscription Id]/resourceGroups/TestRG/providers/Microsoft.Network/virtualNetworks/TestVNet/subnets/DMZ"
                                   },
                                   "PublicIpAddress": {
                                     "Id": "/subscriptions/[Subscription Id]/resourceGroups/TestRG/providers/Microsoft.Network/publicIPAddresses/PIPFW1"
                                   },
                                   "LoadBalancerBackendAddressPools": [],
                                   "LoadBalancerInboundNatRules": [],
                                   "ProvisioningState": "Succeeded"
                                 }
                               ]
        DnsSettings          : {
                                 "DnsServers": [],
                                 "AppliedDnsServers": [],
                                 "InternalDnsNameLabel": null,
                                 "InternalFqdn": null
                               }
        EnableIPForwarding   : True
        NetworkSecurityGroup : null
        Primary              : True

<!---HONumber=Mooncake_1219_2016-->