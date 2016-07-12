<properties
   pageTitle="发生用户错误后在 SQL 数据仓库中恢复数据库 | Azure"
   description="发生用户错误后在 SQL 数据仓库中恢复数据库的步骤。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="sahaj08"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="03/28/2016"
   wacn.date="05/23/2016"/>

# 发生用户错误后在 SQL 数据仓库中恢复数据库

SQL 数据仓库提供两个核心功能，用于在发生导致意外数据损坏或删除的用户错误后进行恢复。

- 还原实时数据库
- 还原已删除的数据库

使用这两项功能可以还原到同一服务器上的新数据库。必须确保要还原到的服务器具有足够的 DTU，可以容纳新数据库的容量。你可以通过[与支持人员联系][]来请求增加此配额。


## 恢复实时数据库
Azure SQL 数据仓库服务至少每隔 8 小时使用数据库快照来保护所有实时数据库，并将快照保留 7 天，为你提供一系列不同的还原点。当你暂停或删除数据库时仍将创建数据库快照，并保留 7 天。当用户错误造成意外的数据修改时，你可以在保留期内将数据库还原到任一还原点。

### PowerShell

使用 Azure PowerShell 以编程方式执行数据库还原。若要下载 Azure PowerShell 模块，请运行 [Microsoft Web 平台安装程序](http://go.microsoft.com/fwlink/p/?linkid=320376&clcid=0x409)。可以通过运行 Get-Module -ListAvailable -Name AzureRM.Sql 来检查你的版本。本文基于 Microsoft AzureRM.Sql PowerShell 版本 1.0.5。

若要还原数据库，请使用 [Restore-AzureRmSqlDatabase][] cmdlet。

1. 打开 Windows PowerShell。
2. 连接到你的 Azure 帐户，并列出与你的帐户关联的所有订阅。
3. 选择包含要还原的数据库的订阅。
4. 列出数据库的还原点。
5. 使用 RestorePointCreationDate 选取所需的还原点。
6. 将数据库还原到所需的还原点。
7. 验证已还原的数据库是否处于联机状态。

```

	Login-AzureRmAccount -EnvironmentName AzureChinaCloud
	Get-AzureRmSubscription
	Select-AzureRmSubscription -SubscriptionName "<Subscription_name>"

	# List the last 10 database restore points
	((Get-AzureRMSqlDatabaseRestorePoints -ResourceGroupName "<YourResourceGroupName>" -ServerName "<YourServerName>" -DatabaseName "<YourDatabaseName>").RestorePointCreationDate)[-10 .. -1]

	# Or list all restore points
	Get-AzureRmSqlDatabaseRestorePoints -ResourceGroupName "<YourResourceGroupName>" -ServerName "<YourServerName>" -DatabaseName "<YourDatabaseName>" 

	# Pick desired restore point using RestorePointCreationDate
	$PointInTime = "<RestorePointCreationDate>"

	# Get the specific database to restore
	$Database = Get-AzureRmSqlDatabase -ResourceGroupName "<YourResourceGroupName>" -ServerName "<YourServerName>" -DatabaseName "<YourDatabaseName>"

	# Restore database from a restore point
	$RestoredDatabase = Restore-AzureRmSqlDatabase –FromPointInTimeBackup –PointInTime $PointInTime -ResourceGroupName $Database.ResourceGroupName -ServerName $Database.ServerName -TargetDatabaseName "<NewDatabaseName>" –ResourceId $Database.ResourceID

	# Verify the status of restored database
	$RestoredDatabase.status

```

>[AZURE.NOTE] 对于服务器 foo.database.chinacloudapi.cn，请使用“foo”作为上述 Powershell cmdlet 中的 -ServerName。


### REST API
使用 REST 以编程方式执行数据库还原。

1. 使用“获取数据库还原点”操作获取数据库还原点列表。
2. 使用[创建数据库还原请求][]操作开始还原。
3. 使用[数据库操作状态][]操作跟踪还原状态。


>[AZURE.NOTE] 完成还原后，你可以根据[确认已恢复的数据库][]指南来配置已恢复的数据库。


## 恢复已删除的数据库
Azure SQL 数据仓库在删除数据库之前会创建数据库快照，并将其保留 7 天。在误删了某个数据库的情况下，你可以将删除的数据库还原到删除时的时间。

### PowerShell
使用 Azure PowerShell 以编程方式还原已删除的数据库。若要下载 Azure PowerShell 模块，请运行 [Microsoft Web 平台安装程序](http://go.microsoft.com/fwlink/p/?linkid=320376&clcid=0x409)。可以通过运行 Get-Module -ListAvailable -Name AzureRM.Sql 来检查你的版本。本文基于 AzureRM.Sql PowerShell 版本 1.0.5。

若要还原已删除的数据库，请使用 [Restore-AzureRmSqlDatabase][] cmdlet。

1. 打开 Windows PowerShell。
2. 连接到你的 Azure 帐户，并列出与你的帐户关联的所有订阅。
3. 选择包含要还原的已删除数据库的订阅。
4. 获取特定的已删除数据库。
5. 还原已删除的数据库。
6. 验证已还原的数据库是否处于联机状态。

```

	Login-AzureRmAccount -EnvironmentName AzureChinaCloud
	Get-AzureRmSubscription
	Select-AzureRmSubscription -SubscriptionName "<Subscription_name>"

	# Get the deleted database to restore
	$DeletedDatabase = Get-AzureRmSqlDeletedDatabaseBackup -ResourceGroupName "<YourResourceGroupName>" -ServerName "<YourServerName>" -DatabaseName "<YourDatabaseName>"

	# Restore deleted database
	$RestoredDatabase = Restore-AzureRmSqlDatabase –FromDeletedDatabaseBackup –DeletionDate $DeletedDatabase.DeletionDate -ResourceGroupName $DeletedDatabase.ResourceGroupName -ServerName $DeletedDatabase.ServerName -TargetDatabaseName "<NewDatabaseName>" –ResourceId $DeletedDatabase.ResourceID

	# Verify the status of restored database
	$RestoredDatabase.status

```

请注意，如果服务器是 foo.database.chinacloudapi.cn，请使用“foo”作为上述 Powershell cmdlet 中的 -ServerName。


### REST API
使用 REST 以编程方式执行数据库还原。

1.	使用[列出可还原的已删除数据库][]操作列出所有可还原的已删除数据库。
2.	使用[获取可还原的已删除数据库][]操作获取你要还原的已删除数据库的详细信息。
3.	使用[创建数据库还原请求][]操作开始还原。
4.	使用[数据库操作状态][]操作跟踪还原状态。


>[AZURE.NOTE] 完成还原后，你可以根据[确认已恢复的数据库][]指南来配置已恢复的数据库。


## 后续步骤
若要了解 Azure SQL 数据库版本的业务连续性功能，请阅读 [Azure SQL 数据库业务连续性概述][]。


<!--Image references-->

<!--Article references-->
[Azure SQL 数据库业务连续性概述]: /documentation/articles/sql-database-business-continuity/
[确认已恢复的数据库]: /documentation/articles/sql-database-recovered-finalize/

<!--MSDN references-->
[创建数据库还原请求]: http://msdn.microsoft.com/zh-cn/library/azure/dn509571.aspx
[数据库操作状态]: http://msdn.microsoft.com/zh-cn/library/azure/dn720371.aspx
[获取可还原的已删除数据库]: http://msdn.microsoft.com/zh-cn/library/azure/dn509574.aspx
[列出可还原的已删除数据库]: http://msdn.microsoft.com/zh-cn/library/azure/dn509562.aspx
[Start-AzureSqlDatabaseRestore]: https://msdn.microsoft.com/zh-cn/library/dn720218.aspx

<!--Other Web references-->
[Azure Portal]: https://manage.windowsazure.cn/
[与支持人员联系]: https://azure.microsoft.com/blog/azure-limits-quotas-increase-requests/

<!---HONumber=Mooncake_0516_2016-->