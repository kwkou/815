<properties
    pageTitle="将 Azure SQL 数据库导出到 BACPAC 文件 | Azure"
    description="使用 Azure 门户将 Azure SQL 数据库导出到 BACPAC 文件"
    services="sql-database"
    documentationcenter=""
    author="CarlRabeler"
    manager="jhubbard"
    editor="" />
<tags
    ms.assetid="41d63a97-37db-4e40-b652-77c2fd1c09b7"
    ms.service="sql-database"
    ms.custom="migrate and move"
    ms.devlang="NA"
    ms.date="02/07/2017"
    wacn.date="03/24/2017"
    ms.author="carlrab"
    ms.workload="data-management"
    ms.topic="article"
    ms.tgt_pltfrm="NA" />  


# 将 Azure SQL 数据库或 SQL Server 数据库导出到 BACPAC 文件

本文介绍了如何将 Azure SQL 数据库或 SQL Server 数据库导出到 BACPAC 文件。

> [AZURE.IMPORTANT]
>Azure SQL 数据库自动导出现在处于预览状态且将于 2017 年 3 月 1 日停用。从 2016 年 12 月 1 日起，不再能够在任何 SQL 数据库上配置自动导出。所有现有的自动导出作业将继续工作，直到 2017 年 3 月 1 日。在 2016 年 12 月 1 日后，可以根据所选的计划定期通过 [Azure 自动化](/documentation/articles/automation-intro/)存档 SQL 数据库。如需示例脚本，可以[从 Github 下载示例脚本](https://github.com/Microsoft/sql-server-samples/tree/master/samples/manage/azure-automation-automated-export)。
>

## 概述

需要导出数据库以便将其存档或移至其他平台时，可以将数据库架构和数据导出到 BACPAC 文件。BACPAC 文件只是一个扩展名为 BACPAC 的 ZIP 文件。之后可将 BACPAC 文件存储在 Azure Blob 存储中或存储在本地位置的本地存储中，之后可以导入回 Azure SQL 数据库或 SQL Server 本地安装中。

* 可以使用 [Azure 门户预览](/documentation/articles/sql-database-export-portal/)、[PowerShell](/documentation/articles/sql-database-export-powershell/)、[SQLPackage](/documentation/articles/sql-database-export-sqlpackage/) 或 [SQL Server Management Studio](/documentation/articles/sql-database-export-ssms/) 导出 Azure SQL 数据库。
* 可以使用 [PowerShell](/documentation/articles/sql-database-export-powershell/)、[SQLPackage](/documentation/articles/sql-database-export-sqlpackage/) 或 [SQL Server Management Studio](/documentation/articles/sql-database-export-ssms/) 导出 SQL Server 数据库。

> [AZURE.IMPORTANT]
>如果要在迁移到 Azure SQL 数据库之前从 SQL Server 进行导出，请参阅[将 SQL Server 数据库迁移到 Azure SQL 数据库](/documentation/articles/sql-database-cloud-migrate/)。
>

## 注意事项

- 为保证导出的事务处理方式一致，必须确保导出期间未发生写入活动，或者你是从 Azure SQL 数据库的[事务处理方式一致性副本](/documentation/articles/sql-database-copy/)中导出。
- 如果是导出到 Blob 存储，则 BACPAC 文件的最大大小为 200 GB。若要存档更大的 BACPAC 文件，请导出到本地存储。
- 不支持使用本文所述方法将 BACPAC 文件导出到 Azure 高级存储。
- 如果从 Azure SQL 数据库进行的导出操作超过 20 个小时，系统可能会取消该操作。为提高导出过程中的性能，你可以进行如下操作：
 - 暂时提高服务级别。
 - 在导出期间终止所有读取和写入活动。
 - 对所有大型表格上的非 null 值使用[聚集索引](https://msdn.microsoft.com/zh-cn/library/ms190457.aspx)。如果不使用聚集索引，当时间超过 6-12 个小时时，导出可能会失败。这是因为导出服务需要完成表格扫描，才能尝试导出整个表格。确认表格是否针对导出进行优化的一个好方法是，运行 **DBCC SHOW\_STATISTICS**，确保 *RANGE\_HI\_KEY* 不是 null 并且值分布良好。有关详细信息，请参阅 [DBCC SHOW\_STATISTICS](https://msdn.microsoft.com/zh-cn/library/ms174384.aspx)。

> [AZURE.NOTE]
>BACPAC 不能用于备份和还原操作。Azure SQL 数据库会自动为每个用户数据库创建备份。有关详细信息，请参阅[业务连续性概述](/documentation/articles/sql-database-business-continuity/)和 [SQL 数据库备份](/documentation/articles/sql-database-automated-backups/)。
> 


## 后续步骤

* 如需 SQL Server 数据库完整迁移过程的介绍，请参阅[将 SQL Server 数据库迁移到 Azure SQL 数据库](/documentation/articles/sql-database-cloud-migrate/)。
* 有关在 Azure 中复制数据库的概述，另请参阅[复制 Azure SQL 数据库](/documentation/articles/sql-database-copy/)。
* 可以使用 [Azure 门户](/documentation/articles/sql-database-copy-portal/)、[PowerShell](/documentation/articles/sql-database-copy-powershell/) 或 [Transact-SQL](/documentation/articles/sql-database-copy-transact-sql/) 在 Azure 中复制 Azure SQL 数据库。

<!---HONumber=Mooncake_0320_2017-->
<!--Update_Description: simplify content structure, remove configure steps-->