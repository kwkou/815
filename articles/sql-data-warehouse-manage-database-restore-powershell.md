<properties
   pageTitle="在 Azure SQL 数据仓库中还原数据库 (PowerShell) | Azure"
   description="PowerShell 任务，用于还原 Azure SQL 数据仓库中实时的、已删除的或无法访问的数据库。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="elfisher"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="05/05/2016"
   wacn.date="06/13/2016"/>

# 备份和还原 Azure SQL 数据仓库中的一个数据库 (PowerShell)

> [AZURE.SELECTOR]
- [概述](/documentation/articles/sql-data-warehouse-overview-manage-database-restore/)
- [门户](/documentation/articles/sql-data-warehouse-manage-database-restore-portal/)
- [PowerShell](/documentation/articles/sql-data-warehouse-manage-database-restore-powershell/)
- [REST](/documentation/articles/sql-data-warehouse-manage-database-restore-rest-api/)

用于在 SQL 数据仓库中还原数据库的 PowerShell 任务。

本主题中的任务：

- 还原实时数据库
- 还原已删除的数据库
- 从另一个 Azure 地理区域还原不能访问的数据库

[AZURE.INCLUDE [SQL 数据仓库备份保留策略](../includes/sql-data-warehouse-backup-retention-policy)]


## 开始之前

### 验证你的 SQL 数据库 DTU 容量。 
由于SQL 数据仓库将还原到你的逻辑 SQL Server 上的新数据库，必须确保要还原到的 SQL Server 具有足够的 DTU 容量来容纳新数据库。有关[如何查看和提高 DTU 配额][]的详细信息，请参阅此博客文章。

### 安装 PowerShell

若要对 SQL 数据仓库使用 Azure PowerShell，需要安装 Azure PowerShell 1.0 或更高版本。可以通过运行 **Get-Module -ListAvailable -Name Azure** 来检查版本。可以通过 [Microsoft Web 平台安装程序][]安装最新版本。有关安装最新版本的详细信息，请参阅[如何安装和配置 Azure PowerShell][]。

## 还原实时数据库

若要从快照还原数据库，请使用 [Restore-AzureRmSqlDatabase][] PowerShell cmdlet。

1. 打开 Windows PowerShell。
2. 连接到你的 Azure 帐户，并列出与你的帐户关联的所有订阅。
3. 选择包含要还原的数据库的订阅。
4. 列出数据库的还原点。
5. 使用 RestorePointCreationDate 选取所需的还原点。
6. 将数据库还原到所需的还原点。
7. 验证已还原的数据库是否处于联机状态。

```

    $SubscriptionName="<YourSubscriptionName>"
    $ResourceGroupName="<YourResourceGroupName>"
    $ServerName="<YourServerNameWithoutURLSuffixSeeNote>"  # Without database.windows.net
    $DatabaseName="<YourDatabaseName>"
    $NewDatabaseName="<YourDatabaseName>"
    
    Login-AzureRmAccount -EnvironmentName AzureChinaCloud
    Get-AzureRmSubscription
    Select-AzureRmSubscription -SubscriptionName $SubscriptionName
    
    # List the last 10 database restore points
    ((Get-AzureRMSqlDatabaseRestorePoints -ResourceGroupName $ResourceGroupName -ServerName $ServerName -DatabaseName ($DatabaseName).RestorePointCreationDate)[-10 .. -1]
    
    # Or list all restore points
    Get-AzureRmSqlDatabaseRestorePoints -ResourceGroupName $ResourceGroupName -ServerName $ServerName -DatabaseName $DatabaseName
    
    # Get the specific database to restore
    $Database = Get-AzureRmSqlDatabase -ResourceGroupName $ResourceGroupName -ServerName $ServerName -DatabaseName $DatabaseName
    
    # Pick desired restore point using RestorePointCreationDate
    $PointInTime="<RestorePointCreationDate>"  
    
    # Restore database from a restore point
    $RestoredDatabase = Restore-AzureRmSqlDatabase –FromPointInTimeBackup –PointInTime $PointInTime -ResourceGroupName $Database.ResourceGroupName -ServerName $Database.$ServerName -TargetDatabaseName $NewDatabaseName –ResourceId $Database.ResourceID
    
    # Verify the status of restored database
    $RestoredDatabase.status

```

>[AZURE.NOTE] 对于服务器 foo.database.windows.net，请使用“foo”作为上述 Powershell cmdlet 中的 -ServerName。

完成还原后，你可以根据[确认已恢复的数据库][]指南来配置已恢复的数据库。

## 还原已删除的数据库

若要还原已删除的数据库，请使用 [Restore-AzureRmSqlDatabase][] cmdlet。

1. 打开 Windows PowerShell。
2. 连接到你的 Azure 帐户，并列出与你的帐户关联的所有订阅。
3. 选择包含要还原的已删除数据库的订阅。
4. 获取特定的已删除数据库。
5. 还原已删除的数据库。
6. 验证已还原的数据库是否处于联机状态。

```

    $SubscriptionName="<YourSubscriptionName>"
    $ResourceGroupName="<YourResourceGroupName>"
    $ServerName="<YourServerNameWithoutURLSuffixSeeNote>"  # Without database.windows.net
    $DatabaseName="<YourDatabaseName>"
    $NewDatabaseName="<YourDatabaseName>"
    
    Login-AzureRmAccount -EnvironmentName AzureChianCloud
    Get-AzureRmSubscription
    Select-AzureRmSubscription -SubscriptionName $SubscriptionName
    
    # Get the deleted database to restore
    $DeletedDatabase = Get-AzureRmSqlDeletedDatabaseBackup -ResourceGroupName $ResourceGroupNam -ServerName $ServerName -DatabaseName $DatabaseName
    
    # Restore deleted database
    $RestoredDatabase = Restore-AzureRmSqlDatabase –FromDeletedDatabaseBackup –DeletionDate $DeletedDatabase.DeletionDate -ResourceGroupName $DeletedDatabase.ResourceGroupName -ServerName $DeletedDatabase.ServerName -TargetDatabaseName $NewDatabaseName –ResourceId $DeletedDatabase.ResourceID
    
    # Verify the status of restored database
    $RestoredDatabase.status

```

>[AZURE.NOTE] 对于服务器 foo.database.windows.net，请使用“foo”作为上述 Powershell cmdlet 中的 -ServerName。

完成还原后，你可以根据[确认已恢复的数据库][]指南来配置已恢复的数据库。

## 从 Azure 地理区域还原

若要恢复数据库，请使用 [Restore-AzureRmSqlDatabase][] cmdlet。

1. 打开 Windows PowerShell。
2. 连接到你的 Azure 帐户，并列出与你的帐户关联的所有订阅。
3. 选择包含要还原的数据库的订阅。
4. 获取要恢复的数据库。
5. 创建对数据库的恢复请求。
6. 验证异地还原的数据库的状态。

```

    Login-AzureRmAccount -EnvironmentName AzureChianCloud
    Get-AzureRmSubscription
    Select-AzureRmSubscription -SubscriptionName "<Subscription_name>"
    
    # Get the database you want to recover
    $GeoBackup = Get-AzureRmSqlDatabaseGeoBackup -ResourceGroupName "<YourResourceGroupName>" -ServerName "<YourServerName>" -DatabaseName "<YourDatabaseName>"
    
    # Recover database
    $GeoRestoredDatabase = Restore-AzureRmSqlDatabase –FromGeoBackup -ResourceGroupName "<YourResourceGroupName>" -ServerName "<YourTargetServer>" -TargetDatabaseName "<NewDatabaseName>" –ResourceId $GeoBackup.ResourceID
    
    # Verify that the geo-restored database is online
    $GeoRestoredDatabase.status

```

### 执行异地还原后配置数据库
此清单可帮助你准备好将恢复的数据库投入生产。

1. **更新连接字符串**：验证客户端工具的连接字符串指向最近恢复的数据库。
2. **修改防火墙规则**：验证目标服务器上的防火墙规则，并确保已启用从客户端计算机或 Azure 到服务器以及最近恢复的数据库的连接。
3. **验证服务器登录名和数据库用户**：验证应用程序使用的所有登录名是否在托管已恢复数据库的服务器上存在。重新创建缺少的登录名，并向其授予对已恢复数据库的适当权限。 
4. **启用审核**：如果需要通过审核来访问数据库，则需要在恢复数据库后启用审核。

如果源数据库启用了 TDE，则已恢复的数据库将启用 TDE。


## 后续步骤
有关详细信息，请参阅 [Azure SQL 数据库业务连续性概述][]和[管理概述][]。

<!--Image references-->

<!--Article references-->
[Azure SQL 数据库业务连续性概述]: /documentation/articles/sql-database-business-continuity/
[确认已恢复的数据库]: /documentation/articles/sql-database-recovered-finalize/
[如何安装和配置 Azure PowerShell]: /documentation/articles/powershell-install-configure/
[管理概述]: /documentation/articles/sql-data-warehouse-overview-manage/

<!--MSDN references-->
[Create database restore request]: https://msdn.microsoft.com/zh-cn/library/azure/dn509571.aspx
[Database operation status]: https://msdn.microsoft.com/zh-cn/library/azure/dn720371.aspx
[Get restorable dropped database]: https://msdn.microsoft.com/zh-cn/library/azure/dn509574.aspx
[List restorable dropped databases]: https://msdn.microsoft.com/zh-cn/library/azure/dn509562.aspx
[Restore-AzureRmSqlDatabase]: https://msdn.microsoft.com/zh-cn/library/mt693390.aspx

<!--Blog references-->
[如何查看和提高 DTU 配额]: https://azure.microsoft.com/blog/azure-limits-quotas-increase-requests/

<!--Other Web references-->
[Azure Portal]: https://manage.windowsazure.cn/
[Microsoft Web 平台安装程序]: https://aka.ms/webpi-azps

<!---HONumber=Mooncake_0606_2016-->