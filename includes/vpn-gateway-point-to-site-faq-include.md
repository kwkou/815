### <a name="what-operating-systems-can-i-use-with-point-to-site"></a>点到站点连接允许使用哪些操作系统？
支持以下操作系统：

* Windows 7（32 位和 64 位）
* Windows Server 2008 R2（仅 64 位）
* Windows 8（32 位和 64 位）
* Windows 8.1（32 位和 64 位）
* Windows Server 2012（仅 64 位）
* Windows Server 2012 R2（仅 64 位）
* Windows 10

### <a name="can-i-use-any-software-vpn-client-for-point-to-site-that-supports-sstp"></a>是否可以使用任何支持 SSTP 的点到站点软件 VPN 客户端？
否。 仅支持上面列出的 Windows 操作系统版本。

### <a name="how-many-vpn-client-endpoints-can-i-have-in-my-point-to-site-configuration"></a>在我的点到站点配置中，可以有多少 VPN 客户端终结点？
我们最多支持 128 个 VPN 客户端可同时连接到一个虚拟网络。

### <a name="can-i-use-my-own-internal-pki-root-ca-for-point-to-site-connectivity"></a>是否可以将我自己的内部 PKI 根 CA 用于点到站点连接？
是。 以前只可使用自签名根证书。 你仍可上传 20 个根证书。

### <a name="can-i-traverse-proxies-and-firewalls-using-point-to-site-capability"></a>是否可以使用点到站点功能穿越代理和防火墙？
是。 我们使用 SSTP（安全套接字隧道协议）作为隧道穿越防火墙。 此隧道将显示为 HTTPS 连接。

### <a name="if-i-restart-a-client-computer-configured-for-point-to-site-will-the-vpn-automatically-reconnect"></a>如果重新启动进行过点到站点配置的客户端计算机，是否会自动重新连接 VPN？
默认情况下，客户端计算机将不自动重新建立 VPN 连接。

### <a name="does-point-to-site-support-auto-reconnect-and-ddns-on-the-vpn-clients"></a>点到站点在 VPN 客户端上是否支持自动重新连接和 DDNS？
点到站点 VPN 中当前不支持自动重新连接和 DDNS。

### <a name="can-i-have-site-to-site-and-point-to-site-configurations-coexist-for-the-same-virtual-network"></a>对于同一虚拟网络，站点到站点和点到站点配置能否共存？
是。 如果为网关使用 RouteBased VPN 类型，这两种解决方案都可行。 对于经典部署模型，需要一个动态网关。 我们不支持对静态路由 VPN 网关或使用 `-VpnType PolicyBased` cmdlet 的网关使用点到站点连接。

### <a name="can-i-configure-a-point-to-site-client-to-connect-to-multiple-virtual-networks-at-the-same-time"></a>是否可以将点到站点客户端配置为同时连接到多个虚拟网络？
是，可以这样做。 但虚拟网络的 IP 前缀不得重叠，并且点到站点地址空间在虚拟网络之间不得重叠。

### <a name="how-much-throughput-can-i-expect-through-site-to-site-or-point-to-site-connections"></a>预计通过站点到站点连接或点到站点连接的吞吐量有多少？
很难维持 VPN 隧道的准确吞吐量。 IPsec 和 SSTP 是重重加密的 VPN 协议。 本地网络与 Internet 之间的延迟和带宽也限制了吞吐量。