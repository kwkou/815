<properties
    pageTitle="Azure CLI：创建 SQL 数据库 | Azure"
    description="了解如何使用 Azure CLI 创建 SQL 数据库逻辑服务器、服务器级防火墙规则和数据库。"
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
    ms.devlang="cli"
    ms.topic="hero-article"
    ms.date="03/13/2017"
    wacn.date="04/17/2017"
    ms.author="carlrab"
    ms.sourcegitcommit="7cc8d7b9c616d399509cd9dbdd155b0e9a7987a8"
    ms.openlocfilehash="4a3bc5b6c1351bb52c47b751e5196dc53ec86fb6"
    ms.lasthandoff="04/07/2017" />

# <a name="create-a-single-azure-sql-database-using-the-azure-cli"></a>使用 Azure CLI 创建单一 Azure SQL 数据库

Azure CLI 用于从命令行或脚本创建和管理 Azure 资源。 本指南详述了如何使用 Azure CLI 在 [Azure 资源组](/documentation/articles/resource-group-overview/)的 [Azure SQL 数据库逻辑服务器](/documentation/articles/sql-database-features/)中部署 Azure SQL 数据库。

在开始之前，请确保已安装 Azure CLI。 有关详细信息，请参阅 [Azure CLI 安装指南](https://docs.microsoft.com/zh-cn/cli/azure/install-azure-cli)。 

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

使用 [az sql server firewall create](https://docs.microsoft.com/zh-cn/cli/azure/sql/server/firewall#create) 命令创建 [Azure SQL 数据库服务器级防火墙规则](/documentation/articles/sql-database-firewall-configure/)。 服务器级防火墙规则允许外部服务器（例如 SQL Server Management Studio 或 SQLCMD 实用程序）通过 SQL 数据库服务防火墙连接到 SQL 数据库。 以下示例为预定义的地址范围创建了防火墙规则，该地址范围在本示例中是整个可能的 IP 地址范围。 将这些预定义的值替换为外部 IP 地址或 IP 地址范围的值。 

    az sql server firewall-rule create --resource-group myResourceGroup --server $servername \
        -n AllowYourIp --start-ip-address 0.0.0.0 --end-ip-address 255.255.255.255

## <a name="create-a-database-in-the-server"></a>在服务器中创建数据库

使用 [az sql db create](https://docs.microsoft.com/zh-cn/cli/azure/sql/db#create) 命令在服务器中创建 [S0 性能级别](/documentation/articles/sql-database-service-tiers/)的数据库。 以下示例创建一个空的名为 `mySampleDatabase` 的数据库。 根据需要替换此预定义的值。

    az sql db create --resource-group myResourceGroup --server $servername \
        --name mySampleDatabase --service-objective S0

## <a name="clean-up-resources"></a>清理资源

此集合中的“连接方式”快速入门以及教程集合中的教程以此快速入门为基础。 如果计划继续使用后续的快速入门或相关教程，请勿清除在本快速入门中创建的资源。 如果不打算继续，请使用以下命令删除通过本快速入门创建的所有资源。

    az group delete --name myResourceGroup

## <a name="next-steps"></a>后续步骤

- 若要使用 SQL Server Management Studio 进行连接和查询，请参阅[使用 SSMS 进行连接和查询](/documentation/articles/sql-database-connect-query-ssms/)
- 若要使用 Visual Studio 进行连接，请参阅[使用 Visual Studio 进行连接和查询](/documentation/articles/sql-database-connect-query/)。
- 有关 SQL 数据库的技术概述，请参阅[关于 SQL 数据库服务](/documentation/articles/sql-database-technical-overview/)。