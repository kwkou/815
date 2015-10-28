<properties 
   pageTitle="使用 REST API 还原已删除的 Azure SQL 数据库" 
   description="Windows Azure SQL 数据库, 还原已删除的数据库, 恢复已删除的数据库, REST API" 
   services="sql-database" 
   documentationCenter="" 
   authors="elfisher" 
   manager="jeffreyg" 
   editor="v-romcal"/>

<tags
   ms.service="sql-database"
   ms.date="07/24/2015"
   wacn.date="09/15/2015"/>

# 使用 REST API 还原已删除的 Azure SQL 数据库

> [AZURE.SELECTOR]
- [还原已删除数据库 - 门户](/documentation/articles/sql-database-restore-deleted-database-tutorial-management-portal)
- [还原已删除数据库 - PowerShell](/documentation/articles/sql-database-restore-deleted-database-tutorial-powershell) 

## 概述

本教程说明如何使用 REST API 还原已删除的 Azure SQL 数据库。此外还提供了更详细操作的链接。

还原已删除的 Azure SQL 数据库会创建一个新的数据库。服务会根据还原时间点使用的备份自动选择服务层。请确保你在逻辑服务器上具有创建另一个数据库所需的可用配额。如果你想要请求增加配额，请联系 [Azure 支持](/support/contact/)。

## 限制和安全性

请参阅[在 Azure 门户中还原已删除的 Azure SQL 数据库](/documentation/articles/sql-database-restore-deleted-database-tutorial-management-portal)。

## 如何：使用 REST API 还原已删除的 Azure SQL 数据库

1.	使用[列出可还原的已删除数据库](https://msdn.microsoft.com/zh-cn/library/azure/dn509562.aspx)操作列出所有可还原的已删除数据库。
	
2.	使用[获取可还原的已删除数据库](http://msdn.microsoft.com/zh-cn/library/azure/dn509574.aspx)操作获取你要还原的已删除数据库的详细信息。

3.	使用[创建数据库还原请求](http://msdn.microsoft.com/zh-cn/library/azure/dn509571.aspx)操作开始还原。
	
4.	使用[数据库操作状态](http://msdn.microsoft.com/zh-cn/library/azure/dn720371.aspx)操作跟踪还原状态。

## 后续步骤

有关详细信息，请参阅以下主题：

[Azure SQL 数据库 业务连续性](http://msdn.microsoft.com/zh-cn/library/azure/hh852669.aspx)

[Azure SQL 数据库 备份和还原](http://msdn.microsoft.com/zh-cn/library/azure/jj650016.aspx)

[服务管理 REST API 参考](http://msdn.microsoft.com/zh-cn/library/azure/ee460799.aspx)

<!---HONumber=69-->