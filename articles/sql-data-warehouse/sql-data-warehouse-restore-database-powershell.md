<properties
   pageTitle="还原 Azure SQL 数据仓库 (PowerShell) | Azure"
   description="用于还原 SQL 数据仓库的 PowerShell 任务。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="elfisher"
   manager="barbkess"
   editor=""/>  


<tags
   ms.service="sql-data-warehouse"
   ms.date="06/28/2016"
   wacn.date="08/22/2016"/>  


# 还原 Azure SQL 数据仓库 (PowerShell)

> [AZURE.SELECTOR]
- [概述][]
- [门户][]
- [PowerShell][]
- [REST][]

在本文中，你将学习如何使用 PowerShell 还原 Azure SQL 数据仓库。

## 开始之前

### 验证你的 SQL 数据库 DTU 容量。 

每个 SQL 数据仓库都由 SQL Server 逻辑服务器来承载。此逻辑服务器具有以 DTU 单位进行度量的容量限制。请务必确保承载数据库的 SQL Server 逻辑服务器对于所还原的数据库具有足够 DTU 容量，然后才能还原 SQL 数据仓库。有关[如何查看和提高 DTU 配额][]的详细信息，请参阅此博客文章。

### 安装 PowerShell

若要对 SQL 数据仓库使用 Azure PowerShell，需要安装 Azure PowerShell 1.0 或更高版本。可以通过运行 **Get-Module -ListAvailable -Name AzureRM** 来检查版本。可以通过 [Microsoft Web 平台安装程序][]安装最新版本。有关安装最新版本的详细信息，请参阅[如何安装和配置 Azure PowerShell][]。

## 还原活动或暂停的数据库

若要从快照还原数据库，请使用 [Restore-AzureRmSqlDatabase][] PowerShell cmdlet。

1. 打开 Windows PowerShell。
2. 连接到你的 Azure 帐户，并列出与你的帐户关联的所有订阅。
3. 选择包含要还原的数据库的订阅。
4. 列出数据库的还原点。
5. 使用 RestorePointCreationDate 选取所需的还原点。
6. 将数据库还原到所需的还原点。
7. 验证已还原的数据库是否处于联机状态。


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


>[AZURE.NOTE] 完成还原后，你可以根据[确认已恢复的数据库][]指南来配置已恢复的数据库。


## 还原已删除的数据库

若要还原已删除的数据库，请使用 [Restore-AzureRmSqlDatabase][] cmdlet。

1. 打开 Windows PowerShell。
2. 连接到你的 Azure 帐户，并列出与你的帐户关联的所有订阅。
3. 选择包含要还原的已删除数据库的订阅。
4. 获取特定的已删除数据库。
5. 还原已删除的数据库。
6. 验证已还原的数据库是否处于联机状态。

        $SubscriptionName="<YourSubscriptionName>"
        $ResourceGroupName="<YourResourceGroupName>"
        $ServerName="<YourServerNameWithoutURLSuffixSeeNote>"  # Without database.windows.net
        $DatabaseName="<YourDatabaseName>"
        $NewDatabaseName="<YourDatabaseName>"
        
        Login-AzureRmAccount -EnvironmentName AzureChinaCloud
        Get-AzureRmSubscription
        Select-AzureRmSubscription -SubscriptionName $SubscriptionName
        
	# 获取要还原的已删除数据库
        $DeletedDatabase = Get-AzureRmSqlDeletedDatabaseBackup -ResourceGroupName $ResourceGroupNam -ServerName $ServerName -DatabaseName $DatabaseName
        
	# 还原已删除的数据库
        $RestoredDatabase = Restore-AzureRmSqlDatabase –FromDeletedDatabaseBackup –DeletionDate $DeletedDatabase.DeletionDate -ResourceGroupName $DeletedDatabase.ResourceGroupName -ServerName $DeletedDatabase.ServerName -TargetDatabaseName $NewDatabaseName –ResourceId $DeletedDatabase.ResourceID
        
	# 验证已还原的数据库的状态
        $RestoredDatabase.status


>[AZURE.NOTE] 完成还原后，你可以根据[确认已恢复的数据库][]指南来配置已恢复的数据库。


## 从 Azure 地理区域还原

若要恢复数据库，请使用 [Restore-AzureRmSqlDatabase][] cmdlet。

1. 打开 Windows PowerShell。
2. 连接到你的 Azure 帐户，并列出与你的帐户关联的所有订阅。
3. 选择包含要还原的数据库的订阅。
4. 获取要恢复的数据库。
5. 创建对数据库的恢复请求。
6. 验证异地还原的数据库的状态。



	Login-AzureRmAccount -EnvironmentName AzureChinaCloud
	Get-AzureRmSubscription
	Select-AzureRmSubscription -SubscriptionName "<Subscription_name>"

	# 获取要恢复的数据库
	$GeoBackup = Get-AzureRmSqlDatabaseGeoBackup -ResourceGroupName "<YourResourceGroupName>" -ServerName "<YourServerName>" -DatabaseName "<YourDatabaseName>"

	# 恢复数据库
	$GeoRestoredDatabase = Restore-AzureRmSqlDatabase –FromGeoBackup -ResourceGroupName "<YourResourceGroupName>" -ServerName "<YourTargetServer>" -TargetDatabaseName "<NewDatabaseName>" –ResourceId $GeoBackup.ResourceID

	# 验证异地还原的数据库是否处于联机状态
	$GeoRestoredDatabase.status



>[AZURE.NOTE] 完成还原后，你可以根据[确认已恢复的数据库][]指南来配置已恢复的数据库。


如果源数据库启用了 TDE，则已恢复的数据库将启用 TDE。


## 后续步骤
若要了解 Azure SQL 数据库版本的业务连续性功能，请阅读 [Azure SQL 数据库业务连续性概述][]。

<!--Image references-->


<!--Article references-->
[Azure SQL 数据库业务连续性概述]: /documentation/articles/sql-database-business-continuity/
[Finalize a recovered database]: /documentation/articles/sql-database-recovered-finalize/
[如何安装和配置 Azure PowerShell]: /documentation/articles/powershell-install-configure/
[概述]: /documentation/articles/sql-data-warehouse-restore-database-overview/
[门户]: /documentation/articles/sql-data-warehouse-restore-database-portal/
[PowerShell]: /documentation/articles/sql-data-warehouse-restore-database-powershell/
[REST]: /documentation/articles/sql-data-warehouse-restore-database-rest-api/
[确认已恢复的数据库]: /documentation/articles/sql-database-recovered-finalize/

<!--MSDN references-->
[Restore-AzureRmSqlDatabase]: https://msdn.microsoft.com/zh-cn/library/mt693390.aspx

<!--Blog references-->
[如何查看和提高 DTU 配额]: https://azure.microsoft.com/blog/azure-limits-quotas-increase-requests/

<!--Other Web references-->
[Azure Portal]: https://portal.azure.cn/
[Microsoft Web 平台安装程序]: https://aka.ms/webpi-azps

<!---HONumber=Mooncake_0815_2016-->