<!-- Temp remove overview, partition, statistics and temporary -->
<properties
   pageTitle="为 SQL 数据仓库中的表编制索引 | Azure"
   description="Azure SQL 数据仓库中的表索引入门。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="jrowlandjones"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.devlang="NA"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="data-services"
   ms.date="07/12/2016"
   wacn.date="12/12/2016"
   ms.author="jrj;barbkess;sonyama"/>

# 为 SQL 数据仓库中的表编制索引

> [AZURE.SELECTOR]
- [数据类型][]
- [分布][]
- [索引][]
<!-- 
- [概述][]
- [Partition][]
- [统计信息][]
- [临时][]
-->

SQL 数据仓库提供多种索引选项，包括[聚集列存储索引][]、[聚集索引和非聚集索引][]。此外，它还提供一个无索引选项，也称为[堆][]。本文介绍每种索引类型的优点，并提供通过索引获得最大性能的提示。有关如何在 SQL 数据仓库中创建表的详细信息，请参阅 [create table syntax][]（创建表语法）。

## 聚集列存储索引

默认情况下，如果未在表中指定任何索引选项，则 SQL 数据仓库将创建聚集列存储索引。聚集列存储表提供最高级别的数据压缩，以及最好的总体查询性能。一般而言，聚集列存储表优于聚集索引或堆表，并且通常是大型表的最佳选择。出于这些原因，在不确定如何编制表索引时，聚集列存储是最佳起点。

若要创建聚集列存储表，只需在 WITH 子句中指定 CLUSTERED COLUMNSTORE INDEX，或省略 WITH 子句：


    CREATE TABLE myTable   
      (  
        id int NOT NULL,  
        lastName varchar(20),  
        zipCode varchar(6)  
      )  
    WITH ( CLUSTERED COLUMNSTORE INDEX );


在某些情况下，聚集列存储可能不是很好的选择：

- 列存储表不支持辅助非聚集索引。可以考虑使用堆或聚集索引表。
- 列存储表不支持 varchar(max)、nvarchar(max) 和 varbinary(max)。可以考虑使用堆或聚集索引。
- 对瞬态数据使用列存储表可能会降低效率。可以考虑使用堆，甚至临时表。
- 包含少于 1 亿行的小型表。可以考虑使用堆表。

## 堆表

当在 SQL 数据仓库上暂时登录数据时，可能发现使用堆表可让整个过程更快速。这是因为堆的加载速度比索引表还要快，在某些情况下，可以从缓存执行后续读取。如果加载数据只是在做运行更多转换之前的预备，将表载入堆表将会远快于将数据载入聚集列存储表。此外，将数据载入[临时表][Temporary]也比将表载入永久存储更快速。

对于包含少于 1 亿行的小型查找表，堆表通常比较适合。超过 1 亿行后，聚集列存储表开始达到最佳压缩性能。

若要创建堆表，只需在 WITH 子句中指定 HEAP：


    CREATE TABLE myTable   
      (  
        id int NOT NULL,  
        lastName varchar(20),  
        zipCode varchar(6)  
      )  
    WITH ( HEAP );


## 聚集与非聚集索引

需要快速检索单个行时，聚集索引可能优于聚集列存储表。对于需要单个或极少数行查找才能极速执行的查询，请考虑使用聚集索引或非聚集辅助索引。使用聚集索引的缺点是只有在聚集索引列上使用高度可选筛选器的查询才可受益。若要改善其他列中的筛选器，可将非聚集索引添加到其他列。但是，添加到表中的每个索引将会增大空间和加载处理时间。

若要创建聚集索引表，只需在 WITH 子句中指定 CLUSTERED INDEX：


    CREATE TABLE myTable   
      (  
        id int NOT NULL,  
        lastName varchar(20),  
        zipCode varchar(6)  
      )  
    WITH ( CLUSTERED INDEX (id) );


若要在表中添加非聚集索引，只需在 WITH 子句中指定 CLUSTERED INDEX：


    CREATE INDEX zipCodeIndex ON t1 (zipCode);


## 优化聚集列存储索引

聚集列存储表将数据组织成多个段。拥有较高的段质量是在列存储表中实现最佳查询性能的关键。压缩行组中的行数可以测量分段质量。每个压缩的行组至少有 10 万行时的段质量最佳，而随着每个行组的行数趋于 1,048,576 行（这是行组可以包含的最大行数），性能会随之提升。

可以在系统上创建并使用以下视图来计算每个行组的平均行数，以及识别所有欠佳的聚集列存储索引。此视图中的最后一列将生成为 SQL 语句，以用于重建索引。


    CREATE VIEW dbo.vColumnstoreDensity
    AS
    SELECT
            GETDATE()                                                               AS [execution_date]
    ,       DB_Name()                                                               AS [database_name]
    ,       s.name                                                                  AS [schema_name]
    ,       t.name                                                                  AS [table_name]
    ,	COUNT(DISTINCT rg.[partition_number])					AS [table_partition_count]
    ,       SUM(rg.[total_rows])                                                    AS [row_count_total]
    ,       SUM(rg.[total_rows])/COUNT(DISTINCT rg.[distribution_id])               AS [row_count_per_distribution_MAX]
    ,	CEILING	((SUM(rg.[total_rows])*1.0/COUNT(DISTINCT rg.[distribution_id]))/1048576) AS [rowgroup_per_distribution_MAX]
    ,       SUM(CASE WHEN rg.[State] = 0 THEN 1                   ELSE 0    END)    AS [INVISIBLE_rowgroup_count]
    ,       SUM(CASE WHEN rg.[State] = 0 THEN rg.[total_rows]     ELSE 0    END)    AS [INVISIBLE_rowgroup_rows]
    ,       MIN(CASE WHEN rg.[State] = 0 THEN rg.[total_rows]     ELSE NULL END)    AS [INVISIBLE_rowgroup_rows_MIN]
    ,       MAX(CASE WHEN rg.[State] = 0 THEN rg.[total_rows]     ELSE NULL END)    AS [INVISIBLE_rowgroup_rows_MAX]
    ,       AVG(CASE WHEN rg.[State] = 0 THEN rg.[total_rows]     ELSE NULL END)    AS [INVISIBLE_rowgroup_rows_AVG]
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
    ,       'ALTER INDEX ALL ON ' + s.name + '.' + t.NAME + ' REBUILD;'             AS [Rebuild_Index_SQL]
    FROM    sys.[pdw_nodes_column_store_row_groups] rg
    JOIN    sys.[pdw_nodes_tables] nt                   ON  rg.[object_id]          = nt.[object_id]
                                                        AND rg.[pdw_node_id]        = nt.[pdw_node_id]
                                                        AND rg.[distribution_id]    = nt.[distribution_id]
    JOIN    sys.[pdw_table_mappings] mp                 ON  nt.[name]               = mp.[physical_name]
    JOIN    sys.[tables] t                              ON  mp.[object_id]          = t.[object_id]
    JOIN    sys.[schemas] s                             ON t.[schema_id]            = s.[schema_id]
    GROUP BY
            s.[name]
    ,       t.[name]
    ;


现在你已创建视图，请运行此查询来识别哪些表的行组中包含的行少于 10 万个。当然，如果你要寻求更理想的段质量，可以将 10 万这个阈值增大。


    SELECT	*
    FROM	[dbo].[vColumnstoreDensity]
    WHERE	COMPRESSED_rowgroup_rows_AVG < 100000
            OR INVISIBLE_rowgroup_rows_AVG < 100000


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
| [Rebuild\_Index\_SQL] | 用于重建表的列存储索引的 SQL |

##<a name="causes-of-poor-columnstore-index-quality"></a> 列存储索引质量不佳的原因

如果你已识别出段质量不佳的表，接下来可以找出根本原因。下面是段质量不佳的其他一些常见原因：

1. 生成索引时内存有压力
2. 有大量的 DML 操作
3. 小型或渗透负载操作
4. 过多的分区

这些因素可能导致列存储索引在每个行组中的行远远少于最佳数量（100 万）。它们还会造成行转到增量行组而不是压缩的行组。

### 生成索引时内存有压力

每个压缩行组的行数，与行宽度以及可用于处理行组的内存量直接相关。当行在内存不足的状态下写入列存储表时，列存储分段质量可能降低。因此，最佳做法是尽可能让写入到列存储索引表的会话访问最多的内存。因为内存与并发性之间有所取舍，正确的内存分配指导原则取决于表的每个行中的数据、已分配给系统的 DWU 数量，以及可以提供给将数据写入表的会话的并发访问槽数。作为一种最佳做法，如果使用 DW300 或更少，我们建议从 xlargerc 开始；如果使用 DW400 到 DW600，则从 largerc 开始；如果使用 DW1000 和更高，则从 mediumrc 开始。

### 有大量的 DML 操作

更新和删除行的大量 DML 操作可能造成列存储低效。当行组中的大多数行已修改时尤其如此。

- 从压缩的行组中删除某行只会以逻辑方式将该行标记为已删除。该行仍会保留在压缩的行组中，直到重新生成分区或表为止。
- 插入某行会将该行添加到名为增量行组的内部行存储表中。在增量行组已满且标记为已关闭之前，插入的行不会转换成列存储。达到 1,048,576 个行的容量上限后，行组将会关闭。
- 更新采用列存储格式的行将依次作为逻辑删除和插入进行处理。插入的行可以存储在增量存储中。

超出每个分区对齐分布区的 102,400 行批量阈值的批次更新和插入操作将直接写入列存储格式。但是，假设在平均分布的情况下，需要在单个操作中修改超过 6.144 百万行才发生这种情况。如果给定分区对齐分布区的行数少于 102,400 个，行将转到增量存储，并在插入足够的行、修改行以关闭行组或已创建索引之前，保留在增量存储中。

### 小型或渗透负载操作

流入 SQL 数据仓库的小型负载有时也称为渗透负载。它们通常代表系统引入的数据的接近恒定流。但是，由于此流接近连续状态，因此行的容量并不特别大。通常数据远低于直接加载到列存储格式所需的阈值。

在这些情况下，最好先将数据保存到 Azure Blob 存储中，并让它在加载之前累积。此技术通常称为*微批处理*。

### 过多的分区

另一个考虑因素是分区对聚集列存储表的影响。分区之前，SQL 数据仓库已将数据分散到 60 个数据库。进一步分区会分割数据。如果将分区，则要考虑的是**每个**分区必须至少有 100 万行，使用聚集列存储索引才有益。如果将表分割成 100 个分区，则你的表必须至少包含 60 亿行才能受益于聚集列存储索引（60 个分布区 * 100 个分区 * 100 万行）。如果包含 100 个分区的表没有 60 亿行，请减少分区数目，或考虑改用堆表。

在表中加载一些数据后，请遵循以下步骤来识别并重建聚集列存储索引质量欠佳的表。

##<a name="rebuilding-indexes-to-improve-segment-quality"></a> 重建索引以提升段质量

### 步骤 1：识别或创建使用适当资源类的用户

立即提升段质量的快速方法是重建索引。上述视图返回的 SQL 将返回可用于重建索引的 ALTER INDEX REBUILD 语句。重建索引时，请确保将足够的内存分配给要重建索引的会话。为此，请提高用户的资源类，该用户有权将此表中的索引重建为建议的最小值。无法更改数据库所有者用户的资源类，因此，如果尚未在系统中创建用户，必须先这样做。如果使用 DW300 或更少，建议的最小值为 xlargerc；如果使用 DW400 到 DW600，则建议的最小值为 largerc；如果使用 DW1000 和更高，则建议的最小值为 mediumrc。

以下示例演示如何通过提高资源类向用户分配更多内存。有关资源类以及如何创建新用户的详细信息，请参阅 [concurrency and workload management][Concurrency]（并发性和工作负荷管理）一文。


  EXEC sp_addrolemember 'xlargerc', 'LoadUser'


### 步骤 2：使用更高的用户资源类重建聚集列存储索引
以步骤 1 中所述用户的身份（例如 LoadUser，该用户现在使用更高的资源类）登录，然后执行 ALTER INDEX 语句。请确保此用户对重建索引的表拥有 ALTER 权限。这些示例演示如何重新生成整个列存储索引或如何重建单个分区。对于大型表，一次重建一个分区的索引比较合适。

或者，可以使用 [CTAS][] 将表复制到新表，而不要重建索引。哪种方法最合适？ 如果数据量很大，[CTAS][] 的速度通常比 [ALTER INDEX][] 要快。对于少量的数据，[ALTER INDEX][] 更容易使用，不需要你换出表。有关如何使用 CTAS 重建索引的详细信息，请参阅下面的**使用 CTAS 和分区切换重建索引**。


    -- Rebuild the entire clustered index
    ALTER INDEX ALL ON [dbo].[DimProduct] REBUILD
    
    
    
    -- Rebuild a single partition
    ALTER INDEX ALL ON [dbo].[FactInternetSales] REBUILD Partition = 5
    
    
    
    -- Rebuild a single partition with archival compression
    ALTER INDEX ALL ON [dbo].[FactInternetSales] REBUILD Partition = 5 WITH (DATA_COMPRESSION = COLUMNSTORE_ARCHIVE)
    
    
    
    -- Rebuild a single partition with columnstore compression
    ALTER INDEX ALL ON [dbo].[FactInternetSales] REBUILD Partition = 5 WITH (DATA_COMPRESSION = COLUMNSTORE)


在 SQL 数据仓库中重建索引是一项脱机操作。有关重建索引的详细信息，请参阅 [Columnstore Indexes Defragmentation][]（列存储索引碎片整理）中的 ALTER INDEX REBUILD 部分和语法主题 [ALTER INDEX][]。
 
### 步骤 3：验证聚集列存储段质量是否已改善
重新运行识别出段质量不佳的表的查询，并验证段质量是否已改善。如果段质量并未改善，原因可能是表中的行太宽。请考虑在重建索引时使用较高的资源类或 DWU。如果需要更多内存，

 
## 使用 CTAS 和分区切换重建索引

此示例使用 [CTAS][] 和分区切换重建表分区。


    -- Step 1: Select the partition of data and write it out to a new table using CTAS
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

    -- Step 2: Create a SWITCH out table
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

    -- Step 3: Switch OUT the data 
    ALTER TABLE [dbo].[FactInternetSales] SWITCH PARTITION 2 TO  [dbo].[FactInternetSales_20000101] PARTITION 2;

    -- Step 4: Switch IN the rebuilt data
    ALTER TABLE [dbo].[FactInternetSales_20000101_20010101] SWITCH PARTITION 2 TO  [dbo].[FactInternetSales] PARTITION 2;


有关使用 `CTAS` 重新创建分区的更多详细信息，请参阅 [Partition][]（分区）一文。

## 后续步骤

有关详细信息，请参阅有关<!-- [表概述][Overview]、--> [表数据类型][Data Types]、[分布表][Distribute] <!--、 [将表分区][Partition]、[维护表统计信息][Statistics]和[临时表][Temporary]--> 的文章。有关最佳实践的详细信息，请参阅 [SQL Data Warehouse Best Practices][]（SQL 数据仓库最佳实践）。

<!--Image references-->

<!--Article references-->
[Overview]: /documentation/articles/sql-data-warehouse-tables-overview/
[概述]: /documentation/articles/sql-data-warehouse-tables-overview/
[Data Types]: /documentation/articles/sql-data-warehouse-tables-data-types/
[数据类型]: /documentation/articles/sql-data-warehouse-tables-data-types/
[Distribute]: /documentation/articles/sql-data-warehouse-tables-distribute/
[分布]: /documentation/articles/sql-data-warehouse-tables-distribute/
[索引]: /documentation/articles/sql-data-warehouse-tables-index/
[Partition]: /documentation/articles/sql-data-warehouse-tables-partition/
[Statistics]: /documentation/articles/sql-data-warehouse-tables-statistics/
[统计信息]: /documentation/articles/sql-data-warehouse-tables-statistics/
[Temporary]: /documentation/articles/sql-data-warehouse-tables-temporary/
[临时]: /documentation/articles/sql-data-warehouse-tables-temporary/
[Concurrency]: /documentation/articles/sql-data-warehouse-develop-concurrency/
[CTAS]: /documentation/articles/sql-data-warehouse-develop-ctas/
[SQL Data Warehouse Best Practices]: /documentation/articles/sql-data-warehouse-best-practices/

<!--MSDN references-->
[ALTER INDEX]: https://msdn.microsoft.com/zh-cn/library/ms188388.aspx
[堆]: https://msdn.microsoft.com/zh-cn/library/hh213609.aspx
[聚集索引和非聚集索引]: https://msdn.microsoft.com/zh-cn/library/ms190457.aspx
[create table syntax]: https://msdn.microsoft.com/zh-cn/library/mt203953.aspx
[Columnstore Indexes Defragmentation]: https://msdn.microsoft.com/zh-cn/library/dn935013.aspx#Anchor_1
[聚集列存储索引]: https://msdn.microsoft.com/zh-cn/library/gg492088.aspx

<!--Other Web references-->

<!---HONumber=Mooncake_0801_2016-->
