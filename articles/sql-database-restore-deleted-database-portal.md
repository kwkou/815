<properties
	pageTitle="还原已删除的 Azure SQL 数据库（Azure 门户）| Azure"
	description="还原已删除的 Azure SQL 数据库（Azure 门户）。"
	services="sql-database"
	documentationCenter=""
	authors="stevestein"
	manager="jhubbard"
	editor=""/>

<tags
	ms.service="sql-database"
	ms.date="05/10/2016"
	wacn.date="06/14/2016"


# 使用 Azure 门户还原已删除的 Azure SQL 数据库


> [AZURE.SELECTOR]
- [Azure 门户](/documentation/articles/sql-database-restore-deleted-database-portal)
- [PowerShell](/documentation/articles/sql-database-restore-deleted-database-powershell)

本文将向你说明如何还原已删除的 Azure SQL 数据库。

在删除了某个数据库的情况下，Azure SQL 数据库允许你将删除的数据库还原到删除时的时间点。Azure SQL 数据库将会根据数据库的保留期存储已删除的数据库备份。

已删除的数据库的保留期由该数据库尚未删除时所在的服务层或者数据库存在的天数确定（以两者中较小的为准）。若要了解有关数据库保留期的详细信息，请阅读[业务连续性概述](/documentation/articles/sql-database-business-continuity)。

## 选择要还原的数据库 

要在 Azure 门户中还原数据库，请执行以下操作：

1.	打开 [Azure 门户](https://portal.azure.cn)。
2.  在屏幕左侧选择“浏览”>“SQL 服务器”。
3.  导航到具有你要还原的已删除数据库的服务器并选择该服务器
4.  向下滚动到服务器边栏选项卡的“操作”部分并选择“已删除的数据库”：
	![还原 Azure SQL 数据库](./media/sql-database-restore-deleted-database-portal/restore-deleted-trashbin.png)
5.  选择要还原的已删除数据库。
6.  指定数据库名称，然后单击“确定”：

    ![还原 Azure SQL 数据库](./media/sql-database-restore-deleted-database-portal/restore-deleted.png)

## 后续步骤

- [确认已恢复的 Azure SQL 数据库](/documentation/articles/sql-database-recovered-finalize)
- [使用 SQL Server Management Studio 连接到 SQL 数据库并执行示例 T-SQL 查询](/documentation/articles/sql-database-connect-query-ssms)



## 其他资源

- [业务连续性概述](/documentation/articles/sql-database-business-continuity)
- [SQL 数据库文档](/documentation/services/sql-databases)



<!---HONumber=Mooncake_0530_2016-->
