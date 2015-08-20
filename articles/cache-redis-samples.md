<properties 
	pageTitle="Azure Redis Cache 示例" 
	description="了解如何使用 Azure Redis Cache" 
	services="redis-cache" 
	documentationCenter="" 
	authors="steved0x" 
	manager="dwrede" 
	editor=""/>
<tags ms.service="redis-cache"
    ms.date="03/16/2015"
    wacn.date="04/11/2015"
    />

# Azure Redis Cache 示例 

本主题提供了 Azure Redis Cache 示例的列表，涵盖了诸如连接到缓存、从缓存中读取数据以及向其中写入数据、以及使用 ASP.NET Redis Cache 提供程序之类的方案。有些示例是可下载的项目，有些示例提供了分步指南并包含代码片段但没有链接到可下载的项目。

## Hello world 示例

本部分中的示例显示了使用各种语言和 Redis 客户端连接到 Azure Redis Cache 实例、从缓存中读取数据以及向其中写入数据方面的基础知识。

[Hello world](https://github.com/rustd/RedisSamples/tree/master/HelloWorld) 示例展示了如何使用 [StackExchange.Redis](https://github.com/StackExchange/StackExchange.Redis) .NET 客户端从缓存中读取项以及向其中写入项。

[如何将 Azure Redis Cache 与 Node.js 配合使用](/documentation/articles/cache-nodejs-get-started)展示了在使用 Node.js 和 [node_redis](https://github.com/mranney/node_redis) 客户端的情况下使用 Azure Redis Cache 的入门知识。

[如何将 Azure Redis Cache 与 Java 配合使用](/documentation/articles/cache-java-get-started)展示了在使用 Java 和 [Jedis](https://github.com/xetorthio/jedis) 客户端的情况下使用 Azure Redis Cache 的入门知识。

[如何将 Azure Redis Cache 与 Python 配合使用](/documentation/articles/cache-python-get-started)展示了在使用 Python 和 [redis-py](https://github.com/andymccurdy/redis-py) 客户端的情况下使用 Azure Redis Cache 的入门知识。

[PHP 示例](https://msdn.microsoft.com/zh-CN/library/azure/dn690470.aspx#PHPExample)展示了将 Azure Redis Cache 与 PHP 和 [predis](https://github.com/nrk/predis) 客户端配合使用的入门知识。

[在缓存中处理 .NET 对象](https://msdn.microsoft.com/zh-CN/library/azure/dn690521.aspx#Objects)展示了如何对 .NET 对象进行序列化以便可以将其写入到 Azure Redis Cache 实例以及从中进行读取。

## 将 Redis Cache 用作 ASP.NET SignalR 的扩展基架

[将 Redis Cache 用作 ASP.NET SignalR 的扩展基架](https://github.com/rustd/RedisSamples/tree/master/RedisAsSignalRBackplane)示例展示了如何将 Azure Redis Cache 用作 SignalR 基架。有关基架的更多信息，请参阅[采用 Redis 的 SignalR 扩展](http://www.asp.net/signalr/overview/performance/scaleout-with-redis)。

## Redis Cache 客户查询示例

此示例对从缓存访问数据与从持久存储访问数据时的性能进行了比较。此示例有两个项目。

-	[展示 Redis Cache 如何通过对数据进行缓存提高性能](https://github.com/rustd/RedisSamples/tree/master/RedisCacheCustomerQuerySample)
-	[为进行展示创立数据库和缓存](https://github.com/rustd/RedisSamples/tree/master/SeedCacheForCustomerQuerySample)

## ASP.NET 会话状态和输出缓存

[使用 Azure Redis Cache 存储 ASP.NET SessionState 和 OutputCache](https://github.com/rustd/RedisSamples/tree/master/SessionState_OutputCaching) 示例展示了如何使用 Azure Redis Cache 通过为 Redis 使用 SessionState 和 OutputCache 提供程序来存储 ASP.NET 会话和输出缓存。

## 使用 MAML 管理 Azure Redis Cache

[使用 Azure Management Libraries 管理 Azure Redis Cache](https://github.com/rustd/RedisSamples/tree/master/ManageCacheUsingMAML) 示例展示了如何使用 Azure Management Libraries 来管理（创建/更新/删除）你的缓存。 

## 自定义监视示例

[访问 Redis Cache 监视数据](https://github.com/rustd/RedisSamples/tree/master/CustomMonitoring)示例展示了如何在 Azure 门户外访问你的 Azure Redis Cache 的监视数据。




<!--HONumber=51-->