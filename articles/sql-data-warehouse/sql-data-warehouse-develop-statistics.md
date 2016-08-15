<properties
   pageTitle="管理 SQL 数据仓库中的统计信息 | Azure"
   description="有关在开发解决方案时于 Azure SQL 数据仓库中管理统计信息的技巧。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="jrowlandjones"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="05/10/2016"
   wacn.date="05/30/2016"/>

# 管理 SQL 数据仓库中的统计信息
 SQL 数据仓库使用统计信息来评估以不同方式执行分布式查询的成本。如果统计信息准确，则查询优化器可以生成高质量查询计划来改善查询性能。

若要达到 SQL 数据仓库预计提供的查询性能，创建和更新统计信息极为重要。本指南提供统计信息的概述，然后说明如何：

- 在数据库设计阶段创建统计信息
- 在数据库维护阶段更新统计信息
- 使用系统视图和函数查看统计信息

## 统计信息简介

单列统计信息是包含单一列中值范围和频率相关信息的对象。查询优化器使用此直方图来估算查询结果中的行数。这会直接影响到有关如何优化查询的决策。

多列统计信息是基于列列表创建的统计信息。其中包含列表中第一个列的单列统计信息，加上一些跨列相关性信息（称为密度）。多列统计信息可以改善某些操作（例如复合 Join 和 Group By）的查询性能。

有关详细信息，请参阅 MSDN 上的 [DBCC SHOW\_STATISTICS][]。

## 为何需要统计信息？
如果没有正确的统计信息，你将无法获得 SQL 数据仓库预期提供的性能。表和列没有 SQL 数据仓库自动生成的统计信息，因此需要你自行创建。在创建表时创建统计信息，然后在填入统计信息后予以更新是个不错的主意。

> [AZURE.NOTE] 如果使用 SQL Server，你可以依赖 SQL Server 根据需要为你创建和更新单列统计信息。SQL 数据仓库在这方面有所不同。由于数据是分布式的，因此 SQL 数据仓库不会自动聚合所有分布式数据的统计信息。它只在创建和更新统计信息时生成聚合统计信息。

## 何时创建统计信息
一组一致的最新统计信息是 SQL 数据仓库的重要部分。因此，请务必在设计表的过程中创建统计信息。

创建每个列的单列统计信息是开始使用统计信息的简单方式。不过，创建和更新统计信息的性能与成本之间总有一些取舍。如果你创建了所有列的单列统计信息，后来发现更新所有统计信息要花费太长的时间，你始终可以丢弃一些统计信息，或较常更新其中一些统计信息。

仅当列位于复合 Join 或 Group By 子句中时，查询优化器才使用多列统计信息。复合筛选条件目前并不会受益于多列统计信息。

开始进行 SQL 数据仓库开发时，实施以下模式是个不错的主意：
- 对每个表上的每个列创建单列统计信息
- 对 Join 和 Group By 子句中查询所用的列创建多列统计信息。

在确定如何查询数据后，你可以修改此模型 - 尤其是在表的范围很广时。有关更高级的方法，请参阅 [实施统计信息管理](## 实施统计信息管理) 部分。

## 何时更新统计信息
请务必在数据库管理日常工作中纳入更新统计信息。数据库中的数据发布改变时，就需要更新统计信息。否则，你可能会发现查询性能欠佳，而进一步排查查询问题所做的努力可能不值得。

最佳实践之一是每天在添加新日期后，更新有关日期列的统计信息。每次有新行载入数据仓库时，就会添加新的加载日期或事务日期。这些操作会更改数据分布情况并使统计信息过时。相反地，有关客户表中的国家/地区列的统计信息可能永远不需要更新，因为值的分布通常不会变化。假设客户间的分布固定不变，将新行添加到表变化并不会改变数据分布情况。但是，如果数据仓库只包含一个国家/地区，并且引入了来自新国家/地区的数据，从而导致存储了多个国家/地区的数据，那么，就绝对需要更新有关国家/地区列的统计信息。

在排查查询问题时，首先要询问的问题之一就是“统计信息是最新的吗？”

此问题不可以根据数据期限提供答案。如果对基础数据未做重大更改，则最新的统计信息对象有可能非常陈旧。如果行数有明显变化或给定列的值分布有重大变化，则需要更新统计信息。

例如，**SQL Server**（不是 SQL 数据仓库）在以下情况下会自动更新这些统计信息：

- 如果表中包含零行，则当你添加一行（或多行）时，则自动更新统计信息
- 如果在最初包含不到 500 行的表中添加超过 500 行（例如，表最初包含 499 行，后来你添加了 500 行，总共 999 行），则会自动更新 
- 行数超过 500 之后，必须额外添加 500 行并加上表大小的 20%，然后系统才会对统计信息执行自动更新

由于系统未提供 DMV 来确定自上次更新统计信息以来表中的数据是否发生更改，因此，如果知道统计信息的期限的话，也许你可以大致猜出更新状态。可以使用以下查询来确定上次更新每个表的统计信息的时间。

> [AZURE.NOTE] 请记住，如果给定列的值分布有重大变化，则应该更新统计信息，不管上次更新时间为何。

    SELECT
        sm.[name] AS [schema_name],
        tb.[name] AS [table_name],
        co.[name] AS [stats_column_name],
        st.[name] AS [stats_name],
        STATS_DATE(st.[object_id],st.[stats_id]) AS [stats_last_updated_date]
    FROM
        sys.objects ob
        JOIN sys.stats st
            ON  ob.[object_id] = st.[object_id]
        JOIN sys.stats_columns sc    
            ON  st.[stats_id] = sc.[stats_id]
            AND st.[object_id] = sc.[object_id]
        JOIN sys.columns co    
            ON  sc.[column_id] = co.[column_id]
            AND sc.[object_id] = co.[object_id]
        JOIN sys.types  ty    
            ON  co.[user_type_id] = ty.[user_type_id]
        JOIN sys.tables tb    
            ON  co.[object_id] = tb.[object_id]
        JOIN sys.schemas sm    
            ON  tb.[schema_id] = sm.[schema_id]
    WHERE
        st.[user_created] = 1;


例如，数据仓库中的日期列往往需要经常更新统计信息。每次有新行载入数据仓库时，就会添加新的加载日期或事务日期。这些操作会更改数据分布情况并使统计信息过时。相反地，客户表上性别列的统计信息可能永远不需要更新。假设客户间的分布固定不变，将新行添加到表变化并不会改变数据分布情况。不过，如果数据仓库只包含一种性别，而新的要求导致多种性别，则肯定需要更新性别列的统计信息。

有关更多说明，请参阅 MSDN 上的[统计信息][]。

## 实施统计信息管理

扩展数据加载过程通常是个不错的想法，可确保在加载结束时更新统计信息。当表更改其大小和/或其值分布时，数据加载最为频繁。因此，这是实施某些管理过程的合理位置。

下面提供了有关在加载过程中更新统计信息的一些指导原则：

- 确保加载的每个表至少包含一个更新的统计信息对象。这会在统计信息更新过程中更新表大小（行计数和页计数）信息。
- 将重点放在参与 JOIN、GROUP BY、ORDER BY 和 DISTINCT 子句的列上
- 考虑更频繁地更新“递增键”列（例如事务日期），因为这些值不包含在统计信息直方图中。
- 考虑较不经常更新静态分布列。
- 请记住，每个统计信息对象是连续更新的。仅实现 `UPDATE STATISTICS <TABLE_NAME>` 可能不太理想 - 尤其是对包含许多统计信息对象的宽型表而言。

> [AZURE.NOTE] 有关 [递增键] 的详细信息，请参阅 SQL Server 2014 基数估计模型白皮书。

有关更多说明，请参阅 MSDN 上的[基数估计][]。

## 示例：创建统计信息

以下示例演示如何使用各种选项来创建统计信息。用于每个列的选项取决于数据特征以及在查询中使用列的方式。

### A.使用默认选项创建单列统计信息

若要基于某个列创建统计信息，只需提供统计信息对象的名称和列的名称。

此语法使用所有默认选项。默认情况下，SQL 数据仓库在创建统计信息时对 20% 的表采样。

```
CREATE STATISTICS [statistics_name] ON [schema_name].[table_name]([column_name]);
```

例如：

```
CREATE STATISTICS col1_stats ON dbo.table1 (col1);
```

### B.通过检查每个行创建单列统计信息

20% 的默认采样率足以应付大多数情况。不过，你可以调整采样率。

若要采样整个表，请使用此语法：

```
CREATE STATISTICS [statistics_name] ON [schema_name].[table_name]([column_name]) WITH FULLSCAN;
```

例如：

```
CREATE STATISTICS col1_stats ON dbo.table1 (col1) WITH FULLSCAN;
```

### C.通过指定样本大小创建单列统计信息

或者，你可以以百分比指定样本大小：

```
CREATE STATISTICS col1_stats ON dbo.table1 (col1) WITH SAMPLE = 50 PERCENT;
```

### D.只对某些行创建单列统计信息

另一个选项是对表中的部分行创建统计信息。这称为筛选的统计信息。

例如，在计划查询大型分区表的特定分区时，你可以使用筛选的统计信息。只对分区值创建统计信息，统计信息的准确度将会改善，并因而改善查询性能。

此示例将会基于一系列的值创建统计信息。可以轻松定义这些值以匹配分区中的值范围。

```
CREATE STATISTICS stats_col1 ON table1(col1) WHERE col1 > '2000101' AND col1 < '20001231';
```

> [AZURE.NOTE] 若要让查询优化器在选择分布式查询计划时考虑使用筛选的统计信息，查询必须符合统计信息对象的定义。使用上述示例，查询的 where 子句需要指定介于 2000101 和 20001231 之间的 col1 值。

### E.使用所有选项创建单列统计信息

当然，你可以将选项组合在一起。以下示例使用自定义样本大小创建筛选的统计信息对象：

```
CREATE STATISTICS stats_col1 ON table1 (col1) WHERE col1 > '2000101' AND col1 < '20001231' WITH SAMPLE = 50 PERCENT;
```

有关完整参考，请参阅 MSDN 上的 [CREATE STATISTICS][]。

### F.创建多列统计信息

若要创建多列统计信息，只需使用上述示例，但要指定更多的列。

> [AZURE.NOTE] 用于估计查询结果中行数的直方图只适用于统计信息对象定义中所列的第一个列。

在此示例中，直方图位于 *product\_category*。跨列统计信息是根据 *product\_category* 和 *product\_sub\_category* 计算的：

```
CREATE STATISTICS stats_2cols ON table1 (product_category, product_sub_category) WHERE product_category > '2000101' AND product_category < '20001231' WITH SAMPLE = 50 PERCENT;
```

由于 *product\_category* 和 *product\_sub\_category* 之间存在关联，因此在同时访问这些列时，多列统计信息相当有用。

### G.基于表中的所有列创建统计信息

创建统计信息的方法之一是在创建表后发出 CREATE STATISTICS 命令。

    CREATE TABLE dbo.table1
    (
       col1 int
    ,  col2 int
    ,  col3 int
    )
    WITH
      (
        CLUSTERED COLUMNSTORE INDEX
      )
    ;
    
    CREATE STATISTICS stats_col1 on dbo.table1 (col1);
    CREATE STATISTICS stats_col2 on dbo.table2 (col2);
    CREATE STATISTICS stats_col3 on dbo.table3 (col3);

### H.使用存储过程基于数据库中的所有列创建统计信息

SQL 数据仓库不提供相当于 SQL Server 中 [sp\_create\_stats][] 的系统存储过程。此存储过程将基于数据库中尚不包含统计信息的每个列创建单列统计信息对象。

这可以帮助你开始进行数据库设计。你可以根据需要任意改写此存储过程。

    CREATE PROCEDURE    [dbo].[prc_sqldw_create_stats]
    (   @create_type    tinyint -- 1 default 2 Fullscan 3 Sample
    ,   @sample_pct     tinyint
    )
    AS
    
    IF @create_type NOT IN (1,2,3)
    BEGIN
        THROW 151000,'Invalid value for @stats_type parameter. Valid range 1 (default), 2 (fullscan) or 3 (sample).',1;
    END;
    
    IF @sample_pct IS NULL
    BEGIN;
        SET @sample_pct = 20;
    END;
    
    IF OBJECT_ID('tempdb..#stats_ddl') IS NOT NULL
    BEGIN;
    	DROP TABLE #stats_ddl;
    END;
    
    CREATE TABLE #stats_ddl
    WITH    (   DISTRIBUTION    = HASH([seq_nmbr])
            ,   LOCATION        = USER_DB
            )
    AS
    WITH T
    AS
    (
    SELECT      t.[name]                        AS [table_name]
    ,           s.[name]                        AS [table_schema_name]
    ,           c.[name]                        AS [column_name]
    ,           c.[column_id]                   AS [column_id]
    ,           t.[object_id]                   AS [object_id]
    ,           ROW_NUMBER()
                OVER(ORDER BY (SELECT NULL))    AS [seq_nmbr]
    FROM        sys.[tables] t
    JOIN        sys.[schemas] s         ON  t.[schema_id]       = s.[schema_id]
    JOIN        sys.[columns] c         ON  t.[object_id]       = c.[object_id]
    LEFT JOIN   sys.[stats_columns] l   ON  l.[object_id]       = c.[object_id]
                                        AND l.[column_id]       = c.[column_id]
                                        AND l.[stats_column_id] = 1
    LEFT JOIN	sys.[external_tables] e	ON	e.[object_id]		= t.[object_id]
    WHERE       l.[object_id] IS NULL
    AND			e.[object_id] IS NULL -- not an external table
    )
    SELECT  [table_schema_name]
    ,       [table_name]
    ,       [column_name]
    ,       [column_id]
    ,       [object_id]
    ,       [seq_nmbr]
    ,       CASE @create_type
            WHEN 1
            THEN    CAST('CREATE STATISTICS '+QUOTENAME('stat_'+table_schema_name+ '_' + table_name + '_'+column_name)+' ON '+QUOTENAME(table_schema_name)+'.'+QUOTENAME(table_name)+'('+QUOTENAME(column_name)+')' AS VARCHAR(8000))
            WHEN 2
            THEN    CAST('CREATE STATISTICS '+QUOTENAME('stat_'+table_schema_name+ '_' + table_name + '_'+column_name)+' ON '+QUOTENAME(table_schema_name)+'.'+QUOTENAME(table_name)+'('+QUOTENAME(column_name)+') WITH FULLSCAN' AS VARCHAR(8000))
            WHEN 3
            THEN    CAST('CREATE STATISTICS '+QUOTENAME('stat_'+table_schema_name+ '_' + table_name + '_'+column_name)+' ON '+QUOTENAME(table_schema_name)+'.'+QUOTENAME(table_name)+'('+QUOTENAME(column_name)+') WITH SAMPLE '+@sample_pct+'PERCENT' AS VARCHAR(8000))
            END AS create_stat_ddl
    FROM T
    ;
    
    DECLARE @i INT              = 1
    ,       @t INT              = (SELECT COUNT(*) FROM #stats_ddl)
    ,       @s NVARCHAR(4000)   = N''
    ;
    
    WHILE @i <= @t
    BEGIN
        SET @s=(SELECT create_stat_ddl FROM #stats_ddl WHERE seq_nmbr = @i);
    
        PRINT @s
        EXEC sp_executesql @s
        SET @i+=1;
    END
    
    DROP TABLE #stats_ddl;


若要使用此过程对表中的所有列创建统计信息，只需调用该过程即可。

```
prc_sqldw_create_stats;
```

## 示例：更新统计信息

若要更新统计信息，你可以：

1. 更新一个统计信息对象。指定要更新的统计信息对象名称。
2. 更新表中的所有统计信息对象。指定表名称，而不是一个特定的统计信息对象。


### A.更新一个特定的统计信息对象 ###
使用以下语法来更新特定的统计信息对象：

```
UPDATE STATISTICS [schema_name].[table_name]([stat_name]);
```

例如：

```
UPDATE STATISTICS [dbo].[table1] ([stats_col1]);
```

通过更新特定统计信息对象，可以减少管理统计信息所需的时间和资源。在选择要更新的最佳统计信息对象之前，需要经过一定的思考。


### B.更新表中的所有统计信息 ###
此示例演示了更新表中所有统计信息对象的一个简单方法。

```
UPDATE STATISTICS [schema_name].[table_name];
```

例如：

```
UPDATE STATISTICS dbo.table1;
```

此语句很容易使用。只要记住，这会更新表中的所有统计信息，因此执行的工作可能会超过所需的数量。如果性能不是一个考虑因素，这绝对是保证拥有最新统计信息的最简单、最全面的操作方式。

> [AZURE.NOTE] 更新表中的所有统计信息时，SQL 数据仓库将执行扫描，以针对每个统计信息进行表采样。如果表很大、包含许多列和许多统计信息，则根据需要更新各项统计信息可能比较有效率。

有关 `UPDATE STATISTICS` 过程的实现，请参阅[临时表]一文。实现方法与上述 `CREATE STATISTICS` 过程略有不同，但最终结果相同。

有关完整语法，请参阅 MSDN 上的[更新统计信息][]。

## 统计信息元数据
可以使用多个系统视图和函数来查找有关统计信息的信息。例如，使用 stats-date 函数查看上次创建或更新统计信息的时间，即可了解统计信息对象是否可能已过时。

### 统计信息的目录视图
这些系统视图提供有关统计信息的信息：

| 目录视图 | 说明 |
| :----------- | :---------- |
| [sys.columns][] | 针对每个列提供一行。 |
| [sys.objects][] | 针对数据库中的每个对象提供一行。 | |
| [sys.schemas][] | 针对数据库中的每个架构提供一行。 | |
| [sys.stats][] | 针对每个统计信息对象提供一行。 |
| [sys.stats\_columns][] | 针对统计信息对象中的每个列提供一行。链接回到 sys.columns。 |
| [sys.tables][] | 针对每个表（包括外部表）提供一行。 |
| [sys.table\_types][] | 针对每个数据类型提供一行。 |


### 统计信息的系统函数
这些系统函数适合用于处理统计信息：

| 系统函数 | 说明 |
| :-------------- | :---------- |
| [STATS\_DATE][] | 上次更新统计信息对象的日期。 |
| [DBCC SHOW\_STATISTICS][] | 提供有关统计信息对象识别的值分布的摘要级别和详细信息。 |

### 将统计信息列和函数合并成一个视图

此视图将统计信息相关的列以及 [STATS\_DATE()][] 函数的结果合并在一起。

    CREATE VIEW dbo.vstats_columns
    AS
    SELECT
            sm.[name]                           AS [schema_name]
    ,       tb.[name]                           AS [table_name]
    ,       st.[name]                           AS [stats_name]
    ,       st.[filter_definition]              AS [stats_filter_defiinition]
    ,       st.[has_filter]                     AS [stats_is_filtered]
    ,       STATS_DATE(st.[object_id],st.[stats_id])
                                                AS [stats_last_updated_date]
    ,       co.[name]                           AS [stats_column_name]
    ,       ty.[name]                           AS [column_type]
    ,       co.[max_length]                     AS [column_max_length]
    ,       co.[precision]                      AS [column_precision]
    ,       co.[scale]                          AS [column_scale]
    ,       co.[is_nullable]                    AS [column_is_nullable]
    ,       co.[collation_name]                 AS [column_collation_name]
    ,       QUOTENAME(sm.[name])+'.'+QUOTENAME(tb.[name])
                                                AS two_part_name
    ,       QUOTENAME(DB_NAME())+'.'+QUOTENAME(sm.[name])+'.'+QUOTENAME(tb.[name])
                                                AS three_part_name
    FROM    sys.objects                         AS ob
    JOIN    sys.stats           AS st ON    ob.[object_id]      = st.[object_id]
    JOIN    sys.stats_columns   AS sc ON    st.[stats_id]       = sc.[stats_id]
                                AND         st.[object_id]      = sc.[object_id]
    JOIN    sys.columns         AS co ON    sc.[column_id]      = co.[column_id]
                                AND         sc.[object_id]      = co.[object_id]
    JOIN    sys.types           AS ty ON    co.[user_type_id]   = ty.[user_type_id]
    JOIN    sys.tables          AS tb ON  co.[object_id]        = tb.[object_id]
    JOIN    sys.schemas         AS sm ON  tb.[schema_id]        = sm.[schema_id]
    WHERE   1=1
    AND     st.[user_created] = 1
    ;


## DBCC SHOW\_STATISTICS() 示例

DBCC SHOW\_STATISTICS() 显示统计信息对象中保存的数据。这些数据包括三个组成部分。

1. 标头
2. 密度向量
3. 直方图

有关统计信息的标头元数据。直方图显示统计信息对象的第一个键列中的值分布。密度向量可度量跨列相关性。SQLDW 使用统计信息对象中的任何数据来计算基数估计值。

### 显示标头、密度和直方图

此简单示例显示了统计信息对象的所有三个组成部分。

```
DBCC SHOW_STATISTICS([<schema_name>.<table_name>],<stats_name>)
```

例如：

```
DBCC SHOW_STATISTICS (dbo.table1, stats_col1);
```

### 显示 DBCC SHOW\_STATISTICS(); 的一个或多个组成部分

如果你只想要查看特定部分，请使用 `WITH` 子句并指定要查看哪些部分：

```
DBCC SHOW_STATISTICS([<schema_name>.<table_name>],<stats_name>) WITH stat_header, histogram, density_vector
```

例如：

```
DBCC SHOW_STATISTICS (dbo.table1, stats_col1) WITH histogram, density_vector
```

## DBCC SHOW\_STATISTICS() 差异
相比于 SQL Server，在 SQL 数据仓库中，DBCC SHOW\_STATISTICS() 的实现更加严格。

1. 未阐述的功能不受支持
- 不能使用 Stats\_stream
- 不能联接特定统计信息子集的结果，例如 (STAT\_HEADER JOIN DENSITY\_VECTOR)
2. 不能针对消息隐藏设置 NO\_INFOMSGS
3. 不能在统计信息名称的前后使用方括号
4. 不能使用列名来标识统计信息对象
5. 不支持自定义错误 2767


## 后续步骤
有关更多开发技巧，请参阅 [SQL 数据仓库开发概述][]。

<!--Image references-->

<!--Link references--In actual articles, you only need a single period before the slash.-->
[SQL 数据仓库开发概述]: /documentation/articles/sql-data-warehouse-overview-develop/
[临时表]: /documentation/articles/sql-data-warehouse-develop-temporary-tables/

<!-- External Links -->
[基数估计]: https://msdn.microsoft.com/zh-cn/library/dn600374.aspx
[CREATE STATISTICS]: https://msdn.microsoft.com/zh-cn/library/ms188038.aspx
[DBCC SHOW\_STATISTICS]: https://msdn.microsoft.com/zh-cn/library/ms174384.aspx
[统计信息]: https://msdn.microsoft.com/zh-cn/library/ms190397.aspx
[STATS\_DATE]: https://msdn.microsoft.com/zh-cn/library/ms190330.aspx
[sys.columns]: https://msdn.microsoft.com/zh-cn/library/ms176106.aspx
[sys.objects]: https://msdn.microsoft.com/zh-cn/library/ms190324.aspx
[sys.schemas]: https://msdn.microsoft.com/zh-cn/library/ms190324.aspx
[sys.stats]: https://msdn.microsoft.com/zh-cn/library/ms177623.aspx
[sys.stats\_columns]: https://msdn.microsoft.com/zh-cn/library/ms187340.aspx
[sys.tables]: https://msdn.microsoft.com/zh-cn/library/ms187406.aspx
[sys.table\_types]: https://msdn.microsoft.com/zh-cn/library/bb510623.aspx
[更新统计信息]: https://msdn.microsoft.com/zh-cn/library/ms187348.aspx

<!---HONumber=Mooncake_0523_2016-->