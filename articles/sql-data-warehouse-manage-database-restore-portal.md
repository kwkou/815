<properties
   pageTitle="在 Azure SQL 数据仓库中还原数据库（Azure 门户）| Azure"
   description="Azure 门户任务，用于还原 Azure SQL 数据仓库中实时的、已删除的或无法访问的数据库。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="elfisher"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.date="05/05/2016"
   wacn.date="06/13/2016"/>

# 备份和还原 Azure SQL 数据仓库中的一个数据库（Azure 门户）

> [AZURE.SELECTOR]
- [概述](/documentation/articles/sql-data-warehouse-overview-manage-database-restore/)
- [门户](/documentation/articles/sql-data-warehouse-manage-database-restore-portal/)
- [PowerShell](/documentation/articles/sql-data-warehouse-manage-database-restore-powershell/)
- [REST](/documentation/articles/sql-data-warehouse-manage-database-restore-rest-api/)

用于在 SQL 数据仓库中还原数据库的 Azure 门户任务。

本主题中的任务：

- 还原实时数据库
- 还原已删除的数据库
- 从另一个 Azure 地理区域还原不能访问的数据库


## 开始之前

验证你的 SQL 数据库 DTU 容量。由于SQL 数据仓库将还原到你的逻辑 SQL Server 上的新数据库，必须确保要还原到的 SQL Server 具有足够的 DTU 容量来容纳新数据库。有关[如何查看和提高 DTU 配额][]的详细信息，请参阅此博客文章。


## 还原实时数据库

还原数据库：

1. 登录到 [Azure 门户][]。
2. 在屏幕左侧选择“浏览”，然后选择“SQL 数据库”。
3. 导航到你的数据库并选择它。
4. 在数据库边栏选项卡顶部，单击“还原”。
5. 指定新的**数据库名称**，选择一个**还原点**，然后单击“创建”。
6. 数据库还原过程随即将会开始，你可以使用“通知”监视还原进度。


## 还原已删除的数据库

还原已删除的数据库：

1. 登录到 [Azure 门户][]。
2. 在屏幕左侧选择“浏览”，然后选择“SQL Sever”。
3. 导航到你的服务器并选择它。
4. 在服务器的边栏选项卡上向下滚动到“操作”，然后单击“删除的数据库”磁贴。
5. 选择要还原的已删除数据库。
5. 指定新的**数据库名称**，然后单击“创建”。
6. 数据库还原过程随即将会开始，你可以使用“通知”监视还原进度。


## 从 Azure 地理区域还原

执行异地还原：

1. 登录到 [Azure 门户][]
2. 在屏幕左侧选择“+新建”，选择“数据和存储”，然后选择“SQL 数据仓库”
3. 选择“备份”作为源，然后选择要从中进行恢复的异地冗余备份
4. 指定余下的数据库属性，然后单击“创建”
5. 数据库还原过程随即将会开始，你可以使用“通知”监视还原进度

## 后续步骤
有关详细信息，请参阅 [Azure SQL 数据库业务连续性概述][]和[管理概述][]。

<!--Image references-->

<!--Article references-->
[Azure SQL 数据库业务连续性概述]: /documentation/articles/sql-database-business-continuity/
[Finalize a recovered database]: /documentation/articles/sql-database-recovered-finalize/
[How to install and configure Azure PowerShell]: /documentation/articles/powershell-install-configure/
[管理概述]: /documentation/articles/sql-data-warehouse-overview-manage/

<!--MSDN references-->

<!--Blog references-->
[如何查看和提高 DTU 配额]: https://azure.microsoft.com/blog/azure-limits-quotas-increase-requests/

<!--Other Web references-->
[Azure 门户]: https://manage.windowsazure.cn/


<!---HONumber=Mooncake_0606_2016-->