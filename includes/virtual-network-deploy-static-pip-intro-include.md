可以在 Azure 中创建虚拟机 (VM)，然后通过使用公共 IP 地址向公共 Internet 公开这些虚拟机。默认情况下，公共 IP 是动态的，并且在删除 VM 时，关联到它们的地址可能会更改。若要确保 VM 始终使用同一公共 IP 地址，需要创建一个静态公共 IP。

在 VM 中实现静态公共 IP 之前，有必要了解何时可以使用静态公共 IP，以及如何使用它们。请阅读 [IP 寻址概述](/documentation/articles/virtual-network-ip-addresses-overview-classic/)以了解有关 Azure 中 IP 寻址的详细信息。

<!---HONumber=Mooncake_0215_2016-->