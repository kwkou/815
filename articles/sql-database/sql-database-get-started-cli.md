<properties
    pageTitle="Azure CLI：创建 SQL 数据库 | Azure"
    description="了解如何使用 Azure CLI 创建 SQL 数据库逻辑服务器、服务器级防火墙规则和数据库。"
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
    ms.devlang="azurecli"
    ms.topic="hero-article"
    ms.date="04/04/2017"
    wacn.date="05/22/2017"
    ms.author="carlrab"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="8fd60f0e1095add1bff99de28a0b65a8662ce661"
    ms.openlocfilehash="4cdc67c73fff8d85ed7383b35d3f26b2581a7e99"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/12/2017" />

# <a name="create-a-single-azure-sql-database-using-the-azure-cli"></a>使用 Azure CLI 创建单一 Azure SQL 数据库

Azure CLI 用于从命令行或脚本创建和管理 Azure 资源。 本指南详述了如何使用 Azure CLI 在 [Azure 资源组](/documentation/articles/resource-group-overview/)的 [Azure SQL 数据库逻辑服务器](/documentation/articles/sql-database-features/)中部署 Azure SQL 数据库。

若要完成本快速入门教程，请确保已安装最新的 [Azure CLI 2.0](https://docs.microsoft.com/zh-cn/cli/azure/install-azure-cli)。 

如果没有 Azure 订阅，可在开始前创建一个[试用](/pricing/1rmb-trial/)帐户。

## <a name="log-in-to-azure"></a>登录 Azure

使用 [az login](https://docs.microsoft.com/zh-cn/cli/azure/#login) 命令登录到 Azure 订阅，并按照屏幕上的说明进行操作。

    az login -e azurechinacloud

## <a name="create-a-resource-group"></a>创建资源组

使用 [az group create](https://docs.microsoft.com/zh-cn/cli/azure/group#create) 命令创建 [Azure 资源组](/documentation/articles/resource-group-overview/)。 资源组是在其中以组的形式部署和管理 Azure 资源的逻辑容器。 以下示例在 `chinaeast` 位置创建名为 `myResourceGroup` 的资源组。

    az group create --name myResourceGroup --location chinaeast

## <a name="create-a-logical-server"></a>创建逻辑服务器

使用 [az sql server create](https://docs.microsoft.com/zh-cn/cli/azure/sql/server#create) 命令创建 [Azure SQL 数据库逻辑服务器](/documentation/articles/sql-database-features/)。 逻辑服务器包含一组作为组管理的数据库。 以下示例使用管理员用户名 `ServerAdmin` 和密码 `ChangeYourAdminPassword1` 在资源组中创建随机命名的服务器。 根据需要替换这些预定义的值。

    servername=server-$RANDOM
    az sql server create --name $servername --resource-group myResourceGroup --location chinaeast \
        --admin-user ServerAdmin --admin-password ChangeYourAdminPassword1

## <a name="configure-a-server-firewall-rule"></a>配置服务器防火墙规则

使用 [az sql server firewall create](https://docs.microsoft.com/zh-cn/cli/azure/sql/server/firewall-rule#create) 命令创建 [Azure SQL 数据库服务器级防火墙规则](/documentation/articles/sql-database-firewall-configure/)。 服务器级防火墙规则允许外部服务器（例如 SQL Server Management Studio 或 SQLCMD 实用程序）通过 SQL 数据库服务防火墙连接到 SQL 数据库。 在以下示例中，防火墙仅对其他 Azure 资源开放。 若要启用外部连接，请将 IP 地址更改为适合环境的地址。 若要开放所有 IP 地址，请使用 0.0.0.0 作为起始 IP 地址，使用 255.255.255.255 作为结束地址。  

    az sql server firewall-rule create --resource-group myResourceGroup --server $servername \
        -n AllowYourIp --start-ip-address 0.0.0.0 --end-ip-address 0.0.0.0

> [AZURE.NOTE]
> 通过端口 1433 进行 SQL 数据库通信。 如果尝试从企业网络内部进行连接，则该网络的防火墙可能不允许经端口 1433 的出站流量。 如果是这样，则无法连接到 Azure SQL 数据库服务器，除非 IT 部门打开了端口 1433。
>

## <a name="create-a-database-in-the-server-with-sample-data"></a>使用示例数据在服务器中创建数据库

使用 [az sql db create](https://docs.microsoft.com/zh-cn/cli/azure/sql/db#create) 命令在服务器中创建 [S0 性能级别](/documentation/articles/sql-database-service-tiers/)的数据库。 以下示例创建名为 `mySampleDatabase` 的数据库，并将 AdventureWorksLT 示例数据加载到该数据库中。 根据需要替换这些预定义的值（此集合中的其他快速入门基于此快速入门中的值）。

    az sql db create --resource-group myResourceGroup --server $servername \
        --name mySampleDatabase --sample-name AdventureWorksLT --service-objective S0

## <a name="clean-up-resources"></a>清理资源

本教程系列中的其他快速入门教程是在本文的基础上制作的。 如果计划继续使用后续的快速入门或相关教程，请勿清除在本快速入门中创建的资源。 如果不打算继续，请使用以下命令删除通过本快速入门创建的所有资源。

    az group delete --name myResourceGroup

## <a name="next-steps"></a>后续步骤

- 若要使用 SQL Server Management Studio 进行连接和查询，请参阅[使用 SSMS 进行连接和查询](/documentation/articles/sql-database-connect-query-ssms/)
- 若要使用 Visual Studio Code 进行连接和查询，请参阅[使用 Visual Studio Code 进行连接和查询](/documentation/articles/sql-database-connect-query-vscode/)。
- 若要使用 .NET 进行连接和查询，请参阅[使用 .NET 进行连接和查询](/documentation/articles/sql-database-connect-query-dotnet/)。
- 若要使用 PHP 进行连接和查询，请参阅[使用 PHP 进行连接和查询](/documentation/articles/sql-database-connect-query-php/)。
- 若要使用 Node.js 进行连接和查询，请参阅[使用 Node.js 进行连接和查询](/documentation/articles/sql-database-connect-query-nodejs/)。
- 若要使用 Java 进行连接和查询，请参阅[使用 Java 进行连接和查询](/documentation/articles/sql-database-connect-query-java/)。
- 若要使用 Python 进行连接和查询，请参阅[使用 Python 进行连接和查询](/documentation/articles/sql-database-connect-query-python/)。
- 若要使用 Ruby 进行连接和查询，请参阅[使用 Ruby 进行连接和查询](/documentation/articles/sql-database-connect-query-ruby/)。
<!--Update_Description:update "配置服务器防火墙规则"-->