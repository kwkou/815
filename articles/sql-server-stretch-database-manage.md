<properties
	pageTitle="SQL Server Stretch Database的管理和故障排除 | Azure"
	description="如何对SQL Server Stretch Database进行管理和故障排除。"
	services="sql-server-stretch-database"
	documentationCenter=""
	authors="douglasl"
	manager="jhubbard"
	editor="monicar"/>

<tags
	ms.service="sql-server-stretch-database"
	ms.date="02/26/2016"
	wacn.date="03/10/2016"/>

# SQL Server Stretch Database的管理和故障排除

若要对SQL Server Stretch Database进行管理和故障排除，请使用本主题中所述的工具和方法。

## <a name="LocalInfo"></a>获取有关已启用SQL Server Stretch Database的本地数据库和表的信息
打开目录视图 **sys.databases** 和 **sys.tables**，以查看有关已启用延伸的 SQL Server 数据库和表的信息。有关详细信息，请参阅 [sys.databases (Transact-SQL)](https://msdn.microsoft.com/zh-cn/library/ms178534.aspx) 和 [sys.tables (Transact-SQL)](https://msdn.microsoft.com/zh-cn/library/ms187406.aspx)。

## <a name="RemoteInfo"></a>获取有关SQL Server Stretch Database使用的远程数据库和表的信息
打开目录视图 **sys.remote\_data\_archive\_databases** 和 **sys.remote\_data\_archive\_tables**，以查看有关存储迁移数据的远程数据库和表的信息。有关详细信息，请参阅 [sys.remote\_data\_archive\_databases (Transact-SQL)](https://msdn.microsoft.com/zh-cn/library/dn934995.aspx) 和 [sys.remote\_data\_archive\_tables (Transact-SQL)](https://msdn.microsoft.com/zh-cn/library/dn935003.aspx)。

## 检查应用于表的筛选器谓词
打开目录视图 **sys.remote\_data\_archive\_tables** 并检查 **filter\_predicate** 列的值。如果值为 null，则整个表符合迁移条件。有关详细信息，请参阅 [sys.remote\_data\_archive\_tables (Transact-SQL)](https://msdn.microsoft.com/zh-cn/library/dn935003.aspx)。

## <a name="Migration"></a>检查数据迁移状态
在 SQL Server Management Studio 中选择数据库对应的“任务 | 延伸 | 监视”，以便在SQL Server Stretch Database监视器中监视数据迁移。有关详细信息，请参阅[数据迁移的监视和故障排除（SQL Server Stretch Database）](/documentation/articles/sql-server-stretch-database-monitor/)。

或者，打开动态管理视图 **sys.dm\_db\_rda\_migration\_status** 查看已迁移的数据批数与行数。

## 提高 Azure 性能级别，以便能够执行索引编制等资源密集型操作
在生成、重新生成或重新组织已针对SQL Server Stretch Database配置的大型表中的索引时，请考虑在该操作的持续时间内提高相应远程数据库的性能级别。

## 不要更改远程表的架构
不要更改与针对SQL Server Stretch Database配置的 SQL Server 表相关联的远程 Azure 表的架构。具体而言，请不要修改列的名称或数据类型。SQL Server Stretch Database功能会在与 SQL Server 表架构关联的远程表架构方面做出多种假设。如果你更改了远程架构，所更改表的SQL Server Stretch Database将停止工作。

## <a name="Firewall"></a>数据迁移的故障排除
有关故障排除的建议，请参阅[数据迁移的监视和故障排除（SQL Server Stretch Database）](/documentation/articles/sql-server-stretch-database-monitor/)。

## 查询性能故障排除
**包含已启用延伸的表的查询速度缓慢。**
包含已启用延伸的表的查询在执行时的速度预期比启用延伸之前要慢。如果查询性能显著下降，请查看以下可能的问题。

-   你的 SQL Server 所在的地理区域是否与 Azure 服务器不同？ 在与 SQL Server 相同的地理区域配置 Azure 服务器可以降低网络延迟。

-   你的网络条件可能有所退化。有关最近出现的问题或服务中断的信息，请与你的网络管理员联系。

## 另请参阅
[监视SQL Server Stretch Database](/documentation/articles/sql-server-stretch-database-monitor/)
[备份和还原已启用延伸的数据库](/documentation/articles/sql-server-stretch-database-backup/)

<!---HONumber=Mooncake_0307_2016-->