<properties 
	pageTitle="如何为高级 Azure Redis 缓存配置虚拟网络支持" 
	description="了解如何为高级层 Azure Redis 缓存实例创建和管理虚拟网络支持" 
	services="redis-cache" 
	documentationCenter="" 
	authors="steved0x" 
	manager="dwrede" 
	editor=""/>

<tags
	ms.service="cache"
	ms.date="04/27/2016"
	wacn.date="06/29/2016"/>

# 如何为高级 Azure Redis 缓存配置虚拟网络支持
Azure Redis 缓存具有不同的缓存产品（包括新推出的高级层），使缓存大小和功能的选择更加灵活。

Azure Redis 缓存高级层包括群集、暂留和虚拟网络 (VNET) 支持。VNET 是你自己的网络在云中的表示形式。使用 VNET 配置 Azure Redis 缓存实例后，该实例不可公开寻址，而只能从 VNET 中的客户端访问。本文说明如何配置高级 Azure Redis 缓存实例的虚拟网络支持。

有关其他高级缓存功能的信息，请参阅[如何配置高级 Azure Redis 缓存的持久性](/documentation/articles/cache-how-to-premium-persistence/)和[如何配置高级 Azure Redis 缓存的群集](/documentation/articles/cache-how-to-premium-clustering/)。

## 为何使用 VNET？
[Azure 虚拟网络 (VNET)](/home/features/networking/) 部署为 Azure Redis 缓存提供增强的安全性，并提供子网、访问控制策略和进一步限制访问 Azure Redis 缓存的其他功能。

## 虚拟网络支持

在 Azure 中国区，只能通过 Azure PowerShell 或 Azure CLI 管理 Redis 缓存


[AZURE.INCLUDE [azurerm-azurechinacloud-environment-parameter](../includes/azurerm-azurechinacloud-environment-parameter.md)]


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
	    return ConnectionMultiplexer.Connect("172.160.0.99,sslhost=contoso5.redis.cache.chinacloudapi.cn,abortConnect=false,ssl=true,password=password");
	});
	
	public static ConnectionMultiplexer Connection
	{
	    get
	    {
	        return lazyConnection.Value;
	    }
	}

## Azure Redis 缓存 VNET 常见问题

以下列表包含有关 Azure Redis 缓存缩放的常见问题的解答。

## Azure Redis 缓存和 VNET 有哪些常见的错误配置问题？

在 VNET 中承载 Azure Redis 缓存时，会使用下表中的端口。如果这些端口受阻，则缓存可能无法正常工作。在 VNET 中使用 Azure Redis 缓存时，阻止这些端口中的一个或多个是最常见的错误配置问题。

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




## 是否可以对标准或基本缓存使用 VNET？

只能对高级缓存使用 VNET。

## 后续步骤
了解如何使用更多的高级版缓存功能。

-	[如何为高级 Azure Redis 缓存配置暂留](/documentation/articles/cache-how-to-premium-persistence/)
-	[如何为高级 Azure Redis 缓存配置群集功能](/documentation/articles/cache-how-to-premium-clustering/)





  
<!-- IMAGES -->

[redis-cache-new-cache-menu]: ./media/cache-how-to-premium-vnet/redis-cache-new-cache-menu.png

[redis-cache-premium-pricing-tier]: ./media/cache-how-to-premium-vnet/redis-cache-premium-pricing-tier.png

[redis-cache-vnet]: ./media/cache-how-to-premium-vnet/redis-cache-vnet.png

[redis-cache-vnet-ip]: ./media/cache-how-to-premium-vnet/redis-cache-vnet-ip.png

[redis-cache-vnet-info]: ./media/cache-how-to-premium-vnet/redis-cache-vnet-info.png


<!---HONumber=Mooncake_0321_2016-->