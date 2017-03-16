<properties
   pageTitle="ExpressRoute 常见问题"
   description="ExpressRoute 常见问题包含有关支持的 Azure 服务、费用、数据和连接、SLA、提供商和位置、带宽的信息和其他技术详细信息。"
   documentationCenter="na"
   services="expressroute"
   authors="cherylmc"
   manager="carmonm"
   editor=""/>
<tags
   ms.service="expressroute"
   ms.devlang="na"
   ms.topic="article" 
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="11/21/2016"
   wacn.date="01/03/2017"
   ms.author="cherylmc"/>

# ExpressRoute 常见问题


## 什么是 ExpressRoute？
ExpressRoute 是一项 Azure 服务，允许在 Azure 数据中心与本地环境或第三方托管设施中的基础结构之间创建专用连接。ExpressRoute 连接不通过公共 Internet，与通过公共 Internet 的典型连接相比，可提供更高的安全性、可靠性、速度以及更低的延迟。

### 使用 ExpressRoute 和专用网络连接有什么好处？
ExpressRoute 连接不通过公共 Internet，与通过公共 Internet 的典型连接相比，可提供更高的安全性、可靠性、速度以及较低的一致延迟。在某些情况下，使用 ExpressRoute 连接在本地设备和 Azure 之间传输数据可以产生显著的成本效益。

### 可通过 ExpressRoute 支持哪些 Azure 服务？
ExpressRoute 目前支持大多数 Azure 服务。

### 哪里提供该服务？
参阅 [ExpressRoute 合作伙伴和位置](/documentation/articles/expressroute-locations/)了解服务上市区域和可用性。

### 如果未与 ExpressRoute 运营商合作伙伴之一建立合作伙伴关系，则如何使用 ExpressRoute 连接到 Azure？
可以将区域运营商和地区以太网连接选择为支持的 Exchange 提供商位置之一。然后，可以在提供商位置与 Azure 实现对接。请查看 [ExpressRoute 合作伙伴和位置](/documentation/articles/expressroute-locations/)的最后一部分，以确定服务提供商是否在任何 Exchange 位置。然后，可以通过服务提供商订购 ExpressRoute 线路以连接到 Azure。

### ExpressRoute 需要多少费用？
有关定价信息，请查看[定价详细信息](/pricing/details/expressroute/)。

### 如果购买具有给定带宽的 ExpressRoute 线路，是否必须从网络服务提供商购买具有相同速度的 VPN 连接？
否。可以从服务提供商购买任何速度的 VPN 连接。但是，与 Azure 的连接速度会限制购买的 ExpressRoute 线路带宽。

### 如果购买具有给定带宽的 ExpressRoute 线路，是否可以根据需要提升到更高的速度？
可以。ExpressRoute 线路的配置支持免费将速度提升到购买带宽限制的两倍。请咨询服务提供商，以确定他们是否支持此功能。

### 是否可以同时与虚拟网络和其他 Azure 服务使用同一专用网络连接？
可以。设置 ExpressRoute 线路后，允许同时访问虚拟网络中的服务和其他 Azure 服务。将通过专用对等路径连接到虚拟网络，通过公用对等路径连接到其他服务。

### ExpressRoute 是否提供服务级别协议 (SLA)？
有关详细信息，请参阅 [ExpressRoute SLA 页](/support/legal/sla/)。

## <a name="supported-services"></a> 支持的服务
ExpressRoute 支持[两种路由域](/documentation/articles/expressroute-circuit-peerings/)，适用于各种类型的服务。

专用对等互连
 - 虚拟网络，包括所有虚拟机和云服务

公共对等互连
 - 大多数 Azure 服务（有少数几个例外，例外的服务将在下面介绍）
 - Power BI
 - Dynamics 365 for Operations（以前称为 Dynamics AX Online）


ExpressRoute 不支持以下 Azure 服务
 - CDN
 - Visual Studio Team Services 负载测试
 - 多重身份验证
 - 流量管理器

## 数据和连接

### 对于使用 ExpressRoute 可以传输的数据量是否有限制？
我们不对数据传输量设置限制。有关带宽价格的信息，请参阅[定价详细信息](/pricing/details/expressroute/)。

### ExpressRoute 支持的连接速度是多少？
支持的带宽如下：

|50 Mbps, 100 Mbps, 200 Mbps, 500 Mbps, 1Gbps, 2 Gbps, 5 Gbps, 10Gbps|

### 可以选择哪些服务提供商？
有关服务提供商和位置的列表，请参阅 [ExpressRoute 合作伙伴和位置](/documentation/articles/expressroute-locations/)。

## 技术详细信息

### 将本地位置连接到 Azure 有哪些技术要求？
有关要求，请参阅 [ExpressRoute 先决条件页](/documentation/articles/expressroute-prerequisites/)。

### 与 ExpressRoute 的连接是否冗余？
是。每条 ExpressRoute 线路都配置一对冗余交叉连接，以提供高可用性。

### 如果某个 ExpressRoute 链路出现故障，会失去连接？
如果其中一个交叉连接出现故障，不会失去连接。冗余连接可用于支持网络负载。另外，还可以在不同对等位置创建多条线路以实现故障恢复。

### <a name="onep2plink"></a>如果我不在云交换中共置，而我的服务提供商提供点到点连接，我需要在本地网络与 Azure 之间订购两个物理连接吗？ 
不需要，如果服务提供商可以通过物理连接建立两条以太网虚拟线路，则只需要一个物理连接。物理连接（例如光纤）的终点在第 1 层 (L1) 设备上（参阅下图）。两条以太网虚拟线路使用不同的 VLAN ID 进行标记，一个供主线路使用，一个供辅助线路使用。这些 VLAN ID 位于外部 802.1Q 以太网标头中。内部 802.1Q 以太网标头（不显示）会映射到特定的 [ExpressRoute 路由域](/documentation/articles/expressroute-circuit-peerings/)。

![](./media/expressroute-faqs/expressroute-p2p-ref-arch.png)


### 是否可以使用 ExpressRoute 将一个 VLAN 扩展到 Azure？
不可以。不支持将第 2 层连接扩展到 Azure。

### 是否可以在订阅中有多条 ExpressRoute 线路？
可以。可以在订阅中有多条 ExpressRoute 线路。专用线路数的默认限制设置为 10。如果需要增大限制，请联系 Azure 支持。

### 是否可以使用不同服务提供商的 ExpressRoute 线路？
可以。可以使用多个服务提供商的 ExpressRoute 线路。每条 ExpressRoute 线路将只与一个服务提供商相关联。

### 如何将虚拟网络连接到 ExpressRoute 线路
基本步骤如下所述：

- 必须建立一条 ExpressRoute 线路并让服务提供商启用它。
- 用户或提供商必须配置 BGP 对等互连。
- 必须将虚拟网络连接到 ExpressRoute 线路。

有关详细信息，请参阅 [ExpressRoute 线路预配工作流和线路状态](/documentation/articles/expressroute-workflows/)。

### ExpressRoute 线路是否存在连接界限？
是。[ExpressRoute 合作伙伴和位置](/documentation/articles/expressroute-locations/)页概述了 ExpressRoute 线路的连接界限。ExpressRoute 线路的连接范围限制为单个地缘政治区域。可以通过启用 ExpressRoute 高级功能，将连接扩展为跨地缘政治区域。

### 是否可以将多个虚拟网络链接到 ExpressRoute 线路？
可以。最多可以将 10 个虚拟网络链接到 ExpressRoute 线路。

### 拥有包含虚拟网络的多个 Azure 订阅。是否可以将不同订阅中的虚拟网络连接到单个 ExpressRoute 线路？
可以。最多可以授权其他 10 个 Azure 订阅使用单个 ExpressRoute 线路。可以通过启用 ExpressRoute 高级功能增加此限制。

有关详细信息，请参阅[在多个订阅之间共享 ExpressRoute 线路](/documentation/articles/expressroute-howto-linkvnet-arm/)。

### 连接到同一线路的虚拟网络是否相互隔离？
否。连接到同一 ExpressRoute 线路的所有虚拟网络都属于同一路由域，从路由角度看不相互隔离。如果需要路由隔离，则需要创建单独的 ExpressRoute 线路。

### 是否可以将一个虚拟网络连接到多条 ExpressRoute 线路？
可以。可以将一个虚拟网络最多链接到 4 条 ExpressRoute 线路。必须通过 4 个不同的 [ExpressRoute 位置](/documentation/articles/expressroute-locations/)订购这些线路。

### 是否可以从连接到 ExpressRoute 线路的虚拟网络访问 Internet？
可以。如果尚未通过 BGP 会话公布默认路由 (0.0.0.0/0) 或 Internet 路由前缀，将能够从连接到 ExpressRoute 线路的虚拟网络连接到 Internet。

### 是否可以阻止与连接到 ExpressRoute 线路的虚拟网络建立 Internet 连接？
可以。可以公布默认路由 (0.0.0.0/0) 以阻止与部署在虚拟网络中的虚拟机建立所有 Internet 连接，并通过 ExpressRoute 线路路由所有流量。请注意，如果播发默认路由，会强制将传送到通过公共对等互连提供的服务（如 Azure 存储空间和 SQL DB）的流量传回到本地。必须将路由器配置为通过公共对等路径或通过 Internet 将流量传回到 Azure。

### 连接到同一 ExpressRoute 线路的虚拟网络是否可以相互通信？
可以。连接到同一 ExpressRoute 线路的虚拟网络中部署的虚拟机可以彼此通信。

### 是否可以将站点到站点连接与 ExpressRoute 一起用于虚拟网络？
可以。ExpressRoute 可以与站点到站点 VPN 共存。

### 是否可以将虚拟网络从站点到站点/点到站点配置转为使用 ExpressRoute？
可以。必须在虚拟网络中创建 ExpressRoute 网关。该过程将会造成较短的停机时间。

### 通过 ExpressRoute 连接到 Azure 存储需要执行哪些操作？
必须建立 ExpressRoute 线路并为公共对等互连配置路由。

### 对可以播发的路由数是否有限制？
是的。对于专用对等互连，最多接受 4000 个路由前缀；对于公共对等互连，接受 200 个。如果启用 ExpressRoute 高级功能，可以将专用对等互连的此限制提高为 10,000 个路由。

### 对可以通过 BGP 会话播发的 IP 地址范围是否有限制？

在公共对等 BGP 会话中，不接受私有前缀 (RFC1918)。

### 如果超过 BGP 限制，会发生什么情况？
会将 BGP 会话删除。当前缀计数低于限制后，将重置这些会话。

### ExpressRoute BGP 保持时间是多少？ 是否可以进行调整？
保持时间为 180。Keep-Alive（保持活动）消息每隔 60 秒发送一次。这些是 Microsoft 端的固定设置，无法更改。

### 将默认路由 (0.0.0.0/0) 播发到虚拟网络后，无法激活 Azure VM 上运行的 Windows。如何解决此问题？
以下步骤可帮助 Azure 识别激活请求：

1. 为 ExpressRoute 线路建立公共对等互连。
2. 执行 DNS 查找，找到 **kms.core.chinacloudapi.cn** 的 IP 地址
3. 然后执行以下两项操作之一，使密钥管理服务能够识别来自 Azure 的激活请求并遵循该请求。
	- 在本地网络上，通过公共对等互连将发往 IP 地址（在步骤 2 中获得）的流量路由回到 Azure。
	- 让 NSP 提供商通过公共对等互连将流量路由回到 Azure。

### 是否可以更改 ExpressRoute 线路的带宽？
可以。可以增加 ExpressRoute 线路的带宽，而无需将其拆解。必须跟进连接服务提供商，确保他们更新其网络中的限制以支持增加带宽。但是，无法降低 ExpressRoute 线路的带宽。降低带宽意味着要拆解，然后重新创建 ExpressRoute 线路。

### 如何更改 ExpressRoute 线路的带宽？
可以使用“更新专用线路 API”和 PowerShell cmdlet 更新 ExpressRoute 线路的带宽。

## ExpressRoute 高级版

### 什么是 ExpressRoute 高级版？
ExpressRoute 高级版是下面列出的功能的集合。

 - 对于专用对等互连，将路由表限制从 4000 个路由提升为 10,000 个路由。
 - 已增加可连接到 ExpressRoute 线路的 VNet 数（默认数为 10 个）。有关详细信息，请参阅下表。
 - 通过 Azure 核心网络建立全局连接。现在，可以将一个地缘政治区域中的 VNet 链接到另一个区域中的 ExpressRoute 线路。**示例：**可以将中国北部创建的 VNet 链接到中国东部创建的 ExpressRoute 线路。

### 如果启用 ExpressRoute 高级版，可将多少个 VNet 链接到 ExpressRoute 线路？
下表显示了 ExpressRoute 限制和每条 ExpressRoute 线路的 VNet 数。


[AZURE.INCLUDE [expressroute-limits](../../includes/expressroute-limits.md)]


### 如何启用 ExpressRoute 高级版？
启用相应功能后可启用 ExpressRoute 高级功能，并且可通过更新线路状态将其关闭。可以在创建线路时启用 ExpressRoute 高级版，或者通过调用“更新专用线路 API”/PowerShell cmdlet 启用 ExpressRoute 高级版。

### 如何禁用 ExpressRoute 高级版？
可以通过调用“更新专用线路 API”/PowerShell cmdlet 禁用 ExpressRoute 高级版。在禁用 ExpressRoute 高级版前，必须确保调整连接需求以满足默认限制。如果利用率级别超出默认限制，将拒绝 ExpressRoute 禁用请求。

### 是否可以从高级功能集选择所需的功能？
不可以。无法选择所需的功能。如果启用 ExpressRoute 高级版，将会启用所有功能。

### ExpressRoute 高级版需要多少费用？
有关费用，请参阅[定价详细信息](/pricing/details/expressroute/)。

### 除了支付 ExpressRoute 高级版费用以外，是否还要支付标准版 ExpressRoute 的费用？
是。ExpressRoute 高级版费用是在 ExpressRoute 线路费用和连接服务提供商所收费用的基础之上收取。


<!---HONumber=Mooncake_1226_2016-->