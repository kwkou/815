<properties
   pageTitle="在 Azure SQL 数据仓库中管理表大小 | Azure"
   description="了解表和分区的大小。" 
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="jrowlandjones"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="04/13/2016"
   wacn.date="06/20/2016"/>

# 在 Azure SQL 数据仓库中管理表大小

本文标识了表分区的大小，并有助于你了解数据库的大小。


## 步骤 1：创建一个汇总了表大小的视图

创建此视图来识别哪些表有数据偏斜。

    CREATE VIEW dbo.vPartitionSizing
    AS
    
    WITH base
    AS
    (
    SELECT 
    	SUBSTRING(@@version,34,4)																AS build_number
    ,	DB_NAME()																				AS database_name
    ,	s.name																					AS schema_name
    ,	t.name																					AS table_name
    ,	QUOTENAME(s.name)+'.'+QUOTENAME(t.name)													AS two_part_name
    ,	nt.name																					AS node_table_name
    ,   i.[type]                                                                                AS table_type
    ,   i.[type_desc]                                                                           AS table_type_desc
    ,	tp.distribution_policy_desc	                        									AS distribution_policy_name
    ,	pn.pdw_node_id                                                                          AS pdw_node_id
    ,	pn.[type]																				AS pdw_node_type
    ,	pn.name																					AS pdw_node_name
    ,   di.distribution_id                                                                      AS dist_id
    ,   di.name                                                                                 AS dist_name
    ,   di.position                                                                             AS dist_position
    ,	nps.partition_number																	AS partition_nmbr
    ,	nps.reserved_page_count																	AS reserved_space_page_count
    ,	nps.reserved_page_count - nps.used_page_count											AS unused_space_page_count
    ,	nps.in_row_data_page_count + nps.row_overflow_used_page_count + nps.lob_used_page_count	AS  data_space_page_count
    ,	nps.reserved_page_count 
    	- (nps.reserved_page_count - nps.used_page_count) 
    	- (in_row_data_page_count+row_overflow_used_page_count+lob_used_page_count)				AS index_space_page_count
    ,	nps.[row_count]                                                                         AS row_count
    from sys.schemas s
    join sys.tables t								ON	s.schema_id			= t.schema_id
    JOIN sys.indexes i                              ON  t.object_id         = i.object_id
                                                    AND i.index_id          <= 1
    join sys.pdw_table_distribution_properties	tp	ON	t.object_id			= tp.object_id
    join sys.pdw_table_mappings tm					ON	t.object_id			= tm.object_id
    join sys.pdw_nodes_tables nt					ON	tm.physical_name	= nt.name
    join sys.dm_pdw_nodes pn 						ON  nt.pdw_node_id		= pn.pdw_node_id
    join sys.pdw_distributions di                   ON  nt.distribution_id  = di.distribution_id
    join sys.dm_pdw_nodes_db_partition_stats nps	ON	nt.object_id		= nps.object_id
    												AND nt.pdw_node_id		= nps.pdw_node_id
                                                    AND nt.distribution_id  = nps.distribution_id
    )
    , size
    AS
    (
    SELECT	build_number
    ,		database_name
    ,		schema_name
    ,		table_name
    ,		two_part_name
    ,		node_table_name
    ,       table_type
    ,       table_type_desc
    ,		distribution_policy_name
    ,       dist_id
    ,       dist_name
    ,       dist_position
    ,		pdw_node_id
    ,		pdw_node_type
    ,		pdw_node_name
    ,		partition_nmbr
    ,		reserved_space_page_count
    ,		unused_space_page_count
    ,		data_space_page_count
    ,		index_space_page_count
    ,		row_count
    ,		(reserved_space_page_count * 8.0)				AS reserved_space_KB
    ,		(reserved_space_page_count * 8.0)/1000			AS reserved_space_MB
    ,		(reserved_space_page_count * 8.0)/1000000		AS reserved_space_GB
    ,		(reserved_space_page_count * 8.0)/1000000000	AS reserved_space_TB
    ,		(unused_space_page_count * 8.0)					AS unused_space_KB
    ,		(unused_space_page_count * 8.0)/1000			AS unused_space_MB
    ,		(unused_space_page_count * 8.0)/1000000			AS unused_space_GB
    ,		(unused_space_page_count * 8.0)/1000000000		AS unused_space_TB
    ,		(data_space_page_count * 8.0)					AS data_space_KB
    ,		(data_space_page_count * 8.0)/1000				AS data_space_MB
    ,		(data_space_page_count * 8.0)/1000000			AS data_space_GB
    ,		(data_space_page_count * 8.0)/1000000000		AS data_space_TB
    ,		(index_space_page_count * 8.0)	    			AS index_space_KB
    ,		(index_space_page_count * 8.0)/1000				AS index_space_MB
    ,		(index_space_page_count * 8.0)/1000000			AS index_space_GB
    ,		(index_space_page_count * 8.0)/1000000000		AS index_space_TB
    FROM	base
    )
    SELECT	* 
    FROM	size
    ;

## 步骤 2：查询视图

现在，你已经创建了该视图，因此可以运行以下示例性查询来详细了解表的大小：

### 数据库中使用的表空间的摘要

    SELECT
    	database_name
    ,	SUM(row_count)				as total_row_count
    ,	SUM(reserved_space_MB)		as total_reserved_space_MB
    ,	SUM(data_space_MB)			as total_data_space_MB
    ,	SUM(index_space_MB)			as total_index_space_MB
    ,	SUM(unused_space_MB)		as total_unused_space_MB
    FROM dbo.vPartitionSizing
    GROUP BY database_name
    ;

### 表所使用的表空间

    SELECT 
    	two_part_name
    ,	SUM(row_count)				as table_row_count
    ,	SUM(reserved_space_GB)		as table_reserved_space_GB
    ,	SUM(data_space_GB)			as table_data_space_GB
    ,	SUM(index_space_GB)			as table_index_space_GB
    ,	SUM(unused_space_GB)		as table_unused_space_GB
    FROM dbo.vPartitionSizing
    GROUP BY two_part_name
    ;

### 按表的 geometry 类型分类的表空间

    SELECT 
       table_type_desc
    ,	SUM(row_count)				as table_type_row_count
    ,	SUM(reserved_space_GB)		as table_type_reserved_space_GB
    ,	SUM(data_space_GB)			as table_type_data_space_GB
    ,	SUM(index_space_GB)			as table_type_index_space_GB
    ,	SUM(unused_space_GB)		as table_type_unused_space_GB
    FROM dbo.vPartitionSizing
    GROUP BY table_type_desc
    ;

### 按分布分类的表空间

    SELECT 
    	pdw_node_id
    ,	distribution_policy_name
    ,	dist_name
    ,	SUM(row_count)				as total_node_distribution_row_count
    ,	SUM(reserved_space_MB)		as total_node_distribution_reserved_space_MB
    ,	SUM(data_space_MB)			as total_node_distribution_data_space_MB
    ,	SUM(index_space_MB)			as total_node_distribution_index_space_MB
    ,	SUM(unused_space_MB)		as total_node_distribution_unused_space_MB
    FROM dbo.vPartitionSizing
    GROUP BY 	distribution_policy_name
    ,			dist_name
    ,			pdw_node_id
    ORDER BY    pdw_node_id
    ,           distribution_policy_name
    ,		    dist_name
    ;

## 步骤 3：管理表

你是否发现了不再需要的表？ 考虑删除这些表，或使用 [CREATE EXTERNAL TABLE AS SELECT (CETAS)][] 将这些表导出到 blob 存储

某些表是否比你想象的要大？ 你可能需要对这些表进行分区。有关详细信息，请参阅有关 [表分区][] 的文章。

你是否接近所配置的数据库最大大小限制？ 使用 [ALTER DATABASE][] 对其进行更改。

## 后续步骤
有关表数据分布的更多详细信息，请参阅以下文章：

* [表设计][]
* [哈希分布][]

<!--Image references-->

<!--Article references-->
[表设计]: /documentation/articles/sql-data-warehouse-develop-table-design/
[哈希分布]: /documentation/articles/sql-data-warehouse-develop-hash-distribution-key/

<!--MSDN references-->

<!--Other Web references-->
<!---HONumber=Mooncake_0613_2016-->