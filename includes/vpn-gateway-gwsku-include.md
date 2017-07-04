创建虚拟网络网关时，需要指定要使用的网关 SKU。 如果选择更高级的网关 SKU，则将为该网关分配更多的 CPU 和网络带宽，这样使网关能够支持到虚拟网络更高的吞吐量。

VPN 网关可以使用以下 SKU：

* 基本
* 标准
* HighPerformance

VPN 网关不使用 UltraPerformance 网关 SKU。 有关 UltraPerformance SKU 的信息，请参阅 [ExpressRoute](/documentation/articles/expressroute-about-virtual-network-gateways/) 文档。

选择 SKU 时，请考虑以下内容：

* 如果想要使用 PolicyBased VPN 类型，必须使用基本 SKU。 任何其他 SKU 均不支持 PolicyBased VPN（之前称为静态路由）。
* 基本 SKU 不支持 BGP。
* 基本 SKU 不支持 ExpressRoute-VPN 网关共存配置。
* 主动-主动 S2S VPN 网关连接只能在 HighPerformance SKU 上配置。