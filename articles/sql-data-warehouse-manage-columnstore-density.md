<properties
   pageTitle="管理表数据分布倾斜 | Azure"
   description="一个指南，帮助用户识别其分布式表的数据分布倾斜"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="jrowlandjones"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="03/18/2016"
   wacn.date="04/18/2016"/>

# 管理列存储索引
列存储索引是 Azure SQL 数据仓库的核心。保持这些索引的最佳状态可以最大化系统性能。

本文介绍了如何查询表的列存储索引元数据，以帮助你诊断问题并快速解决问题。

## 查询列存储元数据
要了解列存储索引的密度，必须对系统元数据进行查询。下面是你可以发现的信息种类的一个示例。

```
CREATE VIEW dbo.vColumnstoreDensity
AS
WITH CSI
AS
(
SELECT
        SUBSTRING(@@version,34,4)                                               AS [build_number]
,       GETDATE()                                                               AS [execution_date]
,       DB_Name()                                                               AS [database_name]
,       t.name                                                                  AS [table_name]
,		COUNT(DISTINCT rg.[partition_number])									AS [table_partition_count]
,       SUM(rg.[total_rows])                                                    AS [row_count_total]
,       SUM(rg.[total_rows])/COUNT(DISTINCT rg.[distribution_id])               AS [row_count_per_distribution_MAX]
,		CEILING	(	(SUM(rg.[total_rows])*1.0/COUNT(DISTINCT rg.[distribution_id])
						)/1048576
				)																AS [rowgroup_per_distribution_MAX]
,       SUM(CASE WHEN rg.[State] = 1 THEN 1                   ELSE 0    END)    AS [OPEN_rowgroup_count]
,       SUM(CASE WHEN rg.[State] = 1 THEN rg.[total_rows]     ELSE 0    END)    AS [OPEN_rowgroup_rows]
,       MIN(CASE WHEN rg.[State] = 1 THEN rg.[total_rows]     ELSE NULL END)    AS [OPEN_rowgroup_rows_MIN]
,       MAX(CASE WHEN rg.[State] = 1 THEN rg.[total_rows]     ELSE NULL END)    AS [OPEN_rowgroup_rows_MAX]
,       AVG(CASE WHEN rg.[State] = 1 THEN rg.[total_rows]     ELSE NULL END)    AS [OPEN_rowgroup_rows_AVG]
,       SUM(CASE WHEN rg.[State] = 2 THEN 1                   ELSE 0    END)    AS [CLOSED_rowgroup_count]
,       SUM(CASE WHEN rg.[State] = 2 THEN rg.[total_rows]     ELSE 0    END)    AS [CLOSED_rowgroup_rows]
,       MIN(CASE WHEN rg.[State] = 2 THEN rg.[total_rows]     ELSE NULL END)    AS [CLOSED_rowgroup_rows_MIN]
,       MAX(CASE WHEN rg.[State] = 2 THEN rg.[total_rows]     ELSE NULL END)    AS [CLOSED_rowgroup_rows_MAX]
,       AVG(CASE WHEN rg.[State] = 2 THEN rg.[total_rows]     ELSE NULL END)    AS [CLOSED_rowgroup_rows_AVG]
,       SUM(CASE WHEN rg.[State] = 3 THEN 1                   ELSE 0    END)    AS [COMPRESSED_rowgroup_count]
,       SUM(CASE WHEN rg.[State] = 3 THEN rg.[total_rows]     ELSE 0    END)    AS [COMPRESSED_rowgroup_rows]
,       SUM(CASE WHEN rg.[State] = 3 THEN rg.[deleted_rows]   ELSE 0    END)    AS [COMPRESSED_rowgroup_rows_DELETED]
,       MIN(CASE WHEN rg.[State] = 3 THEN rg.[total_rows]     ELSE NULL END)    AS [COMPRESSED_rowgroup_rows_MIN]
,       MAX(CASE WHEN rg.[State] = 3 THEN rg.[total_rows]     ELSE NULL END)    AS [COMPRESSED_rowgroup_rows_MAX]
,       AVG(CASE WHEN rg.[State] = 3 THEN rg.[total_rows]     ELSE NULL END)    AS [COMPRESSED_rowgroup_rows_AVG]
FROM    sys.[pdw_nodes_column_store_row_groups] rg
JOIN    sys.[pdw_nodes_tables] nt                   ON  rg.[object_id]          = nt.[object_id]
                                                    AND rg.[pdw_node_id]        = nt.[pdw_node_id]
                                                    AND rg.[distribution_id]    = nt.[distribution_id]
JOIN    sys.[pdw_table_mappings] mp                 ON  nt.[name]               = mp.[physical_name]
JOIN    sys.[tables] t                              ON  mp.[object_id]          = t.[object_id]
GROUP BY
        t.[name]
)
SELECT  *
FROM    CSI
;
```

一旦创建了视图，就可以轻松地分析列存储元数据。下面是一个查询示例。

```
SELECT	[table_name]
,		[table_partition_count]
,		[row_count_total]
,		[row_count_per_distribution_MAX]
,		[COMPRESSED_rowgroup_rows]
,		[COMPRESSED_rowgroup_rows_AVG]
,		[COMPRESSED_rowgroup_rows_DELETED]
,		[COMPRESSED_rowgroup_count]
,		[OPEN_rowgroup_count]
,		[OPEN_rowgroup_rows]
,		[CLOSED_rowgroup_count]
,		[CLOSED_rowgroup_rows]
FROM	[dbo].[vColumnstoreDensity]
WHERE	[table_name] = 'FactInternetSales'
```

## 提高列存储密度
运行查询后就可以开始查看数据并分析结果。

可查看以下数据：

| 列 | 如何使用此数据 |
| ---------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [table\_partition\_count] | 如果表被分区，那么你可能希望看到更高的打开的行组的计数。在理论上分布式架构中每个分区具有一个与之相关联的打开的行组。请在分析中考虑这一点。可以通过完全删除分区来优化已分区的小型表，因为这样可以改进压缩。 |
| [row\_count\_total] | 表的总的行计数，例如，此值可用于计算压缩状态的行的百分比 |
| [row\_count\_per\_distribution\_MAX] | 如果所有行均匀分布，那么此值为每个分布式架构的目标行数。将此值与 compressed\_rowgroup\_count 进行比较。 |
| [COMPRESSED\_rowgroup\_rows] | 表的列存储格式的行的总数 |
| [COMPRESSED\_rowgroup\_rows\_AVG] | 如果平均行数明显小于行组的最大行数，则可以考虑使用 CTAS 或 ALTER INDEX REBUILD 重新压缩数据 |
| [COMPRESSED\_rowgroup\_count] | 列存储格式的行组数。如果该值相对于表的大小显得非常高，那么它表示列存储密度低。 |
| [COMPRESSED\_rowgroup\_rows\_DELETED] | 逻辑上被删除的列存储格式的行数。如果该数值相对于表的大小来说是高的，请考虑重新创建分区或重新生成索引，因为这将以物理方式删除它们。 |
| [COMPRESSED\_rowgroup\_rows\_MIN] | 将该值与 AVG 和 MAX 列一起使用，以便了解列存储格式的行组的值的范围。如果该值稍微高出加载阈值（每分区对齐的分布区 102,400 行），则提示可在数据加载中进行优化 |
| [COMPRESSED\_rowgroup\_rows\_MAX] | 同上 |
| [OPEN\_rowgroup\_count] | 打开的行组都是正常的。理论上，每个表分布区 (60) 预期有一个 OPEN 状态的行组。如果该值过大，则提示需要跨分区进行数据加载。仔细检查分区策略，以确保该值合理。 |
| [OPEN\_rowgroup\_rows] | 每个行组最多可以具有 1,048,576 行。使用此值可以了解当前打开的行组的填满程度。 |
| [OPEN\_rowgroup\_rows\_MIN] | 打开的组表示数据正在以点滴方式加载到表，或者之前的加载超出了剩余行，溢出到此行组。使用 MIN、MAX、AVG 列来查看 OPEN 行组中有多少数据。对于小型表，此数据量可以为所有数据的 100%！ 在此情况下使用 ALTER INDEX REBUILD 语句将数据强制转换为列存储格式。 |
| [OPEN\_rowgroup\_rows\_MAX] | 同上 |
| [OPEN\_rowgroup\_rows\_AVG] | 同上 |
| [CLOSED\_rowgroup\_rows] | 查看已关闭行组的行以进行健全性检查。若有 |
| [CLOSED\_rowgroup\_count] | 如果没有看到几个行组，则已关闭的行组数应该较低。可以将已关闭的行组转换为压缩行组，方法是使用 ALTER INDEX ...REORGANISE 命令。但是，通常并不需要使用此命令。后台的“tuple mover”进程会自动将关闭的组转换为列存储行组。 |
| [CLOSED\_rowgroup\_rows\_MIN] | 已关闭的行组应具有非常高的填充率。如果已关闭的行组的填充率较低，则需要进一步分析列存储。 |
| [CLOSED\_rowgroup\_rows\_MAX] | 同上 |
| [CLOSED\_rowgroup\_rows\_AVG] | 同上 |

## 索引管理
可以通过使用 [CTAS][] 创建分区或使用 [ALTER INDEX][] 重新生成索引本身来实现行组的重新压缩。通常，[CTAS][] 的执行速度快于 [ALTER INDEX][]，尤其是对大型的表或分区。但是，对于较小的表或分区，通常 [ALTER INDEX][] 更为方便。

>[AZURE.NOTE] ALTER INDEX...REBUILD 是一种脱机操作。ALTER INDEX...REORGANISE 是一种联机操作。但是 REORGANISE 不触及打开的行组。若要包括打开的行组，则需要使用 ALTER INDEX...REBUILD。

重新生成索引，尤其是对于大型表，通常会从其他资源中受益。Azure SQL 数据仓库具有一种工作负荷管理功能，此功能可用于向用户分配更多内存。若要了解如何为你的索引重新生成保留更多内存，请参阅[并发][]文章的工作负荷管理部分。

## 后续步骤
有关使用 `CTAS` 重新创建分区的详细信息，请参阅以下文章：

* [表分区][]
* [并发][]

有关索引管理的更具体的建议，请查看[管理索引][]文章。

有关更多管理提示，请转到[管理][]概述

<!--Image references-->

<!--Article references-->
[CTAS]: /documentation/articles/sql-data-warehouse-develop-ctas/
[表分区]: /documentation/articles/sql-data-warehouse-develop-table-partitions/
[并发]: /documentation/articles/sql-data-warehouse-develop-concurrency/
[管理]: /documentation/articles/sql-data-warehouse-manage-monitor/
[管理索引]: /documentation/articles/sql-data-warehouse-manage-indexes/

<!--MSDN references-->
[ALTER INDEX]: https://msdn.microsoft.com/zh-cn/library/ms188388.aspx


<!--Other Web references-->
<!---HONumber=Mooncake_0411_2016-->