<properties 
   pageTitle="在多个订阅之间共享 ExpressRoute 线路 | Windows Azure"
   description="本文指导你完成了在多个 Azure 订阅之间共享 ExpressRoute 线路。"
   services="expressroute"
   documentationCenter="na"
   authors="cherylmc"
   manager="jdial"
   editor="tysonn" />
<tags 
   ms.service="expressroute"
   ms.date="07/20/2015"
   wacn.date="" />

# 在多个订阅之间共享 ExpressRoute 线路

可以在多个订阅之间共享单个 ExpressRoute 线路。**图 1** 显示了在多个订阅之间共享 ExpressRoute 线路的工作原理的简单示意图。

大型云中的每个较小云用于表示属于组织中不同部门的订阅。组织内的每个部门可以使用自己的订阅部署其服务，但可以共享单个 ExpressRoute 专用线路以连接回企业网络。单个部门（在此示例中为：IT 部门）可以拥有 ExpressRoute 专用线路。专用线路的连接和带宽费用将应用于专用线路所有者。组织内的其他订阅可以使用 ExpressRoute 线路。

**图 1**

![订阅共享](./media/expressroute-share-circuit/IC766124.png)

## 管理

*线路所有者*是在其中创建了 ExpressRoute 专用线路的订阅的管理员/共同管理员。线路所有者可以授权其他订阅的管理员/共同管理员（在工作流关系图中称为*线路用户*）使用他们拥有的专用线路。有权使用组织的专用线路的线路用户在获得授权后可以将其订阅中的 VNet 链接到专用线路。

线路所有者有权随时修改和撤消授权。撤消授权将导致从撤消其访问权限的订阅中删除所有链接。

## 工作流

1. 线路所有者可授权其他订阅的管理员使用指定的线路。在下面的示例中，线路 (Contoso IT) 管理员通过指定 Microsoft ID，允许另一个订阅 (Contoso Sales) 的管理员最多将 2 个 VNET 链接到线路。该 cmdlet 不会将电子邮件发送到指定的 Microsoft ID。线路所有者需要显式通知其他订阅所有者：授权已完成。

		PS C:\> New-AzureDedicatedCircuitLinkAuthorization -ServiceKey '6ed7e310-1a02-4261-915f-6ccfedc416f1' -Description 'SalesTeam' -Limit 2 -MicrosoftIds 'salesadmin@contoso.com'
		
		Description         : SalesTeam 
		Limit               : 2 
		LinkAuthorizationId : e2bc2645-6fd4-44a4-94f5-f2e43e6953ed 
		MicrosoftIds        : salesadmin@contoso.com 
		Used                : 0

1. 收到线路所有者的通知后，所授权订阅的管理员可以运行以下 cmdlet 以检索该线路的服务密钥。在此示例中，Contoso Sales 的管理员必须先使用指定的 Microsoft ID salesadmin@contoso.com 登录。

		PS C:\> Get-AzureAuthorizedDedicatedCircuit
		
		Bandwidth                        : 100
		CircuitName                      : ContosoIT
		Location                         : Washington DC
		MaximumAllowedLinks              : 2
		ServiceKey                       : 6ed7e310-1a02-4261-915f-6ccfedc416f1
		ServiceProviderName              : ###########
		ServiceProviderProvisioningState : Provisioned
		Status                           : Enabled
		UsedLinks                        : 0

1. 所授权订阅的管理员将运行以下 cmdlet 以完成链接操作。

		PS C:\> New-AzureDedicatedCircuitLink –servicekey 6ed7e310-1a02-4261-915f-6ccfedc416f1 –VnetName 'SalesVNET1' 
		
			State VnetName 
			----- -------- 
			Provisioned SalesVNET1

这就是所有的操作。Azure 上的 Conto Sales VNet 现已链接到 ContosoIT 创建/拥有的线路。

## 管理授权

线路所有者最多可以与 10 个 Azure 订阅共享线路。线路所有者可以查看已授权哪些用户使用该线路。所有者随时可以撤消授权。如果线路所有者撤消授权（由 LinkAuthorizationId 标识），则该授权允许的所有链接将立即被删除。链接的 VNET 将丢失通过 ExpressRoute 线路建立的与本地网络的连接。

	PS C:\> Get-AzureDedicatedCircuitLinkAuthorization -ServiceKey: 6ed7e310-1a02-4261-915f-6ccfedc416f1 
	
	Description         : EngineeringTeam 
	Limit               : 3 
	LinkAuthorizationId : cc958457-c8c1-4f16-af09-e7f099da64bf 
	MicrosoftIds        : engadmin@contoso.com 
	Used                : 1 
	
	Description         : MarketingTeam 
	Limit               : 1 
	LinkAuthorizationId : d972726f-c7b9-4658-8598-ad3208ac9348 
	MicrosoftIds        : marketingadmin@contoso.com 
	Used                : 0 
	
	Description         : SalesTeam 
	Limit               : 2 
	LinkAuthorizationId : e2bc2645-6fd4-44a4-94f5-f2e43e6953ed 
	MicrosoftIds        : salesadmin@contoso.com 
	Used                : 2 
	
	PS C:\> Remove-AzureDedicatedCircuitLinkAuthorization -ServiceKey '6ed7e310-1a02-4261-915f-6ccfedc416f1' -AuthorizationId 'e2bc2645-6fd4-44a4-94f5-f2e43e6953ed'


下图显示了授权工作流：

![订阅共享工作流](./media/expressroute-share-circuit/IC759525.png)

## 后续步骤

有关 ExpressRoute 的详细信息，请参阅 [ExpressRoute 概述](/documentation/articles/expressroute-introduction)。

<!---HONumber=69-->