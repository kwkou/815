<properties
   pageTitle="使用 DMV 监视工作负荷 | Azure"
   description="了解如何使用 DMV 监视工作负荷。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="sonyam"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="07/22/2016"
   wacn.date="08/29/2016"/>


# 使用 DMV 监视工作负荷

本文介绍如何使用动态管理视图 (DMV) 在 Azure SQL 数据仓库中监视工作负荷及调查查询执行情况。

## 监视连接

[Sys.dm\_pdw\_exec\_sessions][] 视图允许你监视与 Azure SQL 数据仓库数据库的连接。此视图包含活动会话，以及最近断开连接的会话的历史记录。session\_id 是此视图的主键，并为每次新的登录按顺序分配。


	SELECT * FROM sys.dm_pdw_exec_sessions where status <> 'Closed';


## 监视查询执行

若要监视查询执行情况，请使用 [sys.dm\_pdw\_exec\_requests][] 启动。此视图包含进行中的查询，以及最近完成的查询的历史记录。request\_id 对每个查询进行唯一标识，并且为此视图的主键。按顺序对每个新的查询分配 request\_id。针对给定 session\_id 查询此表将显示给定登录的所有查询。

>[AZURE.NOTE] 存储过程使用多个 request\_id。将按顺序分配请求 id。

以下是调查特定查询的查询执行计划和时间所要遵循的步骤。

### 步骤 1：确定想要调查的查询


	-- Monitor active queries
	SELECT * 
	FROM sys.dm_pdw_exec_requests 
	WHERE status not in ('Completed','Failed','Cancelled')
	  AND session_id <> session_id()
	ORDER BY submit_time DESC;

	-- Find top 10 queries longest running queries
	SELECT TOP 10 * 
	FROM sys.dm_pdw_exec_requests 
	ORDER BY total_elapsed_time DESC;


从上面的查询结果中，记下想要调查的查询的**请求 ID**。

由于其中并发限制，处于暂停状态的查询正在排队，有关详细信息，请参阅[并发性和工作负荷管理][]。这些查询也将出现在具有 UserConcurrencyResourceType 类型的 sys.dm\_pdw\_waits 等待查询中。查询也可能因其他原因（如准备锁定）处于等待状态。如果查询正在等待资源，请参阅[调查等待资源的查询][]。

### 步骤 2：查找运行时间最长的查询计划步骤

使用请求 ID 从 [sys.dm\_pdw\_request\_steps][] 中检索查询计划步骤的列表。通过查看总已用时间，查找长时间运行的步骤。



	-- Find the distributed query plan steps for a specific query.
	-- Replace request_id with value from Step 1.

	SELECT * FROM sys.dm_pdw_request_steps
	WHERE request_id = 'QID####'
	ORDER BY step_index;


检查长时间运行的查询步骤的 *operation\_type* 列并记下**步骤索引**：

- 针对以下 **SQL 操作**继续执行步骤 3a：OnOperation、RemoteOperation、ReturnOperation。
- 针对以下**数据移动操作**继续执行步骤 3b：ShuffleMoveOperation、BroadcastMoveOperation、TrimMoveOperation、PartitionMoveOperation、MoveOperation、CopyOperation。

### 步骤 3a：查找 SQL 步骤的执行进度

使用请求 ID 和步骤索引来从 [sys.dm\_pdw\_sql\_requests][] 中检索详细信息，其中包含有关 SQL Server 分布式实例的查询执行情况的信息。


	-- Find the distribution run times for a SQL step.
	-- Replace request_id and step_index with values from Step 1 and 3.

	SELECT * FROM sys.dm_pdw_sql_requests
	WHERE request_id = 'QID####' AND step_index = 2;


如果查询正在运行，则可以使用 [DBCC PDW\_SHOWEXECUTIONPLAN][] 检索特定分发中当前正在运行的 SQL 步骤的 SQL Server 计划高速缓存中的 SQL Server 估计计划。


	-- Find the SQL Server execution plan for a query running on a specific SQL Data Warehouse Compute or Control node.
	-- Replace distribution_id and spid with values from previous query.

	DBCC PDW_SHOWEXECUTIONPLAN(1, 78);


### 步骤 3b：查找 DMS 步骤的执行进度

使用请求 ID 和步骤索引检索对 [sys.dm\_pdw\_dms\_workers][] 的每个分发运行的数据移动步骤的相关信息。


	-- Find the information about all the workers completing a Data Movement Step.
	-- Replace request_id and step_index with values from Step 1 and 3.

	SELECT * FROM sys.dm_pdw_dms_workers
	WHERE request_id = 'QID####' AND step_index = 2;


- 检查 *total\_elapsed\_time* 列，以查看是否有特定分布在数据移动上比其他分布花费了更多时间。
- 对于长时间运行的分布，请检查 *rows\_processed* 列，以查看从该分布移动的行数是否远远多于其他分布。如果是这样，这可能表示底层数据的偏斜。

如果查询正在运行，则可以使用 [DBCC PDW\_SHOWEXECUTIONPLAN][] 检索特定分发中当前正在运行的 SQL 步骤的 SQL Server 计划高速缓存中的 SQL Server 估计计划。


	-- Find the SQL Server estimated plan for a query running on a specific SQL Data Warehouse Compute or Control node.
	-- Replace distribution_id and spid with values from previous query.

	DBCC PDW_SHOWEXECUTIONPLAN(55, 238);

<a name="waiting"></a>
## 监视正在等待的查询

如果查询未取得进展（因其正在等待资源），下面是显示查询正在等待的所有资源的查询。


	-- Find queries 
	-- Replace request_id with value from Step 1.

	SELECT waits.session_id,
	      waits.request_id,  
	      requests.command,
	      requests.status,
	      requests.start_time,  
	      waits.type,
	      waits.state,
	      waits.object_type,
	      waits.object_name
	FROM   sys.dm_pdw_waits waits
	   JOIN  sys.dm_pdw_exec_requests requests
	   ON waits.request_id=requests.request_id
	WHERE waits.request_id = 'QID####'
	ORDER BY waits.object_name, waits.object_type, waits.state;


如果查询正在主动等待另一个查询中的资源，则状态将为 **AcquireResources**。如果查询具有全部所需资源，则状态将为 **Granted**。

## 后续步骤
有关动态管理视图 (DMV) 的详细信息，请参阅[系统视图][]。
有关管理 SQL 数据仓库的提示，请参阅[管理概述][]。
有关最佳实践，请参阅 [SQL 数据仓库最佳实践][]。

<!--Image references-->

<!--Article references-->
[管理概述]: /documentation/articles/sql-data-warehouse-overview-manage/
[SQL 数据仓库最佳实践]: /documentation/articles/sql-data-warehouse-best-practices/
[系统视图]: /documentation/articles/sql-data-warehouse-reference-tsql-system-views/
[并发性和工作负荷管理]: /documentation/articles/sql-data-warehouse-develop-concurrency/
[调查等待资源的查询]: /documentation/articles/sql-data-warehouse-manage-monitor#waiting/

<!--MSDN references-->
[sys.dm\_pdw\_dms\_workers]: http://msdn.microsoft.com/zh-cn/library/mt203878.aspx
[sys.dm\_pdw\_exec\_requests]: http://msdn.microsoft.com/zh-cn/library/mt203887.aspx
[Sys.dm\_pdw\_exec\_sessions]: http://msdn.microsoft.com/zh-cn/library/mt203883.aspx
[sys.dm\_pdw\_request\_steps]: http://msdn.microsoft.com/zh-cn/library/mt203913.aspx
[sys.dm\_pdw\_sql\_requests]: http://msdn.microsoft.com/zh-cn/library/mt203889.aspx
[DBCC PDW\_SHOWEXECUTIONPLAN]: http://msdn.microsoft.com/zh-cn/library/mt204017.aspx
[DBCC PDW_SHOWSPACEUSED]: http://msdn.microsoft.com/zh-cn/library/mt204028.aspx

<!---HONumber=Mooncake_0822_2016-->