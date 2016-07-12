<properties 
   pageTitle="Azure Redis 缓存的常见缓存模式" 
   description="了解可在何处以及为何要使用 Azure Redis 缓存" 
   services="redis-cache" 
   documentationCenter="" 
   authors="Rick-Anderson" 
   manager="wpickett" 
   editor=""/>

<tags
   ms.service="cache"
   ms.date="02/23/2016"
   wacn.date="04/26/2016"/>

# Azure Redis 缓存的常见缓存模式

此页列出了缓存带来的最常见优势。

## 使用缓存优化数据访问

从数据存储中提取数据时，使用缓存可以明显加快数据访问。缓存提供高吞吐量和低延迟。通过从缓存提取热数据，不仅可以提高应用程序的速度，而且还可以降低数据访问负载，提高应用程序响应其他查询的速度。在缓存中存储信息有助于节省资源，并在应用程序需求增大时提高可缩放性。应用程序将会大大提高应对突发性负载的能力，同时还能有效地从缓存中提取数据。

## 分布式会话状态
尽管我们普遍认为最好是避免使用会话状态，但实际上，一些应用程序使用会话数据时可以带来性能提高/复杂性降低的好处，还有一些应用程序迫切地需要使用会话状态。会话状态的默认内存中提供程序不允许扩大（运行 Web 应用的多个实例）。ASP.NET SQL Server 会话状态提供程序允许多个 Web 应用使用会话状态，但付出的代价是比内存中提供程序的延迟更高。Redis 会话状态缓存提供程序是一种低延迟的替代项，它很容易配置和设置。如果你的应用程序只会使用有限数量的会话状态，则你可以使用大部分缓存来缓存数据，使用少量的缓存来缓存会话状态。

## 发生服务停机时得到幸存（缓存回退）
 如果将数据存储在缓存中，在发生系统故障（如网络延迟、Web 服务问题和硬件故障）时，应用程序也许能够幸存。在 Web 服务或数据库恢复之前不断地提供缓存的数据，往往比应用程序完全发生故障要好。

## 后续步骤
了解有关使用 Azure Redis 缓存的详细信息：
 
- [Redis Azure 缓存文档](/documentation/services/redis-cache/)：此页提供有关使用 Redis Azure 缓存的许多优秀文章链接。
- [15 分钟学会创建包含 Azure Redis 缓存的 MVC 影片应用程序](http://azure.microsoft.com/blog/2014/06/05/mvc-movie-app-with-azure-redis-cache-in-15-minutes/)：该博客文章提供有关在 ASP.NET MVC 应用程序中使用 Azure Redis 缓存的快速入门。
- [如何将 ASP.NET 会话状态用于 Azure Web 应用](/documentation/articles/web-sites-dotnet-session-state-caching/)：此主题说明如何为会话状态使用 Azure Redis 缓存服务。

<!---HONumber=71-->