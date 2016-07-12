<properties 
	pageTitle="使用 PowerShell 升级到 Azure SQL 数据库 V12 | Azure" 
	description="介绍如何使用 PowerShell 升级到 Azure SQL 数据库 V12，包括如何升级 Web 和企业数据库，以及如何升级 V11 服务器并将其数据库直接迁移到弹性数据库池。" 
	services="sql-database" 
	documentationCenter="" 
	authors="stevestein" 
	manager="jeffreyg" 
	editor=""/>

<tags 
	ms.service="sql-database" 
	ms.date="11/11/2015" 
	wacn.date="01/29/2016"/>

# 使用 PowerShell 升级到 Azure SQL 数据库 V12


> [AZURE.SELECTOR]
- [PowerShell](/documentation/articles/sql-database-upgrade-server-powershell/)


SQL 数据库 V12 是最新版本，因此我们建议升级到 SQL 数据库 V12。SQL 数据库 V12 具有[旧版所欠缺的许多优点](/documentation/articles/sql-database-v12-whats-new/)，包括：

- 提高了与 SQL Server 的兼容性。
- 经过改进的高级性能和新的性能级别。
- [弹性数据库池](/documentation/articles/sql-database-elastic-pool/)。

本文提供有关将现有 SQL 数据库 V11 服务器和数据库升级到 SQL 数据库 V12 的指导。

在升级到 V12 的过程中，会将所有 Web 和企业数据库升级到新的服务级别，因此本文还包含了有关升级 Web 和企业数据库的说明。

此外，与升级到单一数据库的单独性能级别（定价层）相比，迁移到[弹性数据库池](/documentation/articles/sql-database-elastic-pool/)更具成本效益。池还可以简化数据库管理，因为你只需管理池的性能设置，而无需分开管理单个数据库的性能级别。如果你的数据库位于多台服务器上，请考虑将它们迁移到同一台服务器，并利用入池所带来的优势。

遵循本文中的步骤可以轻松地从 V11 服务器直接迁移到弹性数据库池。

请注意，数据库将保持联机，并且在整个升级操作过程中都会继续保持工作。在实际转换到新的性能级别时，数据库连接可能会暂时中断很短的一段时间，通常约 90 秒，但最长可达 5 分钟。如果你的应用程序有[针对连接终止的暂时性故障处理机制](/documentation/articles/sql-database-connect-central-recommendations/)，则足以防止升级结束时连接中断。

升级到 SQL 数据库 V12 的操作不可撤销。升级后，无法将服务器还原到 V11。

## 准备升级

- **升级所有 Web 和企业数据库**：请[使用 PowerShell 来升级数据库和服务器](/documentation/articles/sql-database-upgrade-server-powershell/)。
- **检查和暂停异地复制**：如果你的 Azure SQL 数据库已针对异地复制进行配置，则你应记录其当前配置并停止异地复制。升级完成后，请重新为异地复制配置数据库。
- **如果客户端在 Azure VM 上，请打开这些端口**：如果客户端程序连接到 SQL 数据库 V12，而客户端运行在 Azure 虚拟机 (VM) 上，则必须打开虚拟机上的端口范围 11000-11999 和 14000-14999。有关详细信息，请参阅 [SQL 数据库 V12 的端口](/documentation/articles/sql-database-develop-direct-route-ports-adonet-v12/)。


## 先决条件 

若要使用 PowerShell 将服务器升级到 V12，你需要安装并运行 Azure PowerShell，然后，根据具体的版本，你可能需要切换到资源管理器模式，以访问 Azure 资源管理器 PowerShell cmdlet。

> [AZURE.IMPORTANT] 从 Azure PowerShell 1.0 预览版开始，Switch-AzureMode cmdlet 不再可用，并且 Azure ResourceManger 模块中的 cmdlet 已重命名。本文中的示例使用新的 PowerShell 1.0 预览版命名约定。有关详细信息，请参阅[弃用 Azure PowerShell 中的 Switch-AzureMode](https://github.com/Azure/azure-powershell/wiki/Deprecation-of-Switch-AzureMode-in-Azure-PowerShell)。

若要运行 PowerShell cmdlet，需要安装并运行 Azure PowerShell，而且由于 Switch-AzureMode 已删除，你应该通过运行 [Microsoft Web 平台安装程序](http://go.microsoft.com/fwlink/p/?linkid=320376&clcid=0x409)来下载并安装最新的 Azure PowerShell。有关详细信息，请参阅[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/)。


## 配置你的凭据，然后选择你的订阅

若要针对 Azure 订阅运行 PowerShell cmdlet，必须先与 Azure 帐户建立访问连接。运行以下项目，然后就会出现一个要求你输入凭据的登录屏幕。使用登录 Azure 门户时所用的相同电子邮件和密码。

	Add-AzureRmAccount -EnvironmentName AzureChinaCloud

成功登录后，你会在屏幕上看到一些信息，其中包括你登录时使用的 ID，以及你有权访问的 Azure 订阅。

若要选择要使用的订阅，你需要提供订阅 ID (**-SubscriptionId**) 或订阅名称 (**-SubscriptionName**)。你可以从前面的步骤中复制该信息，或者，如果你有多个订阅，你可以运行 **Get-AzureRmSubscription** cmdlet，然后从结果集中复制所需的订阅信息。

使用你的订阅信息运行以下 cmdlet，以设置当前订阅：

	Select-AzureRmSubscription -SubscriptionId 4cac86b0-1e56-bbbb-aaaa-000000000000

针对前面刚刚选择的订阅运行以下命令。

## 获取建议

若要获取有关服务器升级的建议，请运行以下 cmdlet：

    $hint = Get-AzureRmSqlServerUpgradeHint -ResourceGroupName “resourcegroup1” -ServerName “server1” 


## 开始升级

若要开始服务器升级，请运行以下 cmdlet：

    Start-AzureRmSqlServerUpgrade -ResourceGroupName “resourcegroup1” -ServerName “server1” -ServerVersion 12.0 -DatabaseCollection $hint.Databases -ElasticPoolCollection $hint.ElasticPools  


运行此命令时，升级过程将开始。你可以自定义建议的输出并将已编辑的建议提供给此 cmdlet 。


## 升级服务器


    # Adding the account
    #
    Add-AzureRmAccount -EnvironmentName AzureChinaCloud
    
    # Setting the variables
    #
    $SubscriptionName = 'YOUR_SUBSCRIPTION' 
    $ResourceGroupName = 'YOUR_RESOURCE_GROUP' 
    $ServerName = 'YOUR_SERVER' 
    
    # Selecting the right subscription 
    # 
    Select-AzureRmSubscription -SubscriptionName $SubscriptionName 
    
    # Getting the upgrade recommendations 
    #
    $hint = Get-AzureRmSqlServerUpgradeHint -ResourceGroupName $ResourceGroupName -ServerName $ServerName 
    
    # Starting the upgrade process 
    #
    Start-AzureRmSqlServerUpgrade -ResourceGroupName $ResourceGroupName -ServerName $ServerName -ServerVersion 12.0 -DatabaseCollection $hint.Databases -ElasticPoolCollection $hint.ElasticPools  


## 自定义升级映射

如果建议不适合于你的服务器和业务案例，你可以选择数据库的升级方式并可以将它们映射到单一或弹性数据库。

ElasticPoolCollection 和 DatabaseCollection 参数是可选的：
    
    # Creating elastic pool mapping
    #
    $elasticPool = New-Object -TypeName Microsoft.Azure.Management.Sql.Models.UpgradeRecommendedElasticPoolProperties 
    $elasticPool.DatabaseDtuMax = 100 
    $elasticPool.DatabaseDtuMin = 0 
    $elasticPool.Dtu = 800 
    $elasticPool.Edition = "Standard" 
    $elasticPool.DatabaseCollection = ("DB1", “DB2”, “DB3”, “DB4”) 
    $elasticPool.Name = "elasticpool_1" 
    $elasticPool.StorageMb = 800 
     
    # Creating single database mapping for 2 databases. DBMain1 mapped to S0 and DBMain2 mapped to S2
    #
    $databaseMap1 = New-Object -TypeName Microsoft.Azure.Management.Sql.Models.UpgradeDatabaseProperties 
    $databaseMap1.Name = "DBMain1" 
    $databaseMap1.TargetEdition = "Standard" 
    $databaseMap1.TargetServiceLevelObjective = "S0"
    
    $databaseMap2 = New-Object -TypeName Microsoft.Azure.Management.Sql.Models.UpgradeDatabaseProperties 
    $databaseMap2.Name = "DBMain2" 
    $databaseMap2.TargetEdition = "Standard" 
    $databaseMap2.TargetServiceLevelObjective = "S2"
     
    # Starting the upgrade
    #
    Start-AzureRmSqlServerUpgrade –ResourceGroupName resourcegroup1 –ServerName server1 -Version 12.0 -DatabaseCollection @($databaseMap1, $databaseMap2) -ElasticPoolCollection @($elasticPool) 

    

## 升级到 SQL 数据库 V12 后监视数据库


升级后，建议你主动监视数据库，以确保应用程序以所需的性能运行，并根据需要优化使用方式。

除了监视单个数据库之外，你还可以通过 [PowerShell](/documentation/articles/sql-database-elastic-pool-powershell/#monitoring-elastic-databases-and-elastic-database-pools) 监视弹性数据库池。


**资源消耗数据：**对于基本、标准和高级数据库，可通过用户数据库中的 [sys.dm\_ db\_ resource\_stats](http://msdn.microsoft.com/zh-cn/library/azure/dn800981.aspx) DMV 查看资源消耗数据。此 DMV 针对前一小时的操作，以 15 秒的粒度级提供接近实时的资源消耗信息。每个间隔的 DTU 消耗百分比将计算为 CPU、IO 和日志维度的最大消耗百分比。下面是一个用于计算过去一小时平均 DTU 消耗百分比的查询：

    SELECT end_time
    	 , (SELECT Max(v)
             FROM (VALUES (avg_cpu_percent)
                         , (avg_data_io_percent)
                         , (avg_log_write_percent)
    	   ) AS value(v)) AS [avg_DTU_percent]
    FROM sys.dm_db_resource_stats
    ORDER BY end_time DESC;

其他监视信息：

- [Azure SQL 数据库的单一数据库性能指导](http://msdn.microsoft.com/zh-cn/library/azure/dn369873.aspx)。
- [弹性数据库池的价格和性能注意事项](/documentation/articles/sql-database=elastic-pool-guidance/)。
- [使用动态管理视图监视 Azure SQL 数据库](/documentation/articles/sql-database-monitoring-with-dmvs/)



**警报：**在 Azure 门户中设置“警报”可在升级后的数据库 DTU 消耗量接近特定的高位时接收通知。你可以针对 DTU、CPU、IO 和日志等各种性能度量值，在 Azure 门户中设置数据库警报。浏览到你的数据库，然后在“设置”边栏选项卡中选择“警报规则”。

例如，你可以针对“DTU 百分比”设置电子邮件警报，以便在过去 5 分钟平均 DTU 百分比值超过 75% 时发出警报。请参阅[接收警报通知](/documentation/articles/insights-receive-alert-notifications/)，以了解有关如何配置警报通知的详细信息。



## 后续步骤

- 创建弹性数据库池并将部分或全部数据库添加到该池。


## 相关信息

- [Get-AzureRmSqlServerUpgrade](https://msdn.microsoft.com/zh-cn/library/azure/mt603582.aspx)
- [Start-AzureRmSqlServerUpgrade](https://msdn.microsoft.com/zh-cn/library/azure/mt619403.aspx)
- [Stop-AzureRmSqlServerUpgrade](https://msdn.microsoft.com/zh-cn/library/azure/mt603589.aspx)

<!---HONumber=Mooncake_0118_2016-->
