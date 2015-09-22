<properties
   pageTitle="ExpressRoute 常见问题"
   description="ExpressRoute 常见问题包含有关支持的 Azure 服务、费用、数据和连接、SLA、提供程序和位置、带宽的信息和其他技术详细信息。"
   documentationCenter="na"
   services="expressroute"
   authors="cherylmc"
   manager="adinah"
   editor="tysonn"/>
<tags
   ms.service="expressroute"
   ms.date="07/28/2015"
   wacn.date=""/>

# ExpressRoute 常见问题


## 什么是 ExpressRoute？
ExpressRoute 是一项 Azure 服务，允许你在 Microsoft 数据中心与你的本地环境或共同租用设施中的基础结构之间创建专用连接。ExpressRoute 连接不通过公共 Internet，与通过公共 Internet 的典型连接相比，提供更高的安全性、可靠性、速度和更低的延迟。

### 使用 ExpressRoute 和专用网络连接的好处是什么？
ExpressRoute 连接不通过公共 Internet，与通过公共 Internet 的典型连接相比，提供更高的安全性、可靠性、速度和一贯较低的延迟。在某些情况下，使用 ExpressRoute 连接在本地设备和 Azure 之间传输数据可以产生显著的成本效益。

### 可通过 ExpressRoute 支持哪些 Microsoft 云服务？
ExpressRoute 目前支持大多数 Microsoft Azure 服务。我们即将宣布可通过 ExpressRoute 支持 Office 365 服务。在正式版发布后，请查看最新信息。

### 哪里提供该服务？
参阅 [ExpressRoute 合作伙伴和位置](/documentation/articles/expressroute-locations)了解服务上市区域和可用性。

### 我如果未与 ExpressRoute 运营商合作伙伴之一建立合作伙伴关系，则如何使用 ExpressRoute 连接到 Microsoft？
你可以将区域运营商和地区以太网连接选择为支持的 Exchange 提供商位置之一。然后，你可以在 EXP 位置与 Microsoft 实现对接。查看 [ExpressRoute 合作伙伴和位置](/documentation/articles/expressroute-locations)的最后一部分，以确定你的网络提供商是否处在任何 Exchange 位置中。然后，你可以从交换提供商订购一条 ExpressRoute 线路以连接到 Azure。

### ExpressRoute 的费用是多少？
有关定价信息，请查看[定价详细信息](/home/features/expressroute/#price)。

### 如果我购买了具有给定带宽的 ExpressRoute 线路，我必须从网络服务提供商购买具有相同速度的 VPN 连接吗？
不是。你可以从服务提供商购买任何速度的 VPN 连接。但是，与 Azure 的连接速度将限制为你购买的 ExpressRoute 线路带宽。

### 如果我购买了具有给定带宽的 ExpressRoute 线路，是否可以根据需要提升到更高的速度？
是的。ExpressRoute 线路的配置支持免费将速度提升到所购带宽限制的两倍。请咨询你的服务提供商，以确定他们是否支持此功能。

### 能否同时与虚拟网络和其他 Azure 服务使用同一专用网络连接？
是的。设置 ExpressRoute 线路后，将允许你同时访问虚拟网络中的服务和其他 Azure 服务。你将通过专用对等路径连接到虚拟网络，通过公用对等路径连接到其他服务。

### ExpressRoute 是否提供服务级别协议 (SLA)？
有关详细信息，请参阅 [ExpressRoute SLA 页](/support/legal/sla/)。

## 支持的 Azure 服务
大多数 Azure 服务都通过 ExpressRoute 提供支持。

与虚拟机和虚拟网络中部署的云服务的连接通过专用对等路径提供支持。

可通过公共对等路径访问 Azure 网站。

可通过公共对等路径访问所有其他服务。下面列出了例外情况 -

**不支持以下项目：**

- CDN
- Visual Studio Online Load Testing
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

### 我是否需要配置这两种链路才能使服务正常工作？
如果你正在通过 NSP 进行连接，NSP 将负责为你配置冗余链接。如果你正在通过 EXP 进行连接，则必须配置这两条链路。如果没有为线路配置冗余，我们的 SLA 将无效。

### 能否使用 ExpressRoute 将我的一个 VLAN 扩展到 Azure？
否。我们不支持将第 2 层连接扩展到 Azure。

### 能否在我的订阅中有多条 ExpressRoute 线路？
是的。你可以在订阅中有多条 ExpressRoute 线路。专用线路数的默认限制设置为 10。如果你需要增大限制，请联系 Microsoft 支持。

### 能否使用不同服务提供商的 ExpressRoute 线路？
是的。你可以使用许多服务提供商的 ExpressRoute 线路。每条 ExpressRoute 线路将只与一个服务提供商相关联。

### 如何将我的虚拟网络连接到 ExpressRoute 线路
基本步骤如下所述。

- 你必须建立一条 ExpressRoute 线路并让服务提供商启用它。
- 你必须为私有对等互连配置 BGP（如果你使用的是 Exchange 提供商）。
- 你必须将虚拟网络连接到 ExpressRoute 线路。

以下教程将帮助你：

- [通过网络服务提供商配置 ExpressRoute 连接](/documentation/articles/expressroute-configuring-nsps)
- [通过 Exchange 提供商配置 ExpressRoute 连接](/documentation/articles/expressroute-configuring-exps)
- [为 ExpressRoute 配置虚拟网络和网关](/documentation/articles/expressroute-configuring-vnet-gateway)

### 我的 ExpressRoute 线路是否存在连接界限？
是的。[ExpressRoute 合作伙伴和位置](/documentation/articles/expressroute-locations)页概述了 ExpressRoute 线路的连接界限。一条 ExpressRoute 线路的连接范围限制为单个地缘政治区域。可以通过启用 ExpressRoute 高级功能，将连接扩展为跨地缘政治区域。

### 能否将多个虚拟网络链接到一条 ExpressRoute 线路？
是的。最多可以将 10 个虚拟网络链接到一条 ExpressRoute 线路。

### 我有多个包含虚拟网络的 Azure 订阅。能否将不同订阅中的虚拟网络连接到单个 ExpressRoute 线路？
是的。最多可以授权其他 10 个 Azure 订阅使用单条 ExpressRoute 线路。可以通过启用 ExpressRoute 高级功能来提高此限制。

有关详细信息，请参阅[在多个订阅之间共享 ExpressRoute 线路](/documentation/articles/expressroute-share-circuit)。

### 连接到同一线路的虚拟网络相互隔离吗？
不能。连接到同一 ExpressRoute 线路的所有虚拟网络都属于同一路由域，从路由角度看不是相互隔离的。如果需要路由隔离，则需要创建单独的 ExpressRoute 线路。

### 能否将一个虚拟网络连接到多条 ExpressRoute 线路？
是的。可以将一个虚拟网络最多链接到 4 条 ExpressRoute 线路。所有 ExpressRoute 线路必须位于同一个大洲。可以通过不同区域的不同服务提供商订购线路。

### 能否从连接到 ExpressRoute 线路的虚拟网络访问 Internet？
是的。如果你尚未通过 BGP 会话公布默认路由 (0.0.0.0/0) 或 Internet 路由前缀，你将能够从连接到 ExpressRoute 线路的虚拟网络连接到 Internet。

### 能否阻止与连接到 ExpressRoute 线路的虚拟网络建立 Internet 连接？
是的。你可以公布默认路由 (0.0.0.0/0) 以阻止与虚拟网络内部署的虚拟机建立所有 Internet 连接，并通过 ExpressRoute 线路路由出所有流量。请注意，如果你播发默认路由，我们会强制将传送到通过公共对等互连提供的服务（如 Azure 存储空间和 SQL DB）的流量，传回到你的本地。你必须将路由器配置为通过公共对等路径或通过 Internet 将流量传回到 Azure。

### 连接到同一 ExpressRoute 线路的虚拟网络能否互相对话？
是的。连接到同一 ExpressRoute 线路的虚拟网络中部署的虚拟机可以彼此通信。

### 能否将站点到站点连接与 ExpressRoute 一起用于虚拟网络？
是的。ExpressRoute 可与站点到站点 VPN 共存。

### 能否将虚拟网络从站点到站点/点到站点配置转为使用 ExpressRoute？
是的。必须在虚拟网络中创建 ExpressRoute 网关。该过程将会造成较短的停机时间。

### 通过 ExpressRoute 连接到 Azure 存储需要执行哪些操作？
你必须建立一条 ExpressRoute 线路并为公共对等互连配置路由。

### 对于我可以公布的路由数有限制吗？
是的。对于专用对等互连和公共对等互连，我们最多接受 4000 个路由前缀。如果启用 ExpressRoute 高级功能，可以将此限制提高为 10,000 个路由。

### 对于我可以通过 BGP 会话公布的 IP 范围有限制吗？
通过 BGP 公布的前缀必须为 /29 或更大（/28 到 /8）。

我们将在公共对等 BGP 会话中筛选出私有前缀 (RFC1918)。

### 如果超过 BGP 限制，会发生什么情况？
BGP 会话将被删除。当前缀计数低于限制后，将重置这些会话。

### 将默认路由 (0.0.0.0/0) 播发到虚拟网络后，我无法激活 Azure VM 上运行的 Windows。我如何解决此问题？
以下步骤可帮助 Azure 识别激活请求：

1. 为 ExpressRoute 线路建立公共对等互连。
2. 执行 DNS 查找，找到 **kms.core.windows.net** 的 IP 地址
3. 然后执行以下两项操作之一，使密钥管理服务能够识别来自 Azure 的激活请求并遵照该请求。
	- 在你的本地网络上，通过公共对等互连将发往 IP 地址（在步骤 2 中获得）的流量路由回到 Azure。
	- 让你的 NSP 提供商通过公共对等互连将流量路由回到 Azure。

### 是否可以更改 ExpressRoute 线路的带宽？
是的。你无需拆解 ExpressRoute 线路，就能提高它的带宽。必须跟进你的连接提供商，以确保他们更新其网络中的阈值，为提高的带宽提供支持。但是，你无法降低 ExpressRoute 线路的带宽。降低带宽意味着要拆解然后重新创建 ExpressRoute 线路。

### 如何更改 ExpressRoute 线路的带宽？
你可以使用“更新专用线路 API”和 PowerShell cmdlet 来更新 ExpressRoute 线路的带宽。

## ExpressRoute 高级版

### 什么是 ExpressRoute 高级版？
ExpressRoute 高级版包括下面列出的功能集合。

 - 对于公共对等互连和专用对等互连，将路由表限制从 4000 个路由提升为 10,000 个路由。
 - 增加了可连接到 ExpressRoute 线路的 VNet 数量（默认数量为 10 个）。有关详细信息，请参阅下表。
 - 通过 Microsoft 核心网络建立全局连接。现在，你可以将一个地缘政治区域中 VNet 链接到另一个区域中的 ExpressRoute 线路。**示例：**可以将欧洲西部创建的 VNet 链接到硅谷创建的 ExpressRoute 线路。

### 如果启用 ExpressRoute 高级版，可将多少个 VNet 链接到一条 ExpressRoute 线路？
下表列出了链接到 ExpressRoute 线路的 VNet 数的更高限制。默认限制为 10。

**针对通过 NSP 创建的线路的限制**

| **线路大小** | **针对默认安装的 VNet 链接数** | **使用 ExpressRoute 高级版时的 VNet 链接数** |
|--------------|----------------------------------------|-----------------------------------------------|
| 10 Mbps | 10 | 不支持 |
| 50 Mbps | 10 | 20 |
| 100 Mbps | 10 | 25 |
| 500 Mbps | 10 | 40 |
| 1Gbps | 10 | 50
|


**针对通过 EXP 创建的线路的限制**

| **线路大小** | **针对默认安装的 VNet 链接数** | **使用 ExpressRoute 高级版时的 VNet 链接数** |
|--------------|-----------------------------------|------------------------------------------------|
| 200 Mbps | 10 | 25 |
| 500 Mbps | 10 | 40 |
| 1 Gbps | 10 | 50 |
| 10 Gbps | 10 | 100 |



### 如何启用 ExpressRoute 高级版？
在启用相应的功能后，将启用 ExpressRoute 高级功能；可以通过更新线路状态关闭高级功能。可以在创建线路时启用 ExpressRoute 高级版，或者通过调用“更新专用线路 API”/PowerShell cmdlet 来启用 ExpressRoute 高级版。

### 如何禁用 ExpressRoute 高级版？
你可以通过调用“更新专用线路 API”/PowerShell cmdlet 来禁用 ExpressRoute 高级版。在禁用 ExpressRoute 高级版之前，必须确保调整连接需求以满足默认限制。如果你的利用率级别超出了默认限制，我们将拒绝 ExpressRoute 禁用请求。

### 我是否可以从高级功能集选择所需的功能？
不可以。你无法选择所需的功能。如果你启用 ExpressRoute 高级版，我们会启用所有功能。

### ExpressRoute 高级版的费用是多少？
有关费用，请参阅[定价详细信息](/home/features/expressroute/#price)。

### 除了支付 ExpressRoute 高级版费用以外，是否还要支付标准版 ExpressRoute 的费用？
是的。ExpressRoute 高级版的费用是在 ExpressRoute 线路费用以及连接提供商所收费用的基础之上收取的。

### ExpressRoute 高级版是否同时支持 EXP 和 NSP 模式？
是的。ExpressRoute 高级版支持通过 EXP 和 NSP 连接的 ExpressRoute 线路。


## ExpressRoute 和 Office 365

### 如何创建一条 ExpressRoute 线路来连接 Office 365 服务？

1. 请查看 [ExpressRoute 先决条件页](/documentation/articles/expressroute-prerequisites)，以确保满足要求
2. 请查看 [ExpressRoute 合作伙伴和位置](expressroute-locations)中的服务提供商和位置列表，以确保满足你的连接需求。
3. 请查看[针对 Office 365 的网络规划和性能优化](http://aka.ms/tune/)，以规划你的容量要求
4. 遵照以下工作流中列出的步骤来设置连接。

	- [通过网络服务提供商配置 ExpressRoute 连接](/documentation/articles/expressroute-configuring-nsps)
	- [通过 Exchange 提供商配置 ExpressRoute 连接](/documentation/articles/expressroute-configuring-exps)

### 我的现有 ExpressRoute 线路是否支持连接到 Office 365 服务？
是的。可以将你的现有 ExpressRoute 线路配置为支持连接到 Office 365 服务。请确保你有足够的容量连接到 Office 365 服务。[针对 Office 365 的网络规划和性能优化](http://aka.ms/tune/)中的内容可帮助你规划连接需求。

以下教程将帮助你：

- [通过网络服务提供商配置 ExpressRoute 连接](/documentation/articles/expressroute-configuring-nsps)
- [通过 Exchange 提供商配置 ExpressRoute 连接](/documentation/articles/expressroute-configuring-exps)

### 通过 ExpressRoute 连接可以访问哪些 Office 365 服务？

**支持以下 Office 365 服务**

- Exchange Online 和 Exchange Online Protection
- SharePoint Online
- Skype for Business Online
- Office Online
- Azure AD 和 Azure AD Sync
- Office 365 Video
- Power BI
- Delve
- Project Online

**不支持以下 Office 365 服务**

- Yammer
- Office 365 ProPlus 客户端下载
- 本地标识提供程序登录
- 中国的 Office 365（由 21 Vianet 运营）服务

可以通过 Internet 连接到这些服务。

### 适用于 Office 365 的 ExpressRoute 费用是多少？
通过 ExpressRoute 连接到 Office 365 不会产生额外的收费。[定价详细信息页](/home/features/expressroute/#price)提供了有关 ExpressRoute 费用的详细信息。

### 哪些区域支持适用于 Office 365 的 ExpressRoute？
有关支持 ExpressRoute 的合作伙伴和位置列表，请参阅 [ExpressRoute 合作伙伴和位置](/documentation/articles/expressroute-locations)。

### 是否可以使用 NSP 和 EXP 连接到 Office 365 服务？
我们支持通过 NSP 和 EXP 连接到 Office 365 服务。有关支持的合作伙伴和位置列表，请参阅 [ExpressRoute 合作伙伴和位置](/documentation/articles/expressroute-locations)。

### 是否即使为组织配置了 ExpressRoute，也可以通过 Internet 访问 Office 365？
是的。即使为你的网络配置了 ExpressRoute，也可以通过 Internet 访问 Office 365 服务终结点。如果你所在的位置已配置为通过 ExpressRoute 连接到 Office 365 服务，则你将通过 ExpressRoute 进行连接。
 

<!---HONumber=69-->