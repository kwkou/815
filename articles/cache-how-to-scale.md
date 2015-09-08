<properties 
	pageTitle="如何缩放 Azure Redis 缓存" 
	description="了解如何缩放你的 Azure Redis 缓存实例" 
	services="redis-cache" 
	documentationCenter="" 
	authors="steved0x" 
	manager="dwrede" 
	editor=""/>

<tags 
	ms.service="cache" 
	ms.date="06/18/2015" 
	wacn.date="08/29/2015"/>

# 如何缩放 Azure Redis 缓存

>[AZURE.NOTE]Azure Redis 缓存缩放功能目前处于预览状态。

Azure Redis 缓存具有不同的缓存产品/服务，使缓存大小和功能的选择更加灵活。如果创建缓存后，你的应用程序的要求发生更改，可以使用 [Azure 门户](https://manage.windowsazure.cn)中的“更改定价层”边栏选项卡缩放缓存的大小。

>[AZURE.NOTE]当你缩放 Azure Redis 缓存时可以更改大小，但不能将缓存从“标准”更改为“基本”，反之亦然。

## 何时缩放

你可以使用 Azure Redis 缓存的[监视](/documentation/articles/cache-how-to-monitor)功能来监视你的缓存应用程序的运行状况和性能，并帮助确定是否需要缩放缓存。

你可以监视以下指标以帮助确定是否需要进行缩放。

-	Redis 服务器负载
-	内存用量
-	网络带宽
-	CPU 使用率

如果确定缓存不再满足应用程序的要求，可以更改到适合应用程序的更大或更小缓存定价层。有关确定应使用哪个缓存定价层的详细信息，请参阅[我应当使用哪些 Redis 缓存产品/服务和大小](/documentation/articles/cache-faq#what-redis-cache-offering-and-size-should-i-use)。

## 缩放缓存
要缩放缓存，在 [Azure 门户](https://manage.windowsazure.cn)中[浏览到缓存](/documentation/articles/cache-configure/)，单击“Redis 缓存”边栏选项卡中的“标准层”或“基本层”部分。

![定价层][redis-cache-pricing-tier-part]

从“定价层”边栏选项卡选择所需的定价层，然后单击“选择”。

![定价层][redis-cache-pricing-tier-blade]

>[AZURE.NOTE]缓存不能从“基本”改为“标准”，反之亦然，你也不能将缓存从更大的大小减少到 250 MB。你可以将缓存从 250 MB 增加到更大大小，但不能缩回到 250MB 定价层。如果需要从“基本”更改为“标准”，或减少到 250 MB 大小，则必须创建新的缓存。

当缓存缩放到新的定价层，将在“Redis 缓存”边栏选项卡中显示**缩放**状态。

![扩展][redis-cache-scaling]

缩放完成后，状态将从**缩放**更改为**运行**。

>[AZURE.IMPORTANT]在缩放操作期间，“基本”缓存处于脱机状态，且所有缓存中的数据都将丢失。缩放操作一完成，“基本”缓存将重新联机，且不含任何数据。“标准”缓存在缩放操作中保持联机状态，当将“标准”缓存扩展至更大大小时通常不会丢失数据。当将“标准”缓存缩小到更小大小时，如果新的大小小于缓存的数据量则可能丢失某些数据。如果缩小时数据丢失，则使用 [allkeys lru](http://redis.io/topics/lru-cache) 逐出策略逐出密钥。注意，当“标准”缓存有 99.9% 可用的 SLA 时，则没有用于数据丢失的 SLA。

## 如何自动执行缩放操作

除了在 Azure 门户中缩放你的 Azure Redis 缓存实例，你还可以使用 <!--[-->Windows Azure 管理库 (MAML)<!--](http://azure.microsoft.com/updates/management-libraries-for-net-release-announcement/) -->进行缩放。要缩放你的缓存，请调用 `IRedisOperations.CreateOrUpdate` 方法并传入 `RedisProperties.SKU.Capacity` 的新大小。

    static void Main(string[] args)
    {
        // For instructions on getting the access token, see
        // https://msdn.microsoft.com/zh-cn/library/azure/dn790557.aspx#bk_portal
        string token = GetAuthorizationHeader();

        TokenCloudCredentials creds = new TokenCloudCredentials(subscriptionId,token);

        RedisManagementClient client = new RedisManagementClient(creds);
        var redisProperties = new RedisProperties();

        // To scale, set a new size for the redisSKUCapacity parameter.
        redisProperties.Sku = new Sku(redisSKUName,redisSKUFamily,redisSKUCapacity);
        redisProperties.RedisVersion = redisVersion;
        var redisParams = new RedisCreateOrUpdateParameters(redisProperties, redisCacheRegion);
        client.Redis.CreateOrUpdate(resourceGroupName,cacheName, redisParams);
    }

有关详细信息，请参阅[使用 MAML 管理 Redis 缓存](https://github.com/rustd/RedisSamples/tree/master/ManageCacheUsingMAML)示例。

## 关于缩放的常见问题

以下列表包含有关 Azure Redis 缓存缩放的常见问题的解答。

## 缩放后，我是否需要更改缓存名称或访问密钥

不需要，在缩放操作期间缓存名称和密钥不变。

## 缩放的工作原理

对一个**基本**缓存进行缩放后，将关闭该缓存，同时使用新的大小设置一个新的缓存。在此期间，缓存不可用，且缓存中的所有数据都将丢失。

对一个**标准**缓存进行缩放后，将关闭其中一个副本，同时将其重新设置为新的大小，将数据转移，然后，在重新设置另一个副本之前，另一个副本将执行一次故障转移，类似于一个缓存节点发生故障时所发生的过程。

## 在缩放过程中是否会丢失缓存上的数据

缩放**基本**缓存后，所有数据都将丢失，且缩放操作期间缓存不可用。

将**标准**缓存扩展到更大大小时，通常将保留所有数据。将**标准**缓存缩小到更小大小时，数据可能会丢失，具体取决于与缩放后的新大小相关的缓存中的数据量。如果缩小时数据丢失，则使用 [allkeys lru](http://redis.io/topics/lru-cache) 逐出策略逐出密钥。注意，当“标准”缓存有 99.9% 可用的 SLA 时，则没有用于数据丢失的 SLA。

## 缩放时缓存是否可用

**标准**缓存在缩放操作期间保持可用。

**基本**缓存在缩放操作期间处于脱机状态。

## 不支持的操作

你不能在缩放操作过程中将**基本**缓存更改为**标准**缓存，反之亦然。

你可以从 **C0** (250 MB) 缓存增加到更大大小，但是你不能从更大大小缩回到 **C0** 缓存。

如果缩放操作失败，该服务将尝试还原操作并且缓存将还原为初始大小。

## 缩放需要多长时间

缩放大约需要 20 分钟，具体取决于缓存中的数据量。

## 如何判断缩放何时完成

在门户中你可以看到进行中的缩放操作。缩放完成后，缓存状态将更改为**正在运行**。

## 为什么该功能在预览中

我们现在发布此功能是为了获取反馈。基于该反馈，我们将很快发布此功能的正式版本。





  
<!-- IMAGES -->
[redis-cache-pricing-tier-part]: ./media/cache-how-to-scale/redis-cache-pricing-tier-part.png

[redis-cache-pricing-tier-blade]: ./media/cache-how-to-scale/redis-cache-pricing-tier-blade.png

[redis-cache-scaling]: ./media/cache-how-to-scale/redis-cache-scaling.png

<!---HONumber=67-->