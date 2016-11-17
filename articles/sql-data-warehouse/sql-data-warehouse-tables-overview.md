<properties
   pageTitle="SQL 数据仓库中的表的概述 | Azure"
   description="Azure SQL 数据仓库表入门。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="sonyam"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.devlang="NA"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="data-services"
   ms.date="08/04/2016"
   wacn.date="09/05/2016"
   ms.author="sonyama;barbkess;jrj"/>  


# 概述 SQL 数据仓库中的表

> [AZURE.SELECTOR]
- [概述][]
- [数据类型][]
- [分布][]
- [索引][]
- [Partition][]
- [统计信息][]
- [临时][]

在 Azure SQL 数据仓库中创建表的入门操作很简单。基本的 [CREATE TABLE][] 语法与常用语法无异，这种语法你在使用其他数据库时很可能已经很熟悉了。创建表时，只需为表和列命名，然后为每个列定义数据类型即可。如果你已经在其他数据库中创建过表，则此操作对你来说应该很熟悉。


    CREATE TABLE Customers (FirstName VARCHAR(25), LastName VARCHAR(25))


以上示例创建名为 Customers 的表，该表包含两个列：FirstName 和 LastName。每个列都定义了数据类型 VARCHAR(25)，其数据限制为 25 个字符。表的这些基本属性以及其他属性大多与其他数据库相同。每个列都定义了数据类型，确保数据的完整性。可以通过添加索引来减少 I/O，从而改进性能。需要修改数据时，可通过添加分区来改进性能。

[重命名][RENAME] SQL 数据仓库表的操作如下所示：


    RENAME OBJECT Customer TO CustomerOrig; 


## 分布式表

由 SQL 数据仓库之类的分布式系统引入的新的基本属性是**分布列**。分布列的含义正如其名。分布列是指决定后台数据如何分布或划分的列。如果你在创建表时未指定分布列，该表的数据会自动根据**轮循机制**进行分布。虽然在某些情况下轮循机制表可能已经足够，但是定义分布列可以大大减少查询期间的数据移动，从而优化性能。请参阅[分布表][Distribute]，以详细了解如何选择分布列。

## 对表进行索引和分区

当你在使用 SQL 数据仓库的过程中变得更老练以后，如果你想要优化性能，则需了解有关表设计的详细信息。若要了解详细信息，请参阅有关[表数据类型][Data Types]、[分布表][Distribute]、[为表编制索引][Index]和[将表分区][Partition]的文章。

## 表统计信息

若要获取 SQL 数据仓库的最佳性能，统计信息异常重要。由于 SQL 数据仓库不会自动为你创建和更新统计信息（这可能与你在 Azure SQL 数据库中遇到的情况一样），因此请阅读我们的有关[统计信息][]的文章。该文章可能是你需要阅读的最重要的文章之一，可以确保你获得最佳查询性能。

## 临时表

临时表是指仅在你登录期间存在且其他用户无法查看的表。临时表可用于防止他人查看临时结果，并且不需清除。由于临时表也利用本地存储，因此对于某些操作来说，临时表可以提供更快速的性能。请参阅[临时表][Temporary]的文章，了解有关临时表的更多详细信息。

## 外部表

外部表，也称 PolyBase 表，是指可以从 SQL 数据仓库查询但其指向的数据却位于 SQL 数据仓库外部的表。例如，你可以创建一个外部表，让其指向 Azure Blob 存储上的文件。有关如何创建和查询外部表的更多详细信息，请参阅[使用 PolyBase 加载数据][]。

## 不支持的表功能

虽然 SQL 数据仓库包含许多与其他数据库提供的表功能相同的表功能，但也有一些功能是不受支持的。下面是目前仍不支持的部分表功能的列表。

| 不支持的功能 |
| --- |
|[标识属性][]（请参阅[分配代理键解决方法][]）|
|主键、外键、Unique 和 Check [表约束][]|
|[唯一索引][]|
|[计算列][]|
|[稀疏列][]|
|[用户定义的类型][]|
|[序列][]|
|[触发器][]|
|[索引视图][]|
|[同义词][]|

## 表大小查询

若要确定这 60 个分布中每个分布的表所占用的空间和行，一个简单的方法是使用 [DBCC PDW\_SHOWSPACEUSED][]。


    DBCC PDW_SHOWSPACEUSED('dbo.FactInternetSales');


但是，使用 DBCC 命令可能会受到很大限制。使用动态管理视图 (DMV)，你可以查看更多详细信息，并可对查询结果进行更多控制。一开始请创建此视图，我们在本文以及其他文章中的许多示例将引用此视图。


	CREATE VIEW dbo.vTableSizes
	AS
	WITH base
	AS
	(
	SELECT 
	 GETDATE()                                                             AS  [execution_time]
	, DB_NAME()                                                            AS  [database_name]
	, s.name                                                               AS  [schema_name]
	, t.name                                                               AS  [table_name]
	, QUOTENAME(s.name)+'.'+QUOTENAME(t.name)                              AS  [two_part_name]
	, nt.[name]                                                            AS  [node_table_name]
	, ROW_NUMBER() OVER(PARTITION BY nt.[name] ORDER BY (SELECT NULL))     AS  [node_table_name_seq]
	, tp.[distribution_policy_desc]                                        AS  [distribution_policy_name]
	, c.[name]                                                             AS  [distribution_column]
	, nt.[distribution_id]                                                 AS  [distribution_id]
	, i.[type]                                                             AS  [index_type]
	, i.[type_desc]                                                        AS  [index_type_desc]
	, nt.[pdw_node_id]                                                     AS  [pdw_node_id]
	, pn.[type]                                                            AS  [pdw_node_type]
	, pn.[name]                                                            AS  [pdw_node_name]
	, di.name                                                              AS  [dist_name]
	, di.position                                                          AS  [dist_position]
	, nps.[partition_number]                                               AS  [partition_nmbr]
	, nps.[reserved_page_count]                                            AS  [reserved_space_page_count]
	, nps.[reserved_page_count] - nps.[used_page_count]                    AS  [unused_space_page_count]
	, nps.[in_row_data_page_count] 
	    + nps.[row_overflow_used_page_count] 
	    + nps.[lob_used_page_count]                                        AS  [data_space_page_count]
	, nps.[reserved_page_count] 
	 - (nps.[reserved_page_count] - nps.[used_page_count]) 
	 - ([in_row_data_page_count] 
	         + [row_overflow_used_page_count]+[lob_used_page_count])       AS  [index_space_page_count]
	, nps.[row_count]                                                      AS  [row_count]
	from 
	    sys.schemas s
	INNER JOIN sys.tables t
	    ON s.[schema_id] = t.[schema_id]
	INNER JOIN sys.indexes i
	    ON  t.[object_id] = i.[object_id]
	    AND i.[index_id] <= 1
	INNER JOIN sys.pdw_table_distribution_properties tp
	    ON t.[object_id] = tp.[object_id]
	INNER JOIN sys.pdw_table_mappings tm
	    ON t.[object_id] = tm.[object_id]
	INNER JOIN sys.pdw_nodes_tables nt
	    ON tm.[physical_name] = nt.[name]
	INNER JOIN sys.dm_pdw_nodes pn
	    ON  nt.[pdw_node_id] = pn.[pdw_node_id]
	INNER JOIN sys.pdw_distributions di
	    ON  nt.[distribution_id] = di.[distribution_id]
	INNER JOIN sys.dm_pdw_nodes_db_partition_stats nps
	    ON nt.[object_id] = nps.[object_id]
	    AND nt.[pdw_node_id] = nps.[pdw_node_id]
	    AND nt.[distribution_id] = nps.[distribution_id]
	LEFT OUTER JOIN (select * from sys.pdw_column_distribution_properties where distribution_ordinal = 1) cdp
	    ON t.[object_id] = cdp.[object_id]
	LEFT OUTER JOIN sys.columns c
	    ON cdp.[object_id] = c.[object_id]
	    AND cdp.[column_id] = c.[column_id]
	)
	, size
	AS
	(
	SELECT
	   [execution_time]
	,  [database_name]
	,  [schema_name]
	,  [table_name]
	,  [two_part_name]
	,  [node_table_name]
	,  [node_table_name_seq]
	,  [distribution_policy_name]
	,  [distribution_column]
	,  [distribution_id]
	,  [index_type]
	,  [index_type_desc]
	,  [pdw_node_id]
	,  [pdw_node_type]
	,  [pdw_node_name]
	,  [dist_name]
	,  [dist_position]
	,  [partition_nmbr]
	,  [reserved_space_page_count]
	,  [unused_space_page_count]
	,  [data_space_page_count]
	,  [index_space_page_count]
	,  [row_count]
	,  ([reserved_space_page_count] * 8.0)                                 AS [reserved_space_KB]
	,  ([reserved_space_page_count] * 8.0)/1000                            AS [reserved_space_MB]
	,  ([reserved_space_page_count] * 8.0)/1000000                         AS [reserved_space_GB]
	,  ([reserved_space_page_count] * 8.0)/1000000000                      AS [reserved_space_TB]
	,  ([unused_space_page_count]   * 8.0)                                 AS [unused_space_KB]
	,  ([unused_space_page_count]   * 8.0)/1000                            AS [unused_space_MB]
	,  ([unused_space_page_count]   * 8.0)/1000000                         AS [unused_space_GB]
	,  ([unused_space_page_count]   * 8.0)/1000000000                      AS [unused_space_TB]
	,  ([data_space_page_count]     * 8.0)                                 AS [data_space_KB]
	,  ([data_space_page_count]     * 8.0)/1000                            AS [data_space_MB]
	,  ([data_space_page_count]     * 8.0)/1000000                         AS [data_space_GB]
	,  ([data_space_page_count]     * 8.0)/1000000000                      AS [data_space_TB]
	,  ([index_space_page_count]  * 8.0)                                   AS [index_space_KB]
	,  ([index_space_page_count]  * 8.0)/1000                              AS [index_space_MB]
	,  ([index_space_page_count]  * 8.0)/1000000                           AS [index_space_GB]
	,  ([index_space_page_count]  * 8.0)/1000000000                        AS [index_space_TB]
	FROM base
	)
	SELECT * 
	FROM size
	;


### 表空间摘要

此查询返回行以及按表划分的空间。此查询适用于查看哪些表是你最大的表，以及这些表是按轮循机制分布的还是按哈希分布的。对于哈希分布表，此查询还显示分布列。大多数情况下，最大的表应该是哈希分布，并使用聚集列存储索引。


	SELECT 
	     database_name
	,    schema_name
	,    table_name
	,    distribution_policy_name
	,	  distribution_column
	,    index_type_desc
	,    COUNT(distinct partition_nmbr) as nbr_partitions
	,    SUM(row_count)                 as table_row_count
	,    SUM(reserved_space_GB)         as table_reserved_space_GB
	,    SUM(data_space_GB)             as table_data_space_GB
	,    SUM(index_space_GB)            as table_index_space_GB
	,    SUM(unused_space_GB)           as table_unused_space_GB
	FROM 
	    dbo.vTableSizes
	GROUP BY 
	     database_name
	,    schema_name
	,    table_name
	,    distribution_policy_name
	,	  distribution_column
	,    index_type_desc
	ORDER BY
	    table_reserved_space_GB desc
	;


### 按分布类型划分的表空间


	SELECT 
	     distribution_policy_name
	,    SUM(row_count)                as table_type_row_count
	,    SUM(reserved_space_GB)        as table_type_reserved_space_GB
	,    SUM(data_space_GB)            as table_type_data_space_GB
	,    SUM(index_space_GB)           as table_type_index_space_GB
	,    SUM(unused_space_GB)          as table_type_unused_space_GB
	FROM dbo.vTableSizes
	GROUP BY distribution_policy_name
	;


### 按索引类型划分的表空间


	SELECT 
	     index_type_desc
	,    SUM(row_count)                as table_type_row_count
	,    SUM(reserved_space_GB)        as table_type_reserved_space_GB
	,    SUM(data_space_GB)            as table_type_data_space_GB
	,    SUM(index_space_GB)           as table_type_index_space_GB
	,    SUM(unused_space_GB)          as table_type_unused_space_GB
	FROM dbo.vTableSizes
	GROUP BY index_type_desc
	;


### 分布空间摘要


    SELECT 
        distribution_id
    ,    SUM(row_count)                as total_node_distribution_row_count
    ,    SUM(reserved_space_MB)        as total_node_distribution_reserved_space_MB
    ,    SUM(data_space_MB)            as total_node_distribution_data_space_MB
    ,    SUM(index_space_MB)            as total_node_distribution_index_space_MB
    ,    SUM(unused_space_MB)        as total_node_distribution_unused_space_MB
    FROM dbo.vTableSizes
    GROUP BY     distribution_id
    ORDER BY    distribution_id
    ;


## 后续步骤

若要了解详细信息，请参阅有关[表数据类型][Data Types]、[分布表][Distribute]、[为表编制索引][Index]、[将表分区][Partition]和[维护表统计信息][Statistics] 的文章。有关最佳实践的详细信息，请参阅 [SQL 数据仓库最佳实践][]。

<!--Image references-->

<!--Article references-->
[概述]: /documentation/articles/sql-data-warehouse-tables-overview/
[Data Types]: /documentation/articles/sql-data-warehouse-tables-data-types/
[数据类型]: /documentation/articles/sql-data-warehouse-tables-data-types/
[Distribute]: /documentation/articles/sql-data-warehouse-tables-distribute/
[分布]: /documentation/articles/sql-data-warehouse-tables-distribute/
[Index]: /documentation/articles/sql-data-warehouse-tables-index/
[索引]: /documentation/articles/sql-data-warehouse-tables-index/
[Partition]: /documentation/articles/sql-data-warehouse-tables-partition/
[Statistics]: /documentation/articles/sql-data-warehouse-tables-statistics/
[统计信息]: /documentation/articles/sql-data-warehouse-tables-statistics/
[Temporary]: /documentation/articles/sql-data-warehouse-tables-temporary/
[临时]: /documentation/articles/sql-data-warehouse-tables-temporary/
[SQL 数据仓库最佳实践]: /documentation/articles/sql-data-warehouse-best-practices/
[使用 PolyBase 加载数据]: /documentation/articles/sql-data-warehouse-load-from-azure-blob-storage-with-polybase/

<!--MSDN references-->
[CREATE TABLE]: https://msdn.microsoft.com/zh-cn/library/mt203953.aspx
[RENAME]: https://msdn.microsoft.com/zh-cn/library/mt631611.aspx
[DBCC PDW_SHOWSPACEUSED]: https://msdn.microsoft.com/zh-cn/library/mt204028.aspx
[标识属性]: https://msdn.microsoft.com/zh-cn/library/ms186775.aspx
[分配代理键解决方法]: https://blogs.msdn.microsoft.com/sqlcat/2016/02/18/assigning-surrogate-key-to-dimension-tables-in-sql-dw-and-aps/
[表约束]: https://msdn.microsoft.com/zh-cn/library/ms188066.aspx
[计算列]: https://msdn.microsoft.com/zh-cn/library/ms186241.aspx
[稀疏列]: https://msdn.microsoft.com/zh-cn/library/cc280604.aspx
[用户定义的类型]: https://msdn.microsoft.com/zh-cn/library/ms131694.aspx
[序列]: https://msdn.microsoft.com/zh-cn/library/ff878091.aspx
[触发器]: https://msdn.microsoft.com/zh-cn/library/ms189799.aspx
[索引视图]: https://msdn.microsoft.com/zh-cn/library/ms191432.aspx
[同义词]: https://msdn.microsoft.com/zh-cn/library/ms177544.aspx
[唯一索引]: https://msdn.microsoft.com/zh-cn/library/ms188783.aspx

<!--Other Web references-->

<!---HONumber=Mooncake_0829_2016-->