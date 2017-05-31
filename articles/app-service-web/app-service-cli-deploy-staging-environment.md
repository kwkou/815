<properties
    pageTitle="Azure CLI 脚本示例 - 创建 Web 应用并将代码部署到过渡环境 | Azure"
    description="Azure CLI 脚本示例 - 创建 Web 应用并将代码部署到过渡环境"
    services="app-service\web"
    documentationcenter=""
    author="cephalin"
    manager="erikre"
    editor=""
    tags="azure-service-management"
    translationtype="Human Translation" />
<tags
    ms.assetid="2b995dcd-e471-4355-9fda-00babcdb156e"
    ms.service="app-service-web"
    ms.workload="web"
    ms.devlang="na"
    ms.topic="article"
    ms.date="03/20/2017"
    wacn.date="04/24/2017"
    ms.author="cephalin"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="d5c646d1bebfe10ff988dba3f7fff619716abe77"
    ms.lasthandoff="04/14/2017" />

# <a name="create-a-web-app-and-deploy-code-to-a-staging-environment"></a>创建 Web 应用并将代码部署到过渡环境

此示例脚本使用名为“过渡”的附加部署槽在应用服务中创建 Web 应用，然后将示例应用部署到“过渡”槽。

[!INCLUDE [sample-cli-install](../../includes/sample-cli-install.md)]

[AZURE.INCLUDE [azure-cli-2-azurechinacloud-environment-parameter](../../includes/azure-cli-2-azurechinacloud-environment-parameter.md)]

## <a name="sample-script"></a>示例脚本

    #!/bin/bash

    gitrepo=<Replace with a public GitHub repo URL. e.g. https://github.com/Azure-Samples/app-service-web-dotnet-get-started.git>
    webappname=mywebapp$RANDOM

    # Create a resource group.
    az group create --location chinanorth --name myResourceGroup

    # Create an App Service plan in STANDARD tier (minimum required by deployment slots).
    az appservice plan create --name $webappname --resource-group myResourceGroup --sku S1

    # Create a web app.
    az appservice web create --name $webappname --resource-group myResourceGroup \
    --plan $webappname

    #Create a deployment slot with the name "staging".
    az appservice web deployment slot create --name $webappname --resource-group myResourceGroup \
    --slot staging

    # Deploy sample code to "staging" slot from GitHub.
    az appservice web source-control config --name $webappname --resource-group myResourceGroup \
    --slot staging --repo-url $gitrepo --branch master --manual-integration

    # Browse to the deployed web app on staging. Deployment may be in progress, so rerun this if necessary.
    az appservice web browse --name $webappname --resource-group myResourceGroup --slot staging

    # Swap the verified/warmed up staging slot into production.
    az appservice web deployment slot swap --name $webappname --resource-group myResourceGroup \
    --slot staging

    # Browse to the production slot. 
    az appservice web browse --name $webappname --resource-group myResourceGroup

[AZURE.INCLUDE [cli-script-clean-up](../../includes/cli-script-clean-up.md)]

## <a name="script-explanation"></a>脚本说明

此脚本使用以下命令。 表中的每条命令均链接到特定于命令的文档。

| 命令 | 说明 |
|---|---|
| [az group create](https://docs.microsoft.com/zh-cn/cli/azure/group#create) | 创建用于存储所有资源的资源组。 |
| [az appservice plan create](https://docs.microsoft.com/zh-cn/cli/azure/appservice/plan#create) | 创建应用服务计划。 |
| [az appservice web create](https://docs.microsoft.com/zh-cn/cli/azure/appservice/web#delete) | 创建 Azure Web 应用。 |
| [az appservice web deployment slot create](https://docs.microsoft.com/zh-cn/cli/azure/appservice/web/deployment/slot#create) | 创建部署槽。 |
| [az appservice web source-control config](https://docs.microsoft.com/zh-cn/cli/azure/appservice/web/source-control#config) | 将 Azure Web 应用与 Git 或 Mercurial 存储库相关联。 |
| [az appservice web browse](https://docs.microsoft.com/zh-cn/cli/azure/appservice/web#browse) | 在浏览器中打开 Azure Web 应用。 |
| [az appservice web deployment slot swap](https://docs.microsoft.com/zh-cn/cli/azure/appservice/web/deployment/slot#swap) | 将指定的部署槽交换到生产环境。 |

## <a name="next-steps"></a>后续步骤

有关 Azure CLI 的详细信息，请参阅 [Azure CLI 文档](https://docs.microsoft.com/zh-cn/cli/azure/overview)。

可以在 [Azure 应用服务文档](/documentation/articles/app-service-cli-samples/)中找到其他应用服务 CLI 脚本示例。