<properties title="数据依赖路由" pageTitle="分片灵活性" description="介绍了分片灵活性（可轻松向外扩展 Azure SQL Database 的能力）的概念并提供了相关示例。" metaKeywords="sharding scaling, Azure SQL DB sharding, elastic scale, elasticity" services="sql-database" documentationCenter=""  manager="jhubbard" authors="sidneyh@microsoft.com"/>

<tags ms.service="sql-database" ms.workload="sql-database" ms.tgt_pltfrm="na" ms.devlang="na" ms.topic="article" ms.date="10/02/2014" ms.author="sidneyh"></tags>

# 分片灵活性

**分片灵活性**允许应用程序开发人员根据需求动态增加和缩减数据库资源，使开发人员能够优化其应用程序的性能并最大程度地降低成本。Azure SQL Database 的灵活扩展与[基本、标准和高级服务层][基本、标准和高级服务层]的组合提供了非常有说服力的灵活性方案。灵活扩展支持**水平缩放** - 一种设计模式，在该模式中，通过从**分片集**添加或删除数据库（在[灵活扩展术语][灵活扩展术语]中称为“分片”）来增加或缩减容量。类似地，SQL Database 服务层提供了**垂直缩放**功能，其中单个数据库的资源可以向上或向下扩展以相应地满足需求。单个分片的垂直缩放和多个分片的水平缩放共同向应用程序开发人员提供了非常灵活的环境，可对该环境进行缩放以满足性能、容量和成本优化需求。

### 水平缩放示例：Concert Spike

一个水平缩放的典型方案是处理演唱会门票交易的应用程序。在正常的客户数量下，该应用程序使用最少的数据库资源来处理购买交易。但是，在销售一场热门演唱会的门票时，单个数据库便无法处理急剧上升的客户需求。

为了处理急剧增加的交易，该应用程序将进行水平缩放。该应用程序现在可以在多个分片上分发交易负载。当不再需要额外的资源时，数据库层将缩减回正常使用量。在这里，水平缩放允许应用程序向外扩展以满足客户需求，并在不再需要资源时向内扩展。

### 垂直缩放示例：遥测

一个垂直缩放的典型方案是使用**分片集**存储操作遥测的应用程序。在此方案中，最好将一天的所有遥测数据一起放置在单个分片上。在此应用程序中，将当天的数据引入到一个分片，并为以后的日子设置一个新的分片。然后，可根据相应情况老化和查询操作数据。

为了在高负载时引入遥测数据，最好使用较高的服务层。换言之，高级数据库优于基本数据库。数据库达到其容量后，它将从引入切换为分析和报告。为此，标准层的性能将等同于该任务。这样，开发人员可以在分片（除了最新创建的分片）上向下扩展服务层，以符合旧数据的较低性能要求。

为了获得提升的性能，还可以将垂直缩放用于提高单个数据库的性能。例如，纳税申报应用程序。在进行申报时，最好将单个客户的数据全部保留在同一个数据库中并提高该分片的性能。根据不同的应用程序，垂直向上和向下扩展资源有利于优化成本和性能要求。

数据库层的水平和垂直缩放都是灵活扩展应用程序和分片灵活性的核心。

本文档提供了分片灵活性方案的背景知识，以及垂直和水平缩放的示例实现的参考。

## 分片灵活性的机制

垂直和水平缩放是三个基本组件的功能：

1.  **遥测**
2.  **规则**
3.  **操作**

## <a name="telemetry"> </a>遥测

**数据驱动的灵活性**是灵活扩展应用程序的核心。根据性能要求，使用遥测来作出是进行垂直缩放还是水平缩放的数据驱动的决策。

#### 遥测数据源

在 Azure SQL DB 的上下文中，有少数关键源可用作分片灵活性的数据源。

1.  **性能遥测**每隔五分钟在 **sys.resource\_stats** 视图中显示一次
2.  每小时**数据库容量遥测**通过 **sys.resource\_usage** 视图显示。

开发人员可以分析性能资源使用情况，方法是使用以下查询来查询主数据库，其中“Shard\_20140623”是目标数据库的名称。

    SELECT TOP 10 *  
    FROM sys.resource_stats  
    WHERE database_name = 'Shard_20140623'  
    ORDER BY start_time DESC 

可汇总一段时间内的**性能遥测**（在以下查询中为七天）以删除暂时性行为。

    SELECT  
    avg(avg_cpu_percent) AS 'Average CPU Utilization In Percent', 
        max(avg_cpu_percent) AS 'Maximum CPU Utilization In Percent', 
        avg(avg_physical_data_read_percent) AS 'Average Physical Data Read Utilization In Percent', 
        max(avg_physical_data_read_percent) AS 'Maximum Physical Data Read Utilization In Percent', 
        avg(avg_log_write_percent) AS 'Average Log Write Utilization In Percent', 
        max(avg_log_write_percent) AS 'Maximum Log Write Utilization In Percent', 
        avg(active_session_count) AS 'Average # of Sessions', 
        max(active_session_count) AS 'Maximum # of Sessions', 
        avg(active_worker_count) AS 'Average # of Workers', 
        max(active_worker_count) AS 'Maximum # of Workers' 
    FROM sys.resource_stats  
    WHERE database_name = ' Shard_20140623' AND start_time > DATEADD(day, -7, GETDATE()); 

可使用针对 **sys.resource\_usage** 视图的类似查询测量**数据库容量**。**storage\_in\_megabytes** 列的最大值将生成数据库的当前大小。对于在特定的分片达到其容量时水平缩放应用程序，这样的遥测非常有用。

    SELECT TOP 10 * 
    FROM [sys].[resource_usage] 
    WHERE database_name = 'Shard_20140623'  
    ORDER BY time DESC 

由于数据已引入特定的分片，提前一天进行计划并确定该分片是否有足够的容量处理未来的工作负载是有用的。虽然不是线性回归的真正实现，但是以下查询将返回连续两天内数据库容量中的最大增量。然后可将这样的遥测应用于规则，之后该规则将导致执行某项操作（或不执行操作）。

    WITH MaxDatabaseDailySize AS( 
        SELECT 
            ROW_NUMBER() OVER (ORDER BY convert(date, [time]) DESC) as [Order], 
            CONVERT(date,[time]) as [date],  
            MAX(storage_in_megabytes) as [MaxSizeDay] 
        FROM [sys].[resource_usage] 
        WHERE  
            database_name = 'Shard_20140623' 
        GROUP BY CONVERT(date,[time]) 
        ) 

    SELECT 
        MAX(ISNULL(Size.[MaxSizeDay] - PreviousDaySize.[MaxSizeDay], 0)) 
    FROM  
        MaxDatabaseDailySize Size INNER JOIN 
        MaxDatabaseDailySize PreviousDaySize ON Size.[order]+1 = PreviousDaySize.[order] 
    WHERE 
        Size.[order] < 8 

## <a name="rule"></a>规则

规则是确定是否执行一项操作的决策引擎。有些规则非常简单，而有些则非常复杂。如下方代码段所示，可以配置基于容量的规则，以便在分片达到其最大容量的 $SafetyMargin（例如，80%）时设置新的分片。

    # Determine if the current DB size plus the maximum daily delta size is greater than the threshold 
    if( ($CurrentDbSizeMb + $MaxDbDeltaMb) -gt ($MaxDbSizeMb * $SafetyMargin))  
    {#provision new shard} 

通过上面提供的数据源，可以制定许多规则以完成大量的分片灵活性方案。

## <a name="action"></a>操作

应用规则的结果将导致执行操作（或不执行操作）。两种最常见的操作是：

-   增加或减少分片的服务层或性能级别
-   从分片集添加或删除分片。

请注意，在水平和垂直缩放解决方案中，操作的结果不是即时的。例如，在进行垂直缩放时，发布将基本数据库的服务层提高到高级数据库的 ALTER DATABASE 命令所花费的时间是不确定的。该持续时间将在很大程度上取决于数据库的大小（有关详细信息，请参阅[更改服务层和性能级别][更改服务层和性能级别]。同样，新分片的分配或设置也不是即时的。因此，对于垂直和水平缩放，应用程序应考虑更改或设置新数据库所花的非零时间。

## 示例分片灵活性方案

下图所示的示例重点介绍两种灵活扩展方案：

1.  分片映射的水平缩放
2.  单个分片的垂直缩放。

![操作数据引入][操作数据引入]

为了进行水平缩放，将使用规则（基于日期或数据库大小）来设置新分片并使用分片映射对它进行注册，从而水平增加数据库层。其次，为了进行垂直缩放，将实施第二个规则，在该规则中，任何超过一天的分片都将从高级版降级到标准或基本版。

请再次考虑遥测方案：按日期排序的应用程序分片。它持续收集遥测数据，在加载时要求高性能版，但在数据老化时则要求较低的性能版。当天的数据 [Tnow] 将写入高性能数据库（高级）。到零点时，前一天的分片（现在为 [T-1]）将不再用于引入。当前数据由当前 [Tnow] 引入。在第二天之前，必须使用分片映射 ([T+1]) 设置并注册新分片。

这可通过总是在第二天之前设置新分片或在当前分片 ([Tnow]) 接近其最大容量时设置新分片来完成。使用这些方法中的任何一个都将保留某一天写入的所有遥测的数据局部性。还可以应用每小时分片，以获得更精细的粒度。设置新分片后，因为 [T-1] 用于查询和报告，所以最好减少数据库的性能级别以降低成本。当数据库中的内容老化时，可进一步减少性能层，并且/或者可将数据库的内容存档到 Azure 存储或将其删除，具体取决于应用程序。

## 执行分片灵活性方案

为了促进水平和垂直方案的实际实现，已在脚本中心上创建并发布了许多[分片灵活性示例脚本][分片灵活性示例脚本]。编写这些 PowerShell Runbook 旨在使其在 Azure Automation 服务中运行，它们提供了许多与灵活扩展客户端库和 Azure SQL Database 交互的方法。通过基于这些代码示例进行构建，或从其中提取部分代码，开发人员可以生成必要的脚本，以使其应用程序的水平缩放、垂直缩放或这两种方案实现自动化。

[AZURE.INCLUDE [elastic-scale-include](../includes/elastic-scale-include.md)]

<!--Image references--> 
<!--anchors-->

  [基本、标准和高级服务层]: http://msdn.microsoft.com/en-us/library/azure/dn741340.aspx
  [灵活扩展术语]: sql-database-elastic-scale-glossary.md
  [更改服务层和性能级别]: http://msdn.microsoft.com/library/azure/dn369872.aspx
  [操作数据引入]: ./media/sql-database-elastic-scale-elasticity/data-ingestion.png
  [分片灵活性示例脚本]: http://go.microsoft.com/?linkid=9862617
