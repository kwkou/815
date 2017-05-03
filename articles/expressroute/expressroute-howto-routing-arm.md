<properties
    pageTitle="如何为 ExpressRoute 线路配置路由（对等互连）：Resource Manager：PowerShell：Azure "
    description="本文介绍完成创建和预配 ExpressRoute 线路的专用、公共对等互连的步骤。 本文还介绍了如何检查状态，以及如何更新或删除线路的对等互连。"
    documentationcenter="na"
    services="expressroute"
    author="ganesr"
    manager="timlt"
    editor=""
    tags="azure-resource-manager"
    translationtype="Human Translation" />
<tags
    ms.assetid="0a036d51-77ae-4fee-9ddb-35f040fbdcdf"
    ms.service="expressroute"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="03/21/2017"
    wacn.date="05/02/2017"
    ms.author="ganesr;cherylmc"
    ms.sourcegitcommit="78da854d58905bc82228bcbff1de0fcfbc12d5ac"
    ms.openlocfilehash="6b84a7b558d97cf0d324eeaeb419b128af15059c"
    ms.lasthandoff="04/22/2017" />

# <a name="create-and-modify-peering-for-an-expressroute-circuit-using-powershell"></a>使用 PowerShell 创建和修改 ExpressRoute 线路的对等互连
> [AZURE.SELECTOR]
> [Resource Manager - Azure 门户](/documentation/articles/expressroute-howto-routing-portal-resource-manager/)

> *[Resource Manager - PowerShell](/documentation/articles/expressroute-howto-routing-arm/)
> *[经典 - PowerShell](/documentation/articles/expressroute-howto-routing-classic/)
>
>

本文介绍使用 PowerShell 和 Azure Resource Manager 部署模型创建和管理 ExpressRoute 线路路由配置的相关步骤。  下面的步骤还说明如何查看状态，以及如何更新、删除和取消预配 ExpressRoute 线路的对等互连。 

**关于 Azure 部署模型**

[AZURE.INCLUDE [vpn-gateway-clasic-rm](../../includes/vpn-gateway-classic-rm-include.md)]

## <a name="configuration-prerequisites"></a>配置先决条件
- 需要最新版本的 Azure Resource Manager PowerShell cmdlet。 有关详细信息，请参阅[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/)。 
- 在开始配置之前，请务必查看[先决条件](/documentation/articles/expressroute-prerequisites/)页、[路由要求](/documentation/articles/expressroute-routing/)页和[工作流](/documentation/articles/expressroute-workflows/)页。
- 必须有活动的 ExpressRoute 线路。在继续下一步之前，请按说明[创建 ExpressRoute 线路](/documentation/articles/expressroute-howto-circuit-arm/)，并通过连接提供商启用该线路。ExpressRoute 线路必须处于已预配和已启用状态，才能运行下述 cmdlet。

这些说明只适用于由提供第 2 层连接服务的服务提供商创建的线路。如果你的服务提供商提供第 3 层托管服务（通常是 IPVPN，如 MPLS），则连接服务提供商将为你设置和管理路由。

>[AZURE.IMPORTANT]
> 我们目前无法通过服务管理门户播发服务提供商配置的对等互连。 我们正在努力不久就实现这一功能。 请在配置 BGP 对等互连之前与服务提供商核对。
> 
> 
你可以为 ExpressRoute 线路配置一到两个对等互连（Azure 专用和Azure 公共）。可以按照所选的任意顺序配置对等互连。但是，你必须确保一次只完成一个对等互连的配置。

## <a name="azure-private-peering"></a>Azure 专用对等互连
本部分说明如何为 ExpressRoute 线路创建、获取、更新和删除 Azure 专用对等互连配置。 

### <a name="to-create-azure-private-peering"></a>创建 Azure 专用对等互连
1. 为 ExpressRoute 导入 PowerShell 模块。

     必须从 [PowerShell 库](http://www.powershellgallery.com/) 安装最新的 PowerShell 安装程序，并将 Azure Resource Manager 模块导入 PowerShell 会话，以便开始使用 ExpressRoute cmdlet。 你需要以管理员身份运行 PowerShell。

        Install-Module AzureRM

        Install-AzureRM

    导入已知语义版本范围内的所有 AzureRM.* 模块

        Import-AzureRM

    也可以只导入已知语义版本范围内的所选模块 

        Import-Module AzureRM.Network 

    登录到帐户

        Login-AzureRmAccount -Environment $(Get-AzureRmEnvironment -Name AzureChinaCloud)

    选择要创建 ExpressRoute 线路的订阅

        Select-AzureRmSubscription -SubscriptionId "<subscription ID>"

2. 创建 ExpressRoute 线路。

    请按说明创建 [ExpressRoute 线路](/documentation/articles/expressroute-howto-circuit-arm/) ，并由连接服务提供商进行预配。 

	如果连接服务提供商提供第 3 层托管服务，可以请求连接服务提供商启用 Azure 专用对等互连。在这种情况下，不需要遵循后续部分中所列的说明。但是，如果连接服务提供商不管理路由，请在创建线路后遵循以下说明。

3. 检查 ExpressRoute 线路以确保其已预配。

	首先，必须检查 ExpressRoute 线路是否已预配且已启用。请参阅以下示例。

		Get-AzureRmExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"

	响应将如下例所示：

		Name                             : ExpressRouteARMCircuit
		ResourceGroupName                : ExpressRouteResourceGroup
		Location                         : chinanorth
		Id                               : /subscriptions/***************************/resourceGroups/ExpressRouteResourceGroup/providers/Microsoft.Network/expressRouteCircuits/ExpressRouteARMCircuit
		Etag                             : W/"################################"
		ProvisioningState                : Succeeded
		Sku                              : {
		                                     "Name": "Standard_MeteredData",
		                                     "Tier": "Standard",
		                                     "Family": "MeteredData"
		                                   }
		CircuitProvisioningState         : Enabled
		ServiceProviderProvisioningState : Provisioned
		ServiceProviderNotes             : 
		ServiceProviderProperties        : {
		                                     "ServiceProviderName": "Beijing Telecom Ethernet",
		                                     "PeeringLocation": "Beijing",
		                                     "BandwidthInMbps": 200
		                                   }
		ServiceKey                       : **************************************
		Peerings                         : []


4. 配置线路的 Azure 专用对等互连。

	在继续执行后续步骤之前，请确保已准备好以下各项：

	- 主链路的 /30 子网。它不能是保留给虚拟网络使用的任何地址空间的一部分。
	- 辅助链路的 /30 子网。它不能是保留给虚拟网络使用的任何地址空间的一部分。
	- 用于建立此对等互连的有效 VLAN ID。请确保线路中没有其他对等互连使用同一个 VLAN ID。
	- 对等互连的 AS 编号。可以使用 2 字节和 4 字节 AS 编号。可以将专用 AS 编号用于此对等互连。请务必不要使用 65515。
	- MD5 哈希（如果选择使用）。**这是可选的**。
	
	可以运行以下 cmdlet 为线路配置 Azure 专用对等互连。

          	Add-AzureRmExpressRouteCircuitPeeringConfig -Name "AzurePrivatePeering" -ExpressRouteCircuit $ckt -PeeringType AzurePrivatePeering -PeerASN 100 -PrimaryPeerAddressPrefix "10.0.0.0/30" -SecondaryPeerAddressPrefix "10.0.0.4/30" -VlanId 200

		Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $ckt

	如果选择使用 MD5 哈希，则可以使用以下 cmdlet。

          	Add-AzureRmExpressRouteCircuitPeeringConfig -Name "AzurePrivatePeering" -ExpressRouteCircuit $ckt -PeeringType AzurePrivatePeering -PeerASN 100 -PrimaryPeerAddressPrefix "10.0.0.0/30" -SecondaryPeerAddressPrefix "10.0.0.4/30" -VlanId 200  -SharedKey "A1B2C3D4"

     
   > [AZURE.IMPORTANT]
   > 请确保将 AS 编号指定为对等互连 ASN，而不是客户 ASN。
   > 
   >

### <a name="to-view-azure-private-peering-details"></a>查看 Azure 专用对等互连详细信息
可以使用以下 cmdlet 来获取配置详细信息。

		$ckt = Get-AzureRmExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"

		Get-AzureRmExpressRouteCircuitPeeringConfig -Name "AzurePrivatePeering" -Circuit $ckt	

### <a name="to-update-azure-private-peering-configuration"></a>更新 Azure 专用对等互连配置

可以使用以下 cmdlet 更新配置的任何部分。在以下示例中，线路的 VLAN ID 将从 100 更新为 500。

	Set-AzureRmExpressRouteCircuitPeeringConfig -Name "AzurePrivatePeering" -ExpressRouteCircuit $ckt -PeeringType AzurePrivatePeering -PeerASN 100 -PrimaryPeerAddressPrefix "10.0.0.0/30" -SecondaryPeerAddressPrefix "10.0.0.4/30" -VlanId 200

	Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $ckt

### <a name="to-delete-azure-private-peering"></a>删除 Azure 专用对等互连

可以运行以下 cmdlet 删除对等互连配置。

> [AZURE.WARNING]
> 运行此 cmdlet 之前，必须确保已从 ExpressRoute 线路取消链接所有虚拟网络。 
> 
> 

	Remove-AzureRmExpressRouteCircuitPeeringConfig -Name "AzurePrivatePeering" -ExpressRouteCircuit $ckt
	Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $ckt

## <a name="azure-public-peering"></a>Azure 公共对等互连

本部分说明如何为 ExpressRoute 线路创建、获取、更新和删除 Azure 公共对等互连配置。

### <a name="to-create-azure-public-peering"></a>创建 Azure 公共对等互连

1. 为 ExpressRoute 导入 PowerShell 模块。

     必须从 [PowerShell 库](http://www.powershellgallery.com/) 安装最新的 PowerShell 安装程序，并将 Azure Resource Manager 模块导入 PowerShell 会话，以便开始使用 ExpressRoute cmdlet。 你需要以管理员身份运行 PowerShell。

	    Install-Module AzureRM

		Install-AzureRM

	导入已知语义版本范围内的所有 AzureRM.* 模块

		Import-AzureRM

	也可以只导入已知语义版本范围内的所选模块
		
		Import-Module AzureRM.Network 

	登录到帐户

		Login-AzureRmAccount -Environment $(Get-AzureRmEnvironment -Name AzureChinaCloud)

	选择要创建 ExpressRoute 线路的订阅
		
		Select-AzureRmSubscription -SubscriptionId "<subscription ID>"

2. 创建 ExpressRoute 线路。
	
	请按说明创建 [ExpressRoute 线路](/documentation/articles/expressroute-howto-circuit-arm/)，并由连接服务提供商进行预配。

	如果连接服务提供商提供第 3 层托管服务，可以请求连接服务提供商启用 Azure 公共对等互连。在这种情况下，不需要遵循后续部分中所列的说明。但是，如果连接服务提供商不管理路由，请在创建线路后遵循以下说明。

3. 检查 ExpressRoute 线路以确保其已预配。

	首先，必须检查 ExpressRoute 线路是否已预配且已启用。请参阅以下示例。

		Get-AzureRmExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"

	响应将如下例所示：

		Name                             : ExpressRouteARMCircuit
		ResourceGroupName                : ExpressRouteResourceGroup
		Location                         : chinanorth
		Id                               : /subscriptions/***************************/resourceGroups/ExpressRouteResourceGroup/providers/Microsoft.Network/expressRouteCircuits/ExpressRouteARMCircuit
		Etag                             : W/"################################"
		ProvisioningState                : Succeeded
		Sku                              : {
		                                     "Name": "Standard_MeteredData",
		                                     "Tier": "Standard",
		                                     "Family": "MeteredData"
		                                   }
		CircuitProvisioningState         : Enabled
		ServiceProviderProvisioningState : Provisioned
		ServiceProviderNotes             : 
		ServiceProviderProperties        : {
		                                     "ServiceProviderName": "Beijing Telecom Ethernet",
		                                     "PeeringLocation": "Beijing",
		                                     "BandwidthInMbps": 200
		                                   }
		ServiceKey                       : **************************************
		Peerings                         : []	

4. 配置线路的 Azure 公共对等互连。

	在继续下一步之前，请确保已准备以下信息。

	- 主链路的 /30 子网。这必须是有效的公共 IPv4 前缀。
	- 辅助链路的 /30 子网。这必须是有效的公共 IPv4 前缀。
	- 用于建立此对等互连的有效 VLAN ID。请确保线路中没有其他对等互连使用同一个 VLAN ID。
	- 对等互连的 AS 编号。可以使用 2 字节和 4 字节 AS 编号。
	- MD5 哈希（如果选择使用）。**这是可选的**。
	
	可以运行以下 cmdlet 为线路配置 Azure 公共对等互连

		Add-AzureRmExpressRouteCircuitPeeringConfig -Name "AzurePublicPeering" -ExpressRouteCircuit $ckt -PeeringType AzurePublicPeering -PeerASN 100 -PrimaryPeerAddressPrefix "12.0.0.0/30" -SecondaryPeerAddressPrefix "12.0.0.4/30" -VlanId 100

		Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $ckt

	如果选择使用 MD5 哈希，则可以使用以下 cmdlet

		Add-AzureRmExpressRouteCircuitPeeringConfig -Name "AzurePublicPeering" -ExpressRouteCircuit $ckt -PeeringType AzurePublicPeering -PeerASN 100 -PrimaryPeerAddressPrefix "12.0.0.0/30" -SecondaryPeerAddressPrefix "12.0.0.4/30" -VlanId 100  -SharedKey "A1B2C3D4"

		Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $ckt

    >[AZURE.IMPORTANT]
    > 请确保将 AS 编号指定为对等互连 ASN，而不是客户 ASN。
    > 
    >

### <a name="to-view-azure-public-peering-details"></a>查看 Azure 公共对等互连详细信息
可以使用以下 cmdlet 来获取配置详细信息：

		$ckt = Get-AzureRmExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"

		Get-AzureRmExpressRouteCircuitPeeringConfig -Name "AzurePublicPeering" -Circuit $ckt

### <a name="to-update-azure-public-peering-configuration"></a>更新 Azure 公共对等互连配置
可以使用以下 cmdlet 来更新配置的任何部分：

    Set-AzureRmExpressRouteCircuitPeeringConfig  -Name "AzurePublicPeering" -ExpressRouteCircuit $ckt -PeeringType AzurePublicPeering -PeerASN 100 -PrimaryPeerAddressPrefix "123.0.0.0/30" -SecondaryPeerAddressPrefix "123.0.0.4/30" -VlanId 600 

	Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $ckt

在上面的示例中，线路的 VLAN ID 将从 200 更新为 600。

### <a name="to-delete-azure-public-peering"></a>删除 Azure 公共对等互连

可以运行以下 cmdlet 来删除对等互连配置

	Remove-AzureRmExpressRouteCircuitPeeringConfig -Name "AzurePublicPeering" -ExpressRouteCircuit $ckt
	Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $ckt

## <a name="next-steps"></a>后续步骤

下一步，[将 VNet 链接到 ExpressRoute 线路](/documentation/articles/expressroute-howto-linkvnet-arm/)。

-  有关 ExpressRoute 工作流的详细信息，请参阅 [ExpressRoute 工作流](/documentation/articles/expressroute-workflows/)。

-  有关线路对等互连的详细信息，请参阅 [ExpressRoute 线路和路由域](/documentation/articles/expressroute-circuit-peerings/)。

-  有关使用虚拟网络的详细信息，请参阅[虚拟网络概述](/documentation/articles/virtual-networks-overview/)。


<!--Update_Description:update wording and code-->
