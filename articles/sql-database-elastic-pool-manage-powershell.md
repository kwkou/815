<properties 
    pageTitle="管理弹性数据库池 (PowerShell) | Azure" 
    description="了解如何使用 PowerShell 管理弹性数据库池。"  
    services="sql-database" 
    documentationCenter="" 
    authors="stevestein" 
    manager="jhubbard" 
    editor=""/>

<tags
    ms.service="sql-database"
    ms.date="03/15/2016"
    wacn.date="04/06/2016"/>

# 监视和管理弹性数据库池 (PowerShell) 

> [AZURE.SELECTOR]
- [Azure 门户](/documentation/articles/sql-database-elastic-pool-manage-portal)
- [PowerShell](/documentation/articles/sql-database-elastic-pool-manage-powershell)
- [C#](/documentation/articles/sql-database-elastic-pool-manage-csharp)


了解如何使用 PowerShell cmdlet 管理[弹性数据库池](/documentation/articles/sql-database-elastic-pool)。

> [AZURE.NOTE] 弹性数据库池目前为预览版，仅适用于 SQL 数据库 V12 服务器。如果你有一个 SQL 数据库 V11 服务器，可以通过一个步骤[使用 PowerShell 升级到 V12 并创建池](/documentation/articles/sql-database-upgrade-server-portal)。

你需要运行 Azure PowerShell 1.0 或更高版本。有关详细信息，请参阅[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure)。




## 在弹性数据库池中创建新的弹性数据库

若要直接在池中创建新的数据库，请使用 [New-AzureRmSqlDatabase](https://msdn.microsoft.com/zh-cn/library/azure/mt619339.aspx) cmdlet 并设置 **ElasticPoolName** 参数。


	New-AzureRmSqlDatabase -ResourceGroupName "resourcegroup1" -ServerName "server1" -DatabaseName "database1" -ElasticPoolName "elasticpool1"


## 将独立的数据库移动到弹性数据库池

若要将现有数据库移到池中，请使用 [Set-AzureRmSqlDatabase](https://msdn.microsoft.com/zh-cn/library/azure/mt619433.aspx) cmdlet 并设置 **ElasticPoolName** 参数。

	Set-AzureRmSqlDatabase -ResourceGroupName "resourcegroup1" -ServerName "server1" -DatabaseName "database1" -ElasticPoolName "elasticpool1"


## 更改弹性数据库池的性能设置

若要更改弹性数据库池的性能设置，请使用 [Set-AzureRmSqlElasticPool](https://msdn.microsoft.com/zh-cn/library/azure/mt603511.aspx) cmdlet。

    Set-AzureRmSqlElasticPool –ResourceGroupName “resourcegroup1” –ServerName “server1” –ElasticPoolName “elasticpool1” –Dtu 1200 –DatabaseDtuMax 100 –DatabaseDtuMin 50 


## 获取弹性数据库池的操作状态

你可以使用 [Get-AzureRmSqlElasticPoolActivity](https://msdn.microsoft.com/zh-cn/library/azure/mt603812.aspx) cmdlet 跟踪弹性数据库池的操作状态（包括创建和更新）。

	Get-AzureRmSqlElasticPoolActivity –ResourceGroupName “resourcegroup1” –ServerName “server1” –ElasticPoolName “elasticpool1” 


## 获取将弹性数据库移入和移出弹性数据库池的状态

你可以使用 [Get-AzureRmSqlDatabaseActivity](https://msdn.microsoft.com/zh-cn/library/azure/mt603687.aspx) cmdlet 跟踪弹性数据库的操作状态（包括创建和更新）。

	Get-AzureRmSqlDatabaseActivity -ResourceGroupName "resourcegroup1" -ServerName "server1" -DatabaseName "database1" -ElasticPoolName "elasticpool1"

## 获取弹性数据库池的使用情况数据

可以检索的以资源池限制值百分比形式表示的指标：

* 平均 CPU 使用率 - cpu\_percent 
* 平均 IO 使用率 - data\_io\_percent 
* 平均日志使用率 - log\_write\_percent 
* 平均内存使用率 - memory\_percent 
* 平均 eDTU 使用率（以 CPU/IO/日志最大使用率的形式表示）– DTU\_percent 
* 并发用户请求（工作进程）最大数 – max\_concurrent\_requests 
* 并发用户会话最大数 – max\_concurrent\_sessions 
* 弹性池的总存储大小 – storage\_in\_megabytes 


指标粒度/保持期：

* 将以 5 分钟的粒度返回数据。  
* 数据保持期为 14 天。  


此 cmdlet 和 API 将能够在一次调用中检索的行数限制为 1000 行（大约 3 天的数据，如果粒度值为 5 分钟的话）。不过，可以使用不同的开始/结束时间间隔来多次调用此命令，以便检索更多数据


检索指标：

	$metrics = (Get-Metrics -ResourceId /subscriptions/<subscriptionId>/resourceGroups/FabrikamData01/providers/Microsoft.Sql/servers/fabrikamsqldb02/elasticPools/franchisepool -TimeGrain ([TimeSpan]::FromMinutes(5)) -StartTime "4/18/2015" -EndTime "4/21/2015") 

通过重复调用和追加数据来获取更多天数：

	$metrics = $metrics + (Get-Metrics -ResourceId /subscriptions/<subscriptionId>/resourceGroups/FabrikamData01/providers/Microsoft.Sql/servers/fabrikamsqldb02/elasticPools/franchisepool -TimeGrain ([TimeSpan]::FromMinutes(5)) -StartTime "4/21/2015" -EndTime "4/24/2015") 
 
设置表的格式：

    $table = Format-MetricsAsTable $metrics 

导出到 CSV 文件：

    foreach($e in $table) { Export-csv -Path c:\temp\metrics.csv -input $e -Append -NoTypeInformation} 

## 获取弹性数据库的资源消耗指标

这些 API 与当前用来监视单独数据库的资源使用率的 (V12) API 相同，但存在以下语义差异

* 就此 API 来说，检索的度量值将表示为为该弹性数据库池设置的单个 databaseDtuMax（或者 CPU、IO 等基础度量值的等效最大值）的百分比。例如，对于任何此类度量值来说，50% 的使用率表示特定资源消耗为父弹性数据库池中该资源的相应 DB 上限的 50%。 

获取指标：

    $metrics = (Get-Metrics -ResourceId /subscriptions/<subscriptionId>/resourceGroups/FabrikamData01/providers/Microsoft.Sql/servers/fabrikamsqldb02/databases/myDB -TimeGrain ([TimeSpan]::FromMinutes(5)) -StartTime "4/18/2015" -EndTime "4/21/2015") 

重复进行调用并追加数据，以便根据需要获取更多天数：

    $metrics = $metrics + (Get-Metrics -ResourceId /subscriptions/<subscriptionId>/resourceGroups/FabrikamData01/providers/Microsoft.Sql/servers/fabrikamsqldb02/databases/myDB -TimeGrain ([TimeSpan]::FromMinutes(5)) -StartTime "4/21/2015" -EndTime "4/24/2015") 

设置表的格式：

    $table = Format-MetricsAsTable $metrics 

导出到 CSV 文件：

    foreach($e in $table) { Export-csv -Path c:\temp\metrics.csv -input $e -Append -NoTypeInformation}


## 监视和管理弹性数据库池 PowerShell 示例


    $subscriptionId = '<Azure subscription id>'
    $resourceGroupName = '<resource group name>'
    $location = '<datacenter location>'
    $serverName = '<server name>'
    $poolName = '<pool name>'
    $databaseName = '<database name>'
    
    Login-AzureRmAccount -EnvironmentName AzrueChinaCloud
    Set-AzureRmContext -SubscriptionId $subscriptionId
    
    
    Set-AzureRmSqlElasticPool –ResourceGroupName $resourceGroupName –ServerName $serverName –ElasticPoolName $poolName –Dtu 1200 –DatabaseDtuMax 100 –DatabaseDtuMin 50 
    
    $poolResourceId = '/subscriptions/' + $subscriptionId + '/resourceGroups/' + $resourceGroupName + '/providers/Microsoft.Sql/servers/' + $serverName + '/elasticPools/' + $poolName
    $dbResourceId = '/subscriptions/' + $subscriptionId + '/resourceGroups/' + $resourceGroupName + '/providers/Microsoft.Sql/servers/' + $serverName + '/databases/' + $databaseName 
    $startTime1 = '2/10/2016'
    $endTime1 = '2/14/2016'
    $startTime2 = '2/14/2016'
    $endTime2 = '2/18/2016'
    
    
    
    $metrics = (Get-Metrics -ResourceId $poolResourceId -TimeGrain ([TimeSpan]::FromMinutes(5)) -StartTime $startTime1 -EndTime $endTime1) 
    $metrics
    
    $metrics = $metrics + (Get-Metrics -ResourceId $poolResourceId -TimeGrain ([TimeSpan]::FromMinutes(5)) -StartTime $startTime2 -EndTime $endTime2)
    $table = Format-MetricsAsTable $metrics
    foreach($e in $table) { Export-csv -Path c:\temp\metrics.csv -input $e -Append -NoTypeInformation}
    
    $metrics = (Get-Metrics -ResourceId $dbResourceId -TimeGrain ([TimeSpan]::FromMinutes(5)) -StartTime $startTime1 -EndTime $endTime1) 
    $metrics = $metrics + (Get-Metrics -ResourceId $dbResourceId -TimeGrain ([TimeSpan]::FromMinutes(5)) -StartTime $startTime2 -EndTime $endTime2)
    $table = Format-MetricsAsTable $metrics
    foreach($e in $table) { Export-csv -Path c:\temp\metrics.csv -input $e -Append -NoTypeInformation}

## 后续步骤

- [创建弹性作业](/documentation/articles/sql-database-elastic-jobs-overview)弹性作业可以用来根据池中数据库的数目来运行 T-SQL 脚本。


## 弹性数据库参考

有关弹性数据库和弹性数据库池的详细信息，包括 API 和错误详细信息，请参阅[弹性数据库池参考](/documentation/articles/sql-database-elastic-pool-reference)。

<!---HONumber=Mooncake_0328_2016-->
