<properties
    pageTitle="Azure CLI 脚本 - 创建 SQL 数据库 | Azure"
    description="Azure CLI 脚本示例 - 使用 Azure CLI 创建 SQL 数据库"
    services="sql-database"
    documentationcenter="sql-database"
    author="janeng"
    manager="jstrauss"
    editor="carlrab"
    tags="azure-service-management"
    translationtype="Human Translation" />
<tags
    ms.assetid=""
    ms.service="sql-database"
    ms.custom="sample"
    ms.devlang="CLI"
    ms.topic="article"
    ms.tgt_pltfrm="sql-database"
    ms.workload="database"
    ms.date="03/16/2017"
    wacn.date="05/08/2017"
    ms.author="janeng"
    ms.sourcegitcommit="7cc8d7b9c616d399509cd9dbdd155b0e9a7987a8"
    ms.openlocfilehash="eb513c8801d974078fc6e290a58c04c6e93a43c4"
    ms.lasthandoff="04/07/2017" />

# <a name="create-a-single-sql-database-and-configure-a-firewall-rule-using-the-azure-cli"></a>使用 Azure CLI 创建单个 SQL 数据库并配置防火墙规则

此示例 CLI 脚本创建 Azure SQL 数据库，并配置服务器级防火墙规则。 成功运行该脚本后，可以通过所有 Azure 服务和配置的 IP 地址访问 SQL 数据库。 

必要时，请使用 [Azure CLI 安装指南](https://docs.microsoft.com/zh-cn/cli/azure/install-azure-cli)中的说明安装 Azure CLI，然后运行 `az login` 创建与 Azure 的连接。

此示例在 Bash shell 中正常工作。 有关在 Windows 上运行 Azure CLI 脚本的选项，请参阅[在 Windows 中运行 Azure CLI](/documentation/articles/virtual-machines-windows-cli-options/)。

## <a name="sample-script"></a>示例脚本

    #!/bin/bash

    # Set an admin login and password for your database
    adminlogin=ServerAdmin
    password=ChangeYourAdminPassword1
    # The logical server name has to be unique in the system
    servername=server-$RANDOM
    # The ip address range that you want to allow to access your DB
    startip=0.0.0.0
    endip=0.0.0.0

    # Create a resource group
    az group create \
        --name myResourceGroup \
        --location "China East"

    # Create a logical server in the resource group
    az sql server create \
        --name $servername \
        --resource-group myResourceGroup \
        --location "China East"  \
        --admin-user $adminlogin \
        --admin-password $password

    # Configure a firewall rule for the server
    az sql server firewall-rule create \
        --resource-group myResourceGroup \
        --server $servername \
        -n AllowYourIp \
        --start-ip-address $startip \
        --end-ip-address $endip

    # Create a database in the server
    az sql db create \
        --resource-group myResourceGroup \
        --server $servername \
        --name mySampleDatabase \
        --sample-name AdventureWorksLT \
        --service-objective S0

## <a name="clean-up-deployment"></a>清理部署

运行脚本示例后，可以使用以下命令删除资源组以及与其关联的所有资源。

    az group delete --name myResourceGroup

## <a name="script-explanation"></a>脚本说明

此脚本使用以下命令。 表中的每条命令均链接到特定于命令的文档。

| 命令 | 说明 |
|---|---|
| [az group create](https://docs.microsoft.com/cli/azure/group#create) | 创建用于存储所有资源的资源组。 |
| [az sql server create](https://docs.microsoft.com/cli/azure/sql/server#create) | 创建用于托管 SQL 数据库的逻辑服务器。 |
| [az sql server firewall create](https://docs.microsoft.com/cli/azure/sql/server/firewall#create) | 创建一个防火墙规则，以允许从输入的 IP 地址范围访问服务器上的所有 SQL 数据库。 |
| [az sql db create](https://docs.microsoft.com/cli/azure/sql/db#create) | 在逻辑服务器中创建 SQL 数据库。 |
| [az group delete](https://docs.microsoft.com/cli/azure/resource#delete) | 删除资源组，包括所有嵌套的资源。 |

## <a name="next-steps"></a>后续步骤

有关 Azure CLI 的详细信息，请参阅 [Azure CLI 文档](https://docs.microsoft.com/zh-cn/cli/azure/overview)。

其他 SQL 数据库 CLI 脚本示例可以在 [Azure SQL 数据库文档](/documentation/articles/sql-database-cli-samples/)中找到。