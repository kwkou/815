<properties 
   pageTitle="ExpressRoute 概述：通过专用连接将本地网络扩展到 Azure | Azure"
   description="此 ExpressRoute 技术概述介绍 ExpressRoute 连接如何通过专用连接将本地网络扩展到 Azure。"
   documentationCenter="na"
   services="expressroute"
   authors="cherylmc"
   manager="timlt"
   editor=""/>
<tags 
   ms.service="expressroute"
   ms.devlang="na"
   ms.topic="get-started-article" 
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services" 
   ms.date="02/09/2017"
   wacn.date="03/24/2017"
   ms.author="cherylmc"/>


# ExpressRoute 技术概述
Azure ExpressRoute 可让你通过连接服务提供商所提供的专用连接，将本地网络扩展到 Azure 云。使用 ExpressRoute 可与 Azure云服务建立连接。

ExpressRoute 连接不通过公共 Internet 。与通过 Internet 的典型连接相比，ExpressRoute 连接提供更高的可靠性、更快的速度、更低的延迟和更高的安全性。

有关使用 ExpressRoute 将本地网络连接到 Azure 的详细信息，请参阅 [ExpressRoute 连接模型](/documentation/articles/expressroute-connectivity-models/)。

![](./media/expressroute-introduction/expressroute-connection-overview-diagram.png)  


## 主要优势

- 通过连接服务提供商在本地网络与 Azure 云之间建立第 3 层连接。可以通过点到点以太网，或通过以太网交换经由虚拟交叉连接来建立这种连接。
- 跨地缘政治区域中的所有区域连接到 Azure 云服务。
- 通过 ExpressRoute 高级版附加组件从全球连接到所有区域的 Azure 服务。
- 通过行业标准协议 (BGP) 在你的网络与 Azure 之间进行动态路由。
- 在每个对等位置提供内置冗余以提高可靠性。
- 连接运行时间 [SLA](/support/legal/sla/)。

有关详细信息，请参阅 [ExpressRoute 常见问题解答](/documentation/articles/expressroute-faqs/)。

## 功能

### 第 3 层连接

Azure 采用行业标准动态路由协议 (BGP)，在本地网络、Azure 中的实例和 Azure 公共地址之间交换路由。我们根据不同的流量配置文件来与网络建立多个 BGP 会话。有关详细信息，请参阅 [ExpressRoute 线路和路由域](/documentation/articles/expressroute-circuit-peerings/)一文。

### 冗余

每个 ExpressRoute 线路有两道连接，用于从连接服务提供商/你的网络边缘连接到两个 Azure 企业边缘路由器 (MSEE)。 Azure 要求从连接服务提供商/你的一端建立双重 BGP 连接 – 各自连接到每个 MSEE。你可以选择不要在你的一端部署冗余设备/以太网路线。但是，连接服务提供商会使用冗余设备，确保以冗余方式将你的连接移交给 Azure。冗余的第 3 层连接配置是 Azure [SLA](/support/legal/sla/) 生效的条件。

### 与 Azure 云服务建立连接

通过 ExpressRoute 连接可访问以下服务。

- Azure 服务

 
你可以访问 [ExpressRoute 常见问题](/documentation/articles/expressroute-faqs/)页，以获取通过 ExpressRoute 支持的服务的详细列表。

### 与地缘政治区域中的所有区域建立连接

可以在我们的某个[对等位置](/documentation/articles/expressroute-locations/)连接到 Azure，然后访问该地缘政治区域中的所有区域。

例如，如果你在北京通过 ExpressRoute 连接到 Azure，则就能够访问在上海托管的所有 Azure 云服务。



### 带宽选项
你可以购买各种带宽的 ExpressRoute 线路。支持的带宽列表如下。请务必咨询连接服务提供商，以获取他们支持的带宽列表。

- 50 Mbps
- 100 Mbps
- 200 Mbps
- 500 Mbps
- 1 Gbps
- 2 Gbps
- 5 Gbps
- 10 Gbps

### 动态调整带宽
可以在不中断连接的情况下增大 ExpressRoute 线路带宽（尽最大努力）。

### 弹性计费模式
你可以选择最适合自己的计费模式。请从以下计费模式中选择。有关详细信息，请参阅 [ExpressRoute 常见问题解答](/documentation/articles/expressroute-faqs/)。

- **无限制数据**。ExpressRoute 线路按月计费，所有入站和出站数据传输不收取费用。 
- **计量数据**。ExpressRoute 线路按月计费。所有入站数据传输免费。出站数据传输按每 GB 数据传输计费。数据传输费率根据区域不同而异。
- **ExpressRoute 高级版附加组件**。ExpressRoute 高级版是 ExpressRoute 线路上的附加组件。ExpressRoute 高级版附加组件提供以下功能： 
	- 提高 Azure 公共和 Azure 专用对等互连的路由限制，从4,000 路由提升至 10,000 路由。
	- 服务的跨地缘政治区域连接。在任何区域创建的 ExpressRoute 线路都将能够访问位于其他区域的资源。例如，创建于中国北部的虚拟网络可以通过在中国东部设置的 ExpressRoute 线路进行访问。
	- 增加了每个 ExpressRoute 线路的 VNet 链接数量，从 10 增加至更大的限制，具体取决于线路的带宽。

## 后续步骤

- 了解 [ExpressRoute 连接模型](/documentation/articles/expressroute-connectivity-models/)。
- 了解 ExpressRoute 连接和路由域。请参阅 [ExpressRoute 线路和路由域](/documentation/articles/expressroute-circuit-peerings/)。
- 查找服务提供商。请参阅 [ExpressRoute 合作伙伴和对等位置](/documentation/articles/expressroute-locations/)。
- 确保符合所有先决条件。请参阅 [ExpressRoute 先决条件](/documentation/articles/expressroute-prerequisites/)。
- 请参阅[路由](/documentation/articles/expressroute-routing/)的要求。
- 配置 ExpressRoute 连接。
	- [创建 ExpressRoute 线路](/documentation/articles/expressroute-howto-circuit-portal-resource-manager/)
	- [配置路由](/documentation/articles/expressroute-howto-routing-portal-resource-manager/)
	- [将 VNet 链接到 ExpressRoute 线路](/documentation/articles/expressroute-howto-linkvnet-portal-resource-manager/)

<!---HONumber=Mooncake_0320_2017-->