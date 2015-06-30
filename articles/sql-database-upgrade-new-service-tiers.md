<properties title="Upgrade SQL Database Web/Business Databases to New Service Tiers" pageTitle="将 SQL Database Web/企业数据库升级到新服务层" description="将 Azure SQL Database Web 或企业数据库升级到新的 Azure SQL Database 服务层/性能级别。" metaKeywords="Azure, SQL. DB, Service, Tier, web, business" services="Azure SQL DB" solutions="upgrade, migrate, move, monitor, t-sql, powershell" documentationCenter="" authors="Srini Acharya, Umachandar Jayachandran" manager="jeffreyg" videoId="" scriptId="" />

<tags ms.service="sql-database" ms.date="03/23/2014" wacn.date="05/25/2015" ms.author="jhubbard" />


# 将 SQL Database Web/企业数据库升级到新服务层

Azure Web 角色和企业 SQL 数据库在共享的多租户环境中运行，这种环境没有为数据库提供任何保留的资源容量。群集中其他数据库的活动可能会影响你的性能。资源在任意给定时间点的可用性很大程度上取决于系统中运行的其他并发工作负载。这可能会导致数据库应用程序性能出现很大的变化和不可预测性。客户反映，这种不可预测的性能很难管理，他们更希望性能可预测。 

为了迎合这种需求，Azure SQL Database 服务引入了新的数据库服务层[(基本、标准和高级)](http://azure.microsoft.com/blog/2014/08/26/new-azure-sql-database-service-tiers-generally-available-in-september-with-reduced-pricing-and-enhanced-sla)，这些版本提供可预测的性能和丰富的新功能，可实现业务连续性和安全性。这些新服务层旨在为数据库工作负载提供指定级别的资源，而不考虑该环境中运行的其他客户工作负载。这样，便使得性能行为高度可预测。随着此版本的发布，Web 和企业服务层即将弃用，最终会在 2015 年 9 月停用。 

这些变化为客户带来了这样的问题：如何评估新的服务层，哪个新的服务层最适合他们目前的 Web 和企业 (W/B) 数据库，以及如何运作实际升级流程自身。本文以引导式的方法介绍了如何将 Web/企业数据库升级到新的服务层/性能级别。

将 Web/企业数据库升级到新服务层的过程包括以下步骤：

1.	确定哪个服务层可以提供满足应用程序或业务要求的最低功能级别
2.	根据历史资源使用情况确定性能级别
3.	升级到选择的服务层/性能级别
4.	监视数据库在升级后的运行情况，并根据需要调整服务层/性能级别

## 1. 根据功能确定服务层

确定哪个服务层可以提供满足应用程序或业务要求的最低功能级别。例如，考虑备份的保留期限、需要标准功能还是活动地域复制功能、是否需要最大的数据库大小等要求。这些要求将会影响最小服务层的选择（请参阅[服务层通告博客）。 ](http://azure.microsoft.com/blog/2014/08/26/new-azure-sql-database-service-tiers-generally-available-in-september-with-reduced-pricing-and-enhanced-sla) 

"基本"层主要用于活动较少的极小型数据库。因此，如果要升级的话，在选择服务层时，你通常会根据功能要求从"标准"或"高级"层开始。有关并发请求/会话的服务层/性能级别特定限制，请参阅 [Azure 文档](http://msdn.microsoft.com/zh-cn/library/azure/dn741336.aspx)。

## 2. 根据资源使用情况确定性能级别 
Azure SQL Database 服务在 sys.resource_stats 视图中提供 Web/企业数据库资源消耗量[（请参阅 MSDN 文档）](http://msdn.microsoft.com/zh-cn/library/azure/dn269979.aspx)，该视图位于当前数据库所在的逻辑服务器的 master 数据库中。其中将以性能级别限制百分比显示资源消耗数据。此视图以 5 分钟为间隔提供过去最多 14 天的数据。由于 Web/企业数据库并不保证有任何关联的 DTU/资源限制，因此，我们在 S2 性能级别中数据库可用资源量方面规范化了百分比值。在任何特定间隔内，数据库的平均 DTU 消耗百分比可计算为该间隔内 CPU、IO 和日志使用量的最高百分比值。对 master 数据库运行以下查询可检索数据库的平均 DTU 消耗量：

 
                   
     SELECT start_time, end_time
	 ,(SELECT Max(v)
         FROM (VALUES (avg_cpu_percent)
                    , (avg_physical_data_read_percent)
                    , (avg_log_write_percent)
    	   ) AS value(v)) AS [avg_DTU_percent]
    FROM sys.resource_stats
    WHERE database_name = '<your db name>'
    ORDER BY end_time DESC;

使用与 S2 数据库级别相关的 DTU 消耗信息可以规范化与新层数据库相关的 Web/企业数据库当前消耗量，并确定这些服务层更适合的级别。例如，如果平均 DTU 消耗百分比显示值 80%，则表示数据库正在消耗 S2 性能级别数据库限制的 80%。如果你在 sys.resource_stats 视图中发现值大于 10%，则表示你需要大于 S2 的性能层。例如，假设你看到峰值 DTU 百分比值 300%。这表示你使用的资源量超过了 S2 中可用资源量的三倍以上。若要确定合理的起始大小，请将 S2 中可用 DTU 数（50 个 DTU）与下一个更高大小（P1 = 100 个 DTU，即 S2 的 200%；P2 = 200 个 DTU，即 S2 的 400%）进行比较。由于你现在达到了 S2 的 300%，因此需要从 P2 开始并重新测试。

根据 DTU 使用百分比和适应工作负载所需的最高版本，你可以确定哪个服务层和性能级别最适合你的数据库工作负载（根据各个[性能级别的 DTU 百分比和相对 DTU 能力的指示）。](http://msdn.microsoft.com/zh-cn/library/azure/dn741336.aspx)下表提供了 Web/企业资源消耗百分比与相应新层性能级别之间的映射： 

![resource consumption](http://i.imgur.com/nPodQZH.png)

> **注意**：各个性能级别之间的相对 DTU 数量基于 [Azure SQL Database 基准](http://msdn.microsoft.com/zh-cn/library/azure/dn741327.aspx)工作负载。由于数据库工作负载可能不同于该基准，因此你应该使用以上计算方式作为指导，来确定 Web/企业数据库最初适合哪个新层。将数据库迁移到新层后，请使用上一部分概述的过程来验证/微调适合工作负载需要的适当服务层。
> 
> **注意**：尽管建议的新版服务层/性能级别考虑到了过去 14 天的数据库活动，但此数据基于过去 5 分钟的平均资源消耗数据样本数。因此，它可以会遗漏持续时间短于 5 分钟的短期活动喷发。应该将本指南用作升级数据库的起点。将数据库升级到建议的层后，需要进行更多的监视、测试和验证，同时，可以根据需要将数据库向上/向下迁移到不同的层/性能级别。

下面是对 master 数据库运行的一个查询，该查询将针对你的 Web/企业层数据库执行计算，然后根据每个 5 分钟数据采样间隔建议可能适合你工作负载的版本。

> **注意：**此查询仅适用于 Web/企业数据库，对于新层中的数据库，它**不会**提供正确的结果。

    WITH DTU_mapping AS
    ( SELECT *
        FROM ( VALUES (1, 10,'Basic'), (2, 20,'S0'), (3, 40,'S1'), (4, 100, 'S2')
                    , (5, 200, 'P1'), (6, 400,'P2'), (7, 1600, 'P3')
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

![Sample Result](http://i.imgur.com/CTnjv26.png)

在图表中，你可以看到不同时间的平均 DTU 消耗百分比变化趋势。下面是大部分时间处于 S2 级别中的某个数据库的示例图表，该数据库的某个高峰活动上冲到了 P1 数据库级别。各时间的 DTU 消耗量从"基本"限制最大升到了"P1"限制。若要使此数据库完全适合新层，你需要购买性能级别为"P1"的"高级"服务层数据库。另一方面，如果 P1 级别极少出现这种偶然喷发现象，则 S2 级别数据库也是可行的。

![DTU Usage](http://i.imgur.com/e4N4ay5.png)

## 3. 升级到新的服务层/性能级别
在确定 Web/企业数据库的服务层/性能级别后，可通过多种方法将数据库升级到新层：


- Azure 管理门户
- Azure PowerShell 
- [服务管理 REST API](http://msdn.microsoft.com/zh-cn/library/azure/dn505719.aspx)
- Transact-SQL 

要进行 Web/企业数据库即席迁移，请使用 [Azure 管理门户](https://manage.windowsazure.cn)。请参阅[更改数据库服务层和性能级别](http://msdn.microsoft.com/zh-cn/library/azure/dn369872.aspx)一文中的"升级到更高的服务层"部分，以了解使用 Azure 管理门户进行升级的步骤。

若要对服务器/订阅中的多个数据库执行操作或者要自动化过程，请使用 PowerShell 或 Transact-SQL 命令。

## 使用 Azure PowerShell 升级
你可以使用以下 Azure PowerShell，基于端到端的示例升级 Web/企业数据库。升级数据库的步骤如下：

1. 使用 **New-AzureSqlDatabaseServerContext** cmdlet 设置服务器上下文 
2. 使用 **Get-AzureSqlDatabase** cmdlet 获取要升级的数据库
3. 使用**Set-AzureSqlDatabase -ServiceObjective** 基于名称获取所需的性能级别（示例：S1、S2、P1）
4. 使用 **Set-AzureSqlDatabase** 指定数据库的新服务层和性能级别

下面显示了一个用于将服务器和数据库升级到所需服务层/性能级别的示例脚本：

    # Get the credential for the server. Provide login that is in dbmanager role of server
    $credential = Get-Credential
    # Provide server name in the cmdlet below:
    $serverContext = New-AzureSqlDatabaseServerContext -ServerName "<server_name>" -Credential $credential
    # Provide database name in the cmdlet below:
    $db = Get-AzureSqlDatabase $serverContext -DatabaseName "<database_name>"
    $service_tier = "<desired_service_tier>"
    $performance_level = Get-AzureSqlDatabaseServiceObjective $serverContext -ServiceObjectiveName "<desired_performance_level>"
    Set-AzureSqlDatabase $serverContext -Database $db -ServiceObjective $performance_level -Edition $service_tier


## 使用 Transact-SQL 升级

若要使用 Transact-SQL 升级现有的 Web/企业数据库，可以使用 
[ALTER DATABASE 语句](http://msdn.microsoft.com/zh-cn/library/azure/ms174269.aspx)。下面显示了用于将数据库升级到新服务层/性能级别的 ALTER DATABASE 命令：

    ALTER DATABASE <db_name> MODIFY (EDITION = '<service_tier>', SERVICE_OBJECTIVE = '<performance_level>');
      	 

## 4.	监视新服务层/性能级别的升级
Azure SQL Database 在 sys.dm_operation_status 动态管理视图中提供有关对数据库执行的管理操作（例如 CREATE、ALTER、DROP）的进度信息，该视图位于当前数据库所在的逻辑服务器的 master 数据库中。[请参阅 sys.dm_operation_status 文档。](http://msdn.microsoft.com/zh-cn/library/azure/dn270022.aspx)使用操作状态 DMV 可以确定数据库的升级操作进度。此示例查询显示了对数据库执行的所有管理操作：

    SELECT o.operation, o.state_desc, o.percent_complete
    , o.error_code, o.error_desc, o.error_severity, o.error_state
    , o.start_time, o.last_modify_time
    FROM sys.dm_operation_status AS o
    WHERE o.resource_type_desc = 'DATABASE'
    and o.major_resource_id = '<database_name>'
    ORDER BY o.last_modify_time DESC;

如果你使用管理门户执行了升级，则门户中也会提供有关该操作的通知。

## 升级的影响
将 Web/企业数据库升级到新服务层/性能级别是一个联机操作。在整个升级操作过程中，你的数据库将继续工作。在实际过渡到新的性能级别时，可能会临时断开与该数据库的连接，不过持续时间很短（几秒钟）。如果应用程序针对连接终止启用了暂时性故障处理，则足以防止升级结束时出现连接断开的情况。有关处理连接终止的最佳实践，请参阅[防止拒绝请求或终止连接的 Azure SQL Database 最佳实践](http://msdn.microsoft.com/zh-cn/library/azure/dn338082.aspx)。

有关如何估计升级操作延迟的详细信息，请参阅[更改数据库服务层和性能级别](http://msdn.microsoft.com/zh-cn/library/azure/dn369872.aspx)中的"了解和估计数据库更改的延迟"部分。

## 升级后数据库工作负载达到资源限制时会发生什么情况？
性能级别将会校准并受到控制，以便在选定服务层/性能级别所允许的最大限制范围内（即资源消耗量达到 100%）提供所需的资源来运行你的数据库工作负载。如果你的工作负载达到了 CPU/数据 IO/日志 IO 限制之一，你将会继续接收资源直到达到最大允许级别，但是，你可能会发现你的查询延迟不断增高。达到其中一个限制不会导致任何错误，而只会减慢你的工作负载，直到严重变慢，以致于查询开始超时。如果你达到了并发用户会话/请求（工作线程）的最大允许数目限制，在进一步建立任何连接/发出请求时，你将看到明确的错误（请参阅 MSDN 文档）。

## 监视数据库在升级后的运行情况
将 Web/企业数据库升级到新层后，建议你主动监视数据库，以确保应用程序以所需的性能运行，并根据需要优化使用方式。建议使用以下附加步骤来监视数据库。


**资源消耗数据：**对于基本、标准和高级数据库，可通过用户数据库中名为 [sys.dm_ db_ resource_stats](http://msdn.microsoft.com/zh-cn/library/azure/dn800981.aspx) 的新 DMV 查看更详细的资源消耗数据。此 DMV 针对前一小时的操作，以 15 秒的粒度级提供接近实时的资源消耗信息。每个间隔的 DTU 消耗百分比将计算为 CPU、IO 和日志维度的最大消耗百分比。下面是一个用于计算过去一小时平均 DTU 消耗百分比的查询：

    SELECT end_time
    	 , (SELECT Max(v)
             FROM (VALUES (avg_cpu_percent)
                         , (avg_data_io_percent)
                         , (avg_log_write_percent)
    	   ) AS value(v)) AS [avg_DTU_percent]
    FROM sys.dm_db_resource_stats
    ORDER BY end_time DESC;

 
其他[文档](http://msdn.microsoft.com/zh-cn/library/dn800981.aspx)包含了有关如何使用此 DMV 的详细信息。[Azure SQL Database 性能指南](http://msdn.microsoft.com/zh-cn/library/azure/dn369873.aspx)介绍了如何监视和优化你的应用程序。

**内存对性能的影响：**尽管内存是影响 DTU 等级的资源维度之一，但 SQL Database 会使用所有可用内存来运行数据库操作。出于此原因，上述查询未在平均 DTU 消耗中包括内存消耗。另一方面，如果你要降级到更低的性能级别，则数据库的可用内存将会减少。这可能会导致 IO 消耗提高，从而影响消耗的 DTU 数量。因此，在降级到更低的性能级别时，请确保 IO 百分比中有足够的余隙。可以使用上述 [sys.dm_db_resource_stats](http://msdn.microsoft.com/zh-cn/library/azure/dn800981.aspx) DMV 来监视这种情况。



- **警报：**设置 '当升级后的数据库 DTU 消耗量接近特定的高位时，Azure 管理门户中的警报'会向你发出通知。你可以针对 DTU、CPU、IO 和日志等各种性能度量值，在 Azure 管理门户中设置数据库警报。 

	例如，你可以针对"DTU 百分比"设置电子邮件警报，以便在过去 5 分钟平均 DTU 百分比值超过 75% 时发出警报。请参阅[如何：在 Azure 中接收警报通知和管理警报规则](http://msdn.microsoft.com/zh-cn/library/azure/dn306638.aspx)，以了解有关如何配置警报通知的详细信息。


- **计划的性能级别升级/降级：**如果应用程序的某些方案只是在一天/一周的特定时间才需要更高的性能，则你可以使用 [Azure Automation](http://msdn.microsoft.com/zh-cn/library/azure/dn643629.aspx)，以计划中操作的形式将数据库升级/降级到更高/更低的性能级别。

	例如，在执行每周批处理/维护作业期间，将数据库升级到更高的性能级别，在完成作业后降级数据库。这种类型的计划也适用于任何大规模的资源密集型操作，例如数据加载、重建索引，等等。请注意，Azure SQL Database 计费模式基于服务层/性能级别的每小时用量。凭借这种灵活性，你可以更加经济高效地安排有计划的升级。

## 摘要
Azure SQL Database 服务提供了遥测数据和工具来评估 Web/企业数据库工作负载，并确定适合升级的最佳服务层。升级过程非常简单，无需使数据库脱机就能完成，且不会丢失数据。升级后的数据库将受益于可预测的性能，以及新服务层提供的更多功能。

<!--HONumber=55-->