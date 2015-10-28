<properties 
   pageTitle="在 Azure PowerShell 中使用时间还原来还原 Azure SQL 数据库" 
   description="时间点还原, Windows Azure SQL 数据库, 还原数据库, 恢复数据库, Azure PowerShell" 
   services="sql-database" 
   documentationCenter="" 
   authors="elfisher" 
   manager="jeffreyg" 
   editor="v-romcal"/>

<tags
   ms.service="sql-database"
   ms.date="07/24/2015"
   wacn.date="09/15/2015"/>

# 在 Azure PowerShell 中使用时间还原来还原 Azure SQL 数据库

> [AZURE.SELECTOR]
- [时间点还原 - 门户](/documentation/articles/sql-database-point-in-time-restore-tutorial-management-portal)
- [时间点还原 - REST API](/documentation/articles/sql-database-point-in-time-restore-tutorial-rest) 

## 概述

本教程说明如何在 [Azure PowerShell](/documentation/articles/install-configure-powershell) 中使用时间点还原来还原 Azure SQL 数据库。Azure SQL 数据库 针对基本、标准和高级服务层提供内置备份，以支持自助时间点还原。

时间点还原会创建一个新的数据库。服务会根据还原时间点使用的备份自动选择服务层。请确保你在逻辑服务器上具有创建另一个数据库所需的可用配额。如果你想要请求增加配额，请联系 [Azure 支持](/support/contact/)。

## 限制和安全性

请参阅[在 Azure 门户中使用时间点还原来还原 Azure SQL 数据库](/documentation/articles/sql-database-point-in-time-restore-tutorial-management-portal)。

## 如何：在 Azure PowerShell 中使用时间点还原来还原 Azure SQL 数据库


必须使用基于证书的身份验证来运行以下 cmdlet。有关详细信息，请参阅[如何安装和配置 Azure PowerShell](/documentation/articles/install-configure-powershell#use-the-certificate-method) 中的*使用证书方法*。

1. 使用 [Get-AzureSqlDatabase](http://msdn.microsoft.com/zh-cn/library/azure/dn546735.aspx) cmdlet 获取你要还原的数据库。指定以下参数：
	* 数据库所在的 **ServerName**。
	* 要还原的数据库的 **DatabaseName**。	

	`PS C:\>$Database = Get-AzureSqlDatabase -ServerName "myserver" –DatabaseName “mydb”`

2. 使用 [Start-AzureSqlDatabaseRestore](http://msdn.microsoft.com/zh-cn/library/azure/dn720218.aspx) cmdlet 开始还原。指定以下参数：
	* 要从中还原的 **SourceDatabase**。
	* 要还原到的数据库的 **TargetDatabaseName**。
	* 要还原到的 **PointInTime**。

	将返回的结果存储在名为 **$RestoreRequest** 的变量中。此变量包含用于监视还原状态的还原请求 ID。

	`PS C:\>$RestoreRequest = Start-AzureSqlDatabaseRestore -SourceDatabase $Database –TargetDatabaseName “myrestoredDB” –PointInTime “2015-01-01 06:00:00”`

还原可能需要一段时间才能完成。若要监视还原状态，请使用 [Get-AzureSqlDatabaseOperation](http://msdn.microsoft.com/zh-cn/library/azure/dn546738.aspx) cmdlet 并指定以下参数：

* 要还原到的数据库的 **ServerName**。
* **OperationGuid**，即执行步骤 2 时存储在 **$RestoreRequest** 变量中的还原请求 ID。

	`PS C:\>Get-AzureSqlDatabaseOperation –ServerName "myserver" –OperationGuid $RestoreRequest.RequestID`

**State** 和 **PercentComplete** 字段显示还原状态。

## 后续步骤

有关详细信息，请参阅以下主题：

[Azure SQL 数据库 业务连续性](http://msdn.microsoft.com/zh-cn/library/azure/hh852669.aspx)

[Azure SQL 数据库 备份和还原](http://msdn.microsoft.com/zh-cn/library/azure/jj650016.aspx)

[Azure SQL 数据库时间点还原（博客）](http://azure.microsoft.com/blog/2014/10/01/azure-sql-database-point-in-time-restore/)

[Azure PowerShell](https://msdn.microsoft.com/zh-cn/library/azure/jj156055.aspx)

<!---HONumber=69-->