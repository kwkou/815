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
	ms.date="05/10/2016"
	wacn.date="06/14/2016" />


# 使用 PowerShell 还原已删除的 Azure SQL 数据库

本文将向你说明如何还原已删除的 Azure SQL 数据库。

在删除了某个数据库的情况下，Azure SQL 数据库允许你将删除的数据库还原到删除时的时间点。Azure SQL 数据库将会根据数据库的保留期存储已删除的数据库备份。

已删除的数据库的保留期由该数据库尚未删除时所在的服务层或者数据库存在的天数确定（以两者中较小的为准）。若要了解有关数据库保留期的详细信息，请参阅[业务连续性概述](/documentation/articles/sql-database-business-continuity/)。


[AZURE.INCLUDE [启动 PowerShell 会话](../includes/sql-database-powershell.md)]


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

- [确认已恢复的 Azure SQL 数据库](/documentation/articles/sql-database-recovered-finalize/)
- [使用 SQL Server Management Studio 连接到 SQL 数据库并执行示例 T-SQL 查询](/documentation/articles/sql-database-connect-query-ssms/)



## 其他资源

- [业务连续性概述](/documentation/articles/sql-database-business-continuity/)
- [SQL 数据库文档](/documentation/services/sql-databases)



<!---HONumber=Mooncake_0530_2016-->
