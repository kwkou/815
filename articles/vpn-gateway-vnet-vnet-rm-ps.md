<properties
   pageTitle="使用 Azure 资源管理器和 PowerShell 创建 VNet 到 VNet 连接 | Windows Azure"
   description="本文指导你使用 Azure 资源管理器和 PowerShell 将虚拟网络连接在一起"
   services="vpn-gateway"
   documentationCenter="na"
   authors="cherylmc"
   manager="carolz"
   editor=""/>

<tags
   ms.service="vpn-gateway"
   ms.date="08/20/2015"
   wacn.date="10/08/2015"/>

# 使用 Azure 资源管理器和 PowerShell 配置 VNet 到 VNet 连接

> [AZURE.SELECTOR]
- [Azure Portal](virtual-networks-configure-vnet-to-vnet-connection.md)
- [PowerShell - Azure Resource Manager](vpn-gateway-vnet-vnet-rm-ps.md)


将虚拟网络连接到虚拟网络（VNet 到 VNet）非常类似于将 VNet 连接到本地站点位置。这两种连接类型都使用 VPN 网关来提供使用 IPsec/IKE 的安全隧道。你连接的 VNet 可位于不同的区域中。你甚至可以将 VNet 到 VNet 通信与多站点配置组合使用。这样，便可以建立将跨界连接与虚拟网络间连接相结合的网络拓扑，如下图所示。


![VNet 到 VNet 连接示意图](./media/virtual-networks-configure-vnet-to-vnet-connection/IC727360.png)

 

>[AZURE.NOTE]Azure 目前有两种部署模式：经典部署模式和 Azure 资源管理器部署模式。配置 cmdlet 和步骤因部署模式而有所不同。本主题指导你连接使用 Azure 资源管理器模式创建的虚拟网络。如果你要使用经典部署模式创建 VNet 到 VNet 连接，请参阅[使用 Azure 门户配置 VNet 到 VNet 连接](virtual-networks-configure-vnet-to-vnet-connection.md)。如果你要将以传统模式创建的虚拟网络连接到在 Azure 资源管理器中创建的虚拟网络，请参阅[将经典 VNet 连接到新 VNet](../virtual-network/virtual-networks-arm-asm-s2s.md)。

## 为什么要连接虚拟网络？

你可能会出于以下原因而连接虚拟网络：

- **跨区域地域冗余和地域存在**
	- 你可以使用安全连接设置自己的地域复制或同步，而无需借助于面向 Internet 的终结点。
	- 使用 Azure 负载平衡器和 Microsoft 或第三方群集技术，你可以设置支持跨多个 Azure 区域实现地域冗余的高可用性工作负荷。一个重要的示例就是对分布在多个 Azure 区域中的可用性组设置 SQL Always On。

- **具有强大隔离边界的区域多层应用程序**
	- 在同一区域中，你可以设置具有多个虚拟网络的多层应用程序，这些虚拟网络相互连接在一起，但同时又能保持强大的隔离性，而且还能进行安全的层间通信。



### 注意事项

本文将指导你连接两个虚拟网络：VNet1 和 VNet2。你需要具有丰富的网络知识，以便能够改用与你的网络设计要求相符的 IP 地址范围。


![将 VNet 连接到 VNet](./media/virtual-networks-configure-vnet-to-vnet-connection/IC727361.png)



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


## 开始之前


在开始之前，请验证您具有以下各项：


- 最新版本的 Azure PowerShell cmdlet。可以从[下载页面](http://azure.microsoft.com/downloads/)的 Windows PowerShell 部分下载并安装最新版本。 
- Azure 订阅。如果你还没有 Azure 订阅，可以激活 [MSDN 订户权益](http://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/)或注册获取[免费试用版](http://azure.microsoft.com/pricing/free-trial/)。
- 如果你已创建虚拟网络，请参阅[连接现有的 VNet](#connecting-existing-vnets)。
	


创建和配置 VNet 到 VNet 连接涉及到多个步骤。请按以下列出顺序配置每个部分。


1. [规划 IP 地址范围](#plan-your-ip-address-ranges)
2. [连接到订阅](#connect-to-your-subscription)
3. [创建虚拟网络](#create-a-virtual-network)
4. [为网关请求一个公共 IP 地址](#request-a-public-ip-address)
5. [创建网关配置](#create-the-gateway-configuration)
6. [创建网关](#create-the-gateway)
7. [重复步骤以配置 VNet2](#create-vnet2)
8. [连接 VPN 网关](#connect-the-gateways)


## 规划 IP 地址范围

**步骤 1**

必须确定要用于配置网络配置的范围。从 VNet1 的角度讲，VNet2 只不过是 Azure 平台中定义的另一个 VPN 连接。从 VNet2 的角度讲，VNet1 也只不过是另一个 VPN 连接。请记住，必须确保没有任何 VNet 范围或本地网络范围存在任何形式的重叠。


以下步骤将创建两个虚拟网络，以及它们各自的网关子网和配置。然后创建两个 VNet 之间的 VPN 网关连接。

在此练习中，我们对 VNet 使用以下值：

VNet1 的值：

- 虚拟网络名称 = VNet1
- 资源组 = testrg1
- 地址空间 = 10.1.0.0/16 
- 区域 = 美国西部
- 网关子网 = 10.1.0.0/28
- Subnet1 = 10.1.1.0/28

VNet2 的值：

- 虚拟网络名称 = VNet2
- 资源组 = testrg2
- 地址空间 = 10.2.0.0/16
- 区域 = 日本东部
- 网关子网 = 10.2.0.0/28
- Subnet1 = 10.2.1.0/28

## 连接到订阅 

**步骤 2**


打开 PowerShell 控制台并切换到 Azure 资源管理器模式。使用下面的示例来帮助您连接：

		Add-AzureAccount

使用 Select-AzureSubscription 连接到您要使用的订阅。

		Select-AzureSubscription "yoursubscription"

接下来，切换到 ARM 模式。这将切换模式以允许您使用 ARM cmdlet。

		Switch-AzureMode -Name AzureResourceManager

## 创建虚拟网络

**步骤 3**


使用下面的示例创建一个虚拟网络和网关子网。替换为您自己的值。在此示例中，我们将创建 VNet1。稍后你要重复这些步骤以创建 VNet2。

首先创建一个资源组。

			New-AzureResourceGroup -Name testrg1 -Location 'West US'

接下来，创建您的虚拟网络。以下示例创建一个名为 *VNet1* 的虚拟网络和两个子网，其中一个名为 *GatewaySubnet*，另一个名为 *Subnet1*。特意创建一个名为 *GatewaySubnet* 的子网非常重要。如果您以其他名称为其命名，则您的连接配置将会失败。在以下示例中，网关子网使用 /28。你可以选择使用最小的 /29 网关子网。请注意，某些功能（例如 ExpressRoute/站点到站点并存连接）需要较大的 /27 网关子网，因此你可能需要创建自己的网关子网来接受未来可以使用的其他功能。

 		$subnet = New-AzureVirtualNetworkSubnetConfig -Name 'GatewaySubnet' -AddressPrefix 10.1.0.0/28
		$subnet1 = New-AzureVirtualNetworkSubnetConfig -Name 'Subnet1' -AddressPrefix '10.1.1.0/28'
		New-AzurevirtualNetwork -Name VNet1 -ResourceGroupName testrg1 -Location 'West US' -AddressPrefix 10.1.0.0/16 -Subnet $subnet, $subnet1

## 请求公共 IP 地址

**步骤 4**

接下来，需要请求一个公共 IP 地址，以分配给要为 VNet 创建的网关。你无法指定要使用的 IP 地址；它动态分配到你的网关。后面的配置部分将使用此 IP 地址。

使用以下示例。此地址的分配方法必须是“动态”。


		$gwpip= New-AzurePublicIpAddress -Name gwpip1 -ResourceGroupName testrg1 -Location 'West US' -AllocationMethod Dynamic


## 创建网关配置

**步骤 5**

网关配置定义要使用的子网和公共 IP 地址。使用下面的示例创建您的网关配置。


		$vnet = Get-AzureVirtualNetwork -Name VNet1 -ResourceGroupName testrg1
		$subnet = Get-AzureVirtualNetworkSubnetConfig -Name 'GatewaySubnet' -VirtualNetwork $vnet
		$gwipconfig = New-AzureVirtualNetworkGatewayIpConfig -Name gwipconfig1 -SubnetId $subnet.Id -PublicIpAddressId $gwpip.Id 


## 创建网关

**步骤 6**

在此步骤中，你将为 VNet 创建虚拟网络网关。VNet 到 VNet 配置需要基于路由的 VPN 类型。创建网关可能需要一段时间，请保持耐心。

		New-AzureVirtualNetworkGateway -Name vnetgw1 -ResourceGroupName testrg1 -Location 'West US' -IpConfigurations $gwipconfig -GatewayType Vpn -VpnType RouteBased

## 创建 VNet2

**步骤 7**

配置 VNet1 后，请重复上述步骤，以配置 VNet2 及其网关配置。完成这两个 VNet 及其各自网关的配置后，请转到[连接网关](#connect-the-gateways)。


## 连接网关

**步骤 8**

此步骤将创建两个虚拟网络网关之间的 VPN 网关连接。示例中引用了共享密钥。你可以对共享密钥使用自己的值。共享密钥必须与两个配置匹配，这一点非常重要。

请注意，创建连接需要一段时间才能完成。

**VNet1 到 VNet2**
    
    $vnetgw1 = Get-AzureVirtualNetworkGateway -Name vnetgw1 -ResourceGroupName testrg1
    $vnetgw2 = Get-AzureVirtualNetworkGateway -Name vnetgw2 -ResourceGroupName testrg2
    
    New-AzureVirtualNetworkGatewayConnection -Name conn1 -ResourceGroupName testrg1 -VirtualNetworkGateway1 $vnetgw1 -VirtualNetworkGateway2 $vnetgw2 -Location 'West US' -ConnectionType Vnet2Vnet -SharedKey 'abc123'


**VNet2 到 VNet1**
    
    $vnetgw1 = Get-AzureVirtualNetworkGateway -Name vnetgw2 -ResourceGroupName testrg2
    $vnetgw2 = Get-AzureVirtualNetworkGateway -Name vnetgw1 -ResourceGroupName testrg1
    
    New-AzureVirtualNetworkGatewayConnection -Name conn2 -ResourceGroupName testrg2 -VirtualNetworkGateway1 $vnetgw1 -VirtualNetworkGateway2 $vnetgw2 -Location 'Japan East' -ConnectionType Vnet2Vnet -SharedKey 'abc123'
    


几分钟后，连接应建立完毕。请注意，此时预览门户还不会显示使用 Azure 资源管理器创建的网关和连接。　

## 连接现有的 VNet

如果你已在 Azure 资源管理器模式下创建了虚拟网络，并且想要连接这些虚拟网络，请确认以下事项：

- 每个 VNet 的网关子网至少设置为 /29 或更大的值。
- 虚拟网络的地址范围不重叠。

如果需要将网关子网添加 VNet，请使用以下示例，并将其中的值替换为你自己的值。请务必将网关子网命名为“GatewaySubnet”。如果以其他名称为其命名，则您的 VPN 配置将无法按预期工作。

	
		$vnet = Get-AzureVirtualNetwork -ResourceGroupName testrg -Name testvnet
		Add-AzureVirtualNetworkSubnetConfig -Name 'GatewaySubnet' -AddressPrefix 10.0.3.0/28 -VirtualNetwork $vnet
		Set-AzureVirtualNetwork -VirtualNetwork $vnet

确认网关子网的配置正确无误后，请转到[请求公共 IP 地址](#request-a-public-ip-address)并根据步骤操作。

## 后续步骤

你可以将虚拟机添加到虚拟网络。[创建虚拟机](../virtual-machines/virtual-machines-windows-tutorial)。

<!---HONumber=71-->