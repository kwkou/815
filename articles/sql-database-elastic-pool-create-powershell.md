<properties
    pageTitle="使用 PowerShell 创建新的弹性数据库池 | Azure"
    description="了解如何通过创建可缩放的弹性数据库池，使用 PowerShell 向外缩放 Azure SQL 数据库资源以管理多个数据库。"
	services="sql-database"
    documentationCenter=""
    authors="sidneyh"
    manager="jhubbard"
    editor=""/>

<tags
    ms.service="sql-database"
    ms.date="04/28/2016"
    wacn.date="06/14/2016"/>

# 使用 PowerShell 创建新的弹性数据库池

> [AZURE.SELECTOR]
- [PowerShell](/documentation/articles/sql-database-elastic-pool-create-powershell/)
- [C#](/documentation/articles/sql-database-elastic-pool-create-csharp/)


了解如何使用 PowerShell cmdlet 创建[弹性数据库池](/documentation/articles/sql-database-elastic-pool/)。

有关常见的错误代码，请参阅 [SQL 数据库客户端应用程序的 SQL 错误代码：数据库连接错误和其他问题](/documentation/articles/sql-database-develop-error-messages/)。

> [AZURE.NOTE] 弹性数据库池目前为预览版，仅适用于 SQL 数据库 V12 服务器。如果你有一个 SQL 数据库 V11 服务器，可以通过一个步骤使用 PowerShell 升级到 V12 并创建池。


你需要运行 Azure PowerShell 1.0 或更高版本。有关详细信息，请参阅[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/)。

## 创建新池

[New-AzureRmSqlElasticPool](https://msdn.microsoft.com/zh-cn/library/azure/mt619378.aspx) cmdlet 将创建新池。每个池的 eDTU 值、最小和最大 DTU 受服务器层值（基本、标准或高级）的约束。请参阅[弹性池和弹性数据库的 eDTU 和存储限制](/documentation/articles/sql-database-elastic-pool/#eDTU-and-storage-limits-for-elastic-pools-and-elastic-databases)。

	New-AzureRmSqlElasticPool -ResourceGroupName "resourcegroup1" -ServerName "server1" -ElasticPoolName "elasticpool1" -Edition "Standard" -Dtu 400 -DatabaseDtuMin 10 -DatabaseDtuMax 100


## 在池中创建新的弹性数据库

使用 [New-AzureRmSqlDatabase](https://msdn.microsoft.com/zh-cn/library/azure/mt619339.aspx) cmdlet 并将 **ElasticPoolName** 参数设置为目标池。要将现有数据库移动到池，请参阅[将数据库移动到弹性池](/documentation/articles/sql-database-elastic-pool-manage-powershell/#Move-a-database-into-an-elastic-pool)。

	New-AzureRmSqlDatabase -ResourceGroupName "resourcegroup1" -ServerName "server1" -DatabaseName "database1" -ElasticPoolName "elasticpool1"

## 创建池，并使用多个新数据库对其进行填充 

使用门户或每次只能创建一个单一数据库的 PowerShell cmdlet 在池中创建大量数据库可能需要一段时间。若要自动创建到新池中，请参阅 [CreateOrUpdateElasticPoolAndPopulate](https://gist.github.com/billgib/d80c7687b17355d3c2ec8042323819ae)。

## 示例：使用 PowerShell 创建池 

此脚本创建新的 Azure 资源组和新的服务器。出现提示时，提供新服务器的管理员用户名和密码（不是你的 Azure 凭据）。

    $subscriptionId = '<your Azure subscription id>'
    $resourceGroupName = '<resource group name>'
    $location = '<datacenter location>'
    $serverName = '<server name>'
    $poolName = '<pool name>'
    $databaseName = '<database name>'

    Login-AzureRmAccount -EnvironmentName AzureChinaCloud
    Set-AzureRmContext -SubscriptionId $subscriptionId

    New-AzureRmResourceGroup -Name $resourceGroupName -Location $location
    New-AzureRmSqlServer -ResourceGroupName $resourceGroupName -ServerName $serverName -Location $location -ServerVersion "12.0"
    New-AzureRmSqlServerFirewallRule -ResourceGroupName $resourceGroupName -ServerName $serverName -FirewallRuleName "rule1" -StartIpAddress "192.168.0.198" -EndIpAddress "192.168.0.199"

    New-AzureRmSqlElasticPool -ResourceGroupName $resourceGroupName -ServerName $serverName -ElasticPoolName $poolName -Edition "Standard" -Dtu 400 -DatabaseDtuMin 10 -DatabaseDtuMax 100

    New-AzureRmSqlDatabase -ResourceGroupName $resourceGroupName -ServerName $serverName -DatabaseName $databaseName -ElasticPoolName $poolName -MaxSizeBytes 10GB



## 后续步骤

- [管理你的池](/documentation/articles/sql-database-elastic-pool-manage-powershell/)
- [创建弹性作业](/documentation/articles/sql-database-elastic-jobs-overview/)弹性作业可以根据池中数据库的数量来运行 T-SQL 脚本。
- [使用 Azure SQL 数据库扩展](/documentation/articles/sql-database-elastic-scale-introduction/)：使用弹性数据库工具扩展。


<!---HONumber=Mooncake_0530_2016-->
