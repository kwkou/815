<properties
    pageTitle="Azure CLI 脚本 - 移动 SQL 数据库和弹性池 | Azure"
    description="Azure CLI 脚本示例 - 使用 Azure CLI 在弹性池之间移动 SQL 数据库"
    services="sql-database"
    documentationcenter="sql-database"
    author="janeng"
    manager="jstrauss"
    editor="carlrab"
    tags="azure-service-management" />
<tags
    ms.assetid=""
    ms.service="sql-database"
    ms.custom="sample"
    ms.devlang="azurecli"
    ms.topic="article"
    ms.tgt_pltfrm="sql-database"
    ms.workload="database"
    ms.date="04/04/2017"
    wacn.date="05/22/2017"
    ms.author="janeng"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="8fd60f0e1095add1bff99de28a0b65a8662ce661"
    ms.openlocfilehash="15b8fd5fc1753828fa596e062b791108a26c8db4"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/12/2017" />

# <a name="create-elastic-pools-and-move-databases-between-pools-and-out-of-a-pool-using-the-azure-cli"></a>使用 Azure CLI 创建弹性池，并在池之间和池外移动数据库

此示例 CLI 脚本创建两个弹性池，将数据库从一个弹性池移到另一个弹性池中，然后将数据库移出弹性池，移到单一数据库性能级别。 

必要时，请使用 [Azure CLI 安装指南](https://docs.microsoft.com/zh-cn/cli/azure/install-azure-cli)中的说明安装 Azure CLI，然后运行 `az login` 创建与 Azure 的连接。

>[AZURE.NOTE]
>在运行 `az login` 前，请先打开 Azure CLI 2.0 配置文件，路径为 C:\\Users\\<\%USERPROFILE\%\>\\.azure\\config, 确保 cloud name 的值为 AzureChinaCloud
><p>```
>[cloud]
>name = AzureChinaCloud
>```</p>


此示例在 Bash shell 中正常工作。 有关在 Windows 上运行 Azure CLI 脚本的选项，请参阅[在 Windows 中运行 Azure CLI](/documentation/articles/virtual-machines-windows-cli-options/)。

## <a name="sample-script"></a>示例脚本

    #!/bin/bash

    # Set an admin login and password for your database
    adminlogin=ServerAdmin
    password=ChangeYourAdminPassword1
    # The logical server name has to be unique in the system
    servername=server-$RANDOM

    # Create a resource group
    az group create \
        --name myResourceGroup \
        --location "China East" 

    # Create a logical server in the resource group
    az sql server create \
        --name $servername \
        --resource-group myResourceGroup \
        --location "China East" \
        --admin-user $adminlogin \
        --admin-password $password

    # Create two pools in the logical server
    az sql elastic-pools create \
        --resource-group myResourceGroup \
        --location "China East"  \
        --server $servername \
        --name myFirstPool \
        --dtu 50 \
        --database-dtu-max 20
    az sql elastic-pools create \
        --resource-group myResourceGroup \
        --location "China East"  \
        --server $servername \
        --name MySecondPool \
        --dtu 50 \
        --database-dtu-max 50

    # Create a database in the first pool
    az sql db create \
        --resource-group myResourceGroup \
        --server $servername \
        --name mySampleDatabase \
        --elastic-pool-name myFirstPool

    # Move the database to the second pool - create command updates the db if it exists
    az sql db create \
        --resource-group myResourceGroup \
        --server-name $servername \
        --name mySampleDatabase \
        --elastic-pool-name mySecondPool

    # Move the database to standalone S1 performance level
    az sql db create \
        --resource-group myResourceGroup \
        --server $servername \
        --name mySampleDatabase \
        --service-objective S1

## <a name="clean-up-deployment"></a>清理部署

运行脚本示例后，可以使用以下命令删除资源组以及与其关联的所有资源。

    az group delete --name myResourceGroup

## <a name="script-explanation"></a>脚本说明

此脚本使用以下命令。 表中的每条命令均链接到特定于命令的文档。

| 命令 | 说明 |
|---|---|
| [az group create](https://docs.microsoft.com/zh-cn/cli/azure/group#create) | 创建用于存储所有资源的资源组。 |
| [az sql server create](https://docs.microsoft.com/zh-cn/cli/azure/sql/server#create) | 创建用于托管数据库或弹性池的逻辑服务器。 |
| [az sql elastic-pools create](https://docs.microsoft.com/zh-cn/cli/azure/sql/elastic-pools#create) | 在逻辑服务器中创建弹性池。 |
| [az sql db create](https://docs.microsoft.com/zh-cn/cli/azure/sql/db#create) | 在逻辑服务器中创建数据库作为单一数据库或入池数据库。 |
| [az sql db update](https://docs.microsoft.com/zh-cn/cli/azure/sql/db#update) | 更新数据库属性，或者将数据库移入、移出弹性池或在弹性池之间移动。 |
| [az group delete](https://docs.microsoft.com/zh-cn/cli/azure/vm/extension#set) | 删除资源组，包括所有嵌套的资源。 |

## <a name="next-steps"></a>后续步骤

有关 Azure CLI 的详细信息，请参阅 [Azure CLI 文档](https://docs.microsoft.com/zh-cn/cli/azure/overview)。

其他 SQL 数据库 CLI 脚本示例可以在 [Azure SQL 数据库文档](/documentation/articles/sql-database-cli-samples/)中找到。