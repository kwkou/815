<properties 
	pageTitle="Azure SQL 数据库弹性数据库池的价格和性能注意事项" 
	description="弹性数据库池是由一组弹性数据库共享的可用资源的集合。本文提供相关的指导来帮助你评估是否适合对一组数据库使用弹性数据库池。" 
	services="sql-database" 
	documentationCenter="" 
	authors="stevestein" 
	manager="jeffreyg" 
	editor=""/>

<tags 
	ms.service="sql-database"
	ms.date="12/01/2015" 
	wacn.date="01/29/2016"/>


# 弹性数据库池的价格和性能注意事项


本文档提供相关的指导来帮助你根据数据库使用模式以及弹性数据库池与单一数据库之间的定价差异，评估对一组数据库使用弹性数据库池是否能够产生成本效益。此外，还提供了其他指导来帮助你确定一组现有 SQL 数据库所需的当前池大小。

- 有关弹性数据库池的概述，请参阅 [SQL 数据库弹性数据库池](/documentation/articles/sql-database-elastic-pool)。
- 有关弹性数据库池的详细信息，请参阅 [SQL 数据库弹性数据库池参考](/documentation/articles/sql-database-elastic-pool-reference)。


> [AZURE.NOTE] 弹性数据库池目前为预览版，仅适用于 SQL 数据库 V12 服务器。

## 弹性数据库池

SaaS ISV 开发构建在由多个数据库组成的大规模数据层上的应用程序。常见的应用程序模式是每个数据库有不同的客户，但有独特的且无法预期的使用模式。因此让 ISV 难以个别预测每个数据库的资源要求。在这些情况下，ISV 可能以可观的费用过度设置资源，以确保所有数据库能有最佳吞吐量和响应时间。或者，ISV 可能用较少的费用，让客户承担体验性能不佳的风险。

Azure SQL 数据库中的弹性数据库池可让 SaaS ISV 将一组数据库的价格性能优化在规定的预算内，同时为每个数据库提供性能弹性。弹性数据库池可让 ISV 为由多个数据库共享的池购买弹性数据库吞吐量单位 (eDTU)，以适应单一数据库使用时段不可预测的情况。池的 eDTU 要求取决于其数据库的聚合使用量。池可用的 eDTU 数量由 ISV 预算控制。弹性数据库池可让 ISV 轻松为其池推断预算对性能的影响，反之亦然。ISV 只要将数据库加入到弹性数据库池、设置数据库需要任何 eDTU 保证或容量，并根据其预算设置池的 eDTU。通过使用弹性数据库池，ISV 可以顺畅地扩大其服务，以渐增的规模从精简的新创公司到成熟的企业。
  


## 何时考虑使用弹性数据库池

弹性数据库池很适合具有特定使用模式的大量数据库。对于给定的数据库，此模式的特征是低平均使用量与相对不频繁的使用高峰。

可以加入池的数据库越多，实现的节省就越大。但根据应用程序使用模式，你可能会看到与使用 2 个 S3 数据库一样少的节约。

以下各部分将帮助你了解如何评估特定的数据库集合是否会因使用弹性数据库池而受益。这些示例使用标准弹性数据库池，但同样的原理也适用于基本和高级池。

### 评估数据库使用模式

下图显示了一个数据库示例，该数据库有大量的闲置时间，但也会定期出现活动高峰。这是非常适合弹性数据库池的使用模式：
 
   ![1 个数据库][1]

对于如上所示的一小时期间，DB1 高峰最高达到 90 个 DTU，但其整体平均使用量低于 5 个 DTU。需要购买 S3 性能级别才能在单一数据库中运行此工作负荷。不过，这会在低活动时段使大多数资源保持为未使用状态。

弹性数据库池可让这些未使用的 DTU 跨多个数据库共享，因此减少了所需的 DTU 总量和总体成本。

以上一个示例为基础，假设有其他数据库具有与 DB1 类似的使用模式。在接下来的两个图形中，4 个数据库和 20 个数据库的使用量分层放在相同的图形，以说明经过一段时间其使用量非重叠的性质：

   ![4 个数据库][2]

   ![20 个数据库][3]

在上图中，黑线表示跨所有 20 个数据库的聚合 DTU 使用量。其中表明聚合 DTU 使用量永远不会超过 100 个 DTU，并指出 20 个数据库可以在这段期间共享 100 个 eDTU。相比于将每个数据库放在单一数据库的 S3 性能级别，这会导致 DTU 减少 20 倍和价格降低 13 倍。


由于以下原因，此示例很理想：

- 每一数据库之间的高峰使用量和平均使用量有相当大的差异。  
- 每个数据库的高峰使用量在不同时间点发生。 
- eDTU 在大量数据库之间共享。


弹性数据库池的价格由池 eDTU 决定。尽管池的 eDTU 单位价格比单一数据库的 DTU 的单位价格多 1.5 倍，但**池 eDTU 可由多个数据库共享，因而在许多情况下所需的 eDTU 总数更少**。定价方面和 eDTU 共享的这些差异是池可以提供成本节省可能性的基础。

<br>

以下数据库计数和数据库使用率相关规则的经验法则可帮助确保弹性数据库池提供相比于使用单一数据库的性能级别降低的成本。


### 数据库的最小数目

如果单一数据库的性能级别的 DTU 总和比池所需的 eDTU 多 1.5 倍，则弹性池更具成本效益。有关可用的大小，请参阅[弹性数据库池和弹性数据库的 eDTU 和存储限制](/documentation/articles/sql-database-elastic-pool-reference/#edtu-and-storage-limits-for-elastic-pools-and-elastic-databases)。


***示例***<br>
100 个 eDTU 弹性数据库池需要至少 2 个 S3 数据库或至少 15 个 S0 数据库，才能比使用单一数据库的性能级别更具成本效益。



### 并发高峰数据库的最大数目

通过共享 eDTU，并非池中的所有数据库都能同时使用 eDTU 达到使用单一数据库的性能级别时的最大限制。并发高峰的数据库越少，可以设置的池 eDTU 就越低，也就能实现越大的成本效益。一般而言，池中不能有 2/3（或 67%）以上的数据库的高峰同时达到其 eDTU 限制。

***示例***<br>
为了降低 200 eDTU 池中 3 个 S3 数据库的成本，在其使用量中最多可以有 2 个数据库同时处于高峰。否则，如果 4 个 S3 数据库中超过 2 个同时高峰，则必须将池缩放为超过 200 个 eDTU。而且，如果将池缩放大小为超过 200 个 eDTU，则需要加入更多的 S3 数据库到池，以使成本保持低于单一数据库的性能级别。


请注意，此示例未考虑池中其他数据库的使用率。如果在任何给定时间点，所有数据库都有一些使用量，则可以同时处于高峰的数据库应少于 2/3（或 67%）。


### 每个数据库的 DTU 使用率

数据库的高峰和平均使用率之间的差异为，长时间的低使用率和短时间的高使用率。这个使用模式非常适合在数据库之间共享资源。当数据库的高峰使用率比平均使用率大 1.5 倍左右时，应考虑将数据库用作池。

    
***示例***<br>
高峰为 100 个 DTU 且平均使用 67 个 DTU 或更少的 S3 数据库是在弹性数据库池中共享 eDTU 的良好候选项。或者，高峰为 20 个 DTU 且平均使用 13 个 DTU 或更少的 S1 数据库是弹性数据库池的良好候选项。
    

## 比较弹性数据库池与单一数据库的定价差异的启发式方法 

下面启发式方法可帮助你预估弹性数据库池是否比使用单个的单一数据库更具成本效益。

1. 通过以下项目来估算池所需的 eDTU：
    
    MAX(*数据库的总数目* * *每一数据库的平均 DTU 使用率*、*并发高峰数据库的数目* * *每一数据库的高峰 DTU 使用率*)

2. 选择大于步骤 1 估算值的池的最小可用 eDTU 值。有关可用的 eDTU 选项，请参阅此处所列的 eDTU 有效值：[弹性数据库池和弹性数据库的 eDTU 和存储限制](/documentation/articles/sql-database-elastic-pool-reference/#edtu-and-storage-limits-for-elastic-pools-and-elastic-databases)。


3. 按如下所述计算池的价格：

    池价格 = *池 eDTU 数* * *池 eDTU 单价*

    有关价格信息，请参阅 [SQL 数据库定价](/home/features/sql-database/#price)。


4. 将步骤 3 的池价格与单一数据库适当性能级别的价格相比较。



## 为现有 SQL 数据库确定最佳的池 eDTU 大小 

弹性数据库池的最佳大小取决于聚合 eDTU 和池中所有数据库所需的存储资源。这涉及到决定以下两个数量的较大值：

* 池中所有数据库使用的最大 DTU。
* 池中所有数据库使用的最大存储字节。 

有关可用的大小，请参阅[弹性数据库池和弹性数据库的 eDTU 和存储限制](/documentation/articles/sql-database-elastic-pool-reference/#edtu-and-storage-limits-for-elastic-pools-and-elastic-databases)。


### 使用服务层顾问 (STA) 和动态管理视图 (DMV) 获得选型建议   

STA 和 DMV 为调整弹性数据库池大小提供了不同的工具选项和功能。无论使用何种工具选项，大小估算值应该仅用于弹性数据库池的初步评估与创建。创建池后，应该准确地监视其资源使用情况，并根据需要调高和调低池的性能设置。

**STA**<br>STA 是 [Azure 门户](https://manage.windowsazure.cn)中的内置工具，它可以自动评估现有 SQL 数据库服务器中数据库的历史资源使用率，并推荐适当的弹性数据库池配置。有关详细信息，请参阅[弹性数据库池定价层建议](/documentation/articles/sql-database-elastic-pool-portal/#elastic-database-pool-pricing-tier-recommendations)。

**DMV 大小调整工具**<br>DMV 大小调整工具以 PowerShell 脚本的形式提供，可让你自定义服务器中现有数据库弹性数据库池的大小估算值。

### STA 和 DMV 工具之间的选择 

选择适用于分析特定应用程序的工具。下表汇总了这两个大小调整工具之间的差异：

| 功能 | STA | DMV |
| :--- | :--- | :--- |
| 分析中使用的数据示例粒度。 | 15 秒 | 15 秒 |
| 考虑单一数据库的池与性能级别之间的价格差异。 | 是 | 否 |
| 允许自定义服务器内所分析数据库的列表。 | 否 | 是 |
| 允许自定义分析中使用的时间段。 | 否 | 是 |


### 使用 STA 估算弹性池大小  

STA 评估数据库的使用率历史记录，并在比使用单一数据库的性能级别更具成本效益时，推荐你使用弹性数据库池。如果推荐使用某个池，该工具将提供推荐的数据库列表，以及推荐的池 eDTU 数量和每个弹性数据库的最小/最大 eDTU 设置。若要将某个数据库视为池的候选项，该数据库必须至少存在 7 天。

将弹性数据库池添加到现有服务器时，门户中会提供 STA。如果出现了有关该服务器弹性数据库池的建议，这些建议将显示在“弹性数据库池”创建页中。客户可以随时更改推荐的设置，以创建自己的弹性数据库池组。

有关详细信息，请参阅[弹性数据库池定价层建议](/documentation/articles/sql-database-elastic-pool-portal/#elastic-database-pool-pricing-tier-recommendations)

### 使用动态管理视图 (DMV) 估算弹性池大小 

[sys.dm\_db\_resource\_stats](https://msdn.microsoft.com/zh-cn/library/dn800981.aspx) DMV 可测量单个数据库的资源使用率。此 DMV 以数据库性能级别限制百分比的形式提供 CPU、IO、日志与数据库的日志使用率。此数据可用于计算任意给定 15 秒间隔内数据库的 DTU 使用率。

可以通过聚合 15 秒间隔内所有候选数据库的 eDTU 使用率，估算弹性数据库池在该时间段的聚合 eDTU 使用率。根据具体的性能目标，可能有必要忽略较小百分比的示例数据。例如，可以应用跨所有时间间隔的聚合 eDTU 第 99 个百分位值，以排除极端值并提供弹性数据库池 eDTU，以符合 99% 的采样时间间隔。

## 用于估算数据库聚合 eDTU 使用率的 PowerShell 脚本

这里提供了一个示例 PowerShell 脚本，用于估算 SQL 数据库服务器中用户数据库的聚合 eDTU 值。

该脚本只在运行时收集数据。对于典型的生产工作负荷，应该至少运行该脚本一天，但运行一周甚至更长时间可以提供更准确的估算值。针对代表数据库典型工作负荷的持续时间运行该脚本。

> [AZURE.IMPORTANT] 运行该脚本时，必须一直打开 PowerShell 窗口。除非你已运行该脚本足够的一段时间，并捕获了足够的数据来呈现跨越正常和高峰使用时间的典型工作负荷，否则请不要关闭 PowerShell 窗口。

### 脚本先决条件 

运行该脚本之前，请安装以下各项：

- 最新的 [PowerShell 命令行工具](http://go.microsoft.com/?linkid=9811175&clcid=0x409)。
- [SQL Server 2014 功能包](http://www.microsoft.com/zh-cn/download/details.aspx?id=42295)。


### 脚本详细信息


你可以从本机计算机或云上的 VM 运行该脚本。从本地计算机运行时，由于脚本需要从目标数据库下载数据，因此可能会产生数据输出费用。下面根据目标数据库数目和运行脚本的持续时间显示了数据卷估算值。有关 Azure 数据传输费用，请参阅[数据传输定价详细信息](/pricing/details/data-transfer)。
       
 -     每小时 1 个数据库 = 38KB
 -     每天 1 个数据库 = 900KB
 -     每周 1 个数据库 = 6MB
 -     每天 100 个数据库 = 90MB
 -     每周 500 个数据库 = 3GB

该脚本将排除不适合用作标准弹性池层的当前公共预览版产品的某些数据库。
如果你需要从目标服务器排除其他数据库，可以根据你的条件更改该脚本。默认情况下，脚本不会编译以下项的信息：

* 弹性数据库（数据库已在弹性池中）。
* 服务器的 master 数据库。

该脚本需要使用一个输出数据库来存储中间数据以供分析。你可以使用新的或现有的数据库。输出数据库应该在不同的服务器，以免影响分析结果，不过，从技术上讲，不需要满足该条件也能运行该工具。建议输出数据库的性能级别至少为 S0 或更高。长时间收集大量数据库的数据时，你可以考虑将输出数据库升级到较高的性能级别。

该脚本需要你提供用于连接目标服务器（弹性数据库池候选项）的凭据与完整服务器名称，例如“abcdef.database.chinacloudapi.cn”。该脚本目前不支持一次分析多个服务器。


提交初始参数集的值之后，系统会提示你使用 Azure 帐户登录。这是用于登录目标服务器而不是输出数据库服务器的帐户。
	
如果你在运行脚本时遇到以下警告，可将其忽略：

- 警告: Switch-AzureMode cmdlet 已过时。
- 警告: 无法获取 SQL Server 服务信息。尝试连接到 'Microsoft.Azure.Commands.Sql.dll' 上的 WMI 时失败并出现以下错误: RPC 服务器不可用。

脚本完成时，会输出要包含目标服务器中所有候选数据库，弹性池需要的 eDTU 估算数目。此估算的 eDTU 可用于创建和配置弹性数据库池，以容纳这些数据库。创建池并将数据库移到池中之后，应密切监控池数天，并根据需要对池 eDTU 配置进行任何调整。

> [AZURE.IMPORTANT] 此脚本包含到 Azure PowerShell 版本 1.0 *但不是包括* 1.0 及更高版本的命令。可以使用 **Get-Module azure | format-table version** 命令查看 Azure PowerShell 的版本。有关详细信息，请参阅[弃用 Azure PowerShell 中的 Switch-AzureMode](https://github.com/Azure/azure-powershell/wiki/Deprecation-of-Switch-AzureMode-in-Azure-PowerShell)。


    
    param (
    [Parameter(Mandatory=$true)][string]$AzureSubscriptionName, # Azure Subscription name - can be found on the Azure portal: https://manage.windowsazure.cn/
    [Parameter(Mandatory=$true)][string]$ResourceGroupName, # Resource Group name - can be found on the Azure portal: https://manage.windowsazure.cn/
    [Parameter(Mandatory=$true)][string]$servername, # full server name like "abcdefg.database.chinacloudapi.cn"
    [Parameter(Mandatory=$true)][string]$username, # user name
    [Parameter(Mandatory=$true)][string]$serverPassword, # password
    [Parameter(Mandatory=$true)][string]$outputServerName, # metrics collection database for analysis. full server name like "zyxwvu.database.chinacloudapi.cn"
    [Parameter(Mandatory=$true)][string]$outputdatabaseName,
    [Parameter(Mandatory=$true)][string]$outputDBUsername,
    [Parameter(Mandatory=$true)][string]$outputDBpassword,
    [Parameter(Mandatory=$true)][int]$duration_minutes # How long to run. Recommend to run for the period of time when your typical workload is running. At least 10 mins.
    )
    
    Add-AzureAccount -Environment AzureChinaCloud 
    Select-AzureSubscription $AzureSubscriptionName
    Switch-AzureMode AzureResourceManager
    
    $server = Get-AzureSqlServer -ServerName $servername.Split('.')[0] -ResourceGroupName $ResourceGroupName
    
    # Check version/upgrade status of the server
    $upgradestatus = Get-AzureSqlServerUpgrade -ServerName $servername.Split('.')[0] -ResourceGroupName $ResourceGroupName
    $version = ""
    if ([string]::IsNullOrWhiteSpace($server.ServerVersion)) 
    {
    $version = $upgradestatus.Status
    }
    else
    {
    $version = $server.ServerVersion
    }
    
    # For Elastic database pool candidates, we exclude master, and any databases that are already in a pool. You may add more databases to the excluded list below as needed
    $ListOfDBs = Get-AzureSqlDatabase -ServerName $servername.Split('.')[0] -ResourceGroupName $ResourceGroupName | Where-Object {$_.DatabaseName -notin ("master") -and $_.CurrentServiceLevelObjectiveName -notin ("ElasticPool") -and $_.CurrentServiceObjectiveName -notin ("ElasticPool")}
    
    $outputConnectionString = "Data Source=$outputServerName;Integrated Security=false;Initial Catalog=$outputdatabaseName;User Id=$outputDBUsername;Password=$outputDBpassword"
    $destinationTableName = "resource_stats_output"
    
    # Create a table in output database for metrics collection
    $sql = "
    IF  NOT EXISTS (SELECT * FROM sys.objects 
    WHERE object_id = OBJECT_ID(N'$($destinationTableName)') AND type in (N'U'))
    
    BEGIN
    Create Table $($destinationTableName) (database_name varchar(128), slo varchar(20), end_time datetime, avg_cpu float, avg_io float, avg_log float, db_size float);
    Create Clustered Index ci_endtime ON $($destinationTableName) (end_time);
    END
    TRUNCATE TABLE $($destinationTableName);
    "
    Invoke-Sqlcmd -ServerInstance $outputServerName -Database $outputdatabaseName -Username $outputDBUsername -Password $outputDBpassword -Query $sql -ConnectionTimeout 120 -QueryTimeout 120 
    
    # waittime (minutes) is interval between data collection queries in the loop below.
    $Waittime = 10
    $end_Time = [DateTime]::UtcNow
    $start_time = $end_time.AddMinutes(-$Waittime)
    $finish_time = $end_Time.AddMinutes($duration_minutes)
    
    While ($end_time -lt $finish_time)
    {
    Write-Host "Collecting metrics..." 
    foreach ($db in $ListOfDBs)
    {
    if ($version -in ("12.0", "Completed")) # for V12 databases 
    {
    $sql = "Declare @dbname varchar(128) = '$($db.DatabaseName)';"
    $sql += "Declare @SLO varchar(20) = '$($db.CurrentServiceLevelObjectiveName)';"
    $sql+= "
    Declare @DTU_cap int, @db_size float;
    Select @DTU_cap = CASE @SLO 
    WHEN 'Basic' THEN 5
    WHEN 'S0' THEN 10
    WHEN 'S1' THEN 20
    WHEN 'S2' THEN 50
    WHEN 'S3' THEN 100
    WHEN 'P1' THEN 125
    WHEN 'P2' THEN 250
    WHEN 'P3' THEN 1000
    ELSE 50 -- assume Web/Business DBs
    END
    SELECT @db_size = SUM(reserved_page_count) * 8.0/1024/1024 FROM sys.dm_db_partition_stats
    SELECT @dbname as database_name, @SLO, dateadd(second, round(datediff(second, '2015-01-01', end_time) / 15.0, 0) * 15,'2015-01-01')
    as end_time, avg_cpu_percent * (@DTU_cap/100.0) AS avg_cpu, avg_data_io_percent * (@DTU_cap/100.0) AS avg_io, avg_log_write_percent * (@DTU_cap/100.0) AS avg_log, @db_size as db_size FROM sys.dm_db_resource_stats
    WHERE end_time > '$($start_time)' and end_time <= '$($end_time)';
    " 
    }
    else
    {
    $sql = "Declare @dbname varchar(128) = '$($db.DatabaseName)';"
    $sql += "Declare @SLO varchar(20) = '$($db.CurrentServiceLevelObjectiveName)';"
    $sql+= "
    Declare @DTU_cap int, @db_size float;
    Select @DTU_cap = CASE @SLO 
    WHEN 'Basic' THEN 5
    WHEN 'S0' THEN 10
    WHEN 'S1' THEN 20
    WHEN 'S2' THEN 50
    WHEN 'P1' THEN 100
    WHEN 'P2' THEN 200
    WHEN 'P3' THEN 800
    ELSE 50 -- assume Web/Business DBs
    END
    SELECT @db_size = SUM(reserved_page_count) * 8.0/1024/1024 from sys.dm_db_partition_stats
    SELECT @dbname as database_name, @SLO, dateadd(second, round(datediff(second, '2015-01-01', end_time) / 15.0, 0) * 15,'2015-01-01')
    as end_time, avg_cpu_percent * (@DTU_cap/100.0) AS avg_cpu, avg_data_io_percent * (@DTU_cap/100.0) AS avg_io, avg_log_write_percent * (@DTU_cap/100.0) AS avg_log, @db_size as db_size FROM sys.dm_db_resource_stats
    WHERE end_time > '$($start_time)' and end_time <= '$($end_time)';
    " 
    }
    
    $result = Invoke-Sqlcmd -ServerInstance $servername -Database $db.DatabaseName -Username $username -Password $serverPassword -Query $sql -ConnectionTimeout 120 -QueryTimeout 3600 
    #bulk copy the metrics to output database
    $bulkCopy = new-object ("Data.SqlClient.SqlBulkCopy") $outputConnectionString 
    $bulkCopy.BulkCopyTimeout = 600
    $bulkCopy.DestinationTableName = "$destinationTableName";
    $bulkCopy.WriteToServer($result);
    
    }
    
    $start_time = $start_time.AddMinutes($Waittime)
    $end_time = $end_time.AddMinutes($Waittime)
    Write-Host $start_time
    Write-Host $end_time
    do {
    Start-Sleep 1
       }
    until (([DateTime]::UtcNow) -ge $end_time)
    }
    
    Write-Host "Analyzing the collected metrics...."
    # Analysis query that does aggregation of the resource metrics to calculate pool size.
    $sql1 = 'Declare @DTU_Perf_99 as float, @DTU_Storage as float;
    WITH group_stats AS
    (
    SELECT end_time, SUM(db_size) AS avg_group_Storage, SUM(avg_cpu) AS avg_group_cpu, SUM(avg_io) AS avg_group_io,SUM(avg_log) AS avg_group_log
    FROM resource_stats_output 
    WHERE slo LIKE '
    
    $sql2 = '
    GROUP BY end_time
    )
    -- calculate aggregate storage and DTUs for all DBs in the group
    , group_DTU AS
    (
    SELECT end_time, avg_group_Storage, 
    (SELECT Max(v)
       FROM (VALUES (avg_group_cpu), (avg_group_log), (avg_group_io)) AS value(v)) AS avg_group_DTU
    FROM group_stats
    )
    -- Get top 1 percent of the storage and DTU utilization samples.
    , top1_percent AS (
    SELECT TOP 1 PERCENT avg_group_Storage, avg_group_dtu FROM group_dtu ORDER BY [avg_group_DTU] DESC
    )
    
    -- Max and 99th percentile DTU for the given list of databases if converted into an elastic pool. Storage is increased by factor of 1.25 to accommodate for future growth. Currently storage limit of the pool is determined by the amount of DTUs based on 1GB/DTU.
    --SELECT MAX(avg_group_Storage)*1.25/1024.0 AS Group_Storage_DTU, MAX(avg_group_dtu) AS Group_Performance_DTU, MIN(avg_group_dtu) AS Group_Performance_DTU_99th_percentile FROM top1_percent;
    SELECT @DTU_Storage = MAX(avg_group_Storage)*1.25/1024.0, @DTU_Perf_99 = MIN(avg_group_dtu) FROM top1_percent;
    IF @DTU_Storage > @DTU_Perf_99 
    SELECT ''Total number of DTUs dominated by storage: '' + convert(varchar(100), @DTU_Storage)
    ELSE 
    SELECT ''Total number of DTUs dominated by resource consumption: '' + convert(varchar(100), @DTU_Perf_99)'
    
    #check if there are any web/biz edition dbs in the collected metrics
    $checkslo = "SELECT TOP 1 slo FROM resource_stats_output WHERE slo LIKE 'shared%'"
    $output = Invoke-Sqlcmd -ServerInstance $outputServerName -Database $outputdatabaseName -Username $outputDBUsername -Password $outputDBpassword -Query $checkslo -QueryTimeout 3600 | select -expand slo
    if ($output -like "Shared*")
    {
    write-host "`nWeb/Business edition:" -BackgroundColor Green -ForegroundColor Black
    $sql = $sql1 + "'Shared%'"  + $sql2
    $data = Invoke-Sqlcmd -ServerInstance $outputServerName -Database $outputdatabaseName -Username $outputDBUsername -Password $outputDBpassword -Query $sql -QueryTimeout 3600
    $data | %{'{0}' -f $_[0]}
    }
    
    #check if there are any basic edition dbs in the collected metrics
    $checkslo = "SELECT TOP 1 slo FROM resource_stats_output WHERE slo LIKE 'Basic%'"
    $output = Invoke-Sqlcmd -ServerInstance $outputServerName -Database $outputdatabaseName -Username $outputDBUsername -Password $outputDBpassword -Query $checkslo -QueryTimeout 3600 | select -expand slo
    if ($output -like "Basic*")
    {
    write-host "`nBasic edition:" -BackgroundColor Green -ForegroundColor Black
    $sql = $sql1 + "'Basic%'"  + $sql2
    $data = Invoke-Sqlcmd -ServerInstance $outputServerName -Database $outputdatabaseName -Username $outputDBUsername -Password $outputDBpassword -Query $sql -QueryTimeout 3600
    $data | %{'{0}' -f $_[0]} 
    }
    
    #check if there are any standard edition dbs in the collected metrics
    $checkslo = "SELECT TOP 1 slo FROM resource_stats_output WHERE slo LIKE 'S%' AND slo NOT LIKE 'Shared%'"
    $output = Invoke-Sqlcmd -ServerInstance $outputServerName -Database $outputdatabaseName -Username $outputDBUsername -Password $outputDBpassword -Query $checkslo -QueryTimeout 3600 | select -expand slo
    if ($output -like "S*")
    {
    write-host "`nStandard edition:" -BackgroundColor Green -ForegroundColor Black
    $sql = $sql1 + "'S%' AND slo NOT LIKE 'Shared%'"  + $sql2
    $data = Invoke-Sqlcmd -ServerInstance $outputServerName -Database $outputdatabaseName -Username $outputDBUsername -Password $outputDBpassword -Query $sql -QueryTimeout 3600
    $data | %{'{0}' -f $_[0]}
    }
    
    #check if there are any premium edition dbs in the collected metrics
    $checkslo = "SELECT TOP 1 slo FROM resource_stats_output WHERE slo LIKE 'P%'"
    $output = Invoke-Sqlcmd -ServerInstance $outputServerName -Database $outputdatabaseName -Username $outputDBUsername -Password $outputDBpassword -Query $checkslo -QueryTimeout 3600 | select -expand slo
    if ($output -like "P*")
    {
    write-host "`nPremium edition:" -BackgroundColor Green -ForegroundColor Black
    $sql = $sql1 + "'P%'"  + $sql2
    $data = Invoke-Sqlcmd -ServerInstance $outputServerName -Database $outputdatabaseName -Username $outputDBUsername -Password $outputDBpassword -Query $sql -QueryTimeout 3600
    $data | %{'{0}' -f $_[0]}
    }
        

## 摘要

并非所有单一数据库都是弹性数据库池的最佳候选项。使用模式表现为低平均使用率与相对不频繁使用率高峰的数据库，是弹性数据库池的绝佳候选项。应用程序使用模式是动态的，因此，请使用本文中所述的信息和工具来进行初始评估，以确定弹性数据库池是否是部分或所有数据库的良好选择。本文只是帮助你确定弹性数据库池是否合适的起点。请记住，你应该持续监视历史资源使用量（使用 STA 或 DMV），并不断地重新评估所有数据库的性能级别。要知道，你可以轻松地将数据库移入和移出弹性数据库池，如果你有极多的数据库，可以创建大小不一的池，以将数据库划分到其中。



<!--Image references-->
[1]: ./media/sql-database-elastic-pool-guidance/one-database.png
[2]: ./media/sql-database-elastic-pool-guidance/four-databases.png
[3]: ./media/sql-database-elastic-pool-guidance/twenty-databases.png

<!---HONumber=Mooncake_0118_2016-->
