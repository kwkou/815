<properties
   pageTitle="采用 ExpressRoute 所要满足的先决条件 | Azure"
   description="本页提供了在订购 Azure ExpressRoute 线路之前需要满足的要求列表。"
   documentationCenter="na"
   services="expressroute"
   authors="cherylmc"
   manager="carolz"
   editor=""/>
<tags
   ms.service="expressroute"
   ms.date="04/19/2016"
   wacn.date="06/06/2016"/>


# ExpressRoute 先决条件  

若要使用 ExpressRoute 连接到 Azure 服务，你需要确认是否符合以下部分中所列的要求。

## 帐户要求

- 使用中的有效 Azure 帐户。需有此帐户才能设置 ExpressRoute 线路。

## 与连接服务提供商的关系

- 与支持列表中的连接服务提供商建立关系，通过这些提供商便于建立连接。你必须与连接服务提供商存在现有的业务关系。你需要确保通过连接服务提供商获得的服务与 ExpressRoute 兼容。
- 如果你要使用某家网络服务提供商，但该提供商不在上面的列表中，你仍可以创建与 Azure 的连接。
	- 请咨询你的网络服务提供商，让你的网络提供商将你的网络扩展到连接提供商。

## 在你的网络与连接服务提供商之间建立物理连接

有关连接模型的详细信息，请参阅“连接模型”部分。客户必须确保其本地基础结构通过所述的某个模型实际连接到服务提供商基础结构。

## 连接的冗余要求

客户基础结构与服务提供商基础结构之间的物理连接没有任何冗余要求。 Azure 不要求在第 3 层配置冗余。但 Azure 确实要求在 Azure 边缘与客户网络之间，通过服务提供商针对要启用的每个对等互连设置冗余路由。如果未以冗余方式配置路由会话，服务的可用性 SLA 将会失效。

## IP 地址和路由注意事项

使用点到点以太网提供商连接到 Azure 的客户必须为每个对等互连配置冗余 BGP 会话，以符合可用性 SLA 要求。连接服务提供商可能会提供此项增值服务。有关限制的详细信息，请参阅 [ExpressRoute 线路和路由域](/documentation/articles/expressroute-circuit-peerings/)一文中的路由域表。

## 安全与防火墙

有关安全与防火墙信息，请参阅<!-- [-->Microsoft 云服务和网络安全<!--](/documentation/articles/best-practices-network-security/)-->一文。

## 后续步骤

- 有关 ExpressRoute 的详细信息，请参阅 [ExpressRoute 常见问题](/documentation/articles/expressroute-faqs/)。
- 查找 ExpressRoute 连接服务提供商。请参阅 [ExpressRoute 合作伙伴和对等位置](/documentation/articles/expressroute-locations/)。
- 请参阅[路由](/documentation/articles/expressroute-routing/)的要求。
- 配置 ExpressRoute 连接。
	- [创建 ExpressRoute 线路](/documentation/articles/expressroute-howto-circuit-classic/)
	- [配置路由](/documentation/articles/expressroute-howto-routing-classic/)
	- [将 VNet 链接到 ExpressRoute 线路](/documentation/articles/expressroute-howto-linkvnet-classic/)
 

<!---HONumber=Mooncake_0104_2016-->