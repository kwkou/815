<properties
   pageTitle="故障排除 | Azure"
   description="排查 SQL 数据仓库问题。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="sonyam"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="05/15/2016"
   wacn.date="05/23/2016"/>

# 故障排除
以下主题列出了客户在使用 Azure SQL 数据仓库时经常遇到的一些问题。

## 连接
出于以下一些常见原因，连接到 Azure SQL 数据仓库可能失败：

- 未设置防火墙规则
- 使用不支持的工具/协议

### 防火墙规则
为了确保只有已知的 IP 地址可以访问数据库，Azure SQL 数据库受到服务器和数据库级别的防火墙保护。默认情况下，防火墙受到保护 - 这意味着，你必须允许你的 IP 地址访问数据库才能建立连接。

若要设置防火墙的访问权限，请遵循[预配](/documentation/articles/sql-data-warehouse-get-started-provision-powershell/)页上的[为客户端 IP 配置服务器防火墙访问权限](/documentation/articles/sql-data-warehouse-get-started-provision-powershell/#step-4-configure-server-firewall-access-for-your-client-ip)部分中所述的步骤。

### 使用不支持的工具/协议
SQL 数据仓库支持使用 [Visual Studio 2013/2015](/documentation/articles/sql-data-warehouse-get-started-connect/) 作为开发环境，支持使用 [SQL Server Native Client 10/11 (ODBC)](https://msdn.microsoft.com/zh-cn/library/ms131415.aspx) 来连接客户端。

有关详细信息，请参阅我们的[连接](/documentation/articles/sql-data-warehouse-get-started-connect/)页。

## 查询性能
SQL 数据仓库使用普通的 SQL Server 构造来执行查询（包括统计信息）。[统计信息](/documentation/articles/sql-data-warehouse-develop-statistics/)是包含数据库列中值范围和频率相关信息的对象。查询引擎使用这些统计信息优化查询执行以及提高查询性能。你可以使用以下查询来确定上次更新统计信息的时间。

```
SELECT
	sm.[name]								    AS [schema_name],
	tb.[name]								    AS [table_name],
	co.[name]									AS [stats_column_name],
	st.[name]									AS [stats_name],
	STATS_DATE(st.[object_id],st.[stats_id])	AS [stats_last_updated_date]
FROM
	sys.objects				AS ob
	JOIN sys.stats			AS st	ON	ob.[object_id]		= st.[object_id]
	JOIN sys.stats_columns	AS sc	ON	st.[stats_id]		= sc.[stats_id]
									AND	st.[object_id]		= sc.[object_id]
	JOIN sys.columns		AS co	ON	sc.[column_id]		= co.[column_id]
									AND	sc.[object_id]		= co.[object_id]
	JOIN sys.types           AS ty	ON	co.[user_type_id]	= ty.[user_type_id]
	JOIN sys.tables          AS tb	ON	co.[object_id]		= tb.[object_id]
	JOIN sys.schemas         AS sm	ON	tb.[schema_id]		= sm.[schema_id]
WHERE
	1=1 
	AND st.[user_created] = 1;
```

有关详细信息，请参阅我们的[统计信息](/documentation/articles/sql-data-warehouse-develop-statistics/)页。

## 关键性能概念

请参阅以下文章，以帮助了解其他一些关键的性能与缩放性概念：

- [性能和缩放性][]
- [并发模型][]
- [设计表][]
- [为表选择哈希分布键][]
- [用于改善性能的统计信息][]

## 后续步骤
请参阅[开发概述][]一文，以获取有关构建 SQL 数据仓库解决方案的一些指导。

<!--Image references-->

<!--Article references-->

[性能和缩放性]: /documentation/articles/sql-data-warehouse-performance-scale/
[并发模型]: /documentation/articles/sql-data-warehouse-develop-concurrency/
[设计表]: /documentation/articles/sql-data-warehouse-develop-table-design/
[为表选择哈希分布键]: /documentation/articles/sql-data-warehouse-develop-hash-distribution-key/
[用于改善性能的统计信息]: /documentation/articles/sql-data-warehouse-develop-statistics/
[开发概述]: /documentation/articles/sql-data-warehouse-overview-develop/

<!--MSDN references-->

<!--Other web references-->

<!---HONumber=Mooncake_0307_2016-->