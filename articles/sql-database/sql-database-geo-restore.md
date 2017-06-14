<properties
    pageTitle="从异地冗余备份还原 Azure SQL 数据库 | Azure"
    description="使用 Azure 门户或 PowerShell 从异地冗余备份中将 Azure SQL 数据库还原到新的服务器中"
    services="sql-database"
    documentationcenter=""
    author="stevestein"
    manager="jhubbard"
    editor="" />
<tags
    ms.assetid="4b42bffa-f98c-406a-9a96-51721cc423d4"
    ms.service="sql-database"
    ms.custom="business continuity"
    ms.devlang="NA"
    ms.topic="article"
    ms.tgt_pltfrm="powershell"
    ms.workload="NA"
    ms.date="12/19/2016"
    wacn.date="01/20/2017"
    ms.author="sstein; carlrab" />  


# 从异地冗余备份还原 Azure SQL 数据库

本文演示了如何使用异地还原将数据库还原到新的服务器中。可以通过 Azure 门户或使用 PowerShell 完成此操作。

## 使用 Azure 门户从异地冗余备份中还原 Azure SQL 数据库

若要在 Azure 门户中异地还原数据库，请执行以下步骤：

1. 转到 [Azure 门户](https://portal.azure.cn)。
2. 在屏幕左侧选择“+新建”>“数据库”>“SQL 数据库”：
   
   ![还原 Azure SQL 数据库](./media/sql-database-geo-restore-portal/new-sql-database.png)  

3. 选择“备份”作为源，然后选择要还原的备份。指定数据库名称、要将数据库还原到其中的服务器，然后单击“创建”：
   
   ![还原 Azure SQL 数据库](./media/sql-database-geo-restore-portal/geo-restore.png)  


4. 单击页面右上方的通知图标，监视还原操作的状态。

## 使用 PowerShell 从异地冗余备份还原 Azure SQL 数据库

[AZURE.INCLUDE [启动 PowerShell 会话](../../includes/sql-database-powershell-h3.md)]

### 将数据库异地还原到独立的数据库中
1. 使用 [Get-AzureRmSqlDatabaseGeoBackup](https://msdn.microsoft.com/zh-cn/library/azure/mt693388(v=azure.300).aspx) cmdlet 获取要还原的数据库的异地冗余备份。
   
        $GeoBackup = Get-AzureRmSqlDatabaseGeoBackup -ResourceGroupName "resourcegroup01" -ServerName "server01" -DatabaseName "database01"
2. 借助 [Restore-AzureRmSqlDatabase](https://msdn.microsoft.com/zh-cn/library/azure/mt693390(v=azure.300).aspx) cmdlet，从异地冗余备份中开始还原。
   
        Restore-AzureRmSqlDatabase -FromGeoBackup -ResourceGroupName "TargetResourceGroup" -ServerName "TargetServer" -TargetDatabaseName "RestoredDatabase" -ResourceId $GeoBackup.ResourceID -Edition "Standard" -RequestedServiceObjectiveName "S2"

### 将数据库异地还原到弹性池中
1. 使用 [Get-AzureRmSqlDatabaseGeoBackup](https://msdn.microsoft.com/zh-cn/library/azure/mt693388(v=azure.300).aspx) cmdlet 获取要还原的数据库的异地冗余备份。
   
        $GeoBackup = Get-AzureRmSqlDatabaseGeoBackup -ResourceGroupName "resourcegroup01" -ServerName "server01" -DatabaseName "database01"
2. 借助 [Restore-AzureRmSqlDatabase](https://msdn.microsoft.com/zh-cn/library/azure/mt693390(v=azure.300).aspx) cmdlet，从异地冗余备份中开始还原。指定要将数据库还原到其中的池的名称。
   
        Restore-AzureRmSqlDatabase -FromGeoBackup -ResourceGroupName "TargetResourceGroup" -ServerName "TargetServer" -TargetDatabaseName "RestoredDatabase" -ResourceId $GeoBackup.ResourceID -ElasticPoolName "elasticpool01"  

## 后续步骤
* 有关业务连续性概述和应用场景，请参阅[业务连续性概述](/documentation/articles/sql-database-business-continuity/)。
* 若要了解 Azure SQL 数据库的自动备份，请参阅 [SQL 数据库自动备份](/documentation/articles/sql-database-automated-backups/)。
* 若要了解如何使用自动备份进行恢复，请参阅[从服务启动的备份中还原数据库](/documentation/articles/sql-database-recovery-using-backups/)。
* 若要了解更快的恢复选项，请参阅[活动异地复制](/documentation/articles/sql-database-geo-replication-overview/)。
* 若要了解如何使用自动备份进行存档，请参阅[数据库复制](/documentation/articles/sql-database-copy/)。

<!---HONumber=Mooncake_0116_2017-->