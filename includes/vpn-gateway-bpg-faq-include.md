### BGP 是否在所有 Azure VPN 网关 SKU 上受支持？

否，BGP 在 Azure **标准** VPN 网关和**高性能** VPN 网关上受支持 。不支持**基本** SKU。

### 能否将 BGP 用于 Azure 基于策略的 VPN 网关？

否，只有基于路由的 VPN 网关支持 BGP。

### 能否使用专用 ASN（自治系统编号）？

能，你可以将自己的公共 ASN 或专用 ASN 同时用于本地网络和 Azure 虚拟网络。

### 能否将同一个 ASN 同时用于本地 VPN 网络和 Azure VNet？

否，必须在本地网络和 Azure VNet 之间分配不同 ASN（如果你要使用 BGP 将它们连接在一起）。无论是否为跨界连接启用了 BGP，都会为 Azure VPN 网关分配默认 ASN（即 65515）。可以通过在创建 VPN 网关时分配不同 ASN，或者在创建网关后更改 ASN 来覆盖此默认值。你需要将本地 ASN 分配给相应的 Azure 本地网关。



### Azure VPN 网关将播发给我哪些地址前缀？

Azure VPN 网关会将以下路由播发到本地 BGP 设备：

- VNet 地址前缀
- 已连接到 Azure VPN 网关的每个本地网关的地址前缀
- 从连接到 Azure VPN 网关的其他 BGP 对等会话获知的路由，**默认路由或与任何 VNet 前缀重叠的路由除外**。

### 能否将 BGP 用于 VNet 到 VNet 连接？

能，可以将 BGP 同时用于跨界连接和 VNet 到 VNet 连接。

### 能否将 BGP 连接与非 BGP 连接混合用于 Azure VPN 网关？

能，你可以将 BGP 连接和非 BGP 连接混合用于同一 Azure VPN 网关。

### Azure VPN 网关是否支持 BGP 传输路由？

是，支持 BGP 传输路由，但例外是 Azure VPN 网关**不**会将默认路由播发到其他 BGP 对等节点。若要启用跨多个 Azure VPN 网关的传输路由，必须在所有中间 VNet 到 VNet 连接上启用 BGP。

### 在 Azure VPN 网关和我的本地网络之间能否有多个隧道？

能，你可以在 Azure VPN 网关和本地网络之间建立多个 S2S VPN 隧道。请注意，所有这些隧道都将计入 Azure VPN 网关的隧道总数。例如，如果你在 Azure VPN 网关与一个本地网络之间有两个冗余隧道，它们将占用 Azure VPN 网关的总配额（标准为 10 个，高性能为 30 个）中的 2 个隧道。

### 在两个使用 BGP 的 Azure VNet 之间能否有多个隧道？

否，不支持在一对虚拟网络之间存在冗余隧道。

### 能否在 ExpressRoute/S2S VPN 共存配置中对 S2S VPN 使用 BGP？

现在不行。

### Azure VPN 网关将哪个地址用于 BGP 对等节点 IP？

Azure VPN 网关将从为虚拟网络定义的 GatewaySubnet 范围内分配单个 IP 地址。默认情况下，它是该范围的倒数第二个地址。例如，如果你的 GatewaySubnet 是 10.12.255.0.0/27（范围从 10.42.255.0.0 到 10.42.255.31），则 Azure VPN 网关上的 BGP 对等节点 IP 地址将是 10.12.255.30。当列出 Azure VPN 网关信息时，可以找到此信息。

### VPN 设备上的 BGP 对等节点 IP 地址的要求是什么？

本地 BGP 对等节点地址**不能**与 VPN 设备的公共 IP 地址相同。在 VPN 设备上对 BGP 对等节点 IP 使用不同的 IP 地址。它可以是分配给该设备上环回接口的地址。在表示该位置的相应本地网关中指定此地址。

### 使用 BGP 时应将什么指定为本地网关的地址前缀？

Azure 本地网关为本地网络指定初始地址前缀。使用 BGP 时，必须分配 BGP 对等节点 IP 地址的主机前缀（/32 前缀）作为本地网络的地址空间。如果你的 BGP 对等节点 IP 为 10.52.255.254，则应指定“10.52.255.254/32”作为表示此本地网络的本地网关的 localNetworkAddressSpace。这是为了确保 Azure VPN 网关通过 S2S VPN 隧道建立 BGP 会话。

### 应为 BGP 对等会话添加到本地 VPN 设备什么内容？

你应在指向 IPsec S2S VPN 隧道的 VPN 设备上添加 Azure BGP 对等节点 IP 地址的主机路由。例如，如果 Azure VPN 对等节点 IP 为“10.12.255.30”，则应在 VPN 设备上添加“10.12.255.30”的主机路由（包含匹配的 IPsec 隧道接口的下一跃点接口）。

<!---HONumber=Mooncake_0613_2016-->
