<properties
   pageTitle="使用 Resource Manager 和 PowerShell 创建和修改 ExpressRoute 线路 | Azure"
   description="本文介绍如何创建、预配、验证、更新、删除和取消预配 ExpressRoute 线路。"
   documentationCenter="na"
   services="expressroute"
   authors="ganesr"
   manager="carmonm"
   editor=""
   tags="azure-resource-manager"/>
<tags
   ms.service="expressroute"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="10/10/2016"
   wacn.date="10/31/2016"
   ms.author="ganesr"/>


# 创建和修改 ExpressRoute 线路

> [AZURE.SELECTOR]
[Azure Portal - Resource Manager](/documentation/articles/expressroute-howto-circuit-portal-resource-manager/)
[PowerShell - Resource Manager](/documentation/articles/expressroute-howto-circuit-arm/)
[PowerShell - Classic](/documentation/articles/expressroute-howto-circuit-classic/)


本文介绍如何使用 Windows PowerShell cmdlet 和 Azure Resource Manager 部署模型创建 Azure ExpressRoute 线路。本文还将介绍如何查看线路状态，以及如何更新、删除和取消预配线路。

**关于 Azure 部署模型**

[AZURE.INCLUDE [vpn-gateway-clasic-rm](../../includes/vpn-gateway-classic-rm-include.md)]

## 开始之前


- 获取最新版本的 Azure PowerShell 模块（至少 1.0 版）。遵循 [How to install and configure Azure PowerShell](/documentation/articles/powershell-install-configure/)（如何安装和配置 Azure PowerShell）中的说明，获取有关如何将计算机配置为使用 PowerShell 模块的分步指导。

- 在开始配置之前，请查看[先决条件](/documentation/articles/expressroute-prerequisites/)和[工作流](/documentation/articles/expressroute-workflows/)。

## 创建和预配 ExpressRoute 线路

### 1\.登录到你的 Azure 帐户，然后选择你的订阅

若要开始你的配置，请登录到你的 Azure 帐户。有关 PowerShell 的详细信息，请参阅 [Using Windows PowerShell with Resource Manager](/documentation/articles/powershell-azure-resource-manager/)（将 Windows PowerShell 与 Resource Manager 配合使用）。使用下面的示例来帮助你连接：

	Login-AzureRmAccount -Environment $(Get-AzureRmEnvironment -Name AzureChinaCloud)

检查该帐户的订阅：

	Get-AzureRmSubscription 

选择要为其创建 ExpressRoute 线路的订阅：

	Select-AzureRmSubscription -SubscriptionId "<subscription ID>"

### 2\.获取支持的提供商、位置和带宽的列表

在创建 ExpressRoute 线路之前，你需要支持的连接服务提供商、位置和带宽选项的列表。

PowerShell cmdlet `Get-AzureRmExpressRouteServiceProvider` 将返回此信息，你将在后面的步骤中使用该信息：

	Get-AzureRmExpressRouteServiceProvider

检查连接服务提供商是否已在该处列出。请记下以下信息，因为稍后创建线路时需要用到：

- 名称
- PeeringLocations
- BandwidthsOffered

	现在，你已准备好创建 ExpressRoute 线路。

### 3\.创建 ExpressRoute 线路

如果你尚未有资源组，则在创建 ExpressRoute 线路之前，必须先创建一个资源组。为此，可以运行以下命令：


	New-AzureRmResourceGroup -Name "ExpressRouteResourceGroup" -Location "China North"

以下示例演示如何通过北京的 Beijing Telecom Ethernet 创建 200 Mbps 的 ExpressRoute 线路。如果你使用的是其他提供商和其他设置，请在发出请求时替换该信息。

下面是请求新的服务密钥的示例：

		New-AzureRmExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup" -Location "China North" -SkuTier Standard -SkuFamily MeteredData -ServiceProviderName "Beijing Telecom Ethernet" -PeeringLocation "Beijing" -BandwidthInMbps 200

请确保指定合适的 SKU 层级和 SKU 系列。
 
- SKU 层决定是否启用 ExpressRoute 标准版或 ExpressRoute 高级版外接程序。可以指定 *Standard* 以获取标准 SKU，或指定 *Premium* 以获取高级版外接程序。

- SKU 系列确定计费类型。可以指定 *Metereddata* 以获取数据流量套餐，指定 *Unlimiteddata* 以获取无限制流量套餐。注意，可以将计费类型从 *Metereddata* 更改为 *Unlimiteddata*，但不能将类型从 *Unlimiteddata* 更改为 *Metereddata*。


>[AZURE.IMPORTANT] 从发布服务密钥的那一刻起，将对 ExpressRoute 线路进行计费。确保连接服务提供商准备好预配线路后就执行此操作。

响应将包含服务密钥。你可以通过运行以下命令获取所有这些参数的详细说明。

		Get-Help New-AzureRmExpressRouteCircuit -detailed 


### 4\.列出所有 ExpressRoute 线路

若要获取你所创建的所有 ExpressRoute 线路的列表，请运行 `Get-AzureRmExpressRouteCircuit` 命令：


    Get-AzureRmExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"


响应将如以下示例中所示：

		Name                             : ExpressRouteARMCircuit
		ResourceGroupName                : ExpressRouteResourceGroup
		Location                         : China North
		Id                               : /subscriptions/***************************/resourceGroups/ExpressRouteResourceGroup/providers/Microsoft.Network/expressRouteCircuits/ExpressRouteARMCircuit
		Etag                             : W/"################################"
		ProvisioningState                : Succeeded
		Sku                              : {
		                                     "Name": "Standard_MeteredData",
		                                     "Tier": "Standard",
		                                     "Family": "MeteredData"
		                                   }
		CircuitProvisioningState         : Enabled
		ServiceProviderProvisioningState : NotProvisioned
		ServiceProviderNotes             : 
		ServiceProviderProperties        : {
		                                     "ServiceProviderName": "Beijing Telecom Ethernet",
		                                     "PeeringLocation": "Beijing",
		                                     "BandwidthInMbps": 200
		                                   }
		ServiceKey                       : **************************************
		Peerings                         : []


你可以随时使用 *Get-AzureRmExpressRouteCircuit* cmdlet 检索此信息。进行不带任何参数的调用将列出所有线路。你的服务密钥将在 *ServiceKey* 字段中列出。

		Get-AzureRmExpressRouteCircuit


响应将如以下示例中所示：
	

		Name                             : ExpressRouteARMCircuit
		ResourceGroupName                : ExpressRouteResourceGroup
		Location                         : China North
		Id                               : /subscriptions/***************************/resourceGroups/ExpressRouteResourceGroup/providers/Microsoft.Network/expressRouteCircuits/ExpressRouteARMCircuit
		Etag                             : W/"################################"
		ProvisioningState                : Succeeded
		Sku                              : {
		                                     "Name": "Standard_MeteredData",
		                                     "Tier": "Standard",
		                                     "Family": "MeteredData"
		                                   }
		CircuitProvisioningState         : Enabled
		ServiceProviderProvisioningState : NotProvisioned
		ServiceProviderNotes             : 
		ServiceProviderProperties        : {
		                                     "ServiceProviderName": "Beijing Telecom Ethernet",
		                                     "PeeringLocation": "Beijing",
		                                     "BandwidthInMbps": 200
		                                   }
		ServiceKey                       : **************************************
		Peerings                         : []


你可以通过运行以下命令获取所有这些参数的详细说明。


	get-help Get-AzureRmExpressRouteCircuit -detailed

### 5\.将服务密钥发送给连接服务提供商进行预配

*ServiceProviderProvisioningState* 提供有关服务提供商端当前预配状态的信息。“状态”提供 Microsoft 端的状态。有关线路预配状态的详细信息，请参阅[工作流](/documentation/articles/expressroute-workflows/#expressroute-circuit-provisioning-states)这篇文章。

当你创建新的 ExpressRoute 线路时，线路将是以下状态：
	
		ServiceProviderProvisioningState : NotProvisioned
		
		CircuitProvisioningState         : Enabled



当连接提供商正在为你启用线路时，线路将转为以下状态。

	ServiceProviderProvisioningState : Provisioning
	Status                           : Enabled

只有 ExpressRoute 线路处于以下状态时，你才能使用它。

	ServiceProviderProvisioningState : Provisioned
	CircuitProvisioningState         : Enabled

### 6\.定期检查线路密钥的状态

检查线路密钥的状态，你可以通过此状态了解提供商何时启用了你的线路。配置线路后，*ServiceProviderProvisioningState* 将显示为 *Provisioned*，如以下例所示：

		Get-AzureRmExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"

	
响应将如以下示例中所示：
	

		Name                             : ExpressRouteARMCircuit
		ResourceGroupName                : ExpressRouteResourceGroup
		Location                         : China North
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

### 7\.创建路由配置

有关分步说明，请参阅 [ExpressRoute circuit routing configuration](/documentation/articles/expressroute-howto-routing-arm/)（ExpressRoute 线路路由配置）一文，了解如何创建和修改线路对等互连。

>[AZURE.IMPORTANT] 这些说明只适用于由提供第 2 层连接服务的服务提供商创建的线路。如果你的服务提供商提供第 3 层托管服务（通常是 IP VPN，如 MPLS），则连接服务提供商将为你配置和管理路由。

### 8\.将虚拟网络链接到 ExpressRoute 线路

接下来，将虚拟网络链接到 ExpressRoute 线路。使用 Resource Manager 部署模式时，请参阅 [Linking virtual networks to ExpressRoute circuits](/documentation/articles/expressroute-howto-linkvnet-arm/)（将虚拟网络链接到 ExpressRoute 线路）一文。

##  获取 ExpressRoute 线路的状态

你可以随时使用 `Get-AzureRmExpressRouteCircuit` cmdlet 检索此信息。进行不带任何参数的调用将列出所有线路。

		Get-AzureRmExpressRouteCircuit


响应将如以下示例中所示：

		Name                             : ExpressRouteARMCircuit
		ResourceGroupName                : ExpressRouteResourceGroup
		Location                         : China North
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


可以通过将资源组名称和线路名称作为参数传递给调用来获取有关特定 ExpressRoute 线路的信息：

		Get-AzureRmExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"


响应将如以下示例中所示：

		Name                             : ExpressRouteARMCircuit
		ResourceGroupName                : ExpressRouteResourceGroup
		Location                         : China North
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


你可以通过运行以下命令获取所有这些参数的详细说明。

	get-help get-azurededicatedcircuit -detailed


## <a name="modify"></a>修改 ExpressRoute 线路

你可以在不影响连接的情况下修改 ExpressRoute 线路的某些属性。

你可以在不停机的情况下执行以下操作：

- 为 ExpressRoute 线路启用或禁用 ExpressRoute 高级版外接程序。
- 增加 ExpressRoute 线路的带宽。请注意，不支持对线路的带宽进行降级。
- 将计量套餐从数据流量套餐更改为无限制流量套餐。请注意，不支持将计量套餐从无限制流量套餐更改为数据流量套餐。
-  你可以启用和禁用“允许经典操作”。

有关限制和局限性的详细信息，请参阅 [ExpressRoute FAQ](/documentation/articles/expressroute-faqs/)（ExpressRoute 常见问题）。

### 启用 ExpressRoute 高级版外接程序

可以使用以下 PowerShell 代码段为现有线路启用 ExpressRoute 高级版外接程序：

		$ckt = Get-AzureRmExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"

	$ckt.Sku.Tier = "Premium"
	$ckt.sku.Name = "Premium_MeteredData"

		Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $ckt
	
		
你的线路现已启用 ExpressRoute 高级版外接程序功能。请注意，该命令成功运行后，我们就会立即对高级版外接程序功能收费。

### 禁用 ExpressRoute 高级版外接程序

>[AZURE.IMPORTANT] 如果你使用的资源超出了标准线路允许的范围，此操作可能会失败。

注意以下事项：

- 从高级版降级到标准版之前，必须确保链接到线路的虚拟网络数少于 10 个。否则，你的更新请求将会失败，并且我们将按高级版费率向你收费。

- 你必须取消其他地理政治区域的所有虚拟网络的链接。否则，你的更新请求将会失败，并且我们将按高级版费率向你收费。

- 路由表中专用对等互连的路由必须少于 4,000。如果你的路由表大小超出 4,000 个路由，则会删除 BGP 会话且不会重新启用它，除非已播发前缀的数目低于 4,000。

可以使用以下 PowerShell cmdlet 为现有线路禁用 ExpressRoute 高级版外接程序：


	$ckt = Get-AzureRmExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"

	$ckt.Sku.Tier = "Standard"
	$ckt.sku.Name = "Standard_MeteredData"

	Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $ckt


### 更新 ExpressRoute 线路带宽

有关提供商支持的带宽选项，请查看 [ExpressRoute FAQ](/documentation/articles/expressroute-faqs/)（ExpressRoute 常见问题）。你可以选取大于现有线路大小的任何大小。

>[AZURE.IMPORTANT] 但是，你无法在不中断的情况下降低 ExpressRoute 线路的带宽。带宽降级需要取消对 ExpressRoute 线路的预配，然后重新预配新的 ExpressRoute 线路。

确定所需的大小后，可以使用以下命令调整线路的大小。


	$ckt = Get-AzureRmExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"
	
	$ckt.ServiceProviderProperties.BandwidthInMbps = 1000
	
	Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $ckt


将在 Azure 端调整线路的大小。然后，你必须联系连接提供商，让他们在那一边根据此更改更新配置。在你发出此通知后，Azure 将开始向你计收更新后的带宽选项费用。


### 将 SKU 从按流量计费转为不受限制

通过使用下面的 PowerShell 代码片段，你可以更改 ExpressRoute 线路的 SKU：

	$ckt = Get-AzureRmExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"
	
	$ckt.Sku.Family = "UnlimitedData"
	$ckt.sku.Name = "Premium_UnlimitedData"
	
	Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $ckt

### 控制对经典环境和 Resource Manager 环境的访问  

查看 [Move ExpressRoute circuits from the classic to the Resource Manager deployment model](/documentation/articles/expressroute-howto-move-arm/)（将 ExpressRoute 线路从经典部署模型转移到 Resource Manager 部署模型）中的说明。


## 取消预配和删除 ExpressRoute 线路

注意以下事项：

- 必须取消所有虚拟网络与 ExpressRoute 线路的链接。如果此操作失败，请查看是否有虚拟网络链接到了该线路。

- 如果 ExpressRoute 线路服务提供商预配状态为“正在预配”或“已预配”，则必须与服务提供商合作，在他们一端取消预配线路。在服务提供商取消对线路的预配并通知我们之前，我们会继续保留资源并向你收费。

- 如果服务提供商已取消预配线路（服务提供商预配状态设置为“未预配”），则可以删除线路。这样就会停止线路计费

可以通过运行以下命令来删除 ExpressRoute 线路：

	Remove-AzureRmExpressRouteCircuit -ResourceGroupName "ExpressRouteResourceGroup" -Name "ExpressRouteARMCircuit"



## 后续步骤

创建你的线路后，请确保执行以下操作：

- [创建和修改 ExpressRoute 线路的路由](/documentation/articles/expressroute-howto-routing-arm/)
- [将虚拟网络链接到 ExpressRoute 线路](/documentation/articles/expressroute-howto-linkvnet-arm/)

<!---HONumber=Mooncake_0215_2016-->