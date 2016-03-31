<properties
   pageTitle="使用 DMV 监视工作负荷 | Azure"
   description="了解如何使用 DMV 监视工作负荷。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="sahaj08"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="01/07/2016"
   wacn.date="03/17/2016"/>

# 使用 DMV 监视工作负荷

本文介绍如何使用动态管理视图 (DMV) 在 Azure SQL 数据仓库中监视工作负荷及调查查询执行情况。



## 监视连接

你可以使用 *sys.dm\_pdw\_nodes\_exec\_connections* 视图来检索有关与 Azure SQL 数据仓库数据库建立的连接的信息。此外，*sys.dm\_exec\_sessions* 视图在检索有关所有活动用户连接的信息时非常有用。

```

SELECT * FROM sys.dm_pdw_nodes_exec_connections;
SELECT * FROM sys.dm_pdw_nodes_exec_sessions;

```


使用以下查询可以检索有关当前连接的信息。

```

SELECT * 
FROM sys.dm_pdw_nodes_exec_connections AS c 
   JOIN sys.dm_pdw_nodes_exec_sessions AS s 
   ON c.session_id = s.session_id 
WHERE c.session_id = @@SPID;

```





## 调查查询执行情况
你可能会遇到查询未完成，或运行时间超过预期的情况。在这种情况下，你可以使用以下步骤收集数据，并缩小问题的范围。



### 步骤 1：查找要调查的查询

```

-- Monitor running queries
SELECT * FROM sys.dm_pdw_exec_requests WHERE status = 'Running';

-- Find the longest running queries
SELECT * FROM sys.dm_pdw_exec_requests ORDER BY total_elapsed_time DESC;

```

保存查询的请求 ID。


  
### 步骤 2：检查查询是否正在等待资源

```

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


上述查询的结果将显示请求的等待状态。

- 如果查询正在等待另一个查询的资源，则状态将为 **AcquireResources**。
- 如果查询具有全部所需资源且不在等待，则状态将为 **Granted**。在这种情况下，请转到查询步骤。




### 步骤 3：查找运行时间最长的查询步骤

使用请求 ID 检索所有分布式查询步骤的列表。通过查看总已用时间，查找长时间运行的步骤。

```

-- Find the distributed query plan steps for a specific query.
-- Replace request_id with value from Step 1.
 
SELECT * FROM sys.dm_pdw_request_steps
WHERE request_id = 'QID33209'
ORDER BY step_index;

```

保存长时间运行步骤的步骤索引。

检查长时间运行的查询步骤的 *operation\_type* 列：

- 针对以下 **SQL 操作**继续执行步骤 4a：OnOperation、RemoteOperation、ReturnOperation。
- 针对以下**数据移动操作**继续执行步骤 4b：ShuffleMoveOperation、BroadcastMoveOperation、TrimMoveOperation、PartitionMoveOperation、MoveOperation、CopyOperation。




### 步骤 4a：查找 SQL 步骤的执行进度

使用请求 ID 和步骤索引，将 SQL Server 查询分布的相关信息检索为查询中 SQL 步骤的一部分。保存分发 ID 和 SPID。

```

-- Find the distribution run times for a SQL step.
-- Replace request_id and step_index with values from Step 1 and 3.

SELECT * FROM sys.dm_pdw_sql_requests
WHERE request_id = 'QID33209' AND step_index = 2;

```


使用以下查询在特定节点上检索 SQL 步骤的 SQL Server 执行计划。

```

-- Find the SQL Server execution plan for a query running on a specific SQL Data Warehouse Compute or Control node. 
-- Replace distribution_id and spid with values from previous query.

DBCC PDW_SHOWEXECUTIONPLAN(1, 78);

```



### 步骤 4b：查找 DMS 步骤的执行进度

使用请求 ID 和步骤索引检索对每个分布运行的数据移动步骤的相关信息。

```

-- Find the information about all the workers completing a Data Movement Step.
-- Replace request_id and step_index with values from Step 1 and 3.
 
SELECT * FROM sys.dm_pdw_dms_workers
WHERE request_id = 'QID33209' AND step_index = 2;

```

- 检查 *total\_elapsed\_time* 列，以查看是否有特定分布在数据移动上比其他分布花费了更多时间。 
- 对于长时间运行的分布，请检查 *rows\_processed* 列，以查看从该分布移动的行数是否远远多于其他分布。这会显示你的查询出现了数据偏斜。





## 调查数据偏斜

```

-- Find data skew for a distributed table
DBCC PDW_SHOWSPACEUSED("dbo.FactInternetSales");

```


此查询的结果将显示存储在数据库中每组 60 个分布内的表行数目。为了获得最佳性能，分布式表中的行应该平均分散在所有分布区中。
若要了解详细信息，请参阅[表设计][]。



## 后续步骤
有关管理 SQL 数据仓库的更多技巧，请参阅[管理概述][]。

<!--Image references-->

<!--Article references-->
[管理概述]: /documentation/articles/sql-data-warehouse-overview-manage
[表设计]: /documentation/articles/sql-data-warehouse-develop-table-design

<!--MSDN references-->

<!---HONumber=Mooncake_0307_2016-->