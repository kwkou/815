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
   ms.date="07/23/2015"
   wacn.date="09/15/2015"/>

# 在发生用户错误后恢复 Azure SQL 数据库

Azure SQL 数据库提供两个核心功能，用于在发生用户错误或意外的数据修改后进行恢复。

- 时间点还原 
- 还原已删除的数据库

可以从这篇[博客文章](http://azure.microsoft.com/blog/2014/10/01/azure-sql-database-point-in-time-restore/)了解有关这些功能的详细信息。

Azure SQL 数据库始终会还原到新数据库。这些还原功能适用于所有基本、标准和高级数据库。
##使用时间点还原进行恢复
在发生用户错误或意外数据修改的情况下，可以使用时间点还原，将数据库还原到数据库保留期内的任一时间点。

基本、标准和高级数据库的保留期分别为 7 天、14 天和 35 天。若要了解有关数据库保留期的详细信息，请阅读[业务连续性概述](/documentation/articles/sql-database-business-continuity)。

###Azure 门户
1. 登录到 [Azure 门户](https://manage.windowsazure.cn)
2. 在屏幕左侧选择“浏览”，然后选择“SQL 数据库”
3. 导航到你的数据库并选择它。
4. 在数据库边栏选项卡的顶部，选择“还原”。
5. 指定数据库名称和时间点，然后单击“创建”。
6. 数据库还原过程随即将会开始，你可以使用屏幕左侧的“通知”监视还原进度。

完成还原后，你可以根据[确认已恢复的数据库](/documentation/articles/sql-database-recovered-finalize)指南，来配置要使用的已恢复数据库。
###PowerShell
使用 PowerShell 以编程方式执行数据库还原。

若要使用时间点还原来还原数据库，请使用 [Start-AzureSqlDatabaseRestore](https://msdn.microsoft.com/zh-cn/library/dn720218.aspx?f=255&MSPPError=-2147217396) cmdlet。<!-- For a detailed walk through, please see our [how-to video](http://azure.microsoft.com/documentation/videos/restore-a-sql-database-using-point-in-time-restore-with-microsoft-azure-powershell/).-->

		$Database = Get-AzureSqlDatabase -ServerName "YourServerName" –DatabaseName “YourDatabaseName”
		$RestoreRequest = Start-AzureSqlDatabaseRestore -SourceDatabase $Database –TargetDatabaseName “NewDatabaseName” –PointInTime “2015-01-01 06:00:00”
		Get-AzureSqlDatabaseOperation –ServerName "YourServerName" –OperationGuid $RestoreRequest.RequestID
		 
完成还原后，你可以根据[确认已恢复的数据库](/documentation/articles/sql-database-recovered-finalize)指南，来配置要使用的已恢复数据库。

###REST API 
使用 REST 以编程方式执行数据库还原。

1. 使用[获取数据库](https://msdn.microsoft.com/zh-CN/library/azure/dn505708.aspx)操作获取你要还原的数据库。

2.	使用[创建数据库还原请求](https://msdn.microsoft.com/zh-CN/library/azure/dn509571.aspx)操作创建还原请求。
	
3.	使用[数据库操作状态](https://msdn.microsoft.com/zh-CN/library/azure/dn720371.aspx)操作跟踪还原请求。

完成还原后，你可以根据[确认已恢复的数据库](/documentation/articles/sql-database-recovered-finalize)指南，来配置要使用的已恢复数据库。

##恢复已删除的数据库
在删除了某个数据库的情况下，Azure SQL 数据库允许你将删除的数据库还原到删除时的时间点。Azure SQL 数据库将会根据数据库的保留期存储已删除的数据库备份。

已删除的数据库的保留期由该数据库尚未删除时所在的服务层或者数据库存在的天数确定（以两者中较小的为准）。若要了解有关数据库保留期的详细信息，请阅读[业务连续性概述](/documentation/articles/sql-database-business-continuity)。

###Azure 门户
1. 登录到 [Azure 门户](https://manage.windowsazure.cn)
2. 在屏幕左侧选择“浏览”，然后选择“SQL Sever”。
3. 导航到你的服务器并选择它。
4. 在服务器边栏选项卡上的“操作”下，选择“已删除的数据库”。
5. 选择要还原的已删除数据库。
6. 指定数据库名称，然后单击“创建”。
7. 数据库还原过程随即将会开始，你可以使用屏幕左侧的“通知”监视还原进度。

完成还原后，你可以根据[确认已恢复的数据库](/documentation/articles/sql-database-recovered-finalize)指南，来配置要使用的已恢复数据库。

###PowerShell
使用 PowerShell 以编程方式执行数据库还原。

若要还原已删除的数据库，请使用 [Start-AzureSqlDatabaseRestore](https://msdn.microsoft.com/zh-cn/library/dn720218.aspx?f=255&MSPPError=-2147217396) cmdlet。

1. 从已删除数据库列表中查找已删除的数据库及其删除日期。
		
		Get-AzureSqlDatabase -RestorableDropped -ServerName "YourServerName"

2. 获取特定的已删除数据库，然后开始还原。

		$Database = Get-AzureSqlDatabase -RestorableDropped -ServerName "YourServerName" -DatabaseName "YourDatabaseName" -DeletionDate "1/01/2015 12:00:00 AM""
		$RestoreRequest = Start-AzureSqlDatabaseRestore -SourceRestorableDroppedDatabase $Database -TargetDatabaseName "NewDatabaseName"
		Get-AzureSqlDatabaseOperation -ServerName "YourServerName" -OperationGuid $RestoreRequest.RequestID
		 
完成还原后，你可以根据[确认已恢复的数据库](/documentation/articles/sql-database-recovered-finalize)指南，来配置要使用的已恢复数据库。

###REST API 
使用 REST 以编程方式执行数据库还原。

1.	使用[列出可还原的已删除数据库](https://msdn.microsoft.com/zh-cn/library/azure/dn509562.aspx)操作列出所有可还原的已删除数据库。
	
2.	使用[获取可还原的已删除数据库](https://msdn.microsoft.com/zh-cn/library/azure/dn509574.aspx)操作获取你要还原的已删除数据库的详细信息。

3.	使用[创建数据库还原请求](https://msdn.microsoft.com/zh-cn/library/azure/dn509571.aspx)操作开始还原。
	
4.	使用[数据库操作状态](https://msdn.microsoft.com/zh-cn/library/azure/dn720371.aspx)操作跟踪还原状态。

完成还原后，你可以根据[确认已恢复的数据库](/documentation/articles/sql-database-recovered-finalize)指南，来配置要使用的已恢复数据库。
 

<!---HONumber=69-->