
<properties
   pageTitle="使用 SQL Server Management Studio 将 SQL Server 数据库导出到 BACPAC 文件 | Azure"
   description="Azure SQL 数据库, 数据库迁移, 导出数据库, 导出 BACPAC 文件, 导出数据层应用程序向导"
   services="sql-database"
   documentationCenter=""
   authors="CarlRabeler"
   manager="jhubbard"
   editor=""/>

<tags
   ms.service="sql-database"
   ms.devlang="NA"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="data-management"
   ms.date="08/16/2016"
   wacn.date="10/17/2016"
   ms.author="carlrab"/>  


# 使用 SQL Server Management Studio 将 SQL Server 数据库导出到 BACPAC 文件

> [AZURE.SELECTOR]
- [SSMS](/documentation/articles/sql-database-cloud-migrate-compatible-export-bacpac-ssms/)
- [SqlPackage](/documentation/articles/sql-database-cloud-migrate-compatible-export-bacpac-sqlpackage/)

 
本文说明如何使用 SQL Server Management Studio 中的“导出数据层应用程序”向导将 SQL Server 数据库导出到 [BACPAC](https://msdn.microsoft.com/zh-cn/library/ee210546.aspx#Anchor_4) 文件。

1. 确认你安装了最新版本的 SQL Server Management Studio。Management Studio 的新版本将每月更新一次，以与 Azure 门户的更新保持同步。

	 > [AZURE.IMPORTANT] 建议始终使用最新版本的 Management Studio 以保持与 Azure 和 SQL 数据库的更新同步。[更新 SQL Server Management Studio](https://msdn.microsoft.com/zh-cn/library/mt238290.aspx)。

2. 打开 Management Studio 并连接到你在对象资源管理器中的源数据库。

	![通过“任务”菜单导出数据层应用程序](./media/sql-database-cloud-migrate/MigrateUsingBACPAC01.png)

3. 右键单击对象资源管理器中的源数据库，指向“任务”，然后单击“导出数据层应用程序...”

	![通过“任务”菜单导出数据层应用程序](./media/sql-database-cloud-migrate/TestForCompatibilityUsingSSMS01.png)

4. 在导出向导中，配置导出以将 BACPAC 文件保存到本地磁盘位置或 Azure blob。导出的 BACPAC 始终包括完整的数据库架构，默认情况下还包括所有表中的数据。若要排除部分或全部表中的数据，请使用“高级”选项卡。例如，你可以选择仅导出引用表的数据，而不是导出所有表中的数据。

***重要说明***：将 BACPAC 导出至 Azure Blob 存储时，请使用标准存储。不支持从高级存储导入 BACPAC。

![Export settings](./media/sql-database-cloud-migrate/MigrateUsingBACPAC02.png)


## 后续步骤

- [最新版本的 SSDT](https://msdn.microsoft.com/zh-cn/library/mt204009.aspx)
- [最新版本的 SQL Server Management Studio](https://msdn.microsoft.com/zh-cn/library/mt238290.aspx)
- [使用 SSMS 将 BACPAC 导入 Azure SQL 数据库](/documentation/articles/sql-database-cloud-migrate-compatible-import-bacpac-ssms/)
- [将 BACPAC 导入 Azure SQL 数据库 SqlPackage](/documentation/articles/sql-database-cloud-migrate-compatible-import-bacpac-sqlpackage/)
- [将 BACPAC 导入 Azure SQL 数据库 PowerShell](/documentation/articles/sql-database-import-powershell/)

## 其他资源

- [SQL 数据库 V12](/documentation/articles/sql-database-v12-whats-new/)
- [Transact-SQL 部分支持或不支持的函数](/documentation/articles/sql-database-transact-sql-information/)
- [使用 SQL Server 迁移助手迁移非 SQL Server 数据库](http://blogs.msdn.com/b/ssma/)

<!---HONumber=Mooncake_1010_2016-->