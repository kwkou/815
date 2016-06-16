<properties
	pageTitle="将 Azure SQL 数据库还原到之前的时间点（Azure 门户）| Azure"
	description="将 Azure SQL 数据库还原到之前的时间点。"
	services="sql-database"
	documentationCenter=""
	authors="stevestein"
	manager="jhubbard"
	editor=""/>

<tags
	ms.service="sql-database"
	ms.date="05/10/2016"
	wacn.date="06/14/2016"


# 使用 Azure 门户将 Azure SQL 数据库还原到之前的时间点


> [AZURE.SELECTOR]
- [Azure 门户](/documentation/articles/sql-database-point-in-time-restore-portal)
- [PowerShell](/documentation/articles/sql-database-point-in-time-restore-powershell)

本文将向你说明如何使用 Azure 门户将数据库还原到以前的时间点。

[**时间点还原**](/documentation/articles/sql-database-point-in-time-restore)是自助服务功能，允许你将数据库从我们为所有数据库进行的自动备份还原到数据库保留期内的任何时间点。若要了解有关自动备份和数据库保留期的详细信息，请参阅[业务连续性概述](/documentation/articles/sql-database-business-continuity)。

## 选择要还原到之前时间点的数据库

要在 Azure 门户中还原数据库，请执行以下操作：

1.	打开 [Azure 门户](https://portal.azure.cn)。
2.  在屏幕左侧选择“浏览”>“SQL 数据库”。
3.  导航到你要还原的数据库并选择它。
4.  在数据库边栏选项卡的顶部，选择“还原”：

    ![还原 Azure SQL 数据库](./media/sql-database-point-in-time-restore-portal/restore.png)

5.  指定数据库名称和时间点，然后单击“确定”：

    ![还原 Azure SQL 数据库](./media/sql-database-point-in-time-restore-portal/restore-details.png)


## 后续步骤

- [确认已恢复的 Azure SQL 数据库](/documentation/articles/sql-database-recovered-finalize)
- [使用 SQL Server Management Studio 连接到 SQL 数据库并执行示例 T-SQL 查询](/documentation/articles/sql-database-connect-query-ssms)



## 其他资源

- [业务连续性概述](/documentation/articles/sql-database-business-continuity)
- [SQL 数据库文档](/documentation/services/sql-databases)



<!---HONumber=Mooncake_0530_2016-->
