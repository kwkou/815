<properties
   pageTitle="Azure 网络安全最佳实践 | Microsoft Azure"
   description="本文提供一系列有关使用内置 Azure 功能提高网络安全性的最佳实践。"
   services="security"
   documentationCenter="na"
   authors="TomShinder"
   manager="swadhwa"
   editor="TomShinder"/>  


<tags
   ms.service="security"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="na"
   ms.date="05/25/2016"
   wacn.date="10/31/2016"
   ms.author="TomSh"/>  


# Azure 网络安全最佳实践

Azure 可让你将虚拟机和设备放在 Azure 虚拟网络上，从而将它们连接到其他网络设备。Azure 虚拟网络是一种虚拟网络构造，可让你将虚拟网络接口卡连接到虚拟网络，允许有网络功能的设备之间进行基于 TCP/IP 的通信。连接到 Azure 虚拟网络的 Azure 虚拟机能够连接到相同 Azure 虚拟网络、不同 Azure 虚拟网络、Internet 甚至你自己的本地网络上的设备。

本文介绍一系列 Azure 网络安全最佳实践。这些最佳实践衍生自我们的 Azure 网络经验和客户的经验。

对于每项最佳实践，我们将说明：

- 最佳实践是什么
- 为何要启用该最佳实践
- 如果你无法启用该最佳实践，可能的结果是什么
- 最佳实践的可能替代方案
- 如何学习启用最佳实践

这篇 Azure 网络安全最佳实践以 Azure 平台功能和特性集（因为在编写本文时已存在）为基础。看法和技术将随着时间改变，本文将会定期更新以反映这些更改。

本文介绍的 Azure 网络安全最佳实践包括：

- 以逻辑方式分段子网
- 控制路由行为
- 启用强制隧道
- 使用虚拟网络设备
- 部署外围网络进行安全分区
- 避免向具有专用 WAN 链接的 Internet 公开
- 优化运行时间和性能
- 使用全局负载均衡
- 禁用对 Azure 虚拟机的 RDP 访问
- 将数据中心扩展到 Azure


## 以逻辑方式分段子网

[Azure 虚拟网络](/documentation/services/networking/)类似于本地网络上的 LAN。Azure 虚拟网络背后的思路是创建单个基于空间的专用 IP 地址网络，将所有 [Azure 虚拟机](/documentation/services/virtual-machines)置于其上。可用的专用 IP 地址空间位于类别 A (10.0.0.0/8)、类别 B (172.16.0.0/12) 和类别 C (192.168.0.0/16) 范围内。

类似于在本地执行的操作，需要将较大的地址空间分段成子网。可以使用基于 [CIDR](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing) 的子网原理来创建子网。

子网之间的路由将自动发生，不需要手动配置路由表。但是，默认设置是 Azure 虚拟网络上创建的子网之间没有任何网络访问控制。若要创建子网之间的网络访问控制，必须在子网之间添加某项设置。

可用于实现此任务的设置之一是[网络安全组](/documentation/articles/virtual-networks-nsg/) (NSG)。NSG 是简单的有状态数据包检查设备，使用 5 个元组（源 IP、源端口、目标 IP、目标端口和第 4 层协议）的方法来创建网络流量的允许/拒绝规则。你可以允许或拒绝单个 IP 地址、多个 IP 地址甚至整个子网的流量。

将 NSG 用于子网之间的网络访问控制，可将属于同一安全区域或角色的资源置于其本身的子网中。例如，简单的 3 层式应用程序具有 Web 层、应用程序逻辑层和数据库层。可将属于上述各层的虚拟机置于其自身的子网中。然后使用 NSG 来控制子网之间的流量：

- Web 层虚拟机只能发起到应用程序逻辑计算机的连接，并且只可接受来自 Internet 的连接
- 应用程序逻辑虚拟机只能发起与数据库层的连接，并且只可接受来自 Web 层的连接
- 数据库层虚拟机只能发起与其本身子网外部任何组件的连接，并且只可接受来自应用程序逻辑层的连接

若要了解网络安全组以及如何使用它们以逻辑方式分段 Azure 虚拟网络的详细信息，请参阅 [What is a Network Security Group](/documentation/articles/virtual-networks-nsg/)（什么是网络安全组 (NSG)）一文。

## 控制路由行为

当你将虚拟机置于 Azure 虚拟网络时，将会注意到虚拟机可以连接到同一 Azure 虚拟网络上的任何其他虚拟机，即使其他虚拟机位于不同的子网。这种情况的可能原因是默认将启用一些允许这种通信类型的系统路由。这些默认路由可让相同 Azure 虚拟网络上的虚拟机彼此发起连接，以及与 Internet 连接（仅适用于 Internet 的出站通信）。

当默认系统路由适用于许多部署方案时，有时你想要为部署自定义路由配置。这些自定义项可让你配置下一跃点地址以访问特定目标。

建议你在部署虚拟网络安全设备时配置“用户定义的路由”，我们将在后面的最佳实践中进行介绍。

> [AZURE.NOTE] 不需要用户定义的路由，默认系统路由适用于大多数情况。

有关用户定义的路由及其配置方法的详细信息，请阅读 [What are User Defined Routes and IP Forwarding](/documentation/articles/virtual-networks-udr-overview/)（什么是用户定义的路由和 IP 转发）。

## 启用强制隧道

若要进一步了解强制隧道，最好能了解“拆分隧道”是什么。最常见的拆分隧道示例出现在 VPN 连接中。假设你要建立从酒店客房到企业网络的 VPN 连接。这种连接可让你连接到企业网络上的资源，而企业网络上的资源的所有通信都将经过 VPN 隧道。

连接到 Internet 上的资源时会发生什么情况？ 启用拆分隧道后，这些连接将直接连接到 Internet，而不会经过 VPN 隧道。某些安全专家将此视为潜在的风险，因此建议禁用拆分隧道，而以 Internet 为目标和以企业资源为目标的所有连接都经过 VPN 隧道。这样做的好处是对 Internet 的连接将被迫通过企业网络安全设备，而如果 VPN 客户端连接到 VPN 隧道之外的 Internet，将不会发生这种情况。

现在让我们回到 Azure 虚拟网络上的虚拟机。Azure 虚拟网络的默认路由可让虚拟机发起向 Internet 的流量。这也可能造成安全风险，因为这些出站连接可能增加虚拟机的受攻击面并遭到攻击者使用。出于此原因，当 Azure 虚拟网络与本地网络之间有跨界连接时，建议在虚拟机上启用强制隧道。我们稍后将在这份 Azure 网络最佳实践文档中介绍跨界连接。

如果没有跨界连接，请务必使用网络安全组（前面已介绍）或 Azure 虚拟网络安全设备（接下来将介绍）来防止从 Azure 虚拟机对 Internet 的出站连接。

若要详细了解强制隧道及其启用方式，请阅读 [Configure Forced Tunneling using PowerShell and Azure Resource Manager](/documentation/articles/vpn-gateway-forced-tunneling-rm/)（使用 PowerShell 和 Azure Resource Manager 配置强制隧道）一文。

## 使用虚拟网络设备

尽管网络安全组和用户定义路由可以在 [OSI 模型](https://en.wikipedia.org/wiki/OSI_model)的网络和传输层提供特定网络安全度量，但在某些情况下，用户想要或需要启用高堆栈级别的安全性。在此类情况下，建议部署 Azure 合作伙伴所提供的虚拟网络安全设备。

Azure 网络安全设备可通过网络级别控件提供的功能来提供大幅增强的安全级别。虚拟网络安全设备提供的网络安全功能包括：

- 防火墙
- 入侵检测/入侵防护
- 漏洞管理
- 应用程序控制
- 基于网络的异常检测
- Web 筛选
- 防病毒
- 僵尸网络防护

如果需要的网络安全级别高于你可以使用网络级别访问控件获取的级别，则建议调查并部署 Azure 虚拟网络安全设备。

若要了解有哪些可用的 Azure 虚拟网络安全设备及其相关功能，请访问 [Azure 映像应用商店](https://market.azure.cn/List/Index?sort=Featured&filters=tag:security)并搜索“安全”和“网络安全”。

##部署外围网络进行安全分区
外围网络或“周边网络”是物理或逻辑网络区段，用于在资产与 Internet 之间提供额外的安全级别。外围网络的目的是要将专用网络访问控制设备放在外围网络网络的边缘，只允许所需的流量通过网络安全设备进入 Azure 虚拟网络。

外围网络非常有用，因为你可以将网络访问控制管理、监视、日志记录和报告的重点放在位于 Azure 虚拟网络边缘的设备上。在此通常将启用 DDoS 预防、入侵检测/入侵防护系统 (IDS/IPS)、防火墙规则和策略、Web 筛选、网络反恶意软件等。网络安全设备位于 Internet 与 Azure 虚拟网络之间，具有两个网络均适用的接口。

尽管这是外围网络的基本设计，但有许多不同的外围网络设计，例如背靠背式、三闸式、多闸式等等。

建议针对所有高安全性部署，考虑部署外围网络以增强 Azure 资源的网络安全级别。


## 避免向具有专用 WAN 链接的 Internet 公开
许多组织选择了混合 IT 路由。在混合 IT 中，有些企业的信息资产是在 Azure 中，而有些企业则维持在本地。在许多情况下，服务的某些组件将在 Azure 中运行，而其他组件则维持在本地。

在混合 IT 方案中，通常有某种类型的跨界连接。这种跨界连接可让公司将其本地网络连接到 Azure 虚拟网络。可用的跨界连接解决方案有两种：

- 站点到站点 VPN
- ExpressRoute

[站点到站点 VPN](/documentation/articles/vpn-gateway-site-to-site-create/) 代表本地网络和 Azure 虚拟网络之间的虚拟专用连接。此连接通过 Internet 进行，可让你在网络与 Azure 之间的加密链接内“输送”信息。站点到站点 VPN 是成熟的安全技术，各种规模的企业已部署数十年。使用 [IPsec 隧道模式](https://technet.microsoft.com/zh-cn/library/cc786385.aspx)可执行隧道加密。

尽管站点到站点 VPN 是可信、可靠且经过证实的技术，但隧道中的流量将遍历 Internet。此外，最大带宽相对受限于大约 200Mbps。

如果你希望跨界连接具有突出的安全或性能级别，建议对跨界连接使用 Azure ExpressRoute。ExpressRoute 是你本地位置与 Exchange 托管提供商之间专用的 WAN 链接。因为这是电信运营商连接，所以数据不会通过 Internet 传输，也不会出现 Internet 通信固有的潜在风险。

若要详细了解 Azure ExpressRoute 的工作原理和部署方式，请阅读 [ExpressRoute Technical Overview](/documentation/articles/expressroute-introduction/)（ExpressRoute 技术概述）一文。

## 优化运行时间和性能
机密性、完整性和可用性 (CIA) 三者构成现今最具影响力的安全模型。机密性与加密和隐私相关，完整与确保数据不会遭到未经授权人员的更改有关，可用性与确保经过授权的个人可以访问他们有权访问的信息有关。上述任何一个方面失败都代表可能破坏安全性。

可用性可被视为与运行时间和性能有关。如果服务已关闭，便无法访问信息。如果性能不佳，以致数据无法使用，我们可将此数据视为不可访问。因此，从安全角度来看，我们需要尽可能确保服务有最佳的运行时间和性能。用于增强可用性和性能的常用且有效的方法是使用负载均衡。负载均衡是将网络流量分布于服务中各服务器的方法。例如，如果服务中有前端 Web 服务器，可以使用负载均衡将流量分布于多台前端 Web 服务器。

这种流量分布将提高可用性，因为如果其中一台 Web 服务器不可用，负载均衡器将会停止将流量发送到该服务器，并将流量重定向到仍在运行的服务器。负载均衡还有助于性能，因为处理请求的处理器、网络和内存开销将分布于所有负载均衡的服务器之间。

建议尽可能为服务采用适当的负载均衡。我们将在以下部分中探讨适当性。在 Azure 虚拟网络级别，Azure 将提供三个主要负载均衡选项：

- 基于 HTTP 的负载均衡
- 外部负载均衡
- 内部负载均衡

## 基于 HTTP 的负载均衡
基于 HTTP 的负载均衡使用 HTTP 协议的特征确定由哪一台服务器发送连接。Azure 提供以应用程序网关名称为名的 HTTP 负载均衡器。

建议在以下情况时使用 Azure 应用程序网关：

- 需要使用来自同一用户/客户端会话的请求来访问相同后端虚拟机的应用程序。此类应用程序的示例包括购物车应用程序和 Web 邮件服务器。
- 希望使用应用程序网关的 [SSL 卸载](https://f5.com/glossary/ssl-offloading)功能消除 Web 服务器场的 SSL 终端开销的应用程序。
- 要求长时间运行的同一 TCP 连接上多个 HTTP 请求路由到或负载均衡到不同后端服务器的应用程序（例如内容传送网络）。

若要详细了解 Azure 应用程序网关的工作原理及其在部署中的使用方式，请阅读 [Application Gateway Overview](/documentation/articles/application-gateway-introduction/)（应用程序网关概述）一文。

## 外部负载均衡
来自 Internet 的传入连接在位于 Azure 虚拟网络中的服务器之间达到平衡负载时，将进行外部负载均衡。Azure 外部负载均衡器可以提供此功能，建议在不需要粘性会话或 SSL 卸载时使用。

相比基于 HTTP 的负载均衡，外部负载均衡器将使用 OSI 网络模型的网络和传输层信息来确定哪一台服务器可平衡连接负载。

建议每当[无状态应用程序](http://whatis.techtarget.com/definition/stateless-app)接受来自 Internet 的传入请求时，使用外部负载均衡。

若要详细了解 Azure 外部负载均衡器的工作原理和部署方式，请阅读 [Get Started Creating an Internet Facing Load Balancer in Resource Manager using PowerShell](/documentation/articles/load-balancer-get-started-internet-arm-ps/)（开始使用 PowerShell 在 Resource Manager 中创建面向 Internet 的负载均衡器）一文。

## 内部负载均衡
内部负载均衡类似于外部负载均衡，使用相同的机制对其背后服务器的连接进行负载均衡。唯一的差别在于，在此情况下的负载均衡器将接受来自不在 Internet 上的虚拟机的连接。在大多数情况下，Azure 虚拟网络上的设备将发起负载均衡可接受的连接。

建议将内部负载均衡用于将受益于此功能的方案，例如当需要对 SQL 服务器或内部 Web 服务器的连接进行负载均衡时。

若要详细了解 Azure 内部负载均衡器的工作原理和部署方式，请阅读 [Get Started Creating an Internal Load Balancer using PowerShell](/documentation/articles/load-balancer-get-started-internet-arm-ps/#update-an-existing-load-balancer)（开始使用 PowerShell 创建 Internet 负载均衡器）一文。

## 使用全局负载均衡
使用公有云计算可部署遍布全球的应用程序，其组件位于世界各地的数据中心。由于 Azure 有全局数据中心，因此这种方案在 Azure 上可行。相比于前面提到的负载均衡技术，全局负载均衡可让服务即使在整个数据中心可能不可用时也能使用。

可以使用 [Azure 流量管理器](/documentation/services/traffic-manager/)在 Azure 中实现这种类型的全局负载均衡。流量管理器可以根据用户的位置，对服务的连接进行负载均衡。

例如，如果用户从欧盟对服务发出请求，此连接将被定向到位于欧盟数据中心的服务。这一部分的流量管理器全局负载均衡有助于改善性能，因为连接到最近的数据中心比连接到远处的数据中心还要快。

在可用性方面，全局负载均衡可确保即使整个数据中心不可用，服务仍可使用。

例如，如果 Azure 数据中心因为环境原因或服务中断（例如区域网络故障）而不可用，对服务的连接将被重新路由到最近的在线数据中心。使用可在流量管理器中创建的 DNS 策略来实现这种全局负载均衡。

如果所开发的云解决方案广泛分布于多个区域且需要最高级别的运行时间，我们建议使用流量管理器。

若要详细了解 Azure 流量管理器及其部署方式，请阅读 [What is Traffic Manager](/documentation/articles/traffic-manager-overview/)（什么是流量管理器）一文。

## 禁用对 Azure 虚拟机的 RDP/SSH 访问
使用[远程桌面协议](https://en.wikipedia.org/wiki/Remote_Desktop_Protocol) (RDP) 和[安全 Shell](https://en.wikipedia.org/wiki/Secure_Shell) (SSH) 协议可以访问 Azure 虚拟机。这些协议可用于从远程位置管理虚拟机，并且是数据中心计算的标准。

在 Internet 上使用这些协议的潜在安全问题是，攻击者可以使用各种[暴力破解](https://en.wikipedia.org/wiki/Brute-force_attack)技术来访问 Azure 虚拟机。攻击者一旦获取访问权限，就可以使用虚拟机作为破坏 Azure 虚拟网络上其他计算机的启动点，甚至攻击 Azure 之外的网络设备。

因此，我们建议禁用从 Internet 对 Azure 虚拟机的直接 RDP 和 SSH 访问。禁用从 Internet 的直接 RDP 和 SSH 访问之后，有其他选项可用于访问这些虚拟机以便进行远程管理：

- 点到站点 VPN
- 站点到站点 VPN
- ExpressRoute

[点到站点 VPN](/documentation/articles/vpn-gateway-point-to-site-create/) 是远程访问 VPN 客户端/服务器连接的另一种说法。点到站点 VPN 可让单个用户通过 Internet 连接到 Azure 虚拟网络。建立点到站点连接之后，用户能够使用 RDP 或 SSH 连接到位于用户通过点到站点 VPN 连接的 Azure 虚拟网络上的所有虚拟机。此处假设用户有权访问这些虚拟机。

点到站点 VPN 比直接 RDP 或 SSH 连接更安全，因为用户必须事先通过两次身份验证才将连接到虚拟机。第一次，用户需要通过身份验证（并获得授权）才能创建点到站点 VPN 连接；第二次，用户需要通过身份验证（并获得授权）才能建立 RDP 或 SSH 会话。

[站点到站点 VPN](/documentation/articles/vpn-gateway-site-to-site-create/) 通过 Internet 将整个网络连接到另一个网络。可以使用站点到站点 VPN 将本地网络连接到 Azure 虚拟网络。如果部署站点到站点 VPN，本地网络上的用户能够通过站点到站点 VPN 连接、使用 RDP 或 SSH 协议来连接到 Azure 虚拟网络上的虚拟机，而不需要你允许通过 Internet 的直接 RDP 或 SSH 访问。

也可以使用专用的 WAN 链接提供类似于站点到站点 VPN 的功能。主要差异在于：1. 专用 WAN 链接不会遍历 Internet，2. 专用 WAN 链接通常更加稳定且性能更好。Azure 提供 [ExpressRoute](/documentation/services/expressroute/) 形式的专用 WAN 链接解决方案。

## 安全地将数据中心扩展到 Azure
许多企业 IT 组织都希望扩展到云中，而不是扩建其本地数据中心。这种扩展意味着要将现有 IT 基础结构扩展到公有云。使用跨界连接选项，可将 Azure 虚拟网络视为本地网络基础结构上的另一个子网。

但是，必须先解决许多规划和设计问题。这一点在网络安全方面格外重要。查看示例是了解如何着手处理这种设计的最佳方式之一。

世纪互联创建了[数据中心扩展参考体系结构关系图](https://gallery.technet.microsoft.com/Datacenter-extension-687b1d84#content)和相关支持材料，以帮助用户了解此类数据中心扩展的形式。其中提供了示例参考实现，可用于规划和设计如何安全地将企业数据中心扩展到云。我们建议你参阅此文档，以大致了解安全解决方案的重要组件。

若要详细了解如何安全地将数据中心扩展到 Azure，请观看视频 [Extending Your Datacenter to Microsoft Azure](https://channel9.msdn.com/Blogs/Windows-Azure/Extending-Your-Datacenter-into-Microsoft-Azure)（将数据中心扩展到 Azure）。

<!---HONumber=Mooncake_1024_2016-->