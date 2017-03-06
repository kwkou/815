<properties
    pageTitle="Azure VPN 网关常见问题解答 | Azure"
    description="VPN 网关常见问题。Azure 虚拟网络跨界连接、混合配置连接和 VPN 网关的常见问题。"
    services="vpn-gateway"
    documentationcenter="na"
    author="cherylmc"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="6ce36765-250e-444b-bfc7-5f9ec7ce0742"
    ms.service="vpn-gateway"
    ms.devlang="na"
    ms.topic="get-started-article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="02/14/2017"
    wacn.date="03/03/2017"
    ms.author="cherylmc" />  


# VPN 网关常见问题
## 连接到虚拟网络
### 是否可以连接不同 Azure 区域中的虚拟网络？
是。事实上，没有任何区域约束。一个虚拟网络可以连接到同一区域中的其他虚拟网络，也可以连接到其他 Azure 区域中的其他虚拟网络。

### 是否可以连接不同订阅中的虚拟网络？
是。

### 是否可以从一个虚拟网络连接到多个站点？
你可以使用 Windows PowerShell 和 Azure REST API 连接到多个站点。请参阅[多站点与 VNet 到 VNet 连接](#V2VMulti)的“常见问题”部分。

### 我的跨界连接选项有哪些？
支持以下跨界连接：

* [站点到站点](/documentation/articles/vpn-gateway-howto-site-to-site-resource-manager-portal/) – 基于 IPsec（IKE v1 和 IKE v2）的 VPN 连接。此类型的连接需要 VPN 设备或 RRAS。
* [点到站点](/documentation/articles/vpn-gateway-howto-point-to-site-resource-manager-portal/) – 基于 SSTP（安全套接字隧道协议）的 VPN 连接。此连接不需要 VPN 设备。
* [VNet 到 VNet](/documentation/articles/vpn-gateway-howto-vnet-vnet-resource-manager-portal/) – 这种连接类型与站点到站点配置相同。VNet 到 VNet 是一种基于 IPsec（IKE v1 和 IKE v2）的 VPN 连接。它不需要 VPN 设备。
* [多站点](/documentation/articles/vpn-gateway-howto-multi-site-to-site-resource-manager-portal/) – 这是站点到站点配置的变体，可将多个本地站点连接到虚拟网络。
* [ExpressRoute](/documentation/articles/expressroute-introduction/) – ExpressRoute 可以从 WAN 直接连接到 Azure，不需要通过公共 Internet。有关详细信息，请参阅 [ExpressRoute 技术概述](/documentation/articles/expressroute-introduction/)和 [ExpressRoute 常见问题解答](/documentation/articles/expressroute-faqs/)。

有关 VPN 网关连接的详细信息，请参阅[关于 VPN 网关](/documentation/articles/vpn-gateway-about-vpngateways/)。

### 站点到站点连接和点到站点连接的区别是什么？
**站点到站点**配置位于本地位置和 Azure 之间。这意味着用户可以从位于本地的任何计算机连接到虚拟网络中的任何虚拟机或角色实例，具体取决于如何选择配置路由。它对于需要始终可用的跨界连接来说是一个极佳的选项，很适合混合配置。此类连接依赖于 IPsec VPN 设备（硬件或软件设备），该设备必须部署在网络边缘。若要创建此类连接，必须具有所需的 VPN 硬件和面向外部的 IPv4 地址。

**点到站点**配置允许用户从任何位置的单一计算机连接到虚拟网络中的任何内容。它使用 Windows 内置的 VPN 客户端。在进行点到站点配置时，你需要安装证书和 VPN 客户端配置包，其中包含的设置允许你的计算机连接到虚拟网络中的任何虚拟机或角色实例。此连接适用于需要连接到虚拟网络但该虚拟网络不在本地的情况。当你无法访问 VPN 硬件或面向外部的 IPv4 地址（二者是进行站点到站点连接所必需的）时，它也是一个很好的选项。

可以将虚拟网络配置为同时使用站点到站点连接和点到站点连接，前提是使用基于路由的 VPN 类型为网关创建站点到站点连接。在经典部署模型中，基于路由的 VPN 类型称为动态网关。

## 虚拟网络网关

### VPN 网关是否为虚拟网络网关？
VPN 网关是一种虚拟网络网关。VPN 网关通过公共连接在虚拟网络和本地位置之间发送加密流量。还可使用 VPN 网关在虚拟网络之间发送流量。创建 VPN 网关时，指定“GatewayType”的值为“Vpn”。有关详细信息，请参阅[关于 VPN 网关配置设置](/documentation/articles/vpn-gateway-about-vpn-gateway-settings/)。

### 什么是基于策略的（静态路由）网关？
基于策略的网关实施基于策略的 VPN。基于策略的 VPN 会根据本地网络和 Azure VNet 之间的地址前缀的各种组合，加密数据包并引导其通过 IPsec 隧道。通常会在 VPN 配置中将策略（或流量选择器）定义为访问列表。

### 什么是基于路由的（动态路由）网关？
基于路由的网关可实施基于路由的 VPN。基于路由的 VPN 使用 IP 转发或路由表中的“路由”将数据包定向到相应的隧道接口中。然后，隧道接口会加密或解密出入隧道的数据包。基于路由的 VPN 的策略或流量选择器配置为任意到任意（或通配符）。

### 是否需要“GatewaySubnet”？
是的。网关子网包含虚拟网络网关服务使用的 IP 地址。若要配置虚拟网络网关，需要为 VNet 创建网关子网。所有网关子网都必须命名为“GatewaySubnet”才能正常工作。不要对网关子网使用其他名称。此外，不要在网关子网中部署 VM 或其他组件。

创建网关子网时，需指定子网包含的 IP 地址数。网关子网中的 IP 地址将分配到网关服务。某些配置需要多个 IP 地址，并将其分配到网关服务而不是执行其他操作。需确保网关子网包含足够的 IP 地址，以适应未来的增长和可能的其他新连接配置。因此，尽管网关子网最小可以创建为 /29，但建议创建 /28 或更大（/28、/27、/26 等）的网关子网。查看要创建的配置的要求，并验证所拥有的网关子网是否可满足这些要求。

### 是否可以将虚拟机或角色实例部署到网关子网？
否。

### 是否可以先获得 VPN 网关 IP 地址，然后再创建网关？
否。必须先创建网关，然后才能获得 IP 地址。如果先删除再重新创建 VPN 网关，IP 地址将会更改。

### VPN 隧道如何进行身份验证？
Azure VPN 使用 PSK（预共享密钥）身份验证。我们在创建 VPN 隧道时生成一个预共享密钥 (PSK)。你可以使用设置预共享密钥 PowerShell cmdlet 或 REST API 将自动生成的 PSK 更改成你自己的 PSK。

### 是否可以使用“设置预共享密钥 API”配置基于策略的（静态路由）网关 VPN？
是，“设置预共享密钥 API”和 PowerShell cmdlet 可用于配置 Azure 基于策略的（静态）VPN 和基于路由的（动态）路由 VPN。

### 是否可以使用其他身份验证选项？
我们仅限使用预共享密钥 (PSK) 进行身份验证。

### 如何指定通过 VPN 网关的流量？

####Resource Manager 部署模型
* PowerShell：使用“AddressPrefix”指定本地网络网关的流量。
* Azure 门户预览：导航到“本地网络网关”>“配置”>“地址空间”。

####经典部署模型
* Azure 门户预览：导航到“经典虚拟网络”>“VPN 连接”>“站点到站点 VPN 连接”>“本地站点名称”>“本地站点”>“客户端地址空间”。
* 经典管理门户：请在“本地网络”下的“网络”页上为虚拟网络添加要通过网关发送的每个范围。

### 是否可以配置强制隧道？
是。请参阅[配置强制隧道](/documentation/articles/vpn-gateway-about-forced-tunneling/)。

### 是否可以在 Azure 中设置自己的 VPN 服务器，然后使用它连接到本地网络？
是。你可以在 Azure 中部署你自己的 VPN 网关或服务器，可以从 Azure 应用商店部署，也可以通过创建你自己的 VPN 路由器部署。需要在虚拟网络中配置用户定义的路由，确保流量在本地网络和虚拟网络子网之间正确路由。

### 我的 VPN 网关上的某些端口为何处于打开状态？
这些端口是进行 Azure 基础结构通信所必需的。它们受 Azure 证书的保护（处于锁定状态）。如果没有适当的证书，外部实体（包括这些网关的客户）将无法对这些终结点施加任何影响。

VPN 网关基本上是一个多宿主设备，其中一个 NIC 进入客户专用网络，另一个 NIC 面向公共网络。因合规性原因，Azure 基础结构实体无法进入客户专用网络，因此需利用公共终结点进行基础结构通信。Azure 安全审核会定期扫描公共终结点。

### <a name="multi-site-and-vnet-to-vnet-connectivity"></a>有关网关类型、要求和吞吐量的详细信息
有关详细信息，请参阅[关于 VPN 网关配置设置](/documentation/articles/vpn-gateway-about-vpn-gateway-settings/)。

## 站点到站点连接和 VPN 设备
### 选择 VPN 设备时应考虑什么？
我们在与设备供应商合作的过程中验证了一系列的标准站点到站点 VPN 设备。可在[此处](/documentation/articles/vpn-gateway-about-vpn-devices/)找到已知兼容的 VPN 设备及其相应的配置说明/示例和设备规范的列表。设备系列中列为已知兼容设备的所有设备都应适用于虚拟网络。若要获取配置 VPN 设备的帮助，请参考对应于相应设备系列的设备配置示例或链接。

### 如果已知兼容设备的列表中没有 VPN 设备，该怎么办？
如果没有看到设备列出作为已知兼容 VPN 设备，但想使用该设备进行 VPN 连接，则需要确认它符合[此处](/documentation/articles/vpn-gateway-about-vpn-devices/)列出的系统支持的 IPsec/IKE 配置选项和参数。满足最低要求的设备应该兼容 VPN 网关。请联系设备制造商了解更多支持和配置说明。

### 在流量处于空闲状态时，为何我的基于策略的 VPN 隧道会关闭？
对于基于策略（也称为静态路由）的 VPN 网关来说，这是预期的行为。当经过隧道的流量处于空闲状态 5 分钟以上时，将销毁该隧道。流量朝任一方向开始流动时，该隧道将立即重建。

### 我可以使用软件 VPN 连接到 Azure 吗？
我们支持将 Windows Server 2012 路由和远程访问 (RRAS) 服务器用于站点到站点跨界配置。

其他软件 VPN 解决方案只要遵循行业标准 IPsec 实现，就会与我们的网关兼容。有关配置和支持说明，请与该软件的供应商联系。

## <a name="P2S"></a>点到站点连接

[AZURE.INCLUDE [vpn-gateway-point-to-site-faq-include](../../includes/vpn-gateway-point-to-site-faq-include.md)]

## <a name="V2VMulti"></a>VNet 到 VNet 和多站点连接

[AZURE.INCLUDE [vpn-gateway-vnet-vnet-faq-include](../../includes/vpn-gateway-vnet-vnet-faq-include.md)]

### 是否可以使用 Azure VPN 网关在我的本地站点之间传输流量或将流量传输到其他虚拟网络？

**Resource Manager 部署模型**<br>
是。有关详细信息，请参阅 [BGP](#bgp) 部分。

**经典部署模型**<br>
使用经典部署模型通过 Azure VPN 网关传输流量是可行的，但依赖于网络配置文件中静态定义的地址空间。使用经典部署模型的 Azure 虚拟网络和 VPN 网关尚不支持 BGP。没有 BGP，手动定义传输地址空间很容易出错，不建议这样做。

### Azure 会为同一虚拟网络的所有 VPN 连接生成同一 IPsec/IKE 预共享密钥吗？
否，默认情况下，Azure 会为不同 VPN 连接生成不同的预共享密钥。但是，你可以使用设置 VPN 网关密钥 REST API 或 PowerShell cmdlet 设置你想要的密钥值。该密钥必须是长度介于 1 到 128 个字符之间的字母数字字符串。

### 使用更多站点到站点 VPN 是否会比为单个虚拟网络获取更多带宽？
否，所有 VPN 隧道（包括点到站点 VPN）共享同一 Azure VPN 网关和可用带宽。

### 是否可以使用多站点 VPN 在我的虚拟网络和本地站点之间配置多个隧道？
是，但必须在两个通向同一位置的隧道上配置 BGP。

### 是否可以将点到站点 VPN 用于具有多个 VPN 隧道的虚拟网络？
是，可以将点到站点 (P2S) VPN 用于连接到多个本地站点的 VPN 网关和其他虚拟网络。

### 是否可以将使用 IPsec VPN 的虚拟网络连接到我的 ExpressRoute 线路？
是，系统支持该操作。有关详细信息，请参阅[配置可共存的 ExpressRoute 连接和站点到站点 VPN 连接](/documentation/articles/expressroute-howto-coexist-classic/)。

## <a name="bgp"></a>BGP
[AZURE.INCLUDE [vpn-gateway-bgp-faq-include](../../includes/vpn-gateway-bpg-faq-include.md)]

## 跨界连接和 VM
### 如果虚拟机位于虚拟网络中，而连接是跨界连接，应如何连接到该 VM？
有几个选择。如果你启用了 RDP 并且创建了终结点，则可以使用 VIP 连接到虚拟机。在这种情况下，你需要指定要连接到的 VIP 和端口。你需要配置用于流量的虚拟机端口。通常，可以转到 Azure 经典管理门户，然后保存与计算机建立的 RDP 连接的设置。这些设置包含所需的连接信息。

如果你有一个配置了跨界连接的虚拟网络，则可以通过使用内部 DIP 或专用 IP 地址连接到你的虚拟机。你也可以使用位于同一虚拟网络中的另一个虚拟机的内部 DIP 连接到你的虚拟机。如果要从虚拟网络外部的位置进行连接，则无法使用 DIP RDP 连接到你的虚拟机。例如，如果你配置了点到站点虚拟网络，并且未从计算机建立连接，则无法通过 DIP 连接到虚拟机。

### 如果我的虚拟机位于使用跨界连接的虚拟网络中，从我的 VM 流出的所有流量是否都会经过该连接？
否。只有其目标 IP 包含在指定虚拟网络本地网络 IP 地址范围内的流量才会通过虚拟网络网关。其目标 IP 位于虚拟网络中的流量将保留在虚拟网络中。其他流量通过负载均衡器发送到公共网络，或者在使用强制隧道的情况下通过 Azure VPN 网关发送。如果你要进行故障排除，请务必确保列出本地网络中你要通过网关发送的所有范围。确保本地网络地址范围没有与虚拟网络中的任何地址范围重叠。另外，还需确保所使用的 DNS 服务器会将名称解析成适当的 IP 地址。

## 虚拟网络常见问题
请在[虚拟网络常见问题](/documentation/articles/virtual-networks-faq/)中查看更多虚拟网络信息。

## 后续步骤

* 有关 VPN 网关的详细信息，请参阅[关于 VPN 网关](/documentation/articles/vpn-gateway-about-vpngateways/)。
* 有关 VPN 网关配置设置的详细信息，请参阅[关于 VPN 网关配置设置](/documentation/articles/vpn-gateway-about-vpn-gateway-settings/)。

<!---HONumber=Mooncake_0227_2017-->
<!--Update_Description: remove some questions while adding some others, add "next step"-->