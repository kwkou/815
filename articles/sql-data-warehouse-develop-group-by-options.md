<properties
   pageTitle="SQL 数据仓库中的 Group By 选项 | Azure"
   description="有关在开发解决方案时实现 Azure SQL 数据仓库中的 Group By 选项的技巧。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="jrowlandjones"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="03/23/2016"
   wacn.date="05/23/2016"/>

# SQL 数据仓库中的 Group By 选项

[GROUP BY] 子句可用于将数据聚合成摘要行集。它还具有一些扩展其功能的选项，但这些选项需要经过处理，因为 Azure SQL 数据仓库不直接支持这些选项。

这些选项为：
- GROUP BY with ROLLUP 
- GROUPING SETS 
- GROUP BY with CUBE

## Rollup 和 grouping sets 选项
此处最简单的选项是改为使用 `UNION ALL` 来执行汇总，而不是依赖显式语法。结果应完全相同

以下是使用 `ROLLUP` 选项的 Group By 语句示例：

```
SELECT [SalesTerritoryCountry]
,      [SalesTerritoryRegion]
,      SUM(SalesAmount)             AS TotalSalesAmount
FROM  dbo.factInternetSales s
JOIN  dbo.DimSalesTerritory t       ON s.SalesTerritoryKey       = t.SalesTerritoryKey
GROUP BY ROLLUP (
                        [SalesTerritoryCountry]
                ,       [SalesTerritoryRegion]
                )
;
```

我们已通过使用 ROLLUP 请求以下聚合：
- Country and Region 
- Country
- Grand Total

若要替换此语句，需要使用 `UNION ALL`；显式指定所需的聚合以返回相同的结果：

```
SELECT [SalesTerritoryCountry]
,      [SalesTerritoryRegion]
,      SUM(SalesAmount) AS TotalSalesAmount
FROM  dbo.factInternetSales s
JOIN  dbo.DimSalesTerritory t     ON s.SalesTerritoryKey       = t.SalesTerritoryKey
GROUP BY 
       [SalesTerritoryCountry]
,      [SalesTerritoryRegion]
UNION ALL
SELECT [SalesTerritoryCountry]
,      NULL
,      SUM(SalesAmount) AS TotalSalesAmount
FROM  dbo.factInternetSales s
JOIN  dbo.DimSalesTerritory t     ON s.SalesTerritoryKey       = t.SalesTerritoryKey
GROUP BY 
       [SalesTerritoryCountry]
UNION ALL
SELECT NULL
,      NULL
,      SUM(SalesAmount) AS TotalSalesAmount
FROM  dbo.factInternetSales s
JOIN  dbo.DimSalesTerritory t     ON s.SalesTerritoryKey       = t.SalesTerritoryKey;
```

对于 GROUPING SETS，我们需要采用相同的主体，但只创建想要查看的聚合级别的 UNION ALL 部分

## Cube 选项
可以使用 UNION ALL 方法创建 GROUP BY WITH CUBE。问题在于，代码可能很快就会变得庞大且失控。若要避免此情况，可以使用这种更高级的方法。

使用上述示例。

第一步是定义“cube”，它定义我们想要创建的所有聚合级别。请务必记下两个派生表的 CROSS JOIN。这样就会生成所有级别。剩余代码确实可以设置格式。

```
CREATE TABLE #Cube
WITH 
(   DISTRIBUTION = ROUND_ROBIN
,   LOCATION = USER_DB
)
AS
WITH GrpCube AS
(SELECT    CAST(ISNULL(Country,'NULL')+','+ISNULL(Region,'NULL') AS NVARCHAR(50)) as 'Cols'
,          CAST(ISNULL(Country+',','')+ISNULL(Region,'') AS NVARCHAR(50))  as 'GroupBy'
,          ROW_NUMBER() OVER (ORDER BY Country) as 'Seq'
FROM       ( SELECT 'SalesTerritoryCountry' as Country
             UNION ALL
             SELECT NULL
           ) c
CROSS JOIN ( SELECT 'SalesTerritoryRegion' as Region
             UNION ALL
             SELECT NULL
           ) r
)
SELECT Cols
,      CASE WHEN SUBSTRING(GroupBy,LEN(GroupBy),1) = ',' 
            THEN SUBSTRING(GroupBy,1,LEN(GroupBy)-1) 
            ELSE GroupBy 
       END AS GroupBy  --Remove Trailing Comma
,Seq
FROM GrpCube;
```

CTAS 的结果如下所示：

![][1]

第二步是指定目标表用于存储临时结果：

```
DECLARE 
 @SQL NVARCHAR(4000)
,@Columns NVARCHAR(4000)
,@GroupBy NVARCHAR(4000)
,@i INT = 1
,@nbr INT = 0
;
CREATE TABLE #Results
(
 [SalesTerritoryCountry] NVARCHAR(50)
,[SalesTerritoryRegion]  NVARCHAR(50)
,[TotalSalesAmount]      MONEY
)
WITH
(   DISTRIBUTION = ROUND_ROBIN
,   LOCATION = USER_DB
)
;
```

第三步是是循环访问执行聚合的列 cube。查询将为 #Cube 临时表中的每个行运行一次，并将结果存储在 #Results 临时表中

```
SET @nbr =(SELECT MAX(Seq) FROM #Cube);

WHILE @i<=@nbr
BEGIN
    SET @Columns = (SELECT Cols    FROM #Cube where seq = @i);
    SET @GroupBy = (SELECT GroupBy FROM #Cube where seq = @i);

    SET @SQL ='INSERT INTO #Results
              SELECT '+@Columns+'
              ,      SUM(SalesAmount) AS TotalSalesAmount
              FROM  dbo.factInternetSales s
              JOIN  dbo.DimSalesTerritory t  
              ON s.SalesTerritoryKey = t.SalesTerritoryKey
              '+CASE WHEN @GroupBy <>'' 
                     THEN 'GROUP BY '+@GroupBy ELSE '' END

    EXEC sp_executesql @SQL;
    SET @i +=1;
END
```

最后，我们只需读取 #Results 临时表即可返回结果

```
SELECT * 
FROM #Results
ORDER BY 1,2,3
;
```

通过将代码拆分成不同的部分并生成循环构造，代码将更好管理和维护。


## 后续步骤
有关更多开发技巧，请参阅[开发概述][]。

<!--Image references-->
[1]: ./media/sql-data-warehouse-develop-group-by-options/sql-data-warehouse-develop-group-by-cube.png

<!--Article references-->
[开发概述]: /documentation/articles/sql-data-warehouse-overview-develop/

<!--MSDN references-->
[GROUP BY]: https://msdn.microsoft.com/zh-cn/library/ms177673.aspx


<!--Other Web references-->


<!---HONumber=Mooncake_0321_2016-->