<properties
   pageTitle="采用 ExpressRoute 所要满足的先决条件"
   description="本页提供了在订购 ExpressRoute 线路之前需要满足的要求列表"
   documentationCenter="na"
   services="expressroute"
   authors="cherylmc"
   manager="carolz"
   editor="tysonn"/>
<tags
   ms.service="expressroute"
   ms.date="07/28/2015"
   wacn.date=""/>


# ExpressRoute 先决条件  

若要使用 ExpressRoute 连接到 Microsoft 云服务，你需要确认是否满足以下先决条件：

## 连接先决条件

- 有效且处于活动状态的 Microsoft Azure 帐户
- 与上述[支持列表](/documentation/articles/expressroute-locations)中的网络服务提供商 (NSP) 或 Exchange 提供商 (EXP) 的关系，通过这些提供商便于实现连接要求。你必须与网络服务提供商或 Exchange 提供商存在现有业务关系。你必须确保所使用的服务与 ExpressRoute 兼容。 
- 如果你想要使用某家网络服务提供商，但该提供商不在上面的列表中，你仍可以创建与 Azure 的连接。
	- 请咨询你的网络提供商，以确定它们是否处在上面所列的 Exchange 位置中。
	- 让你的网络提供商将你的网络扩展到选择的 Exchange 位置。
	- 从 Exchange 提供商处订购一条 ExpressRoute 线路以连接到 Azure。
- 连接到服务提供商的基础结构。你必须至少满足以下所列项目之一的条件：
	- 你是网络服务提供商的 VPN 客户，并且至少已将一个本地站点连接到网络服务提供商的 VPN 基础结构。咨询你的网络服务提供商，以了解你的 VPN 服务是否满足 ExpressRoute 的要求
	- 你的基础结构已并置在 Exchange 提供商的数据中心。
	- 你已与 Exchange 提供商的以太网交换基础结构建立以太网连接。
- 用于路由配置的 IP 地址和 AS 编号。
	- 你可以使用专用 AS 编号连接到 Azure 专用对等路由域。如果你选择这样做，则该编号必须大于 65000。有关 AS 编号的详细信息，请参阅[自治系统 (AS) 编号](http://www.iana.org/assignments/as-numbers/as-numbers.xhtml)。
	- 用于配置路由的 IP 地址。/28 子网是必需的。该子网不能与本地或在 Azure 中使用的任何 IP 地址范围重叠。
	- 你必须使用自己的公共 AS 编号为 BGP 会话配置 Azure 公共服务。

## 后续步骤

- 有关 ExpressRoute 的详细信息，请参阅 [ExpressRoute 常见问题](/documentation/articles/expressroute-faqs)。
- 有关如何配置 ExpressRoute 连接的信息，请参阅：
	- [通过网络服务提供商配置 ExpressRoute 连接](/documentation/articles/expressroute-configuring-nsps)
	- [通过 Exchange 提供商配置 ExpressRoute 连接](/documentation/articles/expressroute-configuring-exps)
 

<!---HONumber=69-->