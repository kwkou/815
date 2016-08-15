<properties
   pageTitle="在迁移到 SQL 数据库之前使用 SQL Server Management Studio 解决 SQL Server 数据库兼容性问题 | Azure"
   description="Azure SQL 数据库, 数据库迁移, 兼容性, SQL Azure 迁移向导"
   services="sql-database"
   documentationCenter=""
   authors="carlrabeler"
   manager="jeffreyg"
   editor=""/>

<tags
   ms.service="sql-database"
   ms.date="06/07/2016"
   wacn.date="07/11/2016"/> 

# 在迁移到 SQL 数据库之前使用 SQL Server Management Studio 解决 SQL Server 数据库兼容性问题

> [AZURE.SELECTOR]
- 使用 [SQL Azure 迁移向导](/documentation/articles/sql-database-cloud-migrate-fix-compatibility-issues/)
- 使用 [SSDT](/documentation/articles/sql-database-cloud-migrate-fix-compatibility-issues-ssdt/)
- 使用 [SSMS](/documentation/articles/sql-database-cloud-migrate-fix-compatibility-issues-ssms/)

在迁移到 Azure SQL 数据库之前，高级用户可使用 SQL Server Management Studio 解决 SQL Server 数据库的兼容性问题。

## 使用 SQL Server Management Studio

使用 SQL Server Management Studio 通过各种 Transact-SQL 命令（如 **ALTER DATABASE**）来修复兼容性问题。该方法主要面向能够在实时数据库上轻松使用 Transact-SQL 的高级用户。如果不是，建议使用 SSDT。



## 后续步骤

- [最新版本的 SSDT](https://msdn.microsoft.com/zh-cn/library/mt204009.aspx)
- [最新版本的 SQL Server Management Studio](https://msdn.microsoft.com/zh-cn/library/mt238290.aspx)
- [将兼容的 SQL Server 数据库迁移到 SQL 数据库](/documentation/articles/sql-database-cloud-migrate/#migrate-a-compatible-sql-server-database-to-sql-database)

## 其他资源

- [SQL 数据库 V12](/documentation/articles/sql-database-v12-whats-new/)
- [Transact-SQL 部分支持或不支持的函数](/documentation/articles/sql-database-transact-sql-information/)
- [使用 SQL Server 迁移助手迁移非 SQL Server 数据库](http://blogs.msdn.com/b/ssma/)

<!---HONumber=Mooncake_0704_2016-->