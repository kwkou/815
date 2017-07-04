> [AZURE.SELECTOR]
- [Portal](/documentation/articles/virtual-network-multiple-ip-addresses-portal/)
- [PowerShell](/documentation/articles/virtual-network-multiple-ip-addresses-powershell/)
- [CLI 2.0](/documentation/articles/virtual-network-multiple-ip-addresses-cli/)
- [CLI 1.0](/documentation/articles/virtual-network-multiple-ip-addresses-cli-nodejs/)
- [模板](/documentation/articles/virtual-network-multiple-ip-addresses-template/)

在一个 Azure 虚拟机 (VM) 上可以附加一个或多个网络接口 (NIC)。 可为任何 NIC 分配一个或多个静态/动态的公共和专用 IP 地址。 将多个 IP 地址分配给 VM 可实现以下功能：

* 在一台服务器上托管具有不同 IP 地址和 SSL 证书的多个网站或服务。
* 用作网络虚拟设备，例如防火墙或负载均衡器。
* 能够将任何 NIC 的任意专用 IP 地址添加到 Azure 负载均衡器后端池。 过去，只有主 NIC 的主 IP 地址才能添加到后端池。 若要详细了解如何对多个 IP 配置进行负载均衡，请阅读[对多个 IP 配置进行负载均衡](/documentation/articles/load-balancer-multiple-ip/)一文。

每个附加到 VM 的 NIC 都具有一个或多个与之关联的 IP 配置。 系统会为每个配置分配一个静态或动态专用 IP 地址。 每个配置还可拥有一个与之关联的公共 IP 地址资源。 系统会为公共 IP 地址资源分配一个动态或静态公共 IP 地址。 若要详细了解 Azure 中的 IP 地址，请阅读 [Azure 中的 IP 地址](/documentation/articles/virtual-network-ip-addresses-overview-arm/)一文。 

分配给 NIC 的专用 IP 地址数目存在限制。 能够在 Azure 订阅中使用的公共 IP 地址数也存在限制。 有关详细信息，请参阅 [Azure 限制](/documentation/articles/azure-subscription-service-limits/#azure-resource-manager-virtual-networking-limits)一文。