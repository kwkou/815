<properties
    pageTitle="Azure PowerShell：创建 SQL 数据库 | Azure"
    description="了解如何在 Azure 门户预览中创建 SQL 数据库逻辑服务器、服务器级防火墙规则和数据库。"
    keywords="SQL 数据库教程：创建 SQL 数据库"
    services="sql-database"
    documentationcenter=""
    author="CarlRabeler"
    manager="jhubbard"
    editor="" />
<tags
    ms.assetid=""
    ms.service="sql-database"
    ms.custom="quick start create"
    ms.workload="data-management"
    ms.tgt_pltfrm="na"
    ms.devlang="PowerShell"
    ms.topic="hero-article"
    ms.date="04/03/2017"
    wacn.date="05/22/2017"
    ms.author="carlrab"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="8fd60f0e1095add1bff99de28a0b65a8662ce661"
    ms.openlocfilehash="77139324edd5eaef4db5b7dbc0de36623910ad56"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/12/2017" />

# <a name="create-a-single-azure-sql-database-using-powershell"></a>使用 PowerShell 创建单一 Azure SQL 数据库

PowerShell 用于从命令行或脚本创建和管理 Azure 资源。 本指南详述了如何使用 PowerShell 在 [Azure 资源组](/documentation/articles/resource-group-overview/)的 [Azure SQL 数据库逻辑服务器](/documentation/articles/sql-database-features/)中部署 Azure SQL 数据库。

若要完成本教程，请确保已安装最新的 [Azure PowerShell](https://docs.microsoft.com/zh-cn/powershell/azureps-cmdlets-docs)。 

如果没有 Azure 订阅，可在开始前创建一个[试用](/pricing/1rmb-trial/)帐户。

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

使用 [New-AzureRmSqlServerFirewallRule](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.sql/v2.5.0/new-azurermsqlserverfirewallrule) 命令创建 [Azure SQL 数据库服务器级防火墙规则](/documentation/articles/sql-database-firewall-configure/)。 服务器级防火墙规则允许外部服务器（例如 SQL Server Management Studio 或 SQLCMD 实用程序）通过 SQL 数据库服务防火墙连接到 SQL 数据库。 在以下示例中，防火墙仅对其他 Azure 资源开放。 若要启用外部连接，请将 IP 地址更改为适合环境的地址。 若要开放所有 IP 地址，请使用 0.0.0.0 作为起始 IP 地址，使用 255.255.255.255 作为结束地址。

    New-AzureRmSqlServerFirewallRule -ResourceGroupName "myResourceGroup" `
        -ServerName $servername `
        -FirewallRuleName "AllowSome" -StartIpAddress "0.0.0.0" -EndIpAddress "0.0.0.0"

> [AZURE.NOTE]
> 通过端口 1433 进行 SQL 数据库通信。 如果尝试从企业网络内部进行连接，则该网络的防火墙可能不允许经端口 1433 的出站流量。 如果是这样，则无法连接到 Azure SQL 数据库服务器，除非 IT 部门打开了端口 1433。
>

## <a name="create-a-blank-database"></a>创建空数据库

使用 [New-AzureRmSqlDatabase](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.sql/v2.5.0/new-azurermsqldatabase) 命令在服务器中创建 [S0 性能级别](/documentation/articles/sql-database-service-tiers/)的空 SQL 数据库。 以下示例创建一个名为 `mySampleDatabase` 的数据库。 根据需要替换此预定义的值。

    New-AzureRmSqlDatabase  -ResourceGroupName "myResourceGroup" `
        -ServerName $servername `
        -DatabaseName "MySampleDatabase" `
        -RequestedServiceObjectiveName "S0"

## <a name="clean-up-resources"></a>清理资源

本教程系列中的其他快速入门教程是在本文的基础上制作的。 如果计划继续使用后续的快速入门或相关教程，请勿清除在本快速入门中创建的资源。 如果不打算继续，请使用以下命令删除通过本快速入门创建的所有资源。

    Remove-AzureRmResourceGroup -ResourceGroupName "myResourceGroup"

## <a name="next-steps"></a>后续步骤

- 若要使用 SQL Server Management Studio 进行连接和查询，请参阅[使用 SSMS 进行连接和查询](/documentation/articles/sql-database-connect-query-ssms/)
- 若要使用 Visual Studio Code 进行连接和查询，请参阅[使用 Visual Studio Code 进行连接和查询](/documentation/articles/sql-database-connect-query-vscode/)。
- 若要使用 .NET 进行连接和查询，请参阅[使用 .NET 进行连接和查询](/documentation/articles/sql-database-connect-query-dotnet/)。
- 若要使用 PHP 进行连接和查询，请参阅[使用 PHP 进行连接和查询](/documentation/articles/sql-database-connect-query-php/)。
- 若要使用 Node.js 进行连接和查询，请参阅[使用 Node.js 进行连接和查询](/documentation/articles/sql-database-connect-query-nodejs/)。
- 若要使用 Java 进行连接和查询，请参阅[使用 Java 进行连接和查询](/documentation/articles/sql-database-connect-query-java/)。
- 若要使用 Python 进行连接和查询，请参阅[使用 Python 进行连接和查询](/documentation/articles/sql-database-connect-query-python/)。
- 若要使用 Ruby 进行连接和查询，请参阅[使用 Ruby 进行连接和查询](/documentation/articles/sql-database-connect-query-ruby/)。
<!--Update_Description:update "配置服务器防火墙规则";add next steps references links-->