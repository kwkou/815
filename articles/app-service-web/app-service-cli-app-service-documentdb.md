<properties
    pageTitle="Azure CLI 脚本示例 - 将 Web 应用连接到 documentdb | Azure"
    description="Azure CLI 脚本示例 - 将 Web 应用连接到 documentdb"
    services="appservice"
    documentationcenter="appservice"
    author="syntaxc4"
    manager="erikre"
    editor=""
    tags="azure-service-management"
    translationtype="Human Translation" />
<tags
    ms.assetid="bbbdbc42-efb5-4b4f-8ba6-c03c9d16a7ea"
    ms.service="app-service"
    ms.devlang="multiple"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="web"
    ms.date="03/20/2017"
    wacn.date="04/24/2017"
    ms.author="cfowler"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="94697f00dd82d87d1887b0eed1b88e6a0219e1f4"
    ms.lasthandoff="04/14/2017" />

# <a name="connect-a-web-app-to-a-documentdb"></a>将 Web 应用连接到 documentdb

在此方案中，你将了解如何创建 Azure documentdb 和 Azure Web 应用。 然后，将使用应用设置将 documentdb 链接到 Web 应用。

[!INCLUDE [sample-cli-install](../../includes/sample-cli-install.md)]

[AZURE.INCLUDE [azure-cli-2-azurechinacloud-environment-parameter](../../includes/azure-cli-2-azurechinacloud-environment-parameter.md)]

## <a name="sample-script"></a>示例脚本

    #/bin/bash

    # Variables
    appName="webappwithdocumentdb$random"
    storageName="webappwithdocumentdb$random"
    location="ChinaNorth"

    # Create a Resource Group 
    az group create --name myResourceGroup --location $location

    # Create an App Service Plan
    az appservice plan create --name WebAppWithDocumentDBPlan --resource-group myResourceGroup --location $location

    # Create a Web App
    az appservice web create --name $appName --plan WebAppWithDocumentDBPlan --resource-group myResourceGroup 

    # Create a DocumentDB
    docdb=$(az documentdb create --name $appName --resource-group myResourceGroup --query documentEndpoint --output tsv)
    docCreds=$(az documentdb list-keys --name $appName --resource-group myResourceGroup --query primaryMasterKey --output tsv)

    # Assign the connection string to an App Setting in the Web App
    az appservice web config appsettings update --settings "DOCDB_URL=$docdb" "DOCDB_KEY=$docCreds" --name $appName --resource-group myResourceGroup

[AZURE.INCLUDE [cli-script-clean-up](../../includes/cli-script-clean-up.md)]

## <a name="script-explanation"></a>脚本说明

此脚本使用以下命令创建资源组、Web 应用、documentdb 和所有相关资源。 表中的每条命令均链接到特定于命令的文档。

| 命令 | 说明 |
|---|---|
| [az group create](https://docs.microsoft.com/zh-cn/cli/azure/group#create) | 创建用于存储所有资源的资源组。 |
| [az appservice plan create](https://docs.microsoft.com/zh-cn/cli/azure/appservice/plan#create) | 创建应用服务计划。 这与 Azure Web 应用的服务器场类似。 |
| [az appservice web create](https://docs.microsoft.com/zh-cn/cli/azure/appservice/web#create) | 创建应用服务计划中的 Azure Web 应用。 |
| [az documentdb create](https://docs.microsoft.com/zh-cn/cli/azure/documentdb#create) | 创建 documentdb。 这将是数据存储位置。 |
| [az documentdb list-keys](https://docs.microsoft.com/zh-cn/cli/azure/documentdb#list-keys) | 列出指定的 Azure DocumentDB 数据库帐户的访问密钥。 |
| [az appservice web config appsetings update](https://docs.microsoft.com/zh-cn/cli/azure/appservice/web/config/appsettings#update) | 创建或更新 Azure Web 应用的应用设置。 应用设置将作为应用的环境变量公开。 |

## <a name="next-steps"></a>后续步骤

有关 Azure CLI 的详细信息，请参阅 [Azure CLI 文档](https://docs.microsoft.com/zh-cn/cli/azure/overview)。

可以在 [Azure 应用服务文档](/documentation/articles/app-service-cli-samples/)中找到其他应用服务 CLI 脚本示例。