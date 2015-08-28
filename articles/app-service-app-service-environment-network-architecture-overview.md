<properties 
	pageTitle="App Service 环境的网络体系结构概述" 
	description="App Service 网络拓扑的体系结构概述。" 
	services="app-service\web" 
	documentationCenter="" 
	authors="stefsch" 
	manager="wpickett" 
	editor=""/>

<tags 
	ms.service="app-service-web" 
	ms.date="07/06/2015" 
	wacn.date=""/>	

# App Service 环境的网络体系结构概述

## 介绍 ##
App Service 环境始终创建于[虚拟网络][virtualnetwork]的子网内，在 App Service 环境中运行的应用可与相同虚拟网络拓扑中的专用终结点通信。由于客户可能锁定了其虚拟网络基础结构的组件，因此请务必了解与 App Service 环境发生的网络通信流类型。

## 常规网络流 ##
 
App Service 环境始终具有一个公共虚拟 IP 地址 (VIP)。所有入站流量都会到达该公共 VIP（包括应用的 HTTP 和 HTTPS 流量，以及 FTP、远程调试功能和 Azure 管理操作的其他流量）。有关公共 VIP 上可用特定端口（必需和可选）的完整列表，请参阅有关[控制发往 App Service 环境的入站流量][controllinginboundtraffic]的文章。

下图显示了各种入站和出站网络流的概览：

![常规网络流][GeneralNetworkFlows]

App Service 环境可与各种专用客户终结点进行通信。例如，在 App Service 环境中运行的应用可以连接到在相同虚拟网络拓扑中的 IaaS 虚拟机上运行的数据库服务器。

App Service 环境还能与管理和操作 App Service 环境所需的 Sql DB 与 Azure 存储空间资源进行通信。与 App Service 环境通信的一些 SQL 和存储资源位于与 App Service 环境相同的区域中，有些则位于远程 Azure 区域中。因此，只有与 Internet 建立了出站连接，App Service 环境才能正常工作。

由于 App Service 环境是在子网中部署的，因此可以使用网络安全组来控制发往子网的入站流量。有关如何控制发往 App Service 环境的入站流量的详细信息，请参阅以下[文章][controllinginboundtraffic]。

有关如何允许来自 App Service 环境的出站 Internet 连接的详细信息，请参阅以下有关使用 [Express Route][ExpressRoute] 的文章。此文章所述的方法同样适用于使用站点到站点连接以及使用强制隧道的情况。

## 出站网络地址 ##
App Service 环境执行出站调用时，IP 地址始终与出站调用相关联。使用的特定 IP 地址取决于所调用的终结点是位于虚拟网络拓扑内部还是外部。

如果调用的终结点在虚拟网络拓扑**外部**，则使用的出站地址（也称为出站 NAT 地址）是 App Service 环境的公共 VIP。你可以在 App Service 环境的门户用户界面中找到此地址（注：UX 制作中）。

也可以通过在 App Service 环境中创建一个应用，然后对该应用的地址执行 *nslookup*，来确定此地址。最终的 IP 地址既是公共 VIP，也是 App Service 环境的出站 NAT 地址。

如果调用的终结点在虚拟网络拓扑**内部**，则调用端应用的出站地址是运行应用的单个计算资源的内部 IP 地址。但是，虚拟网络内部 IP 地址与应用之间不存在持久性的映射。应用可以在不同的计算资源之间移动，并且可以基于缩放操作更改 App Service 环境中的可用计算资源池。

但是，由于 App Service 环境始终位在子网内，可以保证运行应用的计算资源的内部 IP 地址始终处于子网的 CIDR 范围内。因此，使用精细 ACL 或网络安全组来保护虚拟网络中其他终结点的访问时，需要将访问权限授予包含 App Service 环境的子网范围。

下图更详细地演示了这些概念：

![出站网络地址][OutboundNetworkAddresses]

在上图中：

- 由于 App Service 环境的公共 VIP 是 192.23.1.2，因此这是调用“Internet”终结点时使用的出站 IP 地址。
- App Service 环境的包含子网的 CIDR 范围是 10.0.1.0/26。同一虚拟网络基础结构中的其他终结点将看到源自此地址范围内某个应用的调用。

## 其他链接和信息 ##
[此处][controllinginboundtraffic]提供了有关 App Service 环境使用的入站端口以及使用网络安全组控制入站流量的详细信息。

此[文章][ExpressRoute]介绍了有关使用用户定义路由来授予对 App Service 环境的出站 Internet 访问权限的详细信息。


<!-- LINKS -->
[virtualnetwork]: /documentation/services/virtual-network
[controllinginboundtraffic]: /documentation/articles/app-service-app-service-environment-control-inbound-traffic
[ExpressRoute]: /documentation/articles/app-service-app-service-environment-network-configuration-expressroute

<!-- IMAGES -->
[GeneralNetworkFlows]: ./media/app-service-app-service-environment-network-architecture-overview/NetworkOverview-1.png
[OutboundNetworkAddresses]: ./media/app-service-app-service-environment-network-architecture-overview/OutboundNetworkAddresses-1.png

<!---HONumber=66-->