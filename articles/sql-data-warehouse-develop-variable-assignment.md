<properties
   pageTitle="在 SQL 数据仓库中分配变量 | Azure"
   description="有关在开发解决方案时于 Azure SQL 数据仓库中分配 Transact-SQL 变量的技巧。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="jrowlandjones"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="03/23/2016"
   wacn.date="05/23/2016"/>

# 在 SQL 数据仓库中分配变量
SQL 数据仓库中的变量是使用 `DECLARE` 语句或 `SET` 语句设置的。

以下各项是设置变量值的完全有效方式：

## 使用 DECLARE 设置变量

使用 DECLARE 初始化变量是在 SQL 数据仓库中设置变量值的最灵活方式之一。

```
DECLARE @v  int = 0
;
```

你还可以使用 DECLARE 一次性设置多个变量。可以使用 `SELECT` 或 `UPDATE` 来实现此目的：

```
DECLARE @v  INT = (SELECT TOP 1 c_customer_sk FROM Customer where c_last_name = 'Smith')
,       @v1 INT = (SELECT TOP 1 c_customer_sk FROM Customer where c_last_name = 'Jones')
;
```

你无法在同一 DECLARE 语句中初始化和使用某个变量。为了演示要点，**不**允许出现以下示例中的情况，因为 @p1 已在同一个 DECLARE 语句中初始化和使用。这会导致错误。

```
DECLARE @p1 int = 0
,       @p2 int = (SELECT COUNT (*) FROM sys.types where is_user_defined = @p1 )
;
```

## 使用 SET 设置值
SET 是设置单个变量的很常见方法。

以下所有示例都是使用 SET 设置变量的有效方式：

```
SET     @v = (Select max(database_id) from sys.databases);
SET     @v = 1;
SET     @v = @v+1;
SET     @v +=1;
```

一次只能使用 SET 设置一个变量。但是，如上所示，允许使用复合运算符。

## 限制
不能使用 SELECT 或 UPDATE 来分配变量。


## 后续步骤
有关更多开发技巧，请参阅[开发概述][]。

<!--Image references-->

<!--Article references-->
[开发概述]: /documentation/articles/sql-data-warehouse-overview-develop/

<!--MSDN references-->

<!--Other Web references-->

<!---HONumber=Mooncake_0321_2016-->