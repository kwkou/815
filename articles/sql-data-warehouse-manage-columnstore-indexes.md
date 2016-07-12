<properties
   pageTitle="管理 Azure SQL 数据仓库中的列存储索引 | Azure"
   description="有关在 Azure SQL 数据仓库中管理列存储索引以提高压缩和查询性能的教程。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="jrowlandjones"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="05/02/2016"
   wacn.date="06/13/2016"/>

# 管理 Azure SQL 数据仓库中的列存储索引

本教程说明如何管理列存储索引以改进查询性能。

当索引将行一起压缩为一百万行（确切数字为 1,048,576）的“行组”时，对列存储索引的查询可达到最佳运行性能。这可以提供最佳的压缩和最佳的查询性能。但是，在某些情况下，行组中的行数远远少于一百万个。如果行组中未密集填充行，你应该考虑进行调整。

在本教程中，你将了解如何：

- 使用元数据来确定每个行组的行平均数目
- 考虑未密集填充行组的根本原因
- 重新生成列存储索引，以将所有行重新压缩到新的行组中 

若要了解有关列存储索引的基本知识，请参阅 [Columnstore Guide（列存储指南）](https://msdn.microsoft.com/zh-cn/library/gg492088.aspx)。

## 步骤 1：创建显示行组元数据的视图

此视图将计算每个行组的平均行数。此外，将显示有关行组的其他信息。

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

## 步骤 2：查询视图

创建视图后，请运行此示例查询，以查看列存储索引的行组元数据。

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

## 步骤 3：分析结果

运行查询后就可以开始查看数据并分析结果。下表解释了要在行组分析中查看的内容。


| 列 | 如何使用此数据 |
| ---------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [table\_partition\_count] | 如果表被分区，那么你可能希望看到更高的打开的行组的计数。在理论上分布式架构中每个分区具有一个与之相关联的打开的行组。请在分析中考虑这一点。可以通过完全删除分区来优化已分区的小型表，因为这样可以改进压缩。 |
| [row\_count\_total] | 表的行总数。例如，可以使用此值来计算处于压缩状态的行的百分比。 |
| [row\_count\_per\_distribution\_MAX] | 如果所有行均匀分布，那么此值为每个分布式架构的目标行数。将此值与 compressed\_rowgroup\_count 进行比较。 |
| [COMPRESSED\_rowgroup\_rows] | 表的列存储格式的行的总数。 |
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
| [CLOSED\_rowgroup\_rows] | 查看已关闭行组的行以进行健全性检查。 |
| [CLOSED\_rowgroup\_count] | 如果没有看到几个行组，则已关闭的行组数应该较低。可以将已关闭的行组转换为压缩行组，方法是使用 ALTER INDEX ...REORGANISE 命令。但是，通常并不需要使用此命令。后台的“tuple mover”进程会自动将关闭的组转换为列存储行组。 |
| [CLOSED\_rowgroup\_rows\_MIN] | 已关闭的行组应具有非常高的填充率。如果已关闭的行组的填充率较低，则需要进一步分析列存储。 |
| [CLOSED\_rowgroup\_rows\_MAX] | 同上 |
| [CLOSED\_rowgroup\_rows\_AVG] | 同上 |


## 步骤 4：调查根本原因

调查根本原因可帮助确定是否可以在表设计、加载或其他流程方面的更改，以改善行组中的行密度，从而改善压缩和查询性能。

这些因素可能导致列存储索引在每个行组中的行远远少于 1,048,576 个。它们还会造成行转到增量行组而不是压缩的行组。

1. 繁重的 DML 操作
2. 小型或渗透负载操作
3. 过多的分区

### 繁重的 DML 操作** 

更新和删除行的繁重的 DML 操作造成列存储低效。当行组中的大多数行已修改时尤其如此。

- 从压缩的行组中删除某行只会以逻辑方式将该行标记为已删除。该行仍会保留在压缩的行组中，直到重新生成分区或表为止。
- 插入某行会将该行添加到名为增量行组的内部行存储表中。在增量行组已满且标记为已关闭之前，插入的行不会转换成列存储。达到 1,048,576 个行的容量上限后，行组将会关闭。 
- 更新采用列存储格式的行将依次作为逻辑删除和插入进行处理。插入的行存储在增量存储中。

超出每个分区对齐分布区的 102,400 行批量阈值的批次更新和插入操作将直接写入列存储格式。但是，假设在平均分布的情况下，需要在单个操作中修改超过 6.144 百万行才发生这种情况。如果给定分区对齐分布区的行数少于 102,400 个，行将转到增量存储，并在插入足够的行、修改行以关闭行组或已创建索引之前，保留在增量存储中。

### 小型或渗透负载操作

流入 SQL 数据仓库的小型负载有时也称为渗透负载。它们通常代表系统引入的数据的接近恒定流。但是，由于此流接近连续状态，因此行的容量并不特别大。通常数据远低于直接加载到列存储格式所需的阈值。

在这些情况下，最好先将数据保存到 Azure Blob 存储中，并让它在加载之前累积。此技术通常称为微批处理。

### 过多的分区

你可能有太多的分区。在大规模并行处理 (MPP) 体系结构中，用户定义的表是以 60 个表的形式分布和实现的。因此，将每个列存储索引实现为 60 个列存储索引。同样，每个分区跨这 60 个列存储索引实现。这与采用对称多重处理 (SMP) 体系结构的 SQL Server 有**很大的差别**。

如果将 1,048,576 个行载入 SMP SQL Server 表的一个分区中，该分区将压缩到列存储中。但是，如果将 1,048,576 个行载入 SQL 数据仓库表的一个分区，则 60 个分布区的每个分布区仅有 17,467 个行（假设数据平均分布）。由于 17,467 少于每个分布区 102,400 个行的阈值，SQL 数据仓库将数据保留在增量存储行组中。因此，若要直接将行载入压缩的列存储格式，必须在 SQL 数据仓库表的单个分区中加载超过 6.1444 百万行 (60 x 102,400)。

## 步骤 5：分配额外的计算资源

重新生成索引，尤其是对于大型表，通常需要额外的资源。SQL 数据仓库提供可以向用户分配更多内存的工作负荷管理功能。若要了解如何为你的索引重新生成保留更多内存，请参阅 [concurrency][]（并发）文章的工作负荷管理部分。

## 步骤 6：重新生成索引以改善每个行组的平均行数

若要将现有压缩行组与强制增量行组合并到列存储，需要重新生成索引。通常，由于存在太多的数据，因而无法重新生成整个索引，在这种情况下，最好重新生成一个或多个分区。在 SQL 数据仓库中，可以使用以下两种方法之一重新生成索引：

1. 使用 [ALTER INDEX][] 重新生成分区。
2. 使用 [CTAS][] 在新表中重新创建分区，然后使用分区切换将分区移回到原始表。

哪种方法最合适？ 如果数据量很大，[CTAS][] 的速度通常比 [ALTER INDEX][] 要快。如果数据量较小，[ALTER INDEX][] 更易于使用。

### 方法 #1：使用 ALTER INDEX 脱机重新生成少量数据

这些示例演示如何重新生成整个列存储索引和重新生成单个分区。对于大型表，适合重新生成单个分区。这是一种脱机操作。

```
-- Rebuild the entire clustered index
ALTER INDEX ALL ON [dbo].[DimProduct] REBUILD
```

```
-- Rebuild a single partition
ALTER INDEX ALL ON [dbo].[FactInternetSales] REBUILD Partition = 5
```

有关详细信息，请参阅 [Columnstore Indexes Defragmentation（列存储索引碎片整理）](https://msdn.microsoft.com/zh-cn/library/dn935013.aspx#Anchor_1)中的 ALTER INDEX REBUILD 部分和语法主题 [ALTER INDEX (Transact-SQL)](https://msdn.microsoft.com/zh-cn/library/ms188388.aspx)。

### 方法 #2：使用 CTAS 和分区切换联机重新生成大量数据

此示例使用 [CTAS][] 和分区切换重新生成表分区。


```
-- Step 01. Select the partition of data and write it out to a new table using CTAS
CREATE TABLE [dbo].[FactInternetSales_20000101_20010101]
    WITH    (   DISTRIBUTION = HASH([ProductKey])
            ,   CLUSTERED COLUMNSTORE INDEX
            ,   PARTITION   (   [OrderDateKey] RANGE RIGHT FOR VALUES
                                (20000101,20010101
                                )
                            )
            )
AS
SELECT  *
FROM    [dbo].[FactInternetSales]
WHERE   [OrderDateKey] >= 20000101
AND     [OrderDateKey] <  20010101
;

-- Step 02. Create a SWITCH out table
 
CREATE TABLE dbo.FactInternetSales_20000101
    WITH    (   DISTRIBUTION = HASH(ProductKey)
            ,   CLUSTERED COLUMNSTORE INDEX
            ,   PARTITION   (   [OrderDateKey] RANGE RIGHT FOR VALUES
                                (20000101
                                )
                            )
            )
AS
SELECT *
FROM    [dbo].[FactInternetSales]
WHERE   1=2 -- Note this table will be empty

-- Step 03. Switch OUT the data 
ALTER TABLE [dbo].[FactInternetSales] SWITCH PARTITION 2 TO  [dbo].[FactInternetSales_20000101] PARTITION 2;

-- Step 04. Switch IN the rebuilt data
ALTER TABLE [dbo].[FactInternetSales_20000101_20010101] SWITCH PARTITION 2 TO  [dbo].[FactInternetSales] PARTITION 2;
```

有关使用 `CTAS` 重新创建分区的更多详细信息，请参阅 [Table partitioning][]（表分区）和 [concurrency][]（并发）。


## 后续步骤

有关更多管理提示，请转到[管理][]概述。

<!--Image references-->

<!--Article references-->
[CTAS]: /documentation/articles/sql-data-warehouse-develop-ctas/
[Table partitioning]: /documentation/articles/sql-data-warehouse-develop-table-partitions/
[Concurrency]: /documentation/articles/sql-data-warehouse-develop-concurrency/
[管理]: /documentation/articles/sql-data-warehouse-manage-monitor/

<!--MSDN references-->
[ALTER INDEX]: https://msdn.microsoft.com/zh-cn/library/ms188388.aspx


<!--Other Web references-->
<!---HONumber=Mooncake_0606_2016-->