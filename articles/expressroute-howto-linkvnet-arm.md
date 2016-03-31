<properties 
   pageTitle="将虚拟网络链接到 ExpressRoute 线路 | Microsoft Azure"
   description="本文档概述了如何将虚拟网络 (VNet) 链接到 ExpressRoute 线路。"
   services="expressroute"
   documentationCenter="na"
   authors="ganesr"
   manager="carolz"
   editor=""
   tags="azure-service-management"/>
<tags 
   ms.service="expressroute"
   ms.date="01/16/2016"
   wacn.date="03/17/2016" />

# 将虚拟网络链接到 ExpressRoute 线路

> [AZURE.SELECTOR]
- [PowerShell - 经典](/documentation/articles/expressroute-howto-linkvnet-classic)
- [PowerShell - 资源管理器](/documentation/articles/expressroute-howto-linkvnet-arm)
- [模板 - 资源管理器](https://github.com/Azure/azure-quickstart-templates/tree/ecad62c231848ace2fbdc36cbe3dc04a96edd58c/301-expressroute-circuit-vnet-connection)

本文概述了如何将虚拟网络 (VNet) 链接到 ExpressRoute 线路。虚拟网络可以在同一个订阅中，也可以属于另一个订阅。本文适用于使用资源管理器部署模型部署的 VNet。如果你想要链接使用经典部署模型部署的虚拟网络，请参阅[将虚拟网络链接到 ExpressRoute 线路](/documentation/articles/expressroute-howto-linkvnet-classic)。

[AZURE.INCLUDE [vpn-gateway-sm-rm](../includes/vpn-gateway-sm-rm-include.md)]

## 配置先决条件

- 你将需要最新版本的 Azure PowerShell 模块（版本 1.0 或更高版本）。 
- 在开始配置之前，请务必查看[先决条件](/documentation/articles/expressroute-prerequisites)页、[路由要求](/documentation/articles/expressroute-routing)页和[工作流](/documentation/articles/expressroute-workflows)页。
- 你必须有一个活动的 ExpressRoute 线路。 
	- 请按说明[创建 ExpressRoute 线路](/documentation/articles/expressroute-howto-circuit-arm)，并通过连接提供商启用该线路。 
	- 请确保为线路配置 Azure 专用对等互连。如需路由说明，请参阅[配置路由](/documentation/articles/expressroute-howto-routing-arm)一文。 
	- 若要启用端到端连接，必须配置 Azure 专用对等互连并运行你的网络和 Microsoft Azure 之间的 BGP 对等互连。
	- 必须已创建并完全预配一个虚拟网络和一个虚拟网络网关。遵照说明创建 [VPN 网关](/documentation/articles/vpn-gateway-create-site-to-site-rm-powershell)

最多可以将 10 个虚拟网络链接到一条 ExpressRoute 线路。所有 ExpressRoute 线路必须位于同一个地理政治区域。

## 将同一 Azure 订阅中的 VNet 连接到 ExpressRoute 线路

你可以使用以下 cmdlet 将虚拟网络网关连接到 ExpressRoute 线路。在运行 cmdlet 之前，请确保已创建虚拟网络网关并可将其用于进行链接。

	$circuit = Get-AzureRmExpressRouteCircuit -Name "MyCircuit" -ResourceGroupName "MyRG"
	$gw = Get-AzureRmVirtualNetworkGateway -Name "ExpressRouteGw" -ResourceGroupName "MyRG"
	$connection = New-AzureRmVirtualNetworkGatewayConnection -Name "ERConnection" -ResourceGroupName "MyRG" -Location "East US" -VirtualNetworkGateway1 $gw -PeerId $circuit.Id -ConnectionType ExpressRoute

## 将不同 Azure 订阅中的虚拟网络连接到 ExpressRoute 线路

可以在多个订阅之间共享一个 ExpressRoute 线路。下图是在多个订阅之间共享 ExpressRoute 线路的简单示意图。大型云中的每个较小云用于表示属于组织中不同部门的订阅。组织内的每个部门可以使用自己的订阅部署其服务，但可以共享单个 ExpressRoute 线路以连接回本地网络。单个部门（在此示例中为 IT 部门）可以拥有 ExpressRoute 线路。组织内的其他订阅可以使用 ExpressRoute 线路。

>[AZURE.NOTE]专用线路的连接和带宽费用将应用于 ExpressRoute 线路所有者。所有虚拟网络共享相同的带宽。

![跨订阅连接](./media/expressroute-howto-linkvnet-classic/cross-subscription.png)

### 管理

*线路所有者* 是 ExpressRoute 线路资源的已授权超级用户。线路所有者可以创建可供*线路用户*兑现的授权。 *线路用户* 是虚拟网络网关的所有者（这些网关与 ExpressRoute 线路位于不同的订阅中）。 *线路用户* 可以兑现授权（每个虚拟网络需要一个授权）。

*线路所有者* 有权随时修改和撤消授权。撤消授权将导致从撤消其访问权限的订阅中删除所有链路连接。

### 线路所有者操作 

#### 创建授权
	
线路所有者创建授权。这样即可创建授权密钥，供线路用户用来将其虚拟网络网关连接到 ExpressRoute 线路。一个授权只可用于一个连接。

以下 cmdlet 代码段演示如何创建授权。

		Add-AzureRmExpressRouteCircuitAuthorization -Circuit $circuit -Name "MyAuthorization1"
		Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $circuit
		$circuit = Get-AzureRmExpressRouteCircuit -Name "MyCircuit" -ResourceGroupName "MyRG"

		$auth1 = Get-AzureRmExpressRouteCircuitAuthorization -Circuit $circuit -Name "MyAuthorization1"
		

对此操作的响应将包含授权密钥和状态

		Name                   : MyAuthorization1
		Id                     : /subscriptions/&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&/resourceGroups/ERCrossSubTestRG/providers/Microsoft.Network/expressRouteCircuits/CrossSubTest/authorizations/MyAuthorization1
		Etag                   : &&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&& 
		AuthorizationKey       : ####################################
		AuthorizationUseStatus : Available
		ProvisioningState      : Succeeded

		

#### 查看授权

线路所有者可以通过运行以下 cmdlet 来查看针对特定线路发出的所有授权。

	$circuit = Get-AzureRmExpressRouteCircuit -Name "MyCircuit" -ResourceGroupName "MyRG"
	$authorizations = Get-AzureRmExpressRouteCircuitAuthorization -Circuit $circuit
	

#### 添加授权

线路所有者可以使用以下 cmdlet 添加授权。

	$circuit = Get-AzureRmExpressRouteCircuit -Name "MyCircuit" -ResourceGroupName "MyRG"
	Add-AzureRmExpressRouteCircuitAuthorization -Circuit $circuit -Name "MyAuthorization2"
	Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $circuit
	
	$circuit = Get-AzureRmExpressRouteCircuit -Name "MyCircuit" -ResourceGroupName "MyRG"
	$authorizations = Get-AzureRmExpressRouteCircuitAuthorization -Circuit $circuit

	
#### 删除授权

线路所有者可以通过运行以下 cmdlet 来撤销/删除对用户的授权。

	Remove-AzureRmExpressRouteCircuitAuthorization -Name "MyAuthorization2" -Circuit $circuit
	Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $circuit	

### 线路用户操作

线路用户需有对等 ID 以及线路所有者提供的授权密钥。线路密钥类似于下面所列：


授权密钥是一个 GUID。

#### 兑现连接授权

线路用户可以通过运行以下 cmdlet 来兑现链接授权。

	$id = "/subscriptions/********************************/resourceGroups/ERCrossSubTestRG/providers/Microsoft.Network/expressRouteCircuits/MyCircuit"	
	$connection = New-AzureRmVirtualNetworkGatewayConnection -Name "ERConnection" -ResourceGroupName "RemoteResourceGroup -Location "East US" -VirtualNetworkGateway1 $gw -PeerId $id -ConnectionType ExpressRoute -AuthorizationKey "^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^"

#### 释放连接授权

可以通过删除 ExpressRoute 线路与虚拟网络之间的连接来释放授权。

## 后续步骤

有关 ExpressRoute 的详细信息，请参阅 [ExpressRoute 常见问题](/documentation/articles/expressroute-faqs)。

<!---HONumber=Mooncake_0104_2016-->