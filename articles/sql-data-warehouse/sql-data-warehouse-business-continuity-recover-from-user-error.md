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
   ms.date="03/03/2016"
   wacn.date="04/05/2016"/>

# 发生用户错误后在 SQL 数据仓库中恢复数据库

SQL 数据仓库提供两个核心功能，用于在发生导致意外数据损坏或删除的用户错误后进行恢复。

- 还原实时数据库
- 还原已删除的数据库

使用这两项功能可以还原到同一服务器上的新数据库。

有两种不同的 API 支持 SQL 数据仓库数据库还原：Azure PowerShell 和 REST API。可以使用其中任何一种方法访问 SQL 数据仓库还原功能。

## 恢复实时数据库
当用户错误造成意外的数据修改时，你可以在保留期内将数据库还原到任一还原点。实时数据库的数据库快照至少每 8 小时创建一次，并会保留 7 天。

### PowerShell

使用 Azure PowerShell 以编程方式执行数据库还原。若要下载 Azure PowerShell 模块，请运行 [Microsoft Web 平台安装程序](http://go.microsoft.com/fwlink/p/?linkid=320376&clcid=0x409)。可以通过运行 Get-Module -ListAvailable -Name Azure 来检查你的版本。本文基于 Azure PowerShell 版本 1.0.4。

若要还原数据库，请使用 [Start-AzureSqlDatabaseRestore][] cmdlet。

1. 打开 Windows PowerShell。
2. 连接到你的 Azure 帐户，并列出与你的帐户关联的所有订阅。
3. 选择包含要还原的数据库的订阅。
4. 列出数据库的还原点（需要 Azure 资源管理模式）。
5. 使用 RestorePointCreationDate 选取所需的还原点。
6. 将数据库还原到所需的还原点。
7. 监视还原进度。

```

	Login-AzureRmAccount –EnvironmentName AzureChinaCloud
	Get-AzureRmSubscription
	Select-AzureRmSubscription -SubscriptionName "<Subscription_name>"

	# List the last 10 database restore points
	((Get-AzureRMSqlDatabaseRestorePoints -ServerName "<YourServerName>" -DatabaseName "<YourDatabaseName>" -ResourceGroupName "<YourResourceGroupName>").RestorePointCreationDate)[-10 .. -1]

		# Or for all restore points
		Get-AzureRmSqlDatabaseRestorePoints -ServerName "<YourServerName>" -DatabaseName "<YourDatabaseName>" -ResourceGroupName "<YourResourceGroupName>"

	# Pick desired restore point using RestorePointCreationDate
	$PointInTime = "<RestorePointCreationDate>"

	# Get the specific database name to restore
	(Get-AzureRmSqlDatabase -ServerName "<YourServerName>" -ResourceGroupName "<YourResourceGroupName>").DatabaseName | where {$_ -ne "master" }
	#or
	Get-AzureRmSqlDatabase -ServerName "<YourServerName>" –ResourceGroupName "<YourResourceGroupName>"

	# Restore database
	$RestoreRequest = Start-AzureSqlDatabaseRestore -SourceServerName "<YourServerName>" -SourceDatabaseName "<YourDatabaseName>" -TargetDatabaseName "<NewDatabaseName>" -PointInTime $PointInTime

	# Monitor progress of restore operation
	Get-AzureSqlDatabaseOperation -ServerName "<YourServerName>" –OperationGuid $RestoreRequest.RequestID
```

请注意，如果服务器是 foo.database.chinacloudapi.cn，请使用“foo”作为上述 Powershell cmdlet 中的 -ServerName。

### REST API
使用 REST 以编程方式执行数据库还原。

1. 使用“获取数据库还原点”操作获取数据库还原点列表。
2. 使用[创建数据库还原请求][]操作开始还原。
3. 使用[数据库操作状态][]操作跟踪还原状态。

完成还原后，你可以根据[确认已恢复的数据库][]指南，来配置要使用的已恢复数据库。

## 恢复已删除的数据库
在删除了某个数据库的情况下，你可以将删除的数据库还原到删除时的时间。Azure SQL 数据仓库在删除数据库之前会创建数据库快照，并将其保留 7 天。

### PowerShell
使用 Azure PowerShell 以编程方式还原已删除的数据库。若要下载 Azure PowerShell 模块，请运行 [Microsoft Web 平台安装程序](http://go.microsoft.com/fwlink/p/?linkid=320376&clcid=0x409)。

若要还原已删除的数据库，请使用 [Start-AzureSqlDatabaseRestore][] cmdlet。

1. 打开 Azure PowerShell。
2. 连接到你的 Azure 帐户，并列出与你的帐户关联的所有订阅。
3. 选择包含要还原的已删除数据库的订阅。
4. 从已删除数据库列表中查找该数据库及其删除日期。

```
Get-AzureSqlDatabase -RestorableDropped -ServerName "<YourServerName>"
```

5. 获取特定的已删除数据库，然后开始还原。

```
$Database = Get-AzureSqlDatabase -RestorableDropped -ServerName "<YourServerName>" –DatabaseName "<YourDatabaseName>" -DeletionDate "1/01/2015 12:00:00 AM"

$RestoreRequest = Start-AzureSqlDatabaseRestore -SourceRestorableDroppedDatabase $Database –TargetDatabaseName "<NewDatabaseName>"

Get-AzureSqlDatabaseOperation –ServerName "<YourServerName>" –OperationGuid $RestoreRequest.RequestID
```

请注意，如果服务器是 foo.database.chinacloudapi.cn，请使用“foo”作为上述 Powershell cmdlet 中的 -ServerName。

### REST API
使用 REST 以编程方式执行数据库还原。

1.	使用[列出可还原的已删除数据库][]操作列出所有可还原的已删除数据库。
2.	使用[获取可还原的已删除数据库][]操作获取你要还原的已删除数据库的详细信息。
3.	使用[创建数据库还原请求][]操作开始还原。
4.	使用[数据库操作状态][]操作跟踪还原状态。

完成还原后，你可以根据[确认已恢复的数据库][]指南，来配置要使用的已恢复数据库。


## 后续步骤
若要详细了解其他 Azure SQL 数据库版本的业务连续性功能，请阅读 [Azure SQL 数据库业务连续性概述][]。


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

<!---HONumber=Mooncake_0328_2016-->