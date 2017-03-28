<properties
    pageTitle="SqlPackage：将 SQL Server 数据库导出到 BACPAC 文件 (Azure) | Azure"
    description="本文说明如何使用 SqlPackage 命令行实用程序将 SQL Server 数据库导出到 BACPAC 文件。"
    keywords="Azure SQL 数据库, 数据库迁移, 导出数据库, 导出 BACPAC 文件, sqlpackage"
    services="sql-database"
    documentationcenter=""
    author="CarlRabeler"
    manager="jhubbard"
    editor="" />
<tags
    ms.assetid="7b9541c5-5590-4c70-ad36-73007389f6dc"
    ms.service="sql-database"
    ms.custom="migrate and move"
    ms.devlang="NA"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="sqldb-migrate"
    ms.date="02/07/2017"
    wacn.date="03/24/2017"
    ms.author="carlrab" />  


# 使用 SqlPackage 将 Azure SQL 数据库或 SQL Server 数据库导出到 BACPAC 文件

本文说明如何使用 [SqlPackage](https://msdn.microsoft.com/zh-cn/library/hh550080.aspx) 命令行实用程序将 Azure SQL 数据库或 SQL Server 数据库导出到 BACPAC 文件。此实用程序随 [SQL Server Management Studio](https://msdn.microsoft.com/zh-cn/library/mt238290.aspx) 和[用于 Visual Studio 的 SQL Server Data Tools](https://msdn.microsoft.com/zh-cn/library/mt204009.aspx) 的最新版本提供，也可直接从 Microsoft 下载中心下载 [SqlPackage](https://www.microsoft.com/download/details.aspx?id=53876) 的最新版本。

如需大致了解如何导出到 BACPAC 文件，请参阅[导出到 BACPAC](/documentation/articles/sql-database-export/)。

> [AZURE.NOTE]
>也可使用 [Azure 门户预览](/documentation/articles/sql-database-export-portal/)、[SQL Server Management Studio](/documentation/articles/sql-database-export-ssms/) 或 [PowerShell](/documentation/articles/sql-database-export-powershell/) 将 Azure SQL 数据库文件导出到 BACPAC 文件。

## SQLPackage 实用程序

1. 打开命令提示符并更改包含 sqlpackage.exe 命令行实用程序的目录 - 此实用程序随 Visual Studio 和 SQL Server 一起提供。使用计算机上的搜索来查找环境中的路径。
2. 结合环境的以下参数执行以下 sqlpackage.exe 命令：
   
		sqlpackage.exe /Action:Export /ssn:< server_name > /sdn:< database_name > /tf:< target_file >

   
   | 变量 | 说明 | 
   | --- | --- | 
   | < server\_name > |源服务器名称 | 
   | < database\_name > |源数据库名称 | 
   | < target\_file > |BACPAC 文件的文件名和位置 |
   
   ![通过“任务”菜单导出数据层应用程序](./media/sql-database-cloud-migrate/TestForCompatibilityUsingSQLPackage01b.png)  


## 后续步骤

* 若要了解如何使用 SQLPackage 导入 BACPAC，请参阅[使用 SqlPackage 将 BACPAC 导入到 Azure SQL 数据库](/documentation/articles/sql-database-import-sqlpackage/)
* 若要了解如何使用 Azure 门户预览导入 BACPAC，请参阅[使用 Azure 门户预览将 BACPAC 导入到 Azure SQL 数据库](/documentation/articles/sql-database-import-portal/)
* 若要了解如何使用 PowerShell 导入 BACPAC，请参阅[使用 PowerShell 将 BACPAC 导入到 Azure SQL 数据库](/documentation/articles/sql-database-import-powershell/)
* 如需 SQL Server 数据库完整迁移过程的介绍（包括性能建议），请参阅[将 SQL Server 数据库迁移到 Azure SQL 数据库](/documentation/articles/sql-database-cloud-migrate/)。
* 若要了解如何将 BACPAC 导入到 SQL Server 数据库，请参阅[将 BACPCAC 导入 SQL Server 数据库](https://msdn.microsoft.com/zh-cn/library/hh710052.aspx)

<!---HONumber=Mooncake_0320_2017-->