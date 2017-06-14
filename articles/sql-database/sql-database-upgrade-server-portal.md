<properties
	pageTitle="使用 Azure 门户升级到 Azure SQL 数据库 V12 | Azure"
	description="介绍如何使用 Azure 门户升级到 Azure SQL 数据库 V12，包括如何升级 Web 和企业数据库，以及如何升级 V11 服务器并将其数据库直接迁移到弹性数据库池。"
	services="sql-database"
	documentationCenter=""
	authors="stevestein"
	manager="jhubbard"
	editor=""/>

<tags
	ms.service="sql-database"
	ms.devlang="na"
	ms.topic="article"
	ms.tgt_pltfrm="na"
	ms.workload="data-management"
	ms.date="08/08/2016"
	wacn.date="12/19/2016"
	ms.author="sstein"/>


# 使用 Azure 门户升级到 Azure SQL 数据库 V12


> [AZURE.SELECTOR]
- [Azure 门户](/documentation/articles/sql-database-upgrade-server-portal/)
- [PowerShell](/documentation/articles/sql-database-upgrade-server-powershell/)


SQL 数据库 V12 是最新版本，因此建议将现有服务器升级到 SQL 数据库 V12。
SQL 数据库 V12 具有[旧版所欠缺的许多优点](/documentation/articles/sql-database-v12-whats-new/)，包括：

- 提高了与 SQL Server 的兼容性。
- 经过改进的高级性能和新的性能级别。
- [弹性数据库池](/documentation/articles/sql-database-elastic-pool/)。

本文提供有关将现有 SQL 数据库 V11 服务器和数据库升级到 SQL 数据库 V12 的说明。

在升级到 V12 的过程中，会将所有 Web 和企业数据库升级到新的服务层，因此本文还包含了有关升级 Web 和企业数据库的说明。

此外，与升级到单一数据库的单独性能级别（定价层）相比，迁移到[弹性数据库池](/documentation/articles/sql-database-elastic-pool/)更具成本效益。池还可以简化数据库管理，因为你只需管理池的性能设置，而无需分开管理单独数据库的性能级别。如果你的数据库位于多台服务器上，请考虑将它们迁移到同一台服务器，并利用入池所带来的优势。你可以轻松地[使用 PowerShell 将数据库从 V11 服务器自动迁移到弹性数据库池](/documentation/articles/sql-database-upgrade-server-powershell/)。你还可以使用该门户将 V11 数据库迁移到池中，但在门户中，你必须已经有一个可用于创建池的 V12 服务器。本文后面提供了说明，告知你如何在服务器升级后创建池，前提是你拥有[能够利用池的数据库](/documentation/articles/sql-database-elastic-pool-guidance/)。

请注意，数据库将保持联机，并且在整个升级操作过程中都会继续保持工作。在实际转换到新的性能级别时，数据库连接可能会暂时中断很短的一段时间，通常约 90 秒，但最长可达 5 分钟。如果你的应用程序有[针对连接终止的暂时性故障处理机制](/documentation/articles/sql-database-connectivity-issues/)，则足以防止升级结束时连接中断。

升级到 SQL 数据库 V12 的操作不可撤销。升级后，无法将服务器还原到 V11。

升级到 V12 之后，[服务层建议](/documentation/articles/sql-database-service-tier-advisor/)和[弹性池性能注意事项](/documentation/articles/sql-database-elastic-pool-guidance/)不会立即可用，必须等到服务有时间评估新服务器上的工作负荷之后，才可供使用。V11 服务器建议历史记录不适用于 V12 服务器，因此不会保留。

## 准备升级

- **升级所有 Web 和企业数据库**：请参阅下面的[升级所有 Web 和企业数据库](/documentation/articles/sql-database-upgrade-server-portal/#upgrade-all-web-and-business-databases)部分，或[监视和管理弹性数据库池 (PowerShell)](/documentation/articles/sql-database-elastic-pool-manage-powershell/)。
- **检查和暂停异地复制**：如果你的 Azure SQL 数据库已针对异地复制进行配置，则你应记录其当前配置并[停止异地复制](/documentation/articles/sql-database-geo-replication-portal/#remove-secondary-database)。升级完成后，请重新为异地复制配置数据库。
- **如果客户端在 Azure VM 上，请打开这些端口**：如果客户端程序连接到 SQL 数据库 V12，而客户端运行在 Azure 虚拟机 (VM) 上，则必须打开 VM 上的端口范围 11000-11999 和 14000-14999。有关详细信息，请参阅 [SQL 数据库 V12 的端口](/documentation/articles/sql-database-develop-direct-route-ports-adonet-v12/)。



## 开始升级

1. 在 [Azure 门户](https://portal.azure.cn/)中，通过依次选择“浏览”>“SQL Server”，然后选择要升级的 v2.0 服务器，浏览到要升级的服务器。
2. 选择“最新 SQL 数据库更新”，然后选择“升级此服务器”。

      ![升级服务器][1]

3. 将服务器升级到最新 SQL 数据库更新是永久且不可逆的。若要确认升级，请键入服务器的名称，然后单击“确定”。

##<a name="upgrade-all-web-and-business-databases"></a> 升级所有 Web 和企业数据库

如果你的服务器具有任何 Web 或企业数据库，你必须对这些数据库进行升级。在升级到 SQL 数据库 V12 的过程中，需要将所有 Web 和企业数据库更新到新的服务层。

为了帮助你进行升级，SQL 数据库服务会为每个数据库建议适当的服务层和性能级别（定价层）。通过分析数据库的历史使用情况，该服务建议最适合运行你的现有数据库工作负荷的层。

3. 在“升级此服务器”边栏选项卡中，选择要查看的每个数据库，并选择建议的要升级到的定价层。你始终可以浏览可用的定价层，并选择最适合你环境的定价层。


     ![数据库][2]


7. 单击建议的层后，你将会看到“选择你的定价层”边栏选项卡，你可以在其中选择一个层，然后单击“选择”按钮更改到该层。为每个 Web 或企业数据库选择新层。

    请特别注意，SQL 数据库不会锁定到任何特定的服务层或性能级别，因此当你的数据库要求发生变化时，你可以轻松地在可用服务层和性能级别之间切换。事实上，基本、标准和高级 SQL 数据库按小时计费，在 24 小时内，你可以增加或减少每个数据库 4 次。

    ![建议][6]


在服务器上的所有数据库都符合要求后，你便可以开始升级了

## 确认升级

3. 在服务器上的所有数据库都符合升级要求后，请**键入服务器名称**以确认要执行升级，然后单击“确定”。

    ![验证升级][3]


4. 升级将启动并显示正在进行通知。此时将启动升级过程。根据你的特定数据库的详细信息，升级到 V12 可能需要一些时间。在此期间，服务器上的所有数据库都保持联机，但服务器和数据库管理操作会受到限制。

    ![正在进行升级][4]

    在实际转换到新的性能级别时，数据库连接可能会暂时中断一小段时间（通常以秒计）。如果应用程序对连接终止有暂时性故障处理机制（重试逻辑），则足以防止升级结束时连接中断。

5. 升级操作完成后，“最新更新”边栏选项卡会显示“已启用”。

    ![已启用 V12][5]

## 将数据库移到弹性数据库池中

在 [Azure 门户](https://portal.azure.cn)中浏览到 V12 服务器并单击“添加池”。

-或-

如果你看到一条消息说“单击此处以查看针对此服务器建议的弹性数据库池”，则可单击它轻松创建一个已针对你服务器的数据库优化的池。有关详细信息，请参阅[弹性数据库池的价格和性能注意事项](/documentation/articles/sql-database-elastic-pool-guidance/)。

![将池添加到服务器][7]

按照[创建弹性数据库池](/documentation/articles/sql-database-elastic-pool/)这篇文章中的说明进行操作，以便完成池的创建。

## 升级到 SQL 数据库 V12 后监视数据库

>[AZURE.IMPORTANT] 升级到最新版本的 SQL Server Management Studio (SSMS) 以利用新的 v12 功能。[下载 SQL Server Management Studio](https://msdn.microsoft.com/zh-cn/library/mt238290.aspx)。

升级后，主动监视数据库，确保应用程序以所需性能运行，然后根据需要优化设置。

除监视单个数据库外，还可以监视弹性数据库池。使用[Azure 门户](/documentation/articles/sql-database-elastic-pool-manage-portal/)或通过 [PowerShell](/documentation/articles/sql-database-elastic-pool-manage-powershell/) 监视、管理弹性数据库池并设置其大小。


**资源消耗数据：**对于基本、标准和高级数据库，可通过用户数据库中的 [sys.dm_ db_ resource\_stats](http://msdn.microsoft.com/zh-cn/library/azure/dn800981.aspx) DMV 使用资源消耗数据。此 DMV 针对前一小时的操作，以 15 秒的粒度级提供接近实时的资源消耗信息。每个间隔的 DTU 消耗百分比将计算为 CPU、IO 和日志维度的最大消耗百分比。下面是一个用于计算过去一小时平均 DTU 消耗百分比的查询：

    SELECT end_time
    	 , (SELECT Max(v)
             FROM (VALUES (avg_cpu_percent)
                         , (avg_data_io_percent)
                         , (avg_log_write_percent)
    	   ) AS value(v)) AS [avg_DTU_percent]
    FROM sys.dm_db_resource_stats
    ORDER BY end_time DESC;

其他监视信息：

- [Azure SQL 数据库的单一数据库性能指南](/documentation/articles/sql-database-performance-guidance/)。
- [弹性数据库池的价格和性能注意事项](/documentation/articles/sql-database-elastic-pool-guidance/)。
- [使用动态管理视图监视 Azure SQL 数据库](/documentation/articles/sql-database-monitoring-with-dmvs/)




**警报：**在 Azure 门户中设置“警报”可在升级后的数据库 DTU 消耗量接近特定的高位时接收通知。你可以针对 DTU、CPU、IO 和日志等各种性能指标，在 Azure 门户中设置数据库警报。浏览到你的数据库，然后在“设置”边栏选项卡中选择“警报规则”。

例如，你可以针对“DTU 百分比”设置电子邮件警报，以便在过去 5 分钟平均 DTU 百分比值超过 75% 时发出警报。请参阅[接收警报通知](/documentation/articles/insights-receive-alert-notifications/)，以了解有关如何配置警报通知的详细信息。





## 后续步骤

- [检查池建议，然后创建一个池](/documentation/articles/sql-database-elastic-pool-create-portal/)。
- [更改数据库的服务层和性能级别](/documentation/articles/sql-database-scale-up/)。



## 相关链接

- [SQL 数据库 V12 的新增功能](/documentation/articles/sql-database-v12-whats-new/)


<!--Image references-->
[1]: ./media/sql-database-upgrade-server-portal/latest-sql-database-update.png
[2]: ./media/sql-database-upgrade-server-portal/upgrade-server2.png
[3]: ./media/sql-database-upgrade-server-portal/upgrade-server3.png
[4]: ./media/sql-database-upgrade-server-portal/online-during-upgrade.png
[5]: ./media/sql-database-upgrade-server-portal/enabled.png
[6]: ./media/sql-database-upgrade-server-portal/recommendations.png
[7]: ./media/sql-database-upgrade-server-portal/new-elastic-pool.png

<!---HONumber=Mooncake_Quality_Review_1202_2016-->