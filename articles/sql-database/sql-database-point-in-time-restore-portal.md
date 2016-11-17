<properties
	pageTitle="将 Azure SQL 数据库还原到之前的时间点（Azure 门户预览）| Azure"
	description="将 Azure SQL 数据库还原到之前的时间点。"
	services="sql-database"
	documentationCenter=""
	authors="stevestein"
	manager="jhubbard"
	editor=""/>

<tags
	ms.service="sql-database"
	ms.devlang="NA"
	ms.date="07/17/2016"
	wacn.date="09/26/2016"
	ms.author="sstein"
	ms.workload="NA"
	ms.topic="article"
	ms.tgt_pltfrm="NA"/>


# 使用 Azure 门户预览将 Azure SQL 数据库还原到之前的时间点


> [AZURE.SELECTOR]
- [概述](/documentation/articles/sql-database-recovery-using-backups/)
- [时间点还原：PowerShell](/documentation/articles/sql-database-point-in-time-restore-powershell/)

本文介绍如何使用 Azure 门户预览将数据库从 [SQL 数据库自动备份](/documentation/articles/sql-database-automated-backups/)还原到以前的时间点。

## 选择要还原到之前时间点的数据库

要在 Azure 门户预览中还原数据库，请执行以下操作：

1.	打开 [Azure 门户预览](https://portal.azure.cn)。
2.  在屏幕左侧选择“浏览”>“SQL 数据库”。
3.  导航到你要还原的数据库并选择它。
4.  在数据库边栏选项卡的顶部，选择“还原”：

    ![还原 Azure SQL 数据库](./media/sql-database-point-in-time-restore-portal/restore.png)

5.  指定数据库名称和时间点，然后单击“确定”：

    ![还原 Azure SQL 数据库](./media/sql-database-point-in-time-restore-portal/restore-details.png)



## 后续步骤

- 有关业务连续性概述和应用场景，请参阅[业务连续性概述](/documentation/articles/sql-database-business-continuity/)
- 若要了解 Azure SQL 数据库的自动备份，请参阅 [SQL 数据库自动备份](/documentation/articles/sql-database-automated-backups/)
- 若要了解如何使用自动备份进行恢复，请参阅[从服务启动的备份中还原数据库](/documentation/articles/sql-database-recovery-using-backups/)
- 若要了解更快的恢复选项，请参阅[活动异地复制](/documentation/articles/sql-database-geo-replication-overview/)
- 若要了解如何使用自动备份进行存档，请参阅[数据库复制](/documentation/articles/sql-database-copy/)

<!---HONumber=Mooncake_0919_2016-->