<properties
   pageTitle="使用 Azure 资源管理器和 PowerShell 为同一订阅中存在的 VNet 创建 VNet 到 VNet VPN 网关连接 | Microsoft Azure"
   description="本文指导你使用 Azure 资源管理器和 PowerShell 将虚拟网络连接在一起。"
   services="vpn-gateway"
   documentationCenter="na"
   authors="cherylmc"
   manager="carolz"
   editor=""
   tags="azure-resource-manager"/>

<tags
   ms.service="vpn-gateway"
   ms.date="12/18/2015"
   wacn.date="01/15/2016"/>

# 使用 Azure 资源管理器和 PowerShell，为同一订阅中的虚拟网络配置 VNet 到 VNet 连接

> [AZURE.SELECTOR]
- [Azure 经典门户](/documentation/articles/virtual-networks-configure-vnet-to-vnet-connection)
- [PowerShell - Azure 资源管理器](/documentation/articles/vpn-gateway-vnet-vnet-rm-ps)

本文将引导你使用资源管理器部署模型来完成相关步骤。如果你想要适合此配置的其他部署模型，请使用上述选项卡选择所需文章。

目前，我们还没有为使用资源管理器部署方法（存在于其他订阅中）创建的虚拟网络的 VNet 到 VNet 连接提供解决方案。我们的团队目前正在努力寻求解决方案，预计今年年底或明年很早的时候将会提供相关步骤。解决方案发布时，本文会提供相关步骤。以下步骤适用于同一订阅中的 VNet。

**关于 Azure 部署模型**

[AZURE.INCLUDE [vpn-gateway-clasic-rm](../includes/vpn-gateway-classic-rm-include.md)]
	
- 如果你的虚拟网络是使用经典部署模型创建的，请参阅[创建 VNet 到 VNet 连接](/documentation/articles/virtual-networks-configure-vnet-to-vnet-connection)。经典部署模型还支持连接不同订阅中存在的 VNet。
	
- 如果你要将以经典部署模型创建的虚拟网络连接到使用 Azure 资源管理器模型创建的虚拟网络，请参阅[将经典 VNet 连接到新 VNet](/documentation/articles/virtual-networks-arm-asm-s2s)。

## 关于 VNet 到 VNet 的连接

将虚拟网络连接到虚拟网络（VNet 到 VNet）非常类似于将 VNet 连接到本地站点位置。这两种连接类型都使用 VPN 网关来提供使用 IPsec/IKE 的安全隧道。你连接的 VNet 可位于不同的区域中。你甚至可以将 VNet 到 VNet 通信与多站点配置组合使用。这样，便可以建立将跨界连接与虚拟网络间连接相结合的网络拓扑，如下图所示。

![VNet 到 VNet 连接示意图](./media/virtual-networks-configure-vnet-to-vnet-connection/IC727360.png)
 
### 为什么要连接虚拟网络？

你可能会出于以下原因而连接虚拟网络：

- **跨区域地域冗余和地域存在**
	- 你可以使用安全连接设置自己的地域复制或同步，而无需借助于面向 Internet 的终结点。
	- 使用 Azure 负载平衡器和 Microsoft 或第三方群集技术，你可以设置支持跨多个 Azure 区域实现地域冗余的高可用性工作负荷。一个重要的示例就是对分布在多个 Azure 区域中的可用性组设置 SQL Always On。

- **具有强大隔离边界的区域多层应用程序**
	- 在同一区域中，你可以设置具有多个虚拟网络的多层应用程序，这些虚拟网络相互连接在一起，但同时又能保持强大的隔离性，而且还能进行安全的层间通信。


### VNet 到 VNet 常见问题

- 虚拟网络可以在相同或不同的 Azure 区域（位置）中。

- 云服务或负载平衡终结点不能跨虚拟网络，即使它们连接在一起，也是如此。

- 将多个 Azure 虚拟网络连接在一起不需要任何本地 VPN 网关，除非需要跨界连接。

- VNet 到 VNet 通信支持连接虚拟网络。它不支持连接不在虚拟网络中的虚拟机或云服务。

- VNet 到 VNet 连接需要基于路由的 VPN 网关（以前称为动态路由网关）。

- 虚拟网络连接可与多站点 VPN 同时使用，最多可以将一个虚拟网络 VPN 网关的 10 个 VPN 隧道连接到其他虚拟网络或本地站点。

- 虚拟网络和本地网络站点的地址空间不得重叠。地址空间重叠将会导致创建虚拟网络失败。

- 不支持一对虚拟网络之间存在冗余隧道。

- 虚拟网络的所有 VPN 隧道共享 Azure VPN 网关上的可用带宽，以及 Azure 中的相同 VPN 网关运行时间 SLA。

- VNet 到 VNet 流量将会流经 Azure 主干。

## 配置 VNet 到 VNet 的连接

本文将指导你连接两个虚拟网络：VNet1 和 VNet2。你需要具有丰富的网络知识，以便能够改用与你的网络设计要求相符的 IP 地址范围。

![将 VNet 连接到 VNet](./media/virtual-networks-configure-vnet-to-vnet-connection/IC727361.png)

### 开始之前


- 确保你拥有 Azure 订阅。如果你还没有 Azure 订阅，可以激活 [MSDN 订户权益](http://azure.microsoft.com/zh-cn/pricing/member-offers/msdn-benefits-details)或注册获取[试用版](/pricing/1rmb-trial)。

- 安装 PowerShell 模块。你需要最新版本的 Azure 资源管理器 PowerShell cmdlet 来配置你的连接。[AZURE.INCLUDE [vpn-gateway-ps-rm-howto](../includes/vpn-gateway-ps-rm-howto-include.md)]


## 步骤 1 - 规划 IP 地址范围


必须确定要用于配置网络配置的范围。从 VNet1 的角度讲，VNet2 只不过是 Azure 平台中定义的另一个 VPN 连接。从 VNet2 的角度讲，VNet1 也只不过是另一个 VPN 连接。请记住，必须确保没有任何 VNet 范围或本地网络范围存在任何形式的重叠。


以下步骤将创建两个虚拟网络，以及它们各自的网关子网和配置。然后创建两个 VNet 之间的 VPN 网关连接。

在此练习中，我们对 VNet 使用以下值：

VNet1 的值：

- 虚拟网络名称 = VNet1
- 资源组 = testrg1
- 地址空间 = 10.1.0.0/16 
- 区域 = 中国东部
- 网关子网 = 10.1.0.0/28
- Subnet1 = 10.1.1.0/28

VNet2 的值：

- 虚拟网络名称 = VNet2
- 资源组 = testrg2
- 地址空间 = 10.2.0.0/16
- 区域 = 中国东部
- 网关子网 = 10.2.0.0/28
- Subnet1 = 10.2.1.0/28

## 步骤 2 - 连接到订阅 

确保切换到 PowerShell 模式，以便使用资源管理器 cmdlet。有关详细信息，请参阅[将 Windows PowerShell 与资源管理器配合使用](/documentation/articles/powershell-azure-resource-manager)。

打开 PowerShell 控制台并连接到你的帐户。使用下面的示例来帮助你连接：

	Login-AzureRmAccount –Environment (Get-AzureRmEnvironment –Name AzureChinaCloud)

检查该帐户的订阅。

	Get-AzureRmSubscription 

指定要使用的订阅。

	Select-AzureRmSubscription -Subscriptionid "GUID of subscription"


## 步骤 3 - 创建虚拟网络


使用下面的示例创建一个虚拟网络和网关子网。替换为您自己的值。在此示例中，我们将创建 VNet1。稍后你要重复这些步骤以创建 VNet2。

首先创建一个资源组。

	New-AzureRmResourceGroup -Name testrg1 -Location 'China East'

接下来，创建您的虚拟网络。以下示例创建一个名为 *VNet1* 的虚拟网络和两个子网，其中一个名为 *GatewaySubnet*，另一个名为 *Subnet1*。特意创建一个名为 *GatewaySubnet* 的子网非常重要。如果您以其他名称为其命名，则您的连接配置将会失败。在以下示例中，网关子网使用 /28。你可以选择使用最小的 /29 网关子网。请注意，某些功能（例如 ExpressRoute/站点到站点并存连接）需要较大的 /27 网关子网，因此你可能需要创建自己的网关子网来接受未来可以使用的其他功能。

 	$subnet = New-AzureRmVirtualNetworkSubnetConfig -Name 'GatewaySubnet' -AddressPrefix 10.1.0.0/28
	$subnet1 = New-AzureRmVirtualNetworkSubnetConfig -Name 'Subnet1' -AddressPrefix '10.1.1.0/28'
	New-AzureRmVirtualNetwork -Name VNet1 -ResourceGroupName testrg1 -Location 'China East' -AddressPrefix 10.1.0.0/16
	-Subnet $subnet, $subnet1

## 步骤 4 - 请求公共 IP 地址


接下来，需要请求一个公共 IP 地址，以分配给要为 VNet 创建的网关。你无法指定要使用的 IP 地址；它动态分配到你的网关。后面的配置部分将使用此 IP 地址。

使用以下示例。此地址的分配方法必须是“动态”。


	$gwpip= New-AzureRmPublicIpAddress -Name gwpip1 -ResourceGroupName testrg1 -Location 'China East' -AllocationMethod Dynamic

## 步骤 5 - 创建网关配置


网关配置定义要使用的子网和公共 IP 地址。使用下面的示例创建你的网关配置。


	$vnet = Get-AzureRmVirtualNetwork -Name VNet1 -ResourceGroupName testrg1
	$subnet = Get-AzureRmVirtualNetworkSubnetConfig -Name 'GatewaySubnet' -VirtualNetwork $vnet
	$gwipconfig = New-AzureRmVirtualNetworkGatewayIpConfig -Name gwipconfig1 -SubnetId $subnet.Id -PublicIpAddressId $gwpip.Id 

## 步骤 6 - 创建网关


在此步骤中，你将为 VNet 创建虚拟网络网关。VNet 到 VNet 配置需要基于路由的 VPN 类型。创建网关可能需要一段时间，请保持耐心。

	New-AzureRmVirtualNetworkGateway -Name vnetgw1 -ResourceGroupName testrg1 -Location 'China East' -IpConfigurations $gwipconfig -GatewayType Vpn -VpnType RouteBased

## 步骤 7 - 创建 VNet2


配置 VNet1 后，请重复上述步骤，以配置 VNet2 及其网关配置。完成这两个 VNet 及其各自网关的配置后，请转到**步骤 8。连接网关**。

## 步骤 8 - 连接网关

此步骤将创建两个虚拟网络网关之间的 VPN 网关连接。示例中引用了共享密钥。你可以对共享密钥使用自己的值。共享密钥必须与两个配置匹配，这一点非常重要。

请注意，创建连接需要一段时间才能完成。

**VNet1 到 VNet2**
    
    $vnetgw1 = Get-AzureRmVirtualNetworkGateway -Name vnetgw1 -ResourceGroupName testrg1
    $vnetgw2 = Get-AzureRmVirtualNetworkGateway -Name vnetgw2 -ResourceGroupName testrg2
    
    New-AzureRmVirtualNetworkGatewayConnection -Name conn1 -ResourceGroupName testrg1 -VirtualNetworkGateway1 $vnetgw1 -VirtualNetworkGateway2 $vnetgw2 -Location 'China East' -ConnectionType Vnet2Vnet -SharedKey 'abc123'


**VNet2 到 VNet1**
    
    $vnetgw1 = Get-AzureRmVirtualNetworkGateway -Name vnetgw2 -ResourceGroupName testrg2
    $vnetgw2 = Get-AzureRmVirtualNetworkGateway -Name vnetgw1 -ResourceGroupName testrg1
    
    New-AzureRmVirtualNetworkGatewayConnection -Name conn2 -ResourceGroupName testrg2 -VirtualNetworkGateway1 $vnetgw1 -VirtualNetworkGateway2 $vnetgw2 -Location 'China East' -ConnectionType Vnet2Vnet -SharedKey 'abc123'
    

几分钟后，连接应建立完毕。请注意，此时经典门户还不会显示使用 Azure 资源管理器创建的网关和连接。

## 验证连接

此时，通过资源管理器创建的 VPN 连接不在经典门户中显示。但是，可以通过使用 *Get-AzureRmVirtualNetworkGatewayConnection –Debug* 验证你的连接是否成功。将来，我们将为此提供一个 cmdlet 以及在经典门户中查看你的连接的功能。

你可以使用下面的 cmdlet 示例。请确保更改相关值，使之与你要验证的每个连接相匹配。出现提示时，选择 *A* 以运行所有。

	Get-AzureRmVirtualNetworkGatewayConnection -Name vnet2connection -ResourceGroupName vnet2vnetrg -Debug 

 cmdlet 运行完毕后，通过滚动查看该值。在下面的示例中，连接状态显示为“已连接”，且你可以看到入口和出口字节数。

	Body:
	{
	  "name": "Vnet2Connection",
	  "id": "/subscriptions/a932c0e6-b5cb-4e68-b23d-5064372c8a3cdef/resourceGroups/Vnet2VnetRG/providers/Microsoft.Network/connections/Vnet2Connection",
	  "properties": {
	    "provisioningState": "Succeeded",
	    "resourceGuid": "3c0bc049-860d-46ea-9909-e08098680a8bdef",
	    "virtualNetworkGateway1": {
	      "id": "/subscriptions/a932c0e6-b5cb-4e68-b23d-5064372c8a3cdef/resourceGroups/Vnet2VnetRG/providers/Microsoft.Network/virtualNetworkGateways/Vnet2Gw"
	    },
	    "virtualNetworkGateway2": {
	      "id": "/subscriptions/a932c0e6-b5cb-4e68-b23d-5064372c8a3cdef/resourceGroups/Vnet2VnetRG/providers/Microsoft.Network/virtualNetworkGateways/Vnet1Gw"
	    },
	    "connectionType": "Vnet2Vnet",
	    "routingWeight": 3,
	    "sharedKey": "abc123",
	    "connectionStatus": "Connected",
	    "ingressBytesTransferred": 3359044,
	    "egressBytesTransferred": 4142451
	  }
	} 

[AZURE.INCLUDE [vpn-gateway-no-nsg-include](../includes/vpn-gateway-no-nsg-include.md)]

## 连接现有的 VNet

如果你已在 Azure 资源管理器模式下创建了虚拟网络，并且想要连接这些虚拟网络，请确认以下事项：

- 每个 VNet 的网关子网至少设置为 /29 或更大的值。
- 虚拟网络的地址范围不重叠。
- 你的多个 VNet 是在同一订阅中。

如果需要将网关子网添加 VNet，请使用以下示例，并将其中的值替换为你自己的值。请务必将网关子网命名为“GatewaySubnet”。如果以其他名称为其命名，则您的 VPN 配置将无法按预期工作。

	
	$vnet = Get-AzureRmVirtualNetwork -ResourceGroupName testrg -Name testvnet
	Add-AzureRmVirtualNetworkSubnetConfig -Name 'GatewaySubnet' -AddressPrefix 10.0.3.0/28 -VirtualNetwork $vnet
	Set-AzureRmVirtualNetwork -VirtualNetwork $vnet

确认网关子网的配置正确无误后，请转到**步骤 4.请求公共 IP 地址**并根据步骤操作。


## 后续步骤

连接完成后，即可将虚拟机添加到虚拟网络。请参阅[创建虚拟机](/documentation/articles/virtual-machines-windows-tutorial)以获取相关步骤。

<!---HONumber=Mooncake_0104_2016-->
