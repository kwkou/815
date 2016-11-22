<properties
   pageTitle="使用经典部署模型和 PowerShell 创建和修改 ExpressRoute 线路 | Azure"
   description="本文将指导你完成创建和预配 ExpressRoute 线路的步骤。本文还介绍了如何检查状态，以及如何更新、删除和预配线路。"
   documentationCenter="na"
   services="expressroute"
   authors="ganesr"
   manager="carmonm"
   editor=""
   tags="azure-service-management"/>
<tags
   ms.service="expressroute"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="10/10/2016"
   wacn.date="10/31/2016"
   ms.author="ganesr;cherylmc"/>

# 创建和修改 ExpressRoute 线路

> [AZURE.SELECTOR]
[Azure Portal - Resource Manager](/documentation/articles/expressroute-howto-circuit-portal-resource-manager/)
[PowerShell - Resource Manager](/documentation/articles/expressroute-howto-circuit-arm/)
[PowerShell - Classic](/documentation/articles/expressroute-howto-circuit-classic/)

本文将指导你执行相关步骤，以便使用 PowerShell cmdlet 和经典部署模型创建 Azure ExpressRoute 线路。本文还将向你显示如何查看状态，以及如何更新、删除和预配 ExpressRoute 线路。

**关于 Azure 部署模型**

[AZURE.INCLUDE [vpn-gateway-clasic-rm](../../includes/vpn-gateway-classic-rm-include.md)]


## 开始之前

### 1\.查看先决条件和工作流文章

在开始配置之前，请务必查看[先决条件](/documentation/articles/expressroute-prerequisites/)和[工作流](/documentation/articles/expressroute-workflows/)。


### 2\.安装最新版本的 Azure PowerShell 模块 

按照[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/) 中的说明操作，以便获取有关如何配置计算机以使用 Azure PowerShell 模块的分步指导。

### 3\.登录到 Azure 帐户并选择订阅

1. 使用权限提升的 Windows PowerShell 提示符运行以下 cmdlet：

		Add-AzureAccount -Environment AzureChinaCloud
2. 在出现的登录屏幕中登录到帐户。

3. 获取你的订阅的列表。

		Get-AzureSubscription
4. 选择要使用的订阅。
	
		Select-AzureSubscription -SubscriptionName "mysubscriptionname"

## 创建和预配 ExpressRoute 线路

### 1\.为 ExpressRoute 导入 PowerShell 模块

 在开始使用 ExpressRoute cmdlet 之前，必须将 Azure 和 ExpressRoute 模块导入 PowerShell 会话（如果尚未这样做）。将模块从其安装位置导入本地计算机。根据模块的安装方法，该位置可能与下例中所示不同。请根据需要修改示例。

	    Import-Module 'C:\Program Files (x86)\Microsoft SDKs\Azure\PowerShell\ServiceManagement\Azure\Azure.psd1'
	    Import-Module 'C:\Program Files (x86)\Microsoft SDKs\Azure\PowerShell\ServiceManagement\Azure\ExpressRoute\ExpressRoute.psd1'

### 2\.获取支持的提供商、位置和带宽的列表

在创建 ExpressRoute 线路之前，你需要支持的连接服务提供商、位置和带宽选项的列表。

PowerShell cmdlet `Get-AzureDedicatedCircuitServiceProvider` 将返回此信息，你将在后面的步骤中使用该信息：

	Get-AzureDedicatedCircuitServiceProvider

检查连接服务提供商是否已在该处列出。请记下以下信息，因为稍后创建线路时需要用到：

- 名称

- PeeringLocations

- BandwidthsOffered

现在，你已准备好创建 ExpressRoute 线路。

### 3\.创建 ExpressRoute 线路

	以下示例演示如何通过北京的 Beijing Telecom Ethernet 创建 200 Mbps 的 ExpressRoute 线路。如果你使用的是其他提供商和其他设置，请在发出请求时替换该信息。

>[AZURE.IMPORTANT] 从发布服务密钥的那一刻起，将对 ExpressRoute 线路进行计费。确保连接服务提供商准备好预配线路后就执行此操作。


下面是请求新的服务密钥的示例：

		#Creating a new circuit
		$Bandwidth = 200
		$CircuitName = "21vDemo"
		$ServiceProvider = "Beijing Telecom Ethernet"
		$Location = "Beijing"

	New-AzureDedicatedCircuit -CircuitName $CircuitName -ServiceProviderName $ServiceProvider -Bandwidth $Bandwidth -Location $Location -sku Standard -BillingType MeteredData

或者，如果你想要通过高级版外接程序创建 ExpressRoute 线路，则可使用下述示例。请参阅 [ExpressRoute 常见问题](/documentation/articles/expressroute-faqs/)页，了解有关高级版外接程序的更多详细信息。

	New-AzureDedicatedCircuit -CircuitName $CircuitName -ServiceProviderName $ServiceProvider -Bandwidth $Bandwidth -Location $Location -sku Premium - BillingType MeteredData


响应将包含服务密钥。你可以通过运行以下命令获取所有这些参数的详细说明。

		Get-Help New-AzureDedicatedCircuit -detailed 

### 4\.列出所有 ExpressRoute 线路

可以运行 `Get-AzureDedicatedCircuit` 命令，获取创建的所有 ExpressRoute 线路的列表：


	Get-AzureDedicatedCircuit

响应将如以下示例所示：

		Bandwidth                        : 200
		CircuitName                      : 21vDemo
		Location                         : Beijing
		ServiceKey                       : *********************************
		ServiceProviderName              : Beijing Telecom Ethernet
		ServiceProviderProvisioningState : NotProvisioned
		Sku                              : Standard
		Status                           : Enabled

可以随时使用 `Get-AzureDedicatedCircuit` cmdlet 检索此信息。进行不带任何参数的调用将列出所有线路。服务密钥将在 *ServiceKey* 字段中列出。

	Get-AzureDedicatedCircuit

		Bandwidth                        : 200
		CircuitName                      : 21vDemo
		Location                         : Beijing
		ServiceKey                       : *********************************
		ServiceProviderName              : Beijing Telecom Ethernet
		ServiceProviderProvisioningState : NotProvisioned
		Sku                              : Standard
		Status                           : Enabled

	你可以通过运行以下命令获取所有这些参数的详细说明。

		Get-Help Get-AzureDedicatedCircuit -detailed 

### 5\.将服务密钥发送给连接服务提供商进行预配


*ServiceProviderProvisioningState* 提供有关服务提供商端当前预配状态的信息。“状态”提供 Microsoft 端的状态。有关线路预配状态的详细信息，请参阅 [Workflows](/documentation/articles/expressroute-workflows/#expressroute-circuit-provisioning-states)（工作流）一文。

当你创建新的 ExpressRoute 线路时，线路将是以下状态：

	ServiceProviderProvisioningState : NotProvisioned
	Status                           : Enabled


当连接服务提供商正在为你启用线路时，线路将转为以下状态：

	ServiceProviderProvisioningState : Provisioning
	Status                           : Enabled

ExpressRoute 线路处于以下状态时，你才能使用它：

		ServiceProviderProvisioningState : Provisioned
		
		Status                           : Enabled


### 6\.定期检查线路密钥的状态

这样，你就知道提供商何时启用了你的线路。配置线路后，*ServiceProviderProvisioningState* 将显示为 *Provisioned*，如以下示例所示：

		PS C:\> Get-AzureDedicatedCircuit

		Bandwidth                        : 200
		CircuitName                      : 21vDemo
		Location                         : Beijing
		ServiceKey                       : *********************************
		ServiceProviderName              : Beijing Telecom Ethernet
		ServiceProviderProvisioningState : Provisioned
		Sku                              : Standard
		Status                           : Enabled

### 7\.创建路由配置
	
有关分步说明，请参阅 [ExpressRoute circuit routing configuration (create and modify circuit peerings)](/documentation/articles/expressroute-howto-routing-classic/)（ExpressRoute 线路路由配置（创建和修改线路对等互连））一文。

>[AZURE.IMPORTANT] 这些说明只适用于由提供第 2 层连接服务的服务提供商创建的线路。如果你的服务提供商提供第 3 层托管服务（通常是 IP VPN，如 MPLS），则连接服务提供商将为你配置和管理路由。

### 8\.将虚拟网络链接到 ExpressRoute 线路

接下来，将虚拟网络链接到 ExpressRoute 线路。有关分步说明，请参阅 [Linking ExpressRoute circuits to virtual networks](/documentation/articles/expressroute-howto-linkvnet-classic/)（将 ExpressRoute 线路链接到虚拟网络）。如需使用经典部署模型为 ExpressRoute 创建虚拟网络，请参阅 [Create a virtual network for ExpressRoute](/documentation/articles/expressroute-howto-vnet-portal-classic/)（为 ExpressRoute 创建虚拟网络）中的说明。

##  获取 ExpressRoute 线路的状态

可以随时使用 `Get-AzureCircuit` cmdlet 检索此信息。进行不带任何参数的调用将列出所有线路。

	Get-AzureDedicatedCircuit

		Bandwidth                        : 200
		CircuitName                      : 21vDemo
		Location                         : Beijing
		ServiceKey                       : *********************************
		ServiceProviderName              : Beijing Telecom Ethernet
		ServiceProviderProvisioningState : Provisioned
		Sku                              : Standard
		Status                           : Enabled

		Bandwidth                        : 1000
		CircuitName                      : MyAsiaCircuit
		Location                         : China
		ServiceKey                       : #################################
		ServiceProviderName              : equinix
		ServiceProviderProvisioningState : Provisioned
		Sku                              : Standard
		Status                           : Enabled

你可以将服务密钥作为参数传递给调用，从而获取特定 ExpressRoute 线路的相关信息：

	Get-AzureDedicatedCircuit -ServiceKey "*********************************"

		Bandwidth                        : 200
		CircuitName                      : 21vDemo
		Location                         : Beijing
		ServiceKey                       : *********************************
		ServiceProviderName              : Beijing Telecom Ethernet
		ServiceProviderProvisioningState : Provisioned
		Sku                              : Standard
		Status                           : Enabled


你可以通过运行以下命令获取所有这些参数的详细说明。

		get-help get-azurededicatedcircuit -detailed 

##  修改 ExpressRoute 线路

你可以在不影响连接的情况下修改 ExpressRoute 线路的某些属性。

你可以在不停机的情况下执行以下操作：

- 为 ExpressRoute 线路启用或禁用 ExpressRoute 高级版外接程序。
- 增加 ExpressRoute 线路的带宽。请注意，不支持对线路的带宽进行降级。
- 将计量套餐从数据流量套餐更改为无限制流量套餐。请注意，不支持将计量套餐从无限制流量套餐更改为数据流量套餐。
- 可以启用和禁用“允许经典操作”。

有关限制和局限性的详细信息，请参阅 [ExpressRoute FAQ](/documentation/articles/expressroute-faqs/)（ExpressRoute 常见问题）。

### 启用 ExpressRoute 高级版外接程序

可以使用以下 PowerShell cmdlet 为现有线路启用 ExpressRoute 高级版外接程序：

	Set-AzureDedicatedCircuitProperties -ServiceKey "*********************************" -Sku Premium

	Bandwidth                        : 1000
	CircuitName                      : TestCircuit
	Location                         : Beijing
	ServiceKey                       : *********************************
	ServiceProviderName              : Beijing Telecom Ethernet
	ServiceProviderProvisioningState : Provisioned
	Sku                              : Premium
	Status                           : Enabled

你的线路现已启用 ExpressRoute 高级版外接程序功能。请注意，该命令成功运行后，我们将立即开始为你对高级版外接程序功能计费。

### 禁用 ExpressRoute 高级版外接程序

>[AZURE.IMPORTANT] 如果你使用的资源超出了标准线路允许的范围，此操作可能会失败。

注意以下事项：

- 从高级版降级到标准版之前，你必须确保链接到线路的虚拟机的数目少于 10。否则，你的更新请求会失败，将按高级版费率向你收费。

- 你必须取消其他地理政治区域的所有虚拟网络的链接。否则，你的更新请求会失败，将按高级版费率向你收费。

- 路由表中专用对等互连的路由必须少于 4,000。如果你的路由表大小超出 4,000 个路由，则会删除 BGP 会话且不会重新启用它，除非已播发前缀的数目低于 4,000。

可以使用以下 PowerShell cmdlet 为现有线路禁用 ExpressRoute 高级版外接程序：

	Set-AzureDedicatedCircuitProperties -ServiceKey "*********************************" -Sku Standard

	Bandwidth                        : 1000
	CircuitName                      : TestCircuit
	Location                         : Bejing
	ServiceKey                       : *********************************
	ServiceProviderName              : Beijing Telecom Ethernet
	ServiceProviderProvisioningState : Provisioned
	Sku                              : Standard
	Status                           : Enabled



### 更新 ExpressRoute 线路带宽

有关提供商支持的带宽选项，请查看 [ExpressRoute FAQ](/documentation/articles/expressroute-faqs/)（ExpressRoute 常见问题）。你可以选取大于现有线路大小的任何大小。

>[AZURE.IMPORTANT] 但是，你无法在不中断的情况下降低 ExpressRoute 线路的带宽。带宽降级需要取消对 ExpressRoute 线路的预配，然后重新预配新的 ExpressRoute 线路。
确定所需的大小后，即可使用以下命令调整线路的大小：
		PS C:\> Set-AzureDedicatedCircuitProperties -ServiceKey ********************************* -Bandwidth 1000
		
		Bandwidth                        : 1000
		CircuitName                      : TestCircuit
		Location                         : Beijing 
		ServiceKey                       : *********************************
		ServiceProviderName              : Beijing Telecom Ethernet
		ServiceProviderProvisioningState : Provisioned
		Sku                              : Standard
		Status                           : Enabled

将已在 Azure 一侧估计好线路的大小。你必须联系连接提供商，让他们在那一边根据此更改更新配置。请注意，我们将从现在开始按照已更新的带宽选项为你计费。

如果在增加线路带宽时看到以下错误，这意味着创建现有线路所在的物理端口上没有足够的带宽可用。必须删除此线路，然后创建所需大小的新线路。

	Set-AzureDedicatedCircuitProperties : InvalidOperation : Insufficient bandwidth available to perform this circuit
	update operation
	At line:1 char:1
	+ Set-AzureDedicatedCircuitProperties -ServiceKey ********************* ...
	+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	    + CategoryInfo          : CloseError: (:) [Set-AzureDedicatedCircuitProperties], CloudException
	    + FullyQualifiedErrorId : Microsoft.WindowsAzure.Commands.ExpressRoute.SetAzureDedicatedCircuitPropertiesCommand
	

## 取消预配和删除 ExpressRoute 线路

注意以下事项：

- 必须取消所有虚拟网络与 ExpressRoute 线路的链接，此操作才能成功。如果此操作失败，请查看你是否有虚拟网络链接到了此线路。

- 如果 ExpressRoute 线路服务提供商预配状态为“正在预配”或“已预配”，则必须与服务提供商合作，在他们一端取消预配线路。在服务提供商取消对线路的预配并通知我们之前，我们会继续保留资源并向你收费。

- 如果服务提供商已取消预配线路（服务提供商预配状态设置为“未预配”），则可以删除线路。这样就会停止线路计费。

可以通过运行以下命令来删除 ExpressRoute 线路：

	Remove-AzureDedicatedCircuit -ServiceKey "*********************************"



## 后续步骤

创建你的线路后，请确保执行以下操作：

- [创建和修改 ExpressRoute 线路的路由](/documentation/articles/expressroute-howto-routing-classic/)
- [将虚拟网络链接到 ExpressRoute 线路](/documentation/articles/expressroute-howto-linkvnet-classic/)

<!---HONumber=Mooncake_0104_2016-->