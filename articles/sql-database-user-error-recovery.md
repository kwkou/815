<properties 
   pageTitle="发生 SQL 数据库用户错误后进行恢复" 
   description="了解如何在发生用户错误、意外的数据损坏后进行恢复，或者使用 Azure SQL 数据库的时间点还原 (PITR) 功能还恢复已删除的数据库。" 
   services="sql-database" 
   documentationCenter="" 
   authors="elfisher" 
   manager="jhubbard" 
   editor="monicar"/>

<tags
   ms.service="sql-database"
   ms.date="06/16/2016"
   wacn.date="07/25/2016"/>

# 在发生用户错误后恢复 Azure SQL 数据库

Azure SQL 数据库提供两个核心功能，用于在发生用户错误或意外的数据修改后进行恢复。

- [时间点还原](/documentation/articles/sql-database-point-in-time-restore/)

- [还原已删除的数据库](/documentation/articles/sql-database-restore-deleted-database-powershell/)

Azure SQL 数据库始终会还原到新数据库。这些还原功能适用于所有基本、标准和高级数据库。

##时间点还原

在发生用户错误或意外数据修改的情况下，可以使用时间点还原，将数据库还原到数据库保留期内的任一时间点。


基本、标准和高级数据库的保留期分别为 7 天、14 天和 35 天。若要了解有关数据库备份保留期的详细信息，请参阅[自动备份](/documentation/articles/sql-database-automated-backups/)。


若要执行时间点还原，请参阅：



- [通过 PowerShell 进行时间点还原](/documentation/articles/sql-database-point-in-time-restore-powershell/)
- [通过 REST API 进行时间点还原 (createmode = PointInTimeRestore)](https://msdn.microsoft.com/zh-cn/library/azure/mt163685.aspx)


## 还原已删除的数据库

在删除了某个数据库的情况下，Azure SQL 数据库允许你将删除的数据库还原到删除时的时间点。Azure SQL 数据库将会根据数据库的保留期存储已删除的数据库备份。


已删除的数据库的保留期由该数据库尚未删除时所在的服务层或者数据库存在的天数确定（以两者中较小的为准）。若要了解有关数据库保留期的详细信息，请参阅[自动备份](/documentation/articles/sql-database-automated-backups/)。

还原已删除的数据库：


- [通过 PowerShell 还原已删除的数据库](/documentation/articles/sql-database-restore-deleted-database-powershell)

- [使用 REST API 还原已删除的数据库 (createmode=Restore)](https://msdn.microsoft.com/zh-cn/library/azure/mt163685.aspx)


## 后续步骤

- 有关对灾难恢复使用和配置活动异地复制的信息，请参阅[活动异地复制](/documentation/articles/sql-database-geo-replication-overview/)
- 有关对灾难恢复使用异地还原的信息，请参阅[异地还原](/documentation/articles/sql-database-geo-restore/)


## 其他资源

- [SQL 数据库业务连续性和灾难恢复](/documentation/articles/sql-database-business-continuity/)
- [时间点还原](/documentation/articles/sql-database-point-in-time-restore/)
- [异地还原](/documentation/articles/sql-database-geo-restore/)
- [活动异地复制](/documentation/articles/sql-database-geo-replication-overview/)
- [设计用于云灾难恢复的应用程序](/documentation/articles/sql-database-designing-cloud-solutions-for-disaster-recovery/)
- [确认已恢复的 Azure SQL 数据库](/documentation/articles/sql-database-recovered-finalize/)
- [异地复制的安全性配置](/documentation/articles/sql-database-geo-replication-security-config/)
- [SQL 数据库 BCDR 常见问题](/documentation/articles/sql-database-bcdr-faq/)

<!---HONumber=Mooncake_0718_2016-->