###Azure 会对 VNet 之间的流量收费吗？
同一区域中的 VNet 到 VNet 流量两个方向都是免费的。跨区域 VNet 到 VNet 出口流量根据源区域的出站 VNet 间数据传输费率收费。有关详细信息，请参阅[定价页](/pricing/details/vpn-gateway/)。

###VNet 到 VNet 流量是否会通过 Internet？
否。VNet 到 VNet 流量会流经 Azure 主干，而非 Internet。

### VNet 到 VNet 流量是否安全？
是，它通过 IPsec/IKE 加密进行保护。

###是否需要使用 VPN 设备将 VNet 连接在一起？
否。将多个 Azure 虚拟网络连接在一起不需要 VPN 设备，除非需要跨界连接。

###VNet 是否需要处于同一区域？
否。虚拟网络可以在相同或不同的 Azure 区域（位置）中。

###是否可以将 VNet 到 VNet 与多站点连接一起使用？
是的。虚拟网络连接可与多站点 VPN 同时使用。

### 一个虚拟网络可以连接到多少个本地站点和虚拟网络？
请参阅[网关要求](/documentation/articles/vpn-gateway-about-vpn-gateway-settings/#requirements)表。

###是否可以使用 VNet 到 VNet 连接 VNet 外的 VM 或云服务？
否。VNet 到 VNet 支持连接虚拟网络。它不支持连接不在虚拟网络中的虚拟机或云服务。

###云服务或负载均衡终结点可否跨 VNet？
否。云服务或负载均衡终结点不能跨虚拟网络，即使它们连接在一起，也是如此。

###是否可以为 VNet 到 VNet 或多站点连接使用 PolicyBased VPN 类型？
否。VNet 到 VNet 和多站点连接需要 RouteBased（以前称为动态路由）VPN 类型的 Azure VPN 网关。

### 是否可以将 RouteBased VPN 类型的 VNet 连接到另一个 PolicyBased VPN 类型的 VNet？
否，两台虚拟网络都必须使用基于路由（以前称为动态路由）的 VPN。

###VPN 隧道是否共享带宽？
是的。虚拟网络的所有 VPN 隧道共享 Azure VPN 网关上的可用带宽，以及 Azure 中的相同 VPN 网关运行时间 SLA。

###是否支持冗余隧道？
将一个虚拟网络网关配置为主动-主动时，支持在一对虚拟网络之间设置冗余隧道。

###VNet 到 VNet 配置是否可以有重叠的地址空间？
否。不能有重叠的 IP 地址范围。

### 连接的虚拟网络与内部本地站点之间是否可以有重叠的地址空间？
否。不能有重叠的 IP 地址范围。

<!---HONumber=Mooncake_0227_2017-->