<properties 
    pageTitle="创建弹性数据库池 (PowerShell) | Azure" 
    description="了解如何通过创建弹性数据库池，使用 PowerShell 向外缩放 Azure SQL 数据库资源以管理多个数据库。" 
    services="sql-database" 
    documentationCenter="" 
    authors="stevestein" 
    manager="jhubbard" 
    editor=""/>

<tags
    ms.service="sql-database"
    ms.date="03/15/2016"
    wacn.date="04/06/2016"/>

# 创建弹性数据库池 (PowerShell) 

> [AZURE.SELECTOR]
- [Azure 门户](/documentation/articles/sql-database-elastic-pool-create-portal)
- [PowerShell](/documentation/articles/sql-database-elastic-pool-create-powershell)
- [C#](/documentation/articles/sql-database-elastic-pool-create-csharp)


了解如何使用 PowerShell cmdlet 创建[弹性数据库池](/documentation/articles/sql-database-elastic-pool)。

> [AZURE.NOTE] 弹性数据库池目前为预览版，仅适用于 SQL 数据库 V12 服务器。如果你有一个 SQL 数据库 V11 服务器，可以通过一个步骤[使用 PowerShell 升级到 V12 并创建池](/documentation/articles/sql-database-upgrade-server-portal)。


你需要运行 Azure PowerShell 1.0 或更高版本。有关详细信息，请参阅[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure)。



## 创建弹性数据库池

[New-AzureRmSqlElasticPool](https://msdn.microsoft.com/zh-cn/library/azure/mt619378.aspx) cmdlet 将创建弹性数据库池。

	New-AzureRmSqlElasticPool -ResourceGroupName "resourcegroup1" -ServerName "server1" -ElasticPoolName "elasticpool1" -Edition "Standard" -Dtu 400 -DatabaseDtuMin 10 -DatabaseDtuMax 100


## 在弹性数据库池中创建新的弹性数据库

若要直接在池中创建新的数据库，请使用 [New-AzureRmSqlDatabase](https://msdn.microsoft.com/zh-cn/library/azure/mt619339.aspx) cmdlet 并设置 **ElasticPoolName** 参数。


	New-AzureRmSqlDatabase -ResourceGroupName "resourcegroup1" -ServerName "server1" -DatabaseName "database1" -ElasticPoolName "elasticpool1"



## 将独立的数据库移动到弹性数据库池

若要将现有数据库移到池中，请使用 [Set-AzureRmSqlDatabase](https://msdn.microsoft.com/zh-cn/library/azure/mt619433.aspx) cmdlet 并设置 **ElasticPoolName** 参数。

	Set-AzureRmSqlDatabase -ResourceGroupName "resourcegroup1" -ServerName "server1" -DatabaseName "database1" -ElasticPoolName "elasticpool1"



## 创建弹性数据库池 PowerShell 示例

此脚本将创建新的服务器，当系统提示你输入用户名和密码时，请输入你的新服务器（不是你的 Azure 凭据）的管理员登录名和管理员密码。

    $subscriptionId = '<your Azure subscription id>'
    $resourceGroupName = '<resource group name>'
    $location = '<datacenter location>'
    $serverName = '<server name>'
    $poolName = '<pool name>'
    $databaseName = '<database name>'

    Login-AzureRmAccount
    Set-AzureRmContext -SubscriptionId $subscriptionId
    
    New-AzureRmResourceGroup -Name $resourceGroupName -Location $location
    New-AzureRmSqlServer -ResourceGroupName $resourceGroupName -ServerName $serverName -Location $location -ServerVersion "12.0"
    New-AzureRmSqlServerFirewallRule -ResourceGroupName $resourceGroupName -ServerName $serverName -FirewallRuleName "rule1" -StartIpAddress "192.168.0.198" -EndIpAddress "192.168.0.199"

    New-AzureRmSqlElasticPool -ResourceGroupName $resourceGroupName -ServerName $serverName -ElasticPoolName $poolName -Edition "Standard" -Dtu 400 -DatabaseDtuMin 10 -DatabaseDtuMax 100

    New-AzureRmSqlDatabase -ResourceGroupName $resourceGroupName -ServerName $serverName -DatabaseName $databaseName -ElasticPoolName $poolName -MaxSizeBytes 10GB



## 后续步骤

- [管理你的池](/documentation/articles/sql-database-elastic-pool-manage-powershell)
- [创建弹性作业](/documentation/articles/sql-database-elastic-jobs-overview)弹性作业可以用来根据池中数据库的数目来运行 T-SQL 脚本。


## 弹性数据库参考

有关弹性数据库和弹性数据库池的详细信息，请参阅[弹性数据库池参考](/documentation/articles/sql-database-elastic-pool-reference)。

<!---HONumber=Mooncake_0328_2016-->
