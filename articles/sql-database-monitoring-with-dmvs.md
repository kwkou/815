<properties
   pageTitle="使用动态管理视图监视 Azure SQL 数据库 | Azure"
   description="了解如何通过使用动态管理视图检测常见性能问题来监视 Azure SQL 数据库。"
   services="sql-database"
   documentationCenter=""
   authors="BYHAM"
   manager="jeffreyg"
   editor=""
   tags=""/>

<tags
   ms.service="sql-database"
   ms.date="01/22/2016"
   wacn.date="05/16/2016"/>

# 使用动态管理视图监视 Azure SQL 数据库

Azure SQL 数据库支持通过一部分动态管理视图来诊断性能问题，这些问题可能由查询受阻或长时间运行、资源瓶颈、不良查询计划等原因造成。本主题提供有关如何通过使用动态管理视图检测常见性能问题的信息。

SQL 数据库部分支持三种类别的动态管理视图：

- 与数据库相关的动态管理视图。
- 与执行相关的动态管理视图。
- 与事务相关的动态管理视图。

有关动态管理视图的详细信息，请参阅 SQL Server 联机丛书中的[动态管理视图和功能 (Transact-SQL)](https://msdn.microsoft.com/zh-cn/library/ms188754.aspx)。

## 权限

在 SQL 数据库中，查询动态管理视图需要 **VIEW DATABASE STATE** 权限。**VIEW DATABASE STATE** 权限返回有关当前数据库中的所有对象的信息。
若要向特定数据库用户授予 **VIEW DATABASE STATE**权限，请运行以下查询：

```GRANT VIEW DATABASE STATE TO database_user; ```

在本地 SQL Server 的实例中，动态管理视图会返回服务器状态信息。在 SQL 数据库中，这些视图会返回只与当前逻辑数据库相关的信息。

## 计算数据库大小

下面的查询返回你的数据库的大小（以 MB 为单位）：

```
-- Calculates the size of the database.
SELECT SUM(reserved_page_count)*8.0/1024
FROM sys.dm_db_partition_stats;
GO
```

下面的查询将返回数据库中各个对象的大小（以 MB 为单位）：

```
-- Calculates the size of individual database objects.
SELECT sys.objects.name, SUM(reserved_page_count) * 8.0 / 1024
FROM sys.dm_db_partition_stats, sys.objects
WHERE sys.dm_db_partition_stats.object_id = sys.objects.object_id
GROUP BY sys.objects.name;
GO
```

## 监视连接

你可以使用 [sys.dm\_exec\_connections](https://msdn.microsoft.com/zh-cn/library/ms181509.aspx) 视图来检索有关特定的 Azure SQL 数据库服务器所建立连接的信息以及每个连接的详细信息。此外，[sys.dm\_exec\_sessions](https://msdn.microsoft.com/zh-cn/library/ms176013.aspx) 视图在检索有关所有活动用户连接和内部任务的信息时非常有用。
下面的查询将检索当前连接上的信息：

```
SELECT
    c.session_id, c.net_transport, c.encrypt_option,
    c.auth_scheme, s.host_name, s.program_name,
    s.client_interface_name, s.login_name, s.nt_domain,
    s.nt_user_name, s.original_login_name, c.connect_time,
    s.login_time
FROM sys.dm_exec_connections AS c
JOIN sys.dm_exec_sessions AS s
    ON c.session_id = s.session_id
WHERE c.session_id = @@SPID;
```

> [AZURE.NOTE] 在执行 **sys.dm_exec_requests** 和 **sys.dm_exec_sessions** 视图时，如果用户具有对数据库的 **VIEW DATABASE STATE** 权限，用户将会看到数据库上所有正在执行的会话；否则，用户只会看到当前会话。

## 监视查询性能

缓慢或长时间运行的查询会消耗大量系统资源。本部分演示如何使用动态管理视图来检测一些常见的查询性能问题。一个较旧但仍很有帮助的故障排除参考是 Microsoft TechNet 上的[排查 SQL Server 2008 中的性能问题](http://download.microsoft.com/download/D/B/D/DBDE7972-1EB9-470A-BA18-58849DB3EB3B/TShootPerfProbs2008.docx)这篇文章。

### 查找前 n 个查询

下列示例返回了按平均 CPU 时间排名的前五个查询的信息。该示例根据查询散列收集了查询，以便逻辑上等值的查询能够根据累积资源消耗分组。

```
SELECT TOP 5 query_stats.query_hash AS "Query Hash",
    SUM(query_stats.total_worker_time) / SUM(query_stats.execution_count) AS "Avg CPU Time",
    MIN(query_stats.statement_text) AS "Statement Text"
FROM
    (SELECT QS.*,
    SUBSTRING(ST.text, (QS.statement_start_offset/2) + 1,
    ((CASE statement_end_offset
        WHEN -1 THEN DATALENGTH(ST.text)
        ELSE QS.statement_end_offset END
            - QS.statement_start_offset)/2) + 1) AS statement_text
     FROM sys.dm_exec_query_stats AS QS
     CROSS APPLY sys.dm_exec_sql_text(QS.sql_handle) as ST) as query_stats
GROUP BY query_stats.query_hash
ORDER BY 2 DESC;
```

### 监视受阻的查询

缓慢或长时间运行的查询会导致过多的资源消耗并会导致查询受阻。受阻的原因可能是应用程序设计欠佳、查询计划不良、缺乏有用的索引等。你可以使用 sys.dm\_tran\_locks 视图来获取有关 Azure SQL 数据库中当前锁定活动的信息。有关示例代码，请参阅 SQL Server 联机丛书中的 [sys.dm\_tran\_locks (Transact-SQL)](https://msdn.microsoft.com/zh-cn/library/ms190345.aspx)。

### 监视查询计划

低效的查询计划还可能会增加 CPU 占用率。下面的示例使用 [sys.dm\_exec\_query\_stats](https://msdn.microsoft.com/zh-cn/library/ms189741.aspx) 视图来确定哪一查询累积的 CPU 占用率最高。

```
SELECT
    highest_cpu_queries.plan_handle,
    highest_cpu_queries.total_worker_time,
    q.dbid,
    q.objectid,
    q.number,
    q.encrypted,
    q.[text]
FROM
    (SELECT TOP 50
        qs.plan_handle,
        qs.total_worker_time
    FROM
        sys.dm_exec_query_stats qs
    ORDER BY qs.total_worker_time desc) AS highest_cpu_queries
    CROSS APPLY sys.dm_exec_sql_text(plan_handle) AS q
ORDER BY highest_cpu_queries.total_worker_time DESC;
```

## 另请参阅

[SQL 数据库简介](/documentation/articles/sql-database-technical-overview/)
<!---HONumber=Mooncake_0321_2016-->
