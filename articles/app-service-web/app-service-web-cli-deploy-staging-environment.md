<properties
    pageTitle="Azure CLI 脚本示例 - 将 Web 应用部署到过渡环境 | Azure"
    description="Azure CLI 脚本示例 - 将 Web 应用部署到过渡环境"
    services="app-service\web"
    documentationcenter=""
    author="cephalin"
    manager="erikre"
    editor=""
    tags="azure-service-management"
    translationtype="Human Translation" />
<tags
    ms.assetid=""
    ms.service="app-service-web"
    ms.workload="web"
    ms.devlang="na"
    ms.topic="article"
    ms.date="02/01/2017"
    wacn.date="04/24/2017"
    ms.author="cephalin"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="7b69e14fe42b2a90ee14b67d191ed53521b6c989"
    ms.lasthandoff="04/14/2017" />

# <a name="deploy-a-web-app-to-a-staging-environment"></a>将 Web 应用部署到过渡环境

此示例脚本使用 Azure CLI 2.0 执行以下操作： 

* 在中国北部 Azure 区域的 Azure 应用服务中创建 Web 应用。
* 将应用服务计划升级到标准层（部署槽的最小值）。
* 设置名为“过渡”的部署槽。
* 从 GitHub 将 Web 应用代码部署到过渡槽。
* 在浏览器中打开已部署的过渡槽进行验证。
* 将过渡槽交换到生产环境。

## <a name="prerequisites"></a>先决条件

* 运行 `az login` 登录到 Azure。
* 将 Web 应用代码放入你拥有的 GitHub 存储库中。

[AZURE.INCLUDE [azure-cli-2-azurechinacloud-environment-parameter](../../includes/azure-cli-2-azurechinacloud-environment-parameter.md)]

> [AZURE.NOTE]
> 如果使用的是你未拥有的公共 GitHub 存储库，应用服务将从该 GitHub 存储库部署代码，但无法设置连续部署所需的 webhook。
>
>

## <a name="create-vm-sample"></a>创建 VM 示例

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


## <a name="clean-up-deployment"></a>清理部署 

运行脚本示例后，可以使用以下命令删除资源组、Web 应用以及所有相关资源。

    az group delete --name $webappname

## <a name="script-explanation"></a>脚本说明

此脚本使用以下命令。 表中的每条命令均链接到特定于命令的文档。

| 命令 | 说明 |
|---|---|
| [az group create](https://docs.microsoft.com/zh-cn/cli/azure/group#create) | 创建用于存储所有资源的资源组。 |
| [az appservice plan create](https://docs.microsoft.com/zh-cn/cli/azure/appservice/plan#create) | 创建应用服务计划。 |
| [az appservice web create](https://docs.microsoft.com/zh-cn/cli/azure/webapp#delete) | 创建 Web 应用。 |
| [az appservice plan update](https://docs.microsoft.com/zh-cn/cli/azure/appservice/plan#update) | 更新应用服务计划以缩放其定价层。 |
| [az appservice web deployment slot create](https://docs.microsoft.com/zh-cn/cli/azure/webapp/deployment/slot#create) | 创建部署槽。 |
| [az appservice web source-control config](https://docs.microsoft.com/zh-cn/cli/azure/webapp/source-control#config) | 将 Web 应用与 Git 或 Mercurial 存储库相关联。 |
| [az appservice web browse](https://docs.microsoft.com/zh-cn/cli/azure/webapp#browse) | 在浏览器中打开 Web 应用。 |
| [az appservice web deployment slot swap](https://docs.microsoft.com/zh-cn/cli/azure/webapp/deployment/slot#swap) | 将指定的部署槽交换到生产环境。 |

## <a name="next-steps"></a>后续步骤

有关 Azure CLI 的详细信息，请参阅 [Azure CLI 文档](https://docs.microsoft.com/zh-cn/cli/azure/overview)。

可以在 [Azure CLI 示例](https://github.com/Azure/azure-docs-cli-python-samples)中找到 Azure 应用服务 Web 应用的其他 CLI 脚本示例。