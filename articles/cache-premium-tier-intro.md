<properties 
	pageTitle="Azure Redis 缓存高级层简介" 
	description="了解如何为高级层 Azure Redis 缓存实例创建和管理 Redis 持久性、Redis 群集和 VNET 支持" 
	services="redis-cache" 
	documentationCenter="" 
	authors="steved0x" 
	manager="dwrede" 
	editor=""/>

<tags
	ms.service="cache"
	ms.date="04/27/2016"
	wacn.date="06/29/2016"/>

# Azure Redis 缓存高级层简介
Azure Redis 缓存是一种分布式托管缓存，可通过提供对数据的超高速访问权限来帮助你构建高度可缩放且响应灵敏的应用程序。

新的高级层可供企业立即使用，除了标准层的所有功能以外，它还包括其他优势，例如更好的性能、更大的工作负荷、灾难恢复和增强的安全性。请继续阅读，以深入了解高级缓存层的其他功能。

## 与标准或基本层相比性能更佳
**性能优于标准或基本层。** 相对于基本级别或标准级别，高级层的缓存部署在处理器速度更快且性能更高的硬件上。高级级别缓存的吞吐量更高，延迟更低。

**如果缓存大小相同，则高级级别中的吞吐量高于标准级别。** 例如：53 GB 的缓存，其 P4（高级层）的吞吐量是每秒 250K 个请求，相比之下，C6（标准层）只有 150K。

有关高级缓存大小、吞吐量和带宽的更多详细信息，请参阅 [Azure Redis 缓存常见问题](/documentation/articles/cache-faq/#what-redis-cache-offering-and-size-should-i-use)。

## Redis 数据持久性
高级层允许你将缓存数据暂留在 Azure 存储空间帐户中。在基本/标准缓存中，所有数据只存储在内存中。如果底层基础结构出现问题，可能会导致数据丢失。我们建议使用高级级别中的 Redis 数据暂留功能来增加灵活性，防止数据丢失。Azure Redis 缓存提供可在 [Redis 持久性](http://redis.io/topics/persistence)中使用的 RDB 和 AOF（即将推出）选项。

有关配置持久性的说明，请参阅[如何为高级 Azure Redis 缓存配置持久性](/documentation/articles/cache-how-to-premium-persistence/)。

##Redis 群集
如果你想要创建大于 53 GB 的缓存，或者想要将数据通过分片的方式分散到多个 Redis 节点中，则可以使用在高级层中提供的 Redis 群集功能。每个节点都包含一个由 Azure 管理的主/副缓存对，目的是提高可用性。

**Redis 群集可提供最大的缩放能力和吞吐量。** 增加群集中分片（节点）的数量会导致吞吐量线性提高。例如，如果你创建了一个包含 10 个分片的 P4 群集，则可用吞吐量为 250K * 10 = 每秒 250 万个请求。有关高级缓存大小、吞吐量和带宽的更多详细信息，请参阅 [Azure Redis 缓存常见问题](/documentation/articles/cache-faq/#what-redis-cache-offering-and-size-should-i-use)。

若要开始使用群集，请参阅[如何为高级 Azure Redis 缓存配置群集功能](/documentation/articles/cache-how-to-premium-clustering/)。

##增强的安全性和隔离性
可通过公共 Internet 访问基本或标准层中创建的缓存。根据访问密钥来限制对缓存的访问。使用高级层可以进一步确保只有指定网络中的客户端可以访问缓存。可以在 [Azure 虚拟网络 (VNet)](/home/features/networking/) 中部署 Redis 缓存。可以使用 VNet 的所有功能，例如子网、访问控制策略和其他功能，进一步限制对 Redis 的访问。

有关详细信息，请参阅[如何为高级 Azure Redis 缓存配置虚拟网络支持](/documentation/articles/cache-how-to-premium-vnet/)。

## 后续步骤

创建缓存并探索高级层的新功能。

-	[如何为高级 Azure Redis 缓存配置暂留](/documentation/articles/cache-how-to-premium-persistence/)
-	[如何为高级 Azure Redis 缓存配置虚拟网络支持](/documentation/articles/cache-how-to-premium-vnet/)
-	[如何为高级 Azure Redis 缓存配置群集功能](/documentation/articles/cache-how-to-premium-clustering/)
  


<!---HONumber=Mooncake_0321_2016-->