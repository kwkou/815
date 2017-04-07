<properties
    pageTitle="PowerShell：将 Azure SQL 数据库还原到以前的时间点 | Azure"
    description="使用 PowerShell 将 Azure SQL 数据库还原到以前的时间点"
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
    ms.date="12/08/2016"
    wacn.date="04/06/2017"
    ms.author="sstein; carlrab" />

# 使用 PowerShell 将 Azure SQL 数据库还原到之前的时间点

本文介绍如何使用 PowerShell 将数据库从 [SQL 数据库自动备份](/documentation/articles/sql-database-automated-backups/)还原到以前的时间点。也可以[使用 Azure 门户预览](/documentation/articles/sql-database-point-in-time-restore-portal/)执行此任务。


## 使用 PowerShell 还原到上一时间点

[AZURE.INCLUDE [启动 PowerShell 会话](../../includes/sql-database-powershell.md)]

### 将数据库还原到作为单一数据库的某个时间点
1. 使用 [Get-AzureRmSqlDatabase](https://msdn.microsoft.com/zh-cn/library/azure/mt603648(v=azure.300).aspx) cmdlet 获取要还原的数据库。
   
        $Database = Get-AzureRmSqlDatabase -ResourceGroupName "resourcegroup01" -ServerName "server01" -DatabaseName "database01"

2. 使用 [Restore-AzureRmSqlDatabase](https://msdn.microsoft.com/zh-cn/library/azure/mt693390(v=azure.300).aspx) cmdlet 将数据库还原到某个时间点。
    
        Restore-AzureRmSqlDatabase –FromPointInTimeBackup –PointInTime UTCDateTime -ResourceGroupName $Database.ResourceGroupName -ServerName $Database.ServerName -TargetDatabaseName "RestoredDatabase" –ResourceId $Database.ResourceID -Edition "Standard" -ServiceObjectiveName "S2"

### 将数据库还原到某个时间点的弹性池中
1. 使用 [Get-AzureRmSqlDatabase](https://msdn.microsoft.com/zh-cn/library/azure/mt603648(v=azure.300).aspx) cmdlet 获取要还原的数据库。
   
        $Database = Get-AzureRmSqlDatabase -ResourceGroupName "resourcegroup01" -ServerName "server01" -DatabaseName "database01"

2. 使用 [Restore-AzureRmSqlDatabase](https://msdn.microsoft.com/zh-cn/library/azure/mt693390(v=azure.300).aspx) cmdlet 将数据库还原到某个时间点。
    
        Restore-AzureRmSqlDatabase –FromPointInTimeBackup –PointInTime UTCDateTime -ResourceGroupName $Database.ResourceGroupName -ServerName $Database.ServerName -TargetDatabaseName "RestoredDatabase" –ResourceId $Database.ResourceID –ElasticPoolName "elasticpool01"

## 后续步骤
- 若要了解 Azure SQL 数据库的自动备份，请参阅 [SQL 数据库自动备份](/documentation/articles/sql-database-automated-backups/)
- 若要了解如何使用自动备份进行恢复，请参阅[从服务启动的备份中还原数据库](/documentation/articles/sql-database-recovery-using-backups/)
- 有关业务连续性概述和应用场景，请参阅[业务连续性概述](/documentation/articles/sql-database-business-continuity/)

<!---HONumber=Mooncake_0320_2017-->
<!--Update_Description: overview content refine; link references update and clean-->