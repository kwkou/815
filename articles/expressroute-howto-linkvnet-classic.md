<properties 
   pageTitle="将虚拟网络链接到 ExpressRoute 线路 | Windows Azure"
   description="本文档概述了如何将虚拟网络 (VNet) 链接到 ExpressRoute 线路。"
   services="expressroute"
   documentationCenter="na"
   authors="cherylmc"
   manager="carolz"
   editor=""
   tags="azure-service-management"/>
<tags 
   ms.service="expressroute"
   ms.date="09/21/2015"
   wacn.date="11/02/2015" />

# 将 VNet 链接到 ExpressRoute 线路

本文概述了如何将虚拟网络 (VNet) 链接到 ExpressRoute 线路。虚拟网络可以在同一个订阅中，也可以属于另一个订阅。本文适用于使用典型部署模型部署的 VNet。

>[AZURE.IMPORTANT]请务必了解 Azure 当前使用两种部署模型：资源管理器和经典部署模型。在开始你的配置之前，请确保你了解部署模型和工具。

## 配置先决条件

- 你将需要最新版本的 Azure PowerShell 模块。可以从 [Azure 下载页](/downloads)的 PowerShell 部分下载最新 PowerShell 模块。按照[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure) 页上的说明操作，以便获取有关如何配置计算机以使用 Azure PowerShell 模块的分步指导。 
- 在开始配置之前，请务必查看[先决条件](/documentation/articles/expressroute-prerequisites)页、<!--[-->路由要求<!--](/documentation/articles/expressroute-routing)-->页和<!--[-->工作流<!--](/documentation/articles/expressroute-workflows)-->页。
- 你必须有一个活动的 ExpressRoute 线路。 
	- 请按说明[创建 ExpressRoute 线路](/documentation/articles/expressroute-howto-circuit-classic)，并通过连接提供商启用该线路。 
	- 请确保为线路配置 Azure 专用对等互连。<!--如需路由说明，请参阅[配置路由](/documentation/articles/expressroute-howto-routing-classic)一文。-->
	- 若要启用端到端连接，必须配置 Azure 专用对等互连并运行你的网络和 Microsoft 之间的 BGP 对等互连。

最多可以将 10 个 VNet 链接到一条 ExpressRoute 线路。所有 ExpressRoute 线路必须位于同一个地理政治区域。如果你启用了 ExpressRoute 高级版外接程序，则可将更多数量的虚拟网络连接到 ExpressRoute 线路。有关高级版外接程序的更多详细信息，请参阅[常见问题](/documentation/articles/expressroute-faqs)。

## 将同一 Azure 订阅中的 VNet 链接到 ExpressRoute 线路

你可以使用以下 cmdlet 将虚拟网络链接到 ExpressRoute 线路。在运行 cmdlet 之前，请确保已创建虚拟网络网关并可将其用于进行链接。

	C:\> New-AzureDedicatedCircuitLink -ServiceKey "*****************************" -VNetName "MyVNet"
	Provisioned

## 将其他 Azure 订阅中的 VNet 链接到 ExpressRoute 线路

可以在多个订阅之间共享一个 ExpressRoute 线路。下图是在多个订阅之间共享 ExpressRoute 线路的简单示意图。大型云中的每个较小云用于表示属于组织中不同部门的订阅。组织内的每个部门可以使用自己的订阅部署其服务，但可以共享单个 ExpressRoute 线路以连接回本地网络。单个部门（在此示例中为 IT 部门）可以拥有 ExpressRoute 线路。组织内的其他订阅可以使用 ExpressRoute 线路。

>[AZURE.NOTE]专用线路的连接和带宽费用将应用于 ExpressRoute 线路所有者。所有虚拟网络共享相同的带宽。

![跨订阅连接](./media/expressroute-howto-linkvnet-classic/cross-subscription.png)

### 管理

*线路所有者* 是在其中创建了 ExpressRoute 线路的订阅的管理员/共同管理员。线路所有者可以授权其他订阅的管理员/共同管理员（在工作流关系图中称为*线路用户*）使用他们拥有的专用线路。有权使用组织的 ExpressRoute 线路的线路用户在获得授权后可以将其订阅中的 VNet 链接到 ExpressRoute 线路。

线路所有者有权随时修改和撤消授权。撤消授权将导致从撤消其访问权限的订阅中删除所有链接。

### 线路所有者操作 

#### 创建授权
	
线路所有者可授权其他订阅的管理员使用指定的线路。在下面的示例中，线路 (Contoso IT) 管理员通过指定 Microsoft ID，允许另一个订阅（开发-测试）的管理员最多将 2 个 VNET 链接到线路。该 cmdlet 不会将电子邮件发送到指定的 Microsoft ID。线路所有者需要显式通知其他订阅所有者：授权已完成。

		PS C:\> New-AzureDedicatedCircuitLinkAuthorization -ServiceKey "**************************" -Description "Dev-Test Links" -Limit 2 -MicrosoftIds 'devtest@contoso.com'
		
		Description         : Dev-Test Links 
		Limit               : 2 
		LinkAuthorizationId : ********************************** 
		MicrosoftIds        : devtest@contoso.com 
		Used                : 0

#### 查看授权

线路所有者可以通过运行以下 cmdlet 来查看针对特定线路发出的所有授权。

	PS C:\> Get-AzureDedicatedCircuitLinkAuthorization -ServiceKey: "**************************"
	
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

线路所有者可以使用以下 cmdlet 修改授权。

	PS C:\> set-AzureDedicatedCircuitLinkAuthorization -ServiceKey "**************************" -AuthorizationId "&&&&&&&&&&&&&&&&&&&&&&&&&&&&"-Limit 5
		
	Description         : Dev-Test Links 
	Limit               : 5 
	LinkAuthorizationId : &&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&&& 
	MicrosoftIds        : devtest@contoso.com 
	Used                : 0


#### 删除授权

线路所有者可以通过运行以下 cmdlet 来撤销/删除对用户的授权。

	PS C:\> Remove-AzureDedicatedCircuitLinkAuthorization -ServiceKey "*****************************" -AuthorizationId "###############################"


### 线路用户操作

#### 查看授权

线路用户可以使用以下 cmdlet 查看授权。

	PS C:\> Get-AzureAuthorizedDedicatedCircuit
		
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

线路用户可以通过运行以下 cmdlet 来兑现链接授权。

	PS C:\> New-AzureDedicatedCircuitLink ¨Cservicekey "&&&&&&&&&&&&&&&&&&&&&&&&&&" -VnetName 'SalesVNET1' 
		
	State VnetName 
	----- -------- 
	Provisioned SalesVNET1

## 后续步骤

有关 ExpressRoute 的详细信息，请参阅 [ExpressRoute 常见问题](/documentation/articles/expressroute-faqs)。

<!---HONumber=76-->