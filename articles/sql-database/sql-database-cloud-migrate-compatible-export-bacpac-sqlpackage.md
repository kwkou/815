<properties
    pageTitle="使用 SqlPackage 将 SQL Server 数据库导出到 BACPAC 文件 | Azure"
    description="Azure SQL 数据库, 数据库迁移, 导出数据库, 导出 BACPAC 文件, sqlpackage"
    services="sql-database"
    documentationcenter=""
    author="CarlRabeler"
    manager="jhubbard"
    editor="" />
<tags
    ms.assetid="7b9541c5-5590-4c70-ad36-73007389f6dc"
    ms.service="sql-database"
    ms.devlang="NA"
    ms.topic="article"
    ms.tgt_pltfrm="NA"
    ms.workload="sqldb-migrate"
    ms.date="11/08/2016"
    wacn.date="12/19/2016"
ms.author="carlrab" />

# 使用 SqlPackage 将 SQL Server 数据库导出到 BACPAC 文件

- [Azure 门户](/documentation/articles/sql-database-export/)
- [SSMS](/documentation/articles/sql-database-cloud-migrate-compatible-export-bacpac-ssms/)
- [SqlPackage](/documentation/articles/sql-database-cloud-migrate-compatible-export-bacpac-sqlpackage/)
- [PowerShell](/documentation/articles/sql-database-export-powershell/)

本文说明如何使用 [SqlPackage](https://msdn.microsoft.com/zh-cn/library/hh550080.aspx) 命令行实用程序将 SQL Server 数据库导出到 [BACPAC](https://msdn.microsoft.com/zh-cn/library/ee210546.aspx#Anchor_4) 文件。此实用程序随 [SQL Server Management Studio](https://msdn.microsoft.com/zh-cn/library/mt238290.aspx) 和[用于 Visual Studio 的 SQL Server Data Tools](https://msdn.microsoft.com/zh-cn/library/mt204009.aspx) 的最新版本提供，也可直接从 Microsoft 下载中心下载 [SqlPackage](https://www.microsoft.com/zh-cn/download/details.aspx?id=53876) 的最新版本。

1. 打开命令提示符并更改包含 sqlpackage.exe 命令行实用程序的目录 - 此实用程序随 Visual Studio 和 SQL Server 一起提供。使用计算机上的搜索来查找环境中的路径。
2. 结合环境的以下参数执行以下 sqlpackage.exe 命令：

	'sqlpackage.exe /Action:Export /ssn:< server_name > /sdn:< database_name > /tf:< target_file >

	| 参数 | 说明 |
	|---|---|
	| < server_name > | 源服务器名称 |
	| < database_name > | 源数据库名称 |
	| < target_file > | BACPAC 文件的文件名和位置 |

	![通过“任务”菜单导出数据层应用程序](./media/sql-database-cloud-migrate/TestForCompatibilityUsingSQLPackage01b.png)  


## 后续步骤
- [最新版本的 SSDT](https://msdn.microsoft.com/zh-cn/library/mt204009.aspx)
- [最新版本的 SQL Server Management Studio](https://msdn.microsoft.com/zh-cn/library/mt238290.aspx)
- [使用 SSMS 将 BACPAC 导入 Azure SQL 数据库](/documentation/articles/sql-database-cloud-migrate-compatible-import-bacpac-ssms/)
- [将 BACPAC 导入 Azure SQL 数据库 SqlPackage](/documentation/articles/sql-database-cloud-migrate-compatible-import-bacpac-sqlpackage/)
- [将 BACPAC 导入 Azure SQL 数据库 Azure 门户](/documentation/articles/sql-database-import/)
- [将 BACPAC 导入 Azure SQL 数据库 PowerShell](/documentation/articles/sql-database-import-powershell/)

## 其他资源

- [SQL 数据库 V12](/documentation/articles/sql-database-v12-whats-new/)
- [Transact-SQL 部分支持或不支持的函数](/documentation/articles/sql-database-transact-sql-information/)
- [使用 SQL Server 迁移助手迁移非 SQL Server 数据库](http://blogs.msdn.com/b/ssma/)

<!---HONumber=Mooncake_1212_2016-->