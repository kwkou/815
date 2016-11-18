<properties
   pageTitle="还原 Azure SQL 数据仓库 (REST API) | Azure"
   description="用于还原 SQL 数据仓库的 REST API 任务。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="Lakshmi1812"
   manager="barbkess"
   editor=""/>  


<tags
   ms.service="sql-data-warehouse"
   ms.devlang="NA"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="data-services"
   ms.date="09/21/2016"
   wacn.date="10/31/2016"
   ms.author="lakshmir;barbkess;sonyama"/>  


# 还原 Azure SQL 数据仓库 (REST API)

> [AZURE.SELECTOR]
- [概述][]
- [门户][]
- [PowerShell][]
- [REST][]

在本文中，你将学习如何使用 REST API 还原 Azure SQL 数据仓库。

## 开始之前

**验证 DTU 容量。** 每个 SQL 数据仓库都由一个具有默认 DTU 配额的 SQL 服务器（例如 myserver.database.chinacloudapi.cn）托管。在还原 SQL 数据仓库之前，请确保 SQL Server 的剩余 DTU 配额足够进行数据库还原。

## 还原活动或暂停的数据库

还原数据库：

1. 使用“获取数据库还原点”操作获取数据库还原点列表。
2. 使用[创建数据库还原请求][]操作开始还原。
3. 使用[数据库操作状态][]操作跟踪还原状态。

>[AZURE.NOTE] 完成还原后，即可按 [Configure your database after recovery][]（在恢复后配置数据库）中的说明配置恢复的数据库。

## 还原已删除的数据库

还原已删除的数据库：

1.	使用[列出可还原的已删除数据库][]操作列出所有可还原的已删除数据库。
2.	使用[获取可还原的已删除数据库][]操作获取你要还原的已删除数据库的详细信息。
3.	使用[创建数据库还原请求][]操作开始还原。
4.	使用[数据库操作状态][]操作跟踪还原状态。

>[AZURE.NOTE] 若要在完成还原后配置数据库，请参阅 [Configure your database after recovery][]（在恢复后配置数据库）。


## 后续步骤
若要了解 Azure SQL 数据库版本的业务连续性功能，请阅读 [Azure SQL 数据库业务连续性概述][]。

<!--Image references-->


<!--Article references-->
[Azure SQL 数据库业务连续性概述]: /documentation/articles/sql-database-business-continuity/
[Configure your database after recovery]: /documentation/articles/sql-database-disaster-recovery/
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

<!--Other Web references-->
[Azure Portal]: https://portal.azure.cn/
[Microsoft Web Platform Installer]: https://aka.ms/webpi-azps

<!---HONumber=Mooncake_1024_2016-->