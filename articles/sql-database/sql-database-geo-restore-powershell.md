<properties
    pageTitle="PowerShell：从异地冗余备份还原 Azure SQL 数据库 | Azure"
    description="使用 PowerShell 从异地冗余备份中将 Azure SQL 数据库还原到新的服务器中"
    services="sql-database"
    documentationcenter=""
    author="stevestein"
    manager="jhubbard"
    editor="" />
<tags
    ms.service="sql-database"
    ms.custom="business continuity"
    ms.devlang="NA"
    ms.topic="article"
    ms.tgt_pltfrm="powershell"
    ms.workload="NA"
    ms.date="12/19/2016"
    wacn.date="03/24/2017"
    ms.author="sstein" />  



# 使用 PowerShell 从异地冗余备份中还原 Azure SQL 数据库

本文介绍如何通过 PowweShell 使用异地还原将数据库还原到新的服务器中。也可以[使用 Azure 门户预览](/documentation/articles/sql-database-geo-restore-portal/)执行此任务。

## 使用 PowerShell 从异地冗余备份还原 Azure SQL 数据库

[AZURE.INCLUDE [启动 PowerShell 会话](../../includes/sql-database-powershell.md)]

### 将数据库异地还原到独立的数据库中

1. 使用 [Get-AzureRmSqlDatabaseGeoBackup](https://msdn.microsoft.com/zh-cn/library/azure/mt693388(v=azure.300).aspx) cmdlet 获取要还原的数据库的异地冗余备份。
   
        $GeoBackup = Get-AzureRmSqlDatabaseGeoBackup -ResourceGroupName "resourcegroup01" -ServerName "server01" -DatabaseName "database01"

2. 借助 [Restore-AzureRmSqlDatabase](https://msdn.microsoft.com/zh-cn/library/azure/mt693390(v=azure.300).aspx) cmdlet，从异地冗余备份中开始还原。
    
        Restore-AzureRmSqlDatabase –FromGeoBackup -ResourceGroupName "TargetResourceGroup" -ServerName "TargetServer" -TargetDatabaseName "RestoredDatabase" –ResourceId $GeoBackup.ResourceID -Edition "Standard" -RequestedServiceObjectiveName "S2"

### 将数据库异地还原到弹性池中

1. 使用 [Get-AzureRmSqlDatabaseGeoBackup](https://msdn.microsoft.com/zh-cn/library/azure/mt693388(v=azure.300).aspx) cmdlet 获取要还原的数据库的异地冗余备份。

        $GeoBackup = Get-AzureRmSqlDatabaseGeoBackup -ResourceGroupName "resourcegroup01" -ServerName "server01" -DatabaseName "database01"

2. 借助 [Restore-AzureRmSqlDatabase](https://msdn.microsoft.com/zh-cn/library/azure/mt693390(v=azure.300).aspx) cmdlet，从异地冗余备份中开始还原。指定要将数据库还原到其中的池的名称。
    
        Restore-AzureRmSqlDatabase –FromGeoBackup -ResourceGroupName "TargetResourceGroup" -ServerName "TargetServer" -TargetDatabaseName "RestoredDatabase" –ResourceId $GeoBackup.ResourceID –ElasticPoolName "elasticpool01"  

## 后续步骤

- 有关业务连续性概述和应用场景，请参阅[业务连续性概述](/documentation/articles/sql-database-business-continuity/)。
- 若要了解 Azure SQL 数据库的自动备份，请参阅 [SQL 数据库自动备份](/documentation/articles/sql-database-automated-backups/)。
- 若要了解如何使用自动备份进行恢复，请参阅[从服务启动的备份中还原数据库](/documentation/articles/sql-database-recovery-using-backups/)。
- 若要了解更快的恢复选项，请参阅[活动异地复制](/documentation/articles/sql-database-geo-replication-overview/)。
- 若要了解如何使用自动备份进行存档，请参阅[数据库复制](/documentation/articles/sql-database-copy/)。

<!---HONumber=Mooncake_0320_2017-->
<!--Update_Description: link references update-->