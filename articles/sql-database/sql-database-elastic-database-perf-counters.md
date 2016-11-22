<properties
    pageTitle="分片映射管理器的性能计数器"
    description="ShardMapManager 类和数据相关的路由的性能计数器"
    services="sql-database"
    documentationCenter=""
    manager="jhubbard"
    authors="SilviaDoomra"
    editor=""/>

<tags
    ms.service="sql-database"
    ms.workload="sql-database"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="05/23/2016"
    wacn.date="07/11/2016"
    ms.author="SilviaDoomra"/>

# 分片映射管理器的性能计数器

你可以捕获[分片映射管理器](/documentation/articles/sql-database-elastic-scale-shard-map-management/)的性能，尤其是在使用[数据相关的路由](/documentation/articles/sql-database-elastic-scale-data-dependent-routing/)时。计数器是使用 Microsoft.Azure.SqlDatabase.ElasticScale.Client 类的方法创建的。

计数器用于跟踪[数据相关路由](/documentation/articles/sql-database-elastic-scale-data-dependent-routing/)操作的性能。可在“弹性数据库: 分片管理”类别下的“性能监视器”中访问这些计数器。

**有关最新版本：**请转到 [Microsoft.Azure.SqlDatabase.ElasticScale.Client](https://www.nuget.org/packages/Microsoft.Azure.SqlDatabase.ElasticScale.Client/)。另请参阅 [Upgrade an app to use the latest elastic database client library（升级应用以使用最新的弹性数据库客户端库）](/documentation/articles/sql-database-elastic-scale-upgrade-client-library/)。

## 先决条件

* 若要创建性能类别和计数器，用户必须属于托管应用程序的计算机上的本地“管理员”组。  

* 若要创建性能计数器实例和更新计数器，用户必须是“管理员”或“性能监视器用户”组的成员。

## 创建性能类别和计数器 

若要创建计数器，请调用 [ShardMapManagmentFactory 类](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanagerfactory.aspx)的 CreatePeformanceCategoryAndCounters 方法。只有管理员才能执行该方法：

	ShardMapManagerFactory.CreatePerformanceCategoryAndCounters()  

也可以使用[此](https://gallery.technet.microsoft.com/scriptcenter/Elastic-DB-Tools-for-Azure-17e3d283) PowerShell 脚本来执行该方法。该方法将创建以下性能计数器：

* **缓存映射**：为分片映射缓存的映射数目。
*  **DDR 操作次数/秒**：分片映射的数据相关路由操作速率。对 [OpenConnectionForKey()](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmap.openconnectionforkey.aspx) 的调用导致成功连接到目标分片时，将更新此计数器。 
*  **映射查找缓存命中数/秒**：分片映射中映射的成功缓存查找操作速率。 
*  **映射查找缓存未命中数/秒**：分片映射中映射的失败缓存查找操作速率。
*  **在缓存中添加或更新的映射数/秒**：在缓存中为分片映射添加或更新映射的速率。 
*  **从缓存中删除的映射数/秒**：从缓存中删除分片映射的映射速率。 

性能计数器是按进程为每个缓存分片映射创建的。


## 说明
以下事件会触发性能计数器创建：

* 在 ShardMapManager 包含任何分片映射的情况下，使用[积极加载](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanagerloadpolicy.aspx)初始化 [ShardMapManager](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanager.aspx)。这包括 [GetSqlShardMapManager](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanagerfactory.getsqlshardmapmanager.aspx?f=255&MSPPError=-2147217396#M:Microsoft.Azure.SqlDatabase.ElasticScale.ShardManagement.ShardMapManagerFactory.GetSqlShardMapManager%28System.String,Microsoft.Azure.SqlDatabase.ElasticScale.ShardManagement.ShardMapManagerLoadPolicy%29) 和 [TryGetSqlShardMapManager](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.sqldatabase.elasticscale.shardmanagement.shardmapmanagerfactory.trygetsqlshardmapmanager.aspx) 方法。
* 成功查找分片映射（使用 [GetShardMap()](https://msdn.microsoft.com/zh-cn/library/azure/dn824215.aspx)、[GetListShardMap()](https://msdn.microsoft.com/zh-cn/library/azure/dn824212.aspx) 或 [GetRangeShardMap()](https://msdn.microsoft.com/zh-cn/library/azure/dn824173.aspx)）。 

* 使用 CreateShardMap() 成功创建分片映射。

对分片映射和映射执行的所有缓存操作将会更新性能计数器。使用 DeleteShardMap() 成功删除分片映射会导致删除性能计数器实例。

## 最佳实践

* 只应在创建 ShardMapManager 对象之前创建性能类别和计数器一次。每次执行命令 CreatePerformanceCategoryAndCounters() 都会清除以前的计数器（丢失所有实例报告的数据），并创建新计数器。  

* 性能计数器实例是按进程创建的。任何应用程序崩溃或从缓存中删除分片映射都会导致删除性能计数器实例。

### 另请参阅

[弹性数据库功能概述](/documentation/articles/sql-database-elastic-scale-introduction/)

[AZURE.INCLUDE [elastic-scale-include](../../includes/elastic-scale-include.md)]

<!--Anchors-->
<!--Image references-->


<!---HONumber=Mooncake_0704_2016-->