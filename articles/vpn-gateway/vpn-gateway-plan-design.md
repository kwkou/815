<properties
    pageTitle="跨界连接的规划和设计：Azure VPN 网关| Azure"
    description="了解跨界、混合与 VNet 到 VNet 连接的 VPN 网关规划和设计"
    services="vpn-gateway"
    documentationcenter="na"
    author="cherylmc"
    manager="timlt"
    editor=""
    tags="azure-service-management,azure-resource-manager" />
<tags
    ms.assetid="d5aaab83-4e74-4484-8bf0-cc465811e757"
    ms.service="vpn-gateway"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="04/05/2017"
    wacn.date="05/22/2017"
    ms.author="cherylmc"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="8fd60f0e1095add1bff99de28a0b65a8662ce661"
    ms.openlocfilehash="beee655bc636b98b765326b1c4e6885bad09b5a4"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/12/2017" />

# <a name="planning-and-design-for-vpn-gateway"></a>规划和设计 VPN 网关
规划和设计跨界配置和 VNet 到 VNet 配置可能很简单，也可能很复杂，具体取决于网络需求。 本文详细介绍基本规划和设计注意事项。

## <a name="planning"></a>规划
### <a name="compare"></a>跨界连接选项
若要安全地将本地站点连接到虚拟网络，可以使用三种不同的方式实现此目的：站点到站点、点到站点和 ExpressRoute。 比较可用的不同跨界连接。 选择的选项取决于不同的考虑因素，例如：

* 解决方案需要哪种吞吐量？
* 你是希望通信通过经由安全 VPN 的公共 Internet 进行，还是通过专用连接进行？
* 是否有可用的公共 IP 地址？
* 你是否打算使用 VPN 设备？ 如果是，它兼容吗？
* 是要连接几台计算机，还是想要站点持续连接？
* 想要创建的解决方案需要哪种类型的 VPN 网关？
* 你应该使用哪个网关 SKU？

下表可帮助选择最适合解决方案的连接选项。

[AZURE.INCLUDE [vpn-gateway-cross-premises](../../includes/vpn-gateway-cross-premises-include.md)]

### <a name="gwrequire"></a>根据 VPN 类型和 SKU 考虑网关要求
[AZURE.INCLUDE [vpn-gateway-gwsku](../../includes/vpn-gateway-gwsku-include.md)]

有关网关 SKU 的详细信息，请参阅 [VPN 网关设置](/documentation/articles/vpn-gateway-about-vpn-gateway-settings/#gwsku)。

#### <a name="aggregate-throughput-by-sku-and-vpn-type"></a>按 SKU 和 VPN 类型列出的聚合吞吐量
[AZURE.INCLUDE [vpn-gateway-table-gwtype-aggtput](../../includes/vpn-gateway-table-gwtype-aggtput-include.md)]

#### <a name="supported-configurations-by-sku-and-vpn-type"></a>SKU 和 VPN 类型支持的配置
[AZURE.INCLUDE [vpn-gateway-table-requirements](../../includes/vpn-gateway-table-requirements-include.md)]

### <a name="wf"></a>工作流
以下列表概述了云连接的常用工作流：

1. 设计和规划连接拓扑，并列出想要连接的所有网络的地址空间。
2. 创建 Azure 虚拟网络。 
3. 为虚拟网络创建 VPN 网关。
4. 创建并配置与本地网络或其他虚拟网络的连接（根据需要）。
5. 创建并配置 Azure VPN 网关的点到站点连接（根据需要）。

## <a name="design"></a>设计
### <a name="topologies"></a>连接拓扑
首先查看[关于 VPN 网关](/documentation/articles/vpn-gateway-about-vpngateways/)一文中的示意图。 该文章包含基本关系图、每种拓扑的部署模型（Resource Manager 或经典）和可用来部署配置的部署工具。   

### <a name="designbasics"></a>设计基础知识
以下部分介绍 VPN 网关基础知识。 此外，请考虑[网络服务限制](/documentation/articles/azure-subscription-service-limits/#networking-limits)。

#### <a name="subnets"></a>关于子网
创建连接时，必须考虑子网范围。 不能有重叠的子网地址范围。 子网重叠是指一个虚拟网络或本地位置包含其他位置所包含的相同地址空间。 这意味着，需要请求本地网络的网络工程师指定一个可用于 Azure IP 寻址空间/子网的范围。 使用的地址空间不能已在本地网络中使用。 

在使用 VNet 到 VNet 连接时，避免子网重叠也非常重要。 如果子网重叠并且发送端与目标 VNet 中存在同一个 IP 地址，VNet 到 VNet 连接将会失败。 Azure 无法将数据路由到另一个 VNet，因为目标地址是发送端 VNet 的一部分。 

VPN 网关需要一个特定的子网，称为网关子网。 所有网关子网都必须命名为 GatewaySubnet 才能正常工作。 切勿将网关子网命名为其他名称，并且不要在网关子网中部署 VM 或任何其他组件。 请参阅[网关子网](/documentation/articles/vpn-gateway-about-vpn-gateway-settings/#gwsub)。

#### <a name="local"></a>关于本地网络网关
本地网络网关通常是指本地位置。 在经典部署模型中，本地网络网关称为本地网络站点。 配置本地网络网关时，需为它命名、指定本地 VPN 设备的公共 IP 地址，并指定位于本地位置中的地址前缀。 Azure 将查看网络流量的目标地址前缀、查阅针对本地网络网关指定的配置，并相应地路由数据包。 你可以根据需要修改这些地址前缀。 有关详细信息，请参阅[本地网关](/documentation/articles/vpn-gateway-about-vpn-gateway-settings/#lng)。

#### <a name="gwtype"></a>关于网关类型
为拓扑选择正确的网关类型至关重要。 如果选择了错误的类型，网关将无法正常工作。 网关类型指定网关本身如何进行连接，它是 Resource Manager 部署模型的必需配置设置。

网关类型包括：

* Vpn
* ExpressRoute

#### <a name="connectiontype"></a>关于连接类型
每个配置需要特定的连接类型。 连接类型包括：

* IPsec
* Vnet2Vnet
* ExpressRoute
* VPNClient

#### <a name="vpntype"></a>关于 VPN 类型
每个配置需要特定的 VPN 类型。 如果要合并两个配置，例如创建连往相同 VNet 的站点到站点连接和点到站点连接，你必须使用同时符合这两个连接要求的 VPN 类型。

[AZURE.INCLUDE [vpn-gateway-vpntype](../../includes/vpn-gateway-vpntype-include.md)]

下表显示了映射到每个连接配置的 VPN 类型。 请确保网关的 VPN 类型与你想要创建的配置匹配。 

[AZURE.INCLUDE [vpn-gateway-table-vpntype](../../includes/vpn-gateway-table-vpntype-include.md)]

### <a name="devices"></a>用于站点到站点连接的 VPN 设备
若要配置站点到站点连接，不管使用哪种部署模型，都需要以下各项：

* 与 Azure VPN 网关兼容的 VPN 设备
* 不在 NAT 后面的面向公众的 IPv4 IP 地址

必须具有配置 VPN 设备的经验，或者有人可以帮助配置设备。 有关 VPN 设备的详细信息，请参阅[关于 VPN 设备](/documentation/articles/vpn-gateway-about-vpn-devices/)。 “VPN 设备”一文包含有关已验证的设备的信息、未验证的设备的要求，以及设备配置文档（如果有）的链接。

### <a name="forcedtunnel"></a>考虑强制隧道路由
对于大多数配置，你可以配置强制隧道。 借助强制隧道，你可以通过站点到站点 VPN 隧道，将全部 Internet 绑定流量重定向或“强制”返回到本地位置，以进行检查和审核。 这是很多企业 IT 策略的关键安全要求。 

如果没有强制隧道，来自 Azure 中 VM 的 Internet 绑定流量会始终通过 Azure 网络基础设施直接连接到 Internet，而无法选择对流量进行检查或审核。 未经授权的 Internet 访问可能会导致信息泄漏或其他类型的安全漏洞。

可以使用不同的工具在这两种部署模型中配置强制隧道连接。 有关详细信息，请参阅[配置强制隧道](/documentation/articles/vpn-gateway-forced-tunneling-rm/)。

**强制隧道关系图**

![Azure VPN 网关强制隧道示意图](./media/vpn-gateway-plan-design/forced-tunneling-diagram.png)

## <a name="next-steps"></a>后续步骤
请参阅 [VPN 网关常见问题解答](/documentation/articles/vpn-gateway-vpn-faq/)和[关于 VPN 网关](/documentation/articles/vpn-gateway-about-vpngateways/)文章，获取可帮助进行设计的详细信息。

有关特定网关设置的详细信息，请参阅 [About VPN Gateway Settings](/documentation/articles/vpn-gateway-about-vpn-gateway-settings/)（关于 VPN 网关设置）。