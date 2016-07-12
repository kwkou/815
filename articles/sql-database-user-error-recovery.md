<properties 
   pageTitle="发生 SQL 数据库用户错误后进行恢复" 
   description="了解如何在发生用户错误、意外的数据损坏后进行恢复，或者使用 Azure SQL 数据库的时间点还原 (PITR) 功能还恢复已删除的数据库。" 
   services="sql-database" 
   documentationCenter="" 
   authors="elfisher" 
   manager="jeffreyg" 
   editor="monicar"/>

<tags
   ms.service="sql-database"
   ms.date="02/09/2016"
   wacn.date="06/14/2016"/>

# 在发生用户错误后恢复 Azure SQL 数据库

Azure SQL 数据库提供两个核心功能，用于在发生用户错误或意外的数据修改后进行恢复。

- 时间点还原 
- 还原已删除的数据库

可以在这篇[博客文章](https://azure.microsoft.com/blog/2014/10/01/azure-sql-database-point-in-time-restore)中了解到有关这些功能的详细信息。

Azure SQL 数据库始终会还原到新数据库。这些还原功能适用于所有基本、标准和高级数据库。

##时间点还原
在发生用户错误或意外数据修改的情况下，可以使用时间点还原，将数据库还原到数据库保留期内的任一时间点。

基本、标准和高级数据库的保留期分别为 7 天、14 天和 35 天。若要了解有关数据库保留期的详细信息，请参阅[业务连续性概述](/documentation/articles/sql-database-business-continuity/)。

> [AZURE.NOTE] 还原数据库会创建一个新的数据库。必须确保要还原到的服务器具有足够的 DTU，可以容纳新数据库的容量。你可以通过[与支持人员联系](https://azure.microsoft.com/blog/azure-limits-quotas-increase-requests)来请求增加此配额。

###PowerShell
使用 PowerShell 以编程方式通过 [Start-AzureSqlDatabaseRestore](https://msdn.microsoft.com/zh-cn/library/dn720218.aspx?f=255&MSPPError=-2147217396) cmdlet 执行时间点还原操作。

> [AZURE.IMPORTANT] 本文包含的命令适用于最高版本为 1.0（但不含）的 Azure PowerShell。可以使用 **Get-Module azure | format-table version** 命令查看 Azure PowerShell 的版本。

		$Database = Get-AzureSqlDatabase -ServerName "YourServerName" –DatabaseName “YourDatabaseName”
		$RestoreRequest = Start-AzureSqlDatabaseRestore -SourceDatabase $Database –TargetDatabaseName “NewDatabaseName” –PointInTime “2015-01-01 06:00:00”
		Get-AzureSqlDatabaseOperation –ServerName "YourServerName" –OperationGuid $RestoreRequest.RequestID
		 

###REST API 
使用 REST 以编程方式执行数据库还原。

1. 使用[获取数据库](http://msdn.microsoft.com/zh-cn/library/azure/dn505708.aspx)操作获取你要还原的数据库。

2.	使用[创建数据库还原请求](http://msdn.microsoft.com/zh-cn/library/azure/dn509571.aspx)操作创建还原请求。
	
3.	使用[数据库操作状态](http://msdn.microsoft.com/zh-cn/library/azure/dn720371.aspx)操作跟踪还原请求。

##还原已删除的数据库
在删除了某个数据库的情况下，Azure SQL 数据库允许你将删除的数据库还原到删除时的时间点。Azure SQL 数据库将会根据数据库的保留期存储已删除的数据库备份。

已删除的数据库的保留期由该数据库尚未删除时所在的服务层或者数据库存在的天数确定（以两者中较小的为准）。若要了解有关数据库保留期的详细信息，请阅读[业务连续性概述](/documentation/articles/sql-database-business-continuity/)。

> [AZURE.NOTE] 还原数据库会创建一个新的数据库。必须确保要还原到的服务器具有足够的 DTU，可以容纳新数据库的容量。你可以通过[与支持人员联系](https://azure.microsoft.com/blog/azure-limits-quotas-increase-requests)来请求增加此配额。

###PowerShell
若要通过 PowerShell 还原已删除的数据库，请使用 [Start-AzureSqlDatabaseRestore](https://msdn.microsoft.com/zh-cn/library/dn720218.aspx?f=255&MSPPError=-2147217396) cmdlet。

1. 从已删除数据库列表中查找已删除的数据库及其删除日期。
		
		Get-AzureSqlDatabase -RestorableDropped -ServerName "YourServerName"

2. 获取特定的已删除数据库，然后开始还原。

		$Database = Get-AzureSqlDatabase -RestorableDropped -ServerName "YourServerName" –DatabaseName “YourDatabaseName” -DeletionDate "1/01/2015 12:00:00 AM""
		$RestoreRequest = Start-AzureSqlDatabaseRestore -SourceRestorableDroppedDatabase $Database –TargetDatabaseName “NewDatabaseName”
		Get-AzureSqlDatabaseOperation –ServerName "YourServerName" –OperationGuid $RestoreRequest.RequestID
		 

###REST API 
使用 REST 以编程方式执行数据库还原。

1.	使用[列出可还原的已删除数据库](http://msdn.microsoft.com/zh-cn/library/azure/dn509562.aspx)操作列出所有可还原的已删除数据库。
	
2.	使用[获取可还原的已删除数据库](http://msdn.microsoft.com/zh-cn/library/azure/dn509574.aspx)操作获取你要还原的已删除数据库的详细信息。

3.	使用[创建数据库还原请求](http://msdn.microsoft.com/zh-cn/library/azure/dn509571.aspx)操作开始还原。
	
4.	使用[数据库操作状态](http://msdn.microsoft.com/zh-cn/library/azure/dn720371.aspx)操作跟踪还原状态。

<!---HONumber=Mooncake_0606_2016-->
