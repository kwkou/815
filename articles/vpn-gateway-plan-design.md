<properties 
   pageTitle="VPN 网关规划和设计 | Azure"
   description="了解跨界、混合与 VNet 到 VNet 连接的 VPN 网关规划和设计"
   services="vpn-gateway"
   documentationCenter="na"
   authors="cherylmc"
   manager="carmonm"
   editor=""
   tags="azure-service-management,azure-resource-manager"/>
<tags 
   ms.service="vpn-gateway"
   ms.date="04/20/2016"
   wacn.date="06/24/2016"/>

# 规划和设计 VPN 网关

规划和设计跨界配置和 VNet 到 VNet 配置可以很简单，也可以相当复杂，具体取决于你的网络需求。本文将详细介绍基本规划和设计注意事项。

## 规划


### <a name="compare"></a>跨界连接选项比较

如果你决定要安全地将本地站点连接到虚拟网络，可以使用三种不同的方式实现此目的：站点到站点、点到站点和 ExpressRoute。比较可用的不同跨界连接。选择的选项可能取决于不同的考虑因素，例如：


- 解决方案需要哪种吞吐量？
- 你是希望通信通过经由安全 VPN 的公共 Internet 进行，还是通过专用连接进行？
- 是否有可用的公共 IP 地址？
- 你是否打算使用 VPN 设备？ 如果是，它兼容吗？
- 是要连接几台计算机，还是想要站点持续连接？
- 想要创建的解决方案需要哪种类型的 VPN 网关？
- 你应该使用哪个网关 SKU？


下表可以帮助你为解决方案确定最佳的连接选项。


[AZURE.INCLUDE [vpn-gateway-cross-premises](../includes/vpn-gateway-cross-premises-include.md)]



### <a name="gwrequire"></a>根据 VPN 类型和 SKU 考虑网关要求


在创建 VPN 网关时，你需要指定想要使用的网关 SKU。 
共有 3 种 VPN 网关 SKU：

- 基本
- 标准
- 高性能

[AZURE.INCLUDE [vpn-gateway-table-requirements](../includes/vpn-gateway-table-requirements-include.md)]



###<a name="aggthroughput"></a> 网关类型和聚合吞吐量估计

下表显示网关类型和估计的聚合吞吐量。估计的聚合吞吐量可能是设计时的决定因素。
网关 SKU 之间并无定价差异。有关定价的信息，请参阅 [VPN 网关定价](/home/features/vpn-gateway/pricing/)。此表适用于资源管理器与经典部署模型。

[AZURE.INCLUDE [vpn-gateway-table-gwtype-aggtput](../includes/vpn-gateway-table-gwtype-aggtput-include.md)]



### <a name="wf"></a>工作流

以下列表概述了云连接的常用工作流：

1.	设计和规划连接拓扑，并列出想要连接的所有网络的地址空间。
2.	创建 Azure 虚拟网络。 
3.	为虚拟网络创建 VPN 网关。
4.	创建并配置与本地网络或其他虚拟网络的连接（根据需要）。
5.	创建并配置 Azure VPN 网关的点到站点连接（根据需要）。
 

## 设计

###<a name="topologies"></a> 连接拓扑

首先请阅读[Connection toplogies（连接拓扑）](/documentation/articles/vpn-gateway-topology/)一文。该文章包含基本原理图、每种拓扑的部署模型（资源管理器或经典）和可用来部署配置的部署工具。

###<a name="designbasics"></a> 设计基础知识

以下部分介绍了 VPN 网关基础知识。此外，你需要考虑[网络服务限制](/documentation/articles/azure-subscription-service-limits/#networking-limits)。


#### <a name="subnets"></a>关于子网

在规划和设计最适合环境的连接时，必须考虑到可用的 IP 地址范围和子网。

若要配置 VPN 网关，需要为 VNet 创建网关子网。所有网关子网都必须命名为 GatewaySubnet 才能正常工作。切勿将网关子网命名为其他名称，并且不要在网关子网中部署 VM 或任何其他组件。有关网关子网的详细信息，请参阅“About VPN Gateways”（关于 VPN 网关）中的 [Gateway subnet（网关子网）](/documentation/articles/vpn-gateway-about-vpngateways/#gwsub)部分。

创建连接时，在许多情况下，你需要考虑连接之间的子网地址范围重叠。子网重叠是指一个虚拟网络或本地位置包含其他位置所包含的相同地址空间。这意味着，你需要请求本地网络的网络工程师指定一个可用于 Azure IP 寻址空间/子网的范围。你需要尚未在本地网络中使用的地址空间。

在使用 VNet 到 VNet 连接时，避免子网重叠也非常重要。如果子网重叠并且发放端与目标 VNet 中存在同一个 IP 地址，则创建 VNet 到 VNet 连接将会失败。在这种情况下，Azure 无法将数据路由到另一个 VNet，因为目标地址是发送端 VNet 的一部分。



#### <a name="local"></a>关于局域网网关

局域网网关通常是指你的本地位置。在经典部署模型中，局域网网关称为本地站点。指定局域网网关的名称（即本地 VPN 设备的公共 IP 地址），并指定位于本地位置的地址前缀。Azure 将查看网络流量的目标地址前缀、查阅你为局域网网关指定的配置，并相应地路由数据包。你可以根据需要修改这些地址前缀。有关局域网网关的详细信息，请参阅“About VPN Gateways”（关于 VPN 网关）一文中的 [Local network gateways（局域网网关）](/documentation/articles/vpn-gateway-about-vpngateways/#lng)。


#### <a name="gwtype"></a>关于网关类型

为拓扑选择正确的网关类型至关重要。如果选择了错误的类型，你的网关将无法正常工作。网关类型指定网关本身如何进行连接，它是资源管理器部署模型的必需配置设置。

网关类型包括：

- Vpn
- ExpressRoute

####<a name="connectiontype"></a> 关于连接类型

每个配置需要特定的连接类型。连接类型包括：

- IPsec
- Vnet2Vnet
- ExpressRoute
- VPNClient


#### <a name="vpntype"></a>关于 VPN 类型

每个配置需要特定的 VPN 类型才能正常运行。如果要合并两个配置，例如创建连往相同 VNet 的站点到站点连接和点到站点连接，你必须使用同时符合这两个连接要求的 VPN 类型。在点到站点和站点到站点并存连接的情况下，使用 Azure 资源管理器部署模型时必须使用基于路由的 VPN 类型，而使用经典部署模式时必须使用动态网关。

[AZURE.INCLUDE [vpn-gateway-vpntype](../includes/vpn-gateway-vpntype-include.md)]

下表显示了映射到每个连接配置的 VPN 类型。请确保网关的 VPN 类型与你想要创建的配置匹配。


[AZURE.INCLUDE [vpn-gateway-table-vpntype](../includes/vpn-gateway-table-vpntype-include.md)]

### <a name="devices"></a> 为站点到站点连接选择 VPN 设备

若要配置站点到站点连接，不管你使用哪种部署模型，都需要以下各项：

- 与 Azure VPN 网关兼容的 VPN 设备
- 不在 NAT 后面的面向公众的 IPv4 IP 地址

你需要有配置 VPN 设备的经验。有关 VPN 设备的详细信息，请参阅 [About VPN devices（关于 VPN 设备）](/documentation/articles/vpn-gateway-about-vpn-devices/)。“VPN 设备”一文包含有关已验证的设备的信息、未验证的设备的要求，以及每个设备的设备配置文档（如果有）的链接。

### <a name="forcedtunnel"></a> 考虑强制隧道路由

对于大多数配置，你可以配置强制隧道。借助强制隧道，你可以通过站点到站点 VPN 隧道，将全部 Internet 绑定流量重定向或“强制”返回到本地位置，以进行检查和审核。这是很多企业 IT 策略的关键安全要求。

没有强制隧道，来自 Azure 中虚拟机的 Internet 绑定流量会始终通过 Azure 网络基础设施直接连接到 Internet。没有该选项，您无法对流量进行检查或审核。未经授权的 Internet 访问可能会导致信息泄漏或其他类型的安全漏洞。有关配置强制隧道的详细信息，请参阅 [About forced tunneling for the classic deployment model（关于经典部署模型的强制隧道）](/documentation/articles/vpn-gateway-about-forced-tunneling/)和 [About forced tunneling for the Resource Manager deployment model（关于资源管理器部署模型的强制隧道）](/documentation/articles/vpn-gateway-about-forced-tunneling/)。

**强制隧道示意图**

![强制隧道连接](./media/vpn-gateway-plan-design/forced-tunnel.png "强制隧道")


此表列出了强制隧道适用的部署模型、可用于配置强制隧道的部署工具，以及直接指向可用文章的链接。随着新的可用文章的发布，我们会经常更新表格。

[AZURE.INCLUDE [vpn-gateway-table-forcedtunnel](../includes/vpn-gateway-table-forcedtunnel-include.md)]



## 后续步骤

请参阅 [VPN Gateway FAQ（VPN 网关常见问题）](/documentation/articles/vpn-gateway-vpn-faq/)和 [About VPN Gateways（关于 VPN 网关）](/documentation/articles/vpn-gateway-about-vpngateways/)，以获取可帮助你进行设计的详细信息。有关连接拓扑的详细信息，请参阅 [Connection toplogies（连接拓扑）](/documentation/articles/vpn-gateway-topology/)。




<!---HONumber=Mooncake_0613_2016-->
