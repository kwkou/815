<properties 
    pageTitle="从异地冗余备份还原 Azure SQL 数据库 (PowerShell) | Azure" 
    description="从异地冗余备份将 Azure SQL 数据库还原到新的服务器中" 
    services="sql-database" 
    documentationCenter="" 
    authors="stevestein" 
    manager="jhubbard" 
    editor=""/>

<tags
    ms.service="sql-database"
    ms.date="06/17/2016"
    wacn.date="07/18/2016"/>

# 使用 PowerShell 从异地冗余备份还原 Azure SQL 数据库


> [AZURE.SELECTOR]
- [概述](/documentation/articles/sql-database-geo-restore/)
- [PowerShell](/documentation/articles/sql-database-geo-restore-powershell/)

本文展示了如何使用 PowerShell 通过异地还原将数据库还原到新服务器中。

[AZURE.INCLUDE [启动 PowerShell 会话](../../includes/sql-database-powershell.md)]

## 将你的数据库异地还原到独立数据库

1. 使用 [Get-AzureRmSqlDatabaseGeoBackup](https://msdn.microsoft.com/zh-cn/library/azure/mt693388.aspx) cmdlet 获取你想要还原的数据库的异地冗余备份。

        $GeoBackup = Get-AzureRmSqlDatabaseGeoBackup -ResourceGroupName "resourcegroup01" -ServerName "server01" -DatabaseName "database01"

2. 使用 [Restore-AzureRmSqlDatabase](https://msdn.microsoft.com/zh-cn/library/azure/mt693390.aspx) 从异地冗余备份开始进行还原。
    
        Restore-AzureRmSqlDatabase –FromGeoBackup -ResourceGroupName "TargetResourceGroup" -ServerName "TargetServer" -TargetDatabaseName "RestoredDatabase" –ResourceId $GeoBackup.ResourceID -Edition "Standard" -RequestedServiceObjectiveName "S2"


## 将数据库异地还原到弹性数据库池中

1. 使用 [Get-AzureRmSqlDatabaseGeoBackup](https://msdn.microsoft.com/zh-cn/library/azure/mt693388.aspx) cmdlet 获取你想要还原的数据库的异地冗余备份。

        $GeoBackup = Get-AzureRmSqlDatabaseGeoBackup -ResourceGroupName "resourcegroup01" -ServerName "server01" -DatabaseName "database01"

2. 使用 [Restore-AzureRmSqlDatabase](https://msdn.microsoft.com/zh-cn/library/azure/mt693390.aspx) 从异地冗余备份开始进行还原。指定要将数据库还原到其中的池的名称。
    
        Restore-AzureRmSqlDatabase –FromGeoBackup -ResourceGroupName "TargetResourceGroup" -ServerName "TargetServer" -TargetDatabaseName "RestoredDatabase" –ResourceId $GeoBackup.ResourceID –ElasticPoolName "elasticpool01"  

## 后续步骤


- 有关从异地冗余备份还原 Azure SQL 数据库的详细信息，请参阅[使用 PowerShell 进行异地还原](/documentation/articles/sql-database-geo-restore/)
- 有关如何在中断后进行恢复的完整讨论，请参阅[在中断后恢复](/documentation/articles/sql-database-disaster-recovery/)


## 其他资源

- [业务连续性方案](/documentation/articles/sql-database-business-continuity-scenarios/)

<!---HONumber=Mooncake_0711_2016-->