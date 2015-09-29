<properties 
   pageTitle="SQL 数据库灾难恢复" 
   description="了解在发生区域性的数据中心中断或故障后，如何使用 Azure SQL 数据库异地复制和异地还原功能来恢复数据库。" 
   services="sql-database" 
   documentationCenter="" 
   authors="elfisher" 
   manager="jeffreyg" 
   editor="monicar"/>

<tags
   ms.service="sql-database"
   ms.date="07/14/2015"
   wacn.date="09/15/2015"/>

# 在中断后恢复 Azure SQL 数据库

Azure SQL 数据库提供了许多的中断恢复功能：

- 活动异地复制[（博客）](http://azure.microsoft.com/blog/2014/07/12/spotlight-on-sql-database-active-geo-replication/)
- 标准异地复制[（博客）](http://azure.microsoft.com/blog/2014/09/03/azure-sql-database-standard-geo-replication/)
- 异地还原[（博客）](http://azure.microsoft.com/blog/2014/09/13/azure-sql-database-geo-restore/)

若要了解有关应对灾难以及何时恢复数据库的信息，请访问[业务连续性设计](/documentation/articles/sql-database-business-continuity-design)页。 

##何时启动恢复 

恢复操作会影响应用程序。它要求更改 SQL 连接字符串，并可能会导致数据永久丢失。因此，仅当中断的持续时间超过了应用程序的 RTO 时，才应执行恢复操作。如果应用程序已部署到生产环境，你应该定期监视应用程序的运行状况，并使用以下数据点来声明有必要进行恢复：

1. 应用程序层与数据库之间的连接发生永久性故障。
2. Azure 门户显示了警报，指出区域中的某个事件会造成广泛的影响。

## 故障转移到异地复制的辅助数据库
> [AZURE.NOTE]若要使用故障转移进行数据库恢复，必须配置[标准异地复制](https://msdn.microsoft.com/zh-cn/library/azure/dn758204.aspx)或[活动异地复制](https://msdn.microsoft.com/zh-cn/library/azure/dn741339.aspx)。异地复制仅适用于标准和高级数据库。

当主数据库发生中断时，你可以故障转移到辅助数据库以实现还原。为此，你需要强行终止连续复制关系。有关终止连续复制关系的完整说明，请转到[此处](https://msdn.microsoft.com/zh-cn/library/azure/dn741323.aspx)。



###Azure 门户
1. 登录到 [Azure 门户](https://manage.windowsazure.cn)
2. 在屏幕左侧选择“浏览”，然后选择“SQL 数据库”
3. 导航到你的数据库并选择它。 
4. 在数据库边栏选项卡底部选择“异地复制映射”。
4. 在“辅助数据库”下，右键单击你要恢复到的数据库名称所在的行，然后选择“停止”。

在终止连续复制关系后，你可以根据[确认已恢复的数据库](/documentation/articles/sql-database-recovered-finalize)指南，来配置要使用的已恢复数据库。
###PowerShell
使用 PowerShell 以编程方式执行数据库恢复。

若要终止与辅助数据库的关系，请使用 [Stop-AzureSqlDatabaseCopy](https://msdn.microsoft.com/zh-CN/library/dn720223) cmdlet。
		
		$myDbCopy = Get-AzureSqlDatabaseCopy -ServerName "SecondaryServerName" -DatabaseName "SecondaryDatabaseName"
		$myDbCopy | Stop-AzureSqlDatabaseCopy -ServerName "SecondaryServerName" -ForcedTermination
		 
在终止连续复制关系后，你可以根据[确认已恢复的数据库](/documentation/articles/sql-database-recovered-finalize)指南，来配置要使用的已恢复数据库。
###REST API 
使用 REST 以编程方式执行数据库恢复。

1. 使用[获取数据库副本](https://msdn.microsoft.com/zh-CN/library/azure/dn509570.aspx)操作获取数据库的连续副本。
2. 使用[停止数据库副本](https://msdn.microsoft.com/zh-CN/library/azure/dn509573.aspx)操作停止数据库的连续副本。在“停止数据库副本”请求 URI 中使用辅助服务器名称和数据库名称

 在终止连续复制关系后，你可以根据[确认已恢复的数据库](/documentation/articles/sql-database-recovered-finalize)指南，来配置要使用的已恢复数据库。
## 使用异地还原恢复

在某个数据库发生中断的情况下，你可以使用异地还原从该数据库的最新异地冗余备份恢复该数据库。

###Azure 门户
1. 登录到 [Azure 门户](https://manage.windowsazure.cn)
2. 在屏幕左侧选择“新建”，选择“数据和存储”，然后选择“SQL 数据库”。
2. 选择“备份”作为源，然后选择要从中进行恢复的异地冗余备份。
3. 指定余下的数据库属性，然后单击“创建”。
4. 数据库还原过程随即将会开始，你可以使用屏幕左侧的“通知”监视还原进度。

恢复数据库后，你可以根据[确认已恢复的数据库](/documentation/articles/sql-database-recovered-finalize)指南来配置该数据库，以便能够使用它。
###PowerShell 
使用 PowerShell 以编程方式执行数据库恢复。

若要启动异地还原请求，请使用 [Start-AzureSqlDatabaseRecovery](https://msdn.microsoft.com/zh-CN/library/azure/dn720224.aspx) cmdlet。

		$Database = Get-AzureSqlRecoverableDatabase -ServerName "ServerName" -DatabaseName "DatabaseToBeRecovered"
		$RecoveryRequest = Start-AzureSqlDatabaseRecovery -SourceDatabase $Database -TargetDatabaseName "NewDatabaseName" -TargetServerName "TargetServerName"
		Get-AzureSqlDatabaseOperation -ServerName "TargetServerName" -OperationGuid $RecoveryRequest.RequestID

恢复数据库后，你可以根据[确认已恢复的数据库](/documentation/articles/sql-database-recovered-finalize)指南来配置该数据库，以便能够使用它。
###REST API 

使用 REST 以编程方式执行数据库恢复。

1.	使用[列出可恢复的数据库](http://msdn.microsoft.com/zh-cn/library/azure/dn800984.aspx)操作获取可恢复数据库的列表。
	
2.	使用[获取可恢复的数据库](http://msdn.microsoft.com/zh-cn/library/azure/dn800985.aspx)操作获取你要恢复的数据库。
	
3.	使用[创建数据库恢复请求](http://msdn.microsoft.com/zh-cn/library/azure/dn800986.aspx)操作创建恢复请求。
	
4.	使用[数据库操作状态](http://msdn.microsoft.com/zh-cn/library/azure/dn720371.aspx)操作跟踪恢复状态。

恢复数据库后，你可以根据[确认已恢复的数据库](/documentation/articles/sql-database-recovered-finalize)指南来配置该数据库，以便能够使用它。

<!---HONumber=69-->