<properties title="Sharding Overview" pageTitle="分片概述" description="分片理由：扩展数据库资源以增强可用性或性能。" metaKeywords="sharding, scaling, elastic scale, Azure SQL Database" services="sql-database" documentationCenter="" manager="jhubbard" authors="sidneyh@microsoft.com"/>


#分片概述 

##分片原则 

**分片**是一项可跨许多独立的数据库分发大量相同结构数据的技术。这项技术尤其受到为最终客户或企业创建软件即服务 (SAAS) 产品的云开发人员的欢迎。这些最终客户通常称为"租户"。需要分片的原因有很多： 

* 数据总量过大而超出单个数据库的限制范围 
* 整个工作负载的事务吞吐量超出单个数据库的容量 
* 租户可能需要与其他租户物理隔离，因此每个租户都需要单独的数据库 
* 由于合规性、性能或地理政治的原因，不同的数据库部分可能需要驻留在不同的地域中。 
 
当应用程序中的每个事务均受限于分片键的单个值时，分片效果最佳。这将确保所有事务都将本地放置在特定数据库中。 

某些应用程序使用最简单的方法为每个租户创建一个单独的数据库。这就是**单租户分片模式**，该模式提供了隔离、备份/还原功能以及根据租户粒度进行的资源缩放。借助单租户分片，每个数据库都将与特定租户 ID 值（或客户键值）关联，而该键无需始终出现在数据本身中。应用程序负责将每个请求路由到相应的数据库。 

![][1]

其他方案是将多个租户一同打包到数据库中，而非将其隔离到单独的数据库中。这是典型的**多租户分片模式**，该模式的使用可能取决于对成本、效率或应用程序可管理大量极小型租户这一情况的考虑。在多租户分片中，数据库表中的行全都被设计为带有可标识租户 ID 的键或分片键。同样，应用程序层负责将租户的请求路由到相应的数据库。 

在其他方案（例如从分布式设备中引入数据）中，分片可用于填充某个时间段内分发的一组数据库。例如，可以在每天或每周专门使用某个单独的数据库。在此情况下，分片键可以是一个表示日期的整数（显示在分片表的所有行中），应用程序必须将用于检索有关日期范围的信息的查询路由到涉及相关范围的数据库子集。

无论使用何种分片模型，一个特殊的数据结构（称为**分片映射**）都可以用作查找表，该表将分片键值与数据库相关联；这使应用程序可以为数据库请求执行路由。这称为**数据依赖路由**，它是应用程序使用分片数据层时所需的核心功能。[灵活扩展客户端 API](/zh-cn/documentation/articles/sql-database-elastic-scale-introduction/) 提供了[管理分片映射](/zh-cn/documentation/articles/sql-database-elastic-scale-shard-map-management/)和启用应用程序中有效且易于使用的[数据依赖路由功能](/zh-cn/documentation/articles/sql-database-elastic-scale-data-dependent-routing/)时所需的一组丰富功能。 

[AZURE.INCLUDE [elastic-scale-include](../includes/elastic-scale-include.md)]

<!--Image references-->
[1]: ./media/sql-database-elastic-scale-sharding-overview/tenancy.png
