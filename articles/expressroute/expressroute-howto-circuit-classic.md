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
   ms.date="04/15/2016"
   wacn.date="06/06/2016"/>

# 创建和修改 ExpressRoute 线路

> [AZURE.SELECTOR]
[PowerShell - 经典](/documentation/articles/expressroute-howto-circuit-classic/)
[PowerShell - 资源管理器](/documentation/articles/expressroute-howto-circuit-arm/)

本文将指导你执行相关步骤，以便使用 PowerShell cmdlet 和 **经典** 部署模型创建 ExpressRoute 线路。下面的步骤还将向你显示如何查看状态，以及如何更新、删除和预配 ExpressRoute 线路。如果要使用 **资源管理器** 部署模型创建和修改 ExpressRoute 线路，请参阅[使用资源管理器部署模型创建和修改 ExpressRoute 线路](/documentation/articles/expressroute-howto-circuit-arm/)。

[AZURE.INCLUDE [vpn-gateway-clasic-rm](../includes/vpn-gateway-classic-rm-include.md)]


## 开始之前

- 你需要最新版本的 Azure PowerShell 模块。可以从 [Azure 下载页](/downloads/)的 PowerShell 部分下载最新 PowerShell 模块。按照[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/) 上的说明操作，以便获取有关如何配置计算机以使用 Azure PowerShell 模块的分步指导。 
- 在开始配置之前，请务必查看[先决条件](/documentation/articles/expressroute-prerequisites/)和[工作流](/documentation/articles/expressroute-workflows/)。

## 创建和预配 ExpressRoute 线路

1. **为 ExpressRoute 导入 PowerShell 模块。**

 	在开始使用 ExpressRoute cmdlet 之前，必须将 Azure 和 ExpressRoute 模块导入 PowerShell 会话。运行以下命令，将 Azure 和 ExpressRoute 模块导入 PowerShell 会话。

	    Import-Module 'C:\Program Files (x86)\Microsoft SDKs\Azure\PowerShell\ServiceManagement\Azure\Azure.psd1'
	    Import-Module 'C:\Program Files (x86)\Microsoft SDKs\Azure\PowerShell\ServiceManagement\Azure\ExpressRoute\ExpressRoute.psd1'

2. **获取支持的提供商、位置和带宽的列表。**

	在创建 ExpressRoute 线路之前，你需要连接提供商、支持的位置和带宽选项的列表。PowerShell cmdlet *Get-AzureDedicatedCircuitServiceProvider* 会返回此信息，你将在后续步骤中使用此信息。当你运行该 cmdlet 时，结果将类似于下面的示例。

		PS C:\> Get-AzureDedicatedCircuitServiceProvider

		Name                 DedicatedCircuitLocations      DedicatedCircuitBandwidths                                                                                                                                                                                   
		----                 -------------------------      --------------------------                                                                                                                                                                                   
		Beijing Telecom      Beijing                        200Mbps:200, 500Mbps:50,1Gbps:1000,10Gbps:10000                                                                                                                                                   
		Ethernet                                                                                                                                                                                                                                                    
                                                                                                                                                        
    	

3. **创建 ExpressRoute 线路。**

	以下示例演示如何通过北京的 Beijing Telecom Ethernet 创建 200 Mbps 的 ExpressRoute 线路。如果你使用的是其他提供商和其他设置，请在发出请求时替换该信息。

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

4. **列出所有 ExpressRoute 线路。**

	你可以运行 *Get-AzureDedicatedCircuit* 命令，以便获取你所创建的所有 ExpressRoute 线路的列表。


	Get-AzureDedicatedCircuit

	响应将如下例所示：

		Bandwidth                        : 200
		CircuitName                      : 21vDemo
		Location                         : Beijing
		ServiceKey                       : *********************************
		ServiceProviderName              : Beijing Telecom Ethernet
		ServiceProviderProvisioningState : NotProvisioned
		Status                           : Enabled

	你可以随时使用 *Get-AzureDedicatedCircuit* cmdlet 检索此信息。进行不带任何参数的调用将列出所有线路。你的服务密钥将在 *ServiceKey* 字段中列出。

		PS C:\> Get-AzureDedicatedCircuit

		Bandwidth                        : 200
		CircuitName                      : 21vDemo
		Location                         : Beijing
		ServiceKey                       : *********************************
		ServiceProviderName              : Beijing Telecom Ethernet
		ServiceProviderProvisioningState : NotProvisioned
		Status                           : Enabled

	你可以通过运行以下命令获取所有这些参数的详细说明。

		Get-Help Get-AzureDedicatedCircuit -detailed 

5. **将服务密钥发送给连接提供商进行预配。**


    ServiceProviderProvisioningState 提供有关服务提供商端当前预配状态的信息。“状态”提供 Microsoft 端的状态。有关线路预配状态的详细信息，请参阅[工作流](/documentation/articles/expressroute-workflows/#expressroute-circuit-provisioning-states)这篇文章。
    
    当你创建新的 ExpressRoute 线路时，线路将是以下状态：
    
    	ServiceProviderProvisioningState : NotProvisioned
    	Status                           : Enabled
    
    
    当连接服务提供商正在为你启用线路时，线路将转为以下状态：
    
    	ServiceProviderProvisioningState : Provisioning
    	Status                           : Enabled
    
    ExpressRoute 线路处于以下状态时，你才能使用它：
    
    		ServiceProviderProvisioningState : Provisioned
    		
    		Status                           : Enabled



6. **定期检查服务密钥的状态。**

	这样，你就知道提供商何时启用了你的线路。配置线路后，*ServiceProviderProvisioningState* 将显示为 *Provisioned*，如下例所示。

		PS C:\> Get-AzureDedicatedCircuit

		Bandwidth                        : 200
		CircuitName                      : 21vDemo
		Location                         : Beijing
		ServiceKey                       : *********************************
		ServiceProviderName              : Beijing Telecom Ethernet
		ServiceProviderProvisioningState : Provisioned
		Status                           : Enabled

7. **创建路由配置。**
	
	如需分步说明，请参阅 [ExpressRoute 线路路由配置（创建和修改线路对等互连）](/documentation/articles/expressroute-howto-routing-classic/)页。

8. **将 VNet 链接到 ExpressRoute 线路。**

	接下来，将 VNet 链接到 ExpressRoute 线路。如需分步说明，请参阅[将 ExpressRoute 线路链接到 VNet](/documentation/articles/expressroute-howto-linkvnet-classic/)。

##  获取 ExpressRoute 线路的状态

你可以随时使用 *Get-AzureCircuit* cmdlet 检索此信息。进行不带任何参数的调用将列出所有线路。

		PS C:\> Get-AzureDedicatedCircuit

		Bandwidth                        : 200
		CircuitName                      : 21vDemo
		Location                         : Beijing
		ServiceKey                       : *********************************
		ServiceProviderName              : Beijing Telecom Ethernet
		ServiceProviderProvisioningState : Provisioned
		Status                           : Enabled

你可以将服务密钥作为参数传递给调用，从而获取特定 ExpressRoute 线路的相关信息。

		PS C:\> Get-AzureDedicatedCircuit -ServiceKey "*********************************"

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

你可以执行以下操作：

- 在不停机的情况下，增加 ExpressRoute 线路的带宽。

有关限制的详细信息，请参阅 [ExpressRoute 常见问题](/documentation/articles/expressroute-faqs/)页。

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


### 如何更新 ExpressRoute 线路带宽

有关你的提供商的受支持带宽选项，请查看 [ExpressRoute 常见问题](/documentation/articles/expressroute-faqs/)。你可以选取大于现有线路大小的任何大小。

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

现已在 Azure 一侧调整好线路的大小。你必须联系连接提供商，让他们在那一边根据此更改更新配置。请注意，我们将从现在开始按照已更新的带宽选项为你计费。

## 删除和取消预配 ExpressRoute 线路

注意以下事项：

- 必须取消所有虚拟网络与 ExpressRoute 线路的链接，此操作才能成功。如果此操作失败，请查看你是否有虚拟网络链接到了此线路。

- 如果启用了 ExpressRoute 线路服务提供商预配状态，则状态将从启用转为“正在禁用”。你必须通过服务提供商在他们那一侧取消对线路的预配。在服务提供商取消对线路的预配并向我们发送通知之前，我们会继续保留资源并向你收费。

- 如果在你运行上述 cmdlet 之前，服务提供商已取消对线路的预配（服务提供商预配状态已设置为“未预配”），我们会取消对线路的预配，并停止向你收费。

可以通过运行以下命令来删除 ExpressRoute 线路：

	Remove-AzureDedicatedCircuit -ServiceKey "*********************************"



## 后续步骤

创建你的线路后，请确保执行以下操作：

- [创建和修改 ExpressRoute 线路的路由](/documentation/articles/expressroute-howto-routing-classic/)
- [将虚拟网络链接到 ExpressRoute 线路](/documentation/articles/expressroute-howto-linkvnet-classic/)

<!---HONumber=Mooncake_0104_2016-->