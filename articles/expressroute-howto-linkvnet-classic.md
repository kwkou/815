<properties
   pageTitle="使用经典部署模型和 PowerShell 将虚拟网络链接到 ExpressRoute 线路 | Azure"
   description="本文档概述如何使用经典部署模型和 PowerShell 将虚拟网络 (VNet) 链接到 ExpressRoute 线路。"
   services="expressroute"
   documentationCenter="na"
   authors="ganesr"
   manager="carmonm"
   editor=""
   tags="azure-service-management"/>
<tags 
   ms.service="expressroute"
   ms.date="04/14/2016"
   wacn.date="06/06/2016" />

# 将 VNet 链接到 ExpressRoute 线路

> [AZURE.SELECTOR]
- [PowerShell - Resource Manager] (/documentation/articles/expressroute-howto-linkvnet-arm/)  
- [PowerShell - Classic](/documentation/articles/expressroute-howto-linkvnet-classic/)



本文将帮助你使用经典部署模型和 PowerShell 将虚拟网络 (VNet) 链接到 Azure ExpressRoute 线路。虚拟网络可以在同一个订阅中，也可以属于另一个订阅。

**关于 Azure 部署模型**

[AZURE.INCLUDE [vpn-gateway-clasic-rm](../includes/vpn-gateway-classic-rm-include.md)]

## 配置先决条件

1. 你需要最新版本的 Azure PowerShell 模块。可以从 [Azure 下载页](/downloads/)的 PowerShell 部分下载最新 PowerShell 模块。按照[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/) 上的说明操作，以便获取有关如何配置计算机以使用 Azure PowerShell 模块的分步指导。 
2. 在开始配置之前，需查看[先决条件](/documentation/articles/expressroute-prerequisites/)、[路由要求](/documentation/articles/expressroute-routing/)和[工作流](/documentation/articles/expressroute-workflows/)。
3. 你必须有一个活动的 ExpressRoute 线路。 
	- 请按说明[创建 ExpressRoute 线路](/documentation/articles/expressroute-howto-circuit-classic/)，并通过连接服务提供商启用该线路。
	- 请确保为线路配置 Azure 专用对等互连。如需路由说明，请参阅[配置路由](/documentation/articles/expressroute-howto-routing-classic/)一文。 
	- 确保配置 Azure 专用对等互连并运行你的网络和 Microsoft 之间的 BGP 对等互连，以便启用端到端连接。
    - 必须已创建并完全预配一个虚拟网络和一个虚拟网络网关。请按说明[为 ExpressRoute 配置虚拟网络](/documentation/articles/expressroute-howto-vnet-portal-classic/)。

最多可以将 10 个虚拟网络链接到一条 ExpressRoute 线路。所有 ExpressRoute 线路必须位于同一个地理政治区域。如果你启用了 ExpressRoute 高级版外接程序，则可将更多数量的虚拟网络连接到 ExpressRoute 线路。有关高级版外接程序的更多详细信息，请参阅[常见问题](/documentation/articles/expressroute-faqs/)。

## 将同一订阅中的虚拟网络连接到线路

你可以使用以下 cmdlet 将虚拟网络链接到 ExpressRoute 线路。在运行 cmdlet 之前，请确保已创建虚拟网络网关并可将其用于进行链接。

	New-AzureDedicatedCircuitLink -ServiceKey "*****************************" -VNetName "MyVNet"
	Provisioned

## 将另一订阅中的虚拟网络连接到线路

你可以在多个订阅之间共享 ExpressRoute 线路。下图是在多个订阅之间共享 ExpressRoute 线路的简单示意图。

大型云中的每个较小云用于表示属于组织中不同部门的订阅。组织内的每个部门可以使用自己的订阅部署其服务，但这些部门可以共享单个 ExpressRoute 线路以连接回本地网络。单个部门（在此示例中为 IT 部门）可以拥有 ExpressRoute 线路。组织内的其他订阅可以使用 ExpressRoute 线路。

>[AZURE.NOTE]专用线路的连接和带宽费用将应用于 ExpressRoute 线路所有者。所有虚拟网络共享相同的带宽。

![跨订阅连接](./media/expressroute-howto-linkvnet-classic/cross-subscription.png)

### 管理

线路所有者是在其中创建了 ExpressRoute 线路的订阅的管理员/共同管理员。线路所有者可以授权其他订阅的管理员/共同管理员（称为线路用户）使用他们拥有的专用线路。有权使用组织的 ExpressRoute 线路的线路用户在获得授权后可以将其订阅中的虚拟网络链接到 ExpressRoute 线路。

线路所有者有权随时修改和撤消授权。撤消授权将导致从撤消其访问权限的订阅中删除所有链接。

### 线路所有者操作 

#### 创建授权

线路所有者可授权其他订阅的管理员使用指定的线路。在下面的示例中，线路 (Contoso IT) 管理员允许另一个订阅（开发-测试）的管理员最多将两个虚拟网络链接到线路。Contoso IT 管理员可以通过指定开发-测试 Microsoft ID 来启用此功能。该 cmdlet 不会将电子邮件发送到指定的 Microsoft ID。线路所有者需要显式通知其他订阅所有者：授权已完成。

	New-AzureDedicatedCircuitLinkAuthorization -ServiceKey "**************************" -Description "Dev-Test Links" -Limit 2 -MicrosoftIds 'devtest@contoso.com'
		
		Description         : Dev-Test Links 
		Limit               : 2 
		LinkAuthorizationId : ********************************** 
		MicrosoftIds        : devtest@contoso.com 
		Used                : 0

#### 查看授权

线路所有者可以通过运行以下 cmdlet 来查看针对特定线路发出的所有授权：

	Get-AzureDedicatedCircuitLinkAuthorization -ServiceKey: "**************************"
	
	Description         : EngineeringTeam 
	Limit               : 3 
	LinkAuthorizationId : #################################### 
	MicrosoftIds        : engadmin@contoso.com 
	Used                : 1 
	
	Description         : MarketingTeam 
	Limit               : 1 
	LinkAuthorizationId : @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@ 
	MicrosoftIds        : marketingadmin@contoso.com 
	Used                : 0 
	
	Description         : Dev-Test Links 
	Limit               : 2 
	LinkAuthorizationId : &&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&& 
	MicrosoftIds        : salesadmin@contoso.com 
	Used                : 2 
	

#### 更新授权

线路所有者可以使用以下 cmdlet 修改授权：

	Set-AzureDedicatedCircuitLinkAuthorization -ServiceKey "**************************" -AuthorizationId "&&&&&&&&&&&&&&&&&&&&&&&&&&&&"-Limit 5
		
	Description         : Dev-Test Links 
	Limit               : 5 
	LinkAuthorizationId : &&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&& 
	MicrosoftIds        : devtest@contoso.com 
	Used                : 0


#### 删除授权

线路所有者可以通过运行以下 cmdlet 来撤消/删除对用户的授权：

	Remove-AzureDedicatedCircuitLinkAuthorization -ServiceKey "*****************************" -AuthorizationId "###############################"


### 线路用户操作

#### 查看授权

线路用户可以使用以下 cmdlet 查看授权：

	Get-AzureAuthorizedDedicatedCircuit
		
	Bandwidth                        : 200
	CircuitName                      : ContosoIT
	Location                         : Washington DC
	MaximumAllowedLinks              : 2
	ServiceKey                       : &&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&
	ServiceProviderName              : equinix
	ServiceProviderProvisioningState : Provisioned
	Status                           : Enabled
	UsedLinks                        : 0

#### 兑现链接授权

线路用户可以通过运行以下 cmdlet 来兑现链接授权：

	New-AzureDedicatedCircuitLink –servicekey "&&&&&&&&&&&&&&&&&&&&&&&&&&" –VnetName 'SalesVNET1'

	State VnetName
	----- --------
	Provisioned SalesVNET1

## 后续步骤

有关 ExpressRoute 的详细信息，请参阅 [ExpressRoute 常见问题](/documentation/articles/expressroute-faqs/)。

<!---HONumber=Mooncake_0104_2016-->