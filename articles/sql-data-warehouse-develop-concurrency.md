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
   ms.date="07/15/2016"
   wacn.date="08/01/2016"/>

# SQL 数据仓库中的并发性和工作负荷管理

若要大规模提供可预测性能，可以通过 SQL 数据仓库让用户控制并发级别和资源分配（例如内存和 CPU 优先级）。本文将介绍并发性和工作负荷管理的概念，说明如何实现这两种功能，以及如何在数据仓库中控制它们。SQL 数据仓库工作负荷管理旨在协助你提供多用户环境支持。它不适用于多租户工作负荷。

## 并发限制

SQL 数据仓库允许多达 1,024 个并发连接。所有 1,024 个连接都可以通过并发方式提交查询。但是，为了优化吞吐量，SQL 数据仓库可能会让某些查询排队，以便确保每个查询都能收到系统授予的最小内存。在执行查询时会让查询排队。SQL 数据仓库可以在达到并发限制时让查询排队，以便确保活动查询能够访问紧张的内存资源，从而提高总吞吐量。

影响并发限制的有两个概念：**并发查询数**和**并发槽数**。执行查询时，必须满足查询并发限制，并且必须符合并发槽分配要求。

- 简单说来，**并发查询数**是指同时执行的查询的数目。就大型 DW（DW1000 及以上）来说，SQL 数据仓库支持最多 32 个**并发查询**。不过，由于并发查询数随 DWU 数而变，因此我们在下面提供了一个表，该表显示了随 DWU 而变的各种限制。
- **并发槽数**是一个更动态的概念。每个查询可以使用一个或多个并发槽。一个查询可以使用的具体槽数取决于 SQL 数据仓库的大小以及查询的[资源类](#resource-classes)。

下表描述了并发查询数和并发槽数的限制。

### 并发限制

| | DW100 | DW200 | DW300 | DW400 | DW500 | DW600 | DW1000 | DW1200 | DW1500 | DW2000 | DW3000 | DW6000 |
| :--------------------------- | ----: | ----: | ----: | ----: | ----: | ----: | -----: | -----: | -----: | -----: | -----: | -----: |
| 并发查询数上限 | 32 | 32 | 32 | 32 | 32 | 32 | 32 | 32 | 32 | 32 | 32 | 32 |
| 并发槽数上限 | 4 | 8 | 12 | 16 | 20 | 24 | 40 | 48 | 60 | 80 | 120 | 240 |

达到这其中的一个阈值后，系统就会让新查询排队。排队的查询会在其他查询执行完后按照先进先出的原则执行，查询数和槽数受到相应的限制。

> [AZURE.NOTE]  以独占方式在动态管理视图 (DMV) 或目录视图上执行的 SELECT 查询**不**受任何此类并发限制的约束。因此，用户可以对系统进行监视，而不用管在系统中执行的查询的数目。

## 资源类

资源类是 SQL 数据仓库工作负荷管理的重要部分，因为可以通过这些类将更多的内存和/或 CPU 周期分配给给定用户所运行的查询。共有四个可以通过**数据库角色**形式分配给用户的资源类。这四个资源类是：**smallrc、mediumrc、largerc、xlargerc**。smallrc 类的用户获得的内存量较小，因此提高了并发性。与之相反，分配给 xlargerc 类的用户获得的内存量大，因此可以并发运行的查询数较少。

默认情况下，每个用户都是小型资源类 (smallrc) 的成员。过程 `sp_addrolemember` 用于提高资源类的级别，过程 `sp_droprolemember` 用于降低资源类的级别。例如，以下命令会将 loaduser 的资源类提高到 largerc 级别：


	EXEC sp_addrolemember 'largerc', 'loaduser'


最好是先创建用户，然后将其永久分配给资源类，而不是更改用户的资源类。例如，加载到聚集列存储表时，如果分配了更多的内存，则可创建质量更高的索引。为了确保加载项能够访问更多的内存，可创建一个用户来专门加载数据，并将该用户永久分配给更高级的资源类。

有些类型的查询即使分配了更多的内存也没有用处，因此系统会忽略其资源类分配情况，而是始终按小型资源类的标准运行这些查询。强制这些查询始终按小型资源类的标准运行，则当并发槽数快要达到限制时，就可以运行这些查询，防止这些查询消耗过多的并发槽。这些[资源类例外情况](#resource-class-exceptions)在本文后面讲述。

关于资源类的更多详细信息：

- 需要 `ALTER ROLE` 权限才能更改用户的资源类。
- 可以将用户添加到一个或多个更高级别的资源类，但用户会呈现分配给他们的最高资源类的属性。也就是说，如果将用户同时分配给 mediumrc 和 largerc，则级别更高的资源类 largerc 将是用户所属的资源类。
- 不能更改系统管理用户的资源类。
 
有关如何创建用户并为其分配资源类的更多详细信息和示例，可参阅本文末尾的[管理用户](#Managing-users)部分。

## 内存分配

提高用户资源类的级别既有优点也有缺点。虽然提高用户的资源类级别意味着其查询可以访问更多的内存，其执行速度可以更快，但提高资源类的级别也会降低可以运行的并发查询数。这就需要进行权衡，你需要在分配大量内存给单个查询与允许其他查询并发运行（也需进行内存分配）之间进行权衡。如果允许某个用户优先为查询分配内存，则其他需要运行查询的用户将无法访问同一内存。

下表将所分配的内存与按 DWU 和资源类划分的每种分布情况进行了映射。在 SQL 数据仓库中，每个数据库有 60 种分布情况。例如，在 xlarge 资源类中，在 DW2000 上运行的查询可以访问 60 个分布式数据库中每个数据库的 6,400 MB 内存。

### 按分布情况进行的内存分配 (MB)

| | DW100 | DW200 | DW300 | DW400 | DW500 | DW600 | DW1000 | DW1200 | DW1500 | DW2000 | DW3000 | DW6000 |
| :------------- | ----: | ----: | ----: | ----: | ----: | ----: | -----: | -----: | -----: | -----: | -----: | -----: |
| smallrc | 100 | 100 | 100 | 100 | 100 | 100 | 100 | 100 | 100 | 100 | 100 | 100 |
| mediumrc | 100 | 200 | 200 | 400 | 400 | 400 | 800 | 800 | 800 | 1,600 | 1,600 | 3,200 |
| largerc | 200 | 400 | 400 | 800 | 800 | 800 | 1,600 | 1,600 | 1,600 | 3,200 | 3,200 | 6,400 |
| xlargerc | 400 | 800 | 800 | 1,600 | 1,600 | 1,600 | 3,200 | 3,200 | 3,200 | 6,400 | 6,400 | 12,800 |


从上面的同一个示例来看，在系统范围内，在 xlarge 资源类的 DW2000 上运行的查询被系统总共分配了 375 GB 的内存（6,400 MB * 60 种分布/1,024 即可转换为 GB）。

### 系统范围的内存分配 (GB)

| | DW100 | DW200 | DW300 | DW400 | DW500 | DW600 | DW1000 | DW1200 | DW1500 | DW2000 | DW3000 | DW6000 |
| :------------- | ----: | ----: | ----: | ----: | ----: | ----: | -----: | -----: | -----: | -----: | -----: | -----: |
|smallrc | 6 | 6 | 6 | 6 | 6 | 6 | 6 | 6 | 6 | 6 | 6 | 6 |
|mediumrc | 6 | 12 | 12 | 23 | 23 | 23 | 47 | 47 | 47 | 94 | 94 | 188 |
|largerc | 12 | 23 | 23 | 47 | 47 | 47 | 94 | 94 | 94 | 188 | 188 | 375 |
|xlargerc | 23 | 47 | 47 | 94 | 94 | 94 | 188 | 188 | 188 | 375 | 375 | 750 |

## 并发槽使用量

如上所述，资源类的级别越高，授予的内存越多。由于内存是一种固定的资源，因此每个查询分配的内存越多，能够支持的并发查询数就越少。下表在单个视图中重新阐述了上述所有概念，显示了可按 DWU 使用的并发槽数，以及可按每个资源类使用的槽数。

### 分配和使用并发槽

| | DW100 | DW200 | DW300 | DW400 | DW500 | DW600 | DW1000 | DW1200 | DW1500 | DW2000 | DW3000 | DW6000 |
| :---------------------- | ----: | ----: | ----: | ----: | ----: | ----: | -----: | -----: | -----: | -----: | -----: | -----: |
| **分配** | | | | | | | | | | | | |
| 并发查询数上限 | 32 | 32 | 32 | 32 | 32 | 32 | 32 | 32 | 32 | 32 | 32 | 32 |
| 并发槽数上限 | 4 | 8 | 12 | 16 | 20 | 24 | 40 | 48 | 60 | 80 | 120 | 240 |
| **槽使用情况** | | | | | | | | | | | | |
| smallrc | 1 | 1 | 1 | 1 | 1 | 1 | 1 | 1 | 1 | 1 | 1 | 1 |
| mediumrc | 1 | 2 | 2 | 4 | 4 | 4 | 8 | 8 | 8 | 16 | 16 | 32 |
| largerc | 2 | 4 | 4 | 8 | 8 | 8 | 16 | 16 | 16 | 32 | 32 | 64 |
| xlargerc | 4 | 8 | 8 | 16 | 16 | 16 | 32 | 32 | 32 | 64 | 64 | 128 |

从此表可以看到，以 DW1000 方式运行的 SQL 数据仓库总共提供了 40 个并发槽，最多可以处理 32 个并发查询。如果所有用户都是按小型资源类运行，则允许 32 个并发查询，因为每个查询会使用 1 个并发槽。如果所有用户都是按中型资源类运行，则会按分布情况为每个用户分配 800 MB 的内存，总共分配 47 GB 的内存，针对所有这些中型资源类用户的并发限制为 8 个用户。

## 查询重要性

事实上，总共有八个工作负荷组，控制着资源类的行为。但是，对于任何给定的 DWU 来说，八个工作负荷组中只使用了四个。这是因为，每个工作负荷组都分配了一个资源类，即 smallrc、mediumrc、largerc 或 xlargerc。了解这些幕后工作负荷组的重要性在于，这些工作负荷组的一部分已被设置了较高的**重要性**。重要性用于 CPU 计划。重要性为高的查询在运行时所获得的 CPU 周期数将是重要性为中的查询的 3 倍。因此，并发槽映射也决定了 CPU 重要性。如果某个查询使用了至少 16 个槽，则该查询是作为重要性高的查询运行的。

下面是每个工作负荷组的重要性映射。

| 工作负荷组 | 并发槽映射 | 重要性映射 |
| :------------------  | :----------------------: | :----------------- |
| SloDWGroupC00 | 1 | 中型 |
| SloDWGroupC01 | 2 | 中型 |
| SloDWGroupC02 | 4 | 中型 |
| SloDWGroupC03 | 8 | 中型 |
| SloDWGroupC04 | 16 | 高 |
| SloDWGroupC05 | 32 | 高 |
| SloDWGroupC06 | 64 | 高 |
| SloDWGroupC07 | 128 | 高 |

对 DW500 SQL 数据仓库来说，活动的工作负荷组将映射到资源类，如下所示。

| 资源类 | 工作负荷组 | 使用的并发槽数 | 重要性 |
| :--------------- | :------------- | :--------------------:   | :--------- |
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


## 资源类例外情况

大多数查询按资源类来进行，但也存在一些例外情况。这通常发生在执行操作所需的资源不够时。也就是说，当某个查询不会使用级别较高的资源类所分配的数量较大的内存时，通常会出现这种例外情况。在这些情况下，始终使用默认或小型资源类 (smallrc)，而不考虑分配给用户的资源类。例如，`CREATE LOGIN` 将始终按 smallrc 运行。执行此操作所需的资源非常少，因此没有必要将查询纳入并发槽模型中。预先分配大量存储给此操作是一种浪费。从并发槽模型中排除 `CREATE LOGIN` 能够大大提高 SQL 数据仓库的效率。

以下语句**不**接受资源类：

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

### 遵循并发限制的查询

请务必记住，大多数用户查询很可能由资源类控制。一般而言，活动的查询工作负荷必须落在并发查询和并发槽阈值内，除非已被平台专门排除。最终用户无法选择从并发槽模型中排除查询。超出任何一个阈值后，查询将开始排队。排队的查询将依次根据优先顺序和提交时间予以解决。

重申一下，以下语句**接受**资源类：

- INSERT-SELECT
- UPDATE
- DELTE
- SELECT（查询用户表时）
- ALTER INDEX REBUILD
- ALTER INDEX REORGANIZE
- ALTER TABLE REBUILD
- CREATE INDEX
- CREATE CLUSTERED COLUMNSTORE INDEX
- CREATE TABLE AS SELECT
- 数据加载
- 数据移动服务 (DMS) 执行的数据移动操作


## 管理用户

1. **创建登录名：**与 SQL 数据仓库的 **master** 数据库建立连接，然后执行以下命令。
	

		CREATE LOGIN newperson WITH PASSWORD = 'mypassword';
		CREATE USER newperson for LOGIN newperson;


	> [AZURE.NOTE] 最好为你在 Azure SQL 数据库和 Azure SQL 数据仓库的 master 数据库中的登录名创建用户。此级别有两个可用的服务器角色，需要登录才能在 master 数据库中拥有用户以授予成员资格。这些角色为 `Loginmanager` 和 `dbmanager`。在 Azure SQL 数据库和 SQL 数据仓库中，这些角色授予管理登录以及创建数据库的权限。这与 SQL Server 有所不同。有关详细信息，请参阅[在 Azure SQL 数据库中管理数据库和登录名][]一文。

2. **创建用户帐户：**与 **SQL 数据仓库数据库**建立连接，然后执行以下命令。


		CREATE USER newperson FOR LOGIN newperson


3. **授予权限：**以下示例授予对 SQL 数据仓库数据库的 `CONTROL` 权限。数据库级别的 `CONTROL` 相当于 SQL Server 中的 db\_owner。


		GRANT CONTROL ON DATABASE::MySQLDW to newperson;


4. **提高资源类的级别：**若要将用户添加到增加工作负荷管理角色，请使用以下查询。


		EXEC sp_addrolemember 'largerc', 'newperson'


5. **降低资源类的级别：**若要将用户从工作负荷管理角色中删除，请使用以下查询。


		EXEC sp_droprolemember 'largerc', 'newperson'


	> [AZURE.NOTE] 无法从 smallrc 中删除用户。

## 对排队的查询进行的检测，以及其他 DMV

可以使用 `sys.dm_pdw_exec_requests` DMV 来确定在并发队列中等待的查询。


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


SQL 数据仓库具有以下等待类型。

- LocalQueriesConcurrencyResourceType：位于并发槽框架外部的查询。DMV 查询和 `SELECT @@VERSION` 等系统函数是本地查询的示例。
- UserConcurrencyResourceType：位于并发槽框架内部的查询。针对最终用户表的查询代表使用此资源类型的示例。
- DmsConcurrencyResourceType：数据移动操作生成的等待
- BackupConcurrencyResourceType：此等待表明正在备份数据库。此资源类型的最大值为 1。如果同时请求了多个备份，其他备份将会排队。

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
[在 Azure SQL 数据库中管理数据库和登录名]: https://msdn.microsoft.com/zh-cn/library/azure/ee336235.aspx

<!--Other Web references-->

<!---HONumber=Mooncake_0725_2016-->