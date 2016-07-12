<properties 
	pageTitle="Azure Redis Cache 示例" 
	description="了解如何使用 Azure Redis Cache" 
	services="redis-cache" 
	documentationCenter="" 
	authors="steved0x" 
	manager="dwrede" 
	editor=""/>

<tags 
	ms.service="cache" 
	ms.date="03/22/2016" 
	wacn.date="05/24/2016"/>

# Azure Redis Cache 示例 

本主题提供了 Azure Redis Cache 示例的列表，涵盖了诸如连接到缓存、从缓存中读取数据以及向其中写入数据、以及使用 ASP.NET Redis Cache 提供程序之类的方案。有些示例是可下载的项目，有些示例提供了分步指南并包含代码片段但没有链接到可下载的项目。

## Hello world 示例

本部分中的示例显示了使用各种语言和 Redis 客户端连接到 Azure Redis Cache 实例、从缓存中读取数据以及向其中写入数据方面的基础知识。

[Hello world](https://github.com/rustd/RedisSamples/tree/master/HelloWorld) 示例展示了如何使用 [StackExchange.Redis](https://github.com/StackExchange/StackExchange.Redis) .NET 客户端执行不同的缓存操作。

此示例演示如何：

-	使用不同的连接选项
-	使用同步和异步操作与缓存相互读取和写入对象
-	使用 Redis MGET/MSET 命令返回指定键的值
-	执行 Redis 事务操作
-	处理 Redis 列表和排序集
-	使用 JsonConvert 序列化程序存储.NET 对象
-	使用 Redis 集实现标记

有关详细信息，请参阅 github 上的 [StackExchange.Redis](https://github.com/StackExchange/StackExchange.Redis)；有关更多的使用方案，请参阅 [StackExchange.Redis.Tests](https://github.com/StackExchange/StackExchange.Redis/tree/master/StackExchange.Redis.Tests) 单位测试。

[如何将 Azure Redis 缓存与 Python 配合使用](/documentation/articles/cache-python-get-started/)展示了如何使用 Python 和 [redis-py](https://github.com/andymccurdy/redis-py) 客户端开始使用 Azure Redis 缓存。

[在缓存中处理 .NET 对象](/documentation/articles/cache-dotnet-how-to-use-azure-redis-cache/#working-with-caches)演示了如何对 .NET 对象进行序列化以便可以将其写入到 Azure Redis 缓存实例以及从中进行读取。

## 将 Redis Cache 用作 ASP.NET SignalR 的扩展基架

[将 Redis 缓存用作 ASP.NET SignalR 的扩展基架](https://github.com/rustd/RedisSamples/tree/master/RedisAsSignalRBackplane)示例演示了如何将 Azure Redis 缓存用作 SignalR 基架。有关基架的更多信息，请参阅[采用 Redis 的 SignalR 扩展](http://www.asp.net/signalr/overview/performance/scaleout-with-redis)。

## Redis Cache 客户查询示例

此示例对从缓存访问数据与从持久存储访问数据时的性能进行了比较。此示例有两个项目。

-	[展示 Redis Cache 如何通过对数据进行缓存提高性能](https://github.com/rustd/RedisSamples/tree/master/RedisCacheCustomerQuerySample)
-	[为进行展示创立数据库和缓存](https://github.com/rustd/RedisSamples/tree/master/SeedCacheForCustomerQuerySample)

## ASP.NET 会话状态和输出缓存

[使用 Azure Redis 缓存存储 ASP.NET SessionState 和 OutputCache](https://github.com/rustd/RedisSamples/tree/master/SessionState_OutputCaching) 示例演示了如何使用 Azure Redis 缓存通过为 Redis 使用 SessionState 和 OutputCache 提供程序来存储 ASP.NET 会话和输出缓存。

## 使用 MAML 管理 Azure Redis Cache

[使用 Azure Management Libraries 管理 Azure Redis 缓存](https://github.com/rustd/RedisSamples/tree/master/ManageCacheUsingMAML)示例展示了如何使用 Azure Management Libraries 来管理（创建/更新/删除）你的缓存。

## 自定义监视示例

[访问 Redis 缓存监视数据](https://github.com/rustd/RedisSamples/tree/master/CustomMonitoring)示例演示了如何访问你的 Azure Redis 缓存的监视数据。

## 使用 PHP 和 Redis 编写的 Twitter 式克隆

[Retwis](https://github.com/SyntaxC4-MSFT/retwis) 示例是 Redis Hello World。它是最小的 Twitter 样式的社交网络克隆使用 Redis 和 PHP 编写使用 [Predis](https://github.com/nrk/predis) 客户端。源代码旨在是非常简单，并且在同一时间以显示其他 Redis 数据结构。

## 带宽监视器

[带宽监视器](https://github.com/JonCole/SampleCode/tree/master/BandWidthMonitor)示例允许你监视客户端上使用的带宽。若要测量带宽、请在缓存客户端计算机上运行该示例，对缓存执行调用，然后观察带宽监视器示例报告的带宽。

<!---HONumber=76-->