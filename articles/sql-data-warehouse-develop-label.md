<properties
   pageTitle="在 SQL 数据仓库中使用标签检测查询 | Azure"
   description="有关在开发解决方案时于 Azure SQL 数据仓库中使用标签检测查询的技巧。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="jrowlandjones"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="03/23/2016"
   wacn.date="05/23/2016"/>

# 在 SQL 数据仓库中使用标签检测查询
SQL 数据仓库支持称为查询标签的概念。在继续之前，让我们看一个示例：

	```
	SELECT *
	FROM sys.tables
	OPTION (LABEL = 'My Query Label')
	;
	```

最后一行将字符串 'My Query Label' 标记为查询。此字符串特别有用，因为可以通过 DMV 查询标签。这为我们提供了一种机制用于跟踪问题查询，以及帮助通过 ETL 运行来了解进度。

良好的命名约定确实很有帮助。例如，类似 ' PROJECT : PROCEDURE : STATEMENT : COMMENT' 的内容有助于在源代码管理的几乎所有代码中唯一标识查询。

若要按标签进行搜索，可以运行以下使用动态管理视图的查询：

	```
	SELECT  *
	FROM    sys.dm_pdw_exec_requests r
	WHERE   r.[label] = 'My Query Label'
	;
	``` 

> [AZURE.NOTE] 查询时，必须以方括号或双引号括住文字标签。标签是一个保留字，如果未分隔，将会导致错误。


## 后续步骤
有关更多开发技巧，请参阅[开发概述][]。

<!--Image references-->

<!--Article references-->
[开发概述]: /documentation/articles/sql-data-warehouse-overview-develop/

<!--MSDN references-->

<!--Other Web references-->




<!---HONumber=Mooncake_0321_2016-->