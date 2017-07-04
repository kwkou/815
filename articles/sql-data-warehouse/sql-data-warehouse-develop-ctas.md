<properties
    pageTitle="SQL 数据仓库中的 Create Table As Select (CTAS) | Azure"
    description="有关在开发解决方案时使用 Azure SQL 数据仓库中的 create table as select (CTAS) 语句编写代码的技巧。"
    services="sql-data-warehouse"
    documentationcenter="NA"
    author="jrowlandjones"
    manager="jhubbard"
    editor="" />
<tags
    ms.assetid="68ac9a94-09f9-424b-b536-06a125a653bd"
    ms.service="sql-data-warehouse"
    ms.devlang="NA"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="data-services"
    ms.date="01/30/2017"
    wacn.date="03/20/2017"
    ms.author="jrj;barbkess" />  


# SQL 数据仓库中的 Create Table As Select (CTAS)
Create Table As Select (`CTAS`) 是所提供的最重要的 T-SQL 功能之一。它是根据 SELECT 语句的输出创建新表的完全并行化操作。若要创建表的副本，`CTAS` 是最简便快速的方法。本文档提供 `CTAS` 的示例和最佳实践。

## SELECT..INTO vs.CTAS
可以将 `CTAS` 视为 `SELECT..INTO` 的增压版本。

下面是简单 `SELECT..INTO` 语句的一个示例：

    SELECT *
    INTO    [dbo].[FactInternetSales_new]
    FROM    [dbo].[FactInternetSales]

在上面的示例中，`[dbo].[FactInternetSales_new]` 被创建为 ROUND\_ROBIN 分布式表，其中包含 CLUSTERED COLUMNSTORE INDEX，因为这些是 Azure SQL 数据仓库中的表默认值。

但是，`SELECT..INTO` 不允许在操作过程中更改分布方法或索引类型。此时 `CTAS` 便派上了用场。

将上述代码转换为 `CTAS` 相当直接：

    CREATE TABLE [dbo].[FactInternetSales_new]
    WITH
    (
        DISTRIBUTION = ROUND_ROBIN
    ,    CLUSTERED COLUMNSTORE INDEX
    )
    AS
    SELECT  *
    FROM    [dbo].[FactInternetSales]
    ;

通过 `CTAS`，可以更改表数据的分布以及表类型。

> [AZURE.NOTE]
> 如果只希望更改 `CTAS` 操作中的索引，并且源表是哈希分布的，那么只要维持相同的分布列和数据类型，`CTAS` 操作便将实现最佳效果。这将避免操作期间的交叉分布数据移动，从而更加高效。
> 
> 

## 使用 CTAS 复制表
`CTAS` 最常见的用途之一就是创建表副本，使你可以更改 DDL。例如，如果最初你将表创建为 `ROUND_ROBIN`，现在想要改为在列上分布的表，则可以使用 `CTAS` 来更改分布列。也可使用 `CTAS` 来更改分区、索引或列类型。

假设你在 `CREATE TABLE` 中没有指定分布列，因而使用 `ROUND_ROBIN` 分布的默认分布类型创建此表。

    CREATE TABLE FactInternetSales
    (
        ProductKey int NOT NULL,
        OrderDateKey int NOT NULL,
        DueDateKey int NOT NULL,
        ShipDateKey int NOT NULL,
        CustomerKey int NOT NULL,
        PromotionKey int NOT NULL,
        CurrencyKey int NOT NULL,
        SalesTerritoryKey int NOT NULL,
        SalesOrderNumber nvarchar(20) NOT NULL,
        SalesOrderLineNumber tinyint NOT NULL,
        RevisionNumber tinyint NOT NULL,
        OrderQuantity smallint NOT NULL,
        UnitPrice money NOT NULL,
        ExtendedAmount money NOT NULL,
        UnitPriceDiscountPct float NOT NULL,
        DiscountAmount float NOT NULL,
        ProductStandardCost money NOT NULL,
        TotalProductCost money NOT NULL,
        SalesAmount money NOT NULL,
        TaxAmt money NOT NULL,
        Freight money NOT NULL,
        CarrierTrackingNumber nvarchar(25),
        CustomerPONumber nvarchar(25)
    );

现在想要创建此表的新副本并包含群集列存储索引，以便可以使用群集列存储表的性能。还想在 ProductKey 上分布此表（因为预期此列会发生联接）并在联接 ProductKey 期间避免数据移动。最后，还希望在 OrderDateKey 上添加分区，以便通过删除旧分区来快速删除旧数据。以下是可将旧表复制到新表的 CTAS 语句。

    CREATE TABLE FactInternetSales_new
    WITH
    (
        CLUSTERED COLUMNSTORE INDEX,
        DISTRIBUTION = HASH(ProductKey),
        PARTITION
        (
            OrderDateKey RANGE RIGHT FOR VALUES
            (
            20000101,20010101,20020101,20030101,20040101,20050101,20060101,20070101,20080101,20090101,
            20100101,20110101,20120101,20130101,20140101,20150101,20160101,20170101,20180101,20190101,
            20200101,20210101,20220101,20230101,20240101,20250101,20260101,20270101,20280101,20290101
            )
        )
    )
    AS SELECT * FROM FactInternetSales;

最后，你可以重命名表以切换到新表，然后删除旧表。

    RENAME OBJECT FactInternetSales TO FactInternetSales_old;
    RENAME OBJECT FactInternetSales_new TO FactInternetSales;

	DROP TABLE FactInternetSales_old;

> [AZURE.NOTE]
> Azure SQL 数据仓库尚不支持自动创建或自动更新统计信息。为了从查询中获得最佳性能，首次加载数据或者在数据发生重大更改之后，创建所有表的所有列统计信息非常重要。有关统计信息的详细说明，请参阅开发主题组中的[统计信息][Statistics]主题。
> 
> 

## <a name="using-ctas-to-work-around-unsupported-features"></a>使用 CTAS 解决不支持的功能
`CTAS` 还可用于解决以下多种不支持的功能。这往往是一种经过证实的双赢局面，因为代码不但能够兼容，而且通常可以在 SQL 数据仓库中更快速执行。这是完全并行化设计的结果。可以使用 CTAS 解决的方案包括：

* UPDATE 中的 ANSI JOIN
* DELETE 中的 ANSI JOIN
* MERGE 语句

> [AZURE.NOTE]
> 尽量考虑“CTAS 优先”。如果你认为可以使用 `CTAS` 解决问题，则它往往就是最佳的解决方法，即使你要因此写入更多的数据。
> 
> 

## <a id="ansi-join-replacement-for-update-statements"></a>替换 Update 语句的 ANSI Join

你可能有一个复杂的更新使用 ANSI 联接语法来执行 UPDATE 或 DELETE，以将两个以上的表联接在一起。

假设必须更新此表：


    CREATE TABLE [dbo].[AnnualCategorySales]
    (    [EnglishProductCategoryName]    NVARCHAR(50)    NOT NULL
    ,    [CalendarYear]                    SMALLINT        NOT NULL
    ,    [TotalSalesAmount]                MONEY            NOT NULL
    )
    WITH
    (
        DISTRIBUTION = ROUND_ROBIN
    )
    ;

原始查询看起来可能类似于：

    UPDATE    acs
    SET        [TotalSalesAmount] = [fis].[TotalSalesAmount]
    FROM    [dbo].[AnnualCategorySales]     AS acs
    JOIN    (
            SELECT    [EnglishProductCategoryName]
            ,        [CalendarYear]
            ,        SUM([SalesAmount])                AS [TotalSalesAmount]
            FROM    [dbo].[FactInternetSales]        AS s
            JOIN    [dbo].[DimDate]                    AS d    ON s.[OrderDateKey]                = d.[DateKey]
            JOIN    [dbo].[DimProduct]                AS p    ON s.[ProductKey]                = p.[ProductKey]
            JOIN    [dbo].[DimProductSubCategory]    AS u    ON p.[ProductSubcategoryKey]    = u.[ProductSubcategoryKey]
            JOIN    [dbo].[DimProductCategory]        AS c    ON u.[ProductCategoryKey]        = c.[ProductCategoryKey]
            WHERE     [CalendarYear] = 2004
            GROUP BY
                    [EnglishProductCategoryName]
            ,        [CalendarYear]
            ) AS fis
    ON    [acs].[EnglishProductCategoryName]    = [fis].[EnglishProductCategoryName]
    AND    [acs].[CalendarYear]                = [fis].[CalendarYear]
    ;

由于 SQL 数据仓库不支持在 `UPDATE` 语句的 `FROM` 子句中使用 ANSI Join，因此无法在不稍微更改此代码的情况下将它复制过去。

你可以使用 `CTAS` 和隐式联接的组合来替换此代码：

    -- Create an interim table
    CREATE TABLE CTAS_acs
    WITH (DISTRIBUTION = ROUND_ROBIN)
    AS
    SELECT    ISNULL(CAST([EnglishProductCategoryName] AS NVARCHAR(50)),0)    AS [EnglishProductCategoryName]
    ,        ISNULL(CAST([CalendarYear] AS SMALLINT),0)                         AS [CalendarYear]
    ,        ISNULL(CAST(SUM([SalesAmount]) AS MONEY),0)                        AS [TotalSalesAmount]
    FROM    [dbo].[FactInternetSales]        AS s
    JOIN    [dbo].[DimDate]                    AS d    ON s.[OrderDateKey]                = d.[DateKey]
    JOIN    [dbo].[DimProduct]                AS p    ON s.[ProductKey]                = p.[ProductKey]
    JOIN    [dbo].[DimProductSubCategory]    AS u    ON p.[ProductSubcategoryKey]    = u.[ProductSubcategoryKey]
    JOIN    [dbo].[DimProductCategory]        AS c    ON u.[ProductCategoryKey]        = c.[ProductCategoryKey]
    WHERE     [CalendarYear] = 2004
    GROUP BY
            [EnglishProductCategoryName]
    ,        [CalendarYear]
    ;

    -- Use an implicit join to perform the update
    UPDATE  AnnualCategorySales
    SET     AnnualCategorySales.TotalSalesAmount = CTAS_ACS.TotalSalesAmount
    FROM    CTAS_acs
    WHERE   CTAS_acs.[EnglishProductCategoryName] = AnnualCategorySales.[EnglishProductCategoryName]
    AND     CTAS_acs.[CalendarYear]               = AnnualCategorySales.[CalendarYear]
    ;

    --Drop the interim table
    DROP TABLE CTAS_acs
    ;


## <a id="ansi-join-replacement-for-delete-statements"></a> 替换 Delete 语句的 ANSI Join
有时，删除数据的最佳方法是使用 `CTAS`。除了删除数据以外，可以只选择想要保留的数据。这对于使用 ANSI 联接语法的 `DELETE` 语句尤其适用，因为 SQL 数据仓库不支持在 `DELETE` 语句的 `FROM` 子句中使用 ANSI Join。

转换后的 DELETE 语句示例如下所示：

    CREATE TABLE dbo.DimProduct_upsert
    WITH
    (   Distribution=HASH(ProductKey)
    ,   CLUSTERED INDEX (ProductKey)
    )
    AS -- Select Data you wish to keep
    SELECT     p.ProductKey
    ,          p.EnglishProductName
    ,          p.Color
    FROM       dbo.DimProduct p
    RIGHT JOIN dbo.stg_DimProduct s
    ON         p.ProductKey = s.ProductKey
    ;

    RENAME OBJECT dbo.DimProduct        TO DimProduct_old;
    RENAME OBJECT dbo.DimProduct_upsert TO DimProduct;

## <a id="replace-merge-statements"></a>替换 merge 语句
使用 `CTAS` 至少可以部分替换 Merge 语句。可以将 `INSERT` 和 `UPDATE` 合并成单个语句。任何已删除的记录都需要在第二个语句中隔离。

`UPSERT` 的示例如下：

    CREATE TABLE dbo.[DimProduct_upsert]
    WITH
    (   DISTRIBUTION = HASH([ProductKey])
    ,   CLUSTERED INDEX ([ProductKey])
    )
    AS
    -- New rows and new versions of rows
    SELECT      s.[ProductKey]
    ,           s.[EnglishProductName]
    ,           s.[Color]
    FROM      dbo.[stg_DimProduct] AS s
    UNION ALL  
    -- Keep rows that are not being touched
    SELECT      p.[ProductKey]
    ,           p.[EnglishProductName]
    ,           p.[Color]
    FROM      dbo.[DimProduct] AS p
    WHERE NOT EXISTS
    (   SELECT  *
        FROM    [dbo].[stg_DimProduct] s
        WHERE   s.[ProductKey] = p.[ProductKey]
    )
    ;

    RENAME OBJECT dbo.[DimProduct]          TO [DimProduct_old];
    RENAME OBJECT dbo.[DimpProduct_upsert]  TO [DimProduct];

## CTAS 建议：显式声明数据类型和输出是否可为 null
迁移代码时，可能会遇到这种类型的编码模式：

    DECLARE @d decimal(7,2) = 85.455
    ,       @f float(24)    = 85.455

    CREATE TABLE result
    (result DECIMAL(7,2) NOT NULL
    )
    WITH (DISTRIBUTION = ROUND_ROBIN)

    INSERT INTO result
    SELECT @d*@f
    ;

你可能自然而然地认为应该将此代码迁移到 CTAS。这是对的。但是，这里有一个隐含的问题。

以下代码不会生成相同的结果：

    DECLARE @d decimal(7,2) = 85.455
    ,       @f float(24)    = 85.455
    ;

    CREATE TABLE ctas_r
    WITH (DISTRIBUTION = ROUND_ROBIN)
    AS
    SELECT @d*@f as result
    ;

请注意，列“result”沿用表达式的数据类型和可为 null 的值。如果不小心处理，可能会导致值存在细微的差异。

请尝试使用以下内容作为示例：

    SELECT result,result*@d
    from result
    ;

    SELECT result,result*@d
    from ctas_r
    ;


为结果存储的值不相同。因为结果列中保留的值用于其他表达式，错误变得更加严重。

![][1]

这对于数据迁移特别重要。尽管第二个查询看起来更准确，但仍有一个问题。与源系统相比，此数据有所不同，会在迁移中造成完整性问题。这是“错误”答案其实是正确答案的极少见情况之一！

我们看到这两个结果之间有差异，追根究底的原因是隐式类型转型。在第一个示例中，表定义了列定义。插入行后，将发生隐式类型转换。在第二个示例中，没有隐式类型转换，因为表达式定义了列的数据类型。请注意，第二个示例中的列已定义为可为 Null 的列，而在第一个示例中还没有定义。在第一个示例中创建表时，尚未显式定义列可为 null。在第二个示例中，它只留给了表达式，默认情况下，这会导致 NULL 定义。

若要解决这些问题，必须在 `CTAS` 语句的 `SELECT` 部分中明确设置类型转换和可为 null 属性。无法在创建表的部分中设置这些属性。

以下示例演示如何修复代码：

    DECLARE @d decimal(7,2) = 85.455
    ,       @f float(24)    = 85.455

    CREATE TABLE ctas_r
    WITH (DISTRIBUTION = ROUND_ROBIN)
    AS
    SELECT ISNULL(CAST(@d*@f AS DECIMAL(7,2)),0) as result


注意以下事项：

* CAST 或 CONVERT 可能已被使用
* 使用 ISNULL 而非 COALESCE 来强制可为 NULL 属性
* ISNULL 是最外层的函数
* ISNULL 的第二个部分是常量，即 0

> [AZURE.NOTE]
> 若要正确设置可为 null 属性，必须使用 `ISNULL` 而不是 `COALESCE`。`COALESCE` 不是确定性的函数 ，因此表达式的结果始终可为 NULL。`ISNULL` 则不同。它是确定性的。因此当 `ISNULL` 函数的第二个部分是常量或文本时，结果值将是 NOT NULL。
> 
> 

此技巧不仅可用于确保计算的完整性，它对表分区切换也很重要。假设已将此表定义为事实：

    CREATE TABLE [dbo].[Sales]
    (
        [date]      INT     NOT NULL
    ,   [product]   INT     NOT NULL
    ,   [store]     INT     NOT NULL
    ,   [quantity]  INT     NOT NULL
    ,   [price]     MONEY   NOT NULL
    ,   [amount]    MONEY   NOT NULL
    )
    WITH
    (   DISTRIBUTION = HASH([product])
    ,   PARTITION   (   [date] RANGE RIGHT FOR VALUES
                        (20000101,20010101,20020101
                        ,20030101,20040101,20050101
                        )
                    )
    )
    ;

然而，值字段是计算的表达式，而不是源数据的一部分。

若要创建分区数据集，可能需要执行以下操作：

    CREATE TABLE [dbo].[Sales_in]
    WITH    
    (   DISTRIBUTION = HASH([product])
    ,   PARTITION   (   [date] RANGE RIGHT FOR VALUES
                        (20000101,20010101
                        )
                    )
    )
    AS
    SELECT
        [date]    
    ,   [product]
    ,   [store]
    ,   [quantity]
    ,   [price]   
    ,   [quantity]*[price]  AS [amount]
    FROM [stg].[source]
    OPTION (LABEL = 'CTAS : Partition IN table : Create')
    ;

该查询会顺利运行。但是，尝试执行分区切换时，将会出现问题。表定义不匹配。若要使表定义匹配，需要修改 CTAS。

    CREATE TABLE [dbo].[Sales_in]
    WITH    
    (   DISTRIBUTION = HASH([product])
    ,   PARTITION   (   [date] RANGE RIGHT FOR VALUES
                        (20000101,20010101
                        )
                    )
    )
    AS
    SELECT
        [date]    
    ,   [product]
    ,   [store]
    ,   [quantity]
    ,   [price]   
    ,   ISNULL(CAST([quantity]*[price] AS MONEY),0) AS [amount]
    FROM [stg].[source]
    OPTION (LABEL = 'CTAS : Partition IN table : Create');

因此，可以看出，保持类型一致性并维护 CTAS 上的可为 null 属性是良好的工程最佳实践。这有助于维护计算的完整性，而且还可确保分区切换能够实现。

有关使用 [CTAS][CTAS] 的详细信息，请参阅 MSDN。CTAS 是 Azure SQL 数据仓库中最重要的语句之一。请确保全面了解该语句。

## 后续步骤
有关更多开发技巧，请参阅[开发概述][development overview]。

<!--Image references-->
[1]: ./media/sql-data-warehouse-develop-ctas/ctas-results.png

<!--Article references-->
[development overview]: /documentation/articles/sql-data-warehouse-overview-develop/
[Statistics]: /documentation/articles/sql-data-warehouse-tables-statistics/

<!--MSDN references-->
[CTAS]: https://msdn.microsoft.com/zh-cn/library/mt204041.aspx

<!--Other Web references-->

<!---HONumber=Mooncake_0313_2017-->
<!--Update_Description:update meta properties;wroding update;add Select into vs CATS compare-->