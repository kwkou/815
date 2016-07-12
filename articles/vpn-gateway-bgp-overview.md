<properties
   pageTitle="Azure VPN 网关的 BGP 概述 | Azure"
   description="本文概述了 Azure VPN 网关的 BGP。"
   services="vpn-gateway"
   documentationCenter="na"
   authors="yushwang"
   manager="rossort"
   editor=""
   tags=""/>

<tags
   ms.service="vpn-gateway"
   ms.date="04/26/2016"
   wacn.date="06/24/2016"/>

# Azure VPN 网关的 BGP 概述

本文概述了 Azure VPN 网关中支持的 BGP（边界网关协议）。

## 关于 BGP

BGP 是通常在 Internet 上使用的，用于在两个或更多网络之间交换路由和可访问性信息的标准路由协议。在 Azure 虚拟网络的上下文中使用时，BGP 允许 Azure VPN 网关和本地 VPN 设备（称为 BGP 对等节点或邻居）交换“路由”，这些路由将通知这两个网关这些前缀的可用性和可访问性，以便这些前缀可通过涉及的网关或路由器。BGP 还可以通过将 BGP 网关从一个 BGP 对等节点获知的路由传播到所有其他 BGP 对等节点来允许在多个网络之间传输路由。
 
### 为什么使用 BGP？

BGP 是可用于 Azure 基于路由的 VPN 网关的可选功能。在启用此功能之前，你还应确保本地 VPN 设备支持 BGP。你可以继续使用不带 BGP 的 Azure VPN 网关和本地 VPN 设备。它等效于在你的网络与 Azure 之间使用静态路由（不带 BGP）与使用带 BGP 的动态路由。

使用 BGP 有几个优点和新功能：

#### 支持自动和灵活的前缀更新

使用 BGP，你只需通过 IPsec S2S VPN 隧道为特定 BGP 对等节点声明最小前缀。它最小可为本地 VPN 设备的 BGP 对等节点 IP 地址的主机前缀（/32）。你可以控制要将哪些本地网络前缀播发到 Azure 以允许 Azure 虚拟网络访问。
	
你还可以播发更大的前缀，可以包括一些 VNet 地址前缀，如默认路由 (0.0.0.0/0) 或大型专用 IP 地址空间（例如，10.0.0.0/8）。但请注意，这些前缀不能与任一 VNet 前缀相同。与 VNet 前缀相同的这些路由将被拒绝。

#### 支持 VNet 与本地站点之间的多个隧道基于 BGP 自动进行故障转移

你可以在同一位置的 Azure VNet 和本地 VPN 设备之间建立多个连接。在主-主配置中，此功能在两个网络之间提供多个隧道（路径）。如果其中一个隧道断开连接，则将通过 BGP 撤消相应的路由，流量将会自动转移到其余隧道。
	
下图显示了此高度可用设置的简单示例：
	
![多个活动路径](./media/vpn-gateway-bgp-overview/multiple-active-tunnels.png)

#### 支持本地网络与多个 Azure VNet 之间的传输路由

BGP 使多个网关可以从不同网络获知和传播前缀，而无论它们是直接还是间接连接。这可以为本地站点之间或跨多个 Azure 虚拟网络的 Azure VPN 网关启用传输路由。
	
下图显示了多跃点拓扑的示例，其中的多个路径可以通过 Microsoft 网络中的 Azure VPN 网关在两个本地网络之间传输流量：

![多跃点传输](./media/vpn-gateway-bgp-overview/full-mesh-transit.png)

## BGP 常见问题


[AZURE.INCLUDE [vpn-gateway-bgp-faq-include](../includes/vpn-gateway-bpg-faq-include.md)]




## 后续步骤

有关为跨界连接和 VNet 到 VNet 连接配置 BGP 的步骤，请参阅[在 Azure VPN 网关上使用 BGP 入门](/documentation/articles/vpn-gateway-bgp-resource-manager-ps/)。


<!---HONumber=Mooncake_0613_2016-->
