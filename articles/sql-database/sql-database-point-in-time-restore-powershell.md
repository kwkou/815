<properties 
    pageTitle="将 Azure SQL 数据库还原到之前的时间点 (PowerShell) | Azure" 
    description="将 Azure SQL 数据库还原到之前的时间点" 
    services="sql-database" 
    documentationCenter="" 
    authors="stevestein" 
    manager="jhubbard" 
    editor=""/>

<tags
    ms.service="sql-database"
    ms.date="06/17/2016"
    wacn.date="07/18/2016"/>

# 使用 PowerShell 将 Azure SQL 数据库还原到之前的时间点

> [AZURE.SELECTOR]
- [概述](/documentation/articles/sql-database-point-in-time-restore/)
- [PowerShell](/documentation/articles/sql-database-point-in-time-restore-powershell/)

本文将向你说明如何使用 PowerShell 将数据库从 [SQL 数据库自动备份](/documentation/articles/sql-database-automated-backups/)还原到以前的时间点。

[AZURE.INCLUDE [启动 PowerShell 会话](../../includes/sql-database-powershell.md)]

## 将数据库还原到作为独立数据库的某个时间点

1. 使用 [Get-AzureRmSqlDatabase](https://msdn.microsoft.com/zh-cn/library/azure/mt603648.aspx) cmdlet 获取你要还原的数据库。

        $Database = Get-AzureRmSqlDatabase -ResourceGroupName "resourcegroup01" -ServerName "server01" -DatabaseName "database01"

2. 使用 [Restore-AzureRmSqlDatabase](https://msdn.microsoft.com/zh-cn/library/azure/mt693390.aspx) cmdlet 将数据库还原到某个时间点。
    
        Restore-AzureRmSqlDatabase –FromPointInTimeBackup –PointInTime UTCDateTime -ResourceGroupName $Database.ResourceGroupName -ServerName $Database.ServerName -TargetDatabaseName "RestoredDatabase" –ResourceId $Database.ResourceID -Edition "Standard" -ServiceObjectiveName "S2"


## 将数据库还原到某个时间点的弹性数据库池中
   
1. 使用 [Get-AzureRmSqlDatabase](https://msdn.microsoft.com/zh-cn/library/azure/mt603648.aspx) cmdlet 获取你要还原的数据库。

        $Database = Get-AzureRmSqlDatabase -ResourceGroupName "resourcegroup01" -ServerName "server01" -DatabaseName "database01"

2. 使用 [Restore-AzureRmSqlDatabase](https://msdn.microsoft.com/zh-cn/library/azure/mt693390.aspx) cmdlet 将数据库还原到某个时间点。
    
        Restore-AzureRmSqlDatabase –FromPointInTimeBackup –PointInTime UTCDateTime -ResourceGroupName $Database.ResourceGroupName -ServerName $Database.ServerName -TargetDatabaseName "RestoredDatabase" –ResourceId $Database.ResourceID –ElasticPoolName "elasticpool01"

## 后续步骤

- 有关如何使用 REST API 恢复到某个时间点的信息，请参阅 [Point-In-Time Restore using the REST API（使用 REST API 进行的时间点还原）](https://msdn.microsoft.com/zh-cn/library/azure/mt163685.aspx)。
- 有关时间点还原的概述，请参阅[时间点还原](/documentation/articles/sql-database-point-in-time-restore/)
- 有关如何从用户或应用程序错误进行恢复的完整讨论，请参阅[用户错误恢复](/documentation/articles/sql-database-user-error-recovery/)。

## 其他资源

- [业务连续性方案](/documentation/articles/sql-database-business-continuity-scenarios/)

<!---HONumber=Mooncake_0711_2016-->