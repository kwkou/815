<properties
   pageTitle="如何使用 PowerShell 为经典部署模型的 ExpressRoute 线路配置路由 |  Azure"
   description="本文将指导你完成创建和预配 ExpressRoute 线路的专用、公共互连的步骤。本文还介绍了如何检查状态，以及如何更新或删除线路的对等互连。"
   documentationCenter="na"
   services="expressroute"
   authors="ganesr"
   manager="carmonm"
   editor=""
   tags="azure-service-management"/>
<tags
   ms.service="expressroute"
   ms.date="04/08/2016"
   wacn.date="05/05/2016"/>

# 使用 PowerShell 创建和修改 ExpressRoute 线路的路由

> [AZURE.SELECTOR]
[PowerShell - Resource Manager](/documentation/articles/expressroute-howto-routing-arm/)
[PowerShell - Classic](/documentation/articles/expressroute-howto-routing-classic/)



本文将指导你执行相关步骤，以便使用 PowerShell 和经典部署模型创建和管理 ExpressRoute 线路的路由配置。下面的步骤还将说明如何查看状态，以及如何更新、删除和取消预配 ExpressRoute 线路的对等互连。


**关于 Azure 部署模型**

[AZURE.INCLUDE [vpn-gateway-clasic-rm](../includes/vpn-gateway-classic-rm-include.md)]

## 配置先决条件

- 你将需要最新版本的 Azure PowerShell 模块。可以从 [Azure 下载页](/downloads/)的 PowerShell 部分下载最新 PowerShell 模块。按照[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/) 页上的说明操作，以便获取有关如何配置计算机以使用 Azure PowerShell 模块的分步指导。 
- 在开始配置之前，请务必查看[先决条件](/documentation/articles/expressroute-prerequisites/)页、[路由要求](/documentation/articles/expressroute-routing/)页和[工作流](/documentation/articles/expressroute-workflows/)页。
- 你必须有一个活动的 ExpressRoute 线路。在继续下一步之前，请按说明[创建 ExpressRoute 线路](/documentation/articles/expressroute-howto-circuit-classic/)，并通过连接提供商启用该线路。ExpressRoute 线路必须处于已预配和已启用状态，你才能运行下述 cmdlet。

>[AZURE.IMPORTANT] 这些说明只适用于由提供第 2 层连接服务的服务提供商创建的线路。如果你的服务提供商提供第 3 层托管服务（通常是 IPVPN，如 MPLS），则连接服务提供商将为你设置和管理路由。

你可以为 ExpressRoute 线路配置一到两个对等互连（Azure 专用和Azure 公共）。可以按照所选的任意顺序配置对等互连。但是，你必须确保一次只完成一个对等互连的配置。

## Azure 专用对等互连

本部分说明如何为 ExpressRoute 线路创建、获取、更新和删除 Azure 专用对等互连配置。

### 创建 Azure 专用对等互连

1. **为 ExpressRoute 导入 PowerShell 模块。**
	
	在开始使用 ExpressRoute cmdlet 之前，必须将 Azure 和 ExpressRoute 模块导入 PowerShell 会话。运行以下命令，将 Azure 和 ExpressRoute 模块导入 PowerShell 会话。

	    Import-Module 'C:\Program Files (x86)\Microsoft SDKs\Azure\PowerShell\ServiceManagement\Azure\Azure.psd1'
	    Import-Module 'C:\Program Files (x86)\Microsoft SDKs\Azure\PowerShell\ServiceManagement\Azure\ExpressRoute\ExpressRoute.psd1'

2. **创建 ExpressRoute 线路。**
	
	请按说明创建 [ExpressRoute 线路](/documentation/articles/expressroute-howto-circuit-classic/)，并由连接服务提供商进行预配。如果你的连接服务提供商提供第 3 层托管服务，你可以请求连接服务提供商为你启用 Azure 专用对等互连。在此情况下，你不需要遵循后续部分中所列的说明。但是，如果你的连接服务提供商不为你管理路由，请在创建线路之后遵循以下说明。

3. **检查 ExpressRoute 线路以确保它已预配。**

	首先必须检查 ExpressRoute 线路是否已预配并已启用。请参阅以下示例。

		PS C:\> Get-AzureDedicatedCircuit -ServiceKey "*********************************"

		Bandwidth                        : 200
		CircuitName                      : MyTestCircuit
		Location                         : Beijing
		ServiceKey                       : *********************************
		ServiceProviderName              : Beijing Telecom Ethernet
		ServiceProviderProvisioningState : Provisioned
		Sku                              : Standard
		Status                           : Enabled

	确保线路显示为已预配并已启用。否则，请与连接服务提供商合作，使线路变为所需的状态。

		ServiceProviderProvisioningState : Provisioned
		Status                           : Enabled


4. **配置线路的 Azure 专用对等互连。**

	在继续执行后续步骤之前，请确保已准备好以下各项：

	- 主链路的 /30 子网。它不能是保留给虚拟网络使用的任何地址空间的一部分。
	- 辅助链路的 /30 子网。它不能是保留给虚拟网络使用的任何地址空间的一部分。
	- 用于建立此对等互连的有效 VLAN ID。请确保线路中没有其他对等互连使用同一个 VLAN ID。
	- 对等互连的 AS 编号。可以使用 2 字节和 4 字节 AS 编号。可以将专用 AS 编号用于此对等互连。请务必不要使用 65515。
	- MD5 哈希（如果选择使用）。**这是可选的**。
	
	可以运行以下 cmdlet 来为线路配置 Azure 专用对等互连。

		New-AzureBGPPeering -AccessType Private -ServiceKey "*********************************" -PrimaryPeerSubnet "10.0.0.0/30" -SecondaryPeerSubnet "10.0.0.4/30" -PeerAsn 1234 -VlanId 100

	如果选择使用 MD5 哈希，则可以使用以下 cmdlet。

		New-AzureBGPPeering -AccessType Private -ServiceKey "*********************************" -PrimaryPeerSubnet "10.0.0.0/30" -SecondaryPeerSubnet "10.0.0.4/30" -PeerAsn 1234 -VlanId 100 -SharedKey "A1B2C3D4"

	>[AZURE.IMPORTANT]请确保将 AS 编号指定为对等互连 ASN 而不是客户 ASN。

### 查看 Azure 专用对等互连详细信息

可以使用以下 cmdlet 来获取配置详细信息

	Get-AzureBGPPeering -AccessType Private -ServiceKey "*********************************"
	
	AdvertisedPublicPrefixes       : 
	AdvertisedPublicPrefixesState  : Configured
	AzureAsn                       : 12076
	CustomerAutonomousSystemNumber : 
	PeerAsn                        : 1234
	PrimaryAzurePort               : 
	PrimaryPeerSubnet              : 10.0.0.0/30
	RoutingRegistryName            : 
	SecondaryAzurePort             : 
	SecondaryPeerSubnet            : 10.0.0.4/30
	State                          : Enabled
	VlanId                         : 100


### 更新 Azure 专用对等互连配置

可以使用以下 cmdlet 来更新配置的任何部分。在以下示例中，线路的 VLAN ID 将从 100 更新为 500。

	Set-AzureBGPPeering -AccessType Private -ServiceKey "*********************************" -PrimaryPeerSubnet "10.0.0.0/30" -SecondaryPeerSubnet "10.0.0.4/30" -PeerAsn 1234 -VlanId 500 -SharedKey "A1B2C3D4"

### 删除 Azure 专用对等互连

可以运行以下 cmdlet 来删除对等互连配置。

>[AZURE.WARNING]运行此 cmdlet 之前，必须确保已从 ExpressRoute 线路取消链接所有虚拟网络。

	Remove-AzureBGPPeering -AccessType Private -ServiceKey "*********************************"


## Azure 公共对等互连

本部分说明如何为 ExpressRoute 线路创建、获取、更新和删除 Azure 公共对等互连配置。

### 创建 Azure 公共对等互连

1. **为 ExpressRoute 导入 PowerShell 模块。**
	
	在开始使用 ExpressRoute cmdlet 之前，必须将 Azure 和 ExpressRoute 模块导入 PowerShell 会话。运行以下命令，将 Azure 和 ExpressRoute 模块导入 PowerShell 会话。

	    Import-Module 'C:\Program Files (x86)\Microsoft SDKs\Azure\PowerShell\ServiceManagement\Azure\Azure.psd1'
	    Import-Module 'C:\Program Files (x86)\Microsoft SDKs\Azure\PowerShell\ServiceManagement\Azure\ExpressRoute\ExpressRoute.psd1'

2. **创建 ExpressRoute 线路**
	
	请按说明创建 [ExpressRoute 线路](/documentation/articles/expressroute-howto-circuit-classic/)，并由连接服务提供商进行预配。如果你的连接服务提供商提供第 3 层托管服务，你可以请求连接服务提供商为你启用 Azure 专用对等互连。在此情况下，你不需要遵循后续部分中所列的说明。但是，如果你的连接服务提供商不为你管理路由，请在创建线路之后遵循以下说明。

3. **检查 ExpressRoute 线路以确保它已预配**

	首先必须检查 ExpressRoute 线路是否已预配并已启用。请参阅以下示例。

		PS C:\> Get-AzureDedicatedCircuit -ServiceKey "*********************************"

		Bandwidth                        : 200
		CircuitName                      : MyTestCircuit
		Location                         : Beijing
		ServiceKey                       : *********************************
		ServiceProviderName              : Beijing Telecom Ethernet
		ServiceProviderProvisioningState : Provisioned
		Sku                              : Standard
		Status                           : Enabled

	确保线路显示为已预配并已启用。否则，请与连接服务提供商合作，使线路变为所需的状态。

		ServiceProviderProvisioningState : Provisioned
		Status                           : Enabled

	

4. **配置线路的 Azure 公共对等互连**

	在继续下一步之前，请确保已准备好以下信息。

	- 主链路的 /30 子网。这必须是有效的公共 IPv4 前缀。
	- 辅助链路的 /30 子网。这必须是有效的公共 IPv4 前缀。
	- 用于建立此对等互连的有效 VLAN ID。请确保线路中没有其他对等互连使用同一个 VLAN ID。
	- 对等互连的 AS 编号。可以使用 2 字节和 4 字节 AS 编号。
	- MD5 哈希（如果选择使用）。**这是可选的**。
	
	可以运行以下 cmdlet 来为线路配置 Azure 公共对等互连

		New-AzureBGPPeering -AccessType Public -ServiceKey "*********************************" -PrimaryPeerSubnet "131.107.0.0/30" -SecondaryPeerSubnet "131.107.0.4/30" -PeerAsn 1234 -VlanId 200

	如果选择使用 MD5 哈希，则可以使用以下 cmdlet

		New-AzureBGPPeering -AccessType Public -ServiceKey "*********************************" -PrimaryPeerSubnet "131.107.0.0/30" -SecondaryPeerSubnet "131.107.0.4/30" -PeerAsn 1234 -VlanId 200 -SharedKey "A1B2C3D4"

	>[AZURE.IMPORTANT]请确保将 AS 编号指定为对等互连 ASN 而不是客户 ASN。

### 查看 Azure 公共对等互连详细信息

可以使用以下 cmdlet 来获取配置详细信息

	Get-AzureBGPPeering -AccessType Public -ServiceKey "*********************************"
	
	AdvertisedPublicPrefixes       : 
	AdvertisedPublicPrefixesState  : Configured
	AzureAsn                       : 12076
	CustomerAutonomousSystemNumber : 
	PeerAsn                        : 1234
	PrimaryAzurePort               : 
	PrimaryPeerSubnet              : 131.107.0.0/30
	RoutingRegistryName            : 
	SecondaryAzurePort             : 
	SecondaryPeerSubnet            : 131.107.0.4/30
	State                          : Enabled
	VlanId                         : 200


### 更新 Azure 公共对等互连配置

可以使用以下 cmdlet 来更新配置的任何部分

	Set-AzureBGPPeering -AccessType Public -ServiceKey "*********************************" -PrimaryPeerSubnet "131.107.0.0/30" -SecondaryPeerSubnet "131.107.0.4/30" -PeerAsn 1234 -VlanId 600 -SharedKey "A1B2C3D4"

在上面的示例中，线路的 VLAN ID 将从 200 更新为 600。

### 删除 Azure 公共对等互连

可以运行以下 cmdlet 来删除对等互连配置

	Remove-AzureBGPPeering -AccessType Public -ServiceKey "*********************************"

## 后续步骤

下一步，[将 VNet 链接到 ExpressRoute 线路](/documentation/articles/expressroute-howto-linkvnet-classic/)。

- 有关工作流的详细信息，请参阅 [ExpressRoute 工作流](/documentation/articles/expressroute-workflows/)。
- 有关线路对等互连的详细信息，请参阅 [ExpressRoute 线路和路由域](/documentation/articles/expressroute-circuit-peerings/)。

<!---HONumber=Mooncake_1207_2015-->