<properties
   pageTitle="SQL 数据仓库中的存储过程 | Azure"
   description="有关在开发解决方案时实现 Azure SQL 数据仓库中的存储过程的技巧。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="jrowlandjones"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="03/23/2016"
   wacn.date="05/23/2016"/>

# SQL 数据仓库中的存储过程 

SQL 数据仓库支持 SQL Server 中提供的许多 Transact-SQL 功能。更重要的是，我们可以利用特定的扩大功能将解决方案的性能最大化。

但是，为了保持 SQL 数据仓库的缩放性和性能，还有一些具有行为差异的功能以及其他不支持的功能。

本文介绍如何实现 SQL 数据仓库中的存储过程。

## 存储过程简介
存储过程很适合用于封装 SQL 代码；将它存储在数据仓库中数据附近。通过将代码封装成可管理的单位，存储过程帮助开发人员将其解决方案模块化；促使代码有更大的可重复使用性。每个存储过程还可接受参数，使其更具弹性。

SQL 数据仓库提供简化且流畅的存储过程实现。相比于 SQL Server，最大差异是存储过程不是预先编译的代码。在数据仓库中，我们通常不很在乎编译时间。比较重要的是在对大型数据卷操作时，正确优化存储过程代码。目标是要节省时数、分钟数和秒数，而不是毫秒数。因此，将存储过程视为 SQL 逻辑的容器更有帮助。
 
当 SQL 数据仓库执行存储过程时，SQL 语句在运行时进行解析、转换和优化。在此过程中，每个语句都转换为分布式查询。实际针对数据执行的 SQL 代码与提交的查询不同。

## 嵌套存储过程
如果存储过程调用其他存储过程或执行动态 sql，则将内部存储过程或代码调用视为嵌套。

SQL 数据仓库最多支持 8 个嵌套级别。这与 SQL Server 稍有不同。SQL Server 中的嵌套级别为 32。

最上层存储过程调用等同于嵌套级别 1

```
EXEC prc_nesting
``` 
如果存储过程还进行另一个 EXEC 调用，将使嵌套级别提升为 2 
```
CREATE PROCEDURE prc_nesting
AS
EXEC prc_nesting_2  -- This call is nest level 2
GO
EXEC prc_nesting
```
如果第二个程序接着执行某个动态 sql，将使嵌套级别提升为 3
```
CREATE PROCEDURE prc_nesting_2
AS
EXEC sp_executesql 'SELECT 'another nest level'  -- This call is nest level 2
GO
EXEC prc_nesting
```

请注意，SQL 数据仓库当前不支持 @@NESTLEVEL。你需要自行跟踪自己的嵌套级别。不太可能达到 8 个嵌套级别的限制，但如果达到，则你需要重新处理代码并将其“平整化”，使其符合此限制。

## INSERT..EXECUTE
SQL 数据仓库不允许通过 INSERT 语句使用存储过程的结果集。但是，你可以使用替代方法。

有关此方法的示例，请参阅以下有关[临时表]的文章。

## 限制

SQL 数据仓库中未实现 Transact-SQL 存储过程的某些方面。

它们具有以下特点：

- 临时存储过程
- 编号的存储过程
- 扩展的存储过程
- CLR 存储过程
- 加密选项
- 复制选项
- 表值参数
- 只读参数
- 默认参数
- 执行上下文
- return 语句

## 后续步骤
有关更多开发技巧，请参阅[开发概述][]。

<!--Image references-->

<!--Article references-->
[临时表]: /documentation/articles/sql-data-warehouse-develop-temporary-tables/
[开发概述]: /documentation/articles/sql-data-warehouse-overview-develop/

<!--MSDN references-->
[nest level]: https://msdn.microsoft.com/zh-cn/library/ms187371.aspx

<!--Other Web references-->


<!---HONumber=Mooncake_0321_2016-->