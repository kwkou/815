<properties
   pageTitle="Azure 安全入门 | Azure"
   description="本文概述了 Azure 安全功能，以及组织在将其资产迁移到云提供商处时需要注意的一般性注意事项。"
   services="security"
   documentationCenter="na"
   authors="YuriDio"
   manager="swadhwa"
   editor="TomSh"/>  


<tags
   ms.service="security"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="na"
   ms.date="05/19/2016"
   wacn.date="10/31/2016"
   ms.author="yuridio"/>  


#Microsoft Azure 安全入门

当你生成 IT 资产或将其迁移到云提供商处时，你需要依赖该组织来保护你委托给该组织服务的应用程序和数据，并且需要依赖该组织提供给你的安全控制来控制基于云的资产的安全性。

Azure 的基础结构（从设备到应用程序）经过设计，可同时托管数百万的客户，并为企业提供可靠的基础，使之能够满足其安全需求。此外，Azure 还为你提供广泛的可配置安全选项以及对这些选项进行控制的功能，方便你自定义安全措施来满足部署的独特要求。

在这篇有关 Azure 安全性的概述性文章中，我们将了解：

-   可以用来确保 Azure 中服务和数据安全性的 Azure 服务和功能

-   世纪互联如何通过保护 Azure 基础结构来保护数据和应用程序

##数据访问控制和加密

世纪互联所采用的“职责分离”和[最小特权](https://en.wikipedia.org/wiki/Principle_of_least_privilege)原则贯穿整个 Azure 操作。Azure 支持人员访问相关数据需要你根据“实时”原则进行明确许可，并且会进行记录和审核，在完成相关任务后就会取消访问权限。

此外，Azure 还提供多种功能来保护正在传输的数据和静态数据，包括针对数据、文件、应用程序、服务、通信和驱动器进行加密。在将信息放置在 Azure 中以及将密钥存储在本地数据中心之前，你可以选择对其进行加密。

![Azure 中的 Microsoft Antimalware](./media/azure-security-getting-started/sec-azgsfig1.PNG)  


### Azure 加密技术

你可以选择在 Azure 的 VHD（内含敏感信息）上配置 [BitLocker 驱动器加密](https://technet.microsoft.com/zh-cn/library/cc732774.aspx)。

Azure 中其他用于确保数据安全的功能包括：

-   应用程序开发人员可以使用 Windows [CryptoAPI](https://msdn.microsoft.com/zh-cn/library/ms867086.aspx) 和 .NET Framework 将加密内建到 Azure 中部署的应用程序。

- 针对 Blob 存储进行客户端加密可以让你完全控制密钥。存储服务永远看不到这些密钥，因此无法解密数据。

-   Azure 支持在 SQL Server 虚拟机中进行[表级和列级加密 (TDE/CLE)](http://blogs.msdn.com/b/sqlsecurity/archive/2015/05/12/recommendations-for-using-cell-level-encryption-in-azure-sql-database.aspx)，并支持在客户的数据中心部署第三方本地密钥管理服务器。

-   存储帐户密钥、共享访问签名、管理证书以及其他密钥对每个 Azure 租户来说都是唯一的。

-   Azure 支持和使用数据加密机制，包括 SSL/TLS、IPsec 和 AES，具体取决于数据类型、容器以及传输情况。

##虚拟化

Azure 平台使用虚拟化环境。用户实例以单独的虚拟机方式运行，这些虚拟机无法访问物理主机服务器。进行这样的隔离是通过物理[处理器（0 环/3 环）权限级别](https://en.wikipedia.org/wiki/Protection_ring)强制实施的。

0 环表示最高权限，3 环表示最低权限。来宾 OS 在权限较低的 1 环运行，应用程序在权限最低的 3 环运行。对物理资源进行这样的虚拟化会在来宾 OS 和虚拟机监控程序之间形成清晰的隔离，从而在这二者之间实现进一步的安全隔离。

Azure 的虚拟机监控程序相当于微内核，可将所有硬件访问请求从来宾 VM 传递到主机，以便使用名为 VMBus 的共享内存界面进行处理。这样可以防止用户获取对系统的原始读取/写入/执行访问权限，减轻共享系统资源的风险。

![Azure 中的 Microsoft Antimalware](./media/azure-security-getting-started/sec-azgsfig2.PNG)  


### Azure 如何实现虚拟化

Azure 使用虚拟机监控程序防火墙（数据包筛选器），该防火墙是在虚拟机监控程序中实施的，可以通过结构控制器代理进行配置。这有助于防止租户进行未经授权的访问。默认情况下，在创建 VM 时，会阻止所有流量，然后通过结构控制器代理来配置数据包筛选器，添加*规则和例外* 以允许经授权的流量。

此处对两类规则进行了编程：

-   **计算机配置或基础结构规则**：默认情况下，将阻止所有通信。在例外情况下，可以允许 VM 发送和接收 DHCP 和 DNS 流量。VM 还可以将流量发送到“公共”Internet 以及群集和 OS 激活服务器中的其他 VM。VM 的传出目标允许列表不包括 Azure 路由器子网、Azure 管理后端以及其他 属性。

-   **角色配置文件**：根据租户的服务模型定义入站 ACL。例如，如果某个租户在特定 VM 的端口 80 上有一个 Web 前端，Azure 会对所有 IP 开放 TCP 端口 80，前提是你要在 [Azure 服务管理](/documentation/articles/resource-manager-deployment-model/)模型中配置一个终结点。如果 VM 有一个正在运行的后端或辅助角色，我们就会只向同一租户中的 VM 开放该辅助角色。

##隔离

另一项重要的云安全要求是始终进行隔离，防止在共享型多租户体系结构的部署之间对信息进行未经授权的传输和无意的传输。

Azure 通过 VLAN 隔离、ACL、负载均衡器和 IP 筛选器来实施[网络访问控制](https://azure.microsoft.com/blog/network-isolation-options-for-machines-in-windows-azure-virtual-networks/)和隔离。目标为你的虚拟机的外部入站通信仅限于你所定义的端口和协议。实施网络筛选是为了防止欺骗通信，将传入和传出通信限制到可信平台组件。在边界保护设备上会实施流量策略，这些设备默认情况下会拒绝通信。

![Azure 中的 Microsoft Antimalware](./media/azure-security-getting-started/sec-azgsfig3.PNG)  


网络地址转换 (NAT) 用于将内部网络流量与外部流量分开。内部流量不可通过外部进行路由。可以通过外部进行路由的[虚拟 IP 地址](http://blogs.msdn.com/b/cloud_solution_architect/archive/2014/11/08/vips-dips-and-pips-in-microsoft-azure.aspx)将转换成[内部动态 IP](http://blogs.msdn.com/b/cloud_solution_architect/archive/2014/11/08/vips-dips-and-pips-in-microsoft-azure.aspx) 地址，后者只能在 Azure 内部进行路由。

流向 Azure 虚拟机的外部流量会通过访问控制列表 (ACL) 在路由器、负载均衡器以及第 3 层交换机上进行防火墙处理。仅允许特定的已知协议。使用 ACL 是为了限制从来宾 VM 流向其他管理用 VLAN 的流量。此外还会通过 IP 筛选器在主机 OS 上对流量进行筛选，进一步限制数据链接和网络层的流量。

### Azure 如何实现隔离

Azure 结构控制器负责将基础结构资源分配到租户工作负荷，并管理从主机到 VM 的单向通信。Azure 虚拟机监控程序会在 VM 之间强制实施内存和流程的隔离，并通过安全方式将网络流量路由到来宾 OS 租户。Azure 还为租户、存储和虚拟网络实施隔离：

-   每个 Azure AD 租户都通过安全边界进行逻辑隔离。

-   Azure 存储帐户对于每个订阅都是唯一的，访问必须通过存储帐户密钥进行身份验证。

-   通过将唯一的专用 IP 地址、防火墙和 IP ACL 组合起来，可以对虚拟网络进行逻辑隔离。负载均衡器根据终结点定义将流量路由到相应的租户。

##虚拟网络和防火墙

Azure 中的[分布式网络和虚拟网络](http://download.microsoft.com/download/4/3/9/43902EC9-410E-4875-8800-0788BE146A3D/Windows%20Azure%20Network%20Security%20Whitepaper%20-%20FINAL.docx)有助于确保将你的专用网络流量与其他 Azure 虚拟网络上的流量进行逻辑隔离。

![Azure 中的 Microsoft 虚拟网络和防火墙](./media/azure-security-getting-started/sec-azgsfig4.PNG)  


订阅可能包含多个隔离的专用网络（并包括防火墙、负载均衡以及网络地址转换）。

Azure 在每个 Azure 群集中提供三种主要级别的网络隔离，可通过逻辑方式来隔离流量。[虚拟局域网](/home/features/networking/) (VLAN) 用于将客户流量与 Azure 网络的其余部分分开。可以通过负载均衡器对从群集外部访问 Azure 网络进行限制。

流向 VM 以及从 VM 流出的网络流量必须经过虚拟交换机和虚拟机监控程序。根 OS 中的 IP 筛选器组件将根 VM 与来宾 VM 隔离，以及将来宾 VM 相互隔离。它会对流量进行筛选，将通信限制在租户的节点与公共 Internet 之间（基于客户的服务配置），将这些节点与其他租户隔离开。

IP 筛选器可以防止来宾 VM 执行以下操作：

- 生成欺骗性流量

- 接收不发送给它们的流量

- 将流量定向到受保护的基础结构终结点

- 发送或接收不当的广播流量

你可以将虚拟机置于 [Azure 虚拟网络](/documentation/services/networking/)中。这些虚拟网络类似于你在本地环境中配置的网络，在本地环境中，网络通常与虚拟交换机相关联。连接到同一个 Azure 虚拟网络的虚拟机可以相互通信而无需其他配置。你还可以选择在 Azure 虚拟网络中配置不同的子网。

可以使用以下 Azure 虚拟网络技术来帮助实现 Azure 虚拟网络上的安全通信：

-   [**网络安全组 (NSG)**](/documentation/articles/virtual-networks-nsg/)。可以在虚拟网络中使用 NSG 控制流向一个或多个虚拟机 (VM) 实例的流量。NSG 包含根据流量方向、协议、源地址和端口以及目标地址和端口允许或拒绝流量的访问控制规则。

-   [**用户定义的路由**](/documentation/articles/virtual-networks-udr-overview/)。你可以创建用户定义的路由来指定下一跃点，方便数据包流向特定的子网并转到你的虚拟网络安全设备，从而控制数据包通过虚拟设备进行的路由。

-   [**IP 转发**](/documentation/articles/virtual-networks-udr-overview/)。虚拟网络安全设备必须能够接收不发送给自身的传入流量。若要允许 VM 接收发送到其他目标的流量，可以为该 VM 启用 IP 转发。

-   [**强制隧道**](/documentation/articles/vpn-gateway-about-forced-tunneling/)。借助强制隧道，你可以通过站点到站点 VPN 隧道，将 Azure 虚拟网络中的虚拟机所生成的全部 Internet 绑定流量重定向或“强制”返回到本地位置，以便进行检查和审核

-   [**终结点 ACL**](/documentation/articles/virtual-machines-windows-classic-setup-endpoints/)。你可以通过定义终结点 ACL 来控制哪些计算机允许从 Internet 到 Azure 虚拟网络上的虚拟机的入站连接。

-   [**合作伙伴网络安全解决方案**](https://market.azure.cn/List/Index?sort=Featured&filters=tag:security)。可以通过 Azure 映像应用商店访问大量的合作伙伴网络安全解决方案。

### Azure 如何实施虚拟网络和防火墙

默认情况下，Azure 在所有主机和来宾 VM 上实施数据包筛选防火墙。Azure 库中的 Windows OS 映像也默认启用 Windows 防火墙。位于 Azure 面向公众的网络外围的负载均衡器会根据客户管理员所管理的 IP ACL 来控制通信。

如果 Azure 在正常操作过程中或灾难过程中移动某个客户的数据，它会通过专用加密通信通道来进行。Azure 可以在虚拟网络和防火墙中使用的其他功能包括：

-   **本机主机防火墙**：在没有虚拟机监控程序（因此也就没有 Windows 防火墙）的本机 OS 上运行的 Azure 结构和存储是使用上述两组规则进行配置的。存储会运行本机主机防火墙来优化性能。

-   **主机防火墙**：主机防火墙用于保护运行虚拟机监控程序的主机操作系统。可以通过编程方式对规则进行设置，只允许结构控制器和跳转盒在特定端口上与主机 OS 通信。其他例外包括允许 DHCP 响应和 DNS 回复。Azure 使用计算机配置文件，其中包括针对主机 OS 的防火墙规则模板。主机本身受 Windows 防火墙保护，可以免受外部攻击的威胁，该防火墙已配置为仅允许来自已知的经过身份验证的源的通信。

-   **来宾防火墙**：复制 VM 切换数据包筛选器中的规则，不过这些规则是在不同软件（即来宾 OS 的 Windows 防火墙组件）中通过编程方式设置的。来宾 VM 防火墙在经过配置后，可以限制与来宾 VM 的通信，即使主机 IP 筛选器上的配置允许进行这样的通信。例如，你可以选择使用来宾 VM 防火墙来限制你的两个 VNet 之间的通信，这两个 VNet 已配置为可以互相进行连接。

-   **存储防火墙 (FW)**：存储前端的防火墙会对通信进行筛选，只允许在端口 80/443 以及其他必需实用程序端口上进行通信。存储后端的防火墙会将通信限制为只能来自存储前端服务器。

-   **虚拟网络网关**：[Azure 虚拟网络网关](/documentation/articles/vpn-gateway-howto-site-to-site-resource-manager-portal/)充当跨界网关，将你在 Azure 虚拟网络中的工作负荷连接到本地站点。它需要通过 [IPsec 站点到站点 VPN 隧道](/documentation/articles/vpn-gateway-create-site-to-site-rm-powershell/)或 [ExpressRoute](/documentation/articles/expressroute-introduction/) 线路连接到本地站点。对于 IPsec/IKE VPN 隧道，这些网关会执行 IKE 握手，并在虚拟网络和本地站点之间建立 IPsec S2S VPN 隧道。虚拟网络网关还会终止[点到站点 VPN](/documentation/articles/vpn-gateway-howto-point-to-site-classic-azure-portal/)。

##安全远程访问

存储在云中的数据必须具有足够的安全措施来防止遭到攻击，并且需要在传输过程中保持机密性和完整性。这其中包括网络控制，同时结合使用组织的基于策略的、可审核的身份和访问管理机制。

内置加密技术使你能够在部署内部和部署之间、Azure 区域之间以及从 Azure 到本地数据中心之间对通信进行加密。管理员通过[远程桌面会话](/documentation/articles/virtual-machines-windows-classic-connect-logon/)、[远程 Windows PowerShell](http://blogs.technet.com/b/heyscriptingguy/archive/2013/09/07/weekend-scripter-remoting-the-cloud-with-windows-azure-and-powershell.aspx) 和 Azure 管理门户对虚拟机进行的访问始终加密。

为了安全地将你的本地数据中心扩展到云，Azure 提供了[站点到站点 VPN](/documentation/articles/vpn-gateway-create-site-to-site-rm-powershell/) 和[点到站点 VPN](/documentation/articles/vpn-gateway-howto-point-to-site-classic-azure-portal/) 以及通过 [ExpressRoute](/documentation/articles/expressroute-introduction/) 进行的专用链接（通过 VPN 连接到 Azure 虚拟网络会进行加密）。

### Azure 如何实现安全的远程访问

连接到 Azure 门户时，必须始终进行身份验证，并且需要 SSL/TLS。你可以配置管理证书，以便进行安全管理。完全支持各种行业标准安全协议，例如 [SSTP](https://technet.microsoft.com/zh-cn/library/2007.06.cableguy.aspx) 和 [IPsec](https://en.wikipedia.org/wiki/IPsec)。

使用 [Azure ExpressRoute](/documentation/articles/expressroute-introduction/)，可在 Azure 数据中心与你的本地环境或共同租用环境中的基础结构之间创建专用连接。ExpressRoute 连接不通过公共 Internet 。与常用的基于 Internet 的链接相比，这些链接更可靠，速度更快，延迟更低，安全性更高。在某些情况下，使用 ExpressRoute 连接在本地和 Azure 之间传输数据还可以产生显著的成本效益。

##日志记录和监视

Azure 会对生成审计线索的安全相关事件进行经过身份验证的日志记录，其本身也经过设计，可以防止篡改。这包括系统信息，例如 Azure 基础结构 VM 和 Azure AD 中的安全事件日志。安全事件监视包括收集各种事件，例如更改 DHCP 或 DNS 服务器 IP 地址；尝试访问已通过设计进行阻止的端口、协议或 IP 地址；更改安全策略或防火墙设置；创建帐户或组；意外的流程或驱动程序安装。

![Azure 中的 Microsoft Antimalware](./media/azure-security-getting-started/sec-azgsfig5.PNG)  


审核日志（记录特权用户访问和活动）、授权的和未经授权的访问尝试、系统异常以及信息安全事件将会保留固定的时间。日志的保留由你全权决定，因为你可以根据自己的需求来配置日志的收集和保留。

### Azure 如何实施日志记录和监视

Azure 将管理代理 (MA) 和 Azure 安全监视器 (ASM) 代理部署到每个受管的计算节点、存储节点或结构节点，不管它们是本机的还是虚拟的。每个管理代理都经过配置，可以使用从 Azure 证书存储获得的证书通过服务团队存储帐户进行身份验证，并将预先配置的诊断和事件数据转发到存储帐户。这些代理不会部署到客户的虚拟机中。

Azure 管理员通过 Web 门户访问日志，对日志进行的访问必须经过身份验证，并且是可控的。管理员可以对日志进行解析、筛选、关联和分析。与日志对应的 Azure 服务团队存储帐户不允许管理员直接访问，这样是为了防止日志篡改。

世纪互联使用 Syslog 协议从网络设备收集日志，以及从主机服务器收集日志。这些日志放置在日志数据库中，发生可疑事件时，就会生成直接发送给世纪互联管理员的警报。管理员可以访问并分析这些日志。

[Azure 诊断](/documentation/articles/cloud-services-dotnet-diagnostics/)是 Azure 的一项功能，你可以通过它从运行在 Azure 中的应用程序收集诊断数据。可以将这些诊断数据用于调试和故障排除、度量性能、监视资源使用状况、进行流量分析和容量规划以及进行审核。收集诊断数据后，可以将其传输到 Azure 存储帐户进行永久保存。可以按计划传输，也可以按需传输。

##威胁缓解措施

除了隔离、加密和筛选，Azure 还采用了大量的威胁缓解机制和流程来保护基础结构和服务。其中包括内部控制和技术，用于检测和化解各种高级威胁，例如 DDoS 和特权提升。

世纪互联采取的安全控制和风险管理流程目的是确保其云基础结构的安全性，从而降低发生安全事件的风险。但是，在发生事件的情况下，客户支持团队将会全天候进行响应。

### Azure 如何实施威胁缓解措施

Azure 建立安全控制的目的是实施威胁缓解措施，同时协助客户减轻其环境中的可能威胁。以下列表总结了 Azure 提供的威胁缓解功能：

-   [Azure 反恶意软件](/documentation/articles/azure-security-antimalware/)在所有基础结构服务器上默认启用。你可以在自己的 VM 中选择性地启用它。

-   世纪互联会持续监视服务器、网络和应用程序以检测各种威胁，防止遭到攻击。自动警报会将异常行为通知给管理员，因此管理员可以针对内部和外部威胁采取纠正性措施。

-   可以选择在订阅中部署第三方安全解决方案，例如 [Azure 映像应用商店](https://market.azure.cn/List/Index?sort=Featured&filters=tag:security)提供的 Web 应用程序防火墙。

-   采取的渗透测试方法包括“[红队测试](http://download.microsoft.com/download/C/1/9/C1990DBA-502F-4C2A-848D-392B93D9B9C3/Microsoft_Enterprise_Cloud_Red_Teaming.pdf)”，是指 安全专家（非客户）攻击 Azure 中的实时生产系统，以便测试系统对现实世界的高级持久性威胁的防御能力。

-   使用集成的部署系统来管理安全修补程序在 Azure 平台的分发和安装。

##后续步骤

[Azure 信任中心](https://www.trustcenter.cn)

[Azure 安全团队博客](http://blogs.msdn.com/b/azuresecurity/)

[Active Directory 博客](http://blogs.technet.com/b/ad/)

<!---HONumber=Mooncake_1024_2016-->