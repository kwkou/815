<properties 
	pageTitle="如何为高级 Azure Redis 缓存配置虚拟网络支持 | Azure" 
	description="了解如何为高级层 Azure Redis 缓存实例创建和管理虚拟网络支持" 
	services="redis-cache" 
	documentationCenter="" 
	authors="steved0x" 
	manager="douge" 
	editor=""/>

<tags
	ms.service="cache"
	ms.date="06/01/2016"
	wacn.date="07/25/2016"/>

# 如何为高级 Azure Redis 缓存配置虚拟网络支持
Azure Redis 缓存具有不同的缓存产品（包括新推出的高级层），使缓存大小和功能的选择更加灵活。

Azure Redis 缓存高级层包括群集、持久性和虚拟网络 (VNet) 支持。VNet 是云中的专用网络。为 Azure Redis 缓存实例配置了 VNet 后，该实例不可公开寻址，而只能从 VNet 中的虚拟机和应用程序进行访问。本文说明如何为高级 Azure Redis 缓存实例配置虚拟网络支持。

>[AZURE.NOTE] Azure Redis 缓存同时支持经典 VNet 和 ARM VNet。

有关其他高级缓存功能的信息，请参阅[如何配置高级 Azure Redis 缓存的持久性](/documentation/articles/cache-how-to-premium-persistence/)和[如何配置高级 Azure Redis 缓存的群集](/documentation/articles/cache-how-to-premium-clustering/)。

## 为何使用 VNet？
[Azure 虚拟网络 (VNet)](/home/features/networking/) 部署为 Azure Redis 缓存提供增强的安全性和隔离性，并提供子网、访问控制策略和进一步限制访问 Azure Redis 缓存的其他功能。

## 虚拟网络支持

在 Azure 中国区，只能通过 Azure PowerShell 或 Azure CLI 管理 Redis 缓存

[AZURE.INCLUDE [azurerm-azurechinacloud-environment-parameter](../../includes/azurerm-azurechinacloud-environment-parameter.md)]

[AZURE.INCLUDE [redis-cache-create](../../includes/redis-cache-premium-create.md)]


使用以下 PowerShell 脚本创建缓存：

	$VerbosePreference = "Continue"

	# Create a new cache with date string to make name unique. 
	$cacheName = "MovieCache" + $(Get-Date -Format ('ddhhmm')) 
	$location = "China North"
	$resourceGroupName = "Default-Web-ChinaNorth"
	
	$movieCache = New-AzureRmRedisCache -Location $location -Name $cacheName  -ResourceGroupName $resourceGroupName -Size 6GB -Sku Premium -VirtualNetwork /subscriptions/{subid}/Microsoft.ClassicNetwork/VirtualNetworks/vnet1 -Subnet Front -StaticIP 10.10.1.5

**-StaticIP** 参数是可选的。如果选择的静态 IP 已被使用，将会显示错误消息。如果未在此处指定任何 IP，系统会从所选子网中选择一个。

创建缓存后，该缓存只能由同一 VNET 中的客户端访问。

>[AZURE.IMPORTANT] 若要在使用 VNET 时访问 Azure Redis 缓存实例，请传递 VNET 中缓存的静态 IP 地址作为第一个参数，并传入包含缓存终结点的 `sslhost` 参数。在以下示例中，静态 IP 地址为 `172.160.0.99`，缓存终结点为 `contoso5.redis.cache.chinacloudapi.cn`。

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

## Azure Redis 缓存 VNet 常见问题

以下列表包含有关 Azure Redis 缓存缩放的常见问题的解答。

-	[Azure Redis 缓存和 VNet 有哪些常见的错误配置问题？](#what-are-some-common-misconfiguration-issues-with-azure-redis-cache-and-vnets)
-	[是否可以对标准或基本缓存使用 VNet？](#can-i-use-vnets-with-a-standard-or-basic-cache)
-	[为什么在某些子网中创建 Redis 缓存失败，而在其他子网中不会失败？](#why-does-creating-a-redis-cache-fail-in-some-subnets-but-not-others)


### <a name="what-are-some-common-misconfiguration-issues-with-azure-redis-cache-and-vnets"></a> Azure Redis 缓存和 VNet 有哪些常见的错误配置问题？

在 VNet 中托管 Azure Redis 缓存时，会使用下表中的端口。如果这些端口受阻，则缓存可能无法正常工作。在 VNet 中使用 Azure Redis 缓存时，阻止这些端口中的一个或多个是最常见的错误配置问题。

| 端口 | Direction | 传输协议 | 目的 | 远程 IP |
|-------------|------------------|--------------------|-----------------------------------------------------------------------------------|-------------------------------------|
| 80、443 | 出站 | TCP | Azure 存储空间/PKI (Internet) 上的 Redis 依赖关系 | * |
| 53 | 出站 | TCP/UDP | DNS (Internet/VNet) 上的 Redis 依赖关系 | * |
| 6379、6380 | 入站 | TCP | 与 Redis 的客户端通信、Azure 负载平衡 | VIRTUAL\_NETWORK、AZURE\_LOADBALANCER |
| 8443 | 入站/出站 | TCP | Redis 的实现详细信息 | VIRTUAL\_NETWORK |
| 8500 | 入站 | TCP/UDP | Azure 负载平衡 | AZURE\_LOADBALANCER |
| 10221-10231 | 入站/出站 | TCP | Redis 的实现详细信息（可以将远程终结点限制为 VIRTUAL\_NETWORK） | VIRTUAL\_NETWORK、AZURE\_LOADBALANCER |
| 13000-13999 | 入站 | TCP | 与 Redis 群集的客户端通信、Azure 负载平衡 | VIRTUAL\_NETWORK、AZURE\_LOADBALANCER |
| 15000-15999 | 入站 | TCP | 与 Redis 群集的客户端通信、Azure 负载平衡 | VIRTUAL\_NETWORK、AZURE\_LOADBALANCER |
| 16001 | 入站 | TCP/UDP | Azure 负载平衡 | AZURE\_LOADBALANCER |
| 20226 | 入站+出站 | TCP | Redis 群集的实现详细信息 | VIRTUAL\_NETWORK |


在虚拟网络中，可能一开始不符合 Azure Redis 缓存的网络连接要求。在虚拟网络中使用时，Azure Redis 缓存需要以下所有项才能正常运行。

-  与全球 Azure 存储空间终结点建立的出站网络连接。这包括位于与 Azure Redis 缓存实例相同区域中的终结点，以及位于**其他** Azure 区域的存储终结点。Azure 存储空间终结点在以下 DNS 域之下解析：table.core.chinacloudapi.cn、blob.core.chinacloudapi.cn、queue.core.chinacloudapi.cn 和 file.core.chinacloudapi.cn。
-  与 ocsp.msocsp.com、mscrl.microsoft.com 和 crl.microsoft.com 建立的出站网络连接。需要此连接才能支持 SSL 功能。
-  虚拟网络的 DNS 设置必须能够解析前面几点所提到的所有终结点和域。确保已针对虚拟网络配置并维护有效的 DNS 基础结构即可符合这些 DNS 要求。



### <a name="can-i-use-vnets-with-a-standard-or-basic-cache"></a> 是否可以对标准或基本缓存使用 VNet？

只能对高级缓存使用 VNet。

### <a name="why-does-creating-a-redis-cache-fail-in-some-subnets-but-not-others"></a> 为什么在某些子网中创建 Redis 缓存失败，而在其他子网中不会失败？

如果你要将 Azure Redis 缓存部署到 ARM VNet，则该缓存必须在不包含任何其他资源类型的专用子网中。如果尝试将 Azure Redis 缓存部署到包含其他资源的 ARM VNet 子网，则部署将失败。必须先删除该子网中的现有资源，然后才能创建新的 Redis 缓存。

只要你有足够的可用 IP 地址，就可以将多种类型的资源部署到经典 VNet。

## 将 ExpressRoute 用于 Azure Redis 缓存

客户可以将 [Azure ExpressRoute](/home/features/expressroute/) 线路连接到虚拟网络基础结构，从而将其本地网络扩展到 Azure。

默认情况下，新创建的 ExpressRoute 线路将会通告允许出站 Internet 连接的默认路由。使用此配置，客户端应用程序将能够连接到其他 Azure 终结点（包括 Azure Redis 缓存）。

但是，常见的客户配置是定义其自身的默认路由 (0.0.0.0/0)，以强制出站 Internet 流量改为流向本地。此流量一定会中断与 Azure Redis 缓存的连接，因为已在本地阻止出站流量，或者 NAT 到无法再使用各种 Azure 终结点的一组无法识别的地址。

解决方法是在包含 Azure Redis 缓存的子网上定义一个或多个用户定义的路由 (UDR)。UDR 定义了要遵循的子网特定路由，而不是默认路由。

如果可能，建议使用以下配置：

- ExpressRoute 配置播发 0.0.0.0/0 并默认使用强制隧道将所有输出流量发送到本地。
- 已应用到包含 Azure Redis 缓存的子网的 UDR 使用 Internet 的下一个跃点类型定义 0.0.0.0/0（本文后面将提供此示例）。

这些步骤的组合效应是子网级 UDR 将优先于 ExpressRoute 强制隧道，从而确保来自 Azure Redis 缓存的出站 Internet 访问。

虽然由于性能原因，从本地应用程序使用 ExpressRoute 连接到 Azure Redis 缓存实例不是典型使用方案（为了获得最佳性能，Azure Redis 缓存客户端应与 Azure Redis 缓存在同一区域中），但在此方案中出站网络路径不能经过内部公司代理，也不能使用强制隧道发送到本地。否则会更改来自 Azure Redis 缓存的出站网络流量的有效 NAT 地址。更改 Azure Redis 缓存实例的出站网络流量的 NAT 地址会导致上述众多终结点的连接失败。这会导致创建 Azure Redis 缓存尝试失败。

**重要说明：**UDR 中定义的路由**必须**足够明确，以便优先于 ExpressRoute 配置所播发的任何路由。以下示例使用广泛 0.0.0.0/0 地址范围，因此使用更明确的地址范围，有可能意外地被路由播发重写。

**非常重要：****未正确交叉播发从公共对等路径到专用对等路径的路由**的 ExpressRoute 配置不支持 Azure Redis 缓存。已配置公共对等互连的 ExpressRoute 配置将收到来自 Microsoft 的大量 Microsoft Azure IP 地址范围的路由播发。如果这些地址范围在专用对等路径上未正确交叉播发，最后的结果是来自 Azure Redis 缓存实例子网的所有出站网络数据包都不会正确地使用强制隧道发送到客户的本地网络基础结构。此网络流将破坏 Azure Redis 缓存。此问题的解决方法是停止从公共对等路径到专用对等路径的交叉播发路由。

有关用户定义路由的背景信息，请参阅此[概述](/documentation/articles/virtual-networks-udr-overview/)。

有关 ExpressRoute 的详细信息，请参阅 [ExpressRoute 技术概述](/documentation/articles/expressroute-introduction/)

## 后续步骤
了解如何使用更多的高级版缓存功能。

-	[如何为高级 Azure Redis 缓存配置暂留](/documentation/articles/cache-how-to-premium-persistence/)
-	[如何为高级 Azure Redis 缓存配置群集功能](/documentation/articles/cache-how-to-premium-clustering/)





  
<!-- IMAGES -->

[redis-cache-vnet]: ./media/cache-how-to-premium-vnet/redis-cache-vnet.png

[redis-cache-vnet-ip]: ./media/cache-how-to-premium-vnet/redis-cache-vnet-ip.png

[redis-cache-vnet-info]: ./media/cache-how-to-premium-vnet/redis-cache-vnet-info.png

<!---HONumber=AcomDC_0718_2016-->