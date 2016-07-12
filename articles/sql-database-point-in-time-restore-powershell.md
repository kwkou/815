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
    ms.date="05/10/2016"
    wacn.date="06/14/2016"/>

# 使用 PowerShell 将 Azure SQL 数据库还原到之前的时间点

> [AZURE.SELECTOR]
- [PowerShell](/documentation/articles/sql-database-point-in-time-restore-powershell/)

本文将向你说明如何使用 PowerShell 将数据库还原到以前的时间点。

[**时间点还原**](/documentation/articles/sql-database-point-in-time-restore/)是自助服务功能，允许你将数据库从我们为所有数据库进行的自动备份还原到数据库保留期内的任何时间点。若要了解有关自动备份和数据库保留期的详细信息，请参阅[业务连续性概述](/documentation/articles/sql-database-business-continuity/)。

[AZURE.INCLUDE [启动 PowerShell 会话](../includes/sql-database-powershell.md)]

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

- [确认已恢复的 Azure SQL 数据库](/documentation/articles/sql-database-recovered-finalize/)
- [使用 SQL Server Management Studio 连接到 SQL 数据库并执行示例 T-SQL 查询](/documentation/articles/sql-database-connect-query-ssms/)


## 其他资源

- [业务连续性概述](/documentation/articles/sql-database-business-continuity/)
- [SQL 数据库文档](/documentation/services/sql-databases)

<!---HONumber=Mooncake_0530_2016-->
