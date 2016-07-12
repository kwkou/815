<properties
   pageTitle="将 SQL 代码迁移到 SQL 数据仓库 | Azure"
   description="有关在开发解决方案时将 SQL 代码迁移到 Azure SQL 数据仓库的技巧。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="lodipalm"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="05/14/2016"
   wacn.date="06/20/2016"/>

# 将 SQL 代码迁移到 SQL 数据仓库

将代码从其他数据库迁移到 SQL 数据仓库时，你很可能需要更改代码库。某些 SQL 数据仓库功能设计为以分布方式运行，因此可以大幅改善性能。但是，为了保持性能和缩放性，某些功能还无法使用。

## 常见的 T-SQL 限制

以下列表汇总了最常用但却不受 Azure SQL 数据仓库支持的功能。单击这些链接可以转到解决不支持功能的方法：

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
- [通用表表达式](#Common-table-expressions)
- [递归通用表表达式 (CTE)](#Recursive-common-table-expressions-(CTE)
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

## 支持的 CTE 功能

SQL 数据仓库支持部分通用表表达式 (CTE)。目前支持以下 CTE 功能：

- 可以在 SELECT 语句中指定 CTE。
- 可以在 CREATE VIEW 语句中指定 CTE。
- 可以在 CREATE TABLE AS SELECT (CTAS) 语句中指定 CTE。
- 可以在 CREATE REMOTE TABLE AS SELECT (CRTAS) 语句中指定 CTE。
- 可以在 CREATE EXTERNAL TABLE AS SELECT (CETAS) 语句中指定 CTE。
- 可以从 CTE 引用远程表。
- 可以从 CTE 引用外部表。
- 可以在 CTE 中定义多个 CTE 查询定义。

## CTE 限制

在 SQL 数据仓库中，通用表表达式存在一些限制，其中包括：

- CTE 必须后接单个 SELECT 语句。不支持 INSERT、UPDATE、DELETE 和 MERGE 语句。
- 包含自身引用的通用表表达式（即递归通用表表达式）不受支持（参见以下部分）。
- 不允许在 CTE 中指定多个 WITH 子句。例如，如果 CTE\_query\_definition 包含子查询，则该子查询不能包含用于定义其他 CTE 的嵌套式 WITH 子句。
- ORDER BY 子句不能用于 CTE\_query\_definition，除非指定了 TOP 子句。
- 将 CTE 用在属于批处理一部分的语句中时，该 CTE 之前的语句必须以分号结尾。
- 用在通过 sp\_prepare 进行准备的语句中时，CTE 的表现方式与 PDW 中的其他 SELECT 语句相同。但是，如果 CTE 用作 sp\_prepare 所准备的 CETAS 的一部分，则因为针对 sp\_prepare 实现绑定的方式不同，CTE 的行为将与 SQL Server 和其他 PDW 语句不同。如果引用 CTE 的 SELECT 使用了不在 CTE 中的错误列，sp\_prepare 将会通过且不检测错误，但在 sp\_execute 期间将引发错误。

## 递归 CTE

SQL 数据仓库不支持递归 CTE。递归 CTE 的迁移过程可能有点复杂，最好是将其分为多个步骤来进行。通常你可以使用循环，并在循环访问递归的临时查询时填充临时表。填充临时表之后，你可以使用单个结果集返回数据。类似的方法已用于解决[结合 rollup / cube / grouping sets 选项的 Group By 子句][]一文中所述的 `GROUP BY WITH CUBE`。

### 系统函数

还有一些不支持的系统函数。在数据仓库中，你可能经常发现使用了下面这些主要函数：

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
[Update 中的 ANSI Join]: /documentation/articles/sql-data-warehouse-develop-ctas/
[Delete 中的 ANSI Join]: /documentation/articles/sql-data-warehouse-develop-ctas/
[Merge 语句]: /documentation/articles/sql-data-warehouse-develop-ctas/
[INSERT..EXEC]: /documentation/articles/sql-data-warehouse-develop-temporary-tables/

[游标]: /documentation/articles/sql-data-warehouse-develop-loops/
[SELECT..INTO]: /documentation/articles/sql-data-warehouse-develop-ctas/
[结合 rollup / cube / grouping sets 选项的 Group By 子句]: /documentation/articles/sql-data-warehouse-develop-group-by-options/
[嵌套级别超过 8]: /documentation/articles/sql-data-warehouse-develop-transactions/
[通过视图更新]: /documentation/articles/sql-data-warehouse-develop-views/
[使用 select 分配变量]: /documentation/articles/sql-data-warehouse-develop-variable-assignment/
[动态 SQL 字符串没有 MAX 数据类型]: /documentation/articles/sql-data-warehouse-develop-dynamic-sql/
[开发概述]: /documentation/articles/sql-data-warehouse-overview-develop/

<!--MSDN references-->

<!--Other Web references-->

<!---HONumber=Mooncake_0613_2016-->