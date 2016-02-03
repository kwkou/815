<properties
   pageTitle="将 SQL Server 数据库迁移到 Azure SQL 数据库"
   description="Windows Azure SQL 数据库, 数据库部署, 数据库迁移, 导入数据库, 导出数据库, 迁移向导"
   services="sql-database"
   documentationCenter=""
   authors="carlrabeler"
   manager="jeffreyg"
   editor=""/>

<tags
   ms.service="sql-database"
   ms.date="12/17/2015"
   wacn.date="01/15/2016"/>

# 将 SQL Server 数据库迁移到 Azure SQL 数据库

在本文中，你将学习如何将本地 SQL Server 2005 或更高版本数据库迁移到 Azure SQL 数据库。在此迁移过程中，你将从当前环境中的 SQL Server 数据库将架构和数据迁移到 SQL 数据库，前提是现有数据库通过了兼容性测试。使用 [SQL 数据库 V12](/documentation/articles/sql-database-v12-whats-new) 时，除了服务器级别和跨数据库操作，有非常少的剩余兼容性问题。依赖[部分支持或不受支持功能](/documentation/articles/sql-database-transact-sql-information)的数据库和应用程序需要进行一些重新设计[修复这些不兼容性](/documentation/articles/sql-database-cloud-migrate-fix-compatibility-issues)，然后才能迁移 SQL Server 数据库。

> [AZURE.NOTE]若要将非 SQL Server 数据库（包括 Microsoft Access、Sybase、MySQL Oracle 和 DB2）迁移到 Azure SQL 数据库，请参阅 [SQL Server 迁移助手](http://blogs.msdn.com/b/ssma)。

## 确定要迁移到 SQL 数据库的 SQL Server 数据库是否兼容

> [AZURE.SELECTOR]
- [SqlPackage](/documentation/articles/sql-database-cloud-migrate-determine-compatibility-sqlpackage)
- [SQL Server Management Studio](/documentation/articles/sql-database-cloud-migrate-determine-compatibility-ssms)

若要在开始迁移过程之前测试 SQL 数据库兼容性问题，请使用以下方法之一：

- [使用 SqlPackage](/documentation/articles/sql-database-cloud-migrate-determine-compatibility-sqlpackage)：SqlPackage 是一个命令提示实用工具，它将测试兼容性问题，如果发现相关问题，则将生成包含检测到的兼容性问题的报告。
- [使用 SQL Server Management Studio](/documentation/articles/sql-database-cloud-migrate-determine-compatibility-ssms)：SQL Server Management Studio 中的“导出数据层”应用程序向导会将检测到的错误显示到屏幕上。

## 修复兼容性问题

如果检测到兼容性问题时，必须先修复这些兼容性问题，然后才能继续进行迁移。

- 使用 [SQL Azure 迁移向导](/documentation/articles/sql-database-cloud-migrate-fix-compatibility-issues)
- 使用 [SQL Server Data Tools for Visual Studio](/documentation/articles/sql-database-cloud-migrate-fix-compatibility-issues-ssdt)
- 使用 [SQL Server Management Studio](/documentation/articles/sql-database-cloud-migrate-fix-compatibility-issues-ssms)

## 将兼容的 SQL Server 数据库迁移到 SQL 数据库

为了迁移兼容的 SQL Server 数据库，Microsoft 针对各种方案提供了多个迁移方法。所选择的方法取决于你对停机时间的容忍程度、SQL Server 数据库的大小和复杂性，以及与 Windows Azure 云的连接。

> [AZURE.SELECTOR]
- [SSMS 迁移向导](/documentation/articles/sql-database-cloud-migrate-compatible-using-ssms-migration-wizard)
- [导出到 BACPAC 文件](/documentation/articles/sql-database-cloud-migrate-compatible-export-bacpac-ssms)
- [从 BACPAC 文件导入](/documentation/articles/sql-database-cloud-migrate-compatible-import-bacpac-ssms)
- [事务复制](/documentation/articles/sql-database-cloud-migrate-compatible-using-transactional-replication)

若要选择迁移方法，要问的第一个问题是你是否允许在迁移过程中使数据库脱离生产环境。正在发生活动事务时迁移数据库可能会导致数据库不一致，并且数据库可能会损坏。有许多方法可以使数据库处于静默状态，例如禁用客户端连接或者创建[数据库快照](https://msdn.microsoft.com/zh-cn/library/ms175876.aspx)。

若要使迁移时停机时间最短，请使用 [SQL Server 事务复制](/documentation/articles/sql-database-cloud-migrate-compatible-using-transactional-replication)（如果你的数据库满足事务复制的要求）。如果可以承受一定的停机时间，或者正在针对以后的迁移执行生产数据库的测试迁移，请考虑以下三种方法之一：

- [SSMS 迁移向导](/documentation/articles/sql-database-cloud-migrate-compatible-using-ssms-migration-wizard)：对于小型到中型数据库，迁移兼容的 SQL Server 2005 或更高版本数据库只需运行 SQL Server Management Studio 中的[“将数据库部署到 Windows Azure 数据库”向导](/documentation/articles/sql-database-cloud-migrate-compatible-using-migration-wizard)即可。 
- [导出到 BACPAC 文件](/documentation/articles/sql-database-cloud-migrate-compatible-export-bacpac-ssms)，然后[从 BACPAC 文件导入](/documentation/articles/sql-database-cloud-migrate-compatible-import-bacpac-ssms)：如果你遇到连接问题（无连接、低带宽或超时问题）并且对于中型到大型数据库，请使用 [BACPAC](https://msdn.microsoft.com/zh-cn/library/ee210546.aspx#Anchor_4) 文件。使用此方法，可通过 SQL Server Management Studio 中的导出数据层应用程序向导或 [SqlPackage](https://msdn.microsoft.com/zh-cn/library/hh550080.aspx) 命令提示实用工具将 SQL Server 架构和数据导出到 BACPAC 文件，然后将 BACPAC 文件导入到 SQL 数据库中。
- 将 BACPAC 和 BCP 一起使用：对于非常大的数据库使用 [BACPAC](https://msdn.microsoft.com/zh-cn/library/ee210546.aspx#Anchor_4) 文件和 [BCP](https://msdn.microsoft.com/zh-cn/library/ms162802.aspx) 可实现更高的并行化以提高性能，虽然会具有更高的复杂性。使用此方法，分别迁移架构和数据。 
 - [仅将架构导出到 BACPAC 文件](/documentation/articles/sql-database-cloud-migrate-compatible-export-bacpac-ssms)。
 - [仅从 BACPAC 文件中将架构导入](/documentation/articles/sql-database-cloud-migrate-compatible-import-bacpac-ssms)到 SQL 数据库。 
 - 使用 [BCP](https://msdn.microsoft.com/zh-cn/library/ms162802.aspx) 将数据提取到平面文件中，然后再将这些文件[并行加载](https://technet.microsoft.com/zh-cn/library/dd425070.aspx)到 Azure SQL 数据库。

	 ![SSMS 迁移示意图](./media/sql-database-cloud-migrate/01SSMSDiagram_new.png)

<!---HONumber=Mooncake_0104_2016-->
