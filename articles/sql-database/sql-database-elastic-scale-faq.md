<properties 
	pageTitle="Azure SQL 弹性缩放常见问题 | Azure" 
	description="有关 Azure SQL 数据库弹性缩放的常见问题。" 
	services="sql-database" 
	documentationCenter="" 
	manager="jeffreyg" 
	authors="sidneyh" 
	editor=""/>

<tags 
	ms.service="sql-database" 
	ms.date="05/03/2016" 
	wacn.date="05/23/2016"/>

# 弹性数据库工具常见问题 

#### 如果我的分片只有单个租户并且我没有分片键，该如何为架构信息填充分片键？

架构信息对象仅用于“拆分/合并”方案。如果某个应用程序本质上是单租户的，则它不需要“拆分/合并”服务，因此你无需填充架构信息对象。

#### 我已经置了一个数据库，并且已安装分片映射管理器，我应如何将此新数据库注册为分片？

请参阅**[使用弹性数据库客户端库将分片添加到应用程序](/documentation/articles/sql-database-elastic-scale-add-a-shard/)**。

#### 弹性数据库工具的费用如何？

使用弹性数据库客户端库不会产生任何费用。只有你为分片使用的 Azure SQL 数据库和分片映射管理器，以及为“拆分/合并”工具设置的 Web/辅助角色才会产生费用。

#### 为什么当我从另一台服务器添加分片时，我的凭据不起作用？
请不要使用“用户 ID=用户名@服务器名称”格式的凭据，而只需使用“用户 ID = 用户名”格式。此外，请确保“用户名”登录名对分片具有权限。

#### 是否我每次启动应用程序时，都需要创建分片映射管理器并填充分片？

不是 - 分片映射管理器（例如，**[ShardMapManagerFactory.CreateSqlShardMapManager](http://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanagerfactory.createsqlshardmapmanager.aspx)**）只需创建一次。在启动应用程序时，应用程序应使用调用 **[ShardMapManagerFactory.TryGetSqlShardMapManager()](http://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanagerfactory.trygetsqlshardmapmanager.aspx)**。每个应用程序域应该只有一个此类调用。

#### 我在使用弹性数据库工具方面存在疑问，如何才能获得解答？ 

请在 [Azure SQL 数据库论坛](https://social.msdn.microsoft.com/Forums/zh-cn/home?forum=ssdsgetstarted)上联系我们。

#### 当我使用分片键建立数据库连接时，我仍可以对同一分片上的其他分片键查询数据。这是设计使然吗？

弹性缩放 API 可让你连接到分片键的正确数据库，但不提供分片键筛选。如果需要，请在查询中添加 **WHERE** 子句，以将范围限制到提供的分片键。

#### 我是否可为分片集中的每个分片使用不同的 Azure 数据库版本？

是的，每个分片是单独的数据库，因此，一个分片可以是高级版，而另一个是标准版。此外，在分片的生命周期内，该分片的版本可以上调或下调多次。

#### 在拆分或合并操作期间，“拆分/合并”工具是否会设置（或删除）数据库？ 

不会。对于**拆分**操作，必须存在目标数据库和相应的架构，并且必须注册到分片映射管理器。对于**合并**操作，必须从分片映射管理器中删除分片，然后删除数据库。

[AZURE.INCLUDE [elastic-scale-include](../includes/elastic-scale-include.md)]
 

<!---HONumber=Mooncake_0314_2016-->
