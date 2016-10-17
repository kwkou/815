<properties
   pageTitle="SQL 数据仓库开发的设计决策和编码技术 | Azure"
   description="SQL 数据仓库的开发概念、设计决策、建议和编程技术。"
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
   ms.date="08/16/2016"
   wacn.date="10/17/2016"/>  


# SQL 数据仓库的设计决策和编程技术

请阅读以下开发文章，以更好地了解 SQL 数据仓库的关键设计决策、建议和编程技术。

## 关键设计决策
以下文章重点介绍了在使用 SQL 数据仓库开发分布式数据仓库时，需要了解的一些关键概念和设计决策：

- [连接][]
- [并发][]
- [事务][]
- [用户定义的架构][]
- [表分布][]
- [表索引][]
- [表分区][]
- [CTAS][]
- [统计信息][]

## 开发建议和编程技术
以下文章重点介绍了用于开发 SQL 数据仓库的具体编程技术、技巧和建议：

- [存储过程][]
- [标签][]
- [视图][]
- [临时表][]
- [动态 SQL][]
- [循环][]
- [Group By 选项][]
- [变量赋值][]

## 后续步骤
阅读开发文章后，请浏览 [Transact-SQL 参考][]页，以了解有关 SQL 数据仓库支持的语法的更多详细信息。

<!--Image references-->

<!--Article references-->
[并发]: /documentation/articles/sql-data-warehouse-develop-concurrency/
[连接]: /documentation/articles/sql-data-warehouse-connect-overview/
[CTAS]: /documentation/articles/sql-data-warehouse-develop-ctas/
[动态 SQL]: /documentation/articles/sql-data-warehouse-develop-dynamic-sql/
[Group By 选项]: /documentation/articles/sql-data-warehouse-develop-group-by-options/
[标签]: /documentation/articles/sql-data-warehouse-develop-label/
[循环]: /documentation/articles/sql-data-warehouse-develop-loops/
[统计信息]: /documentation/articles/sql-data-warehouse-tables-statistics/
[存储过程]: /documentation/articles/sql-data-warehouse-reference-tsql-statements/
[表分布]: /documentation/articles/sql-data-warehouse-tables-statistics/
[表索引]: /documentation/articles/sql-data-warehouse-tables-overview/
[表分区]: /documentation/articles/sql-data-warehouse-tables-partition/
[临时表]: /documentation/articles/sql-data-warehouse-tables-temporary/
[事务]: /documentation/articles/sql-data-warehouse-develop-transactions/
[用户定义的架构]: /documentation/articles/sql-data-warehouse-develop-user-defined-schemas/
[变量赋值]: /documentation/articles/sql-data-warehouse-develop-variable-assignment/
[视图]: /documentation/articles/sql-data-warehouse-develop-views/
[Transact-SQL 参考]: /documentation/articles/sql-data-warehouse-overview-reference/

<!--MSDN references-->
[renaming objects]: https://msdn.microsoft.com/zh-cn/library/mt631611.aspx

<!--Other Web references-->

<!---HONumber=Mooncake_1010_2016-->