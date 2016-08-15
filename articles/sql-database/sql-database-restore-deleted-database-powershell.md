<properties
	pageTitle="还原已删除的 Azure SQL 数据库 (PowerShell) | Azure"
	description="还原已删除的 Azure SQL 数据库 (PowerShell)。"
	services="sql-database"
	documentationCenter=""
	authors="stevestein"
	manager="jhubbard"
	editor=""/>

<tags
	ms.service="sql-database"
	ms.date="06/09/2016"
	wacn.date="07/25/2016" />


# 使用 PowerShell 还原已删除的 Azure SQL 数据库



[AZURE.INCLUDE [启动 PowerShell 会话](../../includes/sql-database-powershell.md)]


## 将已删除的数据库还原到独立数据库

1. 使用 [Get-AzureRmSqlDeletedDatabaseBackup](https://msdn.microsoft.com/zh-cn/library/azure/mt693387.aspx) cmdlet 获取你要还原的已删除数据库备份。

        $DeletedDatabase = Get-AzureRmSqlDeletedDatabaseBackup -ResourceGroupName "resourcegroup01" -ServerName "server01" -DatabaseName "database01"

2. 使用 [Restore-AzureRmSqlDatabase](https://msdn.microsoft.com/zh-cn/library/azure/mt693390.aspx) cmdlet 从已删除的数据库备份开始还原。
    
        Restore-AzureRmSqlDatabase –FromDeletedDatabaseBackup –DeletionDate $DeletedDatabase.DeletionDate -ResourceGroupName $DeletedDatabase.ResourceGroupName -ServerName $DeletedDatabase.ServerName -TargetDatabaseName "RestoredDatabase" –ResourceId $DeletedDatabase.ResourceID -Edition "Standard" -ServiceObjectiveName "S2"

## 将已删除的数据库还原到弹性数据库池中

1. 使用 [Get-AzureRmSqlDeletedDatabaseBackup](https://msdn.microsoft.com/zh-cn/library/azure/mt693387.aspx) cmdlet 获取你要还原的已删除数据库备份。

        $DeletedDatabase = Get-AzureRmSqlDeletedDatabaseBackup -ResourceGroupName "resourcegroup01" -ServerName "server01" -DatabaseName "database01"

2. 使用 [Restore-AzureRmSqlDatabase](https://msdn.microsoft.com/zh-cn/library/azure/mt693390.aspx) cmdlet 从已删除的数据库备份开始还原。
    
        Restore-AzureRmSqlDatabase –FromDeletedDatabaseBackup –DeletionDate $DeletedDatabase.DeletionDate -ResourceGroupName $DeletedDatabase.ResourceGroupName -ServerName $DeletedDatabase.ServerName -TargetDatabaseName "RestoredDatabase" –ResourceId $DeletedDatabase.ResourceID –ElasticPoolName "elasticpool01" 

## 后续步骤


- 有关如何还原已删除的数据库的信息，请参阅 [Restore a deleted database using the REST API（使用 REST API 还原已删除的数据库）](https://msdn.microsoft.com/zh-cn/library/azure/mt163685.aspx)。
- 有关 Azure SQL 数据库自动备份的详细信息，请参阅 [SQL 数据库自动备份](/documentation/articles/sql-database-automated-backups/)。

## 其他资源

- [时间点还原](/documentation/articles/sql-database-point-in-time-restore/)
- [业务连续性概述](/documentation/articles/sql-database-business-continuity/)
- [异地还原](/documentation/articles/sql-database-geo-restore/)
- [活动异地复制](/documentation/articles/sql-database-geo-replication-overview/)
- [设计用于云灾难恢复的应用程序](/documentation/articles/sql-database-designing-cloud-solutions-for-disaster-recovery/)




<!---HONumber=Mooncake_0718_2016-->