<properties
    pageTitle="Azure PowerShell：创建 SQL 数据库 | Azure"
    description="了解如何在 Azure 门户预览版中创建 SQL 数据库逻辑服务器、服务器级防火墙规则和数据库。"
    keywords="SQL 数据库教程：创建 SQL 数据库"
    services="sql-database"
    documentationcenter=""
    author="CarlRabeler"
    manager="jhubbard"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid=""
    ms.service="sql-database"
    ms.custom="quick start"
    ms.workload="data-management"
    ms.tgt_pltfrm="na"
    ms.devlang="PowerShell"
    ms.topic="hero-article"
    ms.date="03/13/2017"
    wacn.date="04/17/2017"
    ms.author="carlrab"
    ms.sourcegitcommit="7cc8d7b9c616d399509cd9dbdd155b0e9a7987a8"
    ms.openlocfilehash="ef48b526b708fdcb2614268cff531de4959095ec"
    ms.lasthandoff="04/07/2017" />

# <a name="create-a-single-azure-sql-database-using-powershell"></a>使用 PowerShell 创建单一 Azure SQL 数据库

PowerShell 用于从命令行或脚本创建和管理 Azure 资源。 本指南详述了如何使用 PowerShell 在 [Azure 资源组](/documentation/articles/resource-group-overview/)的 [Azure SQL 数据库逻辑服务器](/documentation/articles/sql-database-features/)中部署 Azure SQL 数据库。

在开始之前，请确保安装 PowerShell 的最新版本。 有关详细信息，请参阅 [如何安装和配置 Azure PowerShell](https://docs.microsoft.com/zh-cn/powershell/azureps-cmdlets-docs)。 

## <a name="log-in-to-azure"></a>登录 Azure

使用 [Add-AzureRmAccount](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.profile/v2.5.0/add-azurermaccount) 命令登录到 Azure 订阅，并按照屏幕上的说明进行操作。

    Add-AzureRmAccount -EnvironmentName AzureChinaCloud

## <a name="create-a-resource-group"></a>创建资源组

使用 [New-AzureRmResourceGroup](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.resources/v3.5.0/new-azurermresourcegroup) 命令创建 [Azure 资源组](/documentation/articles/resource-group-overview/)。 资源组是在其中以组的形式部署和管理 Azure 资源的逻辑容器。 以下示例在 `China East` 位置创建名为 `myResourceGroup` 的资源组。

    New-AzureRmResourceGroup -Name "myResourceGroup" -Location "China East"

## <a name="create-a-logical-server"></a>创建逻辑服务器

使用 [New-AzureRmSqlServer](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.sql/v2.5.0/new-azurermsqlserver) 命令创建 [Azure SQL 数据库逻辑服务器](/documentation/articles/sql-database-features/)。 逻辑服务器包含一组作为组管理的数据库。 以下示例使用管理员用户名 `ServerAdmin` 和密码 `ChangeYourAdminPassword1` 在资源组中创建随机命名的服务器。 根据需要替换这些预定义的值。

    $servername = "server-$(Get-Random)"
    New-AzureRmSqlServer -ResourceGroupName "myResourceGroup" `
        -ServerName $servername `
        -Location "China East" `
        -SqlAdministratorCredentials $(New-Object -TypeName System.Management.Automation.PSCredential -ArgumentList "ServerAdmin", $(ConvertTo-SecureString -String "ChangeYourAdminPassword1" -AsPlainText -Force))

## <a name="configure-a-server-firewall-rule"></a>配置服务器防火墙规则

使用 [New-AzureRmSqlServerFirewallRule](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.sql/v2.5.0/new-azurermsqlserverfirewallrule) 命令创建 [Azure SQL 数据库服务器级防火墙规则](/documentation/articles/sql-database-firewall-configure/)。 服务器级防火墙规则允许外部服务器（例如 SQL Server Management Studio 或 SQLCMD 实用程序）通过 SQL 数据库服务防火墙连接到 SQL 数据库。 以下示例为预定义的地址范围创建了防火墙规则，该地址范围在本示例中是整个可能的 IP 地址范围。 将这些预定义的值替换为外部 IP 地址或 IP 地址范围的值。 

    New-AzureRmSqlServerFirewallRule -ResourceGroupName "myResourceGroup" `
        -ServerName $servername `
        -FirewallRuleName "AllowSome" -StartIpAddress "0.0.0.0" -EndIpAddress "255.255.255.255"

## <a name="create-a-blank-database"></a>创建空数据库

使用 [New-AzureRmSqlDatabase](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.sql/v2.5.0/new-azurermsqldatabase) 命令在服务器中创建 [S0 性能级别](/documentation/articles/sql-database-service-tiers/)的空 SQL 数据库。 以下示例创建一个名为 `mySampleDatabase` 的数据库。 根据需要替换此预定义的值。

    New-AzureRmSqlDatabase  -ResourceGroupName "myResourceGroup" `
        -ServerName $servername `
        -DatabaseName "MySampleDatabase" `
        -RequestedServiceObjectiveName "S0"

## <a name="clean-up-resources"></a>清理资源

此集合中的“连接方式”快速入门以及教程集合中的教程以此快速入门为基础。 如果计划继续使用后续的快速入门或相关教程，请勿清除在本快速入门中创建的资源。 如果不打算继续，请使用以下命令删除通过本快速入门创建的所有资源。

    Remove-AzureRmResourceGroup -ResourceGroupName "myResourceGroup"

## <a name="next-steps"></a>后续步骤

- 若要使用 SQL Server Management Studio 进行连接和查询，请参阅[使用 SSMS 进行连接和查询](/documentation/articles/sql-database-connect-query-ssms/)
- 若要使用 Visual Studio 进行连接，请参阅[使用 Visual Studio 进行连接和查询](/documentation/articles/sql-database-connect-query/)。
* 有关 SQL 数据库的技术概述，请参阅[关于 SQL 数据库服务](/documentation/articles/sql-database-technical-overview/)。
<!--Update_Description: refine content structure, simplify powershell commands for each operation-->