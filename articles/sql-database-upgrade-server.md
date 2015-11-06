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
	ms.date="09/05/2015" 
	wacn.date="10/17/2015"/>

# 使用 PowerShell 将 Azure SQL 服务器升级到 V12
 

本文介绍如何使用定价层和弹性池建议将 SQL 数据库服务器升级到 V12。



## 先决条件 

若要使用 PowerShell 将服务器升级到 V12，你需要安装并运行 Azure PowerShell，然后，根据具体的版本，你可能需要切换到资源管理器模式，以访问 Azure 资源管理器 PowerShell cmdlet。

你可以通过运行 [Microsoft Web 平台安装程序](http://go.microsoft.com/fwlink/p/?linkid=320376&clcid=0x409)下载并安装 Azure PowerShell 模块。有关详细信息，请参阅[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure)。

用于创建和管理 Azure SQL 数据库的 cmdlet 位于 Azure 资源管理器模块中。启动 Azure PowerShell 时，默认情况下将导入 Azure 模块中的 cmdlet。若要切换到 Azure 资源管理器模块，请使用 **Switch-AzureMode** cmdlet。

	Switch-AzureMode -Name AzureResourceManager

如果你收到警告“Switch-AzureMode cmdlet 已过时，将在以后的版本中删除”， 你可以忽略该消息并转到下一部分。

有关详细信息，请参阅[将 Windows PowerShell 与资源管理器配合使用](/documentation/articles/powershell-azure-resource-manager)。



## 配置你的凭据

若要针对 Azure 订阅运行 PowerShell cmdlet，必须先与 Azure 帐户建立访问连接。运行以下项目，然后就会出现一个要求你输入凭据的登录屏幕。使用登录 Azure 门户时所用的相同电子邮件和密码。

	Add-AzureAccount -Environment AzureChinaCloud

成功登录后，你会在屏幕上看到一些信息，其中包括你登录时使用的 ID，以及你有权访问的 Azure 订阅。


## 选择 Azure 订阅

若要选择要使用的订阅，你需要提供订阅 ID (**-SubscriptionId**) 或订阅名称 (**-SubscriptionName**)。你可以从前面的步骤中复制该信息，或者，如果你有多个订阅，你可以运行 **Get-AzureSubscription** cmdlet，然后从结果集中复制所需的订阅信息。

使用你的订阅信息运行以下 cmdlet，以设置当前订阅：

	Select-AzureSubscription -SubscriptionId 4cac86b0-1e56-bbbb-aaaa-000000000000

针对前面刚刚选择的订阅运行以下命令。

## 获取建议

若要获取有关服务器升级的建议，请运行以下 cmdlet：

    $hint = Get-AzureSqlServerUpgradeHint -ResourceGroupName “resourcegroup1” -ServerName “server1” 

有关详细信息，请参阅 [Azure SQL 数据库弹性数据库池建议](/documentation/articles/sql-database-elastic-pool-portal#elastic-database-pool-pricing-tier-recommendations)和 [Azure SQL 数据库定价层建议](sql-database-service-tier-advisor)。



## 开始升级

若要开始服务器升级，请运行以下 cmdlet：

    Start-AzureSqlServerUpgrade -ResourceGroupName “resourcegroup1” -ServerName “server1” -ServerVersion 12.0 -DatabaseCollection $hint.Databases -ElasticPoolCollection $hint.ElasticPools  


运行此命令时，升级过程将开始。你可以自定义建议的输出并将已编辑的建议提供给此 cmdlet 。


## 升级 Azure SQL 服务器


    # Adding the account
    #
    Add-AzureAccount -Environment AzureChinaCloud
    
    # Switch mode
    #
    Switch-AzureMode -Name AzureResourceManager

    # Setting the variables
    #
    $SubscriptionName = 'YOUR_SUBSCRIPTION' 
    $ResourceGroupName = 'YOUR_RESOURCE_GROUP' 
    $ServerName = 'YOUR_SERVER' 
    
    # Selecting the right subscription 
    # 
    Select-AzureSubscription $SubscriptionName 
    
    # Getting the upgrade recommendations 
    #
    $hint = Get-AzureSqlServerUpgradeHint -ResourceGroupName $ResourceGroupName -ServerName $ServerName 
    
    # Starting the upgrade process 
    #
    Start-AzureSqlServerUpgrade -ResourceGroupName $ResourceGroupName -ServerName $ServerName -ServerVersion 12.0 -DatabaseCollection $hint.Databases -ElasticPoolCollection $hint.ElasticPools  


## 自定义升级映射

如果建议不适合于你的服务器和业务案例，你可以选择数据库的升级方式并可以将它们映射到单一或弹性数据库。

将数据库升级到弹性数据库池中：

    $elasticPool = New-Object -TypeName Microsoft.Azure.Management.Sql.Models.UpgradeRecommendedElasticPoolProperties
    $elasticPool.DatabaseDtuMax = 100  
    $elasticPool.DatabaseDtuMin = 0  
    $elasticPool.Dtu = 800
    $elasticPool.Edition = "Standard"  
    $elasticPool.DatabaseCollection = ("DB1")  
    $elasticPool.Name = "elasticpool_1"  


将数据库升级到单一数据库中：

    $databaseMap = New-Object -TypeName Microsoft.Azure.Management.Sql.Models.UpgradeDatabaseProperties  
    $databaseMap.Name = "DB2"
    $databaseMap.TargetEdition = "Standard"
    $databaseMap.TargetServiceLevelObjective = "S0"
    Start-AzureSqlServerUpgrade –ResourceGroupName resourcegroup1 –ServerName server1 -Version 12.0 -DatabaseCollection($databaseMap) -ElasticPoolCollection ($elasticPool)
    




## 相关信息

- [Azure SQL 数据库资源管理器 Cmdlet](https://msdn.microsoft.com/zh-cn/library/mt163521.aspx)
- [Azure SQL 数据库服务管理 Cmdlet](https://msdn.microsoft.com/zh-cn/library/dn546726.aspx)
 

<!---HONumber=74-->