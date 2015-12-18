<properties 
	pageTitle="使用 PowerShell 将 Azure SQL 服务器升级到 V12" 
	description="使用 PowerShell 将 Azure SQL 服务器升级到 V12。" 
	services="sql-database" 
	documentationCenter="" 
	authors="stevestein" 
	manager="jeffreyg" 
	editor=""/>

<tags 
	ms.service="sql-database" 
	ms.date="10/08/2015" 
	wacn.date="11/27/2015"/>

# 使用 PowerShell 升级到 SQL 数据库 V12


> [AZURE.SELECTOR]
- [PowerShell](/documentation/articles/sql-database-upgrade-server)


本文介绍如何使用 PowerShell 升级到 SQL 数据库 V12。

在升级到 SQL 数据库 V12 的过程中，还必须将所有 Web 和企业数据库更新到新的服务层。下面说明了如何借助定价层和弹性池建议在服务器上[更新任何 Web 和企业数据库](/documentation/articles/sql-database-upgrade-new-service-tiers)。


## 先决条件 

若要使用 PowerShell 将服务器升级到 V12，你需要安装并运行 Azure PowerShell，然后，根据具体的版本，你可能需要切换到资源管理器模式，以访问 Azure 资源管理器 PowerShell cmdlet。

> [AZURE.IMPORTANT]从 Azure PowerShell 1.0 预览版开始，Switch-AzureMode cmdlet 不再可用，并且 Azure ResourceManger 模块中的 cmdlet 已重命名。本文中的示例使用新的 PowerShell 1.0 预览版命名约定。有关详细信息，请参阅[弃用 Azure PowerShell 中的 Switch-AzureMode](https://github.com/Azure/azure-powershell/wiki/Deprecation-of-Switch-AzureMode-in-Azure-PowerShell)。

若要运行 PowerShell cmdlet，需要安装并运行 Azure PowerShell，而且由于 Switch-AzureMode 已删除，你应该通过运行 [Microsoft Web 平台安装程序](http://go.microsoft.com/fwlink/p/?linkid=320376&clcid=0x409)来下载并安装最新的 Azure PowerShell。有关详细信息，请参阅[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure)。


## 配置你的凭据，然后选择你的订阅

若要针对 Azure 订阅运行 PowerShell cmdlet，必须先与 Azure 帐户建立访问连接。运行以下项目，然后就会出现一个要求你输入凭据的登录屏幕。使用登录 Azure 门户时所用的相同电子邮件和密码。

	Add-AzureAccount -Environment AzureChinaCloud

成功登录后，你会在屏幕上看到一些信息，其中包括你登录时使用的 ID，以及你有权访问的 Azure 订阅。

若要选择要使用的订阅，你需要提供订阅 ID (**-SubscriptionId**) 或订阅名称 (**-SubscriptionName**)。你可以从前面的步骤中复制该信息，或者，如果你有多个订阅，你可以运行 **Get-AzureSubscription** cmdlet，然后从结果集中复制所需的订阅信息。

使用你的订阅信息运行以下 cmdlet，以设置当前订阅：

	Select-AzureSubscription -SubscriptionId 4cac86b0-1e56-bbbb-aaaa-000000000000

针对前面刚刚选择的订阅运行以下命令。

## 获取建议

若要获取有关服务器升级的建议，请运行以下 cmdlet：

    $hint = Get-AzureRMSqlServerUpgradeHint -ResourceGroupName “resourcegroup1” -ServerName “server1” 

有关详细信息，请参阅 [Azure SQL 数据库弹性数据库池建议](/documentation/articles/sql-database-elastic-pool-portal/#elastic-database-pool-pricing-tier-recommendations)。



## 开始升级

若要开始服务器升级，请运行以下 cmdlet：

    Start-AzureRMSqlServerUpgrade -ResourceGroupName “resourcegroup1” -ServerName “server1” -ServerVersion 12.0 -DatabaseCollection $hint.Databases -ElasticPoolCollection $hint.ElasticPools  


运行此命令时，升级过程将开始。你可以自定义建议的输出并将已编辑的建议提供给此 cmdlet 。


## 升级服务器


    # Adding the account
    #
    Add-AzureAccount -Environment AzureChinaCloud
    
    # Setting the variables
    #
    $SubscriptionName = 'YOUR_SUBSCRIPTION' 
    $ResourceGroupName = 'YOUR_RESOURCE_GROUP' 
    $ServerName = 'YOUR_SERVER' 
    
    # Selecting the right subscription 
    # 
    Select-AzureSubscription -SubscriptionName $SubscriptionName 
    
    # Getting the upgrade recommendations 
    #
    $hint = Get-AzureRMSqlServerUpgradeHint -ResourceGroupName $ResourceGroupName -ServerName $ServerName 
    
    # Starting the upgrade process 
    #
    Start-AzureRMSqlServerUpgrade -ResourceGroupName $ResourceGroupName -ServerName $ServerName -ServerVersion 12.0 -DatabaseCollection $hint.Databases -ElasticPoolCollection $hint.ElasticPools  


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
    Start-AzureRMSqlServerUpgrade –ResourceGroupName resourcegroup1 –ServerName server1 -Version 12.0 -DatabaseCollection @($databaseMap1, $databaseMap2) -ElasticPoolCollection @($elasticPool) 

    




## 相关信息

- [Get-AzureRMSqlServerUpgrade](https://msdn.microsoft.com/zh-cn/library/azure/mt603582.aspx)
- [Start-AzureRMSqlServerUpgrade](https://msdn.microsoft.com/zh-cn/library/azure/mt619403.aspx)
- [Stop-AzureRMSqlServerUpgrade](https://msdn.microsoft.com/zh-cn/library/azure/mt603589.aspx)

<!---HONumber=82-->