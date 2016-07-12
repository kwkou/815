<properties
   pageTitle="在 Azure SQL 数据仓库中还原数据库（Azure 门户）| Azure"
   description="用于还原在 Azure SQL 数据仓库中实时、已删除的或无法访问的数据库的 REST API 任务。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="elfisher"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="05/05/2016"
   wacn.date="06/13/2016"/>

# 备份和还原 Azure SQL 数据仓库中的一个数据库 (REST API)

> [AZURE.SELECTOR]
- [概述](/documentation/articles/sql-data-warehouse-overview-manage-database-restore/)
- [门户](/documentation/articles/sql-data-warehouse-manage-database-restore-portal/)
- [PowerShell](/documentation/articles/sql-data-warehouse-manage-database-restore-powershell/)
- [REST](/documentation/articles/sql-data-warehouse-manage-database-restore-rest-api/)

用于在 SQL 数据仓库中还原数据库的 REST API 任务。

本主题中的任务：

- 还原实时数据库
- 还原已删除的数据库
- 从另一个 Azure 地理区域还原不能访问的数据库

## 开始之前

**验证你的 SQL 数据库 DTU 容量。** 由于SQL 数据仓库将还原到你的逻辑 SQL Server 上的新数据库，必须确保要还原到的 SQL Server 具有足够的 DTU 容量来容纳新数据库。有关[如何查看和提高 DTU 配额][]的详细信息，请参阅此博客文章。

## 还原实时数据库

还原数据库：

1. 使用“获取数据库还原点”操作获取数据库还原点列表。
2. 使用[创建数据库还原请求][]操作开始还原。
3. 使用[数据库操作状态][]操作跟踪还原状态。

>[AZURE.NOTE] 完成还原后，你可以根据[确认已恢复的数据库][]指南来配置已恢复的数据库。

## 还原已删除的数据库

还原已删除的数据库

1.	使用[列出可还原的已删除数据库][]操作列出所有可还原的已删除数据库。
2.	使用[获取可还原的已删除数据库][]操作获取你要还原的已删除数据库的详细信息。
3.	使用[创建数据库还原请求][]操作开始还原。
4.	使用[数据库操作状态][]操作跟踪还原状态。

>[AZURE.NOTE] 完成还原后，你可以根据[确认已恢复的数据库][]指南来配置已恢复的数据库。

## 从 Azure 地理区域还原

执行异地还原：

1. 使用 [列出可恢复的数据库][] 操作获取可恢复数据库的列表。
2. 使用 [获取可恢复的数据库][] 操作获取你要恢复的数据库。
3. 使用 [创建数据库恢复请求][] 操作创建恢复请求。
4. 使用[数据库操作状态][]操作跟踪恢复状态。

### 执行异地还原后配置数据库
此清单可帮助你准备好将恢复的数据库投入生产。

1. **更新连接字符串**：验证客户端工具的连接字符串指向最近恢复的数据库。
2. **修改防火墙规则**：验证目标服务器上的防火墙规则，并确保已启用从客户端计算机或 Azure 到服务器以及最近恢复的数据库的连接。
3. **验证服务器登录名和数据库用户**：验证应用程序使用的所有登录名是否在托管已恢复数据库的服务器上存在。重新创建缺少的登录名，并向其授予对已恢复数据库的适当权限。 
4. **启用审核**：如果需要通过审核来访问数据库，则需要在恢复数据库后启用审核。

如果源数据库启用了 TDE，则已恢复的数据库将启用 TDE。


## 后续步骤
若要了解 Azure SQL 数据库版本的业务连续性功能，请阅读 [Azure SQL Database business continuity overview][]（Azure SQL 数据库业务连续性概述）。

<!--Image references-->

<!--Article references-->
[Azure SQL Database business continuity overview]: /documentation/articles/sql-database-business-continuity/
[确认已恢复的数据库]: /documentation/articles/sql-database-recovered-finalize/
[How to install and configure Azure PowerShell]: /documentation/articles/powershell-install-configure/

<!--MSDN references-->
[创建数据库还原请求]: https://msdn.microsoft.com/zh-cn/library/azure/dn509571.aspx
[数据库操作状态]: https://msdn.microsoft.com/zh-cn/library/azure/dn720371.aspx
[获取可还原的已删除数据库]: https://msdn.microsoft.com/zh-cn/library/azure/dn509574.aspx
[列出可还原的已删除数据库]: https://msdn.microsoft.com/zh-cn/library/azure/dn509562.aspx
[Restore-AzureRmSqlDatabase]: https://msdn.microsoft.com/zh-cn/library/mt693390.aspx

<!--Blog references-->
[如何查看和提高 DTU 配额]: https://azure.microsoft.com/blog/azure-limits-quotas-increase-requests/

<!--Other Web references-->
[Azure Portal]: https://manage.windowsazure.cn/
[Microsoft Web Platform Installer]: https://aka.ms/webpi-azps

<!---HONumber=Mooncake_0606_2016-->