<properties
    pageTitle="为高级 Azure Redis 缓存配置虚拟网络 | Azure"
    description="了解如何为高级层 Azure Redis 缓存实例创建和管理虚拟网络支持"
    services="redis-cache"
    documentationcenter=""
    author="steved0x"
    manager="douge"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid="8b1e43a0-a70e-41e6-8994-0ac246d8bf7f"
    ms.service="cache"
    ms.workload="tbd"
    ms.tgt_pltfrm="cache-redis"
    ms.devlang="na"
    ms.topic="article"
    ms.date="04/12/2017"
    wacn.date="05/02/2017"
    ms.author="sdanie"
    ms.sourcegitcommit="78da854d58905bc82228bcbff1de0fcfbc12d5ac"
    ms.openlocfilehash="26aa5eae7d569770bb383b8c39b368dd47a85196"
    ms.lasthandoff="04/22/2017" />

# <a name="how-to-configure-virtual-network-support-for-a-premium-azure-redis-cache"></a>如何为高级 Azure Redis 缓存配置虚拟网络支持
Azure Redis 缓存具有不同的缓存产品（包括群集、持久性和虚拟网络支持等高级层功能），使缓存大小和功能的选择更加灵活。 VNet 是云中的专用网络。 为 Azure Redis 缓存实例配置了 VNet 后，该实例不可公开寻址，而只能从 VNet 中的虚拟机和应用程序进行访问。 本文说明如何为高级 Azure Redis 缓存实例配置虚拟网络支持。

> [AZURE.NOTE]
> Azure Redis 缓存同时支持经典 VNet 和 Resource Manager VNet。
> 
> 

有关其他高级缓存功能的信息，请参阅 [Azure Redis 缓存高级层简介](/documentation/articles/cache-premium-tier-intro/)。

## <a name="why-vnet"></a>为何使用 VNet？
[Azure 虚拟网络 (VNet)](/home/features/networking/) 部署为 Azure Redis 缓存提供增强的安全性和隔离性，并提供子网、访问控制策略，以及用于进一步限制访问的其他功能。

## <a name="virtual-network-support"></a>虚拟网络支持
在创建缓存期间，可在“新建 Redis 缓存”边栏选项卡中配置虚拟网络 (VNet) 支持。 

[AZURE.INCLUDE [redis-cache-create](../../includes/redis-cache-premium-create.md)]

选择高级定价层后，可以通过选择与缓存相同的订阅和位置中的 VNet，来配置 Redis VNet 集成。 若要使用新 VNet，请先创建 VNet，方法是遵循[使用 Azure 门户预览创建虚拟网络](/documentation/articles/virtual-networks-create-vnet-arm-pportal/)或[使用 Azure 门户预览创建虚拟网络（经典）](/documentation/articles/virtual-networks-create-vnet-classic-portal/)中的步骤，然后返回“新建 Redis 缓存”边栏选项卡来创建和配置高级缓存。

若要为新缓存配置 VNet，请单击“新建 Redis 缓存”边栏选项卡上的“虚拟网络”，然后从下拉列表中选择所需的 VNet。

![虚拟网络][redis-cache-vnet]

从“子网”下拉列表中选择所需的子网，然后指定所需的“静态 IP 地址”。 如果使用经典 VNet，则“静态 IP 地址”字段是可选的；如果未指定任何地址，将从选定的子网中选择一个。

> [AZURE.IMPORTANT]
> 将 Azure Redis 缓存部署到 Resource Manager VNet 时，缓存必须位于专用子网中，其中只能包含 Azure Redis 缓存实例，而不能包含其他任何资源。 如果尝试将 Azure Redis 缓存部署到包含其他资源的 Resource Manager VNet 子网，部署将会失败。
> 
> 

![虚拟网络][redis-cache-vnet-ip]

> [AZURE.IMPORTANT]
> Azure 会保留每个子网中的某些 IP 地址，你不可以使用这些地址。 子网的第一个和最后一个 IP 地址仅为协议一致性而保留，其他三个地址用于 Azure 服务。 有关详细信息，请参阅[使用这些子网中的 IP 地址是否有任何限制？](/documentation/articles/virtual-networks-faq/#are-there-any-restrictions-on-using-ip-addresses-within-these-subnets)
> <p> 
> 除了 Azure VNET 基础结构使用的 IP 地址以外，子网中的每个 Redis 实例为每个分片使用两个 IP 地址，为负载均衡器使用一个额外的 IP 地址。 非群集缓存被视为包含一个分片。
> 
> 

创建缓存之后，可以在“资源菜单”中单击“虚拟网络”，查看 VNet 的配置。

![虚拟网络][redis-cache-vnet-info]

若要在使用 VNet 时连接到 Azure Redis 缓存实例，请在连接字符串中指定缓存主机名，如以下示例中所示：

    private static Lazy<ConnectionMultiplexer> lazyConnection = new Lazy<ConnectionMultiplexer>(() =>
    {
        return ConnectionMultiplexer.Connect("contoso5premium.redis.cache.chinacloudapi.cn,abortConnect=false,ssl=true,password=password");
    });

    public static ConnectionMultiplexer Connection
    {
        get
        {
            return lazyConnection.Value;
        }
    }

## <a name="azure-redis-cache-vnet-faq"></a>Azure Redis 缓存 VNet 常见问题
以下列表包含有关 Azure Redis 缓存缩放的常见问题的解答。

* [Azure Redis 缓存和 VNet 有哪些常见的错误配置问题？](#what-are-some-common-misconfiguration-issues-with-azure-redis-cache-and-vnets)
* [是否可以对标准或基本缓存使用 VNet？](#can-i-use-vnets-with-a-standard-or-basic-cache)
* [为什么在某些子网中创建 Redis 缓存失败，而在其他子网中不会失败？](#why-does-creating-a-redis-cache-fail-in-some-subnets-but-not-others)
* [子网地址空间的要求是什么？](#what-are-the-subnet-address-space-requirements)
* [在 VNET 中托管缓存时，是否可以使用所有缓存功能？](#do-all-cache-features-work-when-hosting-a-cache-in-a-vnet)

## <a name="what-are-some-common-misconfiguration-issues-with-azure-redis-cache-and-vnets"></a> Azure Redis 缓存和 VNet 有哪些常见的错误配置问题？
在 VNet 中托管 Azure Redis 缓存时，会使用下表中的端口。 如果这些端口受阻，则缓存可能无法正常工作。 在 VNet 中使用 Azure Redis 缓存时，阻止这些端口中的一个或多个是最常见的错误配置问题。

| 端口 | 方向 | 传输协议 | 目的 | 远程 IP |
| --- | --- | --- | --- | --- |
| 80、443 |出站 |TCP |Azure 存储空间/PKI (Internet) 上的 Redis 依赖关系 |* |
| 53 |出站 |TCP/UDP |DNS (Internet/VNet) 上的 Redis 依赖关系 |* |
| 6379、6380 |入站 |TCP |与 Redis 的客户端通信、Azure 负载均衡 |VIRTUAL_NETWORK、AZURE_LOADBALANCER |
| 8443 |入站/出站 |TCP |Redis 的实现详细信息 |VIRTUAL_NETWORK |
| 8500 |入站 |TCP/UDP |Azure 负载均衡 |AZURE_LOADBALANCER |
| 10221-10231 |入站/出站 |TCP |Redis 的实现详细信息（可以将远程终结点限制为 VIRTUAL_NETWORK） |VIRTUAL_NETWORK、AZURE_LOADBALANCER |
| 13000-13999 |入站 |TCP |与 Redis 群集的客户端通信、Azure 负载均衡 |VIRTUAL_NETWORK、AZURE_LOADBALANCER |
| 15000-15999 |入站 |TCP |与 Redis 群集的客户端通信、Azure 负载均衡 |VIRTUAL_NETWORK、AZURE_LOADBALANCER |
| 16001 |入站 |TCP/UDP |Azure 负载均衡 |AZURE_LOADBALANCER |
| 20226 |入站+出站 |TCP |Redis 群集的实现详细信息 |VIRTUAL_NETWORK |

在虚拟网络中，可能一开始不符合 Azure Redis 缓存的网络连接要求。 在虚拟网络中使用时，Azure Redis 缓存需要以下所有项才能正常运行。

* 与全球 Azure 存储空间终结点建立的出站网络连接。 这包括位于与 Azure Redis 缓存实例相同区域中的终结点，以及位于 **其他** Azure 区域的存储终结点。 Azure 存储终结点在以下 DNS 域之下解析：*table.core.chinacloudapi.cn*、*blob.core.chinacloudapi.cn*、*queue.core.chinacloudapi.cn* 和 *file.core.chinacloudapi.cn*。 
* 与 *ocsp.msocsp.com*、*mscrl.microsoft.com* 和 *crl.microsoft.com* 建立的出站网络连接。 需要此连接才能支持 SSL 功能。
* 虚拟网络的 DNS 设置必须能够解析前面几点所提到的所有终结点和域。 确保已针对虚拟网络配置并维护有效的 DNS 基础结构即可符合这些 DNS 要求。
* 与以下 Azure 监视终结点（在下列 DNS 域下进行解析）的出站网络连接：shoebox2-black.shoebox2.metrics.nsatc.net、north-prod2.prod2.metrics.nsatc.net、azglobal-black.azglobal.metrics.nsatc.net、shoebox2-red.shoebox2.metrics.nsatc.net、east-prod2.prod2.metrics.nsatc.net、azglobal-red.azglobal.metrics.nsatc.net。

### <a name="can-i-use-vnets-with-a-standard-or-basic-cache"></a> 是否可以对标准或基本缓存使用 VNet？
只能对高级缓存使用 VNet。

### <a name="why-does-creating-a-redis-cache-fail-in-some-subnets-but-not-others"></a> 为什么在某些子网中创建 Redis 缓存失败，而在其他子网中不会失败？
如果你要将 Azure Redis 缓存部署到 Resource Manager VNet，该缓存必须在不包含任何其他资源类型的专用子网中。 如果尝试将 Azure Redis 缓存部署到包含其他资源的 Resource Manager VNet 子网，部署将会失败。 必须先删除该子网中的现有资源，然后才能创建新的 Redis 缓存。

只要你有足够的可用 IP 地址，就可以将多种类型的资源部署到经典 VNet。

### <a name="what-are-the-subnet-address-space-requirements"></a>子网地址空间的要求是什么？
Azure 会保留每个子网中的某些 IP 地址，你不可以使用这些地址。 子网的第一个和最后一个 IP 地址仅为协议一致性而保留，其他三个地址用于 Azure 服务。 有关详细信息，请参阅[使用这些子网中的 IP 地址是否有任何限制？](/documentation/articles/virtual-networks-faq/#are-there-any-restrictions-on-using-ip-addresses-within-these-subnets)

除了 Azure VNET 基础结构使用的 IP 地址以外，子网中的每个 Redis 实例为每个分片使用两个 IP 地址，为负载均衡器使用一个额外的 IP 地址。 非群集缓存被视为包含一个分片。

### <a name="do-all-cache-features-work-when-hosting-a-cache-in-a-vnet"></a> 在 VNET 中托管缓存时，是否可以使用所有缓存功能？
如果缓存是 VNET 的一部分，则只有 VNET 中的客户端可以访问缓存。 因此，以下缓存管理功能目前不起作用。

* Redis 控制台 - 由于 Redis 控制台在 VNET 外部的本地浏览器中运行，因此无法连接到你的缓存。

## <a name="use-expressroute-with-azure-redis-cache"></a>将 ExpressRoute 用于 Azure Redis 缓存
客户可以将 [Azure ExpressRoute](/home/features/expressroute/) 线路连接到虚拟网络基础结构，从而将其本地网络扩展到 Azure。 

默认情况下，新创建的 ExpressRoute 线路将会通告允许出站 Internet 连接的默认路由。 使用此配置，客户端应用程序将能够连接到其他 Azure 终结点（包括 Azure Redis 缓存）。

但是，常见的客户配置是定义其自身的默认路由 (0.0.0.0/0)，以强制出站 Internet 流量改为流向本地。 如果出站流量随后在本地被阻止，则流量流会断开与 Azure Redis 缓存的连接，导致 Azure Redis 缓存实例无法与其依赖项通信。

解决方法是在包含 Azure Redis 缓存的子网上定义一个或多个用户定义的路由 (UDR)。 UDR 定义了要遵循的子网特定路由，而不是默认路由。

如果可能，建议使用以下配置：

* ExpressRoute 配置播发 0.0.0.0/0 并默认使用强制隧道将所有输出流量发送到本地。
* 应用到包含 Azure Redis 缓存的子网的 UDR 使用发往公共 Internet 的 TCP/IP 流量的工作路由（例如，通过将下一跃点类型设置为“Internet”）定义 0.0.0.0/0。

这些步骤的组合效应是子网级 UDR 将优先于 ExpressRoute 强制隧道，从而确保来自 Azure Redis 缓存的出站 Internet 访问。

出于性能原因，从本地应用程序使用 ExpressRoute 连接到 Azure Redis 缓存实例不是典型使用方案（为了获得最佳性能，Azure Redis 缓存客户端应与 Azure Redis 缓存在同一区域中）。

>[AZURE.IMPORTANT] 
>UDR 中定义的路由 **必须** 足够明确，以便优先于 ExpressRoute 配置所播发的任何路由。 以下示例使用广泛 0.0.0.0/0 地址范围，因此使用更明确的地址范围，有可能意外地被路由播发重写。

>[AZURE.WARNING]  
>**未正确交叉播发从公共对等路径到专用对等路径的路由**的 ExpressRoute 配置不支持 Azure Redis 缓存。 已配置公共对等互连的 ExpressRoute 配置将收到来自 Microsoft 的大量 Microsoft Azure IP 地址范围的路由播发。 如果这些地址范围在专用对等路径上未正确交叉播发，结果是来自 Azure Redis 缓存实例子网的所有出站网络数据包都不会正确地使用强制隧道发送到客户的本地网络基础结构。 此网络流将破坏 Azure Redis 缓存。 此问题的解决方法是停止从公共对等路径到专用对等路径的交叉播发路由。

有关用户定义路由的背景信息，请参阅此[概述](/documentation/articles/virtual-networks-udr-overview/)。

有关 ExpressRoute 的详细信息，请参阅 [ExpressRoute 技术概述](/documentation/articles/expressroute-introduction/)。

## <a name="next-steps"></a>后续步骤
了解如何使用更多的高级缓存功能。

* [Azure Redis 缓存高级层简介](/documentation/articles/cache-premium-tier-intro/)

<!-- IMAGES -->

[redis-cache-vnet]: ./media/cache-how-to-premium-vnet/redis-cache-vnet.png

[redis-cache-vnet-ip]: ./media/cache-how-to-premium-vnet/redis-cache-vnet-ip.png

[redis-cache-vnet-info]: ./media/cache-how-to-premium-vnet/redis-cache-vnet-info.png
<!--Update_Description: wording update-->