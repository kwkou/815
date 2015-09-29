<properties 
   pageTitle="使用时间还原和 REST API 还原 Azure SQL 数据库" 
   description="时间点还原, Windows Azure SQL 数据库, 还原数据库, 恢复数据库, REST API" 
   services="sql-database" 
   documentationCenter="" 
   authors="elfisher" 
   manager="jeffreyg" 
   editor="v-romcal"/>

<tags
   ms.service="sql-database"
   ms.date="07/24/2015"
   wacn.date="09/15/2015"/>

# 使用时间还原和 REST API 还原 Azure SQL 数据库

> [AZURE.SELECTOR]
- [时间点还原 - 门户](/documentation/articles/sql-database-point-in-time-restore-tutorial-management-portal)
- [时间点还原 - PowerShell](/documentation/articles/sql-database-point-in-time-restore-tutorial-powershell) 

## 概述

本教程说明如何使用时间点还原和 REST API 来还原 Azure SQL 数据库。此外还提供了更详细操作的链接。

时间点还原会创建一个新的数据库。服务会根据还原时间点使用的备份自动选择服务层。请确保你在逻辑服务器上具有创建另一个数据库所需的可用配额。如果你想要请求增加配额，请联系 [Azure 支持](/support/contact/)。

## 限制和安全性

请参阅[在 Azure 门户中使用时间点还原来还原 Azure SQL 数据库](/documentation/articles/sql-database-point-in-time-restore-tutorial-management-portal)。

## 如何：使用 REST API 还原 Azure SQL 数据库

1.	使用[获取数据库](http://msdn.microsoft.com/zh-cn/library/azure/dn505708.aspx)操作获取你要还原的数据库。

2.	使用[创建数据库还原请求](http://msdn.microsoft.com/zh-cn/library/azure/dn509571.aspx)操作创建还原请求。
	
3.	使用[数据库操作状态](http://msdn.microsoft.com/zh-cn/library/azure/dn720371.aspx)操作跟踪还原请求。

## 后续步骤

有关详细信息，请参阅以下主题：

[Azure SQL 数据库 业务连续性](http://msdn.microsoft.com/zh-cn/library/azure/hh852669.aspx)

[Azure SQL 数据库 备份和还原](http://msdn.microsoft.com/zh-cn/library/azure/jj650016.aspx)

[Azure SQL 数据库时间点还原（博客）](http://azure.microsoft.com/blog/2014/10/01/azure-sql-database-point-in-time-restore/)

[服务管理 REST API 参考](https://msdn.microsoft.com/zh-cn/library/azure/ee460799.aspx)

<!---HONumber=69-->