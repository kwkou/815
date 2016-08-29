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
   ms.date="08/02/2016"
   wacn.date="08/29/2016"/>


# SQL 数据仓库中的并发性和工作负荷管理

若要大规模提供可预测性能，可以通过 Azure SQL 数据仓库控制并发级别和资源分配（例如内存和 CPU 优先级）。本文将介绍并发性和工作负荷管理的概念，说明如何实现这两种功能，以及如何在数据仓库中控制它们。SQL 数据仓库工作负荷管理旨在协助您提供多用户环境支持。它不适用于多租户工作负荷。

## 并发限制

SQL 数据仓库允许多达 1,024 个并发连接。所有 1,024 个连接都可以通过并发方式提交查询。但是，为了优化吞吐量，SQL 数据仓库可能会让某些查询排队，以便确保每个查询都能收到系统授予的最小内存。在执行查询时会让查询排队。SQL 数据仓库可以在达到并发限制时让查询排队，以便确保活动查询能够访问迫切需要的内存资源，从而提高总吞吐量。

影响并发限制的有两个概念：*并发查询*和*并发槽*。执行查询时，必须符合查询并发限制和并发槽分配要求。

- 并发查询是指同时执行的查询。SQL 数据仓库支持高达 32 个并发查询。
- 根据 DWU 分配并发槽。每 100 个 DWU 提供 4 个并发槽。例如，DW100 分配 4 个并发槽，DW1000 分配 40 个。每个查询使用一个或多个并发槽，具体取决于查询的[资源类](#resource-classes)。在 smallrc 资源类中运行的查询占用一个并发槽。在更高资源类中运行的查询将占用更多并发槽。

下表描述了不同 DWU 大小的并发查询数和并发槽数的限制。

### 并发限制

| DWU | 并发查询数上限 | 分配的并发槽数 |
| :----  | :---------------------: | :-------------------------: |
| DW100 | 32 | 4 |
| DW200 | 32 | 8 |
| DW300 | 32 | 12 |
| DW400 | 32 | 16 |
| DW500 | 32 | 20 |
| DW600 | 32 | 24 |
| DW1000 | 32 | 40 |
| DW1200 | 32 | 48 |
| DW1500 | 32 | 60 |
| DW2000 | 32 | 80 |
| DW3000 | 32 | 120 |
| DW6000 | 32 | 240 |

达到其中一个阈值后，将排成新的查询队列。SQL 数据仓库会在其他查询执行完后，并且查询数和槽数减少到限制数以下时，按照先进先出的原则执行排队的查询。

> [AZURE.NOTE]  以独占方式在动态管理视图 (DMV) 或目录视图上执行的 *Select* 查询不受任何并发限制的约束。用户可以对系统进行监视，而不用考虑在系统中执行的查询的数目。

## 资源类

资源类有助于控制针对查询的内存分配和 CPU 周期。可以*数据库角色*的形式向用户分配四个资源类。这四个资源类是：**smallrc**、**mediumrc**、**largerc** 和 **xlargerc**。smallrc 类的用户获得的内存量较小，但是可以利用更高的并发性。与之相反，分配给 xlargerc 类的用户获得的内存量大，因此可以并发运行的查询数较少。

默认情况下，每个用户都是小型资源类 (smallrc) 的成员。过程 `sp_addrolemember` 用于提高资源类的级别，过程 `sp_droprolemember` 用于降低资源类的级别。例如，以下命令会将 loaduser 的资源类提高到 largerc 级别：


	EXEC sp_addrolemember 'largerc', 'loaduser'


较好的做法是将用户永久分配给资源类，而不是更改用户的资源类。例如，加载到聚集列存储表时，如果分配了更多的内存，则可创建质量更高的索引。为了确保加载项能够访问更多的内存，可创建一个用户来专门加载数据，并将该用户永久分配给更高级的资源类。

有几种类型的查询不会得益于更大的内存分配。系统将忽略其资源类分配，而始终运行小资源类中的这些查询。如果这些查询始终在小资源类中运行，则当并发槽数紧张时仍可以运行这些查询，它们不会占用比所需的更多的槽。有关详细信息，请参阅[资源类例外](#query-exceptions-to-concurrency-limits)。

关于资源类的更多详细信息：

- 需要*更改角色*权限才能更改用户的资源类。
- 虽然可以将用户添加到一个或多个更高级别的资源类，但用户会呈现分配给他们的最高资源类的属性。也就是说，如果将用户同时分配给 mediumrc 和 largerc，则级别更高的资源类 (largerc) 将是用户所属的资源类。
- 不能更改系统管理用户的资源类。

有关详细的示例，请参阅[更改用户资源类示例](#changing-user-resource-class-example)。

## 内存分配

提高用户资源类的级别既有优点也有缺点。虽然提高用户的资源类级别意味着其查询可以访问更多的内存，其执行速度可以更快，但较高的资源类级别也会降低可以运行的并发查询数。这就需要在分配大量内存给单个查询与允许其他查询并发运行（也需进行内存分配）之间进行权衡。如果为某个用户提供了用于查询的高内存分配，则其他用户将无法访问同一内存来运行查询。

下表将所分配的内存与按 DWU 和资源类划分的每种分布情况进行了映射。在 SQL 数据仓库中，有 60 种分布情况。例如，在 xlargerc 资源类的 DW2000 上运行的查询在 60 个分布式数据库的每个数据库中可以访问 6,400 MB 内存。

### 按分布情况进行的内存分配 (MB)

| DWU | smallrc | mediumrc | largerc | xlargerc |
| :----- | :-----: | :------: | :-----: | :------: |
| DW100 | 100 | 100 | 200 | 400 |
| DW200 | 100 | 200 | 400 | 800 |
| DW300 | 100 | 200 | 400 | 800 |
| DW400 | 100 | 400 | 800 | 1,600 |
| DW500 | 100 | 400 | 800 | 1,600 |
| DW600 | 100 | 400 | 800 | 1,600 |
| DW1000 | 100 | 800 | 1,600 | 3,200 |
| DW1200 | 100 | 800 | 1,600 | 3,200 |
| DW1500 | 100 | 800 | 1,600 | 3,200 |
| DW2000 | 100 | 1,600 | 3,200 | 6,400 |
| DW3000 | 100 | 1,600 | 3,200 | 6,400 |
| DW6000 | 100 | 3,200 | 6,400 | 12,800 |

在上面的示例中，从 SQL 数据仓库的整体来看，在 xlargerc 资源类的 DW2000 上运行的查询总共分配了 375 GB 的内存（6,400 MB * 60 种分布/1,024 即可转换为 GB）。

### 系统范围的内存分配 (GB)

| DWU | smallrc | mediumrc | largerc | xlargerc |
| :----- | :-----: | :------: | :-----: | :------: |
| DW100 | 6 | 6 | 12 | 23 |
| DW200 | 6 | 12 | 23 | 47 |
| DW300 | 6 | 12 | 23 | 47 |
| DW400 | 6 | 23 | 47 | 94 |
| DW500 | 6 | 23 | 47 | 94 |
| DW600 | 6 | 23 | 47 | 94 |
| DW1000 | 6 | 47 | 94 | 188 |
| DW1200 | 6 | 47 | 94 | 188 |
| DW1500 | 6 | 47 | 94 | 188 |
| DW2000 | 6 | 94 | 188 | 375 |
| DW3000 | 6 | 94 | 188 | 375 |
| DW6000 | 6 | 188 | 375 | 750 |


## 并发槽使用量

SQL 数据仓库对在更高资源类级别中运行的查询赋予了更多内存。由于内存是一种固定的资源，因此每个查询分配的内存越多，能够支持的并发查询数就越少。下表在单个视图中重述了上述所有概念，显示了按 DWU 分配的并发槽数，以及每个资源类使用的槽数。

### 分配和使用并发槽

| DWU | 并发查询数上限 | 分配的并发槽数 | smallrc 使用的槽数 | mediumrc 使用的槽数 | largerc 使用的槽数 | xlargerc 使用的槽数 |
| :----  | :---------------------: | :-------------------------: | :-----: | :------: | :-----: | :------: |
| DW100 | 32 | 4 | 1 | 1 | 2 | 4 |
| DW200 | 32 | 8 | 1 | 2 | 4 | 8 |
| DW300 | 32 | 12 | 1 | 2 | 4 | 8 |
| DW400 | 32 | 16 | 1 | 4 | 8 | 16 |
| DW500 | 32 | 20 | 1 | 4 | 8 | 16 |
| DW600 | 32 | 24 | 1 | 4 | 8 | 16 |
| DW1000 | 32 | 40 | 1 | 8 | 16 | 32 |
| DW1200 | 32 | 48 | 1 | 8 | 16 | 32 |
| DW1500 | 32 | 60 | 1 | 8 | 16 | 32 |
| DW2000 | 32 | 80 | 1 | 16 | 32 | 64 |
| DW3000 | 32 | 120 | 1 | 16 | 32 | 64 |
| DW6000 | 32 | 240 | 1 | 32 | 64 | 128 |


从上面的表格中可以看到，以 DW1000 运行的 SQL 数据仓库可分配最多 32 个并发查询和总共 40 个并发槽。如果所有用户都在 smallrc 中运行，则允许 32 个并发查询，因为每个查询只使用 1 个并发槽。如果 DW1000 上的所有用户均在 mediumrc 中运行，则在每个分布中对每个查询分配 800 MB 内存，每个查询将总共分配 47 GB 内存，同时将用户的并发数限制为 5 个（40 个并发槽 / 每个 mediumrc 用户 8 个槽）。

## 查询重要性

SQL 数据仓库通过使用工作负荷组来实现资源类。总共有八个工作负荷组用于控制不同 DWU 大小的资源类的行为。对于任何 DWU，SQL 数据仓库只使用八个工作负荷组中的四个。这样做是合理的，因为每个工作负荷组将被分配到以下四个资源类中的一个：smallrc、mediumrc、largerc 或 xlargerc。了解工作负荷组的重要性在于其中一些工作负荷组已设置为较高的*重要性*。重要性用于 CPU 计划。重要性为高的查询在运行时所获得的 CPU 周期数将是重要性为中的查询的 3 倍。因此，并发槽映射也决定了 CPU 优先级。如果某个查询使用了至少 16 个槽，则该查询是作为重要性高的查询运行的。

下表显示了每个工作负荷组的重要性映射。

### 并发槽和重要性的工作负荷组映射

| 工作负荷组 | 并发槽映射 | 重要性映射 |
| :-------------- | :----------------------: | :----------------- |
| SloDWGroupC00 | 1 | 中型 |
| SloDWGroupC01 | 2 | 中型 |
| SloDWGroupC02 | 4 | 中型 |
| SloDWGroupC03 | 8 | 中型 |
| SloDWGroupC04 | 16 | 高 |
| SloDWGroupC05 | 32 | 高 |
| SloDWGroupC06 | 64 | 高 |
| SloDWGroupC07 | 128 | 高 |

从**分配和使用并发槽**图中可以看到，对于 smallrc、mediumrc、largerc 和 xlargerc，DW500 分别使用 1、4、8 和 16 个并发槽。您可以在上面的图中查找这些值，以找到每个资源类的重要性。

### DW500 的资源类到重要性的映射

| 资源类 | 工作负荷组 | 使用的并发槽数 | 重要性 |
| :------------- | :------------- | :--------------------: | :--------- |
| smallrc | SloDWGroupC00 | 1 | 中型 |
| mediumrc | SloDWGroupC02 | 4 | 中型 |
| largerc | SloDWGroupC03 | 8 | 中型 |
| xlargerc | SloDWGroupC04 | 16 | 高 |


在进行故障诊断时，可以使用以下 DMV 查询，从资源调控器的角度来详细查看内存资源分配的差异，或者分析工作负荷组目前的和历史上的使用情况：


	WITH rg
	AS
	(   SELECT  pn.name									AS node_name
		,		pn.[type]								AS node_type
		,		pn.pdw_node_id							AS node_id
		,		rp.name									AS pool_name
	    ,       rp.max_memory_kb*1.0/1024				AS pool_max_mem_MB
	    ,       wg.name									AS group_name
	    ,       wg.importance							AS group_importance
	    ,       wg.request_max_memory_grant_percent		AS group_request_max_memory_grant_pcnt
	    ,       wg.max_dop								AS group_max_dop
	    ,       wg.effective_max_dop					AS group_effective_max_dop
		,		wg.total_request_count					AS group_total_request_count
		,		wg.total_queued_request_count			AS group_total_queued_request_count
		,		wg.active_request_count					AS group_active_request_count
		,		wg.queued_request_count					AS group_queued_request_count
	    FROM    sys.dm_pdw_nodes_resource_governor_workload_groups wg
	    JOIN    sys.dm_pdw_nodes_resource_governor_resource_pools rp    ON  wg.pdw_node_id  = rp.pdw_node_id
																        AND wg.pool_id      = rp.pool_id
		JOIN	sys.dm_pdw_nodes pn										ON	wg.pdw_node_id	= pn.pdw_node_id
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

## 遵循并发限制的查询

大多数查询都受资源类的约束。这些查询必须同时不超出并发查询和并发槽的阈值。用户无法选择从并发槽模型中排除查询。

重申一下，以下语句接受资源类：

- INSERT-SELECT
- UPDATE
- 删除
- SELECT（查询用户表时）
- ALTER INDEX REBUILD
- ALTER INDEX REORGANIZE
- ALTER TABLE REBUILD
- CREATE INDEX
- CREATE CLUSTERED COLUMNSTORE INDEX
- CREATE TABLE AS SELECT (CTAS)
- 数据加载
- 数据移动服务 (DMS) 执行的数据移动操作

## 并发限制的查询例外

有些查询不能接受将用户分配到的资源类。当特定命令所需的内存资源较低时（通常是因为该命令是元数据操作），会产生并发限制的例外情况。这些例外的产生是为了避免为始终不需要较大内存的查询分配较大内存。在这些情况下，无论分配给用户的实际资源类是什么，都将始终使用默认的小型资源类 (smallrc)。例如，`CREATE LOGIN` 将始终在 smallrc 中运行。执行此操作所需的资源非常少，因此没有必要将查询纳入并发槽模型中。预先分配大量存储给此操作是一种浪费。从并发槽模型中排除 `CREATE LOGIN` 能够大大提高 SQL 数据仓库的效率。

以下语句不接受资源类：

- CREATE 或 DROP TABLE
- ALTER TABLE ...SWITCH、SPLIT 或 MERGE PARTITION
- ALTER INDEX DISABLE
- DROP INDEX
- CREATE、UPDATE 或 DROP STATISTICS
- TRUNCATE TABLE
- ALTER AUTHORIZATION
- CREATE LOGIN
- CREATE、ALTER 或 DROP USER
- CREATE、ALTER 或 DROP PROCEDURE
- CREATE 或 DROP VIEW
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

## 更改用户资源类示例

1. **创建登录名：**与 SQL 数据仓库的 **master** 数据库建立连接，然后执行以下命令。


	CREATE LOGIN newperson WITH PASSWORD = 'mypassword'; 
	CREATE USER newperson for LOGIN newperson;


	> [AZURE.NOTE] 最好为你在 Azure SQL 数据库和 Azure SQL 数据仓库的 master 数据库中的登录名创建用户。此级别有两个可用的服务器角色，这两个角色需要登录名拥有在 **master** 数据库中的用户才能授予成员资格。这些角色为 `Loginmanager` 和 `dbmanager`。在 Azure SQL 数据库和 SQL 数据仓库中，这两个角色授予管理登录名和创建数据库的权限。这与 SQL Server 有所不同。有关详细信息，请参阅 [Managing Databases and Logins in Azure SQL Database][]（在 Azure SQL 数据库中管理数据库和登录名）。

2. **创建用户帐户：**与 **SQL 数据仓库**数据库建立连接，然后执行以下命令。


		CREATE USER newperson FOR LOGIN newperson


3. **授予权限：**以下示例授予对 **SQL 数据仓库**数据库的 `CONTROL` 权限。数据库级别的 `CONTROL` 相当于 SQL Server 中的 db\_owner。


		GRANT CONTROL ON DATABASE::MySQLDW to newperson;


4. **提高资源类的级别：**若要将用户添加到更高级别的工作负荷管理角色，请使用以下查询。


		EXEC sp_addrolemember 'largerc', 'newperson'


5. **降低资源类的级别：**若要将用户从工作负荷管理角色中删除，请使用以下查询。


		EXEC sp_droprolemember 'largerc', 'newperson'


	> [AZURE.NOTE] 无法从 smallrc 中删除用户。

## 对排队的查询进行的检测，以及其他 DMV

可以使用 `sys.dm_pdw_exec_requests` DMV 来确定在并发队列中等待的查询。正在等待并发槽的查询的状态为**挂起**。


	SELECT 	 r.[request_id]									AS Request_ID
			,r.[status]										AS Request_Status
			,r.[submit_time]								AS Request_SubmitTime
			,r.[start_time]									AS Request_StartTime
	        ,DATEDIFF(ms,[submit_time],[start_time])		AS Request_InitiateDuration_ms
	        ,r.resource_class                               AS Request_resource_class
	FROM    sys.dm_pdw_exec_requests r
	;


可以使用 `sys.database_principals` 来查看工作负荷管理角色。


	SELECT  ro.[name]           AS [db_role_name]
	FROM    sys.database_principals ro
	WHERE   ro.[type_desc]      = 'DATABASE_ROLE'
	AND     ro.[is_fixed_role]  = 0;


以下查询显示分配给每个用户的角色。


	SELECT	r.name AS role_principal_name
	,		m.name AS member_principal_name
	FROM	sys.database_role_members rm
	JOIN	sys.database_principals AS r			ON rm.role_principal_id		= r.principal_id
	JOIN	sys.database_principals AS m			ON rm.member_principal_id	= m.principal_id
	WHERE	r.name IN ('mediumrc','largerc', 'xlargerc');


SQL 数据仓库具有以下等待类型：

- **LocalQueriesConcurrencyResourceType**：位于并发槽框架外部的查询。DMV 查询和 `SELECT @@VERSION` 等系统函数是本地查询的示例。
- **UserConcurrencyResourceType**：位于并发槽框架内部的查询。针对最终用户表的查询代表使用此资源类型的示例。
- **DmsConcurrencyResourceType**：数据移动操作生成的等待。
- **BackupConcurrencyResourceType**：此等待表明正在备份数据库。此资源类型的最大值为 1。如果同时请求了多个备份，其他备份将会排队。

可以使用 `sys.dm_pdw_waits` DMV 来查看请求所等待的具体资源。


	SELECT  w.[wait_id]
	,       w.[session_id]
	,       w.[type]											AS Wait_type
	,       w.[object_type]
	,       w.[object_name]
	,       w.[request_id]
	,       w.[request_time]
	,       w.[acquire_time]
	,       w.[state]
	,       w.[priority]
	,		SESSION_ID()										AS Current_session
	,		s.[status]											AS Session_status
	,		s.[login_name]
	,		s.[query_count]
	,		s.[client_id]
	,		s.[sql_spid]
	,		r.[command]											AS Request_command
	,		r.[label]
	,		r.[status]											AS Request_status
	,		r.[submit_time]
	,		r.[start_time]
	,		r.[end_compile_time]
	,		r.[end_time]
	,		DATEDIFF(ms,r.[submit_time],r.[start_time])			AS Request_queue_time_ms
	,		DATEDIFF(ms,r.[start_time],r.[end_compile_time])	AS Request_compile_time_ms
	,		DATEDIFF(ms,r.[end_compile_time],r.[end_time])		AS Request_execution_time_ms
	,		r.[total_elapsed_time]
	FROM    sys.dm_pdw_waits w
	JOIN    sys.dm_pdw_exec_sessions s  ON w.[session_id] = s.[session_id]
	JOIN    sys.dm_pdw_exec_requests r  ON w.[request_id] = r.[request_id]
	WHERE	w.[session_id] <> SESSION_ID()
	;


`sys.dm_pdw_resource_waits` DMV 仅显示给定查询所占用的资源等待。资源等待时间只度量等待提供资源的时间，与信号等待时间相反，后者是基础 SQL Server 将查询调度到 CPU 所需的时间。


	SELECT  [session_id]
	,       [type]
	,       [object_type]
	,       [object_name]
	,       [request_id]
	,       [request_time]
	,       [acquire_time]
	,       DATEDIFF(ms,[request_time],[acquire_time])  AS acquire_duration_ms
	,       [concurrency_slots_used]                    AS concurrency_slots_reserved
	,       [resource_class]
	,       [wait_id]                                   AS queue_position
	FROM    sys.dm_pdw_resource_waits
	WHERE	[session_id] <> SESSION_ID();


可以使用 `sys.dm_pdw_wait_stats` DMV 对等待进行历史趋势分析。


	SELECT	w.[pdw_node_id]
	,		w.[wait_name]
	,		w.[max_wait_time]
	,		w.[request_count]
	,		w.[signal_time]
	,		w.[completed_count]
	,		w.[wait_time]
	FROM	sys.dm_pdw_wait_stats w;


## 后续步骤

有关如何管理数据库用户和安全性的详细信息，请参阅[保护 SQL 数据仓库中的数据库][]。有关如何通过更大型资源类来改进聚集列存储索引质量的详细信息，请参阅[重新生成索引以提高段质量]。

<!--Image references-->


<!--Article references-->
[保护 SQL 数据仓库中的数据库]: /documentation/articles/sql-data-warehouse-overview-manage-security/
[重新生成索引以提高段质量]: /documentation/articles/sql-data-warehouse-tables-index#rebuilding-indexes-to-improve-segment-quality/

<!--MSDN references-->
[Managing Databases and Logins in Azure SQL Database]: https://msdn.microsoft.com/zh-cn/library/azure/ee336235.aspx

<!--Other Web references-->

<!---HONumber=Mooncake_0822_2016-->