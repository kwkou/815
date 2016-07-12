<properties 
   pageTitle="使用 Resource Manager 为 VPN 网关配置强制隧道 | Azure"
   description="如果你的虚拟网络具有跨界 VPN 网关，你可以将全部 Internet 绑定流量重定向或“强制”返回到本地位置。本文适用于 Resource Manager 部署模型。"
   services="vpn-gateway"
   documentationCenter="na"
   authors="cherylmc"
   manager="carmonm"
   editor=""
   tags="azure-resource-manager"/>
<tags 
   ms.service="vpn-gateway"
   ms.date="04/12/2016"
   wacn.date="05/10/2016" />

# 使用 PowerShell 和 Azure 资源管理器配置强制隧道

> [AZURE.SELECTOR]
- [PowerShell - 服务管理](/documentation/articles/vpn-gateway-about-forced-tunneling/)
- [PowerShell - 资源管理器](/documentation/articles/vpn-gateway-forced-tunneling-rm/)

本文适用于通过 Azure 资源管理器部署模型创建的 VNet 和 VPN 网关。

**关于 Azure 部署模型**

[AZURE.INCLUDE [vpn-gateway-clasic-rm](../includes/vpn-gateway-classic-rm-include.md)]

**强制隧道的部署模型和工具**

[AZURE.INCLUDE [vpn-gateway-table-forced-tunneling](../includes/vpn-gateway-table-forcedtunnel-include.md)]


## 关于强制隧道

借助强制隧道，您可以通过站点到站点 VPN 隧道，将全部 Internet 绑定流量重定向或“强制”返回到本地位置，以进行检查和审核。这是很多企业 IT 策略的关键安全要求。没有强制隧道，来自 Azure 中虚拟机的 Internet 绑定流量会始终通过 Azure 网络基础设施直接连接到 Internet。没有该选项，您无法对流量进行检查或审核。未经授权的 Internet 访问可能会导致信息泄漏或其他类型的安全漏洞。

下图说明了强制隧道的工作方式。

![强制隧道](./media/vpn-gateway-forced-tunneling-rm/forced-tunnel.png)

在上面的示例中，前端子网没有使用强制隧道。前端子网中的工作负载可以继续直接接受并响应来自 Internet 的客户请求。中间层和后端子网会使用强制隧道。任何从这两个子网到 Internet 的出站连接将通过一个 S2S VPN 隧道重定向或强制返回到本地站点。这样，在继续支持所需的多层服务体系结构的同时，你可以限制并检查来自虚拟机或 Azure 云服务的 Internet 访问。如果在虚拟网络中没有面向 Internet 的工作负载，您还可以选择在整个虚拟网络应用强制隧道连接。

## 要求和注意事项

在 Azure 中，通过虚拟网络用户定义路由配置强制隧道。将流量重定向到本地站点，这是 Azure VPN 网关的默认路由。有关用户定义路由和虚拟网络的详细信息，请参阅[用户定义路由和 IP 转发](/documentation/articles/virtual-networks-udr-overview/)。

- 每个虚拟网络子网具有内置的系统路由表。系统路由表具有下面的 3 组路由：

	- **本地 VNet 路由：**直接路由到同一个虚拟网络中的目标虚拟机
	
	- **本地路由：**路由到 Azure VPN 网关
	
	- **默认路由：**直接路由到 Internet。请注意，如果要将数据包发送到不包含在前面两个路由中的专用 IP 地址，数据包将被删除。

-  此过程使用用户定义路由 (UDR) 来创建路由表以添加默认路由，然后将路由表关联到 VNet 子网，在这些子网启用强制隧道。

- 强制隧道必须关联到具有基于路由的 VPN 网关的 VNet。您需要在连接到虚拟网络的跨界本地站点中，设置一个“默认站点”。

- 请注意，ExpressRoute 强制隧道不是通过此机制配置的，而是通过 ExpressRoute BGP 对等会话播发默认路由来启用的。有关详细信息，请参阅 [ExpressRoute 文档](/documentation/services/expressroute)。

## 配置强制隧道

### 配置概述

下面的过程将帮助你创建资源组和 VNet。然后，你将创建 VPN 网关，并配置强制隧道。

在本示例中，虚拟网络“MultiTier-VNet”具有 3 个子网：前端、中间层和后端子网，具有 4 个跨界连接：一个 DefaultSiteHQ 和 3 个分支。以下过程步骤将 DefaultSiteHQ 设置为使用强制隧道的默认站点连接，并将中间层和后端子网配置为使用强制隧道。

	
### 开始之前

在开始配置之前，请确认你具有以下各项。

- Azure 订阅。如果你还没有 Azure 订阅，你可以注册一个[试用版](/pricing/1rmb-trial)。

- 你需要安装最新版本的 Azure Resource Manager PowerShell cmdlet（1.0 或更高）。有关安装 PowerShell cmdlet 的详细信息，请参阅[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/)。


### 配置步骤

1. 在 PowerShell 控制台中，登录到你的 Azure 帐户。该 cmdlet 将提示你提供 Azure 帐户的登录凭据。登录后它会下载你的帐户设置，以便这些信息可供 Azure PowerShell 使用。

		Login-AzureRmAccount -EnvironmentName AzureChinaCloud

2. 获取 Azure 订阅的列表。

		Get-AzureRmSubscription

2. 指定要使用的订阅。

		Select-AzureRmSubscription -SubscriptionName "Replace_with_your_subscription_name"
		
3. 创建资源组。

		New-AzureRmResourceGroup -Name "ForcedTunneling" -Location "China North"

4. 创建虚拟网络并指定子网。

		$s1 = New-AzureRmVirtualNetworkSubnetConfig -Name "Frontend" -AddressPrefix "10.1.0.0/24"
		$s2 = New-AzureRmVirtualNetworkSubnetConfig -Name "Midtier" -AddressPrefix "10.1.1.0/24"
		$s3 = New-AzureRmVirtualNetworkSubnetConfig -Name "Backend" -AddressPrefix "10.1.2.0/24"
		$s4 = New-AzureRmVirtualNetworkSubnetConfig -Name "GatewaySubnet" -AddressPrefix "10.1.200.0/28"
		$vnet = New-AzureRmVirtualNetwork -Name "MultiTier-VNet" -Location "China North" -ResourceGroupName "ForcedTunneling" -AddressPrefix "10.1.0.0/16" -Subnet $s1,$s2,$s3,$s4

5. 创建本地网络网关。

		$lng1 = New-AzureRmLocalNetworkGateway -Name "DefaultSiteHQ" -ResourceGroupName "ForcedTunneling" -Location "China North" -GatewayIpAddress "111.111.111.111" -AddressPrefix "192.168.1.0/24"
		$lng2 = New-AzureRmLocalNetworkGateway -Name "Branch1" -ResourceGroupName "ForcedTunneling" -Location "China North" -GatewayIpAddress "111.111.111.112" -AddressPrefix "192.168.2.0/24"
		$lng3 = New-AzureRmLocalNetworkGateway -Name "Branch2" -ResourceGroupName "ForcedTunneling" -Location "China North" -GatewayIpAddress "111.111.111.113" -AddressPrefix "192.168.3.0/24"
		$lng4 = New-AzureRmLocalNetworkGateway -Name "Branch3" -ResourceGroupName "ForcedTunneling" -Location "China North" -GatewayIpAddress "111.111.111.114" -AddressPrefix "192.168.4.0/24"
		
6. 创建路由表和路由规则。

		New-AzureRmRouteTable –Name "MyRouteTable" -ResourceGroupName "ForcedTunneling" –Location "China North"
		$rt = Get-AzureRmRouteTable –Name "MyRouteTable" -ResourceGroupName "ForcedTunneling" 
		Add-AzureRmRouteConfig -Name "DefaultRoute" -AddressPrefix "0.0.0.0/0" -NextHopType VirtualNetworkGateway -RouteTable $rt
		Set-AzureRmRouteTable -RouteTable $rt


6. 将路由表与“中间层”子网和“后端”子网关联起来。

		$vnet = Get-AzureRmVirtualNetwork -Name "MultiTier-Vnet" -ResourceGroupName "ForcedTunneling"
		Set-AzureRmVirtualNetworkSubnetConfig -Name "MidTier" -VirtualNetwork $vnet -AddressPrefix "10.1.1.0/24" -RouteTable $rt
		Set-AzureRmVirtualNetworkSubnetConfig -Name "Backend" -VirtualNetwork $vnet -AddressPrefix "10.1.2.0/24" -RouteTable $rt
		Set-AzureRmVirtualNetwork -VirtualNetwork $vnet

9. 创建默认站点的网关。此步骤需要一些时间才能完成，有时需要 20 分钟或更长时间，因为你需要创建和配置网关。虽然看起来只是控制台中有几个 cmdlet，但后台有许多进程在执行。请注意，GatewayDefaultSite 是 cmdlet 参数，该参数是强制路由配置所不可或缺的，不能跳过它。它仅在 PowerShell 1.0 或更高版本中提供。

		$pip = New-AzureRmPublicIpAddress -Name "GatewayIP" -ResourceGroupName "ForcedTunneling" -Location "China North" -AllocationMethod Dynamic
		$gwsubnet = Get-AzureRmVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet
		$ipconfig = New-AzureRmVirtualNetworkGatewayIpConfig -Name "gwIpConfig" -SubnetId $gwsubnet.Id -PublicIpAddressId $pip.Id
		New-AzureRmVirtualNetworkGateway -Name "Gateway1" -ResourceGroupName "ForcedTunneling" -Location "China North" -IpConfigurations $ipconfig -GatewayType Vpn -VpnType RouteBased -GatewayDefaultSite $lng1 -EnableBgp $false

10. 建立站点到站点 VPN 连接

		$gateway = Get-AzureRmVirtualNetworkGateway -Name "Gateway1" -ResourceGroupName "ForcedTunneling"
		$lng1 = Get-AzureRmLocalNetworkGateway -Name "DefaultSiteHQ" -ResourceGroupName "ForcedTunneling" 
		$lng2 = Get-AzureRmLocalNetworkGateway -Name "Branch1" -ResourceGroupName "ForcedTunneling" 
		$lng3 = Get-AzureRmLocalNetworkGateway -Name "Branch2" -ResourceGroupName "ForcedTunneling" 
		$lng4 = Get-AzureRmLocalNetworkGateway -Name "Branch3" -ResourceGroupName "ForcedTunneling" 

		New-AzureRmVirtualNetworkGatewayConnection -Name "Connection1" -ResourceGroupName "ForcedTunneling" -Location "China North" -VirtualNetworkGateway1 $gateway -LocalNetworkGateway2 $lng1 -ConnectionType IPsec -SharedKey "preSharedKey"
		New-AzureRmVirtualNetworkGatewayConnection -Name "Connection2" -ResourceGroupName "ForcedTunneling" -Location "China North" -VirtualNetworkGateway1 $gateway -LocalNetworkGateway2 $lng2 -ConnectionType IPsec -SharedKey "preSharedKey"
		New-AzureRmVirtualNetworkGatewayConnection -Name "Connection3" -ResourceGroupName "ForcedTunneling" -Location "China North" -VirtualNetworkGateway1 $gateway -LocalNetworkGateway2 $lng3 -ConnectionType IPsec -SharedKey "preSharedKey"
		New-AzureRmVirtualNetworkGatewayConnection -Name "Connection4" -ResourceGroupName "ForcedTunneling" -Location "China North" -VirtualNetworkGateway1 $gateway -LocalNetworkGateway2 $lng4 -ConnectionType IPsec -SharedKey "preSharedKey"

		Get-AzureRmVirtualNetworkGatewayConnection -Name "Connection1" -ResourceGroupName "ForcedTunneling"
		




<!---HONumber=Mooncake_0425_2016-->
