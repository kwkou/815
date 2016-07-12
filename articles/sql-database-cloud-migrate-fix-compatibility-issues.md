<properties
   pageTitle="在迁移到 SQL 数据库之前，修复 SQL Server 数据库兼容性问题"
   description="Azure SQL 数据库, 数据库迁移, 兼容性, SQL Azure 迁移向导"
   services="sql-database"
   documentationCenter=""
   authors="carlrabeler"
   manager="jhubbard"
   editor=""/>

<tags
   ms.service="sql-database"
   ms.date="03/23/2016"
   wacn.date="05/16/2016"/>

# 在迁移到 SQL 数据库之前使用 SQL Azure 迁移向导解决 SQL Server 数据库兼容性问题

如果你确定源 SQL Server 数据库不兼容，则有几个选项可修复已确定的数据库兼容性问题。

> [AZURE.SELECTOR]
- 使用 [SQL Azure 迁移向导](/documentation/articles/sql-database-cloud-migrate-fix-compatibility-issues/)
- 使用 [SSDT](/documentation/articles/sql-database-cloud-migrate-fix-compatibility-issues-ssdt/)
- 使用 [SSMS](/documentation/articles/sql-database-cloud-migrate-fix-compatibility-issues-ssms/)

## 使用 SQL Azure 迁移向导

可以使用 [SQL Azure 迁移向导](http://sqlazuremw.codeplex.com) CodePlex 工具从不兼容的源数据库生成 T-SQL 脚本，向导随后将转换该脚本使其与 SQL 数据库兼容，然后连接到 Azure SQL 数据库以执行该脚本。此工具还将分析跟踪文件以确定兼容性问题。生成的脚本可以只包含架构，也可以包含 BCP 格式的数据。其他文档（包括分步指南）可在 CodePlex 上的 [SQL Azure 迁移向导](http://sqlazuremw.codeplex.com)中找到。

 ![SAMW 迁移示意图](./media/sql-database-cloud-migrate/02SAMWDiagram.png)

  > [AZURE.NOTE] 请注意，向导的内置转换并非能够修复它可检测到的所有不兼容的架构。无法解决的非兼容脚本将报告为错误，将在生成的脚本中注入注释。如果检测到许多错误，请使用 Visual Studio 或 SQL Server Management Studio 来单步执行并修复无法使用 SQL Server 迁移向导修复的每个错误。

## 下一步：选择迁移方法并执行迁移

[选择迁移方法](/documentation/articles/sql-database-cloud-migrate/#migrate-a-compatible-sql-server-database-to-sql-database)。

<!---HONumber=Mooncake_0503_2016-->
