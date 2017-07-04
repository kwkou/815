- **PolicyBased**：PolicyBased VPN 以前在经典部署模型中称为静态路由网关。基于策略的 VPN 会根据使用本地网络和 Azure VNet 之间的地址前缀的各种组合配置的 IPsec 策略，加密数据包并引导其通过 IPsec 隧道。通常会在 VPN 设备配置中将策略（或流量选择器）定义为访问列表。PolicyBased VPN 类型的值为 *PolicyBased*。使用 PolicyBased VPN 时，请记住下列限制：

	- PolicyBased VPN **仅**可在基本网关 SKU 上使用。此 VPN 类型与其他网关 SKU 不兼容。
	- 如果使用 PolicyBased VPN，可以只有 1 个隧道。
	- 只能将 PolicyBased VPN 用于 S2S 连接且只能用于特定配置。大多数 VPN 网关配置需要 RouteBased VPN。

- **RouteBased**：RouteBased VPN 以前在经典部署模型中称为动态路由网关。RouteBased VPN 使用 IP 转发或路由表中的“路由”将数据包引导到相应的隧道接口中。然后，隧道接口会加密或解密出入隧道的数据包。RouteBased VPN 的策略（或流量选择器）配置为任意到任意（或通配符）。RouteBased VPN 类型的值为 *RouteBased*。

<!---HONumber=Mooncake_1031_2016-->