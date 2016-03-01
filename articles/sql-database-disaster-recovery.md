<properties 
   pageTitle="SQL 数据库灾难恢复" 
   description="了解在发生区域性的数据中心中断或故障后，如何使用 Azure SQL 数据库活动异地复制、标准异地复制和异地还原功能来恢复数据库。" 
   services="sql-database" 
   documentationCenter="" 
   authors="elfisher" 
   manager="jeffreyg" 
   editor="monicar"/>

<tags
   ms.service="sql-database"
   ms.date="11/09/2015"
   wacn.date="12/22/2015"/>

# 在中断后恢复 Azure SQL 数据库

Azure SQL 数据库提供以下功能，以便在服务中断后进行恢复：

- 活动异地复制[（博客）](http://azure.microsoft.com/blog/2014/07/12/spotlight-on-sql-database-active-geo-replication/)
- 标准异地复制[（博客）](http://azure.microsoft.com/blog/2014/09/03/azure-sql-database-standard-geo-replication/)
- 异地还原[（博客）](http://azure.microsoft.com/blog/2014/09/13/azure-sql-database-geo-restore/)
- 新的异地复制功能[（博客）](https://azure.microsoft.com/blog/azure-sql-database-geo-replication-october-2015-update/)

若要了解有关应对灾难以及何时恢复数据库的信息，请访问[业务连续性设计](/documentation/articles/sql-database-business-continuity-design)页。

##何时启动恢复 

恢复操作会影响应用程序。它要求更改 SQL 连接字符串，并可能会导致数据永久丢失。因此，仅当中断的持续时间超过了应用程序的 RTO 时，才应执行恢复操作。如果应用程序已部署到生产环境，你应该定期监视应用程序的运行状况，并使用以下数据点来声明有必要进行恢复：

1. 应用程序层与数据库之间的连接发生永久性故障。
2. Azure 门户显示了警报，指出区域中的某个事件会造成广泛的影响。

> [AZURE.NOTE]恢复数据库后，你可以根据[在恢复后配置数据库](/documentation/articles/#postrecovery)指南来配置该数据库，以便能够使用它。

## 故障转移到异地复制的辅助数据库
> [AZURE.NOTE]你必须配置一个用于故障转移的辅助数据库。异地复制仅适用于标准和高级数据库。了解[如何配置异地复制](sql-database-business-continuity-design)

###Azure 门户
使用 Azure 门户终止与异地复制的辅助数据库的连续复制关系。

1. 登录到 [Azure 门户](https://manage.windowsazure.cn)
2. 在屏幕左侧选择“浏览”，然后选择“SQL 数据库”
3. 导航到你的数据库并选择它。 
4. 在数据库边栏选项卡底部选择“异地复制映射”。
4. 在“辅助数据库”下，右键单击你要恢复到的数据库名称所在的行，然后选择“故障转移”。

###PowerShell
在 PowerShell 中使用 [Set-AzureRMSqlDatabaseSecondary](https://msdn.microsoft.com/zh-cn/library/mt619393.aspx) cmdlet 启动故障转移到异地复制的辅助数据库。
		
		$database = Get-AzureRMSqlDatabase –DatabaseName "mydb” –ResourceGroupName "rg2” –ServerName "srv2”
		$database | Set-AzureRMSqlDatabaseSecondary –Failover -AllowDataLoss

###REST API 
使用 REST 以编程方式启动故障转移到辅助数据库。

1. 使用[获取复制链接](https://msdn.microsoft.com/zh-cn/library/mt600778.aspx)操作获取特定辅助数据库的复制链接。
2. 使用[将辅助数据库设置为主数据库](https://msdn.microsoft.com/zh-cn/library/mt582027.aspx)，在允许数据丢失的情况下故障转移到辅助数据库。 

## 使用异地还原进行恢复

在某个数据库发生中断的情况下，你可以使用异地还原从该数据库的最新异地冗余备份恢复该数据库。

> [AZURE.NOTE]恢复数据库会创建一个新的数据库。必须确保要恢复到的服务器具有足够的 DTU，可以容纳新数据库的容量。你可以通过与支持人员联系来请求增加此配额。

###Azure 门户
若要使用 Azure 门户中的“异地还原”来还原 SQL 数据库，请使用以下步骤。

1. 登录到 [Azure 门户](https://manage.windowsazure.cn)
2. 在屏幕左侧选择“新建”，选择“数据和存储”，然后选择“SQL 数据库”。
2. 选择“备份”作为源，然后选择要从中进行恢复的异地冗余备份。
3. 指定余下的数据库属性，然后单击“创建”。
4. 数据库还原过程随即将会开始，你可以使用屏幕左侧的“通知”监视还原进度。

###PowerShell 
若要配合使用异地还原和 PowerShell 来还原 SQL 数据库，请使用 [start-AzureSqlDatabaseRecovery](https://msdn.microsoft.com/zh-cn/library/azure/dn720224.aspx) cmdlet 启动异地还原请求。

		$Database = Get-AzureSqlRecoverableDatabase -ServerName "ServerName" –DatabaseName “DatabaseToBeRecovered"
		$RecoveryRequest = Start-AzureSqlDatabaseRecovery -SourceDatabase $Database –TargetDatabaseName “NewDatabaseName” –TargetServerName “TargetServerName”
		Get-AzureSqlDatabaseOperation –ServerName "TargetServerName" –OperationGuid $RecoveryRequest.RequestID

###REST API 

使用 REST 以编程方式执行数据库恢复。

1.	使用[列出可恢复的数据库](http://msdn.microsoft.com/zh-cn/library/azure/dn800984.aspx)操作获取可恢复数据库的列表。
	
2.	使用[获取可恢复的数据库](http://msdn.microsoft.com/zh-cn/library/azure/dn800985.aspx)操作获取你要恢复的数据库。
	
3.	使用[创建数据库恢复请求](http://msdn.microsoft.com/zh-cn/library/azure/dn800986.aspx)操作创建恢复请求。
	
4.	使用[数据库操作状态](http://msdn.microsoft.com/zh-cn/library/azure/dn720371.aspx)操作跟踪恢复状态。
 
## 在恢复后配置数据库<a name="postrecovery"></a>

可以使用此任务清单来帮助你准备好将恢复的数据库投入生产。

### 更新连接字符串

验证应用程序的连接字符串指向最近恢复的数据库。如果存在以下情况之一，请更新你的连接字符串：

  + 已恢复的数据库使用的名称不同于源数据库名称
  + 已恢复的数据库所在的服务器不同于源服务器

有关更改连接字符串的详细信息，请参阅[与 Azure SQL 数据库的连接：中心建议](/documentation/articles/sql-database-connect-central-recommendations)。
 
### 修改防火墙规则
在服务器级别和数据库级别验证防火墙规则，并确保已启用从客户端计算机或 Azure 到服务器以及最近恢复的数据库的连接。有关详细信息，请参阅[如何：配置防火墙设置（Azure SQL 数据库）](/documentation/articles/sql-database-configure-firewall-settings)。

### 验证服务器登录名和数据库用户

验证应用程序使用的所有登录名是否在托管已恢复数据库的服务器上存在。重新创建缺少的登录名，并向其授予对已恢复数据库的适当权限。有关详细信息，请参阅[在 Azure SQL 数据库中管理数据库和登录名](/documentation/articles/sql-database-manage-logins)。

验证已恢复数据库中的每个数据库用户是否与有效的服务器登录名关联。使用 ALTER USER 语句将用户映射到有效的服务器登录名。有关详细信息，请参阅 [ALTER USER](http://go.microsoft.com/fwlink/?LinkId=397486)。


### 设置遥测警报

验证现有的警报规则设置是否映射到已恢复的数据库。如果存在以下情况之一，请更新设置：

  + 已恢复的数据库使用的名称不同于源数据库名称
  + 已恢复的数据库所在的服务器不同于源服务器

有关数据库警报规则的详细信息，请参阅[接收警报通知](/documentation/articles/insights-receive-alert-notifications)和[跟踪服务运行状况](/documentation/articles/insights-service-health)。


### 启用审核

如果需要通过审核来访问数据库，你需要在恢复数据库后启用审核。如果客户端应用程序使用了 *.database.secure.chinacloudapi.cn 模式的安全连接字符串，则就充分表明需要审核。有关详细信息，请参阅 [SQL 数据库审核入门](/documentation/articles/sql-database-auditing-get-started)。

<!---HONumber=Mooncake_1207_2015-->