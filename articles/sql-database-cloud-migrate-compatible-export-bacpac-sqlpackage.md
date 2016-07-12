<properties
   pageTitle="使用 SqlPackage 将 SQL Server 数据库导出到 BACPAC 文件"
   description="Azure SQL 数据库, 数据库迁移, 导出数据库, 导出 BACPAC 文件, sqlpackage"
   services="sql-database"
   documentationCenter=""
   authors="carlrabeler"
   manager="jeffreyg"
   editor=""/>

<tags
   ms.service="sql-database"
   ms.date="03/14/2016"
   wacn.date="03/24/2016"/>

# 使用 SqlPackage 将 SQL Server 数据库导出到 BACPAC 文件

> [AZURE.SELECTOR]
- [SSMS](/documentation/articles/sql-database-cloud-migrate-compatible-export-bacpac-ssms/)
- [SqlPackage](/documentation/articles/sql-database-cloud-migrate-compatible-export-bacpac-sqlpackage/)

本文说明如何使用 [SqlPackage](https://msdn.microsoft.com/zh-cn/library/hh550080.aspx) 命令提示实用程序将 SQL Server 数据库导出到 [BACPAC](https://msdn.microsoft.com/zh-cn/library/ee210546.aspx#Anchor_4) 文件。此实用程序随 Visual Studio 和 SQL Server 一起提供。你还可以[下载](https://msdn.microsoft.com/zh-cn/library/mt204009.aspx)最新版本的 SQL Server Data Tools 以获取此实用程序。

1. 打开命令提示符并更改包含 sqlpackage.exe 命令行实用工具的目录 - 此实用程序与 Visual Studio 和 SQL Server 一起提供。使用计算机上的搜索来查找你环境中的路径。
2. 结合环境的以下参数运行以下 sqlpackage.exe 命令：

	'sqlpackage.exe /Action:Export /ssn:< server_name > /sdn:< database_name > /tf:< target_file >

	| 参数 | 说明 |
	|---|---|
	| < server_name > | 源服务器名称 |
	| < database_name > | 源数据库名称 |
	| < target_file > | BACPAC 文件的文件名和位置 |

	![通过“任务”菜单导出数据层应用程序](./media/sql-database-cloud-migrate/TestForCompatibilityUsingSQLPackage01b.png)

## 下一步：从 BACPAC 文件导入到 SQL 数据库

- [SSMS](/documentation/articles/sql-database-cloud-migrate-compatible-import-bacpac-ssms/)
- [SqlPackage](/documentation/articles/sql-database-cloud-migrate-compatible-import-bacpac-sqlpackage/)
- [PowerShell](/documentation/articles/sql-database-import-powershell/)

<!---HONumber=Mooncake_0104_2016-->
