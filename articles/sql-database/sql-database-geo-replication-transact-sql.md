<properties
    pageTitle="使用 Transact-SQL 为 Azure SQL 数据库配置异地复制 | Azure"
    description="使用 Transact-SQL 为 Azure SQL 数据库配置异地复制"
    services="sql-database"
    documentationCenter=""
    authors="carlrabeler"
    manager="jhubbard"
    editor=""/>

<tags
    ms.service="sql-database"
    ms.date="07/18/2016"
    wacn.date="08/15/2016"/>

# 使用 Transact-SQL 为 Azure SQL 数据库配置异地复制

> [AZURE.SELECTOR]
- [概述](/documentation/articles/sql-database-geo-replication-overview/)
- [PowerShell](/documentation/articles/sql-database-geo-replication-powershell/)
- [T-SQL](/documentation/articles/sql-database-geo-replication-transact-sql/)

本文说明如何使用 Transact-SQL 为 Azure SQL 数据库配置活动异地复制。

若要使用 Transact-SQL 启动故障转移，请参阅[使用 Transact-SQL 为 Azure SQL 数据库启动计划内或计划外故障转移](/documentation/articles/sql-database-geo-replication-failover-transact-sql/)。

>[AZURE.NOTE] 活动异地复制现在可供标准服务层和高级服务层中的所有数据库使用。可读辅助副本目前只能用于高级服务层中的数据库。

若要使用 Transact-SQL 配置活动异地复制，需提供：

- Azure 订阅。
- 逻辑 Azure SQL 数据库服务器 <MyLocalServer> 和 SQL 数据库 <MyDB> - 你要复制的主数据库。
- 一个或多个逻辑 Azure SQL 数据库服务器 <MySecondaryServer(n)> - 可充当伙伴服务器（将在其中创建辅助数据库）的逻辑服务器。
- 主数据库上 DBManager 的登录名，拥有你要异地复制的本地数据库的 db\_ownership 权限，并且是你要配置异地复制的伙伴服务器上的 DBManager。
- SQL Server Management Studio (SSMS)

> [AZURE.IMPORTANT] 建议始终使用最新版本的 Management Studio 以保持与 Azure 和 SQL 数据库的更新同步。[更新 SQL Server Management Studio](https://msdn.microsoft.com/zh-cn/library/mt238290.aspx)。


## 添加辅助数据库

可以使用 **ALTER DATABASE** 语句在伙伴服务器上创建异地复制的辅助数据库。在包含要复制的数据库服务器的 master 数据库上执行此语句。异地复制数据库（“主数据库”）具备与要复制的数据库相同的名称，并且默认与主数据库具有相同的服务级别。辅助数据库可以是可读或不可读，并且可以是单一数据库或弹性数据库。有关详细信息，请参阅 [ALTER DATABASE (Transact-SQL)](https://msdn.microsoft.com/zh-cn/library/mt574871.aspx) 和[服务层](/documentation/articles/sql-database-service-tiers/)。
创建辅助数据库并设定种子之后，数据将开始以异步方式从主数据库复制。以下步骤说明如何使用 Management Studio 配置异地复制。提供使用单一数据库或弹性数据库创建不可读和可读辅助副本的步骤。

> [AZURE.NOTE] 如果指定的伙伴服务器上的数据库的名称与主数据库相同，该命令会失败。


### 添加不可读的辅助数据库（单一数据库）

可以使用以下步骤将不可读的辅助数据库创建为单一数据库。

1. 使用 13.0.600.65 或更高版本的 SQL Server Management Studio。

 	 > [AZURE.IMPORTANT] 下载[最新](https://msdn.microsoft.com/zh-cn/library/mt238290.aspx)版本的 SQL Server Management Studio。建议始终使用最新版本的 Management Studio 以保持与 Azure 门户的更新同步。


2. 打开“数据库”文件夹、展开“系统数据库”、右键单击“master”，然后单击“新建查询”。

3. 使用以下 **ALTER DATABASE** 语句使本地数据库成为具有 MySecondaryServer1 上不可读辅助数据库的异地复制主数据库，其中，MySecondaryServer1 是服务器的友好名称。

        ALTER DATABASE <MyDB>
           ADD SECONDARY ON SERVER <MySecondaryServer1> WITH (ALLOW_CONNECTIONS = NO);

4. 单击“执行”运行查询。


### 添加可读的辅助数据库（单一数据库）
可以使用以下步骤将可读辅助数据库创建为单一数据库。

1. 在 Management Studio 中，连接到你的 Azure SQL 数据库逻辑服务器。

2. 打开“数据库”文件夹、展开“系统数据库”、右键单击“master”，然后单击“新建查询”。

3. 使用以下 **ALTER DATABASE** 语句使本地数据库成为具有辅助服务器上可读辅助数据库的异地复制主数据库。

        ALTER DATABASE <MyDB>
           ADD SECONDARY ON SERVER <MySecondaryServer2> WITH (ALLOW_CONNECTIONS = ALL);

4. 单击“执行”运行查询。



### 添加不可读的辅助数据库（弹性数据库）

可以使用以下步骤将不可读的辅助数据库创建为弹性数据库。

1. 在 Management Studio 中，连接到你的 Azure SQL 数据库逻辑服务器。

2. 打开“数据库”文件夹、展开“系统数据库”、右键单击“master”，然后单击“新建查询”。

3. 使用以下 **ALTER DATABASE** 语句使本地数据库成为具有弹性池中辅助服务器上的不可读辅助数据库的异地复制主数据库。

        ALTER DATABASE <MyDB>
           ADD SECONDARY ON SERVER <MySecondaryServer3> WITH (ALLOW_CONNECTIONS = NO
           , SERVICE_OBJECTIVE = ELASTIC_POOL (name = MyElasticPool1));

4. 单击“执行”运行查询。



### 添加可读的辅助数据库（弹性数据库）
可以使用以下步骤将可读的辅助数据库创建为弹性数据库。

1. 在 Management Studio 中，连接到你的 Azure SQL 数据库逻辑服务器。

2. 打开“数据库”文件夹、展开“系统数据库”、右键单击“master”，然后单击“新建查询”。

3. 使用以下 **ALTER DATABASE** 语句使本地数据库成为具有弹性池中辅助服务器上的可读辅助数据库的异地复制主数据库。

        ALTER DATABASE <MyDB>
           ADD SECONDARY ON SERVER <MySecondaryServer4> WITH (ALLOW_CONNECTIONS = ALL
           , SERVICE_OBJECTIVE = ELASTIC_POOL (name = MyElasticPool2));

4. 单击“执行”运行查询。



## 删除辅助数据库

可以使用 **ALTER DATABASE** 语句永久终止辅助数据库与其主数据库之间的复制关系。此语句将在主数据库所在的 master 数据库上运行。终止关系后，辅助数据库将成为常规读写数据库。如果与辅助数据库的连接断开，命令将会成功，但辅助数据库必须等到连接恢复后才变为可读写。有关详细信息，请参阅 [ALTER DATABASE (Transact-SQL)](https://msdn.microsoft.com/zh-cn/library/mt574871.aspx) 和[服务层](/documentation/articles/sql-database-service-tiers/)。

使用以下步骤从异地复制合作关系中删除异地复制的辅助数据库。

1. 在 Management Studio 中，连接到你的 Azure SQL 数据库逻辑服务器。

2. 打开“数据库”文件夹、展开“系统数据库”、右键单击“master”，然后单击“新建查询”。

3. 使用以下 **ALTER DATABASE** 语句来删除异地复制的辅助数据库。

        ALTER DATABASE <MyDB>
           REMOVE SECONDARY ON SERVER <MySecondaryServer1>;

4. 单击“执行”运行查询。

## 监视异地复制配置和运行状况

监视任务包括监视异地复制配置和监视数据复制运行状况。可以使用 master 数据库中的 **sys.dm\_geo\_replication\_links** 动态管理视图返回 Azure SQL 数据库逻辑服务器上每个数据库的所有现有复制链接的相关信息。此视图针对每个主数据库和辅助数据库之间的复制链接包含一行。可以使用 **sys.dm\_replication\_status** 动态管理视图针对当前参与复制链接的每个 Azure SQL 数据库返回一行。这包括主数据库和辅助数据库。如果给定主数据库有多个连续复制链接，此表将对每种关系包含一行。将在所有数据库（包括逻辑 master）中创建该视图。但是，在逻辑 master 中查询此视图会返回空集。可以使用 **sys.dm\_operation\_status** 动态管理视图来显示所有数据库操作的状态，包括复制链接的状态。有关详细信息，请参阅 [sys.geo\_replication\_links（Azure SQL 数据库）](https://msdn.microsoft.com/zh-cn/library/mt575501.aspx)、[sys.dm\_geo\_replication\_link\_status（Azure SQL 数据库）](https://msdn.microsoft.com/zh-cn/library/mt575504.aspx)和 [sys.dm\_operation\_status（Azure SQL 数据库）](https://msdn.microsoft.com/zh-cn/library/dn270022.aspx)。

使用以下步骤监视异地复制合作关系。

1. 在 Management Studio 中，连接到你的 Azure SQL 数据库逻辑服务器。

2. 打开“数据库”文件夹、展开“系统数据库”、右键单击“master”，然后单击“新建查询”。

3. 使用以下语句显示具有异地复制链接的所有数据库。

        SELECT database_id, start_date, modify_date, partner_server, partner_database, replication_state_desc, role, secondary_allow_connections_desc FROM [sys].geo_replication_links;

4. 单击“执行”运行查询。
5. 打开“数据库”文件夹、展开“系统数据库”、右键单击“MyDB”，然后单击“新建查询”。
6. 使用以下语句显示复制延迟和辅助数据库 MyDB 上次复制的开始时间。

        SELECT link_guid, partner_server, last_replication, replication_lag_sec FROM sys.dm_geo_replication_link_status

7. 单击“执行”运行查询。
8. 使用以下语句显示与 MyDB 数据库关联的最近异地复制操作。

        SELECT * FROM sys.dm_operation_status where major_resource_id = 'MyDB'
        ORDER BY start_time DESC

9. 单击“执行”运行查询。

## 将不可读辅助数据库升级为可读辅助数据库

非可读辅助类型将在 2017 年 4 月停用，现有的非可读数据库将自动升级到可读辅助数据库。如果你目前使用的是不可读辅助数据库，而你想要将它们升级为可读辅助数据库，则可以针对每个辅助数据库执行下列简单步骤。

> [AZURE.IMPORTANT] 并没有自助升级方法能够将不可读辅助数据库就地升级为可读辅助数据库。如果删除唯一的辅助数据库，主数据库将会维持未受保护的状态，直到新的辅助数据库完全同步为止。如果你的应用程序 SLA 要求主数据库始终受到保护，你应该考虑在其他服务器中创建并行辅助数据库，然后再应用上述升级步骤。请注意，每个主数据库最多可以有 4 个辅助数据库。


1. 首先，连接到辅助服务器，然后删除不可读辅助数据库：
        
        DROP DATABASE <MyNonReadableSecondaryDB>;

2. 现在，连接到主服务器，然后添加新的可读辅助数据库

        ALTER DATABASE <MyDB>
            ADD SECONDARY ON SERVER <MySecondaryServer> WITH (ALLOW_CONNECTIONS = ALL);




## 后续步骤

- 若要了解有关活动异地复制的详细信息，请参阅[活动异地复制](/documentation/articles/sql-database-geo-replication-overview/)
- 若要了解业务连续性设计和恢复方案，请参阅[连续性方案](/documentation/articles/sql-database-business-continuity-scenarios/)

<!---HONumber=Mooncake_0808_2016-->