<properties
    pageTitle="使用 PowerShell 管理 Azure SQL 数据库 | Azure"
    description="使用 PowerShell 管理 Azure SQL 数据库。"
    services="sql-database"
    documentationcenter=""
    author="stevestein"
    manager="jhubbard"
    editor="monicar" />
<tags
    ms.assetid="3f21ad5e-ba99-4010-b244-5e5815074d31"
    ms.service="sql-database"
    ms.custom="overview"
    ms.workload="data-management"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="02/09/2017"
    wacn.date="03/24/2017"
    ms.author="sstein" />  


# 使用 PowerShell 管理 Azure SQL 数据库

本主题介绍用于执行许多 Azure SQL 数据库任务的 PowerShell cmdlet。有关完整列表，请参阅 [Azure SQL 数据库 Cmdlet](https://msdn.microsoft.com/zh-cn/library/mt574084(v=azure.300).aspx)。

> [AZURE.TIP]
>有关演示如何创建服务器、创建基于服务器的防火墙、查看服务器属性、连接和查询 master 数据库、创建示例数据库和空白数据库、查询数据库属性、连接和查询示例数据库的教程，请参阅[入门教程](/documentation/articles/sql-database-get-started-powershell/)。
>

## 如何创建资源组？
若要为 SQL 数据库和相关的 Azure 资源创建资源组，请使用 [New-AzureRmResourceGroup](https://msdn.microsoft.com/zh-cn/library/azure/mt759837(v=azure.300).aspx) cmdlet 。

```
$resourceGroupName = "resourcegroup1"
$resourceGroupLocation = "China East"
New-AzureRmResourceGroup -Name $resourceGroupName -Location $resourceGroupLocation
```

有关详细信息，请参阅[将 Azure PowerShell 与 Azure Resource Manager 配合使用](/documentation/articles/powershell-azure-resource-manager/)。如需完整教程，请参阅 [Azure SQL 数据库服务器、数据库和防火墙规则入门（使用 Azure PowerShell）](/documentation/articles/sql-database-get-started-powershell/)。

## 如何创建 SQL 数据库服务器？
若要创建 SQL 数据库服务器，请使用 [New-AzureRmSqlServer](https://docs.microsoft.com/powershell/resourcemanager/azurerm.sql/v2.5.0/new-azurermsqlserver) cmdlet。将 *server1* 替换为服务器的名称。服务器名称必须在所有 Azure SQL 数据库服务器中都是唯一的。如果服务器名称已使用，将出现错误。此命令可能需要几分钟才能完成。资源组必须已存在于订阅中。


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


有关服务器的详细信息，请参阅 [SQL 数据库功能](/documentation/articles/sql-database-features/)。如需完整教程，请参阅 [Azure SQL 数据库服务器、数据库和防火墙规则入门（使用 Azure PowerShell）](/documentation/articles/sql-database-get-started-powershell/)。

## 如何创建 SQL 数据库服务器防火墙规则？
若要创建防火墙规则以访问服务器，请使用 [New-AzureRmSqlServerFirewallRule](https://msdn.microsoft.com/zh-cn/library/azure/mt603860(v=azure.300).aspx) cmdlet。运行以下命令，将开始和结束 IP 地址替换为客户端的有效值。资源组和服务器必须已存在于订阅中。


	$resourceGroupName = "resourcegroup1"
	$sqlServerName = "server1"

	$firewallRuleName = "firewallrule1"
	$firewallStartIp = "0.0.0.0"
	$firewallEndIp = "255.255.255.255"

	New-AzureRmSqlServerFirewallRule -ResourceGroupName $resourceGroupName `
	 -ServerName $sqlServerName -FirewallRuleName $firewallRuleName `
	 -StartIpAddress $firewallStartIp -EndIpAddress $firewallEndIp


若要允许其他 Azure 服务访问服务器，请创建防火墙规则，将 `-StartIpAddress` 和 `-EndIpAddress` 都设置为 **0.0.0.0**。这个特殊的防火墙规则允许所有 Azure 流量访问服务器。

有关详细信息，请参阅 [Azure SQL 数据库防火墙](/documentation/articles/sql-database-firewall-configure/)。如需完整教程，请参阅 [Azure SQL 数据库服务器、数据库和防火墙规则入门（使用 Azure PowerShell）](/documentation/articles/sql-database-get-started-powershell/)。

## 如何创建 SQL 数据库？
若要创建 SQL 数据库，请使用 [New-AzureRmSqlDatabase](https://msdn.microsoft.com/zh-cn/library/azure/mt619339(v=azure.300).aspx) cmdlet。资源组和服务器必须已存在于订阅中。


	$resourceGroupName = "resourcegroup1"
	$sqlServerName = "server1"

	$databaseName = "database1"
	$databaseEdition = "Standard"
	$databaseServiceLevel = "S0"

	$currentDatabase = New-AzureRmSqlDatabase -ResourceGroupName $resourceGroupName `
	 -ServerName $sqlServerName -DatabaseName $databaseName `
	 -Edition $databaseEdition -RequestedServiceObjectiveName $databaseServiceLevel


有关详细信息，请参阅[什么是 SQL 数据库](/documentation/articles/sql-database-technical-overview/)。如需完整教程，请参阅 [Azure SQL 数据库服务器、数据库和防火墙规则入门（使用 Azure PowerShell）](/documentation/articles/sql-database-get-started-powershell/)。

## 如何更改 SQL 数据库的性能级别？
若要更改性能级别，请使用 [Set-AzureRmSqlDatabase](https://msdn.microsoft.com/zh-cn/library/azure/mt619433(v=azure.300).aspx) cmdlet 扩展或缩减数据库。资源组、服务器和数据库必须已存在于订阅中。对于“基本”层，请将 `-RequestedServiceObjectiveName` 设置为单个空格（如以下代码段）。将其设置为 *S0*、*S1*、*P1*、*P6* 等，如其他层的前述示例。


	$resourceGroupName = "resourcegroup1"
	$sqlServerName = "server1"

	$databaseName = "database1"
	$databaseEdition = "Basic"
	$databaseServiceLevel = " "

	Set-AzureRmSqlDatabase -ResourceGroupName $resourceGroupName `
	 -ServerName $sqlServerName -DatabaseName $databaseName `
	 -Edition $databaseEdition -RequestedServiceObjectiveName $databaseServiceLevel


有关详细信息，请参阅 [SQL 数据库选项和性能：了解每个服务层提供的功能](/documentation/articles/sql-database-service-tiers/)。有关示例脚本，请参阅[用于更改 SQL 数据库的服务层和性能级别的示例 PowerShell 脚本](/documentation/articles/sql-database-manage-single-databases-powershell/#change-the-service-tier-and-performance-level-of-a-single-database)。

## 如何将 SQL 数据库复制到同一台服务器？
若要将 SQL 数据库复制到同一台服务器，请使用 [New-AzureRmSqlDatabaseCopy](https://msdn.microsoft.com/zh-cn/library/azure/mt603644(v=azure.300).aspx) cmdlet。将 `-CopyServerName` 和 `-CopyResourceGroupName` 设置为与源数据库服务器和资源组相同的值。


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

## 如何删除 SQL 数据库？
若要删除 SQL 数据库，请使用 [Remove-AzureRmSqlDatabase](https://msdn.microsoft.com/zh-cn/library/azure/mt619368(v=azure.300).aspx) cmdlet。资源组、服务器和数据库必须已存在于订阅中。


	$resourceGroupName = "resourcegroup1"
	$sqlServerName = "server1"
	$databaseName = "database1"

	Remove-AzureRmSqlDatabase -DatabaseName $databaseName `
	 -ServerName $sqlServerName -ResourceGroupName $resourceGroupName


## 如何删除 SQL 数据库服务器？
若要删除 SQL 数据库服务器，请使用 [Remove-AzureRmSqlServer](https://msdn.microsoft.com/zh-cn/library/azure/mt603488(v=azure.300).aspx) cmdlet。


	$resourceGroupName = "resourcegroup1"
	$sqlServerName = "server1"

	Remove-AzureRmSqlServer -ServerName $sqlServerName -ResourceGroupName $resourceGroupName


## 如何使用 PowerShell 创建和管理弹性池？
有关使用 PowerShell 创建弹性池的详细信息，请参阅[使用 PowerShell 创建新的弹性池](/documentation/articles/sql-database-elastic-pool-manage-powershell/)。

有关使用 PowerShell 管理弹性池的详细信息，请参阅[使用 PowerShell 监视和管理弹性池](/documentation/articles/sql-database-elastic-pool-manage-powershell/)。

## 相关信息
- [Azure SQL 数据库 Cmdlet](https://msdn.microsoft.com/zh-cn/library/azure/mt574084(v=azure.300).aspx)
- [Azure Cmdlet 参考](https://msdn.microsoft.com/zh-cn/library/azure/dn708514(v=azure.300).aspx)

<!---HONumber=Mooncake_0320_2017-->
<!--Update_Description: add tip for SQL powershell get start; link references update-->