<properties
   pageTitle="使用 Azure 资源管理器和 PowerShell 配置 ExpressRoute 线路 | Microsoft Azure"
   description="本文将指导你完成创建和预配 ExpressRoute 线路的步骤。本文还介绍了如何检查状态，以及如何更新、删除和预配线路。"
   documentationCenter="na"
   services="expressroute"
   authors="cherylmc"
   manager="carolz"
   editor=""
   tags="azure-resource-manager"/>
<tags
   ms.service="expressroute"
   ms.date="12/04/2015"
   wacn.date="01/14/2016"/>

# 使用 Azure 资源管理器和 PowerShell 创建和修改 ExpressRoute 线路

> [AZURE.SELECTOR]
[PowerShell - 经典](/documentation/articles/expressroute-howto-circuit-classic)
[PowerShell - 资源管理器](/documentation/articles/expressroute-howto-circuit-arm)

本文将指导你执行相关步骤，以便使用 PowerShell cmdlet 和 Azure 资源管理器部署模型创建 ExpressRoute 线路。下面的步骤还将向你显示如何查看状态，以及如何更新、删除和预配 ExpressRoute 线路。

[AZURE.INCLUDE [vpn-gateway-sm-rm](../includes/vpn-gateway-sm-rm-include.md)] 

## 配置先决条件

- 你将需要最新版本的 Azure PowerShell 模块（版本 1.0 或更高版本）。按照[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure) 页上的说明操作，以便获取有关如何配置计算机以使用 Azure PowerShell 模块的分步指导。 
- 在开始配置之前，请务必查看[先决条件](/documentation/articles/expressroute-prerequisites)页和[工作流](/documentation/articles/expressroute-workflows)页。

## 创建和预配 ExpressRoute 线路

1. **为 ExpressRoute 导入 PowerShell 模块。**

 	你必须从 [PowerShell 库](http://www.powershellgallery.com/)安装最新的 PowerShell 安装程序，并将 Azure 资源管理器模块导入 PowerShell 会话，以便开始使用 ExpressRoute cmdlet。你需要以管理员身份运行 PowerShell。

	    Install-Module AzureRM

		Install-AzureRM

	导入已知语义版本范围内的所有 AzureRM.* 模块

		Import-AzureRM

	也可以只导入已知语义版本范围内的所选模块
		
		Import-Module AzureRM.Network 

	登录到你的帐户

		Login-AzureRmAccount

	选择要创建 ExpressRoute 线路的订阅
		
		Select-AzureRmSubscription -SubscriptionId "<subscription ID>"
			


2. **获取支持的提供商、位置和带宽的列表。**

	在创建 ExpressRoute 线路之前，你需要连接提供商、支持的位置和带宽选项的列表。PowerShell cmdlet *Get-AzureRmExpressRouteServiceProvider* 会返回此信息，你将在后续步骤中使用此信息。

		PS C:\> Get-AzureRmExpressRouteServiceProvider

	检查连接服务提供商是否已在该处列出。记下以下信息，因为你在创建线路时将需要这些信息。
	
	- Name
	- PeeringLocations
	- BandwidthsOffered

	现在，你已准备好创建 ExpressRoute 线路。

		
3. **创建 ExpressRoute 线路。**

	在创建 ExpressRoute 线路之前，必须先创建一个资源组（如果你还没有）。为此，可以运行以下命令。

		New-AzureRmResourceGroup -Name “ExpressRouteResourceGroup” -Location "West US"

	以下示例演示如何通过北京的 Beijing Telecom Ethernet 创建 200 Mbps 的 ExpressRoute 线路。如果你使用的是其他提供商和其他设置，请在发出请求时替换该信息。

	下面是请求新的服务密钥的示例：

		New-AzureRmExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup" -Location "China North" -SkuTier Standard -SkuFamily MeteredData -ServiceProviderName "Beijing Telecom Ethernet" -PeeringLocation "Beijing" -BandwidthInMbps 200

	请确保指定合适的 SKU 层级和 SKU 系列。
 
	 - SKU 层级决定是否启用 ExpressRoute 标准版外接程序。
	 - SKU 系列确定计费类型。可以选择 *metereddata* 以使用数据流量套餐，选择“unlimiteddata”以使用无限制的流量套餐。**注意：**创建线路后，你将不能更改计费类型。

	响应将包含服务密钥。你可以通过运行以下命令获取所有这些参数的详细说明。

		get-help New-AzureRmExpressRouteCircuit -detailed 

4. **列出所有 ExpressRoute 线路。**

	你可以运行 *Get-AzureRmExpressRouteCircuit* 命令，以获取你所创建的所有 ExpressRoute 线路的列表。

		#Getting service key
		Get-AzureRmExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"

	响应将如下例所示：

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

	响应将如下例所示：

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

5. **将服务密钥发送给连接提供商进行预配。**

	当你创建新的 ExpressRoute 线路时，线路将是以下状态：
	
		ServiceProviderProvisioningState : NotProvisioned
		
		CircuitProvisioningState         : Enabled

	*ServiceProviderProvisioningState* 提供有关服务提供商端当前预配状态的信息，而 Status 则提供 Microsoft 端的状态。ExpressRoute 线路处于以下状态时，你才能使用它。

		ServiceProviderProvisioningState : Provisioned
		
		CircuitProvisioningState         : Enabled

	当连接提供商正在为你启用线路时，线路将转为以下状态。

		ServiceProviderProvisioningState : Provisioned
		
		Status                           : Enabled



5. **定期检查服务密钥的状态。**

	这样，你就知道提供商何时启用了你的线路。配置线路后，*ServiceProviderProvisioningState* 将显示为 *Provisioned*，如下例所示。

		Get-AzureRmExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"

	响应将如下例所示：

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

6. **创建路由配置。**
	
	如需分步说明，请参阅 [ExpressRoute 线路路由配置（创建和修改线路对等互连）](/documentation/articles/expressroute-howto-routing-arm)页。

7. **将 VNet 链接到 ExpressRoute 线路。**

	接下来，将 VNet 链接到 ExpressRoute 线路。使用 Azure 资源管理器部署模式时，你可以使用[此模板](https://github.com/Azure/azure-quickstart-templates/tree/ecad62c231848ace2fbdc36cbe3dc04a96edd58c/301-expressroute-circuit-vnet-connection)。我们目前正在致力于 PowerShell 步骤。

##  获取 ExpressRoute 线路的状态

你可以随时使用 *Get-AzureRmExpressRouteCircuit* cmdlet 检索此信息。进行不带任何参数的调用将列出所有线路。

		Get-AzureRmExpressRouteCircuit

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

可以通过将资源组名称和线路名称作为参数传递给调用来获取有关特定 ExpressRoute 线路的信息。

		Get-AzureRmExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"

	The response will be something similar to the example below:

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

## 修改 ExpressRoute 线路

你可以在不影响连接的情况下修改 ExpressRoute 线路的某些属性。

你可以执行以下操作：

- 在不停机的情况下，增加 ExpressRoute 线路的带宽。

有关限制的详细信息，请参阅 [ExpressRoute 常见问题](/documentation/articles/expressroute-faqs)页。


### 如何更新 ExpressRoute 线路带宽

有关你的提供商支持的带宽选项，请查看 [ExpressRoute 常见问题](/documentation/articles/expressroute-faqs)页。你可以选取大于现有线路大小的任何大小。确定所需的大小后，可以使用以下命令调整线路的大小。

		$ckt = Get-AzureRmExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"

		$ckt.ServiceProviderProperties.BandwidthInMbps = 1000

		Set-AzureRmExpressRouteCircuit -ExpressRouteCircuit $ckt

将已在 Microsoft 一侧估计好线路的大小。你必须联系连接提供商，让他们在那一边根据此更改更新配置。请注意，我们将从现在开始按照已更新的带宽选项为你计费。

>[AZURE.IMPORTANT]但是，你无法在不中断的情况下降低 ExpressRoute 线路的带宽。带宽降级需要取消对 ExpressRoute 线路的预配，然后重新预配新的 ExpressRoute 线路。

## 删除和取消预配 ExpressRoute 线路

可以通过运行以下命令来删除 ExpressRoute 线路：

		Remove-AzureRmExpressRouteCircuit -ResourceGroupName "ExpressRouteResourceGroup" -Name "ExpressRouteARMCircuit"

请注意，必须取消所有虚拟网络与 ExpressRoute 的链接，此操作才能成功。如果此操作失败，请查看你是否有虚拟网络链接到了此线路。

如果启用了 ExpressRoute 线路服务提供商预配状态，则状态将从启用转为*禁用*。你必须通过服务提供商在他们那一侧取消对线路的预配。在服务提供商取消对线路的预配并向我们发送通知之前，我们会继续保留资源并向你收费。

如果在你运行上述 cmdlet 之前，服务提供商已取消对线路的预配（服务提供商预配状态已设置为*未预配*），我们会取消对线路的预配，并停止向你收费。

## 后续步骤

- [配置路由](/documentation/articles/expressroute-howto-routing-arm)
- [将 VNet 链接到 ExpressRoute 线路](/documentation/articles/expressroute-howto-linkvnet-arm) 

<!---HONumber=Mooncake_0104_2016-->