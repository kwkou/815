<properties title="Azure SQL Elastic Scale FAQ" pageTitle="Azure SQL 灵活扩展常见问题" description="关于 Azure SQL Database 灵活扩展的常见问题。" metaKeywords="sharding scaling, Azure SQL Database sharding, elastic scale" services="sql-database" documentationCenter="" manager="jhubbard" authors="sidneyh"/>

# Azure SQL Database 灵活扩展常见问题 

####如果我每个分片上都有一个租户但没有分片键，我应该如何用架构信息填充分片键？
架构信息对象仅用于拆分/合并方案。如果应用程序在本质上是单租户，则它不需要拆分/合并服务，从而无需填充架构信息对象。

#### 如果我已设置数据库且已有一个分片映射管理器，那么应如何将此新数据库注册为分片？
请参阅**[将分片添加到灵活扩展应用程序](/zh-cn/documentation/articles/sql-database-elastic-scale-add-a-shard/)**。 

#### 灵活扩展的成本是多少？
使用灵活扩展库时不会产生任何费用。仅在将 Azure SQL Database 用于分片和分片映射管理器以及为拆分/合并服务设置的 Web/辅助角色时才会产生费用。

#### 为什么在我从另一台服务器添加分片时无法使用我的凭据？
不要使用"User ID=username@servername"形式的凭据，而是直接使用"User ID = username"形式的凭据。此外，请确保"username"登录名对该分片具有相应权限。

#### 是否需要在每次启动应用程序时创建分片映射管理器并填充分片？
不需要，因为创建分片映射管理器（例如，**[ShardMapManagerFactory.CreateSqlShardMapManager](http://msdn.microsoft.co/zh-cn/library/azure/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanagerfactory.createsqlshardmapmanager.aspx)**）是一次性操作。您的应用程序应在启动时使用调用 **[ShardMapManagerFactory.TryGetSqlShardMapManager()](http://msdn.microsoft.co/zh-cn/library/azure/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanagerfactory.trygetsqlshardmapmanager.aspx)**。每个应用程序域应只存在一个此类调用。

#### 我对使用灵活扩展有一些疑问，我该如何获取其答案？ 
请通过 [Azure SQL Database 论坛](https://social.msdn.microsoft.com/Forums/zh-CN/home?forum=windowsazurezhchs)联系我们。

#### 在我使用分片键连接数据库时，我仍可以在同一分片上查询到其他分片键的数据。是有意设计成这样的吗？
灵活扩展 API 让您可以连接到分片键的正确数据库，但是不提供分片键筛选。向您的查询添加 **WHERE** 子句，以将范围限制为所提供的分片键（如果需要）。

#### 对于我的分片集中的每个分片，我可以使用不同的 Azure Database 版本吗？
可以，分片是单独的数据库，因此有的分片可以是高级版，而其他的是标准版。此外，在分片的生存期内其版本可以多次向上或向下扩展。

#### 拆分/合并服务是在拆分或合并操作期间设置（或删除）数据库吗？ 
不是。对于**拆分**操作，目标数据库必须具有相应的架构并且必须使用分片映射管理器进行注册。对于**合并**操作，您必须先从分片映射管理器中删除分片，然后才能删除该数据库。

[AZURE.INCLUDE [elastic-scale-include](../includes/elastic-scale-include.md)]
