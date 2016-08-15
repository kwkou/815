<properties
   pageTitle="SQL 数据仓库中的表分区 | Azure"
   description="有关在开发解决方案时使用 Azure SQL 数据仓库中的表分区的技巧。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="jrowlandjones"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="03/25/2016"
   wacn.date="05/23/2016"/>

# SQL 数据仓库中的表分区

若要将 SQL Server 分区定义迁移到 SQL 数据仓库，请执行以下操作：

- 删除 SQL Server 分区函数和方案，因为当你创建表时系统会帮你管理。
- 在创建表时定义分区。只需指定分区边界点，以及希望边界点是有效的 `RANGE RIGHT` 还是 `RANGE LEFT`。

注意：若要了解 SQL Server 中分区的详细信息，请参阅[分区表和索引](https://msdn.microsoft.com/zh-cn/library/ms190787.aspx)。


### 分区大小
SQL DW 为 DBA 提供数个表类型选项：堆、群集索引 (CI) 和群集列存储索引 (CCI)。DBA 还可以针对每个表类型分区表，还就是将其分区成多个部分，以提升性能。但是，创建具有太多分区的表实际上有可能会导致性能降低，或导致查询在某些情况下失败。对于 CCI 表，尤其要考虑到这一点。若要使数据分区有所帮助，DBA 务必要了解使用数据分区的时机，以及要创建的分区数目。这些指导旨在帮助 DBA 在遇到状况时做出最佳选择。

表分区通常在两种主要方式下很有用：

1. 切换分区以快速截断表的某个部分。常用的设计是针对包含只供某段预先决定期限内使用的行的事实表。例如，销售事实表可能仅包含过去 36 个月的数据。在每个月月底，便从表删除最旧月份的销售数据。只要删除最旧月份的所有行即可完成此操作，但逐列删除批量数据可能花费很多时间。为了优化此方案，SQL DW 支持分区交换，使你只需执行一项操作，就能快速删除分区中的整个行集。   

2. 如果查询在分区列上放置谓词，数据分区便可让查询轻松排除处理大量的行集（即分区）。例如，如果使用销售日期字段将销售事实表分区成 36 个月，以销售日期进行筛选的查询便可以跳过处理不符合筛选条件的分区。实际上，以这种方式使用的数据分区是粗糙的索引。

在 SQL DW 中创建群集列存储索引时，DBA 需注意事项另一个因素：行数。CCI 表可以实现高度压缩，并帮助 SQL DW 加速查询性能。由于压缩在 SQL DW 内部运行，因此 CCI 表中的每个分区在压缩数据前都必须拥有相当多的行。此外，SQL DW 在批量的分布之间分布数据，而且分区进一步分割每个分布区。为了实现最佳压缩和性能，每个分布区与分区都需要至少 100,000 个行。根据上述示例，如果销售事实表包含 36 个月的分区，并假设 SQL DW 有 60 个分布区，则销售事实表每个月应包含 600 万行，或者在填充所有月份时包含 2.16 亿行。如果表包含的行数远少于建议的最小值，DBA 应该考虑创建具有较少分区的表，以增加每个分布区的行数。


若要在分区级别调整当前数据库的大小，请使用类似于下面的查询：

```
SELECT      s.[name] AS [schema_name]
,           t.[name] AS [table_name]
,           i.[name] AS [index_name]
,           p.[partition_number] AS [partition_number]
,           SUM(a.[used_pages]*8.0) AS [partition_size_kb]
,           SUM(a.[used_pages]*8.0)/1024 AS [partition_size_mb]
,           SUM(a.[used_pages]*8.0)/1048576 AS [partition_size_gb]
,           p.[rows] AS [partition_row_count]
,           rv.[value] AS [partition_boundary_value]
,           p.[data_compression_desc] AS [partition_compression_desc]
FROM        sys.schemas s
JOIN        sys.tables t ON t.[schema_id] = s.[schema_id]
JOIN        sys.partitions p ON p.[object_id] = t.[object_id]
JOIN        sys.allocation_units a ON a.[container_id] = p.[partition_id]
JOIN        sys.indexes i ON i.[object_id] = p.[object_id]
                                            AND i.[index_id] = p.[index_id]
JOIN        sys.data_spaces ds ON ds.[data_space_id] = i.[data_space_id]
LEFT JOIN   sys.partition_schemes ps ON ps.[data_space_id] = ds.[data_space_id]
LEFT JOIN   sys.partition_functions pf ON pf.[function_id] = ps.[function_id]
LEFT JOIN   sys.partition_range_values rv ON rv.[function_id] = pf.[function_id]
                                            AND rv.[boundary_id] = p.[partition_number]
WHERE       p.[index_id] <=1
GROUP BY    s.[name]
,           t.[name]
,           i.[name]
,           p.[partition_number]
,           p.[rows]
,           rv.[value]
,           p.[data_compression_desc]
;
```

分布区总数等于我们创建表时使用的存储位置数。也就是说，每个表根据每一个分布区创建一次。

你也可以大致预测有多少个行，因而预测每个分区有多大。源数据仓库中的分区再细分为每个分布区。

确定分区大小时，请参考以下计算公式：

MPP 分区大小 = SMP 分区大小 / 分布区数

你可以使用以下查询，找出 SQL 数据仓库数据库有多少个分布区：

```
SELECT  COUNT(*)
FROM    sys.pdw_distributions
;
```

既然知道了源系统中的每个分区有多大，以及预期的 SQLDW 大小为何，就可以确定分区边界。

### 工作负荷管理
需要纳入表分区决策的最后一项信息是工作负荷管理。在 SQL 数据仓库中，此功能控制在查询运行期间分配给每个分布的最大内存。有关[工作负荷管理]的详细信息，请参阅下面的文章。理想情况下，调整分区大小时，将考虑 inmemory 操作（如列存储索引重建）。索引重建是内存密集型操作。因此，你需要确保重建分区索引不会耗尽内存。从默认角色切换到其他可用角色之一，即可增加查询可用的内存量。

查询资源调控器动态管理视图即可获取每个分布的内存分配信息。事实上，内存授予小于以下数据。但是，这可以提供指导，以便你在针对数据管理操作调整分区大小时使用。

```
SELECT  rp.[name] AS [pool_name]
,       rp.[max_memory_kb] AS [max_memory_kb]
,       rp.[max_memory_kb]/1024 AS [max_memory_mb]
,       rp.[max_memory_kb]/1048576 AS [mex_memory_gb]
,       rp.[max_memory_percent] AS [max_memory_percent]
,       wg.[name] AS [group_name]
,       wg.[importance] AS [group_importance]
,       wg.[request_max_memory_grant_percent] AS [request_max_memory_grant_percent]
FROM    sys.dm_pdw_nodes_resource_governor_workload_groups wg
JOIN    sys.dm_pdw_nodes_resource_governor_resource_pools rp ON wg.[pool_id] = rp.[pool_id]
WHERE   wg.[name] like 'SloDWGroup%'
AND     rp.[name] = 'SloDWPool'
;
```

> [AZURE.NOTE] 尽量避免将分区大小调整超过超大型资源类所提供的内存授予。如果分区成长超过此数据，就冒着内存压力的风险，进而导致比较不理想的压缩。

## 分区切换
若要切换两个表间的分区，必须确定分区对齐其各自的边界，而且表定义匹配。检查约束不适用于强制表中的值范围，源表必须包含与目标表相同的分区边界。如果情况不是如此，则分区切换将失败，因为分区元数据不会同步。

### 如何拆分包含数据的分区
使用 `CTAS` 语句是拆分包含数据的分区的最有效方法。如果分区表是群集列存储，则表分区必须为空才能拆分。

以下示例显示了每个分区包含一个行的分区列存储表：

```
CREATE TABLE [dbo].[FactInternetSales]
(
        [ProductKey]            int          NOT NULL
    ,   [OrderDateKey]          int          NOT NULL
    ,   [CustomerKey]           int          NOT NULL
    ,   [PromotionKey]          int          NOT NULL
    ,   [SalesOrderNumber]      nvarchar(20) NOT NULL
    ,   [OrderQuantity]         smallint     NOT NULL
    ,   [UnitPrice]             money        NOT NULL
    ,   [SalesAmount]           money        NOT NULL
)
WITH
(   CLUSTERED COLUMNSTORE INDEX
,   DISTRIBUTION = HASH([ProductKey])
,   PARTITION   (   [OrderDateKey] RANGE RIGHT FOR VALUES
                    (20000101
                    )
                )
)
;

INSERT INTO dbo.FactInternetSales
VALUES (1,19990101,1,1,1,1,1,1);
INSERT INTO dbo.FactInternetSales
VALUES (1,20000101,1,1,1,1,1,1);


CREATE STATISTICS Stat_dbo_FactInternetSales_OrderDateKey ON dbo.FactInternetSales(OrderDateKey);
```

> [AZURE.NOTE] 通过创建统计信息对象，我们可以确保表元数据更加准确。如果我们省略了创建统计信息，SQL 数据仓库将使用默认值。有关统计信息的详细信息，请参阅[统计信息][]。

然后，我们可以利用 `sys.partitions` 目录视图查询行计数：

```
SELECT  QUOTENAME(s.[name])+'.'+QUOTENAME(t.[name]) as Table_name
,       i.[name] as Index_name
,       p.partition_number as Partition_nmbr
,       p.[rows] as Row_count
,       p.[data_compression_desc] as Data_Compression_desc
FROM    sys.partitions p
JOIN    sys.tables     t    ON    p.[object_id]   = t.[object_id]
JOIN    sys.schemas    s    ON    t.[schema_id]   = s.[schema_id]
JOIN    sys.indexes    i    ON    p.[object_id]   = i.[object_Id]
                            AND   p.[index_Id]    = i.[index_Id]
WHERE t.[name] = 'FactInternetSales'
;
```

如果尝试拆分此表，将会收到错误：

```
ALTER TABLE FactInternetSales SPLIT RANGE (20010101);
```

消息 35346，级别 15，状态 1，行 44:
ALTER PARTITION 语句的 SPLIT 子句失败，因为分区不为空。仅当表上存在列存储索引时，才可以拆分空分区。请考虑在发出 ALTER PARTITION 语句前禁用列存储索引，然后在 ALTER PARTITION 完成后重建列存储索引。

但是，我们可以使用 `CTAS` 创建新表以保存数据。

```
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
FROM    FactInternetSales
WHERE   1=2
;
```

分区边界已对齐，因此允许切换。这使源表有空白分区可供我们完成后续拆分。

```
ALTER TABLE FactInternetSales SWITCH PARTITION 2 TO  FactInternetSales_20000101 PARTITION 2;

ALTER TABLE FactInternetSales SPLIT RANGE (20010101);
```

接下来只需使用 `CTAS` 将数据对齐新的分区边界，并将数据切换回到主表

```
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
FROM    [dbo].[FactInternetSales_20000101]
WHERE   [OrderDateKey] >= 20000101
AND     [OrderDateKey] <  20010101
;

ALTER TABLE dbo.FactInternetSales_20000101_20010101 SWITCH PARTITION 2 TO dbo.FactInternetSales PARTITION 2;
```

完成数据移动后，最好是刷新目标表上的统计信息，确保统计信息可在其各自的分区中准确反映数据的新分布：

```
UPDATE STATISTICS [dbo].[FactInternetSales];
```

### 表分区源代码管理
若要避免表定义在源代码管理系统中**失效**，可以考虑以下方法：

1. 将表创建为分区表，但不包含分区值

```
CREATE TABLE [dbo].[FactInternetSales]
(
    [ProductKey]            int          NOT NULL
,   [OrderDateKey]          int          NOT NULL
,   [CustomerKey]           int          NOT NULL
,   [PromotionKey]          int          NOT NULL
,   [SalesOrderNumber]      nvarchar(20) NOT NULL
,   [OrderQuantity]         smallint     NOT NULL
,   [UnitPrice]             money        NOT NULL
,   [SalesAmount]           money        NOT NULL
)
WITH
(   CLUSTERED COLUMNSTORE INDEX
,   DISTRIBUTION = HASH([ProductKey])
,   PARTITION   (   [OrderDateKey] RANGE RIGHT FOR VALUES
                    ()
                )
)
;
```

2. 在部署过程中 `SPLIT` 表：

```
-- Create a table containing the partition boundaries

CREATE TABLE #partitions
WITH
(
    LOCATION = USER_DB
,   DISTRIBUTION = HASH(ptn_no)
)
AS
SELECT  ptn_no
,       ROW_NUMBER() OVER (ORDER BY (ptn_no)) as seq_no
FROM    (
        SELECT CAST(20000101 AS INT) ptn_no
        UNION ALL
        SELECT CAST(20010101 AS INT)
        UNION ALL
        SELECT CAST(20020101 AS INT)
        UNION ALL
        SELECT CAST(20030101 AS INT)
        UNION ALL
        SELECT CAST(20040101 AS INT)
        ) a
;

-- Iterate over the partition boundaries and split the table

DECLARE @c INT = (SELECT COUNT(*) FROM #partitions)
,       @i INT = 1                                 --iterator for while loop
,       @q NVARCHAR(4000)                          --query
,       @p NVARCHAR(20)     = N''                  --partition_number
,       @s NVARCHAR(128)    = N'dbo'               --schema
,       @t NVARCHAR(128)    = N'FactInternetSales' --table
;

WHILE @i <= @c
BEGIN
    SET @p = (SELECT ptn_no FROM #partitions WHERE seq_no = @i);
    SET @q = (SELECT N'ALTER TABLE '+@s+N'.'+@t+N' SPLIT RANGE ('+@p+N');');

    -- PRINT @q;
    EXECUTE sp_executesql @q;

    SET @i+=1;
END

-- Code clean-up

DROP TABLE #partitions;
```

使用这种方法时，源代码管理中的代码将保持静态，允许动态的分区边界值，并不断地与仓库一起演进。

>[AZURE.NOTE] 相比于 SQL Server，分区切换有一些差异。请务必阅读[迁移代码][]，以深入了解此主题。


## 后续步骤
成功将数据库架构迁移到 SQL 数据仓库后，可以继续阅读下列文章之一：

- [迁移数据][]
- [迁移代码][]

<!--Image references-->

<!-- Article references -->
[迁移代码]: /documentation/articles/sql-data-warehouse-migrate-code/
[迁移数据]: /documentation/articles/sql-data-warehouse-migrate-data/
[统计信息]: /documentation/articles/sql-data-warehouse-develop-statistics/
[工作负荷管理]: /documentation/articles/sql-data-warehouse-develop-concurrency/

<!-- MSDN Articles -->

<!-- Other web references -->

<!---HONumber=Mooncake_0328_2016-->