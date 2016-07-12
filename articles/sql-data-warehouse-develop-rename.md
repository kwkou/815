<properties
   pageTitle="在 SQL 数据仓库中重命名 | Azure"
   description="有关在开发解决方案时于 Azure SQL 数据仓库中重命名表的技巧。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="jrowlandjones"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="03/23/2016"
   wacn.date="05/23/2016"/>

# 在 SQL 数据仓库中重命名
SQL Server 通过存储过程 `sp_renamedb` 支持数据库重命名，而 SQL 数据仓库使用 DDL 语法来实现相同的目的。该 DDL 命令是 `RENAME OBJECT`。

## 重命名表

目前只有表可以重命名。重命名表的语法是：

```
RENAME OBJECT Customer TO NewCustomer;
```

重命名表时，与表关联的所有对象和属性都将更新为引用新的表名称。例如，将更新表定义、索引、约束和权限。不会更新视图。

## 重命名外部表

重命名外部表会更改 SQL 数据仓库内的表名称。它不影响该表的外部数据位置。

## 更改表架构
如果目的是要更改对象所属的架构，则可通过 ALTER SCHEMA 实现此目的：

```
ALTER SCHEMA dbo TRANSFER OBJECT::product.item;
```

## 表重命名需要独占表锁

必须记住，无法在表处于使用中状态时将它重命名。重命名表时，需要该表的独占锁。如果表处于使用中状态，你可能需要终止使用该表的会话。若要终止会话，需要使用 [KILL][] 命令。使用 `KILL` 时请小心，因为终止会话时，将回滚所有未提交的工作。SQL 数据仓库中的会话以“SID”作为前缀。调用 KILL 命令时，需要包含此前缀和会话编号。例如，`KILL 'SID1234'`。有关[会话]的详细信息，请参阅连接文章


## 后续步骤
有关更多开发技巧，请参阅[开发概述][]。

<!--Image references-->

<!--Article references-->
[开发概述]: /documentation/articles/sql-data-warehouse-overview-develop/
[会话]: /documentation/articles/sql-data-warehouse-develop-connections/

<!--MSDN references-->
[KILL]: https://msdn.microsoft.com/zh-cn/library/ms173730.aspx
<!---HONumber=Mooncake_0516_2016-->