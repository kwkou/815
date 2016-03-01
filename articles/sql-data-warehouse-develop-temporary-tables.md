<properties
   pageTitle="SQL 数据仓库中的临时表 | Microsoft Azure"
   description="有关在开发解决方案时使用 Azure SQL 数据仓库中的临时表的技巧。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="twounder"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="10/19/2015"
   wacn.date="01/20/2016"/>

# SQL 数据仓库中的临时表
临时表存在于 SQL 数据仓库中的会话级别。这些表定义为本地临时表，与 SQL Server 表不同的是，可以从会话中的任何位置访问它们。

临时表在处理数据时非常有用 - 尤其是具有暂时性中间结果的转换期间。

本文重点介绍利用临时表的一些具体方法，以帮助你将代码模块化。

## 模块化代码

可以利用临时表会出现在用户会话中的任何位置这一事实，帮助将应用程序代码模块化。

我们举个可行的示例。

以下存储过程将生成更新数据库中每个列的统计信息所需的 DDL：

```
CREATE PROCEDURE    [dbo].[prc_sqldw_update_stats]
(   @update_type    tinyint -- 1 default 2 fullscan 3 sample 4 resample
	,@sample_pct     tinyint
)
AS
IF @update_type NOT IN (1,2,3,4)
BEGIN;
    THROW 151000,'Invalid value for @update_type parameter. Valid range 1 (default), 2 (fullscan), 3 (sample) or 4 (resample).',1;
END;
IF @sample_pct IS NULL
BEGIN;
    SET @sample_pct = 20;
END;
CREATE TABLE #stats_ddl
WITH
(
	DISTRIBUTION = HASH([seq_nmbr])
)
AS
(
SELECT
		sm.[name]				                                                AS [schema_name]
,		tb.[name]				                                                AS [table_name]
,		st.[name]				                                                AS [stats_name]
,		st.[has_filter]			                                                AS [stats_is_filtered]
,       ROW_NUMBER()
        OVER(ORDER BY (SELECT NULL))                                            AS [seq_nmbr]
,								 QUOTENAME(sm.[name])+'.'+QUOTENAME(tb.[name])  AS [two_part_name]
,		QUOTENAME(DB_NAME())+'.'+QUOTENAME(sm.[name])+'.'+QUOTENAME(tb.[name])  AS [three_part_name]
FROM	sys.objects			AS ob
JOIN	sys.stats			AS st	ON	ob.[object_id]		= st.[object_id]
JOIN	sys.stats_columns	AS sc	ON	st.[stats_id]		= sc.[stats_id]
									AND st.[object_id]		= sc.[object_id]
JOIN	sys.columns			AS co	ON	sc.[column_id]		= co.[column_id]
									AND	sc.[object_id]		= co.[object_id]
JOIN	sys.tables			AS tb	ON	co.[object_id]		= tb.[object_id]
JOIN	sys.schemas			AS sm	ON	tb.[schema_id]		= sm.[schema_id]
WHERE	1=1
AND		st.[user_created]   = 1
GROUP BY
		sm.[name]
,		tb.[name]
,		st.[name]
,		st.[filter_definition]
,		st.[has_filter]
)
SELECT
    CASE @update_type
    WHEN 1
    THEN 'UPDATE STATISTICS '+[two_part_name]+'('+[stats_name]+');'
    WHEN 2
    THEN 'UPDATE STATISTICS '+[two_part_name]+'('+[stats_name]+') WITH FULLSCAN;'
    WHEN 3
    THEN 'UPDATE STATISTICS '+[two_part_name]+'('+[stats_name]+') WITH SAMPLE '+CAST(@sample_pct AS VARCHAR(20))+' PERCENT;'
    WHEN 4
    THEN 'UPDATE STATISTICS '+[two_part_name]+'('+[stats_name]+') WITH RESAMPLE;'
    END AS [update_stats_ddl]
,   [seq_nmbr]
FROM    t1
;
```

请注意，实际上并未对此数据做任何处理。该程序只是生成了 DDL 并将其存储在临时表中。

但是，在 SQL 数据仓库中有可能将该临时表用于创建它的程序之外。这与 SQL Server 有所不同。事实上，可以在会话中的**任何位置**使用临时表。

这可以提高代码的模块化程度与易管理性。

```
EXEC [dbo].[prc_sqldw_update_stats] @update_type = 1, @sample_pct = NULL;
DECLARE @i INT              = 1
,       @t INT              = (SELECT COUNT(*) FROM #stats_ddl)
,       @s NVARCHAR(4000)   = N''
WHILE @i <= @t
BEGIN
    SET @s=(SELECT update_stats_ddl FROM #stats_ddl WHERE seq_nmbr = @i);
    PRINT @s
    EXEC sp_executesql @s
    SET @i+=1;
END
DROP TABLE #stats_ddl;
```

在某些情况下，还可以使用这种方法来取代内联和多语句函数。

> [AZURE.NOTE]此外，你还可以扩展此解决方案。举例来说，如果你只想要更新单个表，则只需筛选 #stats\_ddl 表

## 临时表的限制
SQL 数据仓库在实现临时表时确实会施加一些限制。

主要限制为：

- 不支持全局临时表
- 不能基于临时表创建视图


## 后续步骤
有关更多开发技巧，请参阅[开发概述][]。

<!--Image references-->

<!--Article references-->
[开发概述]: /documentation/articles/sql-data-warehouse-overview-develop

<!--MSDN references-->

<!--Other Web references-->

<!---HONumber=Mooncake_1207_2015-->
