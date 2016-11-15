<properties
   pageTitle="Microsoft Azure 网络安全概述 | Microsoft Azure"
   description=" 本文旨在帮助读者了解 Microsoft Azure 在网络安全领域提供的功能。其中提供了核心网络安全概念和要求的基本说明，以及有关 Azure 在其中每个领域提供的功能的信息。"
   services="security"
   documentationCenter="na"
   authors="LingChen"
   manager="Shlan"
   editor="Lingche"/>  


<tags
   ms.service="security"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="na"
   ms.date="08/09/2016"
   wacn.date="10/31/2016"
   ms.author="lingche"/>  


# Azure 网络安全概述

Azure 提供稳健的网络基础结构来支持应用程序和服务连接要求。在 Azure 中的资源之间、本地资源与 Azure 托管的资源之间，以及 Internet 与 Azure 之间，都可以建立网络连接。

本文旨在帮助读者了解 Azure 在网络安全领域提供的功能，其中提供了核心网络安全概念与要求的基本说明。此外，还提供了有关 Azure 在其中每个领域提供的功能的信息。本文提供了其他内容的链接，帮助读者更深入地了解感兴趣的领域。

Azure 网络安全概述文章侧重于以下内容：

- Azure 网络
- 网络访问控制
- 安全远程访问和跨界连接
- 可用性
- 日志记录
- 名称解析
- 外围网络体系结构

## Azure 网络

虚拟机需要网络连接。为了满足该要求，Azure 需要虚拟机连接到 Azure 虚拟网络。Azure 虚拟网络是构建在物理 Azure 网络结构基础之上的逻辑构造。每个逻辑 Azure 虚拟网络与其他所有 Azure 虚拟网络隔离。这有助于确保其他 Azure 客户无法访问部署中的网络流量。

了解详细信息：

- [虚拟网络概述](/documentation/articles/virtual-networks-overview/)

## 网络访问控制
网络访问控制是指限制与 Azure 虚拟网络内特定设备或子网之间连接的措施。网络访问控制的目标是确保只有有权限的用户和设备才能访问虚拟机和服务。访问控制基于虚拟机或服务之间的连接的允许或拒绝决策。

Azure 支持多种类型的网络访问控制。其中包括：

- 网络层控制
- 路由控制和强制隧道
- 虚拟网络安全设备

### 网络层控制
任何安全部署都需要某种程度的网络访问控制。网络访问控制的目标是确保虚拟机以及在这些虚拟机上运行的网络服务只能与它们需要通信的其他网络设备通信，阻止所有其他连接企图。

如果需要基本网络级别访问控制（基于 IP 地址和 TCP 或 UDP 协议），可以使用网络安全组。网络安全组 (NSG) 是基本的有状态数据包筛选防火墙，使用它可以基于 [5-tuple](https://www.techopedia.com/definition/28190/5-tuple) 来控制访问。NSG 不提供应用程序层检查或经过身份验证的访问控制。

了解详细信息：

- [网络安全组](/documentation/articles/virtual-networks-nsg/)

### 路由控制和强制隧道
控制 Azure 虚拟网络上的路由行为是关键的网络安全和访问控制功能。如果路由配置不当，虚拟机上托管的应用程序和服务可能连接到不允许它们连接的设备 （包括潜在攻击者拥有或操作的设备）。

Azure 网络支持自定义 Azure 虚拟网络上网络流量的路由行为的功能。这样，便可以改变 Azure 虚拟网络中的默认路由表条目。路由行为的控制可帮助确保来自特定设备或一组设备的所有流量都通过特定位置进入或离开 Azure 虚拟网络。

例如，Azure 虚拟网络上可能有虚拟网络安全设备。用户希望确保与 Azure 虚拟网络之间的所有流量都通过该虚拟安全设备。为此，可以在 Azure 中配置[用户定义的路由](/documentation/articles/virtual-networks-udr-overview/)。

[强制隧道](https://www.petri.com/azure-forced-tunneling)是一种机制，可用于确保不允许服务发起与 Internet 上设备的连接。请注意，这与接受然后响应传入连接不同。前端 Web 服务器需要响应来自 Internet 主机的请求，因此允许来自 Internet 的流量进入这些 Web 服务器，并允许 Web 服务器做出响应。

不想要允许的是前端 Web 服务器发起出站请求。此类请求可能表示安全风险，因为这些连接可用于下载恶意代码。即使希望这些前端服务器向 Internet 发起出站请求，还是可能想要强制它们经过本地 Web 代理，以便可以利用 URL 筛选和日志记录。

可以使用强制隧道来避免此问题。启用强制隧道时，将强制所有 Internet 连接通过本地网关。可以利用用户定义的路由来配置强制隧道。

了解详细信息：

- [什么是用户定义的路由和 IP 转发](/documentation/articles/virtual-networks-udr-overview/)

### 虚拟网络安全设备
尽管网络安全组和用户定义路由可以在 [OSI 模型](https://en.wikipedia.org/wiki/OSI_model)的网络和传输层提供某种程度的安全性，但有时，用户想要在高于网络的级别启用安全性。

例如，安全要求可能包括：

- 允许访问应用程序之前进行身份验证和授权
- 入侵检测和入侵响应
- 高级别协议的应用程序层检查
- URL 筛选
- 网络级别的防病毒和反恶意软件
- 防 Bot 保护
- 应用程序访问控制
- 额外的 DDoS 保护（不仅仅是 Azure 结构本身提供的 DDoS 保护）

可以使用 Azure 合作伙伴解决方案来访问这些增强的网络安全功能。可以访问 [Azure 映像应用商店](https://market.azure.cn/List/Index?sort=Featured&filters=tag:security)找到最新的 Azure 合作伙伴网络安全解决方案。

## 安全远程访问和跨界连接
需要远程进行 Azure 资源的安装、配置和管理。此外，建议部署包含本地和 Azure 公有云中组件的[混合 IT](http://social.technet.microsoft.com/wiki/contents/articles/18120.hybrid-cloud-infrastructure-design-considerations.aspx) 解决方案。这些方案需要安全远程访问权限。

Azure 网络支持以下安全远程访问方案：

- 将单个工作站连接到 Azure 虚拟网络
- 使用 VPN 将本地网络连接到 Azure 虚拟网络
- 使用专用 WAN 链路将本地网络连接到 Azure 虚拟网络
- 将 Azure 虚拟网络彼此连接

### 将单个工作站连接到 Azure 虚拟网络
有时可能想要让单个开发人员或操作人员在 Azure 中管理虚拟机和服务。例如，需要访问 Azure 虚拟网络上的虚拟机，但安全策略不允许通过 RDP 或 SSH 远程访问单个虚拟机。在此情况下，可以使用点到站点 VPN 连接。

点到站点 VPN 连接使用 [SSTP VPN](https://technet.microsoft.com/zh-cn/library/cc731352.aspx) 协议，允许设置用户与 Azure 虚拟网络之间的专用安全连接。建立 VPN 连接后，用户可以通过 VPN 链路使用 RDP 或 SSH 连接到 Azure 虚拟网络上的任何虚拟机（假设可对用户进行身份验证和授权）。

了解详细信息：

- [使用 PowerShell 配置与虚拟网络的点到站点连接](/documentation/articles/vpn-gateway-howto-point-to-site-rm-ps/)

### 使用 VPN 将本地网络连接到 Azure 虚拟网络
用户可能想要将整个企业网络或其组成部分连接到 Azure 虚拟网络。这种情况在公司要[将其本地数据中心扩展到 Azure](https://gallery.technet.microsoft.com/Datacenter-extension-687b1d84) 的混合 IT 方案中很常见。在许多情况下，公司会在 Azure 中和本地托管服务的各个组成部分（例如，某个解决方案在 Azure 中包括前端 Web 服务器，在本地包括后端数据库）。使用此类“跨界”连接，还可以更安全地管理 Azure 所在的资源，可以实现将 Active Directory 域控制器扩展到 Azure 等方案。

实现此目的的方法之一是使用[站点到站点 VPN](https://www.techopedia.com/definition/30747/site-to-site-vpn)。站点到站点 VPN 与点到站点 VPN 之间的差别在于，点到站点 VPN 将单个设备连接到 Azure 虚拟网络，而站点到站点 VPN 将整个公司（例如本地网络）连接到 Azure 虚拟网络。Azure 虚拟网络的站点到站点 VPN 使用高度安全的 IPsec 隧道模式 VPN 协议。

了解详细信息：

- [使用 Azure 门户创建具有站点到站点 VPN 连接的 Resource Manager VNet](/documentation/articles/vpn-gateway-howto-site-to-site-resource-manager-portal/)
- [规划和设计 VPN 网关](/documentation/articles/vpn-gateway-plan-design/)

### 使用专用 WAN 链路将本地网络连接到 Azure 虚拟网络
点到站点和站点到站点 VPN 连接可以有效地启用跨界连接。但是，某些组织认为它们存在以下缺点：

- VPN 连接通过 Internet 移动数据 – 在通过公共网络移动数据时，可能会使连接曝露在安全风险之下。此外，无法保证 Internet 连接的可靠性和可用性。
- 对于某些应用程序和用途，与 Azure 虚拟网络建立 VPN 连接可能会限制带宽，因为它们的最大带宽约为 200Mbps。

需要最高安全性和可用性级别进行其跨界连接的组织通常使用专用 WAN 链路连接到远程网站。Azure 允许使用专用 WAN 链路将本地网络连接到 Azure 虚拟网络。这是通过 Azure ExpressRoute 实现的。

了解详细信息：

- [ExpressRoute 技术概述](/documentation/articles/expressroute-introduction/)

### 将 Azure 虚拟网络彼此连接
可以使用多个 Azure 虚拟网络来完成部署。这种做法的原因有很多。其中一个可能的原因是简化管理；另一个可能的原因是安全性。无论将资源放在不同 Azure 虚拟网络的动机或理由是什么，有时都可能需要将每个网络上的资源彼此连接。

一种做法是通过 Internet“环回”，将一个 Azure 虚拟网络上的服务连接到另一个 Azure 虚拟网络上的服务。连接从某个 Azure 虚拟网络开始，经过 Internet，然后返回到目标 Azure 虚拟网络。这种做法会使连接曝露在安全风险之下，这是任何基于 Internet 的通信存在的通病。

更好的做法可能是在 Azure 虚拟网络之间创建站点到站点 VPN。这种 Azure 虚拟网络之间的站点到站点 VPN 使用与上述跨界站点到站点 VPN 连接相同的 [IPsec 隧道模式](https://technet.microsoft.com/zh-cn/library/cc786385.aspx)协议。

使用 Azure 虚拟网络之间的站点到站点 VPN 的优点在于通过 Azure 网络结构创建 VPN 连接，而不是通过 Internet 连接。与通过 Internet 连接的站点到站点 VPN 相比，可以提供额外的安全层。

了解详细信息：

- [使用 Azure Resource Manager 和 PowerShell 配置 VNet 到 VNet 连接](/documentation/articles/vpn-gateway-vnet-vnet-rm-ps/)

## 可用性
可用性是任何安全程序的重要组件。如果用户和系统无法通过网络访问所要访问的内容，则可以将服务视为已遭入侵。Azure 的网络技术支持以下高可用性机制：

- 基于 HTTP 的负载均衡
- 网络级别负载均衡
- 全局负载均衡

负载均衡是一种机制，旨在将连接平均分布到多个设备。负载均衡的目标如下：

- 提高可用性 – 将连接负载均衡到多个设备时，一个或多个设备可能会变得不可用，在剩余联机设备上运行的服务可以继续提供服务。
- 提高性能 – 在多个设备之间负载均衡连接时，单个设备不需要占用处理器。提供内容的处理和内存需求分散在多个设备之间。

### 基于 HTTP 的负载均衡
经常运行 Web 服务的组织想要在这些 Web 服务之前使用 HTTP 负载均衡器，帮助确保提供足够的性能和高可用性级别。与传统的基于网络的负载均衡器相比，HTTP 负载均衡器做出的负载均衡决策基于 HTTP 协议的特征，而不是网络和传输层协议。

为了向基于 Web 的服务提供基于 HTTP 的负载均衡，Azure 提供了 Azure 应用程序网关。Azure 应用程序网关支持：

- 基于 HTTP 的负载均衡 – 负载均衡决策是基于 HTTP 协议的特征做出的
- 基于 Cookie 的会话相关性 – 此功能确保建立到受该负载均衡器保护的服务器之一的连接，在客户端与服务器之间保持不变。这可确保事务的稳定性。
- SSL 卸载 – 使用负载均衡器创建客户端连接时，使用 HTTPS (SSL/) 协议来加密客户端与负载均衡器之间的会话。但是，为了提高性能，可以选择让负载均衡器与受负载均衡器保护的 Web 服务器之间的连接使用 HTTP（未加密）协议。这称为“SSL 卸载”，由于受负载均衡器保护的 Web 服务器没有加密的相关处理器开销，因此应该可以更快地为请求提供服务。
- 基于 URL 的内容路由 – 此功能使负载均衡器可以根据目标 URL 来确定在何处转发连接。它提供的弹性大于基于 IP 地址做出负载均衡决策的解决方案。

了解详细信息：

- [应用程序网关概述](/documentation/articles/application-gateway-introduction/)

### 网络级别负载均衡
与 HTTP 负载均衡相比，网络级别负载均衡可根据 IP 地址和端口号（TCP 或 UDP）做出负载均衡决策。可以使用 Azure 负载均衡器获取 Azure 中网络级别负载均衡的优点。Azure 负载均衡器的一些重要特征包括：

- 基于 IP 地址和端口号的网络级别负载均衡
- 支持任何应用程序层协议
- 对 Azure 虚拟机和云服务角色实例进行负载均衡
- 可用于面向 Internet（外部负载均衡）与不面向 Internet（内部负载均衡）的应用程序和虚拟机
- 终结点监视，用于确定是否有任何受负载均衡器保护的服务变得不可用

了解详细信息：

- [多个虚拟机或服务之间的面向 Internet 的负载均衡器](/documentation/articles/load-balancer-internet-overview/)
- [内部负载均衡器概述](/documentation/articles/load-balancer-internal-overview/)

### 全局负载均衡
某些组织想要尽可能地实现最高级别的可用性。实现此目标的方法之一是将应用程序托管到全球分布的数据中心。应用程序托管在世界各地的数据中心时，即使整个地缘政治区域服务中断，但应用程序仍可正常运行。

除了将应用程序托管在全球分布的数据中心的可用性优点之外，还可以获得性能优势。使用将服务请求定向到最靠近发出请求设备的数据中心的机制即可获得这些性能优势。

全局负载均衡可以提供这两项优点。在 Azure 中，使用 Azure 流量管理器可以获得全局负载均衡的优点。

了解详细信息：

- [什么是流量管理器？](/documentation/articles/traffic-manager-overview/)

## 日志记录
网络级别的日志记录是任何网络安全方案的重要功能。在 Azure 中，可以记录信息，网络安全组可以通过这些信息获取网络级别的日志记录信息。使用 NSG 日志记录可从以下来源获取信息：

- 审核日志 – 这些日志用于查看提交到 Azure 订阅的所有操作。这些日志默认已启用，可以在 Azure 门户中使用。
- 事件日志 – 这些日志提供有关应用了哪些 NSG 规则的信息。
- 计数器日志 – 通过这些日志可以了解应用每个 NSG 规则以拒绝或允许流量的次数。

还可以使用 [ 世纪互联运营的 Microsoft Power BI](https://powerbi.microsoft.com/what-is-power-bi/)（一个功能强大的数据可视化工具）来查看和分析这些日志。

了解详细信息：

- [面向网络安全组 (NSG) 的 Log Analytics](/documentation/articles/virtual-network-nsg-manage-log/)

## 名称解析
名称解析是对 Azure 中托管的所有服务而言至关重要的功能。从安全角度看，入侵名称解析功能可能会导致攻击者将站点的请求重定向到攻击者的站点。安全名称解析是所有云托管服务的要求。

需要解决两种类型的名称解析：

- 内部名称解析 – Azure 虚拟网络和/或本地网络上的服务使用内部名称解析。通过 Internet 无法访问用于内部名称解析的名称。为了获取最高安全性，外部用户不能访问内部名称解析方案，这一点非常重要。
- 外部名称解析 – 本地和 Azure 虚拟网络外部的人员和设备使用外部名称解析。这些是在 Internet 上公开的、用于将连接定向到云服务的名称。

对于内部名称解析，可以使用两个选项：

- Azure 虚拟网络 DNS 服务器 – 创建新的 Azure 虚拟网络时，系统会创建 DNS 服务器。此 DNS 服务器可以解析该 Azure 虚拟网络上计算机的名称。此 DNS 服务器不可配置，由 Azure 结构管理器管理，因此是安全的名称解析解决方案。
- 自带 DNS 服务器 – 可以选择将自己选择的 DNS 服务器放置在 Azure 虚拟网络上。此 DNS 服务器可以是与 Active Directory 集成的 DNS 服务器，也可以是 Azure 合作伙伴提供的专用 DNS 服务器解决方案（可从 Azure 应用商店获取）。

了解详细信息：

- [虚拟网络概述](/documentation/articles/virtual-networks-overview/)
- [管理虚拟网络 (VNet) 使用的 DNS 服务器](/documentation/articles/virtual-networks-manage-dns-in-vnet/)

对于外部 DNS 解析，可以使用两个选项：

- 在本地托管自己的外部 DNS 服务器
- 在服务提供商那里托管自己的外部 DNS 服务器

许多大型组织都在本地托管其自己的 DNS 服务器。可以这样做的原因是它们具有相应的网络专业技术，并且在全球运营。

在大多数情况下，最好在服务提供商那里托管 DNS 名称解析服务。这些服务提供商具有网络专业技术并在全球运营，可确保名称解析服务具有极高的可用性。可用性对于 DNS 服务至关重要，因为如果名称解析服务失败，任何人都无法连接到面向 Internet 的服务。

Azure 以 Azure DNS 的形式提供高度可用的高性能外部 DNS 解决方案。此外部名称解析解决方案利用全球 Azure DNS 基础结构。它允许使用与其他 Azure 服务相同的凭据、API、工具和计费模式，将域托管在 Azure 中。由于属于 Azure 的一部分，它还会继承平台内置的强大安全控制。


## 外围网络体系结构
许多企业组织使用外围网络将其网络分段，在 Internet 与其服务之间创建一个缓冲区。网络的外围网络部分被视为低安全性区域，不应在该网段中放置高价值资产。通常会看到网络安全设备在外围网络段上有一个网络接口，另有一个网络接口连接到包含接受 Internet 入站连接的虚拟机和服务的网络。

外围网络设计和外围网络部署决策有许多变数，如果决定使用外围网络，要使用的外围网络类型应该根据网络安全要求来确定。

了解详细信息：

- [Azure 网络安全](/documentation/articles/best-practices-network-security/)

<!---HONumber=Mooncake_1024_2016-->