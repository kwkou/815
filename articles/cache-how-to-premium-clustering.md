<properties 
	pageTitle="如何为高级 Azure Redis 缓存配置 Redis 群集功能" 
	description="了解如何为高级级别的 Azure Redis 缓存实例创建和管理 Redis 缓存功能" 
	services="redis-cache" 
	documentationCenter="" 
	authors="steved0x" 
	manager="dwrede" 
	editor=""/>

<tags
	ms.service="cache"
	ms.date="03/04/2016"
	wacn.date="04/19/2016"/>

# 如何为高级 Azure Redis 缓存配置 Redis 群集功能
Azure Redis 缓存具有不同的缓存产品（包括新推出的高级层），使缓存大小和功能的选择更加灵活。本文介绍如何配置高级 Azure Redis 缓存实例中的群集功能。


## 什么是 Redis 群集？
Azure Redis 缓存提供的 Redis 群集与[在 Redis 中实施](http://redis.io/topics/cluster-tutorial)的一样。Redis 群集具有以下优势。

-	能够在多个节点中自动拆分数据集。 
-	能够在部分节点遇到故障或无法与群集其余部分通信的情况下继续运行。 
-	更大的吞吐量：增加分片数时，吞吐量呈线性增加。 
-	更大的内存大小：增加分片数时，内存大小呈线性增加。  

有关高级缓存大小、吞吐量和带宽的更多详细信息，请参阅 [Azure Redis 缓存常见问题](/documentation/articles/cache-faq/#what-redis-cache-offering-and-size-should-i-use)。

在 Azure 中，Redis 群集以主/副模型提供。在该模型中，每个分片都有一个带副本的主/副对，副本由 Azure Redis 缓存服务管理。

## 群集功能

在 Azure 中国区，只能通过 Azure PowerShell 或 Azure CLI 管理 Redis 缓存


[AZURE.INCLUDE [azurerm-azurechinacloud-environment-parameter](../includes/azurerm-azurechinacloud-environment-parameter.md)]


使用以下 PowerShell 脚本创建缓存：

	$VerbosePreference = "Continue"

	# Create a new cache with date string to make name unique. 
	$cacheName = "MovieCache" + $(Get-Date -Format ('ddhhmm')) 
	$location = "China North"
	$resourceGroupName = "Default-Web-ChinaNorth"
	
	$movieCache = New-AzureRmRedisCache -Location $location -Name $cacheName  -ResourceGroupName $resourceGroupName -Size 6GB -Sku Premium -ShardCount 2

群集中最多可以包含 10 个分片，上述脚本将创建“分片计数”为 2（可以是介于 1 和 10 之间的数字）的缓存。

每个分片都是一个由 Azure 管理的主/副缓存对，而缓存的总大小则通过将定价层中选择的缓存大小乘以分片数来计算。

创建缓存后，即可连接到缓存并使用缓存，就像该缓存没有进行群集一样，而 Redis 则会将数据分布到整个缓存分片中。

有关在 StackExchange.Redis 客户端中使用群集的示例代码，请参阅 [Hello World](https://github.com/rustd/RedisSamples/tree/master/HelloWorld) 示例的 [clustering.cs](https://github.com/rustd/RedisSamples/blob/master/HelloWorld/Clustering.cs) 部分。

<a name="move-exceptions"></a>
>[AZURE.IMPORTANT]使用 StackExchange.Redis 连接已启用群集的 Azure Redis 缓存时，你可能会遇到问题并收到 `MOVE` 异常。这是因为 StackExchange.Redis 缓存客户端收集缓存群集中节点信息的间隔时间较短。如果你是第一次连接缓存，并在客户端完成收集此信息前立即调用缓存，便会发生这些异常。在应用程序中解决此问题的最简单方法是连接缓存，然后等待一秒后再对缓存进行任何调用。为此，请按以下示例代码中所示添加 `Thread.Sleep(1000)`。请注意，只会在首次连接到缓存期间发生 `Thread.Sleep(1000)`。有关详细信息，请参阅 [StackExchange.Redis.RedisServerException - MOVED #248](https://github.com/StackExchange/StackExchange.Redis/issues/248)。我们正在开发此问题的修复程序，如有任何更新，将发布在本页面。**更新**: 这个问题已经在最新版本 [prerelease 1.1.572-alpha](https://www.nuget.org/packages/StackExchange.Redis/1.1.572-alpha) StackExchange.Redis 中解决。 在 [StackExchange.Redis NuGet 页面](https://www.nuget.org/packages/StackExchange.Redis/) 中查看最新版本。 


	private static Lazy<ConnectionMultiplexer> lazyConnection = new Lazy<ConnectionMultiplexer>(() =>
	{
        // Connect to the Redis cache for the first time
	    var connection =  ConnectionMultiplexer.Connect("contoso5.redis.cache.chinacloudapi.cn,abortConnect=false,ssl=true,password=...");

		// Wait for 1 second
		Thread.Sleep(1000);

		// Return the connected ConnectionMultiplexer
		return connection;
	});
	
	public static ConnectionMultiplexer Connection
	{
	    get
	    {
	        return lazyConnection.Value;
	    }
	}

<a name="cluster-size"></a>
## 更改正在运行的高级缓存上的群集大小

可以使用以下命令更改正在运行的、已启用群集的高级缓存上的群集大小。

	New-AzureRmRedisCache -Name $cacheName -ResourceGroupName $resourceGroupName -Size 13GB

完成缩放需要花费几分钟的时间。有关详细信息，请参阅[如何缩放 Azure Redis 缓存](/documentation/articles/cache-how-to-scale/)。

## 群集功能常见问题

以下列表包含有关 Azure Redis 缓存群集功能常见问题的解答。

## 使用群集功能时，是否需要对客户端应用程序进行更改？

-	启用群集功能时，仅数据库 0 可用。如果你的客户端应用程序使用多个数据库并尝试读取或写入数据库 0 之外的其他数据库，则会引发以下异常。`Unhandled Exception: StackExchange.Redis.RedisConnectionException: ProtocolFailure on GET --->` `StackExchange.Redis.RedisCommandException: Multiple databases are not supported on this server; cannot switch to database: 6`
-	如果你使用的是 [StackExchange.Redis](https://www.nuget.org/packages/StackExchange.Redis/)，则必须使用 1.0.481 或更高版本。连接到该缓存时，你使用的终结点、端口和密钥与你连接到未启用群集功能的缓存时使用的相同。唯一的区别是，所有读取和写入都必须在数据库 0 中进行。
	-	其他客户端可能有不同的要求。请参阅[是否所有 Redis 客户端都支持群集功能？](#do-all-redis-clients-support-clustering)
-	如果应用程序使用的多个密钥操作都在单个命令中成批执行，则所有密钥都必须位于同一分片。若要完成此操作，请参阅[密钥在群集中是如何分布的？](#how-are-keys-distributed-in-a-cluster)。
-	如果你使用的是 Redis ASP.NET 会话状态提供程序，则必须使用 2.0.1 或更高版本。请参阅[能否在 Redis ASP.NET 会话状态和输出缓存提供程序中使用群集功能？](#can-i-use-clustering-with-the-redis-aspnet-session-state-and-output-caching-providers)。

##<a name="how-are-keys-distributed-in-a-cluster"></a> 密钥在群集中是如何分布的？

请参阅 Redis [密钥分布模型](http://redis.io/topics/cluster-spec#keys-distribution-model)文档：密钥空间拆分成 16384 个槽。每个密钥都经过哈希处理并分配到其中一个槽，这些槽分布在群集的节点中。对密钥的哪部分进行哈希处理是可以配置的，这样可确保多个使用哈希标记的密钥位于同一分片。

-	使用哈希标记的密钥 - 如果将密钥的任意部分括在 `{` 和 `}` 中，则只会对密钥的该部分进行哈希处理，以便确定密钥的哈希槽。例如，以下 3 个密钥将位于同一分片中：`{key}1`、`{key}2` 和 `{key}3`，因为只对名称的 `key` 部分进行了哈希处理。如需密钥哈希标记规范的完整列表，请参阅[密钥哈希标记](http://redis.io/topics/cluster-spec#keys-hash-tags)。
-	没有哈希标记的密钥 - 使用整个密钥名称进行哈希处理。从统计学意义上来说，这样会导致密钥平均分布到缓存的各个分片中。

为了优化性能和吞吐量，我们建议你让密钥平均分布。如果你使用带哈希标记的密钥，则应用程序会负责确保密钥平均分布。

有关详细信息，请参阅[密钥分布模型](http://redis.io/topics/cluster-spec#keys-distribution-model)、[Redis 群集数据分片](http://redis.io/topics/cluster-tutorial#redis-cluster-data-sharding)和[密钥哈希标记](http://redis.io/topics/cluster-spec#keys-hash-tags)。

有关在 StackExchange.Redis 客户端中使用群集和查找同一分片中的密钥的示例代码，请参阅 [Hello World](https://github.com/rustd/RedisSamples/tree/master/HelloWorld) 示例的 [clustering.cs](https://github.com/rustd/RedisSamples/blob/master/HelloWorld/Clustering.cs) 部分。

## 我可以创建的最大缓存大小是多大？

高级缓存的最大大小为 53 GB。你可以创建多达 10 个分片，因此最大大小为 530 GB。如果你需要的大小更大，则可[请求更多](mailto:wapteams@microsoft.com?subject=Redis%20Cache%20quota%20increase)。有关详细信息，请参阅 [Azure Redis 缓存定价](/home/features/redis-cache/pricing/)。

##<a name="do-all-redis-clients-support-clustering"></a> 是否所有 Redis 客户端都支持群集功能？

目前，并非所有客户端都支持 Redis 群集功能。StackExchange.Redis 是不支持该功能的客户端。有关其他客户端的详细信息，请参阅 [Redis 群集教程](http://redis.io/topics/cluster-tutorial)的[操作群集](http://redis.io/topics/cluster-tutorial#playing-with-the-cluster)部分。

>[AZURE.NOTE]如果你使用 StackExchange.Redis 作为客户端，请确保使用最新版本的 [StackExchange.Redis](https://www.nuget.org/packages/StackExchange.Redis/)，即 1.0.481 或更高，以便群集功能能够正常使用。如果你对 `Move` 异常有任何问题，请查看 [move 异常](#move-exceptions)。


## 启用群集功能后，如何连接到我的缓存？

连接到你的缓存时，可以使用的终结点、端口和密钥与你连接到未启用群集功能的缓存时使用的相同。Redis 在后端管理群集功能，因此不需要你通过客户端来管理它。

## 我可以直接连接到缓存的各个分片吗？

尚未正式提供此方面的支持。话虽如此，但每个分片都是由主/副缓存对组成的，该缓存对统称缓存实例。你可以在 GitHub 上通过 Redis 存储库的[不稳定](http://redis.io/download)分支使用 redis-cli 实用程序连接到这些缓存实例。使用 `-c` 开关启动后，此版本可实现基本的支持。有关详细信息，请参阅 [http://redis.io](http://redis.io) 上的 [Redis 群集教程](http://redis.io/topics/cluster-tutorial)中的[操作群集](http://redis.io/topics/cluster-tutorial#playing-with-the-cluster)。

对于非 ssl，请使用以下命令。

	Redis-cli.exe -h <<cachename>> -p 13000 (to connect to instance 0)
	Redis-cli.exe -h <<cachename>> -p 13001 (to connect to instance 1)
	Redis-cli.exe -h <<cachename>> -p 13002 (to connect to instance 2)
	...
	Redis-cli.exe -h <<cachename>> -p 1300N (to connect to instance N)

对于 ssl，请将 `1300N` 替换为 `1500N`。

## 我可以为以前创建的缓存配置群集功能吗？

目前，你只能在创建缓存时启用群集。创建缓存后，你可以更改群集大小，但不能将群集添加到高级缓存，或者从高级缓存中删除群集。已启用群集且只包含一个分片的高级缓存不同于具有相同大小且没有群集的高级缓存。

## 我可以为基本缓存或标准缓存配置群集功能吗？

群集功能仅适用于高级缓存。

##<a name="can-i-use-clustering-with-the-redis-aspnet-session-state-and-output-caching-providers"></a> 能否在 Redis ASP.NET 会话状态和输出缓存提供程序中使用群集功能？

-	**Redis 输出缓存提供程序** - 无需进行更改。
-	**Redis 会话状态提供程序** - 若要使用群集功能，必须使用 [RedisSessionStateProvider](https://www.nuget.org/packages/Microsoft.Web.RedisSessionStateProvider) 2.0.1 或更高版本，否则会引发异常。这是一项重大更改；有关详细信息，请参阅 [2\.0.0 版重大更改详细信息](https://github.com/Azure/aspnet-redis-providers/wiki/v2.0.0-Breaking-Change-Details)。

  
<!-- IMAGES -->

[redis-cache-new-cache-menu]: ./media/cache-how-to-premium-clustering/redis-cache-new-cache-menu.png

[redis-cache-premium-pricing-tier]: ./media/cache-how-to-premium-clustering/redis-cache-premium-pricing-tier.png

[NewCacheMenu]: ./media/cache-how-to-premium-clustering/redis-cache-new-cache-menu.png

[CacheCreate]: ./media/cache-how-to-premium-clustering/redis-cache-cache-create.png

[redis-cache-premium-pricing-group]: ./media/cache-how-to-premium-clustering/redis-cache-premium-pricing-group.png

[redis-cache-premium-features]: ./media/cache-how-to-premium-clustering/redis-cache-premium-features.png

[redis-cache-clustering]: ./media/cache-how-to-premium-clustering/redis-cache-clustering.png

[redis-cache-clustering-selected]: ./media/cache-how-to-premium-clustering/redis-cache-clustering-selected.png

[redis-cache-redis-cluster-size]: ./media/cache-how-to-premium-clustering/redis-cache-redis-cluster-size.png

<!---HONumber=Mooncake_0104_2016-->