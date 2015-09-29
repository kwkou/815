<properties 
   pageTitle="使用地域还原和 REST API 恢复 Azure SQL 数据库" 
   description="地域还原, Windows Azure SQL 数据库, 还原数据库, 恢复数据库, REST API" 
   services="sql-database" 
   documentationCenter="" 
   authors="elfisher" 
   manager="jeffreyg" 
   editor="v-romcal"/>

<tags
   ms.service="sql-database"
   ms.date="07/24/2015"
   wacn.date="09/15/2015"/>

# 使用地域还原和 REST API 恢复 Azure SQL 数据库

> [AZURE.SELECTOR]
- [地域还原 - 门户](/documentation/articles/sql-database-geo-restore-tutorial-management-portal)
- [地域还原 - PowerShell](/documentation/articles/sql-database-geo-restore-tutorial-powershell)

## 概述

本教程说明如何使用 REST API 恢复 Azure SQL 数据库。此外还提供了更详细操作的链接。地域还原是针对所有基本、标准和高级 Azure SQL 数据库 服务层提供的核心灾难恢复保护。

## 限制和安全性

请参阅[在 Azure 门户中使用地域还原恢复 Azure SQL 数据库](/documentation/articles/sql-database-geo-restore-tutorial-management-portal)。

## 如何：使用 REST API 恢复 Azure SQL 数据库

1.	使用[列出可恢复的数据库](http://msdn.microsoft.com/zh-cn/library/azure/dn800984.aspx)操作获取可恢复数据库的列表。
	
2.	使用[获取可恢复的数据库](http://msdn.microsoft.com/zh-cn/library/azure/dn800985.aspx)操作获取你要恢复的数据库。
	
3.	使用[创建数据库恢复请求](http://msdn.microsoft.com/zh-cn/library/azure/dn800986.aspx)操作创建恢复请求。
	
4.	使用[数据库操作状态](http://msdn.microsoft.com/zh-cn/library/azure/dn720371.aspx)操作跟踪恢复状态。

## 后续步骤

有关详细信息，请参阅以下主题：

[在 Azure 门户中使用时间点还原来还原 Azure SQL 数据库](/documentation/articles/sql-database-point-in-time-restore-tutorial-management-portal)

[在 Azure 门户中还原已删除的 Azure SQL 数据库](/documentation/articles/sql-database-restore-deleted-database-tutorial-management-portal)

[Azure SQL 数据库 业务连续性](http://msdn.microsoft.com/zh-cn/library/azure/hh852669.aspx)

[Azure SQL 数据库 备份和还原](http://msdn.microsoft.com/zh-cn/library/azure/jj650016.aspx)

[Azure SQL 数据库异地还原（博客）](http://azure.microsoft.com/blog/2014/09/13/azure-sql-database-geo-restore/)

[服务管理 REST API 参考](https://msdn.microsoft.com/zh-cn/library/azure/ee460799.aspx)

<!---HONumber=69-->