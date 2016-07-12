<properties
   pageTitle="在 Azure SQL 数据仓库中查找长时间运行的用户查询 | Azure"
   description="了解如何使用动态管理视图 (DMV) 在 Azure SQL 数据仓库中监视工作负荷和查找长时间运行的查询。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="sonyam"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="05/03/2016"
   wacn.date="06/13/2016"/>

# 查找长时间运行的查询

本文介绍如何使用动态管理视图 (DMV) 在 Azure SQL 数据仓库中监视工作负荷及调查查询执行情况。

## 监视连接

[Sys.dm\_pdw\_exec\_sessions][] 视图允许你监视与 Azure SQL 数据仓库数据库的连接。此视图包含活动会话，以及最近断开连接的会话的历史记录。session\_id 是此视图的主键，并为每次新的登录按顺序分配。

```sql
SELECT * FROM sys.dm_pdw_exec_sessions where status <> 'Closed';
```

## 调查查询执行情况
若要监视查询执行情况，请使用 [sys.dm\_pdw\_exec\_requests][] 启动。此视图包含进行中的查询，以及最近完成的查询的历史记录。request\_id 对每个查询进行唯一标识，并且为此视图的主键。按顺序对每个新的查询分配 request\_id。针对给定 session\_id 查询此表将显示给定登录的所有查询。

在你想要调查特定查询的查询执行情况的时，以下是一些要遵循的常见步骤。

### 步骤 1：查找要调查的查询

```sql
-- Monitor running queries
SELECT * FROM sys.dm_pdw_exec_requests WHERE status = 'Running';

-- Find 10 queries which ran the longest
SELECT TOP 10 * FROM sys.dm_pdw_exec_requests ORDER BY total_elapsed_time DESC;
```

请记下你想要调查的查询的请求 ID。

### 步骤 2：检查查询是否正在等待资源

```sql
-- Find waiting tasks for your session.
-- Replace request_id with value from Step 1.

SELECT waits.session_id,
      waits.request_id,  
      requests.command,
      requests.status,
      requests.start_time,  
      waits.type,  
      waits.object_type,
      waits.object_name,  
      waits.state  
FROM   sys.dm_pdw_waits waits
   JOIN  sys.dm_pdw_exec_requests requests
   ON waits.request_id=requests.request_id
WHERE waits.request_id = 'QID33188'
ORDER BY waits.object_name, waits.object_type, waits.state;
```

上述查询的结果将显示查询的等待状态。

- 如果查询正在等待另一个查询的资源，则状态将为 **AcquireResources**。
- 如果查询具有全部所需资源且不在等待，则状态将为 **Granted**。在这种情况下，请转到查询步骤。

### 步骤 3：查找运行时间最长的查询计划步骤

使用请求 ID 从 [sys.dm\_pdw\_request\_steps][] 中检索查询计划步骤的列表。通过查看总已用时间，查找长时间运行的步骤。

```sql

-- Find the distributed query plan steps for a specific query.
-- Replace request_id with value from Step 1.

SELECT * FROM sys.dm_pdw_request_steps
WHERE request_id = 'QID33209'
ORDER BY step_index;
```

保存长时间运行步骤的步骤索引。

检查长时间运行的查询步骤的 operation\_type 列：

- 针对以下 **SQL 操作**继续执行步骤 4a：OnOperation、RemoteOperation、ReturnOperation。
- 针对以下**数据移动操作**继续执行步骤 4b：ShuffleMoveOperation、BroadcastMoveOperation、TrimMoveOperation、PartitionMoveOperation、MoveOperation、CopyOperation。

### 步骤 4a：查找 SQL 步骤的执行进度

使用请求 ID 和步骤索引来从 [sys.dm\_pdw\_sql\_requests][] 中检索信息，其中包含有关 SQL Server 分布式实例的查询执行情况的详细信息。如果查询仍在运行并且你想要从 SQL Server 分发获取计划，请记下分发 ID 和 SPID。

```sql
-- Find the distribution run times for a SQL step.
-- Replace request_id and step_index with values from Step 1 and 3.

SELECT * FROM sys.dm_pdw_sql_requests
WHERE request_id = 'QID33209' AND step_index = 2;
```


如果查询当前正在运行，则可以使用 [DBCC PDW\_SHOWEXECUTIONPLAN][] 检索特定分发的当前正在运行的 SQL 步骤的 SQL Server 执行计划。

```sql
-- Find the SQL Server execution plan for a query running on a specific SQL Data Warehouse Compute or Control node.
-- Replace distribution_id and spid with values from previous query.

DBCC PDW_SHOWEXECUTIONPLAN(1, 78);

```

### 步骤 4b：查找 DMS 步骤的执行进度

使用请求 ID 和步骤索引检索对 [sys.dm\_pdw\_dms\_workers][] 的每个分发运行的数据移动步骤的相关信息。

```sql
-- Find the information about all the workers completing a Data Movement Step.
-- Replace request_id and step_index with values from Step 1 and 3.

SELECT * FROM sys.dm_pdw_dms_workers
WHERE request_id = 'QID33209' AND step_index = 2;

```

- 检查 total\_elapsed\_time 列，以查看是否有特定分布在数据移动上比其他分布花费了更多时间。
- 对于长时间运行的分布，请检查 rows\_processed 列，以查看从该分布移动的行数是否远远多于其他分布。如果是这样，这可能表示底层数据的偏斜。

如果查询当前正在运行，则可以使用 [DBCC PDW\_SHOWEXECUTIONPLAN][] 检索特定分发的当前正在运行的 DMS 步骤的 SQL Server 执行计划。

```sql
-- Find the SQL Server execution plan for a query running on a specific SQL Data Warehouse Compute or Control node.
-- Replace distribution_id and spid with values from previous query.

DBCC PDW_SHOWEXECUTIONPLAN(55, 238);

```

## 后续步骤
有关动态管理视图 (DMV) 的详细信息，请参阅[系统视图][]。  
有关管理 SQL 数据仓库的提示，请参阅[管理概述][]。  
有关最佳实践，请参阅 [SQL 数据仓库最佳实践][]。

<!--Image references-->

<!--Article references-->
[manage data skew for distributed tables]: /documentation/articles/sql-data-warehouse-manage-distributed-data-skew/
[管理概述]: /documentation/articles/sql-data-warehouse-overview-manage/
[SQL 数据仓库最佳实践]: /documentation/articles/sql-data-warehouse-best-practices/
[系统视图]: /documentation/articles/sql-data-warehouse-reference-tsql-system-views/

<!--MSDN references-->
[sys.dm\_pdw\_dms\_workers]: http://msdn.microsoft.com/zh-cn/library/mt203878.aspx
[sys.dm\_pdw\_exec\_requests]: http://msdn.microsoft.com/zh-cn/library/mt203887.aspx
[Sys.dm\_pdw\_exec\_sessions]: http://msdn.microsoft.com/zh-cn/library/mt203883.aspx
[sys.dm\_pdw\_request\_steps]: http://msdn.microsoft.com/zh-cn/library/mt203913.aspx
[sys.dm\_pdw\_sql\_requests]: http://msdn.microsoft.com/zh-cn/library/mt203889.aspx
[DBCC PDW\_SHOWEXECUTIONPLAN]: http://msdn.microsoft.com/zh-cn/library/mt204017.aspx
[DBCC PDW_SHOWSPACEUSED]: http://msdn.microsoft.com/zh-cn/library/mt204028.aspx

<!---HONumber=Mooncake_0606_2016-->