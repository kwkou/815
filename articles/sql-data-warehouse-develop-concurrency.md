<properties
   pageTitle="SQL 数据仓库中的并发性和工作负荷管理 | Azure"
   description="在开发解决方案之前，了解 SQL 数据仓库中的并发性和工作负荷管理。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="jrowlandjones"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="03/23/2016"
   wacn.date="05/23/2016"/>

# SQL 数据仓库中的并发性和工作负荷管理
若要在大型 SQL 数据仓库提供可预测的性能，请实施所需的机制来管理工作负荷并发性和计算资源分配。

本文将介绍并发性和工作负荷管理的概念；说明如何实现这两种功能，以及如何在数据仓库中控制它们。

>[AZURE.NOTE] SQL 数据仓库支持多用户而非多租户工作负荷。

## 并发
请务必了解 SQL 数据仓库中的并发性由两个概念控制：**并发查询**和**并发槽**。

并发查询相当于在同一时间执行的查询数目。SQL 数据仓库支持高达 32 个**并发查询**。无论是串行查询（单线程）还是并行查询（多线程），每个查询执行都视为单一查询。这是固定的上限，可应用到所有服务级别和所有查询。

并发槽是更为动态的概念，相对于数据仓库的数据仓库单位 (DWU) 服务等级目标。增加分配给 SQL 数据仓库的 DWU 数目时，就会分配更多的计算资源。但是，增加 DWU 也会增加可用**并发槽**的数目。

一般来说，每个并发执行的查询都会使用一个或多个并发槽。确切的插槽数目取决于三个因素：

1. SQL 数据仓库的 DWU 设置
2. 用户所属的**资源类**
3. 查询或操作是否由并发槽模型控制

> [AZURE.NOTE] 值得注意的是，不是每个查询都由并发槽查询规则所控制。但大多数用户查询都是如此。某些查询和操作完全不使用任何并发槽。这些查询和操作仍然受限于并发查询限制，这就是为什么要同时描述这两个规则。有关详细信息，请参阅下面的[资源类例外情况](#exceptions)部分。

下表描述了并发查询和并发槽的限制；其中假设查询受资源控制。

<!--
| Concurrency Slot Consumption | DW100 | DW200 | DW300 | DW400 | DW500 | DW600 | DW1000 | DW1200 | DW1500 | DW2000 | DW3000 | DW6000 |
| :--------------------------- | :---- | :---- | :---- | :---- | :---- | :---- | :----- | :----- | :----- | :----- | :----- | :----- |
| Max Concurrent Queries       | 32    | 32    | 32    | 32    | 32    | 32    | 32     | 32     | 32     | 32     | 32     | 32     |
| Max Concurrency Slots        | 4     | 8     | 12    | 16    | 20    | 24    | 40     | 48     | 60     | 80     | 120    | 240    |
-->

| 并发槽使用量 | DW100 | DW200 | DW300 | DW400 | DW500 | DW600 | DW1000 | DW1200 | DW1500 | DW2000 |
| :--------------------------- | :---- | :---- | :---- | :---- | :---- | :---- | :----- | :----- | :----- | :----- |
| 并发查询数上限 | 32 | 32 | 32 | 32 | 32 | 32 | 32 | 32 | 32 | 32 |
| 并发槽数上限 | 4 | 8 | 12 | 16 | 20 | 24 | 40 | 48 | 60 | 80 |

SQL 数据仓库查询工作负荷必须保持在这些阈值范围内。如果有 32 个以上的并发查询，或超过并发槽的数目，查询将排入队列，直到可以满足两个阈值。

## 工作负荷管理

SQL 数据仓库以**数据库角色**的形式公开四个不同的资源类，作为其工作负荷管理实现的一部分。

这些角色为：

- smallrc
- mediumrc
- largerc
- xlargerc

资源类是 SQL 数据仓库工作负荷管理中不可或缺的一部分。它们控制分配给查询的计算资源。

默认情况下，每个用户都是小型资源类 - smallrc 的成员。但是，任何用户都可以添加到一个或多个较高的资源类。一般而言，SQL 数据仓库将采用最高的角色成员资格来执行查询。将用户添加到较高的资源类会增加该用户的资源，但也会消耗更多的并发槽；这可能会限制并发性。这是因为，将多个资源分配给一个查询时，系统需要限制其他查询所使用的资源。天下没有免费的午餐。

较高资源类控制的最重要资源就是内存。大多数有意义大小的数据仓库表都使用群集列存储索引。这通常会提供数据仓库工作负荷的最佳性能，而维护工作负荷是一项内存密集型操作。使用较高的资源类通常相当有助于索引重建等数据管理操作。

SQL 数据仓库已通过使用数据库角色来实现资源类。若要成为较高资源类的成员并增加内存和优先级，只需要将数据库用户添加到前面所述的某个角色/资源类。

### 资源类成员资格

你可以使用 `sp_addrolemember` 和 `sp_droprolemember` 过程将自己添加为工作负荷管理数据库角色，或者删除自己的这种角色。请注意，需要 `ALTER ROLE` 权限才能执行此操作。不能使用 ALTER ROLE DDL 语法。必须使用上述存储过程。本文末尾的 [管理用户)[#managing-users] 部分提供了有关如何创建登录名和用户的完整示例。

> [AZURE.NOTE] 相比于在工作负荷管理组内外添加用户，更方便的做法通常是通过永久分配给较高资源类的独立登录名/用户来启动更密集的操作。

### 内存分配

下表详细说明了每个查询可用的内存增量；取决于应用到执行查询的用户的资源类：

<!--
| Memory Available (per dist) | Priority | DW100  | DW200  | DW300  | DW400   | DW500   | DW600   | DW1000  | DW1200  | DW1500  | DW2000  | DW3000  | DW6000   |
| :-------------------------- | :------- | :----  | :----- | :----- | :------ | :------ | :------ | :------ | :------ | :------ | :------ | :------ | :------- |
| smallrc(default) (s)        | Medium   | 100 MB | 100 MB | 100 MB | 100  MB | 100 MB  | 100 MB  | 100 MB  | 100 MB  | 100 MB  | 100 MB  | 100  MB | 100   MB |
| mediumrc (m)                | Medium   | 100 MB | 200 MB | 200 MB | 400  MB | 400 MB  | 400 MB  | 800 MB  | 800 MB  | 800 MB  | 1600 MB | 1600 MB | 3200  MB |
| largerc (l)                 | High     | 200 MB | 400 MB | 400 MB | 800  MB | 800 MB  | 800 MB  | 1600 MB | 1600 MB | 1600 MB | 3200 MB | 3200 MB | 6400  MB |
| xlargerc (xl)               | High     | 400 MB | 800 MB | 800 MB | 1600 MB | 1600 MB | 1600 MB | 3200 MB | 3200 MB | 3200 MB | 6400 MB | 6400 MB | 12800 MB |
-->

<!--
| Memory Available (per dist) | Priority | DW100  | DW200  | DW300  | DW400   | DW500   | DW600   | DW1000  | DW1200  | DW1500  | DW2000  |
| :-------------------------- | :------- | :----  | :----- | :----- | :------ | :------ | :------ | :------ | :------ | :------ | :------ |
| smallrc(default) (s)        | Medium   | 100 MB | 100 MB | 100 MB | 100  MB | 100 MB  | 100 MB  | 100 MB  | 100 MB  | 100 MB  | 100 MB  |
| mediumrc (m)                | Medium   | 100 MB | 200 MB | 200 MB | 400  MB | 400 MB  | 400 MB  | 800 MB  | 800 MB  | 800 MB  | 1600 MB |
| largerc (l)                 | High     | 200 MB | 400 MB | 400 MB | 800  MB | 800 MB  | 800 MB  | 1600 MB | 1600 MB | 1600 MB | 3200 MB |
| xlargerc (xl)               | High     | 400 MB | 800 MB | 800 MB | 1600 MB | 1600 MB | 1600 MB | 3200 MB | 3200 MB | 3200 MB | 6400 MB |
-->

| 可用内存（每个 dist） | DW100 | DW200 | DW300 | DW400 | DW500 | DW600 | DW1000 | DW1200 | DW1500 | DW2000 |
| :-------------------------- | :----  | :----- | :----- | :------ | :------ | :------ | :------ | :------ | :------ | :------ |
| smallrc(default) (s) | 100 MB | 100 MB | 100 MB | 100 MB | 100 MB | 100 MB | 100 MB | 100 MB | 100 MB | 100 MB |
| mediumrc (m) | 100 MB | 200 MB | 200 MB | 400 MB | 400 MB | 400 MB | 800 MB | 800 MB | 800 MB | 1600 MB |
| largerc (l) | 200 MB | 400 MB | 400 MB | 800 MB | 800 MB | 800 MB | 1600 MB | 1600 MB | 1600 MB | 3200 MB |
| xlargerc (xl) | 400 MB | 800 MB | 800 MB | 1600 MB | 1600 MB | 1600 MB | 3200 MB | 3200 MB | 3200 MB | 6400 MB |

### 并发槽使用量

此外，如上所述，分配给用户的资源类越高，并发槽的使用量就越大。下表描述了查询在给定资源类中的并发槽使用量。

<!--
| Consumption | DW100 | DW200 | DW300 | DW400 | DW500 | DW600 | DW1000 | DW1200 | DW1500 | DW2000 | DW3000 | DW6000 |
| :--------------------------- | :---- | :---- | :---- | :---- | :---- | :---- | :----- | :----- | :----- | :----- | :----- | :----- |
| Max Concurrent Queries       | 32    | 32    | 32    | 32    | 32    | 32    | 32     | 32     | 32     | 32     | 32     | 32     |
| Max Concurrency Slots        | 4     | 8     | 12    | 16    | 20    | 24    | 40     | 48     | 60     | 80     | 120    | 240    |
| smallrc(default) (s)         | 1     | 1     | 1     | 1     | 1     | 1     | 1      | 1      | 1      | 1      | 1      | 1      |
| mediumrc (m)                 | 1     | 2     | 2     | 4     | 4     | 4     | 8      | 8      | 8      | 16     | 16     | 32     |
| largerc (l)                  | 2     | 4     | 4     | 8     | 8     | 8     | 16     | 16     | 16     | 32     | 32     | 64     |
| xlargerc (xl)                | 4     | 8     | 8     | 16    | 16    | 16    | 32     | 32     | 32     | 64     | 64     | 128    |
-->

| 使用量 | DW100 | DW200 | DW300 | DW400 | DW500 | DW600 | DW1000 | DW1200 | DW1500 | DW2000 |
| :--------------------------- | :---- | :---- | :---- | :---- | :---- | :---- | :----- | :----- | :----- | :----- |
| 并发查询数上限 | 32 | 32 | 32 | 32 | 32 | 32 | 32 | 32 | 32 | 32 |
| 并发槽数上限 | 4 | 8 | 12 | 16 | 20 | 24 | 40 | 48 | 60 | 80 |
| smallrc(default) (s) | 1 | 1 | 1 | 1 | 1 | 1 | 1 | 1 | 1 | 1 |
| mediumrc (m) | 1 | 2 | 2 | 4 | 4 | 4 | 8 | 8 | 8 | 16 |
| largerc (l) | 2 | 4 | 4 | 8 | 8 | 8 | 16 | 16 | 16 | 32 |
| xlargerc (xl) | 4 | 8 | 8 | 16 | 16 | 16 | 32 | 32 | 32 | 64 |

### 异常

更高资源类的成员资格有时不改变分配给查询或操作的资源。这通常发生于执行操作所需的资源不足。在这些情况下，始终使用默认或小型资源类 (smallrc)，而不考虑分配给用户的资源类。例如，`CREATE LOGIN` 始终在 smallrc 中运行。执行此操作所需的资源非常少，因此没有必要将查询纳入并发槽模型中。预先分配大量存储给此操作是一种浪费。从并发槽模型中排除 `CREATE LOGIN` 能够大大提高 SQL 数据仓库的效率。

下面**是**资源类控制的语句和操作列表：

- INSERT-SELECT
- UPDATE
- 删除
- SELECT（查询用户表时）
- ALTER INDEX REBUILD
- ALTER INDEX REORGANIZE
- ALTER TABLE REBUILD
- CREATE INDEX
- CREATE CLUSTERED COLUMNSTORE INDEX
- CREATE TABLE AS SELECT
- 数据加载
- 数据移动服务 (DMS) 执行的数据移动操作

以下语句**不**接受资源类：

- CREATE TABLE
- ALTER TABLE ...SWITCH PARTITION
- ALTER TABLE ...SPLIT PARTITION
- ALTER TABLE ...MERGE PARTITION
- DROP TABLE
- ALTER INDEX DISABLE
- DROP INDEX
- CREATE STATISTICS
- UPDATE STATISTICS
- DROP STATISTICS
- TRUNCATE TABLE
- ALTER AUTHORIZATION
- CREATE LOGIN
- CREATE USER
- ALTER USER
- DROP USER
- CREATE PROCEDURE
- ALTER PROCEDURE
- DROP PROCEDURE
- CREATE VIEW
- DROP VIEW
- INSERT VALUES
- SELECT（从系统视图和 DMV）
- EXPLAIN
- DBCC

<!--
Removed as these two are not confirmed / supported under SQLDW
- CREATE REMOTE TABLE AS SELECT
- CREATE EXTERNAL TABLE AS SELECT
- REDISTRIBUTE
-->

> [AZURE.NOTE] 值得强调的是，只针对动态管理检视和目录检视执行的 `SELECT` 查询**不是**由资源类控制。

请务必记住，大多数用户查询很可能由资源类控制。一般而言，活动的查询工作负荷必须落在并发查询和并发槽阈值内，除非已被平台专门排除。最终用户无法选择从并发槽模型中排除查询。超出任何一个阈值后，查询将开始排队。排队的查询将依次根据优先顺序和提交时间予以解决。

### 内部

在 SQL 数据仓库的工作负荷管理幕后，情况有点复杂。资源类动态映射到资源调控器内的一组常规工作负荷管理组。使用的组取决于仓库的 DWU 值。但是，SQL 数据仓库总共使用八个工作负荷组。它们具有以下特点：

- SloDWGroupC00
- SloDWGroupC01
- SloDWGroupC02
- SloDWGroupC03
- SloDWGroupC04
- SloDWGroupC05
- SloDWGroupC06
- SloDWGroupC07

这 8 个组映射到并发槽使用量

| 工作负荷组 | 并发槽映射 | 优先级映射 |
| :------------  | :----------------------- | :--------------- |
| SloDWGroupC00 | 1 | 中型 |
| SloDWGroupC01 | 2 | 中型 |
| SloDWGroupC02 | 4 | 中型 |
| SloDWGroupC03 | 8 | 中型 |
| SloDWGroupC04 | 16 | 高 |
| SloDWGroupC05 | 32 | 高 |
| SloDWGroupC06 | 64 | 高 |
| SloDWGroupC07 | 128 | 高 |

因此，举例来说，如果 DW500 是 SQL 数据仓库当前的 DWU 设置，则活动的工作负荷组将映射到资源类，如下所示：

| 资源类 | 工作负荷组 | 使用的并发槽数 | 重要性 |
| :------------- | :------------- | :---------------------   | :--------- |
| smallrc | SloDWGroupC00 | 1 | 中型 |
| mediumrc | SloDWGroupC02 | 4 | 中型 |
| largerc | SloDWGroupC03 | 8 | 中型 |
| xlargerc | SloDWGroupC04 | 16 | 高 |

若要从资源调控器的立场查看详细信息中内存资源分配的差异，请使用以下查询：

```
WITH rg
AS
(   SELECT  pn.name AS node_name
	,		pn.[type] AS node_type
	,		pn.pdw_node_id AS node_id
	,		rp.name AS pool_name
    ,       rp.max_memory_kb*1.0/1024 AS pool_max_mem_MB
    ,       wg.name AS group_name
    ,       wg.importance AS group_importance
    ,       wg.request_max_memory_grant_percent	AS group_request_max_memory_grant_pcnt
    ,       wg.max_dop AS group_max_dop
    ,       wg.effective_max_dop AS group_effective_max_dop
	,		wg.total_request_count AS group_total_request_count
	,		wg.total_queued_request_count AS group_total_queued_request_count
	,		wg.active_request_count AS group_active_request_count
	,		wg.queued_request_count	AS group_queued_request_count
    FROM    sys.dm_pdw_nodes_resource_governor_workload_groups wg
    JOIN    sys.dm_pdw_nodes_resource_governor_resource_pools rp ON wg.pdw_node_id = rp.pdw_node_id
															        AND wg.pool_id = rp.pool_id
	JOIN	sys.dm_pdw_nodes pn										ON	wg.pdw_node_id = pn.pdw_node_id
	WHERE   wg.name like 'SloDWGroup%'
	AND     rp.name = 'SloDWPool'
)
SELECT	pool_name
,		pool_max_mem_MB
,		group_name
,		group_importance
,		(pool_max_mem_MB/100)*group_request_max_memory_grant_pcnt AS max_memory_grant_MB
,		node_name
,		node_type
,       group_total_request_count
,       group_total_queued_request_count
,       group_active_request_count
,       group_queued_request_count
FROM	rg
ORDER BY
	node_name
,	group_request_max_memory_grant_pcnt
,	group_importance
;
```

> [AZURE.NOTE] 上述查询还可以在故障排除时用来分析工作负荷组的活动使用量和历史使用量

## 工作负荷管理示例

本部分提供其他一些示例来说明如何管理用户和检测正在排队的查询。

### 管理用户

若要向某个用户授予对 SQL 数据仓库的访问权限，该用户首先需要一个登录名。

与 SQL 数据仓库的 master 数据库建立连接，然后执行以下命令：

```
CREATE LOGIN newperson WITH PASSWORD = 'mypassword'

CREATE USER newperson for LOGIN newperson
```

> [AZURE.NOTE] 最好为你在 Azure SQL 数据库和 Azure SQL 数据仓库的 master 数据库中的登录名创建用户。此级别有两个可用的服务器角色，需要登录才能在 master 数据库中拥有用户以授予成员资格。这些角色为 `Loginmanager` 和 `dbmanager`。在 Azure SQL 数据库和 SQL 数据仓库中，这些角色授予管理登录以及创建数据库的权限。这与 SQL Server 有所不同。有关详细信息，请参阅[在 Azure SQL 数据库中管理数据库和登录名]一文。

创建登录名后，需要添加用户帐户。

与 SQL 数据仓库数据库建立连接，然后执行以下命令：

```
CREATE USER newperson FOR LOGIN newperson
```

完成之后，必须向用户授予权限。以下示例授予对 SQL 数据仓库数据库的 `CONTROL` 权限。数据库级别的 `CONTROL` 相当于 SQL Server 中的 db\_owner。

```
GRANT CONTROL ON DATABASE::MySQLDW to newperson
```

若要查看工作负荷管理角色，请使用以下查询：

```
SELECT  ro.[name]           AS [db_role_name]
FROM    sys.database_principals ro
WHERE   ro.[type_desc]      = 'DATABASE_ROLE'
AND     ro.[is_fixed_role]  = 0
;
```

若要将用户添加到增加工作负荷管理角色，请使用以下查询：

```
EXEC sp_addrolemember 'largerc', 'newperson'
```

若要从工作负荷管理角色中删除用户，请使用以下查询：

```
EXEC sp_droprolemember 'largerc', 'newperson'
```

> [AZURE.NOTE] 无法从 smallrc 中删除用户。

若要查看哪些用户是给定角色的成员，请使用以下查询：

```
SELECT	r.name AS role_principal_name
,		m.name AS member_principal_name
FROM	sys.database_role_members rm
JOIN	sys.database_principals AS r			ON rm.role_principal_id		= r.principal_id
JOIN	sys.database_principals AS m			ON rm.member_principal_id	= m.principal_id
WHERE	r.name IN ('mediumrc','largerc', 'xlargerc')
;
```

### 排队的查询检测
若要识别保留在并发队列中的查询，始终可以引用 `sys.dm_pdw_exec_requests` DMV。

```
SELECT 	 r.[request_id]	AS Request_ID
		,r.[status]	AS Request_Status
		,r.[submit_time] AS Request_SubmitTime
		,r.[start_time]	AS Request_StartTime
        ,DATEDIFF(ms,[submit_time],[start_time]) AS Request_InitiateDuration_ms
        ,r.resource_class AS Request_resource_class
FROM    sys.dm_pdw_exec_requests r
;
```

SQL 数据仓库提供特定的等待类型来度量并发性。

它们具有以下特点：

- LocalQueriesConcurrencyResourceType
- UserConcurrencyResourceType
- DmsConcurrencyResourceType
- BackupConcurrencyResourceType

LocalQueriesConcurrencyResourceType 引用并发槽框架外部的查询。DMV 查询和 `SELECT @@VERSION` 等系统函数是本地查询的示例。

UserConcurrencyResourceType 引用并发槽框架内部的查询。针对最终用户表的查询代表使用此资源类型的示例。

DmsConcurrencyResourceType 引用数据移动操作生成的等待。

备份数据库时，可以看到 BackupConcurrencyResourceType。此资源类型的最大值为 1。如果同时请求了多个备份，其他备份将会排队。


若要分析当前排队的查询以确定请求正在等待哪些资源，请引用 `sys.dm_pdw_waits` DMV。

```
SELECT  w.[wait_id]
,       w.[session_id] 
,       w.[type] AS Wait_type
,       w.[object_type]
,       w.[object_name]
,       w.[request_id]
,       w.[request_time]
,       w.[acquire_time]
,       w.[state]
,       w.[priority]
,		SESSION_ID() AS Current_session
,		s.[status] AS Session_status
,		s.[login_name]
,		s.[query_count]
,		s.[client_id]
,		s.[sql_spid]
,		r.[command] AS Request_command
,		r.[label]
,		r.[status] AS Request_status
,		r.[submit_time]
,		r.[start_time]
,		r.[end_compile_time]
,		r.[end_time]
,		DATEDIFF(ms,r.[submit_time],r.[start_time]) AS Request_queue_time_ms
,		DATEDIFF(ms,r.[start_time],r.[end_compile_time]) AS Request_compile_time_ms
,		DATEDIFF(ms,r.[end_compile_time],r.[end_time]) AS Request_execution_time_ms
,		r.[total_elapsed_time]
FROM    sys.dm_pdw_waits w
JOIN    sys.dm_pdw_exec_sessions s ON w.[session_id] = s.[session_id]
JOIN    sys.dm_pdw_exec_requests r ON w.[request_id] = r.[request_id]
WHERE	w.[session_id] <> SESSION_ID()
;
```

若只要查看给定查询使用的资源等待，可以引用 `sys.dm_pdw_resource_waits` DMV。资源等待时间只度量等待提供资源的时间，与信号等待时间相反，后者是基础 SQL Server 将查询调度到 CPU 所需的时间。

```
SELECT  [session_id]
,       [type]
,       [object_type]
,       [object_name]
,       [request_id]
,       [request_time]
,       [acquire_time]
,       DATEDIFF(ms,[request_time],[acquire_time]) AS acquire_duration_ms
,       [concurrency_slots_used] AS concurrency_slots_reserved
,       [resource_class]
,       [wait_id] AS queue_position
FROM    sys.dm_pdw_resource_waits
WHERE	[session_id] <> SESSION_ID()
;
```

最后，SQL 数据仓库提供 `sys.dm_pdw_wait_stats` DMV 用于分析等待历史趋势。

```
SELECT	w.[pdw_node_id]
,		w.[wait_name]
,		w.[max_wait_time]
,		w.[request_count]
,		w.[signal_time]
,		w.[completed_count]
,		w.[wait_time]
FROM	sys.dm_pdw_wait_stats w
;
```

## 后续步骤
有关更多开发技巧，请参阅[开发概述][]。

<!--Image references-->

<!--Article references-->
[开发概述]: /documentation/articles/sql-data-warehouse-overview-develop/

<!--MSDN references-->
[在 Azure SQL 数据库中管理数据库和登录名]: /documentation/articles/sql-database-manage-logins/

<!--Other Web references-->

<!---HONumber=Mooncake_0328_2016-->