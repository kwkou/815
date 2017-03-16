<properties
    pageTitle="Azure SQL 数据库以及单一数据库的性能 | Azure"
    description="本文可帮助用户确定为应用程序选择何种服务层，并建议通过多种方式调整应用程序，以便充分利用 Azure SQL 数据库。"
    services="sql-database"
    documentationcenter="na"
    author="CarlRabeler"
    manager="jhubbard"
    editor="" />
<tags
    ms.assetid="dd8d95fa-24b2-4233-b3f1-8e8952a7a22b"
    ms.service="sql-database"
    ms.custom="monitor and tune"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="data-management"
    ms.date="01/04/2017"
    wacn.date="03/16/2017"
    ms.author="carlrab" />  


# Azure SQL 数据库以及单一数据库的性能
Azure SQL 数据库提供三个[服务层](/documentation/articles/sql-database-service-tiers/)：基本、标准、高级和高级 RS。每个服务层都会对供 SQL 数据库使用的资源进行严格的隔离，确保该服务级别的性能可以预测。本文将指导用户为其应用程序选择服务层，并讨论如何通过多种方式调整应用程序，以便充分利用 Azure SQL 数据库。

> [AZURE.NOTE]
本文侧重于 Azure SQL 数据库中单一数据库的性能指南。有关弹性池的性能指南，请参阅[弹性池的价格和性能注意事项](/documentation/articles/sql-database-elastic-pool-guidance/)。但请注意，你可以将本文的许多优化建议应用到弹性池中的数据库，获得类似的性能优势。
> 
> 

下面是可供选择的三个 Azure SQL 数据库服务层（性能按数据库吞吐量单位（简称 [DTU](/documentation/articles/sql-database-what-is-a-dtu/)）衡量）：

* **基本**。可以通过基本服务层按小时对每个数据库进行准确的性能预测。在基本数据库中，资源足以确保没有多个并发请求的小型数据库的良好性能。
* **标准**。标准服务层改进了存在多个并发请求的数据库（例如工作组和 Web 应用程序）的性能可预测性并使之更通畅地运行。选择标准服务层数据库时，可以根据每分钟的可预测性能调整数据库应用程序的大小。
* **高级**。高级服务层针对每个高级数据库提供每秒的可预测性能。选择高级服务层时，可以根据数据库应用程序的峰值负载调整该数据库的大小。此计划会消除因性能变动而导致小型查询在完成易受延迟影响的操作时所花时间超出预期的情况。此模型可大大简化需要明确表明最大资源需求、性能变动或查询延迟的应用程序的开发和产品验证环节。


用户可以在每个服务层设置性能级别，因此拥有相当的灵活性，只需为所需容量付费。可以根据工作负荷变化上下[调整容量](/documentation/articles/sql-database-scale-up/)。例如，如果数据库工作负荷在返校购物季期间高，则可在七月至九月这段固定的时间提高数据库的性能级别，在高峰期结束之后再降低性能级别。可以按业务的季节性因素优化云环境，将支出降至最低。此模型也很适合软件产品发布环节。测试团队可在进行测试运行时分配容量，测试完毕再释放该容量。在容量请求模型中，用户只需支付所需容量的款项，不需为很少使用的专用资源付款。

## 服务层的目的
每个数据库工作负荷可能各不相同，而服务层的目的就是在不同性能级别提供性能可预测性。具有大规模数据库资源需求的客户可以在更专用的计算环境下工作。

### 常见服务层用例
#### 基本
* **用户开始使用 Azure SQL 数据库**。处于开发阶段的应用程序通常需要的性能级别不高。基本数据库是进行数据库开发的理想环境，价位较低。
* **用户的数据库有一个用户**。将单个用户与某个数据库进行关联的应用程序通常在并发能力和性能方面不会提出很高的要求。这些应用程序适合使用基本服务层。

#### 标准
* **用户的数据库有多个并发请求**。同时为多个用户提供服务的应用程序通常需要更高的性能级别。例如，中等流量的网站或需要更多资源的部门应用程序，可考虑使用标准服务层。

#### 高级
大多数高级服务层用例具有下述一项或多项特征：

* **峰值负载高**。需要大量 CPU、内存或输入/输出 (I/O) 来完成其操作的应用程序需要专用的高性能级别。例如，如果已知某个数据库操作长时间占用多个 CPU 核心，则可考虑使用高级服务层。
* **并发请求数较多**。某些数据库应用程序为许多并发请求提供服务（例如，为流量较高的网站提供服务）。基本和标准服务层会限制每个数据库的并发请求数。需要更多连接的应用程序需要选择相应的预留大小以处理最大数量的所需请求。
* **延迟较低**。某些应用程序需要确保在最短时间内从数据库获得响应。如果在更大范围的客户操作期间调用了特定存储过程，则有 99% 的时间可能需要在 20 毫秒内从该调用返回响应。此类应用程序可充分利用高级服务层，确保提供所需计算能力。
* **高级 RS**。专为具有 IO 密集型工作负载但不需要最高可用性保证的客户设计。示例包括测试高性能工作负载，或分析数据库不是记录系统的工作负载。

SQL 数据库的所需服务级别取决于每个资源维度的峰值负载要求。某些应用程序对于某种资源仅少量使用，但对于其他资源需要大量使用。

##<a name="service-tier-capabilities-and-limits"></a> 服务层功能和限制
每个服务层和性能级别都与不同的限制和性能特征相关联。下表描述单一数据库的这些特征。

[AZURE.INCLUDE [SQL 数据库服务层表](../../includes/sql-database-service-tiers-table.md)]

### 最大内存中 OLTP 存储
可以使用 **sys.dm\_db\_resource\_stats** 视图来监视 Azure 内存中存储的使用情况。有关监视的详细信息，请参阅[监视内存中 OLTP 存储](/documentation/articles/sql-database-in-memory-oltp-monitoring/)。

### 最大并发请求数
若要查看并发请求数，请在 SQL 数据库中运行以下 Transact-SQL 查询：

	SELECT COUNT(*) AS [Concurrent_Requests]
	FROM sys.dm_exec_requests R

若要分析本地 SQL Server 数据库的工作负荷，请修改此查询，针对要分析的特定数据库进行筛选。例如，如果有一个名为 MyDatabase 的本地数据库，则以下 Transact-SQL 查询返回该数据库中并发请求的计数：

	SELECT COUNT(*) AS [Concurrent_Requests]
	FROM sys.dm_exec_requests R
	INNER JOIN sys.databases D ON D.database_id = R.database_id
	AND D.name = 'MyDatabase'

这只是某一时刻的快照。若要更好地了解工作负荷和并发请求需求，需在一定时间内收集多个样本。

### 最大并发登录数
你可以通过分析用户和应用程序模式来了解登录频率。还可以在测试环境中运行实际负荷，确保不会超过本文所介绍的这样或那样的限制。无法通过单一查询或动态管理视图 (DMV) 了解并发登录计数或历史记录。

如果多个客户端使用相同的连接字符串，该服务也会对每个登录名进行身份验证。如果 10 个用户使用相同的用户名和密码同时连接到数据库，则会有 10 个并发登录。此限制仅针对使用登录名进行身份验证的那段时间。如果这 10 个用户依次连接到数据库，则并发登录数始终不会超过 1。

>[AZURE.NOTE] 此限制目前不适用于弹性池中的数据库。

### 最大会话数
若要查看当前的活动会话数，请在 SQL 数据库中运行以下 Transact-SQL 查询：

	SELECT COUNT(*) AS [Sessions]
	FROM sys.dm_exec_connections

若要分析本地 SQL Server 工作负荷，可以对查询进行修改，使之专注于特定的数据库。此查询有助于确定数据库可能的会话需求（如果考虑将其移至 Azure SQL 数据库）。

	SELECT COUNT(*)  AS [Sessions]
	FROM sys.dm_exec_connections C
	INNER JOIN sys.dm_exec_sessions S ON (S.session_id = C.session_id)
	INNER JOIN sys.databases D ON (D.database_id = S.database_id)
	WHERE D.name = 'MyDatabase'

同样，这些查询返回时间点计数。如果在一段时间内收集多个样本，则可更好地了解会话使用情况。

进行 SQL 数据库分析时，可以获取会话的历史统计信息。查询 **sys.resource\_stats**，并使用 **active\_session\_count** 列。请参阅下一部分，详细了解如何使用此视图。

##<a name="monitor-resource-use"></a> 监视资源使用情况
可以通过两个视图监视 SQL 数据库相对于其服务层的资源使用情况：

- [sys.dm\_db\_resource\_stats](https://msdn.microsoft.com/zh-cn/library/dn800981.aspx)
- [sys.resource\_stats](https://msdn.microsoft.com/zh-cn/library/dn269979.aspx)

###<a name="monitoring-resource-use-with-sysresourcestats"></a> sys.dm\_db\_resource\_stats
可以使用每个 SQL 数据库中的 [sys.dm\_db\_resource\_stats](https://msdn.microsoft.com/zh-cn/library/dn800981.aspx) 视图。**sys.dm\_db\_resource\_stats** 视图显示相对于服务层的最新资源使用数据。CPU 平均百分比、数据 I/O、日志写入以及内存每 15 秒记录一次，持续记录 1 小时。

由于此视图提供了资源使用方面的更细致的信息，请首先使用 **sys.dm\_db\_resource\_stats** 进行当前状态分析或故障排除。例如，此查询显示了当前数据库在过去 1 小时的平均资源使用率和最大资源使用率：

    SELECT  
        AVG(avg_cpu_percent) AS 'Average CPU use in percent',
        MAX(avg_cpu_percent) AS 'Maximum CPU use in percent',
        AVG(avg_data_io_percent) AS 'Average data I/O in percent',
        MAX(avg_data_io_percent) AS 'Maximum data I/O in percent',
        AVG(avg_log_write_percent) AS 'Average log write use in percent',
        MAX(avg_log_write_percent) AS 'Maximum log write use in percent',
        AVG(avg_memory_usage_percent) AS 'Average memory use in percent',
        MAX(avg_memory_usage_percent) AS 'Maximum memory use in percent'
    FROM sys.dm_db_resource_stats;  

对于其他查询，请参阅 [sys.dm\_db\_resource\_stats](https://msdn.microsoft.com/zh-cn/library/dn800981.aspx) 中的示例。

### sys.resource\_stats
**master** 数据库中的 [sys.resource\_stats](https://msdn.microsoft.com/zh-cn/library/dn269979.aspx) 视图所提供的其他信息可用于监视 SQL 数据库特定服务层和性能级别的性能。数据每 5 分钟收集一次，持续大约 35 天。此视图适用于对 SQL 数据库资源使用情况进行较长期的历史分析。

下图显示了性能级别为 P2 的高级数据库在一周内每小时的 CPU 资源使用情况。此图从星期一开始显示，先显示 5 个工作日，然后显示周末，应用程序在周末使用的资源要少得多。

![SQL 数据库资源使用情况](./media/sql-database-performance-guidance/sql_db_resource_utilization.png)  


从数据来看，此数据库当前的峰值 CPU 负载相对于 P2 性能级别刚好超过 50% 的 CPU 使用率（周二午间）。如果 CPU 是应用程序资源分布曲线中的决定因素，则可确定 P2 就是适当的性能级别，它能够保证工作负荷始终适当。如果预期应用程序的资源使用会随时间而增长，则最好是设置额外的资源缓冲，使应用程序不会达到性能级别限制。如果提高性能级别，则可避免当数据库的功能无法有效处理请求时会发生的错误（这种错误客户也能看到，尤其是在对延迟很敏感的环境中）。例如，如果数据库支持的应用程序根据数据库调用结果绘制网页，则属于这种情况。

请注意，其他应用程序类型对同一图形可能有不同的解释。例如，如果某个应用程序尝试每天处理工资数据并使用相同的图表，则使用 P1 性能级别也许就能让此类“批处理作业”模型正常工作。P1 性能级别有 100 个 DTU，而 P2 性能级别有 200 个 DTU。P1 性能级别提供的性能是 P2 性能级别的一半。因此，P2 级别 50% 的 CPU 使用率相当于 P1 级别 100% 的 CPU 使用率。如果应用程序没有设置超时，则即使有作业耗时 2 小时或 2.5 小时才完成也无关紧要，只要今天完成即可。此类别的应用程序也许只需使用 P1 性能级别。一个事实是，白天有几个时段的资源使用率较低，因此可充分利用这一点，将“大高峰”作业分配一部分到当天晚些时候的某个资源使用低谷。只要作业每天可按时完成，P1 性能级别就可能很适合该类应用程序（并可节省资金）。

Azure SQL 数据库在每个服务器的 **master** 数据库的 **sys.resource\_stats** 视图中公开每个活动数据库使用的资源信息。表中的数据以 5 分钟为间隔收集而得。使用基本、标准和高级服务层，可能要在超过 5 分钟后数据才会出现在表中，因此这些数据更适合进行历史分析而非近实时分析。查询 **sys.resource\_stats** 视图可以看到数据库的最近历史记录，并可验证所选保留设置是否提供按需性能。

>[AZURE.NOTE] 在以下示例中，必须连接到逻辑 SQL 数据库服务器的 **master** 数据库才能查询 **sys.resource\_stats**。

以下示例显示了如何公开此视图中的数据：

	SELECT TOP 10 *
	FROM sys.resource_stats
	WHERE database_name = 'resource1'
	ORDER BY start_time DESC

![sys.resource\_stats 目录视图](./media/sql-database-performance-guidance/sys_resource_stats.png)  


下一示例演示了如何通过不同方式使用 **sys.resource\_stats** 目录视图，了解 SQL 数据库如何使用资源：

1. 若要查看数据库 userdb1 在过去一周的资源使用情况，可运行以下查询：

		SELECT *
		FROM sys.resource_stats
		WHERE database_name = 'userdb1' AND
		      start_time > DATEADD(day, -7, GETDATE())
		ORDER BY start_time DESC;

2. 若要评估工作负荷在相应性能级别的适合程度，需深入查看资源指标的每个方面：CPU、读取次数、写入次数、辅助进程数和会话数。下面是修订后的查询，使用 **sys.resource\_stats** 报告这些资源指标的平均值和最大值：
   
        SELECT
            avg(avg_cpu_percent) AS 'Average CPU use in percent',
            max(avg_cpu_percent) AS 'Maximum CPU use in percent',
            avg(avg_data_io_percent) AS 'Average physical data I/O use in percent',
            max(avg_data_io_percent) AS 'Maximum physical data I/O use in percent',
            avg(avg_log_write_percent) AS 'Average log write use in percent',
            max(avg_log_write_percent) AS 'Maximum log write use in percent',
            avg(max_session_percent) AS 'Average % of sessions',
            max(max_session_percent) AS 'Maximum % of sessions',
            avg(max_worker_percent) AS 'Average % of workers',
            max(max_worker_percent) AS 'Maximum % of workers'
        FROM sys.resource_stats
        WHERE database_name = 'userdb1' AND start_time > DATEADD(day, -7, GETDATE());
3. 使用每个资源指标的上述平均值和最大值信息，可以评估工作负荷在所选性能级别的适合程度。通常，来自 **sys.resource\_stats** 的平均值可提供一个用于目标大小的良好基准。它应该是你的主要测量标杆。例如，你可能正在使用性能级别为 S2 的标准服务层。CPU 以及 I/O 读取和写入的平均使用百分比低于 40%，平均辅助进程数低于 50，平均会话数低于 200。你的工作负荷可能适合 S1 性能级别。很轻松就能判断数据库是否在辅助进程和会话限制范围内。若要判断数据库在 CPU、读取和写入操作方面是否适合更低的性能级别，请将更低性能级别的 DTU 数量除以当前性能级别的 DTU 数量，再乘以 100：
   
    **S1 DTU / S2 DTU * 100 = 20 / 50 * 100 = 40**
   
    该结果是两个性能级别之间的相对性能差异（百分比）。如果资源使用不超出此量，你的工作负荷可能适合更低的性能级别。但是，你需要查看资源用量值的所有范围，并确定数据库工作负荷适合更低性能级别的频率（以百分比计）。以下查询将会根据以上示例计算得出的阈值 40%，输出每个资源维度的适合性百分比：
   
        SELECT
            (COUNT(database_name) - SUM(CASE WHEN avg_cpu_percent >= 40 THEN 1 ELSE 0 END) * 1.0) / COUNT(database_name) AS 'CPU Fit Percent'
            ,(COUNT(database_name) - SUM(CASE WHEN avg_log_write_percent >= 40 THEN 1 ELSE 0 END) * 1.0) / COUNT(database_name) AS 'Log Write Fit Percent'
            ,(COUNT(database_name) - SUM(CASE WHEN avg_data_io_percent >= 40 THEN 1 ELSE 0 END) * 1.0) / COUNT(database_name) AS 'Physical Data IO Fit Percent'
        FROM sys.resource_stats
        WHERE database_name = 'userdb1' AND start_time > DATEADD(day, -7, GETDATE());
   
    可以根据数据库服务级别目标 (SLO) 确定工作负荷是否适合更低性能级别。如果数据库工作负荷 SLO 为 99.9%，而上述查询针对所有三个资源维度返回的值大于 99.9%，则工作负荷可能适合更低性能级别。
   
    查看适合性百分比还可以深入分析是否需要转到下一个更高的性能级别以满足 SLO。例如，userdb1 显示过去一周的 CPU 使用情况如下：
   
   | 平均 CPU 百分比 | 最大 CPU 百分比 |
   | --- | --- |
   | 24.5 |100.00 |
   
    平均 CPU 大约是性能级别限制的四分之一，它应与数据库的性能级别非常匹配。但是，最大值显示该数据库达到了性能级别的限制。在这种情况下，是否需要转到下一个更高的性能级别？你需要查看工作负荷达到 100% 的次数，然后将其与数据库工作负荷 SLO 进行比较。
   
        SELECT
        (COUNT(database_name) - SUM(CASE WHEN avg_cpu_percent >= 100 THEN 1 ELSE 0 END) * 1.0) / COUNT(database_name) AS 'CPU fit percent'
        ,(COUNT(database_name) - SUM(CASE WHEN avg_log_write_percent >= 100 THEN 1 ELSE 0 END) * 1.0) / COUNT(database_name) AS 'Log write fit percent'
        ,(COUNT(database_name) - SUM(CASE WHEN avg_data_io_percent >= 100 THEN 1 ELSE 0 END) * 1.0) / COUNT(database_name) AS 'Physical data I/O fit percent'
        FROM sys.resource_stats
        WHERE database_name = 'userdb1' AND start_time > DATEADD(day, -7, GETDATE());
   
    如果对于三个资源维度中的任何一个维度，此查询返回的值小于 99.9%，请考虑转到下一个更高的性能级别，或使用应用程序优化技术来减少 SQL 数据库上的负载。
4. 此做法还考虑到了未来预计的工作负荷增长。

## 优化应用程序
在传统的本地 SQL Server 中，进行初始容量规划的过程经常与在生产中运行应用程序的过程分离。首先购买硬件和产品许可证，然后进行性能优化。使用 Azure SQL 数据库时，最好是交替完成应用程序的运行和优化过程。通过按需为容量付款的模型，可以优化应用程序，让其只需使用目前需要的最少资源，而非靠推测应用程序的未来增长计划来过度预配硬件（这些计划通常是不正确的）。某些客户可能不选择优化应用程序，而是选择过度预配硬件资源。如果不想在繁忙时段更改关键应用程序，不妨使用此方法。但是，在使用 Azure SQL 数据库中的服务层时，优化应用程序可将资源需求降至最低并减少每月的费用。

### 应用程序特征
尽管可以利用 Azure SQL 数据库服务层提高应用程序的性能稳定性和可预测性，但也可通过某些最佳实践优化应用程序，使之能够更好地利用某个性能级别的资源。尽管许多应用程序只需切换到更高性能级别或服务层即可大幅提升性能，但某些应用程序仍需经过额外的优化才能充分利用更高级别的服务。若要提高性能，可考虑对具有以下特征的应用程序进行额外的优化：

* **应用程序因“健谈”行为而降低性能**。“健谈”应用程序会过多地进行易受网络延迟影响的数据访问操作。可能需要修改这些类型的应用程序，减少对 SQL 数据库的数据访问操作的数量。例如，可使用将即席查询成批处理或将查询移至存储过程等方法，提高应用程序性能。有关详细信息，请参阅[批处理查询](#batch-queries)。
* **数据库工作负荷过大，超出单台计算机的处理能力**。如果数据库所需资源超出最高的高级性能级别，则可考虑横向扩展工作负荷。有关详细信息，请参阅[跨数据库分片](#cross-database-sharding)和[功能分区](#functional-partitioning)。
* **应用程序的查询性能欠佳**。应用程序（尤其是数据访问层的应用程序）如果查询优化不好，则可能无法利用更高性能级别的优势。其中包括缺少 WHERE 子句、缺少索引或统计信息过时的查询。标准查询性能优化技术能够为这些应用程序带来好处。有关详细信息，请参阅[缺少索引](#missing-indexes)和[查询优化和提示](#query-tuning-and-hinting)。
* **应用程序的数据访问设计欠佳**。更高的性能级别可能无法给存在固有数据访问并发问题（例如死锁）的应用程序带来好处。考虑使用 Azure 缓存服务或其他缓存技术将数据缓存在客户端，减少与 Azure SQL 数据库之间的往返次数。请参阅[应用程序层缓存](#application-tier-caching)。

## 优化方法
本部分介绍可用于优化 Azure SQL 数据库的某些方法，这些方法可使应用程序达到最佳性能并以尽可能低的性能级别运行。有些方法可与传统的 SQL Server 优化最佳实践搭配使用，但有些方法专用于 Azure SQL 数据库。在某些情况下，可通过检查数据库使用的资源找到要进一步优化的区域，扩展传统的 SQL Server 方法，使这些方法也可在 Azure SQL 数据库中使用。

### Azure 门户预览工具
Azure 门户预览有两种工具，用于分析和修复 SQL 数据库性能问题：

- [查询性能见解](/documentation/articles/sql-database-query-performance/)
- [SQL 数据库顾问](/documentation/articles/sql-database-advisor/)

Azure 门户预览有这两种工具及其使用方法的详细说明。若要有效地诊断和纠正问题，建议首先在 Azure 门户预览中尝试这些工具。建议使用手动优化方法来解决索引缺失和查询优化问题，这一点我们随后会用特例进行介绍。

###<a name="missing-indexes"></a> 缺少索引
OLTP 数据库性能有一个常见问题与物理数据库设计有关。设计和交付数据库架构时，经常不进行规模（负载或数据卷）测试。遗憾的是，在规模较小时，查询计划的性能可能尚可接受，但面对生产级数据卷时，性能就会大幅降低。此问题最常见的原因是缺乏相应的索引，无法满足筛选器的要求或查询中的其他限制。缺少索引经常导致表扫描，而此时索引搜寻即可满足要求。

在此示例中，所选查询计划在使用搜寻即可满足要求的情况下使用了扫描：

	DROP TABLE dbo.missingindex;
	CREATE TABLE dbo.missingindex (col1 INT IDENTITY PRIMARY KEY, col2 INT);
	DECLARE @a int = 0;
	SET NOCOUNT ON;
	BEGIN TRANSACTION
	WHILE @a < 20000
	BEGIN
	    INSERT INTO dbo.missingindex(col2) VALUES (@a);
	    SET @a += 1;
	END
	COMMIT TRANSACTION;
	GO
	SELECT m1.col1
	FROM dbo.missingindex m1 INNER JOIN dbo.missingindex m2 ON(m1.col1=m2.col1)
	WHERE m1.col2 = 4;

![缺少索引的查询计划](./media/sql-database-performance-guidance/query_plan_missing_indexes.png)  


Azure SQL 数据库可用于查找和修复常见的索引缺失情况。Azure SQL 数据库内置的 DMV 将查找其中索引会大幅降低运行查询的估算成本的查询编译。在查询执行期间，SQL 数据库跟踪每个查询计划的执行频率，以及跟踪执行查询计划与想象其中存在该索引的查询计划之间的估算差距。可以使用这些 DMV 迅速推测出哪些物理数据库设计更改可能减少数据库的总工作负荷成本及其真实工作负荷。

此查询可用于评估可能缺少的索引：

	SELECT CONVERT (varchar, getdate(), 126) AS runtime,
	    mig.index_group_handle, mid.index_handle,
	    CONVERT (decimal (28,1), migs.avg_total_user_cost * migs.avg_user_impact *
	            (migs.user_seeks + migs.user_scans)) AS improvement_measure,
	    'CREATE INDEX missing_index_' + CONVERT (varchar, mig.index_group_handle) + '_' +
	              CONVERT (varchar, mid.index_handle) + ' ON ' + mid.statement + '
	              (' + ISNULL (mid.equality_columns,'')
	              + CASE WHEN mid.equality_columns IS NOT NULL
	                          AND mid.inequality_columns IS NOT NULL
	                     THEN ',' ELSE '' END + ISNULL (mid.inequality_columns, '')
	              + ')'
	              + ISNULL (' INCLUDE (' + mid.included_columns + ')', '') AS create_index_statement,
	    migs.*,
	    mid.database_id,
	    mid.[object_id]
	FROM sys.dm_db_missing_index_groups AS mig
	INNER JOIN sys.dm_db_missing_index_group_stats AS migs
	    ON migs.group_handle = mig.index_group_handle
	INNER JOIN sys.dm_db_missing_index_details AS mid
	    ON mig.index_handle = mid.index_handle
	ORDER BY migs.avg_total_user_cost * migs.avg_user_impact * (migs.user_seeks + migs.user_scans) DESC

在此示例中，查询生成了以下建议：

	CREATE INDEX missing_index_5006_5005 ON [dbo].[missingindex] ([col2])  

创建建议以后，同一 SELECT 语句会选取另一计划，使用搜寻而非扫描，从而提高计划执行效率：

![已更正索引的查询计划](./media/sql-database-performance-guidance/query_plan_corrected_indexes.png)  


其中的要点是共享商用系统与专用服务器计算机相比，I/O 容量更加有限。客观上鼓励将多余 I/O 降至最低，最大限度地在 Azure SQL 数据库服务层的每个性能级别 DTU 范围内利用系统。选择适当的物理数据库设计方式可显著缩短单个查询的延迟、提高按缩放单位处理的并发请求的吞吐量，以及将满足查询所需的成本降至最低。有关缺少索引 DMV 的详细信息，请参阅 [sys.dm\_db\_missing\_index\_details](https://msdn.microsoft.com/zh-cn/library/ms345434.aspx)。

###<a name="query-tuning-and-hinting"></a> 查询优化和提示
Azure SQL 数据库中的查询优化器与传统的 SQL Server 查询优化器相似。有关优化查询和了解查询优化器的推理模型限制的最佳实践大多也适用于 Azure SQL 数据库。如果优化 Azure SQL 数据库中的查询，则可能获得额外的优势：降低总体资源需求。应用程序可以在更低的性能级别运行，因此相对于不进行优化，可以降低运行成本。

在 SQL Server 中常见的一个示例（也适用于 Azure SQL 数据库）是查询优化器如何“探查”参数。在编译期间，查询优化器会计算参数的当前值，确定其是否能够生成更优化的查询计划。尽管与不使用已知参数值进行编译的计划相比，此策略通常能够生成速度明显更快的查询计划，但目前在 SQL Server 和 Azure SQL 数据库中的使用效果仍欠佳。有时不探查参数，有时虽对参数进行探查，但生成的计划对于工作负荷中的整套参数值来说效果欠佳。Microsoft 提供查询提示（指令），让用户可以更谨慎地指定意图并取代参数探查的默认行为。通常情况下，如果使用提示，则可纠正默认的 SQL Server 或 Azure SQL 数据库行为对于给定客户工作负荷不完善的情况。

下一示例演示了查询处理器生成的计划无法完全满足性能和资源要求的情况。此示例还表明，如果使用查询提示，则可缩短 SQL 数据库的查询运行时间并降低资源要求：

	DROP TABLE psptest1;
	CREATE TABLE psptest1(col1 int primary key identity, col2 int, col3 binary(200));

	DECLARE @a int = 0;
	SET NOCOUNT ON;
	BEGIN TRANSACTION
	WHILE @a < 20000
	BEGIN
	    INSERT INTO psptest1(col2) values (1);
	    INSERT INTO psptest1(col2) values (@a);
	    SET @a += 1;
	END
	COMMIT TRANSACTION
	CREATE INDEX i1 on psptest1(col2);
	GO

	CREATE PROCEDURE psp1 (@param1 int)
	AS
	BEGIN
	    INSERT INTO t1 SELECT * FROM psptest1
	    WHERE col2 = @param1
	    ORDER BY col2;
	END
	GO

	CREATE PROCEDURE psp2 (@param2 int)
	AS
	BEGIN
	    INSERT INTO t1 SELECT * FROM psptest1 WHERE col2 = @param2
	    ORDER BY col2
	    OPTION (OPTIMIZE FOR (@param2 UNKNOWN))
	END
	GO

	CREATE TABLE t1 (col1 int primary key, col2 int, col3 binary(200));
	GO

该设置代码将创建一个其数据分布处于偏斜状态的表。最佳查询计划随所选参数的不同而不同。遗憾的是，计划缓存行为并非始终根据最常用参数值来重新编译查询。因此，对于许多值来说，即使平均情况下选择其他计划可能效果更佳，也可能会缓存和使用一个非最优的计划。然后，查询计划会创建两个几乎相同的存储过程，唯一区别是其中一个有特殊的查询提示。

**示例，第 1 部分**

	-- Prime Procedure Cache with scan plan
	EXEC psp1 @param1=1;
	TRUNCATE TABLE t1;

	-- Iterate multiple times to show the performance difference
	DECLARE @i int = 0;
	WHILE @i < 1000
	BEGIN
	    EXEC psp1 @param1=2;
	    TRUNCATE TABLE t1;
	    SET @i += 1;
	END

**示例，第 2 部分**

（建议在开始示例的第 2 部分之前至少等待 10 分钟，这样，生成的遥测数据中的结果就会截然不同。）

	EXEC psp2 @param2=1;
	TRUNCATE TABLE t1;

	DECLARE @i int = 0;
	WHILE @i < 1000
	BEGIN
	    EXEC psp2 @param2=2;
	    TRUNCATE TABLE t1;
	    SET @i += 1;
	END

本例的每个部分均尝试将某个参数化插入语句运行 1,000 次（以产生可用作测试数据集的足够的负载）。当执行存储过程时，查询处理器在其首次编译期间检查传递给过程的参数值（参数“探查”）。处理器会缓存生成的计划，将其用于以后的调用，即使参数值不同也是如此。可能无法在所有情况下均使用最佳计划。有时，用户需要引导优化器选取更适合普通情况而非首次编译查询时的特定情况的计划。在此示例中，初始计划将生成一个“扫描”计划，后者会读取所有行以查找与参数匹配的每个值：

![通过使用扫描计划进行查询优化](./media/sql-database-performance-guidance/query_tuning_1.png)  


由于我们用值 1 执行该过程，因此所得的计划对于值 1 为最佳，但对于表中的所有其他值并非最佳。如果随机选取每个计划，结果可能不如所愿，因为计划的执行速度可能较慢，所用资源可能较多。

如果运行测试时将 `SET STATISTICS IO` 设置为 `ON`，则会在后台完成此示例中的逻辑扫描工作。可以看到该计划进行了 1,148 次读取（如果平均情况下只返回一行，这样的效率不高）：

![通过使用逻辑扫描进行查询优化](./media/sql-database-performance-guidance/query_tuning_2.png)  


本例的第二部分使用查询提示告知优化器在编译过程中使用某个特定值。在本示例中，它强制查询处理器忽略作为参数传递的值，而采用 `UNKNOWN`。这是指在表中的出现频率为平均频率的值（忽略偏斜情况）。所得的计划是一个基于搜寻的计划，平均而言，它比此示例第 1 部分中的计划速度更快，使用资源更少：

![通过使用查询提示进行查询优化](./media/sql-database-performance-guidance/query_tuning_3.png)  


可以在 **sys.resource\_stats** 表中查看效果（从执行测试到数据填充到表中存在时间上的延迟）。就本示例来说，在 22:25:00 时间范围执行第 1 部分，在 22:35:00 执行第 2 部分。请注意，越早的时间范围使用的资源比越晚的时间范围要多（因计划效率提高）。

	SELECT TOP 1000 *
	FROM sys.resource_stats
	WHERE database_name = 'resource1'
	ORDER BY start_time DESC

![查询优化示例结果](./media/sql-database-performance-guidance/query_tuning_4.png)  


> [AZURE.NOTE]
虽然此示例中的卷是特意选择的小卷，但非最佳参数的影响仍很大，较大的数据库尤其如此。这种区别在极端情况下可能在数秒（快速情况）和数小时（慢速情况）之间。
> 
> 

可检查 **sys.resource\_stats**，确定某个测试使用的资源多于还是少于另一个测试。在比较数据时，请让测试相隔足够长的时间，使其不会在 **sys.resource\_stats** 视图中的同一 5 分钟时间范围内重合。本练习的目标是将使用的资源总量降至最低，而非将峰值资源降至最低。一般而言，优化一段产生延迟的代码也将减少资源消耗。请确保对应用程序所做的更改是必需的，且这些更改不会对那些可能会在应用程序中使用查询提示的人的客户体验造成负面影响。

如果工作负荷由一组重复的查询组成，则捕获并验证所做计划选择的最优性通常很有意义，因为这样做会使托管数据库所需的资源大小单位降至最低。验证最优性以后，可以偶尔对计划进行重新检查，确保其未降级。可以详细了解[查询提示 (Transact-SQL)](https://msdn.microsoft.com/zh-cn/library/ms181714.aspx)。

###<a name="cross-database-sharding"></a> 跨数据库分片
由于 Azure SQL 数据库运行在商用硬件上，因此单一数据库的容量限制低于传统的本地 SQL Server 安装。某些客户使用分片技术在数据库操作不符合 Azure SQL 数据库中单一数据库的限制时，将这些操作分摊到多个数据库上。在 Azure SQL 数据库上使用分片技术的大多数客户将单个维度的数据拆分到多个数据库上。对于该方法，需了解 OLTP 应用程序执行的事务经常仅适用于架构中的一行或少数几行。

>[AZURE.NOTE] SQL 数据库现在提供一个库来帮助分片。有关详细信息，请参阅[弹性数据库客户端库概述](/documentation/articles/sql-database-elastic-database-client-library/)。

例如，如果数据库包含客户名称、订单和订单详细信息（就像 SQL Server 附带的传统示例 Northwind 数据库一样），则可将客户与相关订单和订单详细信息集中在一起，从而将这些数据拆分到多个数据库中。可以保证客户的数据留在单一数据库中。应用程序将不同的客户拆分到多个数据库上，实际上就是将负载分散在多个数据库上。通过分片，客户不仅可以避免达到最大数据库大小限制，而且还使 Azure SQL 数据库能够处理明显大于不同性能级别限制的工作负荷，前提是每个数据库适合其 DTU。

数据库分片不会减少解决方案的聚合资源容量，但在支持跨多个数据库的极大型解决方案时很有效。每个数据库都可以在不同性能级别运行，因此支持资源要求高的极大型“高效”数据库。

###<a name="functional-partitioning"></a> 功能分区
SQL Server 用户经常将许多功能集中在单一数据库中。例如，如果应用程序包含管理商店库存的逻辑，则该数据库可能包含与库存、跟踪采购订单、存储过程、管理月末报告的索引视图或具体化视图关联的逻辑。此方法可轻松管理数据库，进行备份之类的操作，但也要求用户调整硬件大小以处理应用程序所有功能的峰值负载。

如果在 Azure SQL 数据库中使用横向扩展体系结构，则可将应用程序的不同功能拆分到不同的数据库中。每个应用程序均可使用此方法独立缩放。随着应用程序变得更加繁忙（并且数据库上出现更多负载），管理员可针对应用程序中的每项功能单独选择性能级别。在限制范围内，使用此体系结构时，由于负载分散在多个计算机上，因此应用程序的规模可超出单个商用计算机的处理能力。

###<a name="batch-queries"></a> 批处理查询
对于以大量、频繁的即席查询形式访问数据的应用程序，在应用程序层与 Azure SQL 数据库层之间的网络通信上花费了大量响应时间。即使在应用程序与 Azure SQL 数据库同处一个数据中心时，大量数据访问操作也可能会增大二者之间的网络延迟。若要减少进行数据访问操作所需的网络往返，可考虑使用相应选项，要么批处理即席查询，要么将其编译为存储过程。如果将即席查询分批，可将多个查询作为一个大批次在一次行程中发送到 Azure SQL 数据库。将即席查询编入存储过程可获得与分批相同的结果。使用存储过程还有一个好处，即可以有更多的机会将查询计划缓存在 Azure SQL 数据库中，以便再次使用存储过程。

某些应用程序频繁写入。有时可以考虑通过适当方法来统一批处理写入，减少数据库上的总 I/O 负载。通常，这与在存储过程和即席批处理中使用显式事务代替自动提交事务一样简单。如需对各种可用方法的评估，请参阅 [Azure 中 SQL 数据库应用程序的批处理方法](https://msdn.microsoft.com/zh-cn/library/windowsazure/dn132615.aspx)。使用自己的工作负荷进行实验，找到正确的批处理模型。请务必了解，模型的事务一致性保证可能略有不同。若要找到将资源用量降至最低的正确工作负荷，需要找到一致性与性能折中的正确组合。

###<a name="application-tier-caching"></a> 应用程序层缓存
某些数据库应用程序的工作负荷包含大量的读取操作。缓存层可以减少数据库上的负载，还可以降低通过 Azure SQL 数据库提供数据库支持所需的性能级别。通过 [Azure Redis 缓存](/home/features/redis-cache/)，读取工作负荷较重的客户只需读取数据一次（也许只需对每个应用程序层计算机读取一次，具体取决于其配置方式），然后将该数据存储在 SQL 数据库外部。这种方式可降低数据库负载（CPU 和读取 I/O），但对于事务一致性有影响，因为从缓存读取的数据可能与数据库中的数据异步。虽然许多应用程序可接受一定程度的不一致，但并非所有工作负荷都是这样。应该先完全了解任何应用程序要求，然后再实施应用程序层缓存策略。

## 后续步骤

- 有关服务层的详细信息，请参阅 [SQL 数据库选项和性能](/documentation/articles/sql-database-service-tiers/)
- 有关弹性池的详细信息，请参阅 [Azure 弹性池是什么？](/documentation/articles/sql-database-elastic-pool/)
- 有关性能和弹性池的信息，请参阅[何时考虑弹性池](/documentation/articles/sql-database-elastic-pool-guidance/)

<!---HONumber=Mooncake_0116_2017-->
<!--Update_Description: add premium RS service tier-->