<properties
   pageTitle="ExpressRoute 常见问题"
   description="ExpressRoute 常见问题包含有关支持的 Azure 服务、费用、数据和连接、SLA、提供程序和位置、带宽的信息和其他技术详细信息。"
   documentationCenter="na"
   services="expressroute"
   authors="cherylmc"
   manager="carolz"
   editor=""/>
<tags
   ms.service="expressroute"
   ms.date="02/09/2016"
   wacn.date="03/17/2016"/>

# ExpressRoute 常见问题


## 什么是 ExpressRoute？
ExpressRoute 是一项 Azure 服务，允许你在 Microsoft Azure 数据中心与你的本地环境或第三方托管设施中的基础结构之间创建专用连接。ExpressRoute 连接不通过公共 Internet，与通过公共 Internet 的典型连接相比，提供更高的安全性、可靠性、速度和更低的延迟。

### 使用 ExpressRoute 和专用网络连接的好处是什么？
ExpressRoute 连接不通过公共 Internet，与通过公共 Internet 的典型连接相比，提供更高的安全性、可靠性、速度和一贯较低的延迟。在某些情况下，使用 ExpressRoute 连接在本地设备和 Azure 之间传输数据可以产生显著的成本效益。

### 可通过 ExpressRoute 支持哪些 Microsoft Azure 服务？
ExpressRoute 目前支持大多数 Azure 服务。

### 哪里提供该服务？
参阅 [ExpressRoute 合作伙伴和位置](/documentation/articles/expressroute-locations)了解服务上市区域和可用性。

### 我如果未与 ExpressRoute 运营商合作伙伴之一建立合作伙伴关系，则如何使用 ExpressRoute 连接到 Microsoft Azure？
你可以通过区域运营商来建立以太网连接到 Microsoft Azure支持的连接提供商。然后，你可以在连接服务商的位置与 Microsoft Azure 实现对接。查看 [ExpressRoute 合作伙伴和位置](/documentation/articles/expressroute-locations)的最后一部分，以确定你的网络提供商是否处在任何 Exchange 位置中。然后，你可以从交换提供商订购一条 ExpressRoute 线路以连接到 Azure。

### ExpressRoute 的费用是多少？
有关定价信息，请查看[定价详细信息](/home/features/expressroute/#price)。

### 如果我购买了具有给定带宽的 ExpressRoute 线路，我必须从网络服务提供商购买具有相同速度的 VPN 连接吗？
不是。你可以从服务提供商购买任何速度的 VPN 连接。但是，与 Azure 的连接速度将限制为你购买的 ExpressRoute 线路带宽。

### 如果我购买了具有给定带宽的 ExpressRoute 线路，是否可以根据需要提升到更高的速度？
是的。ExpressRoute 线路的配置支持将速度提升。请咨询你的服务提供商，以确定他们是否支持带宽升速。

### 能否同时与虚拟网络和其他 Azure 服务使用同一专用网络连接？
是的。设置 ExpressRoute 线路后，将允许你同时访问虚拟网络中的服务和其他 Azure 服务。你将通过专用对等路径连接到虚拟网络，通过公用对等路径连接到其他服务。

### ExpressRoute 是否提供服务级别协议 (SLA)？
有关详细信息，请参阅 [ExpressRoute SLA 页](/support/legal/sla/)。

## 支持的服务
大多数 Azure 服务都通过 ExpressRoute 提供支持。

- 与虚拟机和虚拟网络中部署的云服务的连接通过私有对等互联路径提供支持。

- 可通过公共对等路径访问 Azure 网站。

- 可通过公共对等路径访问所有其他服务。下面列出了例外情况 -

**不支持以下服务：**

- CDN
- Visual Studio Team Services 负载测试
- 多重身份验证

## 数据和连接

### 对于使用 ExpressRoute 可以传输的数据量是否有限制？
我们不对数据传输量设置限制。有关带宽价格的信息，请参阅[定价详细信息](/home/features/expressroute/#price)。

### ExpressRoute 支持的连接速度是多少？
支持带宽提供：

|**提供商**|**带宽**|
|---|---|
|**网络提供商**|10 Mbps、50 Mbps、100 Mbps、500 Mbps、1 Gbps|
|**Exchange 提供商**|200 Mbps、500 Mbps、1Gbps、10Gbps|

### 可以选择哪些服务提供商？
有关服务提供商和位置的列表，请参阅 [ExpressRoute 合作伙伴和位置](/documentation/articles/expressroute-locations)。

## 技术详细信息

### 将本地位置连接到 Azure 有哪些技术要求？
有关要求，请参阅 [ExpressRoute 先决条件页](/documentation/articles/expressroute-prerequisites)。

### 与 ExpressRoute 的连接是冗余的吗？
是的。每条 ExpressRoute 线路都配置了一对冗余的交叉连接，以便为你提供高可用性。

### 如果我的某个 ExpressRoute 链路出现故障，我会失去连接吗？
如果其中一个交叉连接出现故障，你不会失去连接。冗余连接可用于支持网络负载。另外，你还可以在不同对等位置创建多条线路以获得故障恢复能力。

### 我是否需要配置两条链路才能使服务正常工作？
是的，你必须为ExpressRoute配置两条 LAN 链路。如果你从专用数据中心通过单条 WAN 链路连接到 ExpressRoute 连接点，则需要连接服务商在ExpressRoute连接点终止你自己的路由器上的 WAN 链路，然后为 ExpressRoute 配置两条 LAN 链路。如果没有为线路配置冗余，我们的 SLA 将无效。

### 能否使用 ExpressRoute 将我的一个 VLAN 扩展到 Azure？
否。我们不支持将第 2 层连接扩展到 Azure。

### 能否在我的订阅中有多条 ExpressRoute 线路？
是的。你可以在订阅中有多条 ExpressRoute 线路。专用线路数的默认限制设置为 10。如果你需要增大限制，请联系 Microsoft Azure 支持。

### 能否使用不同服务提供商的 ExpressRoute 线路？
是的。你可以使用许多服务提供商的 ExpressRoute 线路。每条 ExpressRoute 线路将只与一个服务提供商相关联。

### 如何将我的虚拟网络连接到 ExpressRoute 线路？
基本步骤如下所述：

- 你必须建立一条 ExpressRoute 线路并让服务提供商启用它。
- 你必须为私有对等互连配置 BGP（如果你使用的是连接提供商）。
- 你必须将虚拟网络连接到 ExpressRoute 线路。

有关详细信息，请参阅 [ExpressRoute 线路预配工作流和线路状态](/documentation/articles/expressroute-workflows)。

### 我的 ExpressRoute 线路是否存在连接界限？
是的。[ExpressRoute 合作伙伴和位置](/documentation/articles/expressroute-locations)页概述了 ExpressRoute 线路的连接界限。一条 ExpressRoute 线路的连接范围限制为单个地缘政治区域。

### 能否将多个虚拟网络链接到一条 ExpressRoute 线路？
是的。最多可以将 10 个虚拟网络链接到一条 ExpressRoute 线路。

### 我有多个包含虚拟网络的 Azure 订阅。能否将不同订阅中的虚拟网络连接到单个 ExpressRoute 线路？
是的。最多可以授权其他 10 个 Azure 订阅使用单条 ExpressRoute 线路。

有关详细信息，请参阅[在多个订阅之间共享 ExpressRoute 线路](/documentation/articles/expressroute-howto-linkvnet-arm)。

### 连接到同一线路的虚拟网络相互隔离吗？
不能。连接到同一 ExpressRoute 线路的所有虚拟网络都属于同一路由域，从路由角度看不是相互隔离的。如果需要路由隔离，则需要创建单独的 ExpressRoute 线路。

### 能否将一个虚拟网络连接到多条 ExpressRoute 线路？
是的。可以将一个虚拟网络最多链接到 4 条 ExpressRoute 线路。必须通过 4 个不同的位置订购这些线路。

### 能否从连接到 ExpressRoute 线路的虚拟网络访问 Internet？
是的。如果你尚未通过 BGP 会话公布默认路由 (0.0.0.0/0) 或 Internet 路由前缀，你将能够从连接到 ExpressRoute 线路的虚拟网络连接到 Internet。

### 能否阻止与连接到 ExpressRoute 线路的虚拟网络建立 Internet 连接？
是的。你可以发布默认路由 (0.0.0.0/0) 以阻止与虚拟网络内部署的虚拟机建立所有 Internet 连接，并通过 ExpressRoute 线路路由出所有流量。请注意，如果你播发默认路由，我们会强制将传送到通过公共对等互连提供的服务（如 Azure 存储空间和 SQL DB）的流量，传回到你的本地。你必须将路由器配置为通过公共对等路径或通过 Internet 将流量传回到 Azure。

### 连接到同一 ExpressRoute 线路的虚拟网络能否互相对话？
是的。连接到同一 ExpressRoute 线路的虚拟网络中部署的虚拟机可以彼此通信。

### 能否将站点到站点的VPN连接与 ExpressRoute 一起用于虚拟网络？
是的。ExpressRoute 可与站点到站点 VPN 共存。

### 能否将虚拟网络从站点到站点/点到站点VPN配置转为使用 ExpressRoute？
是的。必须在虚拟网络中创建 ExpressRoute 网关。该过程将会造成较短的停机时间。

### 通过 ExpressRoute 连接到 Azure 存储需要执行哪些操作？
你必须建立一条 ExpressRoute 线路并为公共对等互连配置路由。

### 对于我可以公布的路由数有限制吗？
是的。对于专用对等互连和公共对等互连，我们最多接受 4000 个路由前缀。

### 对于我可以通过 BGP 会话公布的 IP 范围有限制吗？
通过 BGP 公布的前缀必须为 /29 或更大（/28 到 /8）。

我们将在公共对等 BGP 会话中筛选出私有前缀 (RFC1918)。

### 如果超过 BGP 限制，会发生什么情况？
BGP 会话将被删除。当前缀计数低于限制后，将重置这些会话。

### ExpressRoute BGP 保持时间是多少？ 是否可以调整它？
保持时间为 180 秒。Keep-Alive（保持活动）消息每隔 60 秒发送一次。这是 Microsoft 端的固定设置，不能更改。

### 将默认路由 (0.0.0.0/0) 播发到虚拟网络后，我无法激活 Azure VM 上运行的 Windows。我如何解决此问题？
以下步骤可帮助 Azure 识别激活请求：

1. 为 ExpressRoute 线路建立公共对等互连。
2. 执行 DNS 查找，找到 **kms.core.chinacloudapi.cn** 的 IP 地址
3. 然后执行以下两项操作之一，使密钥管理服务能够识别来自 Azure 的激活请求并遵照该请求。
	- 在你的本地网络上，通过公共对等互连将发往 IP 地址（在步骤 2 中获得）的流量路由回到 Azure。

### 是否可以更改 ExpressRoute 线路的带宽？
是的。你无需拆解 ExpressRoute 线路，就能提高它的带宽。必须跟进你的连接提供商，以确保他们更新其网络中的阈值，为提高的带宽提供支持。但是，你无法降低 ExpressRoute 线路的带宽。降低带宽意味着要拆解然后重新创建 ExpressRoute 线路。

### 如何更改 ExpressRoute 线路的带宽？
你可以使用“更新专用线路 API”和 PowerShell cmdlet 来更新 ExpressRoute 线路的带宽。



<!---HONumber=Mooncake_0104_2016-->