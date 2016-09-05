<!-- Temp remove overview, partiion, statistics and temporary -->
<properties
   pageTitle="在 SQL 数据仓库中分布表 | Azure"
   description="开始在 Azure SQL 数据仓库中分布表。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="jrowlandjones"
   manager="barbkess"
   editor=""/>  


<tags
   ms.service="sql-data-warehouse"
   ms.date="08/01/2016"
   wacn.date="09/05/2016"/>

# 在 SQL 数据仓库中分布表

> [AZURE.SELECTOR]
- [概述][]
- [数据类型][]
- [分布][]
- [索引][]
- [Partition][]
- [统计信息][]
- [临时][]

SQL 数据仓库是一种大规模并行处理 (MPP) 分布式数据库系统。通过将数据和处理能力分布于多个节点，SQL 数据仓库能够提供极大的缩放性 - 远超任何单一系统。确定如何在 SQL 数据仓库内分布数据是达到最佳性能的最重要因素之一。要达到最佳性能的关键是将数据移动降到最低，而将数据移动降到最低的关键是选择正确的分布策略。

## 了解数据移动

在 MPP 系统中，每个表中的数据被分散到多个底层数据库。只需传递 MPP 系统上优化程度最高的查询以便在单个分布式数据库上执行，而无需与其他数据库交互。例如，假设包含销售数据的数据库有“销售”和“客户”两个表。如果查询需要将销售表和客户表联接，将销售和客户表按客户编号划分，并将每个客户放在不同的数据库，则任何联接销售和客户的查询都可以在各个数据库中求解，而不需要知道其他数据库的存在。相比之下，如果将销售数据按订单编号划分，将客户数据按客户编号划分，则任何给定数据库都不包含每位客户的相应数据，因此，如果你想要将销售数据联接到客户数据，便需要从其他数据库获取每位客户的数据。在第二个示例中，需要移动数据，以将客户数据移到销售数据，如此才可以联接两个表。

数据移动不一定是一件坏事，有时是求解查询的必要手段。但如果可以避免此附加步骤，则查询的运行速度自然就更快。数据移动往往发生在联接表或者执行聚合时。你通常需要执行这两种操作，因此即使可以针对其中一种方案（例如联接）进行优化，也仍需要移动数据来帮解决另一种方案（例如聚合）。诀窍就是找出哪种方案的作用更小。大多数情况下，在通常联接的列上分布大型事实表，是将数据移动降到最低的最有效方法。相比于分布聚合涉及到的列数据，分布联接列上的数据是减少数据移动更为普遍的方法。

## 选择分布方法

SQL 数据仓库在幕后将数据划分成 60 个数据库。每个数据库称为**分布区**。当数据载入每个表时，SQL 数据仓库必须知道如何将数据分散到这 60 个分布区。

分布方法在表级别定义，目前有两种选择：

1. **轮循机制**：平均、随机分布数据。
2. **哈希分布**：根据单个列中的哈希值分布数据。

默认情况下，如果未定义数据分布方法，表将使用**轮循机制**分布方法进行分布。但是，随着你对自己的实施方案日益精通，可以考虑使用**哈希分布**表将数据移动降到最低，从而使查询性能达到优化。

### 轮循机制表

使用轮循机制数据分布方法名符其实。加载数据后，每行只发送到下一个分布区。此种数据分布方法始终将数据非常平均地随机分布到所有分布区。也就是说，在轮循机制处理期间放置数据时，不会执行任何排序。出于此原因，轮循机制分布有时称为随机哈希。使用轮循机制分布式表时不需要了解数据。出于此原因，轮循机制表通常具有良好的加载目标。

默认情况下，如果未选择分布方法，则会使用轮循机制方法。尽管轮循机制表很容易使用，但因为数据随机分布在系统中，这意味着系统无法保证每个行位于哪个分布区。因此，系统有时候需要调用数据移动操作，以便在求解查询前更加合理地组织数据。此附加步骤可能使查询变慢。

在以下情况下，请考虑对表使用轮循机制分布：

- 从一个简单的起点入门时
- 没有明显的联接键时
- 没有合适的候选列可供哈希分布表时
- 表没有与其他表共享通用的联接键时
- 该联接比查询中的其他联接更不重要时
- 表是临时过渡表时

这两个示例都将创建轮循机制表：


    -- Round Robin created by default
    CREATE TABLE [dbo].[FactInternetSales]
    (   [ProductKey]            int          NOT NULL
    ,   [OrderDateKey]          int          NOT NULL
    ,   [CustomerKey]           int          NOT NULL
    ,   [PromotionKey]          int          NOT NULL
    ,   [SalesOrderNumber]      nvarchar(20) NOT NULL
    ,   [OrderQuantity]         smallint     NOT NULL
    ,   [UnitPrice]             money        NOT NULL
    ,   [SalesAmount]           money        NOT NULL
    )
    ;

    -- Explicitly Created Round Robin Table
    CREATE TABLE [dbo].[FactInternetSales]
    (   [ProductKey]            int          NOT NULL
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
    ,   DISTRIBUTION = ROUND_ROBIN
    )
    ;


> [AZURE.NOTE] 尽管轮循机制是默认的表类型，但最好在 DDL 中明确规定此方法，使其他人能够清楚了解表布局的意图。

### 哈希分布表

使用**哈希分布**算法来分布表可以减少查询时的数据移动，从而改善许多方案的性能。哈希分布表是在选择的单个列中使用哈希算法分散在分布式数据库之间的表。分布列确定如何将数据分散在分布式数据库之间。哈希函数使用分布列将行分配到分布区。哈希算法与最终的分布区具有确定性。也就是，相同数据类型的相同值始终使用相同的分布区。

本示例将创建基于 ID 分布的表：


    CREATE TABLE [dbo].[FactInternetSales]
    (   [ProductKey]            int          NOT NULL
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
    ,  DISTRIBUTION = HASH([ProductKey])
    )
    ;


## 选择分布列

选择**哈希分布**表时，必须选择单个分布列。选择分布列时，需要考虑三个主要因素。

选择具有以下特征的单个列：

1. 不会更新
2. 平均分布数据，以避免数据偏斜
3. 最小化数据移动

### 选择不会更新的分布列

分布列不可更新，因此，请选择包含静态值的列。如果列需要更新，该列通常不是理想的分布候选项。如果必须更新分布列，做法是先删除行，然后插入新行。

### 选择将会平均分布数据的分布列

分布式系统的执行速度只与其最慢的分布区一样快，因此务必将任务平均分散于各分布区，以便让系统实现平衡的执行性能。工作将根据每个分布区的数据所在位置，分散到分布式系统。这对于选择正确的分布列来分布数据非常重要，这样，每个分布区都有相等的工作，并且完成其部分工作所需的时间相同。当工作已妥善分散到系统时，就称为平衡的执行。如果数据未平均分散在系统中并且不平衡，则称为**数据偏斜**。

若要平均分散数据以避免数据偏斜，请在选择分布列时考虑以下事项：

1. 选择包含大量非重复值的列。
2. 避免将数据分布在有一些值频繁出现或 null 频繁出现的列中。
3. 避免在日期列中分布数据。
4. 避免分布在少于 60 个列中

由于每个值将散列到 60 个分布区中的一个，若要实现平均分布，需要选择极为独特并提供超过 60 个唯一值的列。举例来说，假设在极端情况下，某个列只有 40 个唯一值。如果选择此列作为分布键，则该表的数据只会分散到系统的一部分，以致 20 个分布区不包含任何数据，并且没有要执行的处理。相反，如果数据平均分散到 60 个分布区，则其他 40 个分布区有更多工作需要完成。

如果在频繁出现 null 的列中分布表，则所有 null 值将出现在同一个分布区，该分布区必须完成比其他分布区更多的工作，这会使整个系统变慢。如果查询对日期具有高度可选性，并且只有少数日期涉及查询，则在日期列中分布还可能导致处理偏斜。

若没有理想的候选列，请考虑使用轮循机制作为分布方法。

### 选择可将数据移动降到最低的分布列

通过选择适当分布列来将数据移动降到最低，是让 SQL 数据仓库的性能达到优化的最重要策略之一。数据移动往往发生在联接表或者执行聚合时。在 `JOIN`、`GROUP BY`、`DISTINCT`、`OVER` 和 `HAVING` 子句中使用的列都生成**适当**的哈希分布候选项。另一方面，`WHERE` 子句中的列**不是**理想的哈希列候选项，因为它们限制哪些分布区参与查询。

一般而言，如果有两个经常涉及联接的大型事实表，将两个表分布到其中一个联接列可获得最佳性能。如果有永远不会联接到另一个大型事实表的表，请查找经常出现在 `GROUP BY` 子句中的列。

必须符合几个关键条件，以避免联接期间数据移动：

1. 参与联接的列的相关表必须哈希分布在**一个**联接列中。
2. 两个表之间联接列的数据类型必须匹配。
3. 必须使用 equals 运算符联接列。
4. 联接类型不能是 `CROSS JOIN`。


## 排查数据偏斜问题

使用哈希分布方法分布表数据时，某些分布可能会偏斜，比其他表具有更多数据。过度的数据偏斜可能会影响查询性能，因为分布式查询的最终结果必须先等待运行时间最长的分布完成。根据数据偏斜的程度，你可能需要解决这种问题。

### 标识偏斜

识别表偏斜的简单方法是使用 `DBCC PDW_SHOWSPACEUSED`。这是一种非常快捷简便的方法，可以查看存储在数据库中每组 60 个分布区内的表行数目。请记住，为了获得最平衡的性能，分布式表中的行应该平均分散在所有分布区中。


    -- Find data skew for a distributed table
    DBCC PDW_SHOWSPACEUSED('dbo.FactInternetSales');


但是，如果你查询 Azure SQL 数据仓库动态管理视图 (DMV)，则可以执行更详细的分析。若要开始，请使用 [Table Overview][Overview]（表概述）一文中的 SQL 创建 <!--[-->dbo.vTableSizes<!--][]--> 视图。创建该视图后，运行此查询来识别哪些表有 10% 以上的数据偏斜。


    select *
    from dbo.vTableSizes 
    where two_part_name in 
        (
        select two_part_name
    	from dbo.vTableSizes
        where row_count > 0
        group by two_part_name
        having min(row_count * 1.000)/max(row_count * 1.000) > .10
        )
    order by two_part_name, row_count
    ;


### 解决数据倾斜

不能保证可以解决所有偏斜。在某些情况下，某些查询中的表性能可能比数据偏斜的损害更严重。若要确定是否应该解决表中的数据偏斜，你应该尽可能了解工作负荷中的数据卷和查询。调查偏斜影响的方法之一是使用 [Query Monitoring][]（查询监视）一文中的步骤来监视偏斜对查询性能的影响，尤其是对单个分布区完成查询所需时间的影响。

分布式数据是找出将数据偏斜降至最低与将数据移动降至最低两者之间的适当平衡。这些可能是相反的目标，有时你会想要保留数据偏斜，以减少数据移动。例如，当分布列经常是联接和聚合中的共享列时，你便会将数据移动降至最低。最少的数据移动带来的好处可能胜过数据偏斜造成的影响。

解决数据偏斜的典型方法是重新创建具有不同分布列的表。由于没有任何方法可更改现有表上的分布列，因此更改表分布的方法是使用 [CTAS][] 重新创建表。下面两个示例演示了如何解决数据偏斜：

### 示例 1︰重新创建包含新分布列的表

本示例使用 [CTAS][] 来重新创建具有不同哈希分布列的表。


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

    --Create statistics on new table
    CREATE STATISTICS [ProductKey] ON [FactInternetSales_CustomerKey] ([ProductKey]);
    CREATE STATISTICS [OrderDateKey] ON [FactInternetSales_CustomerKey] ([OrderDateKey]);
    CREATE STATISTICS [CustomerKey] ON [FactInternetSales_CustomerKey] ([CustomerKey]);
    CREATE STATISTICS [PromotionKey] ON [FactInternetSales_CustomerKey] ([PromotionKey]);
    CREATE STATISTICS [SalesOrderNumber] ON [FactInternetSales_CustomerKey] ([SalesOrderNumber]);
    CREATE STATISTICS [OrderQuantity] ON [FactInternetSales_CustomerKey] ([OrderQuantity]);
    CREATE STATISTICS [UnitPrice] ON [FactInternetSales_CustomerKey] ([UnitPrice]);
    CREATE STATISTICS [SalesAmount] ON [FactInternetSales_CustomerKey] ([SalesAmount]);

    --Rename the tables
    RENAME OBJECT [dbo].[FactInternetSales] TO [FactInternetSales_ProductKey];
    RENAME OBJECT [dbo].[FactInternetSales_CustomerKey] TO [FactInternetSales];


### 示例 2：使用轮循机制分布重新创建表

本示例使用 [CTAS][] 来重新创建具有轮循机制分布而不是哈希分布的表。这种变化将生成平均的数据分布，但代价是数据移动加大。


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

    --Create statistics on new table
    CREATE STATISTICS [ProductKey] ON [FactInternetSales_ROUND_ROBIN] ([ProductKey]);
    CREATE STATISTICS [OrderDateKey] ON [FactInternetSales_ROUND_ROBIN] ([OrderDateKey]);
    CREATE STATISTICS [CustomerKey] ON [FactInternetSales_ROUND_ROBIN] ([CustomerKey]);
    CREATE STATISTICS [PromotionKey] ON [FactInternetSales_ROUND_ROBIN] ([PromotionKey]);
    CREATE STATISTICS [SalesOrderNumber] ON [FactInternetSales_ROUND_ROBIN] ([SalesOrderNumber]);
    CREATE STATISTICS [OrderQuantity] ON [FactInternetSales_ROUND_ROBIN] ([OrderQuantity]);
    CREATE STATISTICS [UnitPrice] ON [FactInternetSales_ROUND_ROBIN] ([UnitPrice]);
    CREATE STATISTICS [SalesAmount] ON [FactInternetSales_ROUND_ROBIN] ([SalesAmount]);

    --Rename the tables
    RENAME OBJECT [dbo].[FactInternetSales] TO [FactInternetSales_HASH];
    RENAME OBJECT [dbo].[FactInternetSales_ROUND_ROBIN] TO [FactInternetSales];


## 后续步骤

有关表设计的详细信息，请参阅 [Distribute][]（分布）、[Index][]（索引）、[Partition][]（分区）、[Data Types][]（数据类型）、[Statistics][]（统计信息）和 [Temporary Tables][Temporary]（临时表）文章。

有关最佳实践的概述，请参阅 [SQL 数据仓库最佳实践][]。


<!--Image references-->

<!--Article references-->
[Overview]: /documentation/articles/sql-data-warehouse-tables-overview/
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
[SQL Data Warehouse Best Practices]: /documentation/articles/sql-data-warehouse-best-practices/
[Query Monitoring]: /documentation/articles/sql-data-warehouse-manage-monitor/
[dbo.vTableSizes]: /documentation/articles/sql-data-warehouse-tables-overview/#querying-table-sizes

<!--MSDN references-->
[DBCC PDW_SHOWSPACEUSED()]: https://msdn.microsoft.com/zh-cn/library/mt204028.aspx

<!--Other Web references-->

<!---HONumber=Mooncake_0829_2016-->