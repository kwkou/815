<properties
   pageTitle="使用 SqlPackage 从 BACPAC 文件导入到 SQL 数据库"
   description="Azure SQL 数据库, 数据库迁移, 导入数据库, 导入 BACPAC 文件, sqlpackage"
   services="sql-database"
   documentationCenter=""
   authors="carlrabeler"
   manager="jeffreyg"
   editor=""/>

<tags
   ms.service="sql-database"
   ms.date="12/17/2015"
   wacn.date="01/15/2016"/>

# 使用 SqlPackage 从 BACPAC 文件导入到 SQL 数据库

> [AZURE.SELECTOR]
- [SSMS](/documentation/articles/sql-database-cloud-migrate-compatible-import-bacpac-ssms)
- [SqlPackage](/documentation/articles/sql-database-cloud-migrate-compatible-import-bacpac-sqlpackage)
- [Azure 门户](/documentation/articles/sql-database-import)
- [PowerShell](/documentation/articles/sql-database-import-powershell)

本文说明如何使用 [SqlPackage](https://msdn.microsoft.com/zh-cn/library/hh550080.aspx) 命令提示实用程序从 [BACPAC](https://msdn.microsoft.com/zh-cn/library/ee210546.aspx#Anchor_4) 文件导入到 SQL 数据库。此实用程序随 Visual Studio 和 SQL Server 一起提供。你还可以[下载](https://msdn.microsoft.com/zh-cn/library/mt204009.aspx)最新版本的 SQL Server Data Tools 以获取此实用程序。

> [AZURE.NOTE]下面的步骤假定你已预配 SQL 数据库服务器，手头有连接信息，并且已验证你的源数据库兼容。

## 使用 SqlPackage 从 BACPAC 文件导入到 Azure SQL 数据库

使用下面的步骤通过 [SqlPackage.exe](https://msdn.microsoft.com/zh-cn/library/hh550080.aspx) 命令行实用程序从 BACPAC 文件导入兼容的 SQL Server 数据库（或 Azure SQL 数据库）。

> [AZURE.NOTE]下面的步骤假定你已预配 Azure SQL 数据库服务器并且手头有连接信息。

1. 打开命令提示符并更改包含 sqlpackage.exe 命令行实用工具的目录 - 此实用程序与 Visual Studio 和 SQL Server 一起提供。
2. 结合环境的以下参数运行以下 sqlpackage.exe 命令：

	'sqlpackage.exe /Action:Import /tsn:< server_name > /tdn:< database_name > /tu:< user_name > /tp:< password > /sf:< source_file >

	| 参数 | 说明 |
	|---|---|
	| < server_name > | 目标服务器名称 |
	| < database_name > | 目标数据库名称 |
	| < user_name > | 目标服务器中的用户名 |
	| < password > | 用户的密码 |
	| < source_file > | 要导入的 BACPAC 文件的文件名和位置 |

	![通过“任务”菜单导出数据层应用程序](./media/sql-database-cloud-migrate/TestForCompatibilityUsingSQLPackage01c.png)

<!---HONumber=Mooncake_0104_2016-->
