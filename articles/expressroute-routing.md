<properties
   pageTitle="ExpressRoute 路由要求 | Azure"
   description="本页提供有关为 ExpressRoute 线路配置和管理路由的详细要求。"
   documentationCenter="na"
   services="expressroute"
   authors="cherylmc"
   manager="carmonm"
   editor=""/>
<tags
   ms.service="expressroute"
   ms.date="05/16/2016"
   wacn.date="06/06/2016"/>

# ExpressRoute 路由要求  

若要使用 ExpressRoute 连接到 Azure 云服务，你需要设置并管理路由。某些连接服务提供商以托管服务形式提供路由的设置和管理。请咨询连接服务提供商，以确定他们是否提供此类服务。如果不提供，则你必须遵守以下所述的要求。

有关需要在设置后才能建立连接的路由会话的说明，请参阅[线路和路由域](/documentation/articles/expressroute-circuit-peerings/)一文。

**注意：** ExpressRoute 不支持任何高可用性配置的路由器冗余协议（如 HSRP 和 VRRP）。我们依赖每个对等互连的一组冗余 BGP 会话来获得高可用性。

## 对等互连的 IP 地址

你需要保留一些 IP 地址用于配置网络与 Microsoft Enterprise Edge (MSEE) 路由器之间的路由。本部分提供了要求列表，并介绍有关如何获取和使用这些 IP 地址的规则。

### Azure 专用对等互连的 IP 地址

你可以使用专用 IP 地址或公共 IP 地址来配置对等互连。用于配置路由的地址范围不得与用于在 Azure 中创建虚拟网络的地址范围重叠。

 - 必须为路由接口保留一个 /29 子网或两个 /30 子网。
 - 用于路由的子网可以是专用 IP 地址或公共 IP 地址。
 - 子网不得与客户保留用于 Microsoft 云的范围冲突。
 - 如果使用 /29 子网，它将拆分成两个 /30 子网。 
	 - 第一个 /30 子网用于主链路，第二个 /30 子网用于辅助链路。
	 - 对于每个 /30 子网，必须在路由器上使用 /30 子网的第一个 IP 地址。Microsoft 使用 /30 子网的第二个 IP 地址来设置 BGP 会话。
	 - 只有设置了两个 BGP 会话，我们的[可用性 SLA](https://azure.microsoft.com/support/legal/sla/) 才有效。  

#### 专用对等互连示例

如果你选择使用 a.b.c.d/29 来设置对等互连，它将拆分成两个 /30 子网。在以下示例中，我们可以了解 a.b.c.d/29 子网的用法。

a.b.c.d/29 拆分成 a.b.c.d/30 和 a.b.c.d+4/30 并通过预配 API 一路传递到 Azure。你将使用 a.b.c.d+1 作为主要 PE 的 VRF IP，而 Azure 将使用 a.b.c.d+2 作为主要 MSEE 的 VRF IP。你将使用 b.c.d+5 作为辅助 PE 的 VRF IP，而 Azure 将使用 a.b.c.d+6 作为辅助 MSEE 的 VRF IP。

假设你选择 192.168.100.128/29 来设置专用对等互连。192.168.100.128/29 包括从 192.168.100.128 到 192.168.100.135 的地址，其中：

- 192\.168.100.128/30 将分配给 link1（提供商使用 192.168.100.129，而 Azure 使用 192.168.100.130）。
- 192\.168.100.132/30 将分配给 link2（提供商使用 192.168.100.133，而 Azure 使用 192.168.100.134）。

## 动态路由交换

路由交换将通过 eBGP 协议进行。EBGP 会话在 MSEE 与路由器之间建立。不要求对 BGP 会话进行身份验证。如果需要，可以配置 MD5 哈希。有关配置 BGP 会话的信息，请参阅[配置路由](/documentation/articles/expressroute-howto-routing-classic/)及[线路预配工作流和线路状态](/documentation/articles/expressroute-workflows/)。

## 自治系统编号

Azure 使用 AS 12076 进行 Azure 公共和Azure 专用。我们保留了 AS 65515 供内部使用。支持 16 和 32 位 AS 编号。

数据传输对称没有相关要求。转发与返回路径可以遍历不同的路由器对。相同的路由必须在你拥有的多个线路对上，从任何一端播发。路由指标不需要完全相同。

## 路由聚合与前缀限制

我们支持通过 Azure 专用对等互连向我们播发最多 4000 个前缀。如果已启用 ExpressRoute 高级版附加组件，则可增加到 10,000 个前缀。我们接受为每个 BGP 会话最多使用 200 个前缀建立 Azure 公共互连。

如果前缀数目超过此限制，将丢弃 BGP 会话。我们只接受专用对等互连链路上的默认路由。

## 传输路由和跨区域路由

ExpressRoute 不能配置为传输路由器。你必须依赖连接服务提供商的传输路由服务。

## 播发默认路由

只有 Azure 专用对等互连会话允许默认路由。在这种情况下，我们将所有流量从关联的虚拟网络路由到你的网络。在专用对等互连中播发默认路由会导致来自 Azure 的 Internet 路径遭到阻止。你必须依赖企业网络边缘，为 Azure 中托管的服务往返路由 Internet 的流量。

 若要连接其他 Azure 服务和基础结构服务，你必须确保已准备好下列其中一项：

 - 已启用 Azure 公共对等互连，以将流量路由到公共终结点
 - 已使用用户定义的路由，为需要 Internet 连接的每个子网建立 Internet 连接。

**注意：**播发默认路由会中断 Windows 和其他 VM 许可证激活。请按照[此处](http://blogs.msdn.com/b/mast/archive/2015/05/20/use-azure-custom-routes-to-enable-kms-activation-with-forced-tunneling.aspx)的说明来解决此问题。

## BGP 社区支持（即将推出）

本部分概述如何配合 ExpressRoute 使用 BGP 社区。Azure 将播发公共互连路径中的路由并为路由标记适当的社区值。下面将会介绍这种方案的理由以及有关社区值的详细信息。但是，Azure 不遵循向 Azure 播发的路由的任何标记社区值。

如果你要在某个地缘政治区域内的任何一个对等互连位置通过 ExpressRoute 连接到 Azure，就能够访问该地缘政治边界内所有区域中的所有 Azure 云服务。

例如，如果你在北京通过 ExpressRoute 连接到 Azure，则就能够访问在上海托管的所有 Azure 云服务。

有关地缘政治地区、关联的 Azure 区域和对应的 ExpressRoute 对等互连位置的详细列表，请参阅 [ExpressRoute 合作伙伴和对等位置](/documentation/articles/expressroute-locations/)。

可以针对每个地缘政治区域购买多个 ExpressRoute 线路。如果拥有多个连接，则可以从异地冗余中获得明显的高可用性优势。如果你多条 ExpressRoute 线路，将从 Azure 收到同一组公共对等互连路径的前缀。这意味着你可以使用多个路径从你的网络接入 Azure。这可能会导致在网络中做出欠佳的路由决策。因此，你可能会在不同的服务上遇到欠佳的连接体验。

 Azure 使用适当的 BGP 社区值（表示托管前缀的区域）来标记通过公共对等互连播发的前缀。你可以依赖社区值来做出适当的路由决策，以向客户提供最佳路由。

### 操作路由首选项

Azure 不遵循你设置的任何 BGP 社区值。你需要为每个对等互连设置一对 BGP 会话，才能确保满足[可用性 SLA](/support/legal/sla/) 要求。但是，你可以依赖标准的 BGP 路由操作方法，将网络配置为首选某个链路。你可以将不同的 BGP 本地首选项应用到每个链路，以优先选择某个路径来从你的网络连接到 Azure。你可以在路由播发的前面加上 AS-PATH，以控制从 Azure 到你网络的流量传送。

## 后续步骤

- 配置 ExpressRoute 连接。

	- [创建经典部署模型的 ExpressRoute 线路](/documentation/articles/expressroute-howto-circuit-classic/)或[使用 Azure 资源管理器创建和修改 ExpressRoute 线路](/documentation/articles/expressroute-howto-circuit-arm/)
	- [为经典部署模型配置路由](/documentation/articles/expressroute-howto-routing-classic/)或[为资源管理器部署模型配置路由](/documentation/articles/expressroute-howto-routing-arm/)
	- [将经典 VNet 链接到 ExpressRoute 线路](/documentation/articles/expressroute-howto-linkvnet-classic/)或[将资源管理器 VNet 链接到 ExpressRoute 线路](/documentation/articles/expressroute-howto-linkvnet-arm/)

<!---HONumber=Mooncake_0104_2016-->