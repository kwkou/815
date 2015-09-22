<properties 
   pageTitle="ExpressRoute 简介 | Windows Azure"
   description="本页提供 ExpressRoute 服务的概述，包括 ExpressRoute 连接的工作方式、使用 Exchange 提供商和网络服务提供商，以及 ExpressRoute 公共对等互连、专用对等互连和 Microsoft 对等互连。"
   documentationCenter="na"
   services="expressroute"
   authors="cherylmc"
   manager="adinah"
   editor="tysonn"/>
<tags 
   ms.service="expressroute"
   ms.date="06/15/2015"
   wacn.date=""/>

# ExpressRoute 技术概述

使用 Microsoft Azure ExpressRoute，可在 Azure 数据中心与你的本地环境或共同租用环境中的基础结构之间创建专用连接。使用 ExpressRoute，你可以在 ExpressRoute 合作伙伴共同租用的设施中建立与 Azure 的连接，也可以直接从现有的 WAN 网络（如网络服务提供商提供的 MPLS VPN）连接到 Microsoft 云服务（例如 Microsoft Azure 和 Office 365）。
 
与通过 Internet 的典型连接相比，ExpressRoute 连接提供更高的安全性、更多的可靠性、更快的速度和更少的延迟。在某些情况下，使用 ExpressRoute 连接在本地网络和 Azure 之间传输数据还可以产生显著的成本效益。如果你已创建从本地网络到 Azure 的跨界连接，则可以在不改动虚拟网络的情况下迁移到 ExpressRoute 连接。

有关详细信息，请参阅 [ExpressRoute 常见问题](/documentation/articles/expressroute-faqs)。

## ExpressRoute 连接的工作原理

若要将 WAN 连接到 Windows 云服务，你必须通过连接提供商订购并启用一条专用线路。有两种类型的连接供你选择：通过交换提供商 (EXP) 的第 3 层直接连接，或通过网络服务提供商 (NSP) 的第 3 层连接。你可以选择在 WAN 和 Microsoft 云之间启用一种或两种类型的连接。

## Exchange 提供商和网络服务提供商
ExpressRoute 提供商划分为网络服务提供商 (NSP) 和 Exchange 提供商 (EXP)。

![](./media/expressroute-introduction/expressroute-nsp-exp.png)


| |**Exchange 提供商**|**网络服务提供商**|
|---|---|---|
|**典型连接模型**| 点对点以太网链接或通过云交换连接 | 通过电信公司 VPN 的任意对等连接 |
|**支持的带宽**|200 Mbps、500 Mbps、1 Gbps 和 10 Gbps|10 Mbps、50 Mbps、100 Mbps、500 Mbps、1 Gbps|
|**连接提供商**|[Exchange 提供商](/documentation/articles/expressroute-locations)|[网络服务提供商](/documentation/articles/expressroute-locations)|
|**路由**|直接使用客户边缘路由器建立 BGP 会话| 与电信公司建立 BGP 会话|
|**价格**|[EXP 价格](/home/features/expressroute/#price)|[NSP 价格](/home/features/expressroute/#price)|

### Exchange 提供商 (EXP)
我们与 Equinix 和 TeleCity 集团等云交换服务提供商，以及 Cole 和 Level 3 等点对点连接服务提供商合作，以帮助在 Azure 与客户本地之间建立连接。我们提供从 200 Mbps 到 10 Gbps（200 Mbps、500 Mbps、1 Gbps 和 10 Gbps）的线路带宽。

如果你想要通过交换提供商建立第 3 层直接连接，可以采用下列 3 种方法之一：

- 可以将 Equinix 的 Cloud Exchange 或 TeleCity 的 Cloud IX 设备放置在我们提供服务的位置。在这种情况下，你需要订购云交换设施的冗余连接。 
- 你可以与提供商（例如 Level 3）合作，以便在数据中心与 Microsoft 之间设置以太网线路。 
- 你可以与本地连接提供商合作，以购买最接近交换提供商设施的冗余连接，以及连接到云交换设施。

我们确实需要你建立冗余连接，以满足我们的 SLA 要求。我们不支持直接连接到 Microsoft 边缘。专用线路始终应通过以太网提供商或本地云交换设施启用。尽管这样会在 Microsoft 与你的网络之间建立第 2 层连接，但我们不支持扩展第 2 层域。你必须在边缘路由器与 Microsoft 边缘路由器之间设置冗余路由会话，以建立第 3 层连接。

若要了解有关配置的详细信息并查看实际示例，你可以遵循以下分步指南操作：[通过 Exchange 提供商配置 ExpressRoute 连接](/documentation/articles/expressroute-configuring-exps)。


### 网络服务提供商 (NSP)

我们与 AT&T 和 British Telecom 等电信公司合作，以便在 Azure 与你的 WAN 之间建立连接。我们提供从 10 Mbps 到 1 Gbps（10 Mbps、50 Mbps、100 Mbps 和 500 Mbps、1 Gbps）的线路带宽。

如果你使用了我们一家合作伙伴提供的 VPN 服务，则无需部署任何新硬件或者对现有网络做出重大配置更改，就能将网络扩展到 Azure。

若要了解有关配置的详细信息并查看实际示例，你可以遵循以下分步指南操作：[通过网络服务提供商配置 ExpressRoute 连接](/documentation/articles/expressroute-configuring-nsps)。

## ExpressRoute 对等互连
下图提供了你的 WAN 与 Microsoft 之间的连接的逻辑表示形式。你必须通过连接提供商 (NSP/EXP) 订购一条*专用线路*，才能将 WAN 连接到 Microsoft。专用线路表示通过连接提供商在你的 WAN 与 Microsoft 之间建立的逻辑连接。你可以订购许多的专用线路，每个线路可部署在相同或不同的区域，并可以通过不同的服务提供商连接到你的 WAN。

![](./media/expressroute-introduction/expressroute-basic.png)

一条专用线路具有多个关联的路由域 - 公共域、专用域和 Microsoft 域。在一组路由器上，每个路由域采用相同的配置（主动-主动，或负载分担）以实现高可用性。

![](./media/expressroute-introduction/expressroute-peerings.png)


### 专用对等互连
可以通过专用对等域来连接虚拟网络内部署的 Azure 计算服务（即虚拟机 (IaaS) 和云服务 (PaaS)）。专用对等域被视为进入 Microsoft Azure 的核心网络的受信任扩展。可以在核心网络和 Azure 虚拟网络 (VNet) 之间设置双向连接。这样，将便可以使用专用 IP 地址直接连接到虚拟机和云服务。

可以将多个虚拟网络连接到专用对等域。有关限制和局限性的信息请查看[常见问题页](/documentation/articles/expressroute-faqs)。
  

### 公共对等互连
Azure 存储空间、SQL Database 和网站等服务是使用公共 IP 地址提供的。你可以通过公共对等路由域私下连接到公共 IP 地址（包括云服务的 VIP）上托管的服务。可以将公共对等域连接到 Extranet，并从 WAN 连接到公共 IP 地址上的所有 Azure 服务，而无需通过 Internet 连接。始终会从 WAN 发起到 Windows Azure 服务的连接。Windows Azure 服务无法通过此路由域发起到你网络的连接。启用公共对等互连后，你将能够连接到所有 Azure 服务。我们不允许选择要将路由播发到的服务。可以在 [Windows Azure 数据中心 IP 范围](http://www.microsoft.com/download/details.aspx?id=41653)页上查看我们通过此对等互连播发给你的前缀列表。你可以在网络中定义自定义路由筛选器，以只使用所需的路由。

有关通过公共对等路由域支持的服务的详细信息，请查看[常见问题页](/documentation/articles/expressroute-faqs)。
 
### Windows 对等互连
与其他所有 Windows Online Services（如 Office 365 服务）的连接将通过 Windows 对等互连建立。我们将通过 Windows 对等路由域在你的 WAN 和 Windows 云服务之间启用双向连接。你只能通过由你或者你的连接提供商拥有的公共 IP 地址连接到 Windows 云服务，并且必须遵守我们规定的所有规则。有关详细信息，请查看 [ExpressRoute 先决条件](/documentation/articles/expressroute-prerequisites)页。

有关支持的服务、费用和配置详细信息，请查看[常见问题页](/documentation/articles/expressroute-faqs)。有关提供 Windows 对等互连支持的连接提供商列表，请查看 [ExpressRoute 位置](/documentation/articles/expressroute-locations)页。


下表比较了三种路由域。

||**专用对等互连**|**公共对等互连**|**Windows 对等互连**| |---|---|---|---| |**每个对等互连支持的最大前缀数**|默认为 4000，使用 ExpressRoute 高级版时为 10,000|默认为 4000，使用 ExpressRoute 高级版时为 10,000|200| |**支持的 IP 地址范围**|WAN 中任何有效的 IPv4 地址|由你或你的连接提供商拥有的公共 IPv4 地址|由你或你的连接提供商拥有的公共 IPv4 地址| |**AS 编号要求**|专用和公共 AS 编号。客户必须拥有公共 AS 编号。|专用和公共 AS 编号。客户必须拥有公共 AS 编号。| 仅限公共 AS 编号。必须向路由注册机构验证 AS 编号，以验证所有权。| |**路由接口 IP 地址**|RFC1918 和公共 IP 地址|向客户注册的公共 IP 地址/路由注册机构中的 NSP。| 向客户注册的公共 IP 地址/路由注册机构中的 NSP。| |**MD5 哈希支持**| 是|是|是|

你可以选择启用一个或多个路由域作为专用线路的一部分。如果客户想要引入单个路由域，你可以选择将所有路由域放置在同一个 VPN 中（对于 NSP）。此外，可以像以上示意图中一样，将它们放置在不同的路由域中。建议的配置是将专用对等域直接连接到核心网络，并将公共和 Windows 对等链接连接到 Extranet。
 
如果你选择使用所有三个对等会话，必须使用三对 BGP 会话（每一对用于一个对等类型）。BGP 会话对提供高度可用的链接。如果你要通过 EXP 进行连接，则需要负责配置和管理路由（除非 EXP 可为你管理路由）。如果你选择通过 NSP 进行连接，则可以依赖 NSP 来为你管理路由。可以通过查看设置 ExpressRoute 的工作流来了解详细信息


## 后续步骤

- 查找服务提供商。请参阅 [ExpressRoute 服务提供商和位置](/documentation/articles/expressroute-locations)。
- 配置 ExpressRoute 连接。有关说明，请参阅[通过网络服务提供商配置 ExpressRoute 连接](/documentation/articles/expressroute-configuring-nsps)或[通过 Exchange 提供商配置 ExpressRoute 连接](/documentation/articles/expressroute-configuring-exps)。 

<!---HONumber=69-->