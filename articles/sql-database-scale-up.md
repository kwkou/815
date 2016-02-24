<properties
	pageTitle="更改 Azure SQL 数据库的服务层和性能级别"
	description="“更改 Azure SQL 数据库的服务层和性能级别”介绍如何扩展或缩减 SQL 数据库。更改 Azure SQL 数据库的定价层。"
	services="sql-database"
	documentationCenter=""
	authors="stevestein"
	manager="jeffreyg"
	editor=""/>

<tags
	ms.service="sql-database"
	ms.date="12/01/2015"
	wacn.date="01/29/2016"/>


# 更改 SQL 数据库的服务层和性能级别（定价层）

**单一数据库**

> [AZURE.SELECTOR]
- [Azure 门户](/documentation/articles/sql-database-scale-up)
- [PowerShell](/documentation/articles/sql-database-scale-up-powershell)

本文介绍如何使用 [Azure 门户](https://manage.windowsazure.cn)更改 SQL 数据库的服务层和性能级别。

使用[将 SQL 数据库 Web/企业数据库升级到新服务层](/documentation/articles/sql-database-upgrade-new-service-tiers)和 [Azure SQL 数据库服务层和性能级别](/documentation/articles/sql-database-service-tiers)中的信息为 Azure SQL 数据库确定适当的服务层和性能级别。

> [AZURE.IMPORTANT] 更改 SQL 数据库的服务层和性能级别是一项联机操作。这意味着在整个操作期间数据库将保持联机并可用而没有停机时间。

- 若要对数据库进行降级，数据库应小于目标服务层允许的最大大小。 
- 在启用了[标准异地复制](/documentation/articles/sql-database-business-continuity-design)或[活动异地复制](/documentation/articles/sql-database-geo-replication-overview)的情况下升级数据库时，必须先将次要数据库升级到所需的性能层，然后再升级主数据库。
- 从高级服务层降级时，必须先终止所有异地复制关系。你可以按照[终止连续复制关系](/documentation/articles/sql-database-disaster-recovery)主题中所述的步骤停止主数据库与活动次要数据库之间的复制过程。
- 各服务层提供的还原服务是不同的。如果进行降级，你可能无法再还原到某个时间点，或者备份保留期变短。有关详细信息，请参阅 [Azure SQL 数据库备份和还原](/documentation/articles/sql-database-business-continuity)。
- 你可以在 24 小时内进行最多四项单独的数据库更改（服务层或性能级别）。
- 所做的更改完成之前不会应用数据库的新属性。


**若要完成本文，你需要以下各项：**

- Azure 订阅。如果你需要 Azure 订阅，只需单击本页顶部的“试用”，然后再回来完成本文的相关操作即可。
- Azure SQL 数据库。如果你没有 SQL 数据库，请按照[创建你的第一个 Azure SQL 数据库](/documentation/articles/sql-database-get-started)一文中的步骤创建一个。


## 更改数据库的服务层和性能级别


打开要增加或减少的数据库的 SQL 数据库边栏选项卡：

1.	转到 [Azure 门户](https://manage.windowsazure.cn)。
2.	单击“浏览全部”。
3.	单击“SQL 数据库”。
2.	单击要更改的数据库。
3.	在 SQL 数据库边栏选项卡中，单击“定价层”磁贴：
1.  选择新层，然后单击“选择”：

    单击“选择”将提交更改数据库层的缩放请求。根据数据库的大小，缩放操作可能需要一些时间才能完成。单击通知可了解缩放操作的详细信息和状态。



3.	在左侧功能区中，单击“通知”：



## 验证数据库是否处于所选定价层

   在缩放操作完成后，检查并确认该数据库处于所需的层：

2.	单击“浏览全部”。
3.	单击“SQL 数据库”。
2.	单击已更新的数据库。
3.	检查**定价层**磁贴，并确认它已设置为正确的层。


## 后续步骤

- [扩大和缩小](/documentation/articles/sql-database-elastic-scale-get-started)
- [使用 SSMS 连接和查询 SQL 数据库](/documentation/articles/sql-database-connect-query-ssms)
- [导出 Azure SQL 数据库](/documentation/articles/sql-database-export)

## 其他资源

- [业务连续性概述](/documentation/articles/sql-database-business-continuity)
- [SQL 数据库文档](/documentation/services/sql-databases)


<!--Image references-->
[1]: ./media/sql-database-scale-up/pricing-tile.png
[2]: ./media/sql-database-scale-up/choose-tier.png
[3]: ./media/sql-database-scale-up/scale-notification.png
[4]: ./media/sql-database-scale-up/new-tier.png

<!---HONumber=Mooncake_0118_2016-->
