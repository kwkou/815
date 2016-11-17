<properties
   pageTitle="SQL 数据仓库中的临时表 | Microsoft Azure"
   description="开始使用 Azure SQL 数据仓库中的临时表。"
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
   ms.date="06/29/2016"
   wacn.date="11/15/2016"
   ms.author="jrj;barbkess;sonyama"/>  


# SQL 数据仓库中的临时表

> [AZURE.SELECTOR]
- [概述][]
- [数据类型][]
- [分布][]
- [索引][]
- [Partition][]
- [统计信息][]
- [临时][]

临时表在处理数据时非常有用 - 尤其是具有暂时性中间结果的转换期间。临时表存在于 SQL 数据仓库中的会话级别。它们仅对创建它们的会话可见，并在该会话注销时被自动删除。临时表可以提高性能，因为其结果将写入到本地而不是远程存储。Azure SQL 数据仓库中的临时表与 Azure SQL 数据库中的临时表略有不同，它们可以从会话内的任何位置进行访问，包括存储过程内部和外部。

本文包含使用临时表的基本指导，并重点介绍会话级别临时表的原则。使用本文中的信息可以帮助你将代码模块化，从而同时提高代码的可重用性和易维护性。

## 创建临时表

只需在表名的前面添加 `#` 作为前缀，即可创建临时表。例如：


    CREATE TABLE #stats_ddl
    (
        [schema_name]		NVARCHAR(128) NOT NULL
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


此外可以使用 `CTAS` 通过完全相同的方法来创建临时表：


    CREATE TABLE #stats_ddl
    WITH
    (
        DISTRIBUTION = HASH([seq_nmbr])
    ,	HEAP
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


>[AZURE.NOTE] `CTAS` 是一个非常强大的命令，具有附加优势，可以非常有效地利用事务日志空间。


## 删除临时表

创建新会话时，应不存在任何临时表。但是，如果调用同一存储过程，它将使用相同名称创建临时表，若要确保 `CREATE TABLE` 语句成功执行，可以使用带 `DROP` 的简单预存在检查，如下面的示例中所示：


    IF OBJECT_ID('tempdb..#stats_ddl') IS NOT NULL
    BEGIN
        DROP TABLE #stats_ddl
    END


为了实现编码一致性，好的做法是对表和临时表都使用此模式。当在代码中完成临时表时，使用 `DROP TABLE` 来删除临时表也很不错。在存储过程开发中，将 drop 命令捆绑在过程结尾来确保清除这些对象非常普遍。


    DROP TABLE #stats_ddl


## 模块化代码

由于可以在用户会话中的任何位置查看临时表，可以利用这一点帮助将应用程序代码模块化。例如，下面的存储过程会将上面建议的做法组合在一起生成 DDL，该 DDL 会按统计名称更新数据库中的所有统计信息。


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
    GO


在此阶段发生的唯一操作是创建存储过程，该存储过程只使用 DDL 语句生成了临时表 #stats\_ddl。如果此存储过程在会话中运行了不止一次，它会删除已存在的 #stats\_ddl，以确保它不会失败。但是，由于存储过程的末尾没有 `DROP TABLE`，当存储过程完成后，它将保留创建的表，以便可以在存储过程以外读取它。在 SQL 数据仓库中，与其他 SQL Server 数据库不同，有可能在创建临时表的过程外部使用该临时表。可以在会话中的**任何位置**使用 SQL 数据仓库临时表。这可以提高代码的模块化程度与易管理性，如以下示例所示：


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


## 临时表的限制

SQL 数据仓库在实现临时表时确实会施加一些限制。目前，仅支持会话范围的临时表。不支持全局临时表。此外，不能在临时表上创建视图。

## 后续步骤

若要了解详细信息，请参阅有关[表概述][Overview]、[表数据类型][Data Types]、[分布表][Distribute]、[为表编制索引][Index]、[将表分区][Partition]和[维护表统计信息][Statistics]的文章。有关最佳实践的详细信息，请参阅 [SQL 数据仓库最佳实践][]。

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
[临时]: /documentation/articles/sql-data-warehouse-tables-temporary/
[SQL 数据仓库最佳实践]: /documentation/articles/sql-data-warehouse-best-practices/

<!--MSDN references-->

<!--Other Web references-->

<!---HONumber=Mooncake_0815_2016-->