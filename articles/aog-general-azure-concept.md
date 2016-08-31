<properties 
	pageTitle="Azure网络相关名词解释" 
	description="Azure网络相关名词解释" 
	services="automation" 
	documentationCenter="" 
	authors=""
	manager="" 
	editor=""/>
<tags ms.service="aog-concept" ms.date="" wacn.date="08/31/2016"/>

# Azure网络相关名词解释
* [5-tuples/5元组](#tuples)
* [Application Gateway/应用程序网关](#application-gateway)
* [Azure SLB/Azure负载平衡器](#azure-slb)
* [DIP/Dynamic IP address，动态IP](#dip)
* [Endpoint/终结点](#endpoint)
* [ExpressRoute](#expressroute)
* [Azure ILB/Azure Internal load balancer/Azure内部负载平衡器](#azure-ilb)
* [ILPIP/Instance-level public IP address/实例级公共IP](#ilpip)
* [NSG/Network Security Group/网络安全组](#nsg)
* [P2S VPN/Point-to-Site VPN/点到站点VPN](#p2s-vpn)
* [S2S VPN/Site-to-Site VPN/站点到站点VPN](#s2s-vpn)
* [Traffic Manager/流量管理器](#traffic-manager)
* [VIP/Virtual IP address/虚拟IP](#vip)
* [VNET/Virtual Network/虚拟网络](#vnet)
* [VPN Gateway/VPN网关](#vpn-gateway)

## <a id="tuples"></a>5-tuples/5元组
5元组，包含源IP、源端口、目标 IP、目标端口和协议类型。它是工作在传输层（OSI的第4层）的Azure负载平衡器（Azure SLB）中的负载平衡模式。通过对5元组的哈希计算，负载平衡器将流量映射到VIP后的可用服务。根据不同的需求，Azure还支持2元组（源 IP、目标 IP）或 3元组（源 IP、目标 IP、协议类型），此操作需要通过PowerShell实现。

[详细介绍](https://www.azure.cn/blog/2015/03/13/azure-load-balancer-new-distribution-mode)


## <a id="application-gateway"></a>Application Gateway/应用程序网关
Azure应用程序网关是应用程序层负载平衡（OSI的第7层）的HTTP负载平衡解决方案。它充当反向代理服务，截获客户端连接，并将请求转发到后端终结点。
应用程序网关当前支持以下第7层应用程序传送功能：HTTP负载平衡，基于Cookie的会话相关性，安全套接字层 (SSL) 卸载，和基于URL的内容路由。

[详细介绍](https://www.azure.cn/documentation/articles/application-gateway-introduction/)

## <a id="azure-slb"></a>Azure SLB/Azure负载平衡器
Azure负载平衡器是一种工作在传输层（OSI的第4层）类型的负载平衡器。它可以将传入的TCP、UDP流量分发到云服务中正常运行的服务实例上，或者分发到负载平衡器集内所定义的虚拟机上。
Azure负载平衡器使用的分发算法是对5元组（或2元组，3元组）进行哈希运算。通过对运算结果的比较，负载平衡器将流量映射到对应的可用服务上。

[详细介绍（英文）](https://azure.microsoft.com/en-us/documentation/articles/load-balancer-overview/)

## <a id="dip"></a>DIP/Dynamic IP address/动态IP
DIP是虚拟机在部署时，Azure通过DHCP分配给虚拟机的内部IP地址。只要虚拟机没有被停机或删除，DIP将一直分配在这台虚拟机上。但是，如果虚拟机被停机或删除，DIP将被释放，并分配给新部署的虚拟机。若虚拟机部署在虚拟网络的子网中，DIP将从虚拟机所在的子网中分配。如果你需要虚拟机使用固定的内部IP，则需要为其设置静态内部专用IP。

[如何设置静态内部专用IP](https://www.azure.cn/documentation/articles/virtual-networks-reserved-private-ip/)

## <a id="endpoint"></a>Endpoint/终结点
终结点包含IP协议和对应的端口。每个终结点都有一个公用端口和一个专用端口。Azure负载平衡器使用公用端口侦听从Internet传入的虚拟机流量。虚拟机使用专用端口侦听通常发送到虚拟机上运行的应用程序或服务的传入流量。
使用Azure经典管理门户创建终结点时，将为 IP 协议和众所周知的网络协议的TCP或UDP端口提供默认值。例如用于远程桌面和Windows PowerShell远程处理的终结点。对于自定义终结点，必须指定正确的IP协议（TCP 或 UDP）以及公用和专用端口。若要将传入流量随机分布到多个虚拟机上，必须创建包含多个终结点的负载平衡集。
请注意，Azure流量管理器（Traffic Manager）和CDN中也有终结点的概念，与此处的终结点不同，请勿混淆。

[在经典Azure虚拟机上设置终结点](https://www.azure.cn/documentation/articles/virtual-machines-windows-classic-setup-endpoints/)

[在经典Azure Linux虚拟机上设置终结点](https://www.azure.cn/zh-cn/documentation/articles/virtual-machines-set-up-endpoints/)

## <a id="expressroute"></a>ExpressRoute
ExpressRoute 是一项 Azure网络相关的服务，允许你在 Azure 数据中心与你的本地环境或第三方托管设施中的基础结构之间创建专用连接。ExpressRoute 连接不通过公共 Internet。与通过 Internet 的典型连接相比，ExpressRoute 连接提供更高的可靠性、更快的速度、更低的延迟和更高的安全。

[详细介绍](https://www.azure.cn/documentation/articles/expressroute-introduction/)

## <a id="azure-ilb"></a>Azure ILB/Azure Internal load balancer/Azure内部负载平衡器
Azure内部负载平衡器是一种面向Azure内部的负载均衡器。只有Azure内部的资源以及接入VPN通道的设备能够访问Azure内部均衡器。所有通过内部负载平衡器的数据流量都不会直接流向公网。Azure内部负载均衡器适用于对安全级别要求较高的负载均衡。例如在Azure内部部署的多层级应用的后端数据库需要负载平衡时，就可以使用Azure内部负载平衡器。对于跨界连接（例如S2S VPN）中的负载均衡需求，Azure内部负载平衡器也能很好的满足。

[详细介绍（英文）](https://azure.microsoft.com/en-us/documentation/articles/load-balancer-internal-overview/)

## <a id="ilpip"></a>ILPIP/Instance-level public IP address/实例级公共IP
实例级公共IP是可直接向VM或角色实例分配的公共 IP 地址。它不是用来代替分配给云服务的VIP（虚拟IP），而是可以用来直接连接到VM或角色实例的IP地址。需要注意的是，ILPIP及其绑定的端口都是直接暴露在公网上的，访问时数据将不再通过云服务和负载均衡器。在使用时，需要格外注意网络安全的配置。

[详细介绍](https://www.azure.cn/documentation/articles/virtual-networks-instance-level-public-ip/)

![ILPIP](media/aog-general-azure-concept/ILPIP.png)
## <a id="nsg"></a>NSG/Network Security Group/网络安全组
网络安全组包含一系列访问控制列表 (ACL) 规则，这些规则可以允许或拒绝虚拟网络中流向 VM 实例的网络流量。NSG 可以与子网或该子网中的各个 VM 实例相关联。当 NSG 与某个子网相关联时，ACL 规则适用于该子网中的所有 VM 实例。另外，可以进一步通过将 NSG 直接关联到单个 VM 对流向该 VM 的流量进行限制。
网络安全组的规则分为入站规则和出站规则，每一条规则都有唯一的优先级。所有的网络安全组都包含一组默认规则，且无法删除，但是它们的优先级最低。

[详细介绍](https://www.azure.cn/documentation/articles/virtual-networks-nsg/)
## <a id="p2s-vpn"></a>P2S VPN/Point-to-Site VPN/点到站点VPN
点到站点连接允许你从任何位置的单台计算机连接到Azure虚拟网络中的任何内容。它使用 Windows 内置的 VPN 客户端。在进行点到站点配置时，你需要安装证书和VPN客户端配置包，其中包含的设置允许你的计算机连接到虚拟网络中的任何虚拟机或角色实例。当你无法访问VPN硬件或面向公网的IPv4地址（二者是进行站点到站点VPN连接所必需的）时，后者仅有少量客户端需要连接到虚拟网络时，点到站点VPN连接是很好的选择。

[使用经典管理门户配置与VNET的点到站点VPN连接](https://www.azure.cn/documentation/articles/vpn-gateway-point-to-site-create/)

[使用PowerShell配置与虚拟网络的点到站点连接](https://www.azure.cn/documentation/articles/vpn-gateway-howto-point-to-site-rm-ps/)

![P2S VPN](media/aog-general-azure-concept/P2S-VPN.png)

## <a id="s2s-vpn"></a>S2S VPN/Site-to-Site VPN/站点到站点VPN
站点到站点VPN用于建立你的本地网络到Azure虚拟网络中的任何虚拟机或角色实例的连接。此类连接依赖于IPsec VPN设备（硬件或软件设备），该设备必须部署在网络边缘（不能位于NAT后面）。若要创建此类连接，你必须具有必需的VPN硬件设备和面向公网的IPv4地址。

[使用Azure经典管理门户创建具有站点到站点VPN连接的虚拟网络](https://www.azure.cn/documentation/articles/vpn-gateway-site-to-site-create/)

[使用Azure门户预览和Azure Resource Manager创建具有站点到站点VPN连接的VNET](https://www.azure.cn/documentation/articles/vpn-gateway-howto-site-to-site-resource-manager-portal/)

[使用PowerShell和Azure资源管理器创建具有站点到站点VPN连接的虚拟网络](https://www.azure.cn/documentation/articles/vpn-gateway-create-site-to-site-rm-powershell/)

[站点到站点 VPN 网关连接 VPN 设备](https://www.azure.cn/documentation/articles/vpn-gateway-about-vpn-devices/)

![S2S VPN](media/aog-general-azure-concept/S2S-VPN.png)
## <a id="traffic-manager"></a>Traffic Manager/流量管理器
Azure流量管理器用于控制用户流量的分布，根据需要将用户流量分布到在全球不同数据中心运行的服务终结点。流量管理器支持的服务终结点包括 Azure VM、Web Apps和云服务。也可将流量管理器用于外部的非Azure终结点。流量管理器在DNS级别工作。它使用DNS响应将客户端引导到相应的服务终结点。然后客户端直接连接到服务终结点，不通过流量管理器进行连接。流量管理器不是代理，它看不到流量在客户端和服务之间传递。更多详细请阅读[这篇文章](https://www.azure.cn/documentation/articles/traffic-manager-overview/)，[这篇文章](https://www.azure.cn/documentation/articles/traffic-manager-how-traffic-manager-works/)是其
工作原理。
## <a id="vip"></a>VIP/Virtual IP address/虚拟IP
虚拟IP是允许你从外部访问在Azure中部署的资源的IP地址。它可以被用于云服务，IaaS虚拟机，PaaS角色实例，和应用程序网关。如果需要为Azure资源分配虚拟IP地址，该IP地址将动态地从资源的创建位置中的可用公共IP地址池中分配。停止该资源时，此IP地址将被释放。对于需要使用固定IP的资源，可以使用静态（保留）IP来实现。

[Azure 中的 IP 地址（经典）](https://www.azure.cn/documentation/articles/virtual-network-ip-addresses-overview-classic)

[保留IP概述](https://www.azure.cn/documentation/articles/virtual-networks-reserved-public-ip/)
## <a id="vnet"></a>VNET/Virtual Network/虚拟网络
Azure 虚拟网络是你自己的网络在云中的表示形式。它是对专用于你的订阅的 Azure 云进行的逻辑隔离。你可以完全控制该网络中的 IP 地址块、DNS 设置、安全策略和路由表。你还可以进一步将VNET细分成各个子网，并启动 Azure IaaS 虚拟机(VM)和/或云服务（PaaS 角色实例）。实际上，你可以将网络扩展到 Azure，对 IP 地址块进行完全的控制，并享受企业级 Azure 带来的好处。更多详细请阅读[这篇文章](https://www.azure.cn/documentation/articles/virtual-networks-overview/)
## <a id="vpn-gateway"></a>VPN Gateway/VPN网关
VPN网关用于建立虚拟网络和本地位置之间的VPN通道，并承载其间的网络流量。VPN网关还用于在Azure的多个虚拟网络之间发送流量（VNET到VNET）。
若要配置VPN网关，需要先在VNET中创建网关子网。根据不同的配置，网关子网可以创建不同的大小。最小的为/29，但是建议创建/28甚至更大的网关。若要配置站点到站点/Express Route并存配置，网关子网必须创建为/28或者更大。
Azure 的VPN有两种类型，分别为基于策略的VPN（PolicyBased）和基于路由的VPN（RouteBased）。根据不同的部署方式，需要选择不同的类型。

[详细介绍](https://www.azure.cn/documentation/articles/vpn-gateway-about-vpngateways/)

[VPN 网关常见问题](https://www.azure.cn/documentation/articles/vpn-gateway-vpn-faq/)








