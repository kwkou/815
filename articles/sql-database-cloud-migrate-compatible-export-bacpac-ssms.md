
<properties
   pageTitle="使用 SSMS 将 SQL Server 数据库导出到 BACPAC 文件"
   description="Azure SQL 数据库, 数据库迁移, 导出数据库, 导出 BACPAC 文件,“导出数据层应用程序”向导"
   services="sql-database"
   documentationCenter=""
   authors="carlrabeler"
   manager="jeffreyg"
   editor=""/>

<tags
   ms.service="sql-database"
   ms.date="03/14/2016"
   wacn.date="03/24/2016"/>

# 使用 SSMS 将 SQL Server 数据库导出到 BACPAC 文件

> [AZURE.SELECTOR]
- [SSMS](/documentation/articles/sql-database-cloud-migrate-compatible-export-bacpac-ssms/)
- [SqlPackage](/documentation/articles/sql-database-cloud-migrate-compatible-export-bacpac-sqlpackage/)

 
本文说明如何使用 SQL Server Management Studio 中的“导出数据层应用程序”向导将 SQL Server 数据库导出到 [BACPAC](https://msdn.microsoft.com/zh-cn/library/ee210546.aspx#Anchor_4) 文件。

1. 确认你安装了最新版本的 SQL Server Management Studio。Management Studio 的新版本将每月更新一次，以与 Azure 门户的更新保持同步。

	 >[AZURE.IMPORTANT]建议始终使用最新版本的 Management Studio 以保持与 Azure 和 SQL 数据库的更新同步。[更新 SQL Server Management Studio](https://msdn.microsoft.com/zh-cn/library/mt238290.aspx)。

2. 打开 Management Studio 并连接到你在对象资源管理器中的源数据库。

	![通过“任务”菜单导出数据层应用程序](./media/sql-database-cloud-migrate/MigrateUsingBACPAC01.png)

3. 右键单击对象资源管理器中的源数据库，指向“任务”，然后单击“导出数据层应用程序...”

	![通过“任务”菜单导出数据层应用程序](./media/sql-database-cloud-migrate/TestForCompatibilityUsingSSMS01.png)

4. 在导出向导中，配置导出以将 BACPAC 文件保存到本地磁盘位置或 Azure blob。导出的 BACPAC 始终包括完整的数据库架构，默认情况下还包括所有表中的数据。如果你要排除部分或全部表中的数据，请使用“高级”选项卡。例如，你可以选择仅导出引用表的数据，而不是导出所有表中的数据。

	![导出设置](./media/sql-database-cloud-migrate/MigrateUsingBACPAC02.png)

## 下一步：从 BACPAC 文件导入到 SQL 数据库

- [SSMS](/documentation/articles/sql-database-cloud-migrate-compatible-import-bacpac-ssms/)
- [SqlPackage](/documentation/articles/sql-database-cloud-migrate-compatible-import-bacpac-sqlpackage/)
- [PowerShell](/documentation/articles/sql-database-import-powershell/)

<!---HONumber=Mooncake_0104_2016-->
