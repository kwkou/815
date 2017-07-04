<properties
    pageTitle="将 Azure SQL 数据库还原到之前的时间点 | Azure"
    description="将 Azure SQL 数据库还原到之前的时间点"
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
    wacn.date="01/20/2017"
    ms.author="sstein; carlrab" />  


# 将 Azure SQL 数据库还原到之前的时间点 

本操作方法文章介绍如何将数据库从 [SQL 数据库自动备份](/documentation/articles/sql-database-automated-backups/)还原到以前的时间点。

## 使用 Azure 门户还原到上一时间点

> [AZURE.TIP]
有关教程，请参阅[开始使用备份和还原进行数据保护和恢复](/documentation/articles/sql-database-get-started-backup-recovery/)
>

选择要在 Azure 门户中还原的数据库：

1. 打开 [Azure 门户](https://portal.azure.cn)。
2. 在屏幕左侧，选择“更多服务”>“SQL 数据库”。
3. 选择要还原的数据库。
4. 在数据库页面的顶部，选择“还原”：
   
   ![还原 Azure SQL 数据库](./media/sql-database-point-in-time-restore-portal/restore.png)  

5. 在“还原”页上，选择要将数据库还原到的日期和时间（UTC 时间），然后单击“确定”：
   
   ![还原 Azure SQL 数据库](./media/sql-database-point-in-time-restore-portal/restore-details.png)  


6. 在上一步中单击“确定”后，单击页面右上方的通知图标，然后单击“还原 SQL 数据库”通知以显示详细信息。
   
    ![还原 Azure SQL 数据库](./media/sql-database-point-in-time-restore-portal/notification-icon.png)  

7. “还原 SQL 数据库”页面随即打开，显示有关还原状态的信息。可以单击行项查看更多详细信息：
   
    ![还原 Azure SQL 数据库](./media/sql-database-point-in-time-restore-portal/inprogress.png)  


## 使用 PowerShell 还原到上一时间点

[AZURE.INCLUDE [启动 PowerShell 会话](../../includes/sql-database-powershell-h3.md)]

### 将数据库还原到作为单一数据库的某个时间点
1. 使用 [Get-AzureRmSqlDatabase](https://msdn.microsoft.com/zh-cn/library/azure/mt603648(v=azure.300).aspx) cmdlet 获取要还原的数据库。
   
        $Database = Get-AzureRmSqlDatabase -ResourceGroupName "resourcegroup01" -ServerName "server01" -DatabaseName "database01"
2. 使用 [Restore-AzureRmSqlDatabase](https://msdn.microsoft.com/zh-cn/library/azure/mt693390(v=azure.300).aspx) cmdlet 将数据库还原到某个时间点。
   
        Restore-AzureRmSqlDatabase -FromPointInTimeBackup -PointInTime UTCDateTime -ResourceGroupName $Database.ResourceGroupName -ServerName $Database.ServerName -TargetDatabaseName "RestoredDatabase" -ResourceId $Database.ResourceID -Edition "Standard" -ServiceObjectiveName "S2"

### 将数据库还原到某个时间点的弹性池中
1. 使用 [Get-AzureRmSqlDatabase](https://msdn.microsoft.com/zh-cn/library/azure/mt603648(v=azure.300).aspx) cmdlet 获取要还原的数据库。
   
        $Database = Get-AzureRmSqlDatabase -ResourceGroupName "resourcegroup01" -ServerName "server01" -DatabaseName "database01"
2. 使用 [Restore-AzureRmSqlDatabase](https://msdn.microsoft.com/zh-cn/library/azure/mt693390(v=azure.300).aspx) cmdlet 将数据库还原到某个时间点。
   
        Restore-AzureRmSqlDatabase -FromPointInTimeBackup -PointInTime UTCDateTime -ResourceGroupName $Database.ResourceGroupName -ServerName $Database.ServerName -TargetDatabaseName "RestoredDatabase" -ResourceId $Database.ResourceID -ElasticPoolName "elasticpool01"

## 后续步骤
* 若要了解 Azure SQL 数据库的自动备份，请参阅 [SQL 数据库自动备份](/documentation/articles/sql-database-automated-backups/)
* 若要了解如何使用自动备份进行恢复，请参阅[从服务启动的备份中还原数据库](/documentation/articles/sql-database-recovery-using-backups/)
* 若要查看服务生成的数据库备份的最早还原点，请参阅[查看最早还原点](/documentation/articles/sql-database-view-oldest-restore-point/)
* 有关业务连续性概述和应用场景，请参阅[业务连续性概述](/documentation/articles/sql-database-business-continuity/)

<!---HONumber=Mooncake_0116_2017-->