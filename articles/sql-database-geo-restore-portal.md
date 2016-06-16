<properties
	pageTitle="从异地冗余备份还原 Azure SQL 数据库（Azure 门户）。| Azure"
	description="从异地冗余备份异地还原 Azure SQL 数据库（Azure 门户）。"
	services="sql-database"
	documentationCenter=""
	authors="stevestein"
	manager="jhubbard"
	editor=""/>

<tags
	ms.service="sql-database"
	ms.date="05/10/2016"
	wacn.date="06/14/2016"


# 使用 Azure 门户从异地冗余备份异地还原 Azure SQL 数据库


> [AZURE.SELECTOR]
- [Azure 门户](/documentation/articles/sql-database-geo-restore-portal)
- [PowerShell](/documentation/articles/sql-database-geo-restore-powershell)

本文演示了如何使用 Azure 门户通过异地还原将数据库还原到新服务器中。

[异地还原](/documentation/articles/sql-database-geo-restore)让你能够从异地冗余备份还原数据库以创建新数据库。可以在任意 Azure 区域中的任何服务器上创建数据库。由于它使用异地冗余备份作为其源，因此即使由于停电而无法访问数据库，也能够用其恢复数据库。将自动为所有服务层启用异地还原，而无需支付额外费用。

## 选择要还原的数据库

要在 Azure 门户中还原数据库，请执行以下操作：

1.	打开 [Azure 门户](https://portal.azure.cn)。
2.  在屏幕左侧选择“新建”>“数据和存储”>“SQL 数据库”。
3.  选择“备份”作为源，然后选择要从中进行恢复的异地冗余备份。

    ![还原 Azure SQL 数据库](./media/sql-database-geo-restore-portal/geo-restore.png)

4.  指定数据库名称、要将数据库还原到其中的服务器，然后单击“创建”：

## 后续步骤

- [使用 SQL Server Management Studio 连接到 SQL 数据库并执行示例 T-SQL 查询](/documentation/articles/sql-database-connect-query-ssms)
- [灾难恢复练习](/documentation/articles/sql-database-disaster-recovery-drills)


## 其他资源

- [异地还原](/documentation/articles/sql-database-geo-restore)
- [业务连续性概述](/documentation/articles/sql-database-business-continuity)
- [SQL 数据库文档](/documentation/services/sql-databases)


<!---HONumber=Mooncake_0530_2016-->
