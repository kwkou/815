<properties 
    pageTitle="使用弹性数据库池向外缩放资源 | Azure" 
    description="了解如何通过创建弹性数据库池，使用 PowerShell 向外缩放 Azure SQL 数据库资源以管理多个数据库。" 
    keywords="多个数据库,向外缩放"    
    services="sql-database" 
    documentationCenter="" 
    authors="stevestein" 
    manager="jeffreyg" 
    editor=""/>

<tags
    ms.service="sql-database"
    ms.date="02/23/2016"
    wacn.date="04/06/2016"/>

# 使用 PowerShell 创建弹性数据库池以便向外缩放多个 SQL 数据库的资源 

> [AZURE.SELECTOR]
- [C#](/documentation/articles/sql-database-elastic-pool-csharp/)
- [PowerShell](/documentation/articles/sql-database-elastic-pool-powershell/)

了解如何通过使用 PowerShell cmdlet 创建[弹性数据库池](/documentation/articles/sql-database-elastic-pool/)来管理多个数据库。

> [AZURE.NOTE] 弹性数据库池目前为预览版，仅适用于 SQL 数据库 V12 服务器。如果你有一个 SQL 数据库 V11 服务器，可以通过一个步骤使用 PowerShell 升级到 V12 并创建池。

弹性数据库池允许你向外缩放数据库资源和跨多个 SQL 数据库进行管理。

本文说明如何创建所有必要项目（包括 V12 服务器，Azure 订阅除外）来创建和配置弹性数据库池。如果你需要 Azure 订阅，只需单击本页顶部的“试用”，然后再回来完成本文的相关操作即可。




我们对使用 Azure PowerShell 创建弹性数据库池的各个步骤进行了剖析。如果你只需命令的简明列表，请参阅本文底部的“汇总”部分。


若要运行 PowerShell cmdlet，需要已安装并运行 Azure PowerShell。有关详细信息，请参阅[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/)。




## 配置你的凭据，然后选择你的订阅

现在，你已经运行 Azure 资源管理器模块，因此可以访问创建和配置弹性数据库池所需的所有 cmdlet。首先，你必须能够访问 Azure 帐户。运行以下项目，然后就会出现一个要求你输入凭据的登录屏幕。使用登录 Azure 门户时所用的相同电子邮件和密码。

	Login-AzureRmAccount -EnvironmentName AzureChinaCloud

成功登录后，你会在屏幕上看到一些信息，其中包括你登录时使用的 ID，以及你有权访问的 Azure 订阅。


### 选择 Azure 订阅

若要选择订阅，你需要提供订阅 ID 或订阅名称 (**-SubscriptionName**)。你可以从前面的步骤中复制该信息，或者，如果你有多个订阅，你可以运行 **Get-AzureSubscription** cmdlet，然后从结果集中复制所需的订阅信息。获得订阅以后，你可以运行以下 cmdlet：

	Select-AzureRmSubscription -SubscriptionId 4cac86b0-1e56-bbbb-aaaa-000000000000


## 创建资源组、服务器和防火墙规则

现在，你已经有了针对 Azure 订阅运行 cmdlet 所需的访问权限，因此下一步是建立一个资源组，其中包含创建弹性数据库池（包含多个数据库）所需的服务器。你可以编辑下一个命令，以便使用所选择的有效位置。运行 **(Get-AzureRmLocation | where-object {$\_.Name -eq "Microsoft.Sql/servers" }).Locations** 以获取有效位置的列表。

如果你已经有了一个资源组，则可转到下一步，也可运行以下命令来创建新的资源组：

	New-AzureRmResourceGroup -Name "resourcegroup1" -Location "China North"

### 创建服务器 

弹性数据库池在 Azure SQL 数据库服务器中创建。如果你已经有了一个服务器，则可转到下一步，也可运行 [New-AzureRmSqlServer](https://msdn.microsoft.com/zh-cn/library/azure/mt603715.aspx) cmdlet 来创建新的 V12 服务器。将 ServerName 替换为你的服务器的名称。该服务器名称对于 Azure SQL Server 必须是唯一的，因此如果该服务器名称已被使用，你会在此处收到一个错误。还必须指出的是，该命令可能需要数分钟才能运行完毕。成功创建服务器后，会显示服务器详细信息和 PowerShell 提示。你可以编辑该命令，以便使用所选的任何有效位置。

	New-AzureRmSqlServer -ResourceGroupName "resourcegroup1" -ServerName "server1" -Location "China North" -ServerVersion "12.0"

当你运行此命令时，会打开一个窗口，要求你提供**用户名**和**密码**。这不是你的 Azure 凭据，请输入用户名和密码，将其作为你要为新服务器创建的管理员凭据。


### 配置服务器防火墙规则，允许对服务器进行访问

建立针对服务器访问的防火墙规则。运行 [New-AzureRmSqlServerFirewallRule](https://msdn.microsoft.com/zh-cn/library/azure/mt603586.aspx) 命令，将开始和结尾的 IP 地址替换为你计算机的有效值。

如果你的服务器需要允许到其他 Azure 服务的访问，请添加 **-AllowAllAzureIPs** 开关，以便添加一个特殊的防火墙规则，允许到服务器的所有 Azure 流量访问。

	New-AzureRmSqlServerFirewallRule -ResourceGroupName "resourcegroup1" -ServerName "server1" -FirewallRuleName "rule1" -StartIpAddress "192.168.0.198" -EndIpAddress "192.168.0.199"

有关详细信息，请参阅 [Azure SQL 数据库防火墙](/documentation/articles/sql-database-firewall-configure/)。


## 创建弹性数据库池和弹性数据库

现在，你已经有了一个资源组、一个服务器，并且配置了防火墙规则，因此可以访问服务器。[New-AzureRmSqlElasticPool](https://msdn.microsoft.com/zh-cn/library/azure/mt619378.aspx) cmdlet 将创建弹性数据库池。该命令将创建一个池，可以共享总共 400 eDTU。可以确保池中的每个数据库始终有 10 eDTU (DatabaseDtuMin) 可用。池中的各个数据库最多可以耗用 100 eDTU (DatabaseDtuMax)。有关详细的参数说明，请参阅 [Azure SQL 数据库弹性池](/documentation/articles/sql-database-elastic-pool/)。


	New-AzureRmSqlElasticPool -ResourceGroupName "resourcegroup1" -ServerName "server1" -ElasticPoolName "elasticpool1" -Edition "Standard" -Dtu 400 -DatabaseDtuMin 10 -DatabaseDtuMax 100


### 在弹性数据库池中创建或添加弹性数据库

在上一步创建的池是空的，里面没有弹性数据库。以下部分说明如何在池中创建新的弹性数据库，以及如何将现有数据库添加到池中。

创建池后，你还可以使用 Transact-SQL 在该池中创建新的弹性数据库，以及将现有数据库移入和移出池。有关详细信息，请参阅[弹性数据库池参考 - Transact-SQL](/documentation/articles/sql-database-elastic-pool-reference/#Transact-SQL)。

### 在弹性数据库池中创建新的弹性数据库

若要直接在池中创建新的数据库，请使用 [New-AzureRmSqlDatabase](https://msdn.microsoft.com/zh-cn/library/azure/mt619339.aspx) cmdlet 并设置 **ElasticPoolName** 参数。


	New-AzureRmSqlDatabase -ResourceGroupName "resourcegroup1" -ServerName "server1" -DatabaseName "database1" -ElasticPoolName "elasticpool1"



### 将现有数据库移入弹性数据库池

若要将现有数据库移到池中，请使用 [Set-AzureRmSqlDatabase](https://msdn.microsoft.com/zh-cn/library/azure/mt619433.aspx) cmdlet 并设置 **ElasticPoolName** 参数。


为了演示，请创建一个不在弹性数据库池中的数据库。

	New-AzureRmSqlDatabase -ResourceGroupName "resourcegroup1" -ServerName "server1" -DatabaseName "database1" -Edition "Standard"

将现有数据库移到弹性数据库池中。

	Set-AzureRmSqlDatabase -ResourceGroupName "resourcegroup1" -ServerName "server1" -DatabaseName "database1" -ElasticPoolName "elasticpool1"

## 更改弹性数据库池的性能设置


    Set-AzureRmSqlElasticPool –ResourceGroupName “resourcegroup1” –ServerName “server1” –ElasticPoolName “elasticpool1” –Dtu 1200 –DatabaseDtuMax 100 –DatabaseDtuMin 50 


## 监视弹性数据库和弹性数据库池
弹性数据库池提供度量值报告以帮助你向外缩放资源来管理多个数据库。


### 获取弹性数据库池的操作状态

你可以跟踪弹性数据库池的操作（包括创建和更新）状态。

	Get-AzureRmSqlElasticPoolActivity –ResourceGroupName “resourcegroup1” –ServerName “server1” –ElasticPoolName “elasticpool1” 


### 获取将弹性数据库移入和移出弹性数据库池的状态

	Get-AzureRmSqlDatabaseActivity -ResourceGroupName "resourcegroup1" -ServerName "server1" -DatabaseName "database1" -ElasticPoolName "elasticpool1"

### 获取弹性数据库池的资源消耗度量值

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

	$metrics = (Get-Metrics -ResourceId /subscriptions/d7c1d29a-ad13-4033-877e-8cc11d27ebfd/resourceGroups/FabrikamData01/providers/Microsoft.Sql/servers/fabrikamsqldb02/elasticPools/franchisepool -TimeGrain ([TimeSpan]::FromMinutes(5)) -StartTime "4/18/2015" -EndTime "4/21/2015") 

通过重复调用和追加数据来获取更多天数：

	$metrics = $metrics + (Get-Metrics -ResourceId /subscriptions/d7c1d29a-ad13-4033-877e-8cc11d27ebfd/resourceGroups/FabrikamData01/providers/Microsoft.Sql/servers/fabrikamsqldb02/elasticPools/franchisepool -TimeGrain ([TimeSpan]::FromMinutes(5)) -StartTime "4/21/2015" -EndTime "4/24/2015") 
 
设置表的格式：

    $table = Format-MetricsAsTable $metrics 

导出到 CSV 文件：

    foreach($e in $table) { Export-csv -Path c:\temp\metrics.csv -input $e -Append -NoTypeInformation} 

### 获取弹性数据库的资源消耗指标

这些 API 与当前用来监视单独数据库的资源使用率的 (V12) API 相同，但存在以下语义差异

* 就此 API 来说，检索的度量值将表示为为该弹性数据库池设置的单个 databaseDtuMax（或者 CPU、IO 等基础度量值的等效最大值）的百分比。例如，对于任何此类度量值来说，50% 的使用率表示特定资源消耗为父弹性数据库池中该资源的相应 DB 上限的 50%。 

获取指标：

    $metrics = (Get-Metrics -ResourceId /subscriptions/d7c1d29a-ad13-4033-877e-8cc11d27ebfd/resourceGroups/FabrikamData01/providers/Microsoft.Sql/servers/fabrikamsqldb02/databases/myDB -TimeGrain ([TimeSpan]::FromMinutes(5)) -StartTime "4/18/2015" -EndTime "4/21/2015") 

重复进行调用并追加数据，以便根据需要获取更多天数：

    $metrics = $metrics + (Get-Metrics -ResourceId /subscriptions/d7c1d29a-ad13-4033-877e-8cc11d27ebfd/resourceGroups/FabrikamData01/providers/Microsoft.Sql/servers/fabrikamsqldb02/databases/myDB -TimeGrain ([TimeSpan]::FromMinutes(5)) -StartTime "4/21/2015" -EndTime "4/24/2015") 

设置表的格式：

    $table = Format-MetricsAsTable $metrics 

导出到 CSV 文件：

    foreach($e in $table) { Export-csv -Path c:\temp\metrics.csv -input $e -Append -NoTypeInformation}


## 汇总


    Add-AzureRmAccount -EnvironmentName AzureChinaCloud
    Select-AzureRmSubscription -SubscriptionId 4cac86b0-1e56-bbbb-aaaa-000000000000
    New-AzureRmResourceGroup -Name "resourcegroup1" -Location "China North"
    New-AzureRmSqlServer -ResourceGroupName "resourcegroup1" -ServerName "server1" -Location "China North" -ServerVersion "12.0"
    New-AzureRmSqlServerFirewallRule -ResourceGroupName "resourcegroup1" -ServerName "server1" -FirewallRuleName "rule1" -StartIpAddress "192.168.0.198" -EndIpAddress "192.168.0.199"
    New-AzureRmSqlElasticPool -ResourceGroupName "resourcegroup1" -ServerName "server1" -ElasticPoolName "elasticpool1" -Edition "Standard" -Dtu 400 -DatabaseDtuMin 10 -DatabaseDtuMax 100
    New-AzureRmSqlDatabase -ResourceGroupName "resourcegroup1" -ServerName "server1" -DatabaseName "database1" -ElasticPoolName "elasticpool1" -MaxSizeBytes 10GB
    Set-AzureRmSqlElasticPool –ResourceGroupName “resourcegroup1” –ServerName “server1” –ElasticPoolName “elasticpool1” –Dtu 1200 –DatabaseDtuMax 100 –DatabaseDtuMin 50 
    
    $metrics = (Get-Metrics -ResourceId /subscriptions/d7c1d29a-ad13-4033-877e-8cc11d27ebfd/resourceGroups/FabrikamData01/providers/Microsoft.Sql/servers/fabrikamsqldb02/elasticPools/franchisepool -TimeGrain ([TimeSpan]::FromMinutes(5)) -StartTime "4/18/2015" -EndTime "4/21/2015") 
    $metrics = $metrics + (Get-Metrics -ResourceId /subscriptions/d7c1d29a-ad13-4033-877e-8cc11d27ebfd/resourceGroups/FabrikamData01/providers/Microsoft.Sql/servers/fabrikamsqldb02/elasticPools/franchisepool -TimeGrain ([TimeSpan]::FromMinutes(5)) -StartTime "4/21/2015" -EndTime "4/24/2015")
    $table = Format-MetricsAsTable $metrics
    foreach($e in $table) { Export-csv -Path c:\temp\metrics.csv -input $e -Append -NoTypeInformation}

    $metrics = (Get-Metrics -ResourceId /subscriptions/d7c1d29a-ad13-4033-877e-8cc11d27ebfd/resourceGroups/FabrikamData01/providers/Microsoft.Sql/servers/fabrikamsqldb02/databases/myDB -TimeGrain ([TimeSpan]::FromMinutes(5)) -StartTime "4/18/2015" -EndTime "4/21/2015") 
    $metrics = $metrics + (Get-Metrics -ResourceId /subscriptions/d7c1d29a-ad13-4033-877e-8cc11d27ebfd/resourceGroups/FabrikamData01/providers/Microsoft.Sql/servers/fabrikamsqldb02/databases/myDB -TimeGrain ([TimeSpan]::FromMinutes(5)) -StartTime "4/21/2015" -EndTime "4/24/2015")
    $table = Format-MetricsAsTable $metrics
    foreach($e in $table) { Export-csv -Path c:\temp\metrics.csv -input $e -Append -NoTypeInformation}

## 后续步骤

创建弹性数据库池后，你可以通过创建弹性作业来管理池中的弹性数据库。弹性作业可以用来根据池中数据库的数目来运行 T-SQL 脚本。有关详细信息，请参阅[弹性数据库作业概述](/documentation/articles/sql-database-elastic-jobs-overview/)。


## 弹性数据库参考

有关弹性数据库和弹性数据库池的详细信息，包括 API 和错误详细信息，请参阅[弹性数据库池参考](/documentation/articles/sql-database-elastic-pool-reference/)。

<!---HONumber=Mooncake_0328_2016-->
