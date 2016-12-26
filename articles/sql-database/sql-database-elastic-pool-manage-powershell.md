<properties 
    pageTitle="管理弹性数据库池 (PowerShell) | Azure" 
    description="了解如何使用 PowerShell 管理弹性数据库池。"  
	services="sql-database" 
    documentationCenter="" 
    authors="srinia" 
    manager="jhubbard" 
    editor=""/>

<tags
    ms.service="sql-database"
    ms.devlang="NA"
    ms.topic="article"
    ms.tgt_pltfrm="powershell"
    ms.workload="data-management" 
    ms.date="06/22/2016" 
    wacn.date="12/26/2016"
    ms.author="srinia"/>

# 使用 PowerShell 监视和管理弹性数据库池 

> [AZURE.SELECTOR]
- [Azure 门户预览](/documentation/articles/sql-database-elastic-pool-manage-portal/)
- [PowerShell](/documentation/articles/sql-database-elastic-pool-manage-powershell/)
- [C#](/documentation/articles/sql-database-elastic-pool-manage-csharp/)
- [T-SQL](/documentation/articles/sql-database-elastic-pool-manage-tsql/)

使用 PowerShell cmdlet 管理[弹性数据库池](/documentation/articles/sql-database-elastic-pool/)。

有关常见的错误代码，请参阅 [SQL 数据库客户端应用程序的 SQL 错误代码：数据库连接错误和其他问题](/documentation/articles/sql-database-develop-error-messages/)。

可以在 [eDTU 和存储限制](/documentation/articles/sql-database-elastic-pool/#eDTU-and-storage-limits-for-elastic-pools-and-elastic-databases)中找到有关池的值。

## 先决条件

* Azure PowerShell 1.0 或更高版本。有关详细信息，请参阅[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/)。
* 弹性数据库池只能在 SQL 数据库 V12 服务器中使用。如果有一个 SQL 数据库 V11 服务器，可以通过一个步骤[使用 PowerShell 升级到 V12 并创建池](/documentation/articles/sql-database-upgrade-server-portal/)。


##<a name="Move-a-database-into-an-elastic-pool"></a> 将数据库移入弹性池

使用 [Set-AzureRmSqlDatabase](https://msdn.microsoft.com/zh-cn/library/azure/mt619433.aspx) 可以将数据库移入或移出池。

	Set-AzureRmSqlDatabase -ResourceGroupName "resourcegroup1" -ServerName "server1" -DatabaseName "database1" -ElasticPoolName "elasticpool1"

##<a name="change-performance-settings-of-a-pool"></a> 更改池的性能设置

当性能受到影响时，可以更改池的设置以适应增长。使用 [Set-AzureRmSqlElasticPool](https://msdn.microsoft.com/zh-cn/library/azure/mt603511.aspx) cmdlet。将 -Dtu 参数设置为每个池的 eDTU。有关该参数可能的值，请参阅 [eDTU 和存储限制](/documentation/articles/sql-database-elastic-pool/#eDTU-and-storage-limits-for-elastic-pools-and-elastic-databases)。

    Set-AzureRmSqlElasticPool –ResourceGroupName “resourcegroup1” –ServerName “server1” –ElasticPoolName “elasticpool1” –Dtu 1200 –DatabaseDtuMax 100 –DatabaseDtuMin 50 


## 获取池操作的状态

创建池需要一些时间。可以使用 [Get-AzureRmSqlElasticPoolActivity](https://msdn.microsoft.com/zh-cn/library/azure/mt603812.aspx) cmdlet 跟踪池操作（包括创建和更新）的状态。

	Get-AzureRmSqlElasticPoolActivity –ResourceGroupName “resourcegroup1” –ServerName “server1” –ElasticPoolName “elasticpool1” 


## 获取将弹性数据库移入和移出池的状态

移动数据库需要一些时间。可以使用 [Get AzureRmSqlDatabaseActivity](https://msdn.microsoft.com/zh-cn/library/azure/mt603687.aspx) cmdlet 跟踪移动状态。

	Get-AzureRmSqlDatabaseActivity -ResourceGroupName "resourcegroup1" -ServerName "server1" -DatabaseName "database1" -ElasticPoolName "elasticpool1"

## 获取池的资源使用情况数据

以资源池限制的百分比形式所表示的可以检索的指标：


| 指标名称 | 说明 |
| :-- | :-- |
| cpu\_percent | 以池限制的百分比形式所表示的平均计算使用率。 |
| physical\_data\_read\_percent | 以基于池限制的百分比形式所表示的平均 I/O 使用率。 |
| log\_write\_percent | 以池限制的百分比形式所表示的平均写入资源使用率。 | 
| DTU\_consumption\_percent | 以池的 eDTU 限制的百分比形式所表示的平均 eDTU 使用率 | 
| storage\_percent | 以池的存储限制的百分比形式所表示的平均存储使用率。 |  
| workers\_percent | 以基于池的限制的百分比形式所表示的最大并发辅助角色（请求）数量。 |  
| sessions\_percent | 以基于池的限制的百分比形式所表示的最大并发会话数量。 | 
| eDTU\_limit | 在此时间间隔内该弹性池的当前最大弹性池 DTU 设置。 |
| storage\_limit | 在此时间间隔内该弹性池的当前最大弹性池存储限制设置（以兆字节为单位）。 |
| eDTU\_used | 在此时间间隔内池使用的平均 eDTU。 |
| storage\_used | 在此时间间隔内池使用的平均存储空间（以字节为单位） |

**指标粒度/保留期：**

* 将以 5 分钟的粒度返回数据。
* 数据保留期为 35 天。

此 cmdlet 和 API 将能够在一次调用中检索的行数限制为 1000 行（以 5 分钟的粒度为准，为大约 3 天的数据）。不过，可以使用不同的开始/结束时间间隔来多次调用此命令，以检索更多数据

检索指标：

	$metrics = (Get-AzureRmMetric -ResourceId /subscriptions/<subscriptionId>/resourceGroups/FabrikamData01/providers/Microsoft.Sql/servers/fabrikamsqldb02/elasticPools/franchisepool -TimeGrain ([TimeSpan]::FromMinutes(5)) -StartTime "4/18/2015" -EndTime "4/21/2015")  


##<a name="elastic-database-monitoring"></a> 获取弹性数据库的资源使用情况数据

这些 API 与用于监视单独数据库的资源使用情况的当前 (V12) API 相同，但存在以下语义差异。

对于此 API，检索的指标表示为为该池设置的单个最大 eDTU（或者 CPU、IO 等基础指标的等效最大值）的百分比。例如，对于任何此类指标来说，50% 的使用率表示特定资源消耗为父池中该资源的每个数据库上限的 50%。

检索指标：

    $metrics = (Get-AzureRmMetric -ResourceId /subscriptions/<subscriptionId>/resourceGroups/FabrikamData01/providers/Microsoft.Sql/servers/fabrikamsqldb02/databases/myDB -TimeGrain ([TimeSpan]::FromMinutes(5)) -StartTime "4/18/2015" -EndTime "4/21/2015") 

##<a name="add-an-alert-to-a-pool-resource"></a> 向池资源添加警报

可以向资源添加警报规则，以便在资源达到设置的使用阈值时，向 [URL 终结点](https://msdn.microsoft.com/zh-cn/library/mt718036.aspx)发送电子邮件通知或警报字符串。使用 Add-AzureRmMetricAlertRule cmdlet。

> [AZURE.IMPORTANT]对弹性池资源利用率的监视存在至少 20 分钟的延迟。当前不支持为弹性池设置短于 30 分钟的警报。为弹性池设置的任何时长（PowerShell API 中名为“-WindowSize”的参数）短于 30 分钟的警报可能无法被触发。请确保为弹性池定义的任何警报的时长不短于 30 分钟 (WindowSize)。

该示例添加了一个警报，以便在池的 eDTU 消耗超出特定阈值时获取通知。

    # Set up your resource ID configurations
    $subscriptionId = '<Azure subscription id>'      # Azure subscription ID
    $location =  '<location'                         # Azure region
    $resourceGroupName = '<resource group name>'     # Resource Group
    $serverName = '<server name>'                    # server name
    $poolName = '<elastic pool name>'                # pool name 

    #$Target Resource ID
    $ResourceID = '/subscriptions/' + $subscriptionId + '/resourceGroups/' +$resourceGroupName + '/providers/Microsoft.Sql/servers/' + $serverName + '/elasticpools/' + $poolName

    # Create an email action
    $actionEmail = New-AzureRmAlertRuleEmail -SendToServiceOwners -CustomEmail JohnDoe@contoso.com

    # create a unique rule name
    $alertName = $poolName + "- DTU consumption rule"

    # Create an alert rule for DTU_consumption_percent
    Add-AzureRMMetricAlertRule -Name $alertName -Location $location -ResourceGroup $resourceGroupName -TargetResourceId $ResourceID -MetricName "DTU_consumption_percent"  -Operator GreaterThan -Threshold 80 -TimeAggregationOperator Average -WindowSize 00:60:00 -Actions $actionEmail 

## 将警报添加到池中的所有数据库

可以将警报规则添加到弹性池中的所有数据库，以便在资源达到警报设置的使用阈值时，向 [URL 终结点](https://msdn.microsoft.com/zh-cn/library/mt718036.aspx)发送电子邮件通知或警报字符串。

> [AZURE.IMPORTANT] 对弹性池资源利用率的监视存在至少 20 分钟的延迟。当前不支持为弹性池设置短于 30 分钟的警报。为弹性池设置的任何时长（PowerShell API 中名为“-WindowSize”的参数）短于 30 分钟的警报可能无法被触发。请确保为弹性池定义的任何警报的时长不短于 30 分钟 (WindowSize)。

该示例向池中的所有数据库添加了一个警报，以便在数据库的 DTU 消耗超出特定阈值时获取通知。

    # Set up your resource ID configurations
    $subscriptionId = '<Azure subscription id>'      # Azure subscription ID
    $location = '<location'                          # Azure region
    $resourceGroupName = '<resource group name>'     # Resource Group
    $serverName = '<server name>'                    # server name
    $poolName = '<elastic pool name>'                # pool name 

    # Get the list of databases in this pool.
    $dbList = Get-AzureRmSqlElasticPoolDatabase -ResourceGroupName $resourceGroupName -ServerName $serverName -ElasticPoolName $poolName

    # Create an email action
    $actionEmail = New-AzureRmAlertRuleEmail -SendToServiceOwners -CustomEmail JohnDoe@contoso.com

    # Get resource usage metrics for a database in an elastic database for the specified time interval.
    foreach ($db in $dbList)
    {
    $dbResourceId = '/subscriptions/' + $subscriptionId + '/resourceGroups/' + $resourceGroupName + '/providers/Microsoft.Sql/servers/' + $serverName + '/databases/' + $db.DatabaseName

    # create a unique rule name
    $alertName = $db.DatabaseName + "- DTU consumption rule"

    # Create an alert rule for DTU_consumption_percent
    Add-AzureRMMetricAlertRule -Name $alertName  -Location $location -ResourceGroup $resourceGroupName -TargetResourceId $dbResourceId -MetricName "dtu_consumption_percent"  -Operator GreaterThan -Threshold 80 -TimeAggregationOperator Average -WindowSize 00:60:00 -Actions $actionEmail

    # drop the alert rule
    #Remove-AzureRmAlertRule -ResourceGroup $resourceGroupName -Name $alertName
    } 



## 在一个订阅的多个池中收集和监视资源使用情况数据

当一个订阅中有大量数据库时，单独监视每个弹性池非常麻烦。相反，可以将 SQL 数据库 PowerShell cmdlet 和 T-SQL 查询结合使用，从多个池及其数据库中收集资源使用情况数据，以便监视和分析资源使用情况。可以在 GitHub SQL Server 示例存储库及有关该存储库的作用和如何使用的文档中找到 powershell 脚本集合的[示例实现](https://github.com/Microsoft/sql-server-samples/tree/master/samples/manage/azure-sql-db-elastic-pools)。

要使用此示例实现，请按照下面所列的这些步骤进行操作。


1. 下载[脚本和文档](https://github.com/Microsoft/sql-server-samples/tree/master/samples/manage/azure-sql-db-elastic-pools)：
2. 修改环境的脚本。指定在其上托管弹性池的一个或多个服务器。
3. 指定要存储收集的指标的遥测数据库。
4. 自定义脚本以指定脚本的执行持续时间。

在高级别中，脚本执行以下操作：

*	枚举给定的 Azure 订阅（或指定的服务器列表）中的所有服务器。
*	为每个服务器运行后台作业。该作业在固定的时间间隔内循环运行，并收集服务器中所有池的遥测数据。然后将所收集的数据加载到指定的遥测数据库中。
*	枚举每个池中的数据库列表，以收集数据库资源使用情况数据。然后将所收集的数据加载到遥测数据库中。

可以对遥测数据库中收集的指标值进行分析，以监视弹性池及其中的数据库的运行状况。该脚本还在遥测数据库中安装预定义的表值函数 (TVF)，以帮助聚合指定时间范围内的指标值。例如，TVF 的结果可用来显示“指定时间范围内具有最大 eDTU 使用量的前 N 个弹性池”。 或者，使用诸如 Excel 或 Power BI 这样的分析工具对收集的数据进行查询和分析。

## 示例：检索池及其数据库的资源消耗指标值

该示例将检索给定的弹性池及其所有数据库的资源消耗指标值。所收集的数据被格式化并写入到 .csv 格式文件中。可以使用 Excel 浏览该文件。

	$subscriptionId = '<Azure subscription id>'	      # Azure subscription ID
	$resourceGroupName = '<resource group name>'             # Resource Group
	$serverName = <server name>                              # server name
	$poolName = <elastic pool name>                          # pool name
		
	# Login to Azure account and select the subscription.
	Login-AzureRmAccount -EnvironmentName AzrueChinaCloud
	Set-AzureRmContext -SubscriptionId $subscriptionId
	
	# Get resource usage metrics for an elastic pool for the specified time interval.
	$startTime = '4/27/2016 00:00:00'  # start time in UTC
	$endTime = '4/27/2016 01:00:00'    # end time in UTC
	
	# Construct the pool resource ID and retrive pool metrics at 5 minute granularity.
	$poolResourceId = '/subscriptions/' + $subscriptionId + '/resourceGroups/' + $resourceGroupName + '/providers/Microsoft.Sql/servers/' + $serverName + '/elasticPools/' + $poolName
	$poolMetrics = (Get-AzureRmMetric -ResourceId $poolResourceId -TimeGrain ([TimeSpan]::FromMinutes(5)) -StartTime $startTime -EndTime $endTime) 
	
	# Get the list of databases in this pool.
	$dbList = Get-AzureRmSqlElasticPoolDatabase -ResourceGroupName $resourceGroupName -ServerName $serverName -ElasticPoolName $poolName
	
	# Get resource usage metrics for a database in an elastic database for the specified time interval.
	$dbMetrics = @()
	foreach ($db in $dbList)
	{
	    $dbResourceId = '/subscriptions/' + $subscriptionId + '/resourceGroups/' + $resourceGroupName + '/providers/Microsoft.Sql/servers/' + $serverName + '/databases/' + $db.DatabaseName
	    $dbMetrics = $dbMetrics + (Get-AzureRmMetric -ResourceId $dbResourceId -TimeGrain ([TimeSpan]::FromMinutes(5)) -StartTime $startTime -EndTime $endTime)
	}
	
	#Optionally you can format the metrics and output as .csv file using the following script block.
	$command = {
    param($metricList, $outputFile)

    # Format metrics into a table.
    $table = @()
    foreach($metric in $metricList) { 
      foreach($metricValue in $metric.MetricValues) {
        $sx = New-Object PSObject -Property @{
            Timestamp = $metricValue.Timestamp.ToString()
            MetricName = $metric.Name; 
            Average = $metricValue.Average;
            ResourceID = $metric.ResourceId 
          }
          $table = $table += $sx
      }
    }
    
    # Output the metrics into a .csv file.
    write-output $table | Export-csv -Path $outputFile -Append -NoTypeInformation
	}
	
	# Format and output pool metrics
	Invoke-Command -ScriptBlock $command -ArgumentList $poolMetrics,c:\temp\poolmetrics.csv
	
	# Format and output database metrics
	Invoke-Command -ScriptBlock $command -ArgumentList $dbMetrics,c:\temp\dbmetrics.csv



## 弹性池操作延迟

- 更改每个数据库的最小 eDTU 数或每个数据库的最大 eDTU 数通常可在 5 分钟或更少的时间内完成。
- 更改每个池的 eDTU 数取决于池中所有数据库使用的空间总量。每 100 GB 的更改平均需要 90 分钟或更短的时间。例如，如果池中所有数据库使用的总空间为 200 GB，则更改每个池的池 eDTU 时，预计延迟为 3 小时或更短的时间。

## 从 V11 服务器迁移到 V12 服务器

可以使用 PowerShell cmdlett 来启动、停止或监视从 Azure SQL 数据库 V11 或其他任何低于 V12 的版本到 V12 的升级。

- [使用 PowerShell 升级到 SQL 数据库 V12](/documentation/articles/sql-database-upgrade-server-powershell/)

有关这些 PowerShell cmdlet 的参考文档，请参阅：


- [Get-AzureRMSqlServerUpgrade](https://msdn.microsoft.com/zh-cn/library/azure/mt603582.aspx)
- [Start-AzureRMSqlServerUpgrade](https://msdn.microsoft.com/zh-cn/library/azure/mt619403.aspx)
- [Stop-AzureRMSqlServerUpgrade](https://msdn.microsoft.com/zh-cn/library/azure/mt603589.aspx)


Stop- cmdlet 表示取消，而不是暂停。无法在中途恢复升级，只能从头开始重新升级。Stop- cmdlet 将清理并释放所有相应的资源。

## 后续步骤

- 请参阅[使用 Azure SQL 数据库进行扩展](/documentation/articles/sql-database-elastic-scale-introduction/)：使用弹性数据库工具扩展、移动数据、查询或创建事务。

<!---HONumber=Mooncake_Quality_Review_1215_2016-->