<properties 
   pageTitle="在 Azure PowerShell 中使用地域还原恢复 Azure SQL 数据库" 
   description="地域还原, Windows Azure SQL 数据库, 还原数据库, 恢复数据库, Azure PowerShell" 
   services="sql-database" 
   documentationCenter="" 
   authors="elfisher" 
   manager="jeffreyg" 
   editor="v-romcal"/>

<tags
   ms.service="sql-database"
   ms.date="07/24/2015"
   wacn.date="09/15/2015"/>

# 在 Azure PowerShell 中使用地域还原恢复 Azure SQL 数据库

> [AZURE.SELECTOR]
- [地域还原 - 门户](/documentation/articles/sql-database-geo-restore-tutorial-management-portal)
- [地域还原 - REST API](/documentation/articles/sql-database-geo-restore-tutorial-rest)   

## 概述

本教程说明如何在 [Azure PowerShell](/documentation/articles/install-configure-powershell) 中使用地域还原恢复 Azure SQL 数据库地域还原是针对所有基本、标准和高级 Azure SQL 数据库 服务层提供的核心灾难恢复保护。

## 限制和安全性

请参阅[在 Azure 门户中使用异地还原恢复 Azure SQL 数据库](/documentation/articles/sql-database-geo-restore-tutorial-management-portal)。

## 如何：在 Azure PowerShell 中使用异地还原恢复 Azure SQL 数据库

<!--<iframe src="http://channel9.msdn.com/Blogs/Windows-Azure/Restore-a-SQL-Database-Using-Geo-Restore-With-Microsoft-Azure-PowerShell/player" width="960" height="540" allowFullScreen frameBorder="0"></iframe>-->

必须使用基于证书的身份验证来运行以下 cmdlet。有关详细信息，请参阅[如何安装和配置 Azure PowerShell](/documentation/articles/install-configure-powershell#use-the-certificate-method) 中的*使用证书方法*。

1. 使用 [Get-AzureSqlRecoverableDatabase](http://msdn.microsoft.com/zh-cn/library/azure/dn720219.aspx) cmdlet 获取可恢复的数据库列表。指定以下参数：
	* 数据库所在的 **ServerName**。	

	`PS C:&gt;Get-AzureSqlRecoverableDatabase -ServerName "myserver"`

2. 使用 [Get-AzureSqlRecoverableDatabase](http://msdn.microsoft.com/zh-cn/library/azure/dn720219.aspx) cmdlet 选择你要从中进行恢复的数据库。指定以下参数：
	* 数据库所在的 **ServerName**。
	* 要从中进行恢复的数据库的 **DatabaseName**。

	`PS C:\>$Database = Get-AzureSqlRecoverableDatabase -ServerName "myserver" –DatabaseName “mydb”`
	 
3. 使用 [Start-AzureSqlDatabaseRecovery](http://msdn.microsoft.com/zh-cn/library/dn720224.aspx) cmdlet 开始恢复。指定以下参数：
	* 要恢复的 **SourceDatabase**。
	* 要恢复到的数据库的 **TargetDatabaseName**。
	* 要将数据库恢复到的 **TargetServerName**。

	将返回的结果存储在名为 **$RestoreRequest** 的变量中。此变量包含用于监视还原状态的还原请求 ID。

	`PS C:\>$RecoveryRequest = Start-AzureSqlDatabaseRecovery -SourceDatabase $Database –TargetDatabaseName “myrecoveredDB” –TargetServerName “mytargetserver”`
	
数据库恢复可能需要一段时间才能完成。若要监视恢复状态，请使用 [Get-AzureSqlDatabaseOperation](http://msdn.microsoft.com/zh-cn/library/azure/dn546738.aspx) cmdlet 并指定以下参数：

* 要还原到的数据库的 **ServerName**。
* **OperationGuid**，即执行步骤 3 时存储在 **$RecoveryRequest** 变量中的还原请求 ID。

	`PS C:\>Get-AzureSqlDatabaseOperation –ServerName “mytargetserver” –OperationGuid $RecoveryRequest.ID`

**State** 和 **PercentComplete** 字段显示还原状态。

## 后续步骤

有关详细信息，请参阅以下主题：

[在 Azure 门户中使用时间点还原来还原 Azure SQL 数据库](/documentation/articles/sql-database-point-in-time-restore-tutorial-management-portal)

[在 Azure 门户中还原已删除的 Azure SQL 数据库](/documentation/articles/sql-database-restore-deleted-database-tutorial-management-portal)

[Azure SQL 数据库 业务连续性](http://msdn.microsoft.com/zh-cn/library/azure/hh852669.aspx)

[Azure SQL 数据库 备份和还原](http://msdn.microsoft.com/zh-cn/library/azure/jj650016.aspx)

[Azure SQL 数据库异地还原（博客）](http://azure.microsoft.com/blog/2014/09/13/azure-sql-database-geo-restore/)

[Azure PowerShell](https://msdn.microsoft.com/zh-cn/library/azure/jj156055.aspx)

<!---HONumber=69-->