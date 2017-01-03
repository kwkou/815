<properties
    pageTitle="关于 VPN 网关 | Azure"
    description="了解 Azure 虚拟网络的 VPN 网关连接。"
    services="vpn-gateway"
    documentationcenter="na"
    author="cherylmc"
    manager="carmonm"
    editor=""
    tags="azure-resource-manager,azure-service-management" />  

<tags
    ms.assetid="2358dd5a-cd76-42c3-baf3-2f35aadc64c8"
    ms.service="vpn-gateway"
    ms.devlang="na"
    ms.topic="get-started-article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="10/18/2016"
    wacn.date="01/03/2017"
    ms.author="cherylmc" />  


# 关于 VPN 网关
虚拟网络网关用于在 Azure 虚拟网络与本地位置之间以及 Azure 内的虚拟网络（VNet 到 VNet）之间发送网络流量。配置 VPN 网关时，必须创建并配置虚拟网络网关和虚拟网络网关连接。

在 Resource Manager 部署模型中创建虚拟网络网关资源时，需要指定几项设置。其中一个必需的设置是“-GatewayType”。虚拟网络网关类型有两种：Vpn 和 ExpressRoute。

如果网络流量是在专用连接上发送，可以使用网关类型“ExpressRoute”，也称为 ExpressRoute 网关。如果网络流量通过公共连接以加密形式发送，可以使用网关类型“Vpn”，称为 VPN 网关。站点到站点、点到站点和 VNet 到 VNet 连接都使用 VPN 网关。

对于每种网关类型，每个虚拟网络只能有一个虚拟网络网关。例如，一个虚拟网络网关使用 -GatewayType ExpressRoute，另一个使用 GatewayType Vpn。本文重点介绍 VPN 网关。有关 ExpressRoute 的详细信息，请参阅 [ExpressRoute 技术概述](/documentation/articles/expressroute-introduction/)。

## 定价
[AZURE.INCLUDE [vpn-gateway-about-pricing-include](../../includes/vpn-gateway-about-pricing-include.md)]

## <a name="vpntype"></a>网关 SKU
[AZURE.INCLUDE [vpn-gateway-gwsku-include](../../includes/vpn-gateway-gwsku-include.md)]

有关 VPN 网关的网关 SKU 的详细信息，请参阅[网关 SKU](/documentation/articles/vpn-gateway-about-vpn-gateway-settings/#gwsku)。

### 按 SKU 列出的估计聚合吞吐量
[AZURE.INCLUDE [vpn-gateway-table-gwtype-aggthroughput](../../includes/vpn-gateway-table-gwtype-aggtput-include.md)]

## 配置 VPN 网关
配置 VPN 网关时，使用的说明取决于用于创建虚拟网络的部署模型。例如，如果使用经典部署模型创建了 VNet，将使用经典部署模型的指导原则和说明来创建及配置 VPN 网关设置。有关部署模型的详细信息，请参阅 [Understanding Resource Manager and classic deployment models](/documentation/articles/resource-manager-deployment-model/)（了解 Resource Manager 和经典部署模型）。

VPN 网关连接需依赖于多个具有特定设置的资源。大多数资源可单独进行配置，不过，在某些情况下，必须按特定的顺序进行配置。可以使用一种配置工具（例如 Azure 门户预览）开始创建和配置资源。稍后可以切换到另一种工具（如 PowerShell）来配置其他资源，或者在适当的情况下修改现有资源。目前无法在 Azure 门户预览中配置每一项资源和资源设置。需要使用特定的配置工具时，本文中针对每种连接拓扑提供的说明都有指明。有关 VPN 网关的各项资源和设置的信息，请参阅 [About VPN Gateway settings](/documentation/articles/vpn-gateway-about-vpn-gateway-settings/)（关于 VPN 网关设置）。

以下部分包含的表格列出了以下信息：

* 可用的部署模型
* 可用的配置工具
* 直接转到某篇文章的链接（如果有）

使用图示和描述来帮助选择符合要求的连接拓扑。这些图示显示主要基准拓扑，但也可以使用这些图示作为指导来构建更复杂的配置。

## <a name="site-to-site-and-multi-site"></a> 站点到站点和多站点
### 站点到站点
站点到站点 (S2S) VPN 网关连接是通过 IPsec/IKE（IKEv1 或 IKEv2）VPN 隧道建立的连接。这种类型的连接要求 VPN 设备位于本地，并且分配有公共 IP 地址，不在 NAT 的后面。S2S 连接可用于跨界和混合配置。

![S2S 连接](./media/vpn-gateway-about-vpngateways/demos2s.png "站点到站点")  


### 多站点
可以在 VNet 与多个本地网络之间创建并配置 VPN 网关连接。使用多个连接时，必须使用 RouteBased VPN 类型（经典 VNet 的动态网关）。由于一个 VNet 只能有一个 VPN 网关，通过该网关的所有连接将共享可用带宽。这通常称为“多站点”连接。

![多站点连接](./media/vpn-gateway-about-vpngateways/demomulti.png "多站点")  


### 站点到站点和多站点的部署模型与方法
[AZURE.INCLUDE [vpn-gateway-table-site-to-site](../../includes/vpn-gateway-table-site-to-site-include.md)]

## VNet 到 VNet
将虚拟网络连接到虚拟网络（VNet 到 VNet）类似于将 VNet 连接到本地站点位置。这两种连接类型都使用 VPN 网关来提供使用 IPsec/IKE 的安全隧道。甚至可以将 VNet 到 VNet 通信与多站点连接配置结合使用。这样，便可以建立将跨界连接与虚拟网络间连接相结合的网络拓扑。

连接的 VNet 可以：

* 在相同或不同的区域中
* 在相同或不同的订阅中
* 在相同或不同的部署模型中

![VNet 到 VNet 连接](./media/vpn-gateway-about-vpngateways/demov2v.png "vnet-to-vnet")  


#### 部署模型之间的连接
Azure 当前具有两个部署模型：经典模型和 Resource Manager 模型。如果 Azure 已经使用了一段时间，则您的 Azure VM 和实例角色可能是在经典 VNet 上运行。而较新的 VM 和角色实例可能是在 Resource Manager 中创建的 VNet 上运行。可以在 Vnet 之间创建连接，使其中一个 VNet 中的资源能够直接与另一个 VNet 中的资源通信。

#### VNet 对等互连
只要虚拟网络符合特定要求，就能使用 VNet 对等互连来创建连接。VNet 对等互连不使用虚拟网络网关。有关详细信息，请参阅 [VNet 对等互连](/documentation/articles/virtual-network-peering-overview/)。

### VNet 到 VNet 的部署模型和方法
[AZURE.INCLUDE [vpn-gateway-table-vnet-to-vnet](../../includes/vpn-gateway-table-vnet-to-vnet-include.md)]

## <a name="point-to-site"></a>点到站点
使用点到站点 (P2S) VPN 网关连接可以创建从单个客户端计算机到虚拟网络的安全连接。P2S 是基于 SSTP（安全套接字隧道协议）的 VPN 连接。P2S 连接不需要 VPN 设备或面向公众的 IP 地址即可运行。从客户端计算机启动 VPN 连接即可建立这种连接。如果要从远程位置（例如从家里或会议室）连接到 VNet，或者只有少数几个需要连接到 VNet 的客户端，则此解决方案会很有用。可以通过相同的 VPN 网关将 P2S 连接与 S2S 连接结合使用，前提是这两个连接的所有配置要求都兼容。

![点到站点连接](./media/vpn-gateway-about-vpngateways/demop2s.png "点到站点")  


### 点到站点的部署模型和方法
[AZURE.INCLUDE [vpn-gateway-table-point-to-site](../../includes/vpn-gateway-table-point-to-site-include.md)]

## ExpressRoute
[AZURE.INCLUDE [expressroute-intro](../../includes/expressroute-intro-include.md)]

在 ExpressRoute 连接中，虚拟网络网关的网关类型配置为“ExpressRoute”而不是“Vpn”。有关 ExpressRoute 的详细信息，请参阅 [ExpressRoute 技术概述](/documentation/articles/expressroute-introduction/)。

## 站点到站点和 ExpressRoute 的共存连接
ExpressRoute 可以从 WAN 与 Microsoft 服务（包括 Azure）直接建立专用连接，不需要通过公共 Internet。站点到站点 VPN 流量以加密方式通过公共 Internet 传输。能够为同一个虚拟网络配置站点到站点 VPN 和 ExpressRoute 连接可带来诸多好处。

可以将站点到站点 VPN 配置为 ExpressRoute 的安全故障转移路径，或者使用站点到站点 VPN 连接到不属于网络的一部分但却已通过 ExpressRoute 进行连接的站点。请注意，对于同一虚拟网络，需要两个虚拟网络网关，一个使用 -GatewayType Vpn，另一个使用 -GatewayType ExpressRoute。

![共存连接](./media/vpn-gateway-about-vpngateways/demoer.png "expressroute-site2site")  


### S2S 和 ExpressRoute 的部署模型与方法
[AZURE.INCLUDE [vpn-gateway-table-coexist](../../includes/vpn-gateway-table-coexist-include.md)]

## 后续步骤
规划 VPN 网关配置。请参阅 [VPN Gateway Planning and Design](/documentation/articles/vpn-gateway-plan-design/)（VPN 网关规划和设计）。

<!---HONumber=Mooncake_1226_2016-->