<properties 
	pageTitle="将 SQL 数据库 Web 或企业数据库升级到新服务层"
	description="将 Azure SQL 数据库 Web 或企业数据库升级到新的 Azure SQL 数据库基本、标准和高级服务层与性能级别。" 
	services="sql-database" 
	documentationCenter="" 
	authors="stevestein" 
	manager="jeffreyg" 
	editor=""/>

<tags 
	ms.service="sql-database"
	ms.date="06/18/2015" 
	wacn.date="09/15/2015"/>


# 将 SQL 数据库 Web 或企业数据库升级到新服务层

Azure SQL Web 和企业数据库将[在 2015 年 9 月淘汰](http://msdn.microsoft.com/zh-cn/library/azure/dn741330.aspx)，因此现在是开始规划将现有的 Web 或企业数据库升级到基本、标准或高阶服务层的时候了。

下载 [Web 和企业数据库迁移指导手册](http://download.microsoft.com/download/3/C/9/3C9FF3D8-E702-4F5D-A38C-5EFA1DBA73E6/Azure_SQL_Database_Migration_Cookbook.pdf)。

> [AZURE.NOTE] [Pricing tier recommendations](/documentation/articles/sql-database-service-tier-advisor) Web 和企业数据库的 现在发布。

## 概述

<p> Azure Web 和企业 SQL 数据库在共享的多租户环境中运行，这种环境没有为数据库提供任何保留的资源容量。此共享资源环境中其他数据库的活动可能会影响你的性能。资源在任意给定时间点的可用性很大程度上取决于系统中运行的其他并发工作负荷。这可能会导致数据库应用程序性能出现很大的变化和不可预测性。客户反映，这种不可预测的性能很难管理，他们更希望性能可预测。

为了迎合这种需求，Azure SQL 数据库服务引入了新的数据库服务层[（基本、标准和高级）](http://msdn.microsoft.com/zh-cn/library/dn741340.aspx)，这些版本提供可预测的性能和丰富的新功能，可实现业务连续性和安全性。这些新服务层旨在为数据库工作负荷提供指定级别的资源，而不考虑该环境中运行的其他客户工作负荷。这样，便使得性能行为高度可预测。

这些变化为客户带来了这样的问题：如何评估新的服务层，哪个新的服务层最适合他们目前的 Web 和企业 (W/B) 数据库，以及如何运作实际升级流程自身。

最终，最好的选择就是可在功能、性能和成本间提供最佳平衡，同时又完全符合业务需求和应用程序要求的服务层与性能级别组合。

本文以引导式的方法介绍了如何将 Web/企业数据库升级到新的服务层/性能级别。

请特别注意，Azure SQL 数据库不会锁定到任何特定的服务层或性能级别，因此当你的数据库要求发生变化时，你可以轻松在可用服务层和性能级别之间切换。事实上，基本、标准和高级版 Azure SQL 数据库按小时计费，在 24 小时内，你可以扩展或收缩每个数据库 4 次（使用 [Azure 门户或编程方式](http://msdn.microsoft.com/zh-cn/library/azure/ff394116.aspx)）。这意味着，你可以调整服务层和性能级别，以根据应用程序的要求和各种工作负荷，最大程度充分发挥数据库在性能需求、功能集和成本方面的优势。这也意味着，评估和更改数据库的服务层与性能级别（扩展和收缩）是持续不断的过程，应该是计划的维护与性能优化例行工作的一部分。


## 升级 Web 和企业数据库

将 Web 或企业数据库升级到新的服务层/性能级别不会使数据库脱机。数据库将继续进行升级操作。在实际转换到新的性能级别时，数据库连接可能会暂时中断一小段时间（通常以秒计）。如果应用程序对连接终止有暂时性故障处理机制，则足以防止升级结束时连接中断。

将 Web 或企业数据库升级到新服务层的过程包括以下步骤：


1. 根据功能确定服务层
2. 根据历史资源使用量确定可接受的性能级别
3. 为什么 Web 或企业数据库的现有性能会映射到更高的“高级”级别？
4. 优化工作负荷以适应较低的性能级别
5. *升级到新的服务层/性能级别*
6. 监视升级到新服务层/性能级别
7. 升级后监视数据库



## 1\.根据功能确定服务层

基本、标准和高级服务层提供不同的功能集，因此选择适当层的第一个步骤就是确定可提供应用程序和业务所需最低功能级别的服务层。

例如，请考虑备份需要保留多久，或是否需要[标准或活动异地复制](http://msdn.microsoft.com/zh-cn/library/azure/dn783447.aspx)功能，或所需的整体最大数据库大小等等。这些要求确定了要选择的最低服务层。

“基本”层主要用于极小型、活动少的数据库。因此，通常应根据所需功能的最低级别，优先选择升级到“标准”或“高级”层。

下表汇总及比较了新服务层的功能和性能级别：

![服务层功能比较][1]


**有关比较服务层和性能级别的其他资源：**

| 文章 | 说明 |
|:--|:--|
|[Azure SQL Database 服务层（版本）](http://msdn.microsoft.com/zh-cn/library/azure/dn741340.aspx)| 基本、标准和高级服务层的概述。|
|[Azure SQL Database 服务层和性能级别](http://msdn.microsoft.com/zh-cn/library/dn741336.aspx)| 各服务层的度量值和功能（以及如何在管理门户中使用 DMV 监视数据库利用率）。 |
|[服务层有哪些不同之处？](http://msdn.microsoft.com/zh-cn/library/dn369873.aspx#Different)| 有关不同服务层的详细信息，包括你为何要选择一个层而不是另一个层。 |
|[Azure SQL Database 业务连续性](http://msdn.microsoft.com/zh-cn/library/azure/hh852669.aspx)|不同服务层提供的业务连续性和灾难恢复功能（时间点还原、异地还原、异地复制）的详细信息。|
|[SQL 数据库定价](/home/features/sql-database/#price)|不同服务层和性能级别的详细定价信息。|

<br>

选择符合数据库要求的适当服务层之后，下一步就是对实际数据库工作负荷选择可接受的性能级别。



## 2\.根据历史资源使用量确定可接受的性能级别

Azure SQL 数据库服务将在管理门户和“系统视图”中公开信息，以针对现有的 Web 或企业数据库建议适当的新服务层和性能级别。

由于 Web 和企业数据库没有关联的任何有保证 DTU/资源限制，我们以 S2 性能级别数据库可用的资源量将百分比值规范化。任何特定间隔的数据库平均 DTU 百分比消耗量均可计算为该间隔内最高的 CPU、IO 和日志使用量百分比值。


使用管理门户获取高级 DTU 百分比使用量的高级概览，然后使用系统视图深入到详细信息。

当你将服务器升级到 Azure SQL 数据库 V12（[在某些区域以预览版提供](/documentation/articles/sql-database-preview-whats-new#V12AzureSqlDbPreviewGaTable)）时，也可以使用新的 Azure 管理门户来查看针对 Web 或企业数据库建议的服务层。

### 如何在新 Azure 管理门户中查看建议的服务层
管理门户在将服务器升级到 Azure SQL 数据库 V12 的过程中，针对 Web 或企业数据库建议适当的服务层。该建议基于数据库资源消耗量的历史分析。

**新管理门户**

1. 登录到[新管理门户](https://portal.azure.com)并导航到包含 Web 或企业数据库的服务器。
2. 单击服务器边栏选项卡中的“最新更新”部分。
3. 单击“升级此服务器”。

“升级此服务器”边栏选项卡现在会显示服务器上的 Web 或企业数据库列表，以及建议的服务层。



### 如何在管理门户中查看 DTU 消耗量
管理门户可让你深入了解现有 Web 或企业数据库的 DTU 消耗。当前 Azure 门户中会提供 DTU 信息。


**管理门户**

1. 登录到[管理门户](https://manage.windowsazure.cn)并导航到现有的 Web 或企业数据库。
2. 单击“监视”选项卡。
3. 单击“添加度量值”。
4. 选择“DTU 百分比”，然后单击底部的复选标记进行确认。

确认之后，表中应会出现“DTU 百分比”数据。请记住，这是相比于“标准 (S2)”级别数据库中 DTU（50 个 DTU）的百分比。

此外，应该注意的是，此数据是每隔 5 分钟采样一次的平均值，因此这些度量值可能未反映样本之间的短期活动突发。

![DTU 百分比数据][2]

请注意，上述示例中的数据显示大约 10 DTD 的平均使用量（50 的 19.23%）和最多 28 DTU 的最大 DTU 百分比（55.83% x 50）。假设此数据表示我的典型工作负荷，我可能在初次升级时选择“标准 (S1)”。“标准 (S0)”提供 10 DTU，这是我的平均使用量，不过那就表示我的数据库平均执行 100% 容量，绝对不是很好的计划。虽然 S1 可能是很适合我的平均使用量，而我达到最大值的次数又如何？ 或许我知道高峰来自于某些夜间维护进程，而实际的客户使用量并不受影响，因此该时间范围内的性能降低可能无妨。但是，也许我不知道何时达到最大值，因此 DTU 百分比消耗量可能需要进一步分析。

若要深入到数据库资源消耗状况的详细信息，可以使用提供的系统视图。


### 系统视图


在当前数据库所在的逻辑服务器的 master 数据库中，Web 和企业数据库资源消耗数据是通过 [sys.resource\_stats](http://msdn.microsoft.com/zh-cn/library/azure/dn269979.aspx) 视图访问的。该视图以性能级别限制的百分比显示资源消耗量数据。此视图最早可提供 14 天前的数据，间隔是 5 分钟。

> [AZURE.NOTE]现在可在 Web 和企业数据库中使用 [sys.dm\_db\_resource\_stats](https://msdn.microsoft.com/zh-cn/library/dn800981.aspx) 视图获取资源消耗量数据的较高粒度视图（15 秒间隔）。由于 sys.dm\_db\_resource\_stats 只将历史数据保留一小时，因此可以每小时查询此 DMV 一次，并存储数据以供进一步分析。

对 master 数据库运行以下查询可检索数据库的平均 DTU 消耗量：

 
                   
     SELECT start_time, end_time
	 ,(SELECT Max(v)
         FROM (VALUES (avg_cpu_percent)
                    , (avg_physical_data_read_percent)
                    , (avg_log_write_percent)
    	   ) AS value(v)) AS [avg_DTU_percent]
    FROM sys.resource_stats
    WHERE database_name = '<your db name>'
    ORDER BY end_time DESC;

Web 和企业层的 [sys.resource\_stats](https://msdn.microsoft.com/zh-cn/library/dn269979.aspx) 与 [sys.dm\_db\_resource\_stats](https://msdn.microsoft.com/zh-cn/library/dn800981.aspx) 返回的数据指示标准 S2 性能层的百分比。例如，当针对 Web 或企业数据库执行时，如果值返回 70%，表示 S2 层限制的 70%。此外，对于 Web 和企业层，百分比可能反映超出 100% 的数字，这也是基于 S2 层限制的值。

以 S2 数据库级别为依据的 DTU 消耗量信息可将 Web 和企业数据库当前的消耗量以新服务层数据库的方式规范化，从而找出较合适的服务层。例如，如果平均 DTU 百分比消耗量显示值为 80%，表示数据库消耗 DTU 的比率是 S2 性能级别的数据库限制的 80%。如果你在 **sys.resource\_stats** 视图中看到大于 100% 的值，则表示需要大于 S2 的性能层。例如，假设你看到了值为 300% 的高峰 DTU 百分比值。这表明使用的资源比 S2 中可用的资源多三倍。若要确定合理的起始大小，请比较 S2 中可用的 DTU（50 DTU）与下一个较高的大小（S3/P1 = 100 DTU 或 S2 的 200 %，P2 = 200 DTU 或 S2 的 400 %）。因为比率是 S2 的 300%，可以从 P2 开始再重新测试。

根据 DTU 使用量百分比和符合工作负荷所需的最大版本，可以确定哪个服务层和性能级别最适合数据库工作负荷（如同通过 DTU 百分比和各种[性能级别](http://msdn.microsoft.com/zh-cn/library/azure/dn741336.aspx)的相对 DTU 能力所示）。下表提供了 Web/业务资源消耗量百分比和对等新服务层性能级别之间的映射：

![资源消耗量][4]

> **注意：**各种性能级别间的相对 DTU 数基于 [Azure SQL 数据库基准](http://msdn.microsoft.com/zh-cn/library/azure/dn741327.aspx)工作负荷。由于数据库工作负荷很可能不同于基准，应使用上述计算作为指导原则来确定最初适合 Web/企业数据库的新服务层。在将数据库移到新服务层后，请使用上一部分中所述的过程来验证/优化符合工作负荷需求的适当服务层。
> 
> 尽管建议的新版服务层/性能级别将过去 14 天的数据库活动纳入考虑，但这些数据基于平均超过 5 分钟的资源消耗量数据样本。因此可能会遗漏持续时间少于 5 分钟的短期活动突发。在升级数据库时，应使用本指南作为起点。将数据库升级到建议的服务层后，需要进行更多的监视、测试和验证，以根据需要将数据库升级或降级到不同的服务层/性能级别。

下面是对 master 数据库运行的一个查询，该查询将针对你的 Web/企业层数据库执行计算，然后根据每个 5 分钟数据采样间隔建议可能适合你工作负荷的版本。

> **注意：**此查询仅适用于 Web/企业数据库，对于新层中的数据库，它**不会**提供正确的结果。

    WITH DTU_mapping AS
    ( SELECT *
        FROM ( VALUES (1, 10,'Basic'), (2, 20,'S0'), (3, 40,'S1'), (4, 100, 'S2')
                    , (5, 200, 'S3/P1'), (6, 1600,'Premium')
           ) AS t(id, percent_of_S2, target_edition)
    ), rc as
    ( SELECT start_time, end_time
           , (SELECT Max(v)
                FROM (VALUES (avg_cpu_percent)
                       		, (avg_physical_data_read_percent)
                       		, (avg_log_write_percent)
                   ) AS value(v)) as [avg_DTU_percent]
        FROM sys.resource_stats	
       WHERE database_name = 'WebDB'
    )
    SELECT rc.*
         , (SELECT TOP(1) t.target_edition
              FROM DTU_mapping AS t
             WHERE t.percent_of_S2 > CAST(1.2*rc.avg_DTU_percent as int)
             ORDER BY t.percent_of_S2) as target_edition
    FROM rc;

**示例结果：**

![示例结果](./media/sql-database-upgrade-new-service-tiers/sample_result.png)

在图表中，你可以看到不同时间的平均 DTU 消耗百分比变化趋势。下面是大部分时间处于 S2 级别中的某个数据库的示例图表，该数据库的某个高峰活动上冲到了 P1 数据库级别。各时间的 DTU 消耗量从“基本”限制最大升到了“P1”限制。若要使此数据库完全适合新层，你需要购买性能级别为“P1”的“高级”服务层数据库。另一方面，如果 P1 级别极少出现这种偶然喷发现象，则 S2 级别数据库也是可行的。

![DTU 使用量](./media/sql-database-upgrade-new-service-tiers/DTU_usage.png)

**内存对性能的影响：**尽管内存是影响 DTU 等级的资源维度之一，但 SQL 数据库会使用所有可用内存来运行数据库操作。出于此原因，上述查询未在平均 DTU 消耗中包括内存消耗。另一方面，如果你要降级到更低的性能级别，则数据库的可用内存将会减少。这可能会导致 IO 消耗提高，从而影响消耗的 DTU 数量。因此，在降级到更低的性能级别时，请确保 IO 百分比中有足够的余隙。可以使用上述 [sys.dm\_db\_resource\_stats](http://msdn.microsoft.com/zh-cn/library/azure/dn800981.aspx) DMV 来监视这种情况。



## 3\.为什么 Web 或企业数据库的现有性能会映射到更高的“高级”级别？

Web 和企业数据库没有针对任一单个数据库保留特定数量的资源容量。此外，没有任何机制可供客户向上或向下缩放 Web 或企业数据库的性能。这会导致 Web 和企业数据库性能不一，范围从相当缓慢到“高级”级别。这种多变的性能范围*不公平地*依赖于在多租户环境中共享资源的其他数据库在任何时间点的整体资源消耗级别。

当然 Azure SQL 数据库服务的目标是让所有 Web 和企业数据库尽可能以接近 100% 最佳状况运行。此外，这些服务还让 Web 和企业数据库的平均性能级别提升到“高级”级别。这就是为何现有数据库的性能可能映射到当前“高级”级别的原因。遗憾的是，在评估要升级到哪个服务层的过程中比较 Web 和企业数据库与新的服务层/性能级别时，这会导致有点不切实际的预期。

若要更清楚地了解 Web/Business 与基本、标准和高级服务层之间的差异，请查看下图。相比于在基本、标准和高级服务层模型中运行的 9 个 SQL 数据库，请考虑在共享资源 Web/企业模型中运行的 9 个 SQL 数据库。在 Web/企业模型中，可以清楚看到“干扰性邻居”对此共享资源池中其他数据库的影响。当 1 个数据库正在运行资源密集型工作负荷时，池中的所有其他数据库都会受影响，并且可用的资源开始减少。在基本、标准和高级服务层中，将为每个数据库***分配***特定数量的资源，因此共享环境中的其他数据库基本上不受干扰性邻居问题以及受限于为数据库选择的性能级别的影响。

![新服务层的可预测性能][3]

如果整体 DTU 百分比极高，你应该开始调查构成 DTU 的详细度量值；请特别深化到数据库的日志 I/O 和内存使用量详细信息。你有可能会发现可最优化并减少 DTU 消耗量的潜在区域。


## 4\.优化数据库工作负荷以适应较低的性能级别
如果数据库的历史资源使用量分析指示应该升级到成本超出预期的性能级别，可以研究一下额外性能优化有所帮助的区域。

假设你了解应用程序的详细信息，如果相比于典型工作负荷预期应有的使用量，资源使用量似乎非常高，也许你有一定的机会通过性能优化来使应用程序受益。

事实上，所有数据库均可受益于另一轮的性能优化。

除了典型优化维护（例如分析索引、执行计划等），你应该针对 Azure SQL 数据库优化数据访问例程。

| 文章 | 说明 |
|:--|:--|
| [Azure SQL 数据库性能指南](http://msdn.microsoft.com/zh-cn/library/dn369873.aspx) | “优化应用程序”部分提供了有关优化 Azure SQL 数据库性能的详细建议和方法。|
|[性能监视和优化工具](https://msdn.microsoft.com/zh-cn/library/ms179428.aspx)| 可用于在 SQL Server 中监视事件及优化物理数据库设计的工具的链接和说明。|
|[性能监视和优化](https://msdn.microsoft.com/zh-cn/library/ms189081.aspx)|MSDN 上的“SQL Server 2014 性能优化”部分。|
| [排查 SQL Server 2008 中的性能问题](https://msdn.microsoft.com/zh-cn/library/dd672789.aspx) | 较旧但仍能提供有关排查常见性能问题指导的白皮书，包括有关排查内存和 CPU 瓶颈的极佳信息。|
|[Azure SQL 数据库的故障排除和查询优化](http://social.technet.microsoft.com/wiki/contents/articles/1104.troubleshoot-and-optimize-queries-with-azure-sql-database.aspx)|较旧但与动态管理视图以及如何将它用于故障排除相关的有用主题。|



## 5\.升级到新的服务层/性能级别
在确定 Web 或企业数据库的适当服务层和性能级别后，可通过多种方法将数据库升级到新层：
      	 
| 管理工具 | 更改数据库的服务层和性能级别|
| :---| :---|
| [Azure 管理门户](https://manage.windowsazure.cn) | 单击数据库仪表板页面上的“缩放”选项卡。 |
| [Azure PowerShell](http://msdn.microsoft.com/zh-cn/library/azure/dn546726.aspx) | 使用 [Set-AzureSqlDatabase](http://msdn.microsoft.com/zh-cn/library/azure/dn546732.aspx) cmdlet。 |
| [服务管理 REST API](http://msdn.microsoft.com/zh-cn/library/azure/dn505719.aspx) | 使用“更新数据库”命令[](http://msdn.microsoft.com/zh-cn/library/dn505718.aspx)。|
| [Transact-SQL](http://msdn.microsoft.com/zh-cn/library/bb510741.aspx) | 使用 [ALTER DATABASE (Transact-SQL)](http://msdn.microsoft.com/zh-cn/library/ms174269.aspx) 语句。 |

有关详细信息，请参阅[更改数据库服务层和性能级别](http://msdn.microsoft.com/zh-cn/library/dn369872.aspx)


## 6\.监视升级到新服务层/性能级别
Azure SQL 数据库在 sys.dm\_operation\_status 动态管理视图中提供有关对数据库执行的管理操作（例如 CREATE、ALTER、DROP）的进度信息，该视图位于当前数据库所在的逻辑服务器的 master 数据库中 [请参阅 sys.dm\_operation\_status 文档](http://msdn.microsoft.com/zh-cn/library/azure/dn270022.aspx)。使用操作状态 DMV 可以确定数据库的升级操作进度。此示例查询显示了对数据库执行的所有管理操作：

    SELECT o.operation, o.state_desc, o.percent_complete
    , o.error_code, o.error_desc, o.error_severity, o.error_state
    , o.start_time, o.last_modify_time
    FROM sys.dm_operation_status AS o
    WHERE o.resource_type_desc = 'DATABASE'
    and o.major_resource_id = '<database_name>'
    ORDER BY o.last_modify_time DESC;

如果你使用管理门户执行了升级，则门户中也会提供有关该操作的通知。

### 升级后数据库工作负荷达到资源限制时会发生什么情况？
性能级别将会校准并受到控制，以便在选定服务层/性能级别所允许的最大限制范围内（即资源消耗量达到 100%）提供所需的资源来运行你的数据库工作负荷。如果你的工作负荷达到了 CPU/数据 IO/日志 IO 限制之一，你将会继续接收资源直到达到最大允许级别，但是，你可能会发现你的查询延迟不断增高。达到其中一个限制不会导致任何错误，而只会减慢你的工作负荷，直到严重变慢，以致于查询开始超时。如果你达到了并发用户会话/请求（工作线程）的最大允许数目限制，你将看到[错误 10928 或 10929](http://msdn.microsoft.com/zh-cn/library/azure/dn338078.aspx)。


## 7\.升级后监视数据库
将 Web/企业数据库升级到新层后，建议你主动监视数据库，以确保应用程序以所需的性能运行，并根据需要优化使用方式。建议使用以下附加步骤来监视数据库。


**资源消耗数据：**对于基本、标准和高级数据库，可通过用户数据库中名为 [sys.dm\_ db\_ resource\_stats](http://msdn.microsoft.com/zh-cn/library/azure/dn800981.aspx) 的新 DMV 查看更详细的资源消耗数据。此 DMV 针对前一小时的操作，以 15 秒的粒度级提供接近实时的资源消耗信息。每个间隔的 DTU 消耗百分比将计算为 CPU、IO 和日志维度的最大消耗百分比。下面是一个用于计算过去一小时平均 DTU 消耗百分比的查询：

    SELECT end_time
    	 , (SELECT Max(v)
             FROM (VALUES (avg_cpu_percent)
                         , (avg_data_io_percent)
                         , (avg_log_write_percent)
    	   ) AS value(v)) AS [avg_DTU_percent]
    FROM sys.dm_db_resource_stats
    ORDER BY end_time DESC;

 
其他[文档](http://msdn.microsoft.com/zh-cn/library/dn800981.aspx)包含了有关如何使用此 DMV 的详细信息。[Azure SQL 数据库性能指南](http://msdn.microsoft.com/zh-cn/library/azure/dn369873.aspx)介绍了如何监视和优化你的应用程序。


- **警报：**在 Azure 管理门户中设置“警报”可在升级后的数据库 DTU 消耗量接近特定的高位时接收通知。你可以针对 DTU、CPU、IO 和日志等各种性能度量值，在 Azure 管理门户中设置数据库警报。 

	例如，你可以针对“DTU 百分比”设置电子邮件警报，以便在过去 5 分钟平均 DTU 百分比值超过 75% 时发出警报。请参阅[如何：在 Azure 中接收警报通知和管理警报规则](http://msdn.microsoft.com/zh-cn/library/azure/dn306638.aspx)，以了解有关如何配置警报通知的详细信息。


- **计划的性能级别升级/降级：**如果应用程序的某些方案只是在一天/一周的特定时间才需要更高的性能，则你可以使用 [Azure 自动化](http://msdn.microsoft.com/zh-cn/library/azure/dn643629.aspx)，以计划中操作的形式将数据库升级/降级到更高/更低的性能级别。

	例如，在执行每周批处理/维护作业期间，将数据库升级到更高的性能级别，在完成作业后降级数据库。这种类型的计划也适用于任何大规模的资源密集型操作，例如数据加载、重建索引，等等。请注意，Azure SQL 数据库计费模式基于服务层/性能级别的每小时用量。凭借这种灵活性，你可以更加经济高效地安排有计划的升级。



## 摘要
Azure SQL 数据库 服务提供了遥测数据和工具来评估 Web/企业数据库工作负荷，并确定适合升级的最佳服务层。升级过程非常简单，无需使数据库脱机就能完成，且不会丢失数据。升级后的数据库将受益于可预测的性能，以及新服务层提供的更多功能。




<!--Image references-->
[1]: ./media/sql-database-upgrade-new-service-tiers/service-tier-features.png
[2]: ./media/sql-database-upgrade-new-service-tiers/portal-dtus.JPG
[3]: ./media/sql-database-upgrade-new-service-tiers/web-business-noisy-neighbor.png
[4]: ./media/sql-database-upgrade-new-service-tiers/resource_consumption.png

<!---HONumber=69-->