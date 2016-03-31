<properties 
	pageTitle="使用 PowerShell 管理 Azure SQL 数据库" 
	description="使用 PowerShell 管理 Azure SQL 数据库。" 
	services="sql-database" 
	documentationCenter="" 
	authors="stevestein" 
	manager="jeffreyg" 
	editor="monicar"/>

<tags 
	ms.service="sql-database" 
	ms.date="12/01/2015" 
	wacn.date="01/15/2016"/>

# 使用 PowerShell 管理 Azure SQL 数据库


> [AZURE.SELECTOR]
- [Azure 门户](/documentation/articles/sql-database-manage-portal)
- [Transact-SQL (SSMS)](/documentation/articles/sql-database-manage-azure-ssms)
- [PowerShell](/documentation/articles/sql-database-command-line-tools)

本主题提供了用于执行许多 Azure SQL 数据库任务的 PowerShell 命令。


若要运行 PowerShell cmdlet，需要已安装并运行 Azure PowerShell。有关详细信息，请参阅[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure)。



## 配置你的凭据

若要针对 Azure 订阅运行 PowerShell cmdlet，必须先与 Azure 帐户建立访问连接。运行以下项目，然后就会出现一个要求你输入凭据的登录屏幕。使用登录 Azure 经典门户时所用的相同电子邮件和密码。

	Add-AzureAccount -Environment AzureChinaCloud

成功登录后，你会在屏幕上看到一些信息，其中包括你登录时使用的 ID，以及你有权访问的 Azure 订阅。


## 选择 Azure 订阅

若要选择要使用的订阅，你需要提供订阅 ID (**-SubscriptionId**) 或订阅名称 (**-SubscriptionName**)。你可以从前面的步骤中复制该信息，或者，如果你有多个订阅，你可以运行 **Get-AzureSubscription** cmdlet，然后从结果集中复制所需的订阅信息。

使用你的订阅信息运行以下 cmdlet，以设置当前订阅：

	Select-AzureRmSubscription -SubscriptionId 4cac86b0-1e56-bbbb-aaaa-000000000000

针对前面刚刚选择的订阅运行以下命令。

## 创建资源组

创建包含服务器的资源组。你可以编辑下一个命令，以使用任何有效位置。

如需有效 Azure SQL 数据库服务器位置的列表，请运行以下 cmdlet：

	$AzureSQLLocations = (Get-AzureRmResourceProvider -ListAvailable | Where-Object {$_.ProviderNamespace -eq 'Microsoft.Sql'}).Locations

如果你已有资源组，则可以直接创建服务器，或者编辑并运行以下命令，以创建新的资源组：

	New-AzureRmResourceGroup -Name "resourcegroupChinaEast" -Location "China East"

## 创建服务器 

若要创建新的 V12 服务器，请使用 [New-AzureRmSqlServer](https://msdn.microsoft.com/zh-cn/library/azure/mt603715.aspx) cmdlet。将 server12 替换为你的服务器名称。该服务器名称对于 Azure SQL Server 必须是唯一的，因此如果该服务器名称已被使用，你会在此处收到一个错误。还必须指出的是，该命令可能需要数分钟才能运行完毕。成功创建服务器后，会显示服务器详细信息和 PowerShell 提示。你可以编辑该命令，以使用任何有效位置。

	New-AzureRmSqlServer -ResourceGroupName "resourcegroupChinaEast" -ServerName "server12" -Location "China East" -ServerVersion "12.0"

当你运行此命令时，会打开一个窗口，要求你提供**用户名**和**密码**。这不是你的 Azure 凭据，请输入用户名和密码，将其作为你要为新服务器创建的管理员凭据。

## 创建服务器防火墙规则

若要创建防火墙规则以访问服务器，请使用 [New-AzureRmSqlServerFirewallRule](https://msdn.microsoft.com/zh-cn/library/azure/mt603860.aspx) 命令。运行以下命令，将开始和结尾的 IP 地址替换为你客户端的有效值。

如果你的服务器需要允许到其他 Azure 服务的访问，请添加 **-AllowAllAzureIPs** 开关，以便添加一个特殊的防火墙规则，允许到服务器的所有 Azure 流量访问。

	New-AzureRmSqlServerFirewallRule -ResourceGroupName "resourcegroupChinaEast" -ServerName "server12" -FirewallRuleName "clientFirewallRule1" -StartIpAddress "192.168.0.198" -EndIpAddress "192.168.0.199"

有关详细信息，请参阅 [Azure SQL 数据库防火墙](https://msdn.microsoft.com/zh-cn/library/azure/ee621782.aspx)。

## 创建 SQL 数据库

若要创建数据库，请使用 [New-AzureRmSqlDatabase](https://msdn.microsoft.com/zh-cn/library/azure/mt619339.aspx) 命令。你需要使用服务器来创建数据库。以下示例将创建名为 TestDB12 的 SQL 数据库。创建的数据库是一个标准 S1 数据库。

	New-AzureRmSqlDatabase -ResourceGroupName "resourcegroupChinaEast" -ServerName "server12" -DatabaseName "TestDB12" -Edition Standard -RequestedServiceObjectiveName "S1"


## 更改 SQL 数据库的性能级别

可以使用 [Set-AzureRmSqlDatabase](https://msdn.microsoft.com/zh-cn/library/azure/mt619433.aspx) 命令增加或减少数据库。以下示例将名为 TestDB12 的 SQL 数据库从其当前性能级别扩展为标准 S3 级别。

	Set-AzureRmSqlDatabase -ResourceGroupName "resourcegroupChinaEast" -ServerName "server12" -DatabaseName "TestDB12" -Edition Standard -RequestedServiceObjectiveName "S3"


## 删除 SQL 数据库

可以使用 [Remove-AzureRmSqlDatabase](https://msdn.microsoft.com/zh-cn/library/azure/mt619368.aspx) 命令删除 SQL 数据库。以下示例将删除名为 TestDB12 的 SQL 数据库。

	Remove-AzureRmSqlDatabase -ResourceGroupName "resourcegroupChinaEast" -ServerName "server12" -DatabaseName "TestDB12"

## 删除服务器

也可以使用 [Remove-AzureRmSqlServer](https://msdn.microsoft.com/zh-cn/library/azure/mt603488.aspx) 命令删除服务器。以下示例将删除名为 server12 的服务器。

	Remove-AzureRmSqlServer -ResourceGroupName "resourcegroupChinaEast" -ServerName "server12"



如果你要再次创建这些 Azure SQL 资源或类似资源，可以：

- 将这个命令集另存为 PowerShell 脚本文件 (*.ps1)
- 在 Azure 经典门户的“自动化”部分中，将这个命令集另存为 Azure 自动化 Runbook 

## 后续步骤

合并命令并自动化。例如，用你的值替换引号中的任何内容（包括 < and > 字符）以创建服务器、防火墙规则和数据库：


    New-AzureRmResourceGroup -Name "<resourceGroupName>" -Location "<Location>"
    New-AzureRmSqlServer -ResourceGroupName "<resourceGroupName>" -ServerName "<serverName>" -Location "<Location>" -ServerVersion "12.0"
    New-AzureRmSqlServerFirewallRule -ResourceGroupName "<resourceGroupName>" -ServerName "<serverName>" -FirewallRuleName "<firewallRuleName>" -StartIpAddress "<192.168.0.198>" -EndIpAddress "<192.168.0.199>"
    New-AzureRmSqlDatabase -ResourceGroupName "<resourceGroupName>" -ServerName "<serverName>" -DatabaseName "<databaseName>" -Edition <Standard> -RequestedServiceObjectiveName "<S1>"

## 相关信息

- [Azure SQL 数据库 Cmdlet](https://msdn.microsoft.com/zh-cn/library/azure/mt574084.aspx)

<!---HONumber=Mooncake_0104_2016-->
