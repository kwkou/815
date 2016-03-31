<properties
   pageTitle="SQL 数据仓库中的事务 | Azure"
   description="有关在开发解决方案时实现 Azure SQL 数据仓库中的事务的技巧。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="jrowlandjones"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="01/07/2016"
   wacn.date="03/28/2016"/>

# SQL 数据仓库中的事务

如你所料，SQL 数据仓库为所有事务属性提供支持。但是，为了确保 SQL 数据仓库的性能维持在一定的程度，相比于 SQL Server，其某些功能会受到限制。本文将突出两者的差异，并列出其他信息。

## 事务隔离级别
SQL 数据仓库实现 ACID 事务。但是，事务支持的隔离仅限于 `READ UNCOMMITTED`，这无法更改。你可以实现许多编码方法，以避免脏读数据（如果你对此有所考虑的话）。大多数流行方法利用 CTAS 和表分区切换（通常称为滑动窗口模式），以防止用户查询仍正准备的数据。预先筛选数据的视图也是常用的方法。

## 事务状态
SQL 数据仓库使用 XACT\_STATE() 函数（采用值 -2）来报告失败的事务。这表示事务已失败并标记为仅可回滚

> [AZURE.NOTE] XACT\_STATE 函数使用 -2 表示失败的事务，以代表 SQL Server 中不同的行为。SQL Server 使用值 -1 来代表无法提交的事务。SQL Server 可以容忍事务内的某些错误，而无需将其标记为无法提交。例如，SELECT 1/0 导致错误，但不强制事务进入无法提交状态。SQL Server 还允许读取无法提交的事务。但是，在 SQLDW 中，情况并非如此。如果 SQLDW 事务内发生错误，它就会自动进入 -2 状态：包括 SELECT 1/0 错误。因此，必须查看应用程序代码是否使用 XACT\_STATE()。

在 SQL Server 中，你可能会看到如下所示的代码片段：

```
BEGIN TRAN
    BEGIN TRY
        DECLARE @i INT;
        SET     @i = CONVERT(int,'ABC');
    END TRY
    BEGIN CATCH

        DECLARE @xact smallint = XACT_STATE();

        SELECT  ERROR_NUMBER()    AS ErrNumber
        ,       ERROR_SEVERITY()  AS ErrSeverity
        ,       ERROR_STATE()     AS ErrState
        ,       ERROR_PROCEDURE() AS ErrProcedure
        ,       ERROR_MESSAGE()   AS ErrMessage
        ;

        ROLLBACK TRAN;

    END CATCH;
```

请注意，`SELECT` 语句出现在 `ROLLBACK` 语句的前面。另外请注意，`@xact` 变量的设置使用 DECLARE 而非 `SELECT`。

在 SQL 数据仓库中，代码需如下所示：

```
BEGIN TRAN
    BEGIN TRY
        DECLARE @i INT;
        SET     @i = CONVERT(int,'ABC');
    END TRY
    BEGIN CATCH

        ROLLBACK TRAN;

        DECLARE @xact smallint = XACT_STATE();

        SELECT  ERROR_NUMBER()    AS ErrNumber
        ,       ERROR_SEVERITY()  AS ErrSeverity
        ,       ERROR_STATE()     AS ErrState
        ,       ERROR_PROCEDURE() AS ErrProcedure
        ,       ERROR_MESSAGE()   AS ErrMessage
        ;
    END CATCH;

SELECT @xact;
```

请注意，事务的回滚必须发生于在 `CATCH` 块中读取错误信息之前。

## Error\_Line() 函数
另外值得注意的是，SQL 数据仓库不实现或支持 ERROR\_LINE() 函数。如果你的代码中包含此函数，需要将它删除才能符合 SQL 数据仓库的要求。请在代码中使用查询标签，而不是实现等效的功能。有关此功能的详细信息，请参阅 [查询标签] 一文。

## 使用 THROW 和 RAISERROR
THROW 是在 SQL 数据仓库中引发异常的新式做法，但也支持 RAISERROR。不过，有些值得注意的差异。

- 对于 THROW，用户定义的错误消息数目不能在 100,000 - 150,000 的范围内
- RAISERROR 错误消息固定为 50,000
- 不支持 sys.messages

## 限制
SQL 数据仓库有一些与事务相关的其他限制。

这些限制如下：

- 无分布式事务
- 不允许嵌套事务
- 不允许保存点

## 后续步骤
有关更多开发技巧，请参阅[开发概述][]。

<!--Image references-->

<!--Article references-->
[开发概述]: /documentation/articles/sql-data-warehouse-overview-develop

<!--MSDN references-->

<!--Other Web references-->

<!---HONumber=Mooncake_0321_2016-->