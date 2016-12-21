<properties
    pageTitle="Azure SQL 弹性缩放常见问题 | Azure"
    description="关 Azure SQL 数据库弹性缩放的常见问题。"
    services="sql-database"
    documentationcenter=""
    manager="jhubbard"
    author="ddove"
    editor="" />
<tags
    ms.assetid="e60dde9c-bb7b-4f2f-b52c-bdb506d49fcb"
    ms.service="sql-database"
    ms.workload="sql-database"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="10/24/2016"
    wacn.date="12/19/2016"
ms.author="ddove" />

# 弹性数据库工具常见问题
#### 如果每个分片只有单个租户且没有分片键，该如何为架构信息填充分片键？
架构信息对象仅用于“拆分/合并”方案。如果某个应用程序本质上是单租户，那么它则不需要“拆分/合并”工具，因此无需填充架构信息对象。

#### 在已预配了一个数据库，并且已安装了分片映射管理器的情况下，应如何将此新数据库注册为分片？
请参阅**[使用弹性数据库客户端库将分片添加到应用程序](/documentation/articles/sql-database-elastic-scale-add-a-shard/)**。

#### 弹性数据库工具的费用如何？
使用弹性数据库客户端库不会产生任何费用。只有为分片使用的 Azure SQL 数据库和分片映射管理器，以及为“拆分/合并”工具预配的 Web/辅助角色才会产生费用。

#### 为什么从另一台服务器添加分片时，凭据不起作用？
请不要使用“用户 ID=用户名@服务器名称”格式的凭据，只需使用“用户 ID = 用户名”的格式即可。此外，请确保“用户名”登录名对分片具有权限。

#### 每次启动应用程序时，是否都需要创建分片映射管理器并填充分片？

不是 - 分片映射管理器（例如，**[ShardMapManagerFactory.CreateSqlShardMapManager](http://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanagerfactory.createsqlshardmapmanager.aspx)**）只需创建一次。在启动应用程序时，应用程序应使用调用 **[ShardMapManagerFactory.TryGetSqlShardMapManager()](http://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanagerfactory.trygetsqlshardmapmanager.aspx)**。每个应用程序域应该只有一个此类调用。

#### 如果在使用弹性数据库工具方面存在疑问，如何才能获得解答？ 

请在 [Azure SQL 数据库论坛](https://social.msdn.microsoft.com/Forums/zh-cn/home?forum=ssdsgetstarted)上联系我们。

#### 当使用分片键建立数据库连接的同时，仍然可以对同一分片上的其他分片键查询数据。这是设计使然吗？
弹性缩放 API 使用户能够连接到分片键的正确数据库，但不提供分片键筛选。如果需要，请在查询中添加 **WHERE** 子句，以将范围限制到提供的分片键。

#### 是否可为分片集中的每个分片使用不同的 Azure 数据库版本？
是的，每个分片是单独的数据库，因此，一个分片可以是高级版，而另一个可以是标准版。此外，在分片的生命周期内，该分片的版本可以进行多次上调或下调。

#### 在拆分或合并操作期间，“拆分/合并”工具是否会预配（或删除）数据库？
不会。对于**拆分**操作，必须存在目标数据库和相应的架构，并且必须注册到分片映射管理器。对于**合并**操作，必须从分片映射管理器中删除分片，然后删除数据库。

[AZURE.INCLUDE [elastic-scale-include](../../includes/elastic-scale-include.md)]
 

<!---HONumber=Mooncake_1212_2016-->