<properties
    pageTitle="用于在 Azure 中开发数据仓库的资源 | Azure"
    description="SQL 数据仓库的开发概念、设计决策、建议和编程技术。"
    services="sql-data-warehouse"
    documentationcenter="NA"
    author="jrowlandjones"
    manager="barbkess"
    editor="" />
<tags
    ms.assetid="996e3afc-c21c-4e21-b9df-997f953f6dfd"
    ms.service="sql-data-warehouse"
    ms.devlang="NA"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="data-services"
    ms.date="10/31/2016"
    wacn.date="03/20/2017"
    ms.author="jrj;barbkess" />  


# SQL 数据仓库的设计决策和编程技术
请阅读以下开发文章，以更好地了解 SQL 数据仓库的关键设计决策、建议和编程技术。

## 关键设计决策
以下文章重点介绍了在使用 SQL 数据仓库开发分布式数据仓库时，需要了解的一些关键概念和设计决策：

* [连接][connections]
* [并发][concurrency]
* [事务][transactions]
* [用户定义的架构][user-defined schemas]
* [表分布][table distribution]
* [表索引][table indexes]
* [表分区][table partitions]
* [CTAS][CTAS]
* [统计信息][statistics]

## 开发建议和编程技术
以下文章重点介绍了用于开发 SQL 数据仓库的具体编程技术、技巧和建议：

* [存储过程][stored procedures]
* [标签][labels]
* [视图][views]
* [临时表][temporary tables]
* [动态 SQL][dynamic SQL]
* [循环][looping]
* [Group By 选项][group by options]
* [变量赋值][variable assignment]

## 后续步骤
阅读开发文章后，请浏览 [Transact-SQL 参考][Transact-SQL reference]页，以了解有关 SQL 数据仓库支持的语法的更多详细信息。

<!--Image references-->

<!--Article references-->
[concurrency]: /documentation/articles/sql-data-warehouse-develop-concurrency/
[connections]: /documentation/articles/sql-data-warehouse-connect-overview/
[CTAS]: /documentation/articles/sql-data-warehouse-develop-ctas/
[dynamic SQL]: /documentation/articles/sql-data-warehouse-develop-dynamic-sql/
[group by options]: /documentation/articles/sql-data-warehouse-develop-group-by-options/
[labels]: /documentation/articles/sql-data-warehouse-develop-label/
[looping]: /documentation/articles/sql-data-warehouse-develop-loops/
[statistics]: /documentation/articles/sql-data-warehouse-tables-statistics/
[stored procedures]: /documentation/articles/sql-data-warehouse-develop-stored-procedures/
[table distribution]: /documentation/articles/sql-data-warehouse-tables-distribute/
[table indexes]: /documentation/articles/sql-data-warehouse-tables-index/
[table partitions]: /documentation/articles/sql-data-warehouse-tables-partition/
[temporary tables]: /documentation/articles/sql-data-warehouse-tables-temporary/
[transactions]: /documentation/articles/sql-data-warehouse-develop-transactions/
[user-defined schemas]: /documentation/articles/sql-data-warehouse-develop-user-defined-schemas/
[variable assignment]: /documentation/articles/sql-data-warehouse-develop-variable-assignment/
[views]: /documentation/articles/sql-data-warehouse-develop-views/
[Transact-SQL reference]: /documentation/articles/sql-data-warehouse-overview-reference/

<!--MSDN references-->
[renaming objects]: https://msdn.microsoft.com/zh-cn/library/mt631611.aspx

<!--Other Web references-->

<!---HONumber=Mooncake_0313_2017-->
<!--Update_Description:update meta properties;wording update-->