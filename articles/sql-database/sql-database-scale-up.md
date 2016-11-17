<properties
	pageTitle="更改 Azure SQL 数据库的服务层和性能级别"
	description="“更改 Azure SQL 数据库的服务层和性能级别”介绍如何扩展或缩减 SQL 数据库。更改 Azure SQL 数据库的定价层。"
	services="sql-database"
	documentationCenter=""
	authors="stevestein"
	manager="jhubbard"
	editor=""/>

<tags
	ms.service="sql-database"
	ms.devlang="NA"
	ms.date="10/12/2016"
	wacn.date="09/26/2016"
	ms.author="sstein"
	ms.workload="data-management"
	ms.topic="article"
	ms.tgt_pltfrm="NA"/>


# 更改 SQL 数据库的服务层和性能级别（定价层）


> [AZURE.SELECTOR]
- [Azure 门户预览](/documentation/articles/sql-database-scale-up/)
- [PowerShell](/documentation/articles/sql-database-scale-up-powershell/)


服务层和性能级别描述了适用于你的 SQL 数据库的功能和资源，并且可以随着应用程序更改的需要进行更新。有关详细信息，请参阅[服务层](/documentation/articles/sql-database-service-tiers/)。

请注意，更改数据库的服务层和/或性能级别将在新的性能级别创建原始数据库的副本，然后将切换连接到副本。当我们切换到副本时，在此过程中不会丢失任何数据，但在短暂的瞬间，将禁用与数据库的连接，因此可能回滚某些处于进行状态的事务。此窗口会有所不同，但平均起伏少于 4 秒，而在超过 99%的情况下为不足 30 秒。极少情况下，尤其有很大数量的事务在连接禁用的瞬间正在进行中，则此窗口停留时间可能会更长。

整个扩展过程的持续时间同时取决于更改前后数据库的大小和服务层。例如，一个正在更改到标准服务层，从标准服务层更改或在标准服务层内更改的 250 GB 的数据库，应在 6 小时内完成。对于与正在高级服务层内更改性能级别的大小相同的数据库，它应在 3 小时内完成。


使用[将 SQL 数据库 Web/企业数据库升级到新服务层](/documentation/articles/sql-database-upgrade-server-portal/)和 [Azure SQL 数据库服务层和性能级别](/documentation/articles/sql-database-service-tiers/)中的信息为 Azure SQL 数据库确定适当的服务层和性能级别。

- 若要对数据库进行降级，数据库应小于目标服务层允许的最大大小。
- 在启用了[异地复制](/documentation/articles/sql-database-geo-replication-overview/)的情况下升级数据库时，必须先将次要数据库升级到所需的性能层，然后再升级主数据库。
- 服务层降级时，必须先终止所有异地复制关系。
- 各服务层提供的还原服务是不同的。如果进行降级，你可能无法再还原到某个时间点，或者备份保留期变短。有关详细信息，请参阅 [Azure SQL 数据库备份和还原](/documentation/articles/sql-database-business-continuity/)。
- 更改数据库定价层不会更改最大数据库大小。若要更改数据库最大大小，请使用 [Transact-SQL (T-SQL)](https://msdn.microsoft.com/zh-cn/library/mt574871.aspx) 或 [PowerShell](https://msdn.microsoft.com/zh-cn/library/mt619433.aspx)。
- 所做的更改完成之前不会应用数据库的新属性。



**若要完成本文，你需要以下各项：**

- Azure 订阅。如果你需要 Azure 订阅，只需单击本页顶部的“试用”，然后再回来完成本文的相关操作即可。
- Azure SQL 数据库。如果你没有 SQL 数据库，请按照[创建你的第一个 Azure SQL 数据库](/documentation/articles/sql-database-get-started/)一文中的步骤创建一个。


## 更改数据库的服务层和性能级别


打开要增加或减少的数据库的 SQL 数据库边栏选项卡：

1.	转到 [Azure 门户预览](https://manage.windowsazure.cn)。
2.	单击“浏览全部”。
3.	单击“SQL 数据库”。
2.	单击要更改的数据库。
3.	在“SQL 数据库”边栏选项卡上，单击“所有设置”，然后单击“定价层(规模 DTU)”：

    ![定价层][1]


1.  选择新层，然后单击“选择”：

    单击“选择”将提交更改数据库层的缩放请求。根据数据库的大小，缩放操作可能需要一些时间才能完成。单击通知可了解缩放操作的详细信息和状态。

    > [AZURE.NOTE] 更改数据库定价层不会更改最大数据库大小。若要更改数据库最大大小，请使用 [Transact-SQL (T-SQL)](https://msdn.microsoft.com/zh-cn/library/mt574871.aspx) 或 [PowerShell](https://msdn.microsoft.com/zh-cn/library/mt619433.aspx)。

    ![选择定价层][2]

3.	在左侧功能区中，单击“通知”：

    ![通知][3]

## 验证数据库是否处于所选定价层

   在缩放操作完成后，检查并确认该数据库处于所需的层：

2.	单击“浏览全部”。
3.	单击“SQL 数据库”。
2.	单击已更新的数据库。
3.	检查“定价层”，确认它已设置为正确的层。

    ![新价格][4]


## 后续步骤

- 更改数据库最大大小，使用 [Transact-SQL (T-SQL)](https://msdn.microsoft.com/zh-cn/library/mt574871.aspx) 或 [PowerShell](https://msdn.microsoft.com/zh-cn/library/mt619433.aspx)。
- [扩大和缩小](/documentation/articles/sql-database-elastic-scale-get-started/)
- [使用 SSMS 连接和查询 SQL 数据库](/documentation/articles/sql-database-connect-query-ssms/)
- [导出 Azure SQL 数据库](/documentation/articles/sql-database-export/)

## 其他资源

- [业务连续性概述](/documentation/articles/sql-database-business-continuity/)
- [SQL 数据库文档](/documentation/services/sql-databases)


<!--Image references-->
[1]: ./media/sql-database-scale-up/pricing-tile.png
[2]: ./media/sql-database-scale-up/choose-tier.png
[3]: ./media/sql-database-scale-up/scale-notification.png
[4]: ./media/sql-database-scale-up/new-tier.png

<!---HONumber=Mooncake_0919_2016-->