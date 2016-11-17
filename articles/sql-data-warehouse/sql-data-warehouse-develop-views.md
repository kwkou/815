<properties
   pageTitle="SQL 数据仓库中的视图 | Azure"
   description="有关在开发解决方案时使用 Azure SQL 数据仓库中的 Transact-SQL 视图的技巧。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="jrowlandjones"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="07/01/2016"
   wacn.date="08/08/2016"/>

 
# SQL 数据仓库中的视图

SQL 数据仓库中的视图特别有用。可以通过多种不同的方式使用这些视图以提升解决方案的质量。本文重点介绍几个示例，说明如何使用视图来丰富你的解决方案，以及需要注意的限制。

> [AZURE.NOTE] 本文不讨论 `CREATE VIEW` 的语法。有关这方面的参考信息，请参阅 MSDN 上的 [CREATE VIEW][] 文章。

## 体系结构摘要
一种很常见的应用模式是在加载数据时，使用 CREATE TABLE AS SELECT (CTAS) 并后接对象重命名模式来重建表。

以下示例将新的日期记录添加到日期维度。请注意，这里先创建了一个新表 DimDate_New，然后将它重命名以替换表的原始版本。


	CREATE TABLE dbo.DimDate_New
	WITH (DISTRIBUTION = ROUND_ROBIN
	, CLUSTERED INDEX (DateKey ASC)
	)
	AS
	SELECT *
	FROM   dbo.DimDate  AS prod
	UNION ALL
	SELECT *
	FROM   dbo.DimDate_stg AS stg
	;

	RENAME OBJECT DimDate TO DimDate_Old;
	RENAME OBJECT DimDate_New TO DimDate;



但是，此方法可能导致表在用户的视图中出现然后消失，并返回“表不存在”错误消息。使用视图可为用户提供一致的呈现层，同时将基础对象重命名。通过视图向用户提供对数据的访问权限，意味着用户不需要基础表的可见性。这提供了一致的用户体验，同时确保数据仓库设计人员可以改进数据模型，并在数据加载过程中使用 CTAS 充分发挥性能。

## 性能优化
视图还可用于强化表之间的性能优化联接。例如，视图可以合并冗余分布键作为联接条件的一部分，以便最大程度地减少数据移动。视图的另一个好处是强制执行特定查询或联接提示。以这种方式使用视图可确保始终以最佳方式执行联接，用户不需要记住其联接的正确构造。

## 限制
SQL 数据仓库中的视图只是元数据。因此无法使用以下选项：

- 	不提供任何架构绑定选项
- 	无法通过视图更新基表
- 	无法基于临时表创建视图
- 	不支持 EXPAND/NOEXPAND 提示
- 	SQL 数据仓库中没有索引视图


## 后续步骤
有关更多开发技巧，请参阅 [SQL 数据仓库开发概述][]。有关 `CREATE VIEW` 语法，请参阅 [CREATE VIEW][]。

<!--Image references-->

<!--Article references-->
[SQL 数据仓库开发概述]: /documentation/articles/sql-data-warehouse-overview-develop/

<!--MSDN references-->
[CREATE VIEW]: https://msdn.microsoft.com/zh-cn/library/ms187956.aspx
<!--Other Web references-->

<!---HONumber=Mooncake_0801_2016-->