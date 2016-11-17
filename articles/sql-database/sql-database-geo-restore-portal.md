<properties
	pageTitle="从异地冗余备份还原 Azure SQL 数据库（Azure 门户预览）。| Azure"
	description="从异地冗余备份异地还原 Azure SQL 数据库（Azure 门户预览）。"
	services="sql-database"
	documentationCenter=""
	authors="stevestein"
	manager="jhubbard"
	editor=""/>

<tags
	ms.service="sql-database"
	ms.date="07/17/2016"
	wacn.date="09/19/2016" />


# 使用 Azure 门户预览从异地冗余备份异地还原 Azure SQL 数据库


> [AZURE.SELECTOR]
- [概述](/documentation/articles/sql-database-recovery-using-backups/)
- [异地还原：PowerShell](/documentation/articles/sql-database-geo-restore-powershell/)

本文演示了如何使用 Azure 门户预览通过异地还原将数据库还原到新服务器中。

## 选择要还原的数据库

要在 Azure 门户预览中还原数据库，请执行以下操作：

1.  打开 [Azure 门户预览](https://portal.azure.cn)。
2.  在屏幕左侧选择“新建”>“数据和存储”>“SQL 数据库”。
3.  选择“备份”作为源，然后选择要从中进行恢复的异地冗余备份。

    ![还原 Azure SQL 数据库](./media/sql-database-geo-restore-portal/geo-restore.png)

4.  指定数据库名称、要将数据库还原到其中的服务器，然后单击“创建”：


## 后续步骤

- 有关业务连续性概述和应用场景，请参阅[业务连续性概述](/documentation/articles/sql-database-business-continuity/)
- 若要了解 Azure SQL 数据库的自动备份，请参阅 [SQL 数据库自动备份](/documentation/articles/sql-database-automated-backups/)
- 若要了解如何使用自动备份进行恢复，请参阅[从服务启动的备份中还原数据库](/documentation/articles/sql-database-recovery-using-backups/)
- 若要了解更快的恢复选项，请参阅[活动异地复制](/documentation/articles/sql-database-geo-replication-overview/)
- 若要了解如何使用自动备份进行存档，请参阅[数据库复制](/documentation/articles/sql-database-copy/)

<!---HONumber=Mooncake_0912_2016-->