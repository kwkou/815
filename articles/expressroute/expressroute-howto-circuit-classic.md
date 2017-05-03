<properties
    pageTitle="创建和修改 ExpressRoute 线路：PowerShell：Azure 经典 | Azure"
    description="本文介绍完成创建和预配 ExpressRoute 线路的步骤。 本文还介绍了如何检查状态，以及如何更新、删除和预配线路。"
    documentationcenter="na"
    services="expressroute"
    author="ganesr"
    manager="timlt"
    editor=""
    tags="azure-service-management"
    translationtype="Human Translation" />
<tags
    ms.assetid="0134d242-6459-4dec-a2f1-4657c3bc8b23"
    ms.service="expressroute"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="03/21/2017"
    wacn.date="05/02/2017"
    ms.author="ganesr;cherylmc"
    ms.sourcegitcommit="eece5b23ce0c05c9e6e8a1938c34faf0383fb06a"
    ms.openlocfilehash="58cd1a83f45a2bcaa1988d7adb4e03a0b153843c"
    ms.lasthandoff="04/25/2017" />

# <a name="create-and-modify-an-expressroute-circuit-using-powershell-classic"></a>使用 PowerShell 创建和修改 ExpressRoute 线路（经典）
> [AZURE.SELECTOR]
- [Resource Manager - Azure 门户预览](/documentation/articles/expressroute-howto-circuit-portal-resource-manager/)
- [Resource Manager - PowerShell](/documentation/articles/expressroute-howto-circuit-arm/)
- [经典 - PowerShell](/documentation/articles/expressroute-howto-circuit-classic/)

本文介绍使用 PowerShell cmdlet 和经典部署模型创建 Azure ExpressRoute 线路的相关步骤。 本文还介绍如何查看状态，以及如何更新、删除和取消预配 ExpressRoute 线路。

[AZURE.INCLUDE [expressroute-classic-end-include](../../includes/expressroute-classic-end-include.md)]


**关于 Azure 部署模型**

[AZURE.INCLUDE [vpn-gateway-clasic-rm](../../includes/vpn-gateway-classic-rm-include.md)]

## <a name="before-you-begin"></a>开始之前
### <a name="step-1-review-the-prerequisites-and-workflow-articles"></a>步骤 1. 查看先决条件和工作流文章

在开始配置之前，请务必查看[先决条件](/documentation/articles/expressroute-prerequisites/)和[工作流](/documentation/articles/expressroute-workflows/)。

### <a name="step-2-install-the-latest-versions-of-the-azure-service-management-sm-powershell-modules"></a>步骤 2. 安装最新版本的 Azure 服务管理 (SM) PowerShell 模块

有关如何配置计算机以使用 Azure PowerShell 模块的分步指导，请遵循 [Azure PowerShell cmdlet 入门](/documentation/articles/powershell-install-configure/)中的说明。

### <a name="step-3-log-in-to-your-azure-account-and-select-a-subscription"></a>步骤 3. 登录到 Azure 帐户并选择订阅
1. 使用提升的权限打开 PowerShell 控制台，然后连接到帐户。 使用下面的示例来帮助连接：

        Login-AzureRmAccount -Environment $(Get-AzureRmEnvironment -Name AzureChinaCloud)

2. 检查该帐户的订阅。

        Get-AzureRmSubscription

3. 如果有多个订阅，请选择要使用的订阅。

        Select-AzureRmSubscription -SubscriptionName "Replace_with_your_subscription_name"

4. 接下来，使用以下 cmdlet 将 Azure 订阅添加到经典部署模型的 PowerShell。

        Add-AzureAccount -Environment AzureChinaCloud

## <a name="create-and-provision-an-expressroute-circuit"></a>创建和预配 ExpressRoute 线路
### <a name="step-1-import-the-powershell-modules-for-expressroute"></a>步骤 1. 为 ExpressRoute 导入 PowerShell 模块
 在开始使用 ExpressRoute cmdlet 之前，必须将 Azure 和 ExpressRoute 模块导入 PowerShell 会话（如果尚未导入）。 将模块从其安装位置导入本地计算机。 根据模块的安装方法，该位置可能与下例中所示不同。 请根据需要修改示例。  

    Import-Module 'C:\Program Files (x86)\Microsoft SDKs\Azure\PowerShell\ServiceManagement\Azure\Azure.psd1'
    Import-Module 'C:\Program Files (x86)\Microsoft SDKs\Azure\PowerShell\ServiceManagement\Azure\ExpressRoute\ExpressRoute.psd1'

### <a name="step-2-get-the-list-of-supported-providers-locations-and-bandwidths"></a>步骤 2. 获取支持的提供商、位置和带宽的列表
在创建 ExpressRoute 线路之前，需要支持的连接服务提供商、位置和带宽选项的列表。

PowerShell cmdlet `Get-AzureDedicatedCircuitServiceProvider` 将返回此信息，你将在后面的步骤中使用该信息：

    Get-AzureDedicatedCircuitServiceProvider

检查连接服务提供商是否已在该处列出。请记下以下信息，因为稍后创建线路时需要用到：

- 名称

- PeeringLocations

- BandwidthsOffered

现在，已经准备创建 ExpressRoute 线路。

### <a name="step-3-create-an-expressroute-circuit"></a>步骤 3. 创建 ExpressRoute 线路
以下示例演示如何在北京通过 Beijing Telecom Ethernet 创建 200-Mbps 的 ExpressRoute 线路。如果使用其他提供商和其他设置，请在发出请求时替换该信息。

>[AZURE.IMPORTANT] 从发出服务密钥时开始，将对 ExpressRoute 线路进行计费。确保在连接服务提供商准备预配线路时执行此操作。


下面是请求新的服务密钥的示例：

	#Creating a new circuit
	$Bandwidth = 200
	$CircuitName = "21vDemo"
	$ServiceProvider = "Beijing Telecom Ethernet"
	$Location = "Beijing"

	New-AzureDedicatedCircuit -CircuitName $CircuitName -ServiceProviderName $ServiceProvider -Bandwidth $Bandwidth -Location $Location -sku Standard -BillingType MeteredData

或者，如果想要通过高级版外接程序创建 ExpressRoute 线路，则可使用以下示例。 请参阅 [ExpressRoute 常见问题解答](/documentation/articles/expressroute-faqs/)，了解有关高级外接程序的更多详细信息。

	New-AzureDedicatedCircuit -CircuitName $CircuitName -ServiceProviderName $ServiceProvider -Bandwidth $Bandwidth -Location $Location -sku Premium - BillingType MeteredData


响应将包含服务密钥。可以通过运行以下命令获取所有这些参数的详细说明。

	Get-Help New-AzureDedicatedCircuit -detailed 

### <a name="step-4-list-all-the-expressroute-circuits"></a>步骤 4. 列出所有 ExpressRoute 线路
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

可以随时使用 `Get-AzureDedicatedCircuit` cmdlet 检索此信息。 进行不带任何参数的调用将列出所有线路。 服务密钥将在 ServiceKey 字段中列出。

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

    	get-help get-azurededicatedcircuit -detailed

### <a name="step-5-send-the-service-key-to-your-connectivity-provider-for-provisioning"></a>步骤 5. 将服务密钥发送给连接服务提供商进行预配

ServiceProviderProvisioningState 提供有关服务提供商端当前预配状态的信息。 状态提供 Microsoft 端的状态。 有关线路预配状态的详细信息，请参阅[工作流](/documentation/articles/expressroute-workflows/#expressroute-circuit-provisioning-states)一文。

当你创建新的 ExpressRoute 线路时，线路将是以下状态：

	ServiceProviderProvisioningState : NotProvisioned
	Status                           : Enabled


当连接服务提供商正在启用线路时，线路将转为以下状态：

	ServiceProviderProvisioningState : Provisioning
	Status                           : Enabled

ExpressRoute 线路必须处于以下状态时才能使用：

		ServiceProviderProvisioningState : Provisioned
		
		Status                           : Enabled


### <a name="step-6-periodically-check-the-status-and-the-state-of-the-circuit-key"></a>步骤 6. 定期检查线路密钥的状态
这样，你就知道提供商何时启用了你的线路。 配置线路后，ServiceProviderProvisioningState 将显示为 Provisioned，如以下示例所示：

	Get-AzureDedicatedCircuit

		Bandwidth                        : 200
		CircuitName                      : 21vDemo
		Location                         : Beijing
		ServiceKey                       : *********************************
		ServiceProviderName              : Beijing Telecom Ethernet
		ServiceProviderProvisioningState : Provisioned
		Sku                              : Standard
		Status                           : Enabled

### <a name="step-7-create-your-routing-configuration"></a>步骤 7. 创建路由配置

有关分步说明，请参阅 [ExpressRoute 线路路由配置（创建和修改线路对等互连）](/documentation/articles/expressroute-howto-routing-classic/)一文。

> [AZURE.IMPORTANT]
> 这些说明只适用于由提供第 2 层连接服务的服务提供商创建的线路。 如果服务提供商提供第 3 层托管服务（通常是 IP VPN，如 MPLS），则连接服务提供商将配置和管理路由。
> 
> 

### <a name="step-8-link-a-virtual-network-to-an-expressroute-circuit"></a>步骤 8. 将虚拟网络链接到 ExpressRoute 线路

接下来，将虚拟网络链接到 ExpressRoute 线路。 有关分步说明，请参阅[将 ExpressRoute 线路链接到虚拟网络](/documentation/articles/expressroute-howto-linkvnet-classic/)。 如需使用经典部署模型为 ExpressRoute 创建虚拟网络，请参阅[为 ExpressRoute 创建虚拟网络](/documentation/articles/expressroute-howto-vnet-portal-classic/) 

## <a name="getting-the-status-of-an-expressroute-circuit"></a>获取 ExpressRoute 线路的状态
可以随时使用 `Get-AzureCircuit` cmdlet 检索此信息。 进行不带任何参数的调用将列出所有线路。

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


可以通过运行以下示例获取所有这些参数的详细说明：

		get-help get-azurededicatedcircuit -detailed 

## <a name="modifying-an-expressroute-circuit"></a>修改 ExpressRoute 线路
可以在不影响连接的情况下修改 ExpressRoute 线路的某些属性。

你可以在不停机的情况下执行以下操作：

* 为 ExpressRoute 线路启用或禁用 ExpressRoute 高级版外接程序。
* 增加 ExpressRoute 线路的带宽，前提是端口上有可用容量。 请注意，不支持对线路的带宽进行降级。 
* 将计量套餐从数据流量套餐更改为无限制流量套餐。 请注意，不支持将计量套餐从无限制流量套餐更改为数据流量套餐。
* 可以启用和禁用允许经典操作。

有关限制和局限性的详细信息，请参阅 [ExpressRoute 常见问题](/documentation/articles/expressroute-faqs/)。

### <a name="to-enable-the-expressroute-premium-add-on"></a>启用 ExpressRoute 高级外接程序
可以使用以下 PowerShell cmdlet 为现有线路启用 ExpressRoute 高级外接程序：

	Set-AzureDedicatedCircuitProperties -ServiceKey "*********************************" -Sku Premium

	Bandwidth                        : 1000
	CircuitName                      : TestCircuit
	Location                         : Beijing
	ServiceKey                       : *********************************
	ServiceProviderName              : Beijing Telecom Ethernet
	ServiceProviderProvisioningState : Provisioned
	Sku                              : Premium
	Status                           : Enabled

现在，线路已启用 ExpressRoute 高级外接程序功能。请注意，该命令成功运行后，将立即开始对高级外接程序功能计费。

### <a name="to-disable-the-expressroute-premium-add-on"></a>禁用 ExpressRoute 高级外接程序

> [AZURE.IMPORTANT]
> 如果使用的资源超出标准线路允许的范围，此操作可能会失败。
> 
> 

#### <a name="considerations"></a>注意事项

- 从高级版降级到标准版之前，必须确保链接到线路的虚拟机数少于 10。否则，更新请求会失败，并且将按高级版费率收费。

- 必须取消其他地理政治区域的所有虚拟网络的链接。否则，更新请求会失败，并且将按高级版费率收费。

- 路由表中专用对等互连的路由必须少于 4,000。如果路由表大小超出 4,000 个路由，则会删除 BGP 会话且不会重新启用，除非已播发前缀的数目低于 4,000。

#### <a name="disable-the-premium-add-on"></a>禁用高级外接程序
可以使用以下 PowerShell cmdlet 为现有线路禁用 ExpressRoute 高级外接程序：

	Set-AzureDedicatedCircuitProperties -ServiceKey "*********************************" -Sku Standard

	Bandwidth                        : 1000
	CircuitName                      : TestCircuit
	Location                         : Bejing
	ServiceKey                       : *********************************
	ServiceProviderName              : Beijing Telecom Ethernet
	ServiceProviderProvisioningState : Provisioned
	Sku                              : Standard
	Status                           : Enabled


### <a name="to-update-the-expressroute-circuit-bandwidth"></a>更新 ExpressRoute 线路带宽
有关提供商支持的带宽选项，请查看 [ExpressRoute 常见问题](/documentation/articles/expressroute-faqs/)。 只要在其上创建线路的物理端口允许，即可选取大于现有线路大小的任何大小。

> [AZURE.IMPORTANT]
> 如果现有端口上的容量不足，可能需要重新创建 ExpressRoute 线路。 如果该位置没有额外的可用容量，则不能升级线路。
>
> 但是，你无法在不中断的情况下降低 ExpressRoute 线路的带宽。 带宽降级需要取消对 ExpressRoute 线路的预配，然后重新预配新的 ExpressRoute 线路。
> 
> 

#### <a name="resize-a-circuit"></a>调整线路大小

确定所需的大小后，可以使用以下命令调整线路的大小：

	Set-AzureDedicatedCircuitProperties -ServiceKey ********************************* -Bandwidth 1000
		
		Bandwidth                        : 1000
		CircuitName                      : TestCircuit
		Location                         : Beijing 
		ServiceKey                       : *********************************
		ServiceProviderName              : Beijing Telecom Ethernet
		ServiceProviderProvisioningState : Provisioned
		Sku                              : Standard
		Status                           : Enabled

已经在 Azure 端估计线路的大小。必须联系连接提供商，以便根据此更改更新其配置。请注意，将从现在开始按照已更新的带宽选项计费。

如果在增加线路带宽时看到以下错误，这意味着创建现有线路的物理端口上没有足够的带宽可用。必须删除此线路，然后创建所需大小的新线路。

	Set-AzureDedicatedCircuitProperties : InvalidOperation : Insufficient bandwidth available to perform this circuit
	update operation
	At line:1 char:1
	+ Set-AzureDedicatedCircuitProperties -ServiceKey ********************* ...
	+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	    + CategoryInfo          : CloseError: (:) [Set-AzureDedicatedCircuitProperties], CloudException
	    + FullyQualifiedErrorId : Microsoft.WindowsAzure.Commands.ExpressRoute.SetAzureDedicatedCircuitPropertiesCommand
	

## <a name="deprovisioning-and-deleting-an-expressroute-circuit"></a>取消预配和删除 ExpressRoute 线路

### <a name="considerations"></a>注意事项

- 必须取消所有虚拟网络与 ExpressRoute 线路的链接，才能成功执行此操作。 如果此操作失败，请查看你是否有虚拟网络链接到了此线路。

- 如果 ExpressRoute 线路服务提供商预配状态为“正在预配”或“已预配”，则必须与服务提供商合作，在他们一端取消预配线路。 在服务提供商取消对线路的预配并通知我们之前，我们会继续保留资源并向你收费。

- 如果服务提供商已取消预配线路（服务提供商预配状态设置为“未预配”），则可以删除线路。 这样就会停止线路计费。

#### <a name="delete-a-circuit"></a>删除线路

可以通过运行以下命令删除 ExpressRoute 线路：

    Remove-AzureDedicatedCircuit -ServiceKey "*********************************"


## <a name="next-steps"></a>后续步骤
创建你的线路后，请确保执行以下操作：

- [创建和修改 ExpressRoute 线路的路由](/documentation/articles/expressroute-howto-routing-classic/)
- [将虚拟网络链接到 ExpressRoute 线路](/documentation/articles/expressroute-howto-linkvnet-classic/)

<!---HONumber=Mooncake_0320_2017-->