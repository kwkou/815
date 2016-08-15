<properties
   pageTitle="在 Azure SQL 数据仓库中管理分布式表的数据偏斜 | Azure"
   description="当行在 Azure SQL 数据仓库中的哈希分布式表的所有分布区中不均衡分布时，查找并修复数据偏斜。" 
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="jrowlandjones"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="04/14/2016"
   wacn.date="06/13/2016"/>

# 在 Azure SQL 数据仓库中管理分布式表的数据偏斜
本教程介绍如何识别哈希分布式表中的数据偏斜，并提供解决问题的建议。

使用哈希分布方法分布表数据时，某些分布可能会偏斜，比其他表具有更多数据。过度的数据偏斜可能会影响查询性能，因为在运行时间最长的分布完成之前，分布式查询的最终结果无法准备就绪。根据数据偏斜的程度，你可能需要解决这种问题。

在本教程中你将：

- 使用元数据来确定哪些表有数据偏斜
- 了解知道何时可解决数据偏斜的提示
- 重新创建表以解决数据偏斜

## DBCC PDW\_SHOWSPACEUSED

识别数据偏斜的方法之一是使用 [DBCC PDW\_SHOWSPACEUSED()][]

```sql
-- Find data skew for a distributed table
DBCC PDW_SHOWSPACEUSED('dbo.FactInternetSales');
```

这是一种非常快捷简便的方法，可以查看存储在数据库中每组 60 个分布区内的表行数目。请记住，为了获得最平衡的性能，分布式表中的行应该平均分散在所有分布区中。

但是，如果你查询 Azure SQL 数据仓库动态管理视图 (DMV)，则可以执行更详细的分析。本文的余下部分将说明如何执行此操作。

## 步骤 1︰创建一个视图用于查找数据偏斜

创建此视图来识别哪些表有数据偏斜。

```sql
CREATE VIEW dbo.vDistributionSkew
AS
WITH base
AS
(
SELECT 
	SUBSTRING(@@version,34,4)															AS  [build_number]
,	GETDATE()																			AS  [execution_time]
,	DB_NAME()																			AS  [database_name]
,	s.name																				AS  [schema_name]
,	t.name																				AS  [table_name]
,	QUOTENAME(s.name)+'.'+QUOTENAME(t.name)												AS  [two_part_name]
,	nt.[name]																			AS  [node_table_name]
,	ROW_NUMBER() OVER(PARTITION BY nt.[name] ORDER BY (SELECT NULL))					AS  [node_table_name_seq]
,	tp.[distribution_policy_desc]														AS  [distribution_policy_name]
,	nt.[distribution_id]																AS  [distribution_id]
,	nt.[pdw_node_id]																	AS  [pdw_node_id]
,	pn.[type]																			AS	[pdw_node_type]
,	pn.[name]																			AS	[pdw_node_name]
,	nps.[partition_number]																AS	[partition_nmbr]
,	nps.[reserved_page_count]															AS	[reserved_space_page_count]
,	nps.[reserved_page_count] - nps.[used_page_count]									AS	[unused_space_page_count]
,	nps.[in_row_data_page_count] 
	+ nps.[row_overflow_used_page_count] + nps.[lob_used_page_count]					AS  [data_space_page_count]
,	nps.[reserved_page_count] 
	- (nps.[reserved_page_count] - nps.[used_page_count]) 
	- ([in_row_data_page_count]+[row_overflow_used_page_count]+[lob_used_page_count])	AS  [index_space_page_count]
,	nps.[row_count]																		AS  [row_count]
from sys.schemas s
join sys.tables t								ON	s.[schema_id]			= t.[schema_id]
join sys.pdw_table_distribution_properties	tp	ON	t.[object_id]			= tp.[object_id]
join sys.pdw_table_mappings tm					ON	t.[object_id]			= tm.[object_id]
join sys.pdw_nodes_tables nt					ON	tm.[physical_name]		= nt.[name]
join sys.dm_pdw_nodes pn 						ON  nt.[pdw_node_id]		= pn.[pdw_node_id]
join sys.dm_pdw_nodes_db_partition_stats nps	ON	nt.[object_id]			= nps.[object_id]
												AND nt.[pdw_node_id]		= nps.[pdw_node_id]
												AND nt.[distribution_id]	= nps.[distribution_id]
)
, size
AS
(
SELECT	[build_number]
,		[execution_time]
,		[database_name]
,		[schema_name]
,		[table_name]
,		[two_part_name]
,		[node_table_name]
,		[node_table_name_seq]
,		[distribution_policy_name]
,		[distribution_id]
,		[pdw_node_id]
,		[pdw_node_type]
,		[pdw_node_name]
,		[partition_nmbr]
,		[reserved_space_page_count]
,		[unused_space_page_count]
,		[data_space_page_count]
,		[index_space_page_count]
,		[row_count]
,		([reserved_space_page_count] * 8.0)				AS [reserved_space_KB]
,		([reserved_space_page_count] * 8.0)/1000		AS [reserved_space_MB]
,		([reserved_space_page_count] * 8.0)/1000000		AS [reserved_space_GB]
,		([reserved_space_page_count] * 8.0)/1000000000	AS [reserved_space_TB]
,		([unused_space_page_count]   * 8.0)				AS [unused_space_KB]
,		([unused_space_page_count]   * 8.0)/1000		AS [unused_space_MB]
,		([unused_space_page_count]   * 8.0)/1000000		AS [unused_space_GB]
,		([unused_space_page_count]   * 8.0)/1000000000	AS [unused_space_TB]
,		([data_space_page_count]     * 8.0)				AS [data_space_KB]
,		([data_space_page_count]     * 8.0)/1000		AS [data_space_MB]
,		([data_space_page_count]     * 8.0)/1000000		AS [data_space_GB]
,		([data_space_page_count]     * 8.0)/1000000000	AS [data_space_TB]
,		([index_space_page_count]	 * 8.0)				AS [index_space_KB]
,		([index_space_page_count]	 * 8.0)/1000		AS [index_space_MB]
,		([index_space_page_count]	 * 8.0)/1000000		AS [index_space_GB]
,		([index_space_page_count]	 * 8.0)/1000000000	AS [index_space_TB]
FROM	base
)
SELECT	* 
FROM	size
;
```

## 步骤 2：查询视图

现在你已创建视图，请运行此示例查询来识别哪些表有数据偏斜。

```sql
SELECT	[two_part_name]
,		[distribution_id]
,		[row_count]
,		[reserved_space_GB]
,		[unused_space_GB]
,		[data_space_GB]
,		[index_space_GB]
FROM	[dbo].[vDistributionSkew]
WHERE	[table_name] = 'FactInternetSales'
ORDER BY [row_count] DESC
```

>[AZURE.NOTE] ROUND\_ROBIN 分布式表不应倾斜。数据按照设计均匀分布在各个节点上。

## 步骤 3：确定你是否应解决数据偏斜

若要确定是否应该解决表中的数据偏斜，你应该尽可能了解工作负荷中的数据卷和查询。

分布式数据是找出将数据偏斜降至最低与将数据移动降至最低两者之间的适当平衡。这些可能是相反的目标，有时你会想要保留数据偏斜，以减少数据移动。例如，当分布列经常是联接和聚合中的共享列时，你便会将数据移动降至最低。最少的数据移动带来的好处可能胜过数据偏斜造成的影响。

## 步骤 4：解决数据偏斜

解决数据偏斜有两个可行的方法。如果你已决定要解决偏斜，请使用其中一种。

### 方法 1︰重新创建具有不同分布列的表

解决数据偏斜的典型方法是重新创建具有不同分布列的表。如需选择哈希分布列的指引，请参阅 [Hash distribution][]（哈希分布）。本示例使用 [CTAS][] 来重新创建具有不同分布列的表。

```sql
CREATE TABLE [dbo].[FactInternetSales_CustomerKey] 
WITH (  CLUSTERED COLUMNSTORE INDEX
     ,  DISTRIBUTION =  HASH([CustomerKey])
     ,  PARTITION       ( [OrderDateKey] RANGE RIGHT FOR VALUES (   20000101, 20010101, 20020101, 20030101
                                                                ,   20040101, 20050101, 20060101, 20070101
                                                                ,   20080101, 20090101, 20100101, 20110101
                                                                ,   20120101, 20130101, 20140101, 20150101
                                                                ,   20160101, 20170101, 20180101, 20190101
                                                                ,   20200101, 20210101, 20220101, 20230101
                                                                ,   20240101, 20250101, 20260101, 20270101
                                                                ,   20280101, 20290101
                                                                )
                        )
    )
AS
SELECT  *
FROM    [dbo].[FactInternetSales]
OPTION  (LABEL  = 'CTAS : FactInternetSales_CustomerKey')
;

--Rename the objects
RENAME OBJECT [dbo].[FactInternetSales] TO [FactInternetSales_ProductKey];
RENAME OBJECT [dbo].[FactInternetSales_CustomerKey] TO [FactInternetSales];
```

### 方法 2：使用轮循机制分布重新创建表

本示例使用轮循机制而不是哈希分布来重新创建表。这项更改将会生成平均的数据分布。不过，通常会增大查询的数据移动。

```sql
CREATE TABLE [dbo].[FactInternetSales_ROUND_ROBIN] 
WITH (  CLUSTERED COLUMNSTORE INDEX
     ,  DISTRIBUTION =  ROUND_ROBIN
     ,  PARTITION       ( [OrderDateKey] RANGE RIGHT FOR VALUES (   20000101, 20010101, 20020101, 20030101
                                                                ,   20040101, 20050101, 20060101, 20070101
                                                                ,   20080101, 20090101, 20100101, 20110101
                                                                ,   20120101, 20130101, 20140101, 20150101
                                                                ,   20160101, 20170101, 20180101, 20190101
                                                                ,   20200101, 20210101, 20220101, 20230101
                                                                ,   20240101, 20250101, 20260101, 20270101
                                                                ,   20280101, 20290101
                                                                )
                        )
    )
AS
SELECT  *
FROM    [dbo].[FactInternetSales]
OPTION  (LABEL  = 'CTAS : FactInternetSales_ROUND_ROBIN')
;

--Rename the objects
RENAME OBJECT [dbo].[FactInternetSales] TO [FactInternetSales_HASH];
RENAME OBJECT [dbo].[FactInternetSales_ROUND_ROBIN] TO [FactInternetSales];
```

## 后续步骤
有关表数据分布的更多详细信息，请参阅以下文章：

* [表设计][]
* [哈希分布][]

<!--Image references-->

<!--Article references-->
[表设计]: /documentation/articles/sql-data-warehouse-develop-table-design/
[Hash distribution]: /documentation/articles/sql-data-warehouse-develop-hash-distribution-key/
[哈希分布]: /documentation/articles/sql-data-warehouse-develop-hash-distribution-key/

<!--MSDN references-->
[DBCC PDW\_SHOWSPACEUSED()]: https://msdn.microsoft.com/zh-cn/library/mt204028.aspx

<!--Other Web references-->
<!---HONumber=Mooncake_0606_2016-->