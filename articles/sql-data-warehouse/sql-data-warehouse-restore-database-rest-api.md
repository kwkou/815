<properties
   pageTitle="还原 Azure SQL 数据仓库 (REST API) | Azure"
   description="用于还原 SQL 数据仓库的 REST API 任务。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="sonyama"
   manager="barbkess"
   editor=""/>  


<tags
   ms.service="sql-data-warehouse"
   ms.date="06/28/2016"
   wacn.date="08/22/2016"/>


# 还原 Azure SQL 数据仓库 (REST API)

> [AZURE.SELECTOR]
- [概述][]
- [门户][]
- [PowerShell][]
- [REST][]

在本文中，你将学习如何使用 REST API 还原 Azure SQL 数据仓库。

## 开始之前

**验证 DTU 容量。** 每个 SQL 数据仓库都由 SQL Server 逻辑服务器来承载。此逻辑服务器具有以 DTU 单位进行度量的容量限制。请务必确保承载数据库的 SQL Server 逻辑服务器对于所还原的数据库具有足够 DTU 容量，然后才能还原 SQL 数据仓库。有关[如何查看和提高 DTU 配额][]的详细信息，请参阅此博客文章。

## 还原活动或暂停的数据库

还原数据库：

1. 使用“获取数据库还原点”操作获取数据库还原点列表。
2. 使用[创建数据库还原请求][]操作开始还原。
3. 使用[数据库操作状态][]操作跟踪还原状态。

>[AZURE.NOTE] 完成还原后，你可以根据[确认已恢复的数据库][]指南来配置已恢复的数据库。

## 还原已删除的数据库

还原已删除的数据库：

1.	使用[列出可还原的已删除数据库][]操作列出所有可还原的已删除数据库。
2.	使用[获取可还原的已删除数据库][]操作获取你要还原的已删除数据库的详细信息。
3.	使用[创建数据库还原请求][]操作开始还原。
4.	使用[数据库操作状态][]操作跟踪还原状态。

>[AZURE.NOTE] 完成还原后，你可以根据[确认已恢复的数据库][]指南来配置已恢复的数据库。


## 后续步骤
若要了解 Azure SQL 数据库版本的业务连续性功能，请阅读 [Azure SQL 数据库业务连续性概述][]。

<!--Image references-->


<!--Article references-->
[Azure SQL 数据库业务连续性概述]: /documentation/articles/sql-database-business-continuity/
[确认已恢复的数据库]: /documentation/articles/sql-database-recovered-finalize/
[How to install and configure Azure PowerShell]: /documentation/articles/powershell-install-configure/
[概述]: /documentation/articles/sql-data-warehouse-restore-database-overview/
[门户]: /documentation/articles/sql-data-warehouse-restore-database-portal/
[PowerShell]: /documentation/articles/sql-data-warehouse-restore-database-powershell/
[REST]: /documentation/articles/sql-data-warehouse-restore-database-rest-api/

<!--MSDN references-->
[创建数据库还原请求]: https://msdn.microsoft.com/zh-cn/library/azure/dn509571.aspx
[数据库操作状态]: https://msdn.microsoft.com/zh-cn/library/azure/dn720371.aspx
[获取可还原的已删除数据库]: https://msdn.microsoft.com/zh-cn/library/azure/dn509574.aspx
[列出可还原的已删除数据库]: https://msdn.microsoft.com/zh-cn/library/azure/dn509562.aspx
[Restore-AzureRmSqlDatabase]: https://msdn.microsoft.com/zh-cn/library/mt693390.aspx


<!--Blog references-->
[如何查看和提高 DTU 配额]: https://azure.microsoft.com/blog/azure-limits-quotas-increase-requests/

<!--Other Web references-->

[Azure Portal]: https://portal.azure.cn/
[Microsoft Web Platform Installer]: https://aka.ms/webpi-azps

<!---HONumber=Mooncake_0815_2016-->