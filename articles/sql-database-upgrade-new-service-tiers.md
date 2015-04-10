<properties title="Upgrade SQL Database Web/Business Databases to New Service Tiers" pageTitle="将 SQL Database Web/企业版数据库升级到新服务层" description="将 Azure SQL Database Web 版或企业版数据库升级到新的 Azure SQL Database 服务层/性能级别。" metaKeywords="Azure, SQL. DB, Service, Tier, web, business" services="Azure SQL DB" solutions="upgrade, migrate, move, monitor, t-sql, powershell" documentationCenter="" authors="Srini Acharya, Umachandar Jayachandran" manager="jeffreyg" videoId="" scriptId="" />

<tags ms.service="sql-database" ms.date="12/3/2014" ms.author="jhubbard" />


#将 SQL Database Web/企业版数据库升级到新服务层

<p> Azure Web 版和企业版 SQL Database 在共享的、多租户的环境中运行，而没有为数据库预留任何资源容量。您群集中其他数据库的活动可能会影响性能。在任意给定时间的资源可用性很大程度取决于在系统中运行的其他并发工作负载。这可能导致数据库应用程序的性能截然不同且不可预测。客户反馈此不可预测的性能难以管理，他们希望性能更易于预测。 

为了解决此反馈，Azure SQL Database 服务引入了新的数据库服务层[（基本、标准和高级）](http://azure.microsoft.com/blog/2014/08/26/new-azure-sql-database-service-tiers-generally-available-in-september-with-reduced-pricing-and-enhanced-sla/ )，这些服务层可针对业务连续性和安全性提供可预测的性能和丰富的新功能。这些新服务层旨在为数据库工作负载提供指定级别的资源，而不考虑在该环境中运行的其他客户工作负载。这将产生高度可预测的性能行为。在此版本发行期间，Web 版和企业版服务层将于 2015 年 9 月停用。 

这些更改也产生了一些问题：如何评估和确定哪个新服务层最适合其当前的 Web 版和企业版 (W/B) 数据库，以及有关实际升级流程本身的问题。本文提供了将 Web/企业版数据库升级到新的服务层/性能级别的指导方法。

Web/企业版数据库到新服务层的升级涉及以下步骤：

1.	确定向您提供您的应用程序或业务要求所需的最低级别的功能可用性的服务层
2.	根据历史资源使用情况确定性能级别
3.	升级到所选服务层/性能级别
4.	监视升级后的数据库，并根据需要调整服务层/性能级别

## 1.根据功能可用性确定服务层

确定向您提供您的应用程序或业务要求所需的最低级别的功能可用性的服务层。例如，请考虑您的如下要求：备份保留多长时间、是否需要标准或活动的地域复制功能、数据库整体大小是否需要调整为最大等。这些要求会影响您的最低服务层选择（请参考[服务层公告博客）。 ](http://azure.microsoft.com/blog/2014/08/26/new-azure-sql-database-service-tiers-generally-available-in-september-with-reduced-pricing-and-enhanced-sla/ ) 

"基本"层主要用于非常小、活动性低的数据库。因此，对于升级，在选择服务层时，通常应根据您的功能从"标准"或"高级"层开始。有关针对并发请求/会话的特定于服务层/性能级别的限制，请参考 [Azure 文档](http://msdn.microsoft.com/zh-cn/library/azure/dn741336.aspx)。

##2.根据资源使用情况确定性能级别 
Azure SQL Database 服务在 sys.resource_stats 视图中提供了 Web/企业版数据库的资源消耗量[（请参阅 MSDN 文档）](http://msdn.microsoft.com/library/azure/dn269979.aspx)，该视图位于您的当前数据库所在的逻辑服务器的主数据库中。它采用占性能级别上限的百分比来显示资源消耗量数据。此视图提供长达最近 14 天的数据，时间间隔为 5 分钟。由于 Web/企业版数据库不具有与之关联的任何有保证的 DTU/资源限制，我们将根据 S2 性能级别中的数据库可使用的资源量来规范化百分比值。在任一特定时间间隔内，某个数据库的 DTU 平均消耗量百分比可以按照以下方式计算：在该特定时间间隔内 CPU、IO 和日志使用率中的最高百分比值。若要检索某个数据库的 DTU 平均消耗量，请在主数据库上运行以下查询：

 
                   
     SELECT start_time, end_time
	 ,(SELECT Max(v)
         FROM (VALUES (avg_cpu_percent)
                    , (avg_physical_data_read_percent)
                    , (avg_log_write_percent)
    	   ) AS value(v)) AS [avg_DTU_percent]
    FROM sys.resource_stats
    WHERE database_name = '<your db name>'
    ORDER BY end_time DESC;

基于 S2 数据库级别的 DTU 消耗量信息使您可以根据新层数据库规范化 Web/企业版数据库的当前消耗量并查看更适合它们的层。例如，如果您的 DTU 平均消耗量百分比值显示为 80%，则它指示数据库占用了 S2 性能级别的数据库上限 80% 比率的 DTU。如果您在 sys.resource_stats 视图中看到大于 100% 的值，这意味着您需要高于 S2 的性能层。例如，假设您看到峰值 DTU 百分比的值为 300%。这表示您已使用的资源是 S2 中可用资源的三倍。若要确定合理的起始大小，请将 S2 中可用的 DTU (50 DTU) 与下一更高层的大小（P1 = 100 DTU 或 S2 的 200%，P2 = 200 DTU 或 S2 的 400%）相比较。因为您处于 S2 的 300%，所以建议您从 P2 开始重新测试。

根据 DTU 使用率百分比和为满足您的工作负载所需的最大版本，您可以确定哪个服务层和性能级别最适合您的数据库工作负载（如各个[性能级别](http://msdn.microsoft.com/library/azure/dn741336.aspx)的 DTU 百分比和相对 DTU 幂所示）。下表提供了 Web/企业版资源消耗量百分比到与之等效的新层性能级别的对应关系： 

![resource consumption](http://i.imgur.com/nPodQZH.png)

> **注意**：在各个性能级别之间的相对 DTU 数来源于 [Azure SQL Database 基准](http://msdn.microsoft.com/library/azure/dn741327.aspx)工作负载。由于您的数据库工作负载可能与该基准不同，您应将上面的计算结果用作衡量新层中 Web/企业版数据库的初始适合级别的指导原则。将数据库移动到新层后，请使用上一节所述过程来验证/微调可满足您的工作负载需要的适当的服务层。
> 
> **注意**：虽然建议的新版本层/性能级别考虑了最近 14 天内您的数据库活动，但是此数据来源于 5 分钟内的平均资源消耗量数据示例。正因如此，它可能漏掉持续时间少于 5 分钟的短期内突发的活动。因此，应使用本指南作为升级您的数据库的起点。将数据库升级到建议的层后，将需要进行更多的监视、测试和验证，并且可以根据需要将该数据库向上/向下移动到不同的层/性能级别。

每个 5 分钟的数据采样间隔期间，以下查询会在主数据库上针对您的 Web/企业版层数据库执行计算，并表明哪个版本更适合您的工作负载。

> **注意：**此查询仅可用于 Web/企业版数据库，并且**不**会提供新层中的数据库的正确结果。

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

在下图中，您可以看到 DTU 平均消耗量百分比随时间推移的趋势。下面是大多数时间处于 S2 级别中的数据库的示例图，其中一些峰值活动飙升到了 P1 数据库级别。随着时间的推移，DTU 消耗量会在"Basic"上限和"P1"上限之间变化。若要完全适合新层中的此数据库，您将需要性能级别为"P1"的高级服务层数据库。另一方面，如果这些偶然突增到 P1 级别的活动很少出现，则 S2 级别的数据库也可以起作用。

![DTU Usage](http://i.imgur.com/e4N4ay5.png)

##3.升级到新的服务层/性能级别
确定 Web/企业版数据库的服务层/性能级别之后，可以使用多种方法来将该数据库升级到新层：


- Azure 管理门户
- Azure PowerShell 
- [服务管理 REST API](http://msdn.microsoft.com/zh-cn/library/azure/dn505719.aspx)
- Transact-SQL 

对于 Web/企业版数据库的临时迁移，请使用 [Azure 管理门户](https://manage.windowsazure.com/)。有关使用 Azure 管理门户进行升级的步骤，请参阅文章[更改数据库服务层和性能级别](http://msdn.microsoft.com/zh-cn/library/azure/dn369872.aspx)中的"升级到更高的服务层"部分。

若要在服务器/订阅中的多个数据库上执行该操作或使此过程自动执行，请使用 PowerShell 或 Transact-SQL 命令。

##使用 Azure PowerShell 进行升级
您可以通过端到端示例，使用下面的 Azure PowerShell 升级 Web/企业版数据库。升级数据库的步骤如下所示：

1. 使用 **New-AzureSqlDatabaseServerContext** cmdlet 设置服务器上下文 
2. 使用 **Get-AzureSqlDatabase** cmdlet 获取要升级的数据库
3. 使用**Set-AzureSqlDatabase -ServiceObjective** 基于名称（示例：S1、S2、P1）获取所需的性能级别
4. 使用 **Set-AzureSqlDatabase** 为数据库指定新服务层和性能级别

下面显示了一个将服务器和数据库升级到所需的服务层/性能级别的示例脚本：

    # Get the credential for the server. Provide login that is in dbmanager role of server
    $credential = Get-Credential
    # Provide server name in the cmdlet below:
    $serverContext = New-AzureSqlDatabaseServerContext -ServerName "<server_name>" -Credential $credential
    # Provide database name in the cmdlet below:
    $db = Get-AzureSqlDatabase $serverContext -DatabaseName "<database_name>"
    $service_tier = "<desired_service_tier>"
    $performance_level = Get-AzureSqlDatabaseServiceObjective $serverContext -ServiceObjectiveName "<desired_performance_level>"
    Set-AzureSqlDatabase $serverContext -Database $db -ServiceObjective $performance_level -Edition $service_tier


## 使用 Transact-SQL 进行升级

若要使用 Transact-SQL 升级现有 Web/企业版数据库，您可以使用 
[ALTER DATABASE 语句](http://msdn.microsoft.com/library/azure/ms174269.aspx)。用于将数据库升级到新服务层/性能级别的 ALTER DATABASE 命令，如下所示：

    ALTER DATABASE <db_name> MODIFY (EDITION = '<service_tier>', SERVICE_OBJECTIVE = '<performance_level>');
      	 

##4.	监视新服务层/性能级别的升级
Azure SQL Database 在您当前数据库所在的逻辑服务器的主数据库的 sys.dm_operation_status 动态管理视图中提供了在数据库上执行的管理操作（例如创建、更改、删除）的进度信息。[请参阅 sys.dm_operation_status 文档](http://msdn.microsoft.com/library/azure/dn270022.aspx)。使用操作状态 DMV 来确定数据库升级操作的进度。此示例查询显示了在某个数据库上执行的所有管理操作：

    SELECT o.operation, o.state_desc, o.percent_complete
    , o.error_code, o.error_desc, o.error_severity, o.error_state
    , o.start_time, o.last_modify_time
    FROM sys.dm_operation_status AS o
    WHERE o.resource_type_desc = 'DATABASE'
    and o.major_resource_id = '<database_name>'
    ORDER BY o.last_modify_time DESC;

如果您使用了管理门户进行升级，则还可以从该门户中获取关于操作的通知。

## 升级的影响
将 Web/企业版数据库升级到新的服务层/性能级别是一个联机操作。在整个升级操作过程中，您的数据库依旧会工作。在实际转换为新的性能级别时，在非常短的持续时间内（以秒为单位）可能会临时删除到该数据库的连接。如果应用程序已针对连接终止进行临时故障处理，则足以防止在升级结束时断开连接。有关处理连接终止的最佳做法，请参考[防止拒绝请求或终止连接的 Azure SQL Database 最佳做法。](http://msdn.microsoft.com/library/azure/dn338082.aspx)

有关如何估计升级操作的延迟的详细信息，请参阅[更改数据库服务层和性能级别](http://msdn.microsoft.com/library/azure/dn369872.aspx)中的"了解和估算数据库更改的延迟"部分。

##升级后，数据库工作负载达到资源上限时会发生什么情况？
将校准并控制性能级别，以提供运行数据库工作负载所需的资源，直到达到您所选的服务层/性能级别所允许的最大限制（即资源消耗量处于 100%）。如果您的工作负载达到任一 CPU/数据 IO/日志 IO 的上限，您仍然可以接收到所允许的最大级别的资源，但您很可能会看到查询的延迟会有所增加。达到这些限制中的任一上限不会导致任何错误，只不过会导致您的工作负载减速，除非减速过于严重以致查询开始超时。如果已经达到所允许的并发用户会话/请求（工作线程）数的上限，则在进行任何进一步的连接/请求时您将会看到明显的错误（请参考 MSDN 文档）。

##监视升级后的数据库
将 Web/企业版数据库升级到新层中之后，建议主动监视该数据库，以确保应用程序以预期的性能运行并根据需要优化使用情况。建议使用以下附加步骤监视数据库。


**资源消耗量数据：**对于"基本"、"标准"和"高级"数据库，通过名为 [sys.dm_db_resource_stats](http://msdn.microsoft.com/library/azure/dn800981.aspx) 的新 DMV 提供了更精确的用户数据库中的资源消耗量数据。此 DMV 提供操作前一个小时内接近实时的资源消耗量信息，以 15 秒为间隔。一个时间间隔内的 DTU 百分比消耗量按照以下方式计算：CPU、IO 和日志大小的最大百分比消耗量。以下是用于计算最后一个小时内的 DTU 平均百分比消耗量的查询：

    SELECT end_time
    	 , (SELECT Max(v)
             FROM (VALUES (avg_cpu_percent)
                         , (avg_data_io_percent)
                         , (avg_log_write_percent)
    	   ) AS value(v)) AS [avg_DTU_percent]
    FROM sys.dm_db_resource_stats
    ORDER BY end_time DESC;

 
附加[文档](http://msdn.microsoft.com/zh-cn/library/dn800981.aspx)包含如何使用此 DMV 的详细信息。[Azure SQL Database 性能指南](http://msdn.microsoft.com/zh-cn/library/azure/dn369873.aspx)介绍了如何监视和优化您的应用程序。

**对性能的内存影响：**虽然内存是影响 DTU 分级的资源维度之一，但是 SQL Database 旨在用于将所有可用内存都用于数据库操作。由于这个原因，内存消耗量未纳入上面查询中的 DTU 平均消耗量中。另一方面，如果您缩减到更低的性能级别，则数据库的可用内存将减少。这可能导致更高的 IO 消耗影响已使用的 DTU。因此，当缩减到更低性能级别时，请确保您在 IO 百分比方面具有足够的空余空间。使用上述 [sys.dm_ db_ resource_stats](http://msdn.microsoft.com/library/azure/dn800981.aspx) DMV 对此进行监视。



- **警报：**在 Azure 管理门户中设置警报，以在已升级数据库的 DTU 消耗量达到特定较高级别时通知您。可以在 Azure 管理门户中针对不同的性能指标（如 DTU、CPU、IO 和日志）设置数据库警报。

	例如，在过去 5 分钟内 DTU 平均百分比值超过 75% 时，您可以设置有关"DTU 百分比"的电子邮件警报。请参考[如何：在 Azure 中接收警报通知和管理警报规则](http://msdn.microsoft.com/zh-cn/library/azure/dn306638.aspx)，以了解有关如何配置警报通知的详细信息。


- **计划的性能级别升级/降级：**如果您的应用程序具有特定的应用场景，并且只在每天/每周中的某些时间需要更高的性能，则可以将 [Azure Automation](http://msdn.microsoft.com/library/azure/dn643629.aspx) 用作计划操作，将您的数据库扩充/缩减到更高/更低的性能级别。

	例如，在每周的批处理/维护作业的持续时间内将数据库升级到更高的性能级别，然后在作业完成后缩减该数据库。这种类型的计划也可用于任何大规模的占用大量资源的操作，如数据加载、索引重新生成等。请注意：Azure SQL Database 按服务层/性能级别每小时的使用情况计费。这种灵活性使您能够以更高的成本效率规划计划的（或规划的）升级。

##摘要
Azure SQL Database 服务提供了遥测数据和工具，用于评估您的 Web/企业版数据库工作负载并确定满足升级要求的最佳服务层。升级过程非常简单，无需使数据库处于脱机状态即可完成，并且不会丢失任何数据。升级后的数据库可从新服务层提供的可预测性能和附加功能中受益。
