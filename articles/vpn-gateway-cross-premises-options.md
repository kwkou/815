<properties 
   pageTitle="关于虚拟网络的安全跨界连接 | Azure"
   description="了解虚拟网络的安全跨界连接类型，包括站点到站点连接、点到站点连接和 ExpressRoute 连接。"
   services="vpn-gateway"
   documentationCenter="na"
   authors="cherylmc"
   manager="carmonm"
   editor="" />
<tags 
   ms.service="vpn-gateway"
   ms.date="03/08/2016"
   wacn.date="05/10/2016" />

# 关于虚拟网络的安全跨界连接

本文介绍用于将本地站点连接到 Azure 虚拟网络的不同方法。本文适用于 Resource Manager 与经典部署模型。

可以使用三个连接选项：站点到站点、点到站点和 ExpressRoute。选择的选项可能取决于不同的考虑因素，例如：


- 解决方案需要哪种吞吐量？
- 你是希望通信通过经由安全 VPN 的公共 Internet 进行，还是通过专用连接进行？
- 是否有可用的公共 IP 地址？
- 你是否打算使用 VPN 设备？ 如果是，它兼容吗？
- 是要连接几台计算机，还是想要站点持续连接？
- 想要创建的解决方案需要哪种类型的 VPN 网关？

下表可以帮助你为解决方案确定最佳的连接选项。


|- | **点到站点** | **站点到站点** | **ExpressRoute** |
|------------------------------|----------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------|
| **Azure 支持的服务** | 云服务和虚拟机 | 云服务和虚拟机 | [服务列表](/documentation/articles/expressroute-faqs/#supported-services) |
| **典型带宽** | 通常 < 100 Mbps（总计） | 通常 < 100 Mbps（总计） | 50 Mbps、100 Mbps、200 Mbps、500 Mbps、1 Gbps、2 Gbps、5 Gbps、10 Gbps |
| **支持的协议** | 安全套接字隧道协议 (SSTP) | IPsec | 通过 VLAN、NSP 的 VPN 技术（MPLS、VPLS...）直接连接 |
| **路由** | 基于路由（动态） | 支持基于策略（静态路由）和基于路由（动态路由 VPN） | BGP |
| **连接复原能力** | 主动-被动 | 主动-被动 | 主动-主动 |
| **典型用例** | 云服务和虚拟机的原型设计、开发/测试/实验方案 | 云服务和虚拟机的开发/测试/实验方案和小规模生产工作负荷 | 访问所有 Azure 服务（已验证列表）、企业级和任务关键型工作负荷、备份、大数据、Azure 即 DR 站点 |
| **SLA** | [SLA](/support/legal/sla) | [SLA](/support/legal/sla) | [SLA](/support/legal/sla) |
| **价格** | [价格](/home/features/vpn-gateway/pricing/) | [价格](/home/features/vpn-gateway/pricing/) | [价格](/home/features/expressroute/pricing/) |
| **技术文档** | [VPN 网关文档](/documentation/services/vpn-gateway) | [VPN 网关文档](/documentation/services/vpn-gateway) | [ExpressRoute 文档](/documentation/services/expressroute) |
| **常见问题** | [VPN 网关常见问题](/documentation/articles/vpn-gateway-vpn-faq/) | [VPN 网关常见问题](/documentation/articles/vpn-gateway-vpn-faq/) | [ExpressRoute 常见问题](/documentation/articles/expressroute-faqs/) |


## 站点到站点连接

通过站点到站点的 VPN，可以在本地站点和虚拟网络之间创建安全连接。若要创建站点到站点连接，请配置本地网络上的 VPN 设备，使其与 Azure VPN 网关创建安全连接。连接该创建后，本地网络上的资源可直接并安全地与位于虚拟网络中的资源进行通信。站点到站点连接不需要为本地网络上的每台客户端计算机都单独建立一个连接，即可访问虚拟网络中的资源。

**在下列情况下，请使用站点到站点连接：**

- 你想要创建混合解决方案。
- 你希望在本地位置和虚拟网络之间建立连接，而无需在客户端进行配置。
- 你希望建立持久的连接。 

**要求**

- 本地 VPN 设备必须有面向 Internet 的 IPv4 IP 地址。该设备不能在 NAT 后面。
- 你必须有兼容的 VPN 设备。请参阅[关于 VPN 设备](/documentation/articles/vpn-gateway-about-vpn-devices/)。 
- 使用的 VPN 设备必须与解决方案所需的网关类型兼容。请参阅[关于 VPN 网关](/documentation/articles/vpn-gateway-about-vpngateways/)。
- 网关 SKU 也会影响聚合吞吐量。有关详细信息，请参阅[网关 SKU](/documentation/articles/vpn-gateway-about-vpngateways/#gateway-skus)。 

有关如何使用 Azure 经典管理门户和经典部署模型配置站点到站点 VPN 网关连接的信息，请参阅 [Configure a virtual network with a Site-to-Site VPN connection for the classic deployment model（使用经典部署模型的站点到站点 VPN 连接配置虚拟网络）](/documentation/articles/vpn-gateway-site-to-site-create/)。有关如何使用资源管理器部署模型配置站点到站点 VPN 的信息，请参阅 [Create a virtual network with a Site-to-Site VPN connection for the Resource Manager deployment model（使用 Resource Manager 部署模型的站点到站点 VPN 连接创建虚拟网络）](/documentation/articles/vpn-gateway-create-site-to-site-rm-powershell/)。


## 点到站点连接

使用点到站点 VPN 也可以创建与虚拟网络的安全连接。在点到站点配置中，在每台要连接到虚拟网络的客户端计算机上单独配置连接。点到站点连接不需要 VPN 设备。此类连接使用安装在每台客户端计算机上的 VPN 客户端。通过手动从本地客户端计算机上启动连接，建立该 VPN。

点到站点和站点到站点配置可以并存，但与站点到站点连接不同的是，无法将点到站点连接配置为与 ExpressRoute 同时连接到同一虚拟网络。

**在下列情况下，请使用点到站点连接：**

- 只想配置少数几个客户端连接到虚拟网络站点。

- 你希望从远程位置连接到虚拟网络。例如，从咖啡厅或大会现场进行连接。

- 你拥有站点到站点连接，但有一些客户端需要从远程位置连接。

- 无权访问符合站点到站点连接的最低要求的 VPN 设备。

- VPN 设备没有面向 Internet 的 IPv4 IP 地址。

有关为经典部署模型配置点到站点连接的详细信息，请参阅 [Configure a Point-to-Site VPN connection to a virtual network for the classic deployment model（配置与经典部署模型的虚拟网络的点到站点 VPN 连接）](/documentation/articles/vpn-gateway-point-to-site-create/)。有关为 Resource Manager 部署模型配置点到站点连接的详细信息，请参阅 [Configure a Point-to-Site VPN connection to a virtual network for the Resource Manager deployment model（配置与 Resource Manager 部署模型的虚拟网络的点到站点 VPN 连接）](/documentation/articles/vpn-gateway-howto-point-to-site-rm-ps/)。

## ExpressRoute 连接

使用 Azure ExpressRoute，可在 Azure 数据中心与你的本地环境或共同租用环境中的基础结构之间创建专用连接。ExpressRoute 连接不通过公共 Internet，与通过 Internet 的典型连接相比，提供更高的可靠性、更快的速度、更低的延迟和更高的安全性。

在某些情况下，使用 ExpressRoute 连接在本地和 Azure 之间传输数据还可以产生显著的成本效益。使用 ExpressRoute，你可以在 ExpressRoute 位置（Exchange 提供商设施）建立与 Azure 的连接，也可以直接从网络服务提供商提供的现有 WAN 网络（例如 MPLS VPN）连接到 Azure。

有关 ExpressRoute 的详细信息，请参阅 [ExpressRoute 技术概述](/documentation/articles/expressroute-introduction/)。


## 后续步骤

有关详细信息，请参阅 [VPN Gateway FAQ（VPN 网关常见问题）](/documentation/articles/vpn-gateway-vpn-faq/)和 [ExpressRoute FAQ（ExpressRoute 常见问题）](/documentation/articles/expressroute-faqs/)。




<!---HONumber=Mooncake_0425_2016-->
