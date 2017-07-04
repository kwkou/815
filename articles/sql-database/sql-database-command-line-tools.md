<properties
	pageTitle="使用 PowerShell 管理 Azure SQL 数据库 | Azure"
	description="使用 PowerShell 管理 Azure SQL 数据库。"
	services="sql-database"
	documentationCenter=""
	authors="stevestein"
	manager="jhubbard"
	editor="monicar"/>

<tags
	ms.service="sql-database"
	ms.workload="data-management"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="09/13/2016"
	wacn.date="12/19/2016"
	ms.author="sstein"/>  


# 使用 PowerShell 管理 Azure SQL 数据库


> [AZURE.SELECTOR]
- [Transact-SQL (SSMS)](/documentation/articles/sql-database-manage-azure-ssms/)
- [PowerShell](/documentation/articles/sql-database-command-line-tools/)

本主题介绍用于执行许多 Azure SQL 数据库任务的 PowerShell cmdlet。有关完整列表，请参阅 [Azure SQL 数据库 Cmdlet](https://msdn.microsoft.com/zh-cn/library/mt574084.aspx)。


## 创建资源组

使用 [New-AzureRmResourceGroup](https://msdn.microsoft.com/zh-cn/library/azure/mt759837.aspx) cmdlet 为 SQL 数据库和相关的 Azure 资源创建资源组。


	$resourceGroupName = "resourcegroup1"
	$resourceGroupLocation = "China East"
	New-AzureRmResourceGroup -Name $resourceGroupName -Location $resourceGroupLocation


有关详细信息，请参阅[将 Azure PowerShell 与 Azure Resource Manager 配合使用](/documentation/articles/powershell-azure-resource-manager/)。有关示例脚本，请参阅[创建 SQL 数据库 PowerShell 脚本](/documentation/articles/sql-database-get-started-powershell/#create-a-sql-database-powershell-script)。

## 创建 SQL 数据库服务器

使用 [New-AzureRmSqlServer](https://msdn.microsoft.com/zh-cn/library/azure/mt603715.aspx) cmdlet 创建 SQL 数据库服务器。将 *server1* 替换为服务器的名称。服务器名称必须在所有 Azure SQL 数据库服务器中都是唯一的。如果服务器名称已使用，则会出现错误。此命令可能需要几分钟才能完成。资源组必须已存在于订阅中。


	$resourceGroupName = "resourcegroup1"

	$sqlServerName = "server1"
	$sqlServerVersion = "12.0"
	$sqlServerLocation = "China East"
	$serverAdmin = "loginname"
	$serverPassword = "password" 
	$securePassword = ConvertTo-SecureString –String $serverPassword –AsPlainText -Force
	$creds = New-Object –TypeName System.Management.Automation.PSCredential –ArgumentList $serverAdmin, $securePassword
    

	$sqlServer = New-AzureRmSqlServer -ServerName $sqlServerName `
	 -SqlAdministratorCredentials $creds -Location $sqlServerLocation `
	 -ResourceGroupName $resourceGroupName -ServerVersion $sqlServerVersion


有关详细信息，请参阅[什么是 SQL 数据库](/documentation/articles/sql-database-technical-overview/)。有关示例脚本，请参阅[创建 SQL 数据库 PowerShell 脚本](/documentation/articles/sql-database-get-started-powershell/#create-a-sql-database-powershell-script)。


## 创建 SQL 数据库服务器防火墙规则

使用 [New-AzureRmSqlServerFirewallRule](https://msdn.microsoft.com/zh-cn/library/azure/mt603860.aspx) cmdlet 创建访问服务器的防火墙规则。运行以下命令，将开始和结尾的 IP 地址替换为客户端的有效值。资源组和服务器必须已存在于订阅中。


	$resourceGroupName = "resourcegroup1"
	$sqlServerName = "server1"

	$firewallRuleName = "firewallrule1"
	$firewallStartIp = "0.0.0.0"
	$firewallEndIp = "255.255.255.255"

	New-AzureRmSqlServerFirewallRule -ResourceGroupName $resourceGroupName `
	 -ServerName $sqlServerName -FirewallRuleName $firewallRuleName `
	 -StartIpAddress $firewallStartIp -EndIpAddress $firewallEndIp


若要允许其他 Azure 服务访问服务器，请创建防火墙规则，将 `-StartIpAddress` 和 `-EndIpAddress` 都设置为 **0.0.0.0**。这个特殊的防火墙规则允许所有 Azure 流量访问服务器。

有关详细信息，请参阅 [Azure SQL 数据库防火墙](/documentation/articles/sql-database-firewall-configure/)。有关示例脚本，请参阅[创建 SQL 数据库 PowerShell 脚本](/documentation/articles/sql-database-get-started-powershell/#create-a-sql-database-powershell-script)。


## 创建 SQL 数据库（空）

使用 [New-AzureRmSqlDatabase](https://msdn.microsoft.com/zh-cn/library/azure/mt619339.aspx) cmdlet 创建数据库。资源组和服务器必须已存在于订阅中。


	$resourceGroupName = "resourcegroup1"
	$sqlServerName = "server1"

	$databaseName = "database1"
	$databaseEdition = "Standard"
	$databaseServiceLevel = "S0"

	$currentDatabase = New-AzureRmSqlDatabase -ResourceGroupName $resourceGroupName `
	 -ServerName $sqlServerName -DatabaseName $databaseName `
	 -Edition $databaseEdition -RequestedServiceObjectiveName $databaseServiceLevel


有关详细信息，请参阅[什么是 SQL 数据库](/documentation/articles/sql-database-technical-overview/)。有关示例脚本，请参阅[创建 SQL 数据库 PowerShell 脚本](/documentation/articles/sql-database-get-started-powershell/#create-a-sql-database-powershell-script)。


## 更改 SQL 数据库的性能级别

使用 [Set-AzureRmSqlDatabase](https://msdn.microsoft.com/zh-cn/library/azure/mt619433.aspx) cmdlet 增加或减少数据库。资源组、服务器和数据库必须已存在于订阅中。对于“基本”层，请将 `-RequestedServiceObjectiveName` 设置为单个空格（如以下代码段）。将其设置为 *S0*、*S1*、*P1*、*P6* 等，如其他层的前述示例。


	$resourceGroupName = "resourcegroup1"
	$sqlServerName = "server1"

	$databaseName = "database1"
	$databaseEdition = "Basic"
	$databaseServiceLevel = " "

	Set-AzureRmSqlDatabase -ResourceGroupName $resourceGroupName `
	 -ServerName $sqlServerName -DatabaseName $databaseName `
	 -Edition $databaseEdition -RequestedServiceObjectiveName $databaseServiceLevel


有关详细信息，请参阅 [SQL 数据库选项和性能：了解每个服务层提供的功能](/documentation/articles/sql-database-service-tiers/)。有关示例脚本，请参阅[用于更改 SQL 数据库的服务层和性能级别的示例 PowerShell 脚本](/documentation/articles/sql-database-scale-up-powershell/#sample-powershell-script-to-change-the-service-tier-and-performance-level-of-your-sql-database)。

## 将 SQL 数据库复制到同一台服务器

使用 [New-AzureRmSqlDatabaseCopy](https://msdn.microsoft.com/zh-cn/library/azure/mt603644.aspx) cmdlet 将 SQL 数据库复制到同一台服务器。将 `-CopyServerName` 和 `-CopyResourceGroupName` 设置为与源数据库服务器和资源组相同的值。


	$resourceGroupName = "resourcegroup1"
	$sqlServerName = "server1"
	$databaseName = "database1"

	$copyDatabaseName = "database1_copy"
	$copyServerName = $sqlServerName
	$copyResourceGroupName = $resourceGroupName

	New-AzureRmSqlDatabaseCopy -DatabaseName $databaseName `
	 -ServerName $sqlServerName -ResourceGroupName $resourceGroupName `
	 -CopyDatabaseName $copyDatabaseName -CopyServerName $sqlServerName `
	 -CopyResourceGroupName $copyResourceGroupName


有关详细信息，请参阅[复制 Azure SQL 数据库](/documentation/articles/sql-database-copy/)。有关示例脚本，请参阅[复制 SQL 数据库 PowerShell 脚本](/documentation/articles/sql-database-copy-powershell/#example-powershell-script)。


## 删除 SQL 数据库

使用 [Remove-AzureRmSqlDatabase](https://msdn.microsoft.com/zh-cn/library/azure/mt619368.aspx) cmdlet 删除 SQL 数据库。资源组、服务器和数据库必须已存在于订阅中。


	$resourceGroupName = "resourcegroup1"
	$sqlServerName = "server1"
	$databaseName = "database1"

	Remove-AzureRmSqlDatabase -DatabaseName $databaseName `
	 -ServerName $sqlServerName -ResourceGroupName $resourceGroupName


## 删除 SQL 数据库服务器

使用 [Remove-AzureRmSqlServer](https://msdn.microsoft.com/zh-cn/library/azure/mt603488.aspx) cmdlet 删除服务器。


	$resourceGroupName = "resourcegroup1"
	$sqlServerName = "server1"

	Remove-AzureRmSqlServer -ServerName $sqlServerName -ResourceGroupName $resourceGroupName


## 使用 PowerShell 创建和管理弹性数据库池

有关使用 PowerShell 创建弹性数据库池的详细信息，请参阅[使用 PowerShell 创建新的弹性数据库池](/documentation/articles/sql-database-elastic-pool-create-powershell/)。

有关使用 PowerShell 管理弹性数据库池的详细信息，请参阅[使用 PowerShell 监视和管理弹性数据库池](/documentation/articles/sql-database-elastic-pool-manage-powershell/)。



## 相关信息

- [Azure SQL 数据库 Cmdlet](https://msdn.microsoft.com/zh-cn/library/azure/mt574084.aspx)
- [Azure Cmdlet 参考](https://msdn.microsoft.com/zh-cn/library/azure/dn708514.aspx)

<!---HONumber=Mooncake_Quality_Review_1202_2016-->