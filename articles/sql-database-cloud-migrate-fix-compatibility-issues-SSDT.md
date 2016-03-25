<properties
   pageTitle="在迁移到 SQL 数据库之前，修复 SQL Server 数据库兼容性问题"
   description="Azure SQL 数据库, 数据库迁移, 兼容性, SQL Azure 迁移向导, SSDT"
   services="sql-database"
   documentationCenter=""
   authors="carlrabeler"
   manager="jeffreyg"
   editor=""/>

<tags
   ms.service="sql-database"
   ms.date="03/14/2016"
   wacn.date="03/24/2016"/>

# 在迁移到 SQL 数据库之前，修复 SQL Server 数据库兼容性问题

如果你确定源 SQL Server 数据库不兼容，则有几个选项可修复已确定的数据库兼容性问题。

> [AZURE.SELECTOR]
- 使用 [SQL Azure 迁移向导](/documentation/articles/sql-database-cloud-migrate-fix-compatibility-issues)
- 使用 [SSDT](/documentation/articles/sql-database-cloud-migrate-fix-compatibility-issues-SSDT)
- 使用 [SSMS](/documentation/articles/sql-database-cloud-migrate-fix-compatibility-issues-SSMS)

## 使用 SQL Server Data Tools for Visual Studio

可以使用 SQL Server Data Tools for Visual Studio ("SSDT") 将数据库架构导入到 Visual Studio 数据库项目中以进行分析。若要分析，请将该项目的目标平台指定为 SQL 数据库 V12，然后生成该项目。如果生成成功，则数据库是兼容的。如果生成失败，则可以解决 SSDT（或本主题中讨论的其他工具之一）中的错误。成功生成项目后，你便可以将其发布为源数据库的副本，然后使用 SSDT 中的数据比较功能将数据从源数据库复制到这个与 Azure SQL V12 兼容的数据库。然后，可以迁移这个已更新的数据库。若要使用此选项，请下载[最新版本的 SSDT](https://msdn.microsoft.com/zh-cn/library/mt204009.aspx)。

  ![VSSSDT 迁移示意图](./media/sql-database-cloud-migrate/03VSSSDTDiagram.png)

  >[AZURE.NOTE]如果需要进行仅有架构的迁移，则可以将架构直接从 Visual Studio 发布到 Azure SQL 数据库。当数据库架构所需的更改量超过了 SAMW 单独可处理的数量时使用。

## 下一步：选择迁移方法并执行迁移

[选择迁移方法](/documentation/articles/sql-database-cloud-migrate/#migrate-a-compatible-sql-server-database-to-sql-database)。

<!---HONumber=Mooncake_0104_2016-->
