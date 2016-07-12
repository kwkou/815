<properties 
    pageTitle="使用 PowerShell 新建 SQL 数据库设置 | Azure" 
    description="了解如何使用 PowerShell 创建新的 SQL 数据库。可以通过 PowerShell cmdlet 管理常见的数据库设置任务。" 
    keywords="新建 sql 数据库,数据库设置"
	services="sql-database" 
    documentationCenter="" 
    authors="stevestein" 
    manager="jeffreyg" 
    editor="cgronlun"/>

<tags
    ms.service="sql-database"
    ms.date="01/20/2016"
    wacn.date="06/14/2016"/>

# 使用 PowerShell cmdlet 创建新的 SQL 数据库并执行常见的数据库设置任务 

**单一数据库**

> [AZURE.SELECTOR]
- [Azure 门户](/documentation/articles/sql-database-get-started/)
- [C#](/documentation/articles/sql-database-get-started-csharp/)
- [PowerShell](/documentation/articles/sql-database-get-started-powershell/)


了解如何使用 PowerShell cmdlet 创建新的 SQL 数据库并执行常见的数据库设置任务。


若要运行 PowerShell cmdlet，需要已安装并运行 Azure PowerShell。有关详细信息，请参阅[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/)。

- 如果你需要 Azure 订阅，只需单击本页顶部的“试用”，然后再回来完成本文的相关操作即可。


## 配置你的凭据，然后选择你的订阅

现在，你已经运行 Azure 资源管理器模块，因此可以访问创建 SQL 数据库所需的所有 cmdlet。

你必须先建立与 Azure 帐户的访问连接，才能运行以下 cmdlet，并且会出现一个要求你输入凭据的登录屏幕。使用登录 Azure 门户时所用的相同电子邮件和密码。

	Add-AzureRmAccount -EnvironmentName AzureChinaCloud

成功登录后，你会在屏幕上看到一些信息，其中包括你登录时使用的 ID，以及你有权访问的 Azure 订阅。


### 选择 Azure 订阅

若要选择订阅，你需要提供订阅 ID。你可以从前面的步骤中复制该信息，或者，如果你有多个订阅，你可以运行 **Get-AzureRmSubscription** cmdlet，然后从结果集中复制所需的订阅信息。获得订阅以后，你可以运行以下 cmdlet：

	Select-AzureRmSubscription -SubscriptionId 4cac86b0-1e56-bbbb-aaaa-000000000000

成功运行 **Select-AzureRmSubscription** 后，将返回到 PowerShell 提示符处。如果你有多个订阅，可以运行 **Get-AzureRmSubscription** 并验证要使用的订阅是否显示 **IsCurrent: True**。

## 数据库设置：创建资源组、服务器和防火墙规则

现在，你已经有了针对所选 Azure 订阅运行 cmdlet 所需的访问权限，因此下一步是建立一个资源组，使其中包含创建数据库所需的服务器。你可以编辑下一个命令，以便使用所选择的有效位置。运行 **(Get-AzureRmLocation | where-object {$\_.Name -eq "Microsoft.Sql/servers" }).Locations** 以获取有效位置的列表。

运行以下命令来创建新资源组：

	New-AzureRmResourceGroup -Name "resourcegroupsqlgsps" -Location "China North"

成功创建新资源组后，你会看到屏幕上显示的信息中包含 **ProvisioningState : Succeeded**。


### 创建服务器 

SQL 数据库在 Azure SQL 数据库服务器中创建。运行 **New-AzureRmSqlServer** 命令来创建新服务器。将 ServerName 替换为你的服务器的名称。该服务器名称对于所有 Azure SQL Server 必须是唯一的，因此如果名称已被使用，你会在此处收到一个错误。还必须指出的是，该命令可能需要数分钟才能运行完毕。你可以编辑该命令，以便使用所选择的任何有效位置，但你应使用在上一步中创建的资源组使用的相同位置。

	New-AzureRmSqlServer -ResourceGroupName "resourcegroupsqlgsps" -ServerName "server1" -Location "China North" -ServerVersion "12.0"

当你运行此命令时，会打开一个窗口，要求你提供**用户名**和**密码**。这不是你的 Azure 凭据，请输入用户名和密码，将其作为你要为新服务器创建的管理员凭据。

成功创建服务器后，会显示服务器详细信息。

### 配置服务器防火墙规则，允许对服务器进行访问

建立针对服务器访问的防火墙规则。运行以下命令，将开始和结尾的 IP 地址替换为你计算机的有效值。

	New-AzureRmSqlServerFirewallRule -ResourceGroupName "resourcegroupsqlgsps" -ServerName "server1" -FirewallRuleName "rule1" -StartIpAddress "192.168.0.0" -EndIpAddress "192.168.0.0"

成功创建规则后，会显示防火墙规则详细信息。

若要允许其他 Azure 服务访问该服务器，请添加一个防火墙规则并将 tartIpAddress 和 EndIpAddress 都设置为 0.0.0.0。请注意，这会允许来自任何 Azure 订阅的 Azure 流量访问该服务器。

有关详细信息，请参阅 [Azure SQL 数据库防火墙](/documentation/articles/sql-database-firewall-configure/)。


## 创建新的 SQL 数据库

现在，你已经有了一个资源组、一个服务器，并且配置了防火墙规则，因此可以访问服务器。

下面的命令会通过 S1 性能级别在标准服务层创建新的（空白）SQL 数据库：


	New-AzureRmSqlDatabase -ResourceGroupName "resourcegroupsqlgsps" -ServerName "server1" -DatabaseName "database1" -Edition "Standard" -RequestedServiceObjectiveName "S1"


成功创建数据库后，会显示数据库详细信息。

## 创建新的 SQL 数据库 PowerShell 脚本

    $SubscriptionId = "4cac86b0-1e56-bbbb-aaaa-000000000000"
    $ResourceGroupName = "resourcegroupname"
    $Location = "China East"
    
    $ServerName = "uniqueservername"
    
    $FirewallRuleName = "rule1"
    $FirewallStartIP = "192.168.0.0"
    $FirewallEndIp = "192.168.0.0"
    
    $DatabaseName = "database1"
    $DatabaseEdition = "Standard"
    $DatabasePerfomanceLevel = "S1"
    
    
    Add-AzureRmAccount -EnvironmentName AzureChinaCloud
    Select-AzureRmSubscription -SubscriptionId $SubscriptionId
    
    $ResourceGroup = New-AzureRmResourceGroup -Name $ResourceGroupName -Location $Location
    
    $Server = New-AzureRmSqlServer -ResourceGroupName $ResourceGroupName -ServerName $ServerName -Location $Location -ServerVersion "12.0"
    
    $FirewallRule = New-AzureRmSqlServerFirewallRule -ResourceGroupName $ResourceGroupName -ServerName $ServerName -FirewallRuleName $FirewallRuleName -StartIpAddress $FirewallStartIP -EndIpAddress $FirewallEndIp
    
    $SqlDatabase = New-AzureRmSqlDatabase -ResourceGroupName $ResourceGroupName -ServerName $ServerName -DatabaseName $DatabaseName -Edition $DatabaseEdition -RequestedServiceObjectiveName $DatabasePerfomanceLevel
    
    $SqlDatabase
    


## 后续步骤
创建新的 SQL 数据库并执行基本的数据库设置任务后，可以执行以下操作：

- [使用 SQL Server Management Studio 连接到 SQL 数据库并执行示例 T-SQL 查询](/documentation/articles/sql-database-connect-query-ssms/)


## 其他资源

- [Azure SQL 数据库](/documentation/services/sql-databases)

<!---HONumber=Mooncake_0606_2016-->
