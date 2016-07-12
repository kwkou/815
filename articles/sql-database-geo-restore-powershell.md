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
    ms.date="05/10/2016"
    wacn.date="06/14/2016"/>

# 使用 PowerShell 从异地冗余备份还原 Azure SQL 数据库

> [AZURE.SELECTOR]
- [PowerShell](/documentation/articles/sql-database-geo-restore-powershell/)

本文展示了如何使用 PowerShell 通过异地还原将数据库还原到新服务器中。

[异地还原](/documentation/articles/sql-database-geo-restore/)让你能够从异地冗余备份还原数据库以创建新数据库。可以在任意 Azure 区域中的任何服务器上创建数据库。由于它使用异地冗余备份作为其源，因此即使由于停电而无法访问数据库，也能够用其恢复数据库。将自动为所有服务层启用异地还原，而无需支付额外费用。

[AZURE.INCLUDE [启动 PowerShell 会话](../includes/sql-database-powershell.md)]

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

- [确认已恢复的 Azure SQL 数据库](/documentation/articles/sql-database-recovered-finalize/)
- [使用 SQL Server Management Studio 连接到 SQL 数据库并执行示例 T-SQL 查询](/documentation/articles/sql-database-connect-query-ssms/)


## 其他资源

- [业务连续性概述](/documentation/articles/sql-database-business-continuity/)
- [SQL 数据库文档](/documentation/services/sql-databases)

<!---HONumber=Mooncake_0530_2016-->
