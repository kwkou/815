<properties
   pageTitle="将 SQL 代码迁移到 SQL 数据仓库 | Windows Azure"
   description="有关在开发解决方案时将 SQL 代码迁移到 Azure SQL 数据仓库的技巧。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="lodipalm"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="09/22/2015"
   wacn.date="01/20/2016"/>

# 将 SQL 代码迁移到 SQL 数据仓库

为了确保代码符合 SQL 数据仓库的要求，你很可能需要更改代码基。某些 SQL 数据仓库功能设计为直接以分布方式运行，因此可以大幅改善性能。但是，为了保持性能和缩放性，某些功能还无法使用。

## Transact-SQL 代码更改

以下列表汇总了 Azure SQL 数据仓库不支持的主要功能。单击这些链接可以转到解决不支持功能的方法：

- [Update 中的 ANSI Join][]
- [Delete 中的 ANSI Join][]
- [Merge 语句][]
- 跨数据库联接
- [游标][]
- [SELECT..INTO][]
- [INSERT..EXEC][]
- output 子句
- 内联用户定义的函数
- 多语句函数
- [递归通用表表达式 (CTE)](#Recursive-common-table-expressions-(CTE)
- [通过 CTE 更新](#Updates-through-CTEs)
- CLR 函数和过程
- $partition 函数
- 表变量
- 表值参数
- 分布式事务
- 提交/回滚工作
- 保存事务
- 执行上下文 (EXECUTE AS)
- [结合 rollup / cube / grouping sets 选项的 Group By 子句][]
- [嵌套级别超过 8][]
- [通过视图更新][]
- [使用 select 分配变量][]
- [动态 SQL 字符串没有 MAX 数据类型][]

幸好可以解决其中的大多数限制。上面提到的相关开发文章已提供了说明。

### 递归通用表表达式 (CTE)

这是一种复杂的情形，目前没有快速解决办法。CTE 需要细分并逐步处理。通常你可以使用相当复杂的循环；当你循环访问递归的临时查询时，将会填充临时表。填充临时表之后，你可以使用单个结果集返回数据。类似的方法已用于解决[结合 rollup / cube / grouping sets 选项的 Group By 子句][]一文中所述的 `GROUP BY WITH CUBE`。

### 通过 CTE 更新

如果 CTE 不是递归性的，你可以重新编写查询来使用子查询。对于递归 CTE，需要如前所述先创建结果集，然后将最终结果集加入目标表并执行更新。

### 系统函数

还有一些不支持的系统函数。在数据仓库中，你可能经常发现使用了下面这些主要函数：

- NEWID()
- NEWSEQUENTIALID()
- @@NESTLEVEL()
- @@IDENTITY()
- @@ROWCOUNT()
- ROWCOUNT\_BIG
- ERROR\_LINE()

同样，其中的许多问题都可以得到解决。

例如，以下代码是检索 @@ROWCOUNT 信息的替代解决方法：

```
SELECT  SUM(row_count) AS row_count 
FROM    sys.dm_pdw_sql_requests 
WHERE   row_count <> -1 
AND     request_id IN 
                    (   SELECT TOP 1    request_id 
                        FROM            sys.dm_pdw_exec_requests 
                        WHERE           session_id = SESSION_ID() 
                        ORDER BY end_time DESC
                    )
;
``` 

## 后续步骤
有关开发代码的建议，请参阅[开发概述][]。

<!--Image references-->

<!--Article references-->
[Update 中的 ANSI Join]: /documentation/articles/sql-data-warehouse-develop-ctas
[Delete 中的 ANSI Join]: /documentation/articles/sql-data-warehouse-develop-ctas
[Merge 语句]: /documentation/articles/sql-data-warehouse-develop-ctas
[INSERT..EXEC]: /documentation/articles/sql-data-warehouse-develop-temporary-tables

[游标]: /documentation/articles/sql-data-warehouse-develop-loops
[SELECT..INTO]: /documentation/articles/sql-data-warehouse-develop-ctas
[结合 rollup / cube / grouping sets 选项的 Group By 子句]: /documentation/articles/sql-data-warehouse-develop-group-by-options
[嵌套级别超过 8]: /documentation/articles/sql-data-warehouse-develop-transactions
[通过视图更新]: /documentation/articles/sql-data-warehouse-develop-views
[使用 select 分配变量]: /documentation/articles/sql-data-warehouse-develop-variable-assignment
[动态 SQL 字符串没有 MAX 数据类型]: /documentation/articles/sql-data-warehouse-develop-dynamic-sql
[开发概述]: /documentation/articles/sql-data-warehouse-overview-develop

<!--MSDN references-->

<!--Other Web references-->

<!---HONumber=Mooncake_1207_2015-->
