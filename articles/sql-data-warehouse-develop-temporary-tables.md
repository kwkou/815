<properties
   pageTitle="SQL 数据仓库中的临时表 | Azure"
   description="有关在开发解决方案时使用 Azure SQL 数据仓库中的临时表的技巧。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="jrowlandjones"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="03/23/2016"
   wacn.date="05/23/2016"/>

# SQL 数据仓库中的临时表
临时表在处理数据时非常有用 - 尤其是具有暂时性中间结果的转换期间。临时表存在于 SQL 数据仓库中的会话级别。但这些表仍定义为本地临时表，与 SQL Server 表不同的是，可以从会话中的任何位置访问它们。

本文包含一些使用临时表的基本指导，并突出显示会话级别临时表的原则。使用此信息可以帮助你将代码模块化。代码模块化对于方便维护和代码重用至关重要。

## 创建临时表
创建临时表简单明了。需要执行的就只是给表名称添加以 # 开头的前缀，如以下示例所示：

```
CREATE TABLE #stats_ddl
(
	[schema_name]			NVARCHAR(128) NOT NULL
,	[table_name]            NVARCHAR(128) NOT NULL
,	[stats_name]            NVARCHAR(128) NOT NULL
,	[stats_is_filtered]     BIT           NOT NULL
,	[seq_nmbr]              BIGINT        NOT NULL
,	[two_part_name]         NVARCHAR(260) NOT NULL
,	[three_part_name]       NVARCHAR(400) NOT NULL
)
WITH
(
	DISTRIBUTION = HASH([seq_nmbr])
,	HEAP
)
```

此外可以使用 `CTAS` 通过相同的方法来创建临时表。

```
CREATE TABLE #stats_ddl
WITH
(
	DISTRIBUTION = HASH([seq_nmbr])
,	HEAP
)
AS
(
SELECT
		sm.[name] AS [schema_name]
,		tb.[name] AS [table_name]
,		st.[name] AS [stats_name]
,		st.[has_filter] AS [stats_is_filtered]
,       ROW_NUMBER()
        OVER(ORDER BY (SELECT NULL)) AS [seq_nmbr]
,								 QUOTENAME(sm.[name])+'.'+QUOTENAME(tb.[name]) AS [two_part_name]
,		QUOTENAME(DB_NAME())+'.'+QUOTENAME(sm.[name])+'.'+QUOTENAME(tb.[name]) AS [three_part_name]
FROM	sys.objects AS ob
JOIN	sys.stats AS st ON ob.[object_id] = st.[object_id]
JOIN	sys.stats_columns AS sc ON st.[stats_id] = sc.[stats_id]
									AND st.[object_id] = sc.[object_id]
JOIN	sys.columns AS co ON sc.[column_id] = co.[column_id]
									AND	sc.[object_id] = co.[object_id]
JOIN	sys.tables AS tb ON	co.[object_id] = tb.[object_id]
JOIN	sys.schemas	 AS sm ON tb.[schema_id] = sm.[schema_id]
WHERE	1=1
AND		st.[user_created] = 1
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

>[AZURE.NOTE] `CTAS` 是一个非常强大的命令，具有附加优势，可以非常有效地利用事务日志空间。


## 删除临时表

若要确保 `CREATE TABLE` 语句成功，则务必要确保表已经不存在于会话中。使用以下模式，通过简单的预存在检查即可处理：

```
IF OBJECT_ID('tempdb..#stats_ddl') IS NOT NULL
BEGIN
	DROP TABLE #stats_ddl
END
```

> [AZURE.NOTE] 为了实现编码一致性，建议对表和临时表都使用此模式。

当在代码中完成临时表时，使用 `DROP TABLE` 来删除临时表也很不错。

```
DROP TABLE #stats_ddl
```

在存储过程开发中，将 drop 命令捆绑在过程结尾来确保清除这些对象非常普遍。

## 模块化代码

确实可以利用临时表会出现在用户会话中的任何位置这一事实，帮助将应用程序代码模块化。

我们举个可行的示例。

下面的存储过程汇集了上面提到的示例。代码可用于生成更新数据库中每个列的统计信息所需的 DDL：

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

IF OBJECT_ID('tempdb..#stats_ddl') IS NOT NULL
BEGIN
	DROP TABLE #stats_ddl
END

CREATE TABLE #stats_ddl
WITH
(
	DISTRIBUTION = HASH([seq_nmbr])
)
AS
(
SELECT
		sm.[name] AS [schema_name]
,		tb.[name] AS [table_name]
,		st.[name] AS [stats_name]
,		st.[has_filter] AS [stats_is_filtered]
,       ROW_NUMBER()
        OVER(ORDER BY (SELECT NULL)) AS [seq_nmbr]
,								 QUOTENAME(sm.[name])+'.'+QUOTENAME(tb.[name]) AS [two_part_name]
,		QUOTENAME(DB_NAME())+'.'+QUOTENAME(sm.[name])+'.'+QUOTENAME(tb.[name]) AS [three_part_name]
FROM	sys.objects AS ob
JOIN	sys.stats AS st	ON ob.[object_id] = st.[object_id]
JOIN	sys.stats_columns AS sc	ON st.[stats_id] = sc.[stats_id]
									AND st.[object_id] = sc.[object_id]
JOIN	sys.columns			AS co	ON	sc.[column_id] = co.[column_id]
									AND	sc.[object_id] = co.[object_id]
JOIN	sys.tables			AS tb	ON	co.[object_id] = tb.[object_id]
JOIN	sys.schemas			AS sm	ON	tb.[schema_id] = sm.[schema_id]
WHERE	1=1
AND		st.[user_created] = 1
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
GO
```

在此阶段表中未发生任何操作。该过程只生成更新统计信息所需的 DDL，并将该代码存储在临时表。

但还请注意，存储过程不在结尾处包括 `DROP TABLE` 命令。但我们将预存在检查包含在存储过程以使代码更可靠且可重复；确保我们的 `CTAS` 将不会由于会话中存在重复对象而失败。

接下来是令人感兴趣的部分！

在 SQL 数据仓库中有可能将该临时表用于创建它的程序之外。这与 SQL Server 有所不同。事实上，可以在会话中的**任何位置**使用临时表。

这可以提高代码的模块化程度与易管理性。看看下面的示例：

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

生成的代码更为紧凑。

在某些情况下，还可以使用这种方法来取代内联和多语句函数。

> [AZURE.NOTE] 此外，你还可以扩展此解决方案。举例来说，如果你只想要更新单个表，则只需筛选 #stats\_ddl 表

## 临时表的限制
SQL 数据仓库在实现临时表时确实会施加一些限制。

主要限制为：

- 不支持全局临时表
- 不能基于临时表创建视图

## 后续步骤
有关更多开发技巧，请参阅[开发概述][]。

<!--Image references-->

<!--Article references-->
[开发概述]: /documentation/articles/sql-data-warehouse-overview-develop/

<!--MSDN references-->

<!--Other Web references-->

<!---HONumber=Mooncake_0405_2016-->