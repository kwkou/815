<properties
   pageTitle="将 SQL Server 数据库迁移到 SQL 数据库 | Azure"
   description="了解如何将本地 SQL Server 数据库迁移到云中的 Azure SQL 数据库。在执行数据库迁移之前使用数据库迁移工具测试兼容性。"
   keywords="数据库迁移、SQL Server 数据库迁移、数据库迁移工具、迁移数据库、迁移 SQL 数据库"
   services="sql-database"
   documentationCenter=""
   authors="carlrabeler"
   manager="jhubbard"
   editor=""/>

<tags
   ms.service="sql-database"
   ms.date="03/22/2016"
   wacn.date="05/16/2016"/>

# 将 SQL Server 数据库迁移到云中的 SQL 数据库

在本文中，你将学习如何将本地 SQL Server 2005 或更高版本数据库迁移到 Azure SQL 数据库。在此数据库迁移过程中，你将从当前环境中的 SQL Server 数据库将架构和数据迁移到 SQL 数据库，前提是现有数据库通过了兼容性测试。使用 [SQL 数据库 V12](/documentation/articles/sql-database-v12-whats-new/) 时，除了服务器级别和跨数据库操作，有非常少的剩余兼容性问题。依赖[部分支持或不受支持的函数](/documentation/articles/sql-database-transact-sql-information/)的数据库和应用程序需要进行一些重新设计修复这些不兼容性，然后才能迁移 SQL Server 数据库。

若要迁移，下面是要执行的步骤：

- **兼容性测试**：必须先验证数据库是否与 [SQL 数据库 V12](/documentation/articles/sql-database-v12-whats-new/) 兼容。 
- **解决兼容性问题（如果有）**：如果验证失败，你必须先解决验证错误。  
- **运行迁移**：如果数据库兼容，你可以使用一种或多种方法来执行迁移。 

SQL Server 提供多种方法来完成每个任务。本文将提供可供每个任务使用的方法概述。下图演示了步骤和方法。

  ![VSSSDT 迁移示意图](./media/sql-database-cloud-migrate/03VSSSDTDiagram.png)
  
 > [AZURE.NOTE] 若要将非 SQL Server 数据库（包括 Microsoft Access、Sybase、MySQL Oracle 和 DB2）迁移到 Azure SQL 数据库，请参阅 [SQL Server 迁移助手](http://blogs.msdn.com/b/ssma)。

## 数据库迁移工具将测试 SQL Server 数据库与 SQL 数据库的兼容性

在开始数据库迁移过程之前，请使用以下方法之一测试 SQL 数据库的兼容性问题：

- [SQL Server Data Tools for Visual Studio (SSDT)](/documentation/articles/sql-database-cloud-migrate-fix-compatibility-issues-ssdt/)：SSDT 使用最新的兼容性规则来检测 SQL 数据库 V12 不兼容性。如果检测到不兼容，你可以直接在此工具中解决检测到的问题。这是目前用于测试和解决 SQL 数据库 V12 兼容性问题的建议方法。 
- [SqlPackage](/documentation/articles/sql-database-cloud-migrate-determine-compatibility-sqlpackage/)：SqlPackage 是一个命令提示实用工具，它将测试兼容性问题，如果发现相关问题，则将生成包含检测到的兼容性问题的报告。如果使用此工具，请确保使用最新版本，以便使用最新的兼容性规则。如果检测到错误，必须使用其他工具来解决任何检测到的兼容性问题 - 建议使用 SSDT。  
- [SQL Server Management Studio 中的“导出数据层”应用程序向导](/documentation/articles/sql-database-cloud-migrate-determine-compatibility-ssms/)：此向导可以检测错误并在屏幕上报告错误。如果未检测到错误，你可以继续迁移到 SQL 数据库。如果检测到错误，必须使用其他工具来解决任何检测到的兼容性问题 - 建议使用 SSDT。
- [Microsoft SQL Server 2016 Upgrade Advisor Preview](http://www.microsoft.com/download/details.aspx?id=48119)：此独立工具目前以预览版提供，将检测并生成 SQL 数据库 V12 不兼容性报告。此工具还没有最新的兼容性规则。如果未检测到错误，你可以继续迁移到 SQL 数据库。如果检测到错误，必须使用其他工具来解决任何检测到的兼容性问题 - 建议使用 SSDT。 
- [SQL Azure 迁移向导 (SAMW)](/documentation/articles/sql-database-cloud-migrate-fix-compatibility-issues/)：SAMW 是一个 codeplex 工具，它使用 Azure SQL 数据库 V11 兼容性规则来检测 Azure SQL 数据库 V12 的不兼容性。如果检测到不兼容，某些问题可直接在此工具中解决。此工具可能会发现不需要解决的不兼容性，但它是第一个可使用的 Azure SQL 数据库迁移帮助工具，并且有很多来自 SQL Server 社区的支持。此外，此工具可在工具本身内部完成迁移。 

## 修复数据库迁移的兼容性问题

如果检测到兼容性问题，必须先修复这些兼容性问题，才能继续 SQL Server 数据库迁移。根据源数据库中的 SQL Server 版本以及正在迁移的数据库复杂性，可能会发现各种不同的不兼容性问题。源数据库的 SQL Server 版本越旧，就越可能发现不兼容。除了使用所选搜索引擎的目标 Internet 搜索以外，还可以使用以下资源：

- [Azure SQL 数据库中不支持的 SQL Server 数据库功能](/documentation/articles/sql-database-transact-sql-information/)
- [SQL Server 2016 中已停用的数据库引擎功能](https://msdn.microsoft.com/zh-cn/library/ms144262%28v=sql.130%29)
- [SQL Server 2014 中已停用的数据库引擎功能](https://msdn.microsoft.com/zh-cn/library/ms144262%28v=sql.120%29)
- [SQL Server 2012 中已停用的数据库引擎功能](https://msdn.microsoft.com/zh-cn/library/ms144262%28v=sql.110%29)
- [SQL Server 2008 R2 中已停用的数据库引擎功能](https://msdn.microsoft.com/zh-cn/library/ms144262%28v=sql.105%29)
- [SQL Server 2005 中已停用的数据库引擎功能](https://msdn.microsoft.com/zh-cn/library/ms144262%28v=sql.90%29)

除了搜索 Internet 和使用这些资源以外，你可使用 [MSDN SQL Server 社区论坛](https://social.msdn.microsoft.com/Forums/sqlserver/home?category=sqlserver)或 [StackOverflow](http://stackoverflow.com)，这些极佳的资源可帮助你找到解决不兼容性问题的最适当方法。

使用以下数据库迁移工具之一来解决检测到的问题：

- [使用 SQL Server Data Tools for Visual Studio (SSDT)](/documentation/articles/sql-database-cloud-migrate-fix-compatibility-issues-ssdt/)：若要使用 SSDT，需将数据库架构导入 SQL Server Data Tools for Visual Studion (SSDT)，构建 SQL 数据库 V12 部署的项目，在 SSDT 中解决所有检测到的兼容性问题，然后将更改同步回到源数据库（或源数据库的副本）。这是目前用于测试和解决 SQL 数据库 V12 兼容性问题的建议方法。遵循[演练 SSDT](/documentation/articles/sql-database-cloud-migrate-fix-compatibility-issues-ssdt/) 的链接。
- [使用 SQL Server Management Studio (SSMS)](/documentation/articles/sql-database-cloud-migrate-fix-compatibility-issues-ssms/)：若要使用 SSMS，可以使用其他工具来解决检测到的错误，以运行 Transact-SQL 命令来解决检测到的错误。此方法主要是供高级用户直接在源数据库中修改数据库架构。 
- [使用 SQL Azure 迁移向导 (SAMW)](/documentation/articles/sql-database-cloud-migrate-fix-compatibility-issues/)：若要使用 SAMW，需要从源数据库生成 Transact-SQL 脚本，稍后由向导在适当时机将其转换，让架构和 SQL 数据库 V12 兼容。完成后，SAMW 可以连接到 SQL 数据库 V12 以执行脚本。此工具还将分析跟踪文件以确定兼容性问题。生成的脚本可以只包含架构，也可以包含 BCP 格式的数据。

## 将兼容的 SQL Server 数据库迁移到 SQL 数据库

为了迁移兼容的 SQL Server 数据库，Microsoft 针对各种方案提供了多个迁移方法。所选择的方法取决于你对停机时间的容忍程度、对 SQL Server 数据库的大小和复杂性要求，以及与 Azure 云的连接。

> [AZURE.SELECTOR]
- [SSMS 迁移向导](/documentation/articles/sql-database-cloud-migrate-compatible-using-ssms-migration-wizard/)
- [导出到 BACPAC 文件](/documentation/articles/sql-database-cloud-migrate-compatible-export-bacpac-ssms/)
- [从 BACPAC 文件导入](/documentation/articles/sql-database-cloud-migrate-compatible-import-bacpac-ssms/)
- [事务复制](/documentation/articles/sql-database-cloud-migrate-compatible-using-transactional-replication/)

若要选择迁移方法，要问的第一个问题是你是否允许在迁移过程中使数据库脱离生产环境。正在发生活动事务时迁移数据库可能会导致数据库不一致，并且数据库可能会损坏。有许多方法可以使数据库处于静默状态，例如禁用客户端连接或者创建[数据库快照](https://msdn.microsoft.com/zh-cn/library/ms175876.aspx)。

若要使迁移时停机时间最短，请使用 [SQL Server 事务复制](/documentation/articles/sql-database-cloud-migrate-compatible-using-transactional-replication/)（如果你的数据库满足事务复制的要求）。如果可以承受一定的停机时间，或者正在针对以后的迁移执行生产数据库的测试迁移，请考虑以下三种方法之一：

- [SSMS 迁移向导](/documentation/articles/sql-database-cloud-migrate-compatible-using-ssms-migration-wizard/)：对于小型到中型数据库，迁移兼容的 SQL Server 2005 或更高版本数据库就像在 SQL Server Management Studio 中运行[将数据库部署到 Azure 数据库](/documentation/articles/sql-database-cloud-migrate-compatible-using-ssms-migration-wizard/)向导一样简单。
- [导出到 BACPAC 文件](/documentation/articles/sql-database-cloud-migrate-compatible-export-bacpac-ssms/)，然后[从 BACPAC 文件导入](/documentation/articles/sql-database-cloud-migrate-compatible-import-bacpac-ssms/)：如果你遇到连接问题（无连接、低带宽或超时问题）并且对于中型到大型数据库，请使用 [BACPAC](https://msdn.microsoft.com/zh-cn/library/ee210546.aspx#Anchor_4) 文件。使用此方法，可通过 SQL Server Management Studio 中的导出数据层应用程序向导或 [SqlPackage](https://msdn.microsoft.com/zh-cn/library/hh550080.aspx) 命令提示实用工具将 SQL Server 架构和数据导出到 BACPAC 文件，然后将 BACPAC 文件导入到 SQL 数据库中。
- 将 BACPAC 和 BCP 一起使用：对于非常大的数据库使用 [BACPAC](https://msdn.microsoft.com/zh-cn/library/ee210546.aspx#Anchor_4) 文件和 [BCP](https://msdn.microsoft.com/zh-cn/library/ms162802.aspx) 可实现更高的并行化以提高性能，虽然会具有更高的复杂性。使用此方法，分别迁移架构和数据。
 - [仅将架构导出到 BACPAC 文件](/documentation/articles/sql-database-cloud-migrate-compatible-export-bacpac-ssms/)。
 - [仅从 BACPAC 文件中将架构导入](/documentation/articles/sql-database-cloud-migrate-compatible-import-bacpac-ssms/)到 SQL 数据库。
 - 使用 [BCP](https://msdn.microsoft.com/zh-cn/library/ms162802.aspx) 将数据提取到平面文件中，然后再将这些文件[并行加载](https://technet.microsoft.com/zh-cn/library/dd425070.aspx)到 Azure SQL 数据库。

	 ![SQL Server 数据库迁移 - 将 SQL 数据库迁移到云。](./media/sql-database-cloud-migrate/01SSMSDiagram_new.png)

<!---HONumber=Mooncake_0503_2016-->
