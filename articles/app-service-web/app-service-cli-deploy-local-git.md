<properties
    pageTitle="Azure CLI 脚本示例 - 从本地 Git 存储库创建 Web 应用并部署代码 | Azure"
    description="Azure CLI 脚本示例 - 从本地 Git 存储库创建 Web 应用并部署代码"
    services="app-service\web"
    documentationcenter=""
    author="cephalin"
    manager="erikre"
    editor=""
    tags="azure-service-management"
    translationtype="Human Translation" />
<tags
    ms.assetid="048f98aa-f708-44cb-9b9e-953f67dc6da8"
    ms.service="app-service-web"
    ms.workload="web"
    ms.devlang="na"
    ms.topic="article"
    ms.date="03/20/2017"
    wacn.date="04/24/2017"
    ms.author="cephalin"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="7c423c0e7a81e9631bf4c9c9cc7a402c2f726e0f"
    ms.lasthandoff="04/14/2017" />

# <a name="create-a-web-app-and-deploy-code-from-a-local-git-repository"></a>从本地 Git 存储库创建 Web 应用并部署代码

此示例脚本使用其相关资源，在应用服务中创建 Web 应用，然后在本地 Git 存储库中部署 Web 应用代码。

[!INCLUDE [sample-cli-install](../../includes/sample-cli-install.md)]

[AZURE.INCLUDE [azure-cli-2-azurechinacloud-environment-parameter](../../includes/azure-cli-2-azurechinacloud-environment-parameter.md)]

## <a name="sample-script"></a>示例脚本

    #!/bin/bash

    gitdirectory=<Replace with path to local Git repo>
    username=<Replace with desired deployment username>
    password=<Replace with desired deployment password>
    webappname=mywebapp$RANDOM

    # Create a resource group.
    az group create --location chinanorth --name myResourceGroup

    # Create an App Service plan in FREE tier.
    az appservice plan create --name $webappname --resource-group myResourceGroup --sku FREE

    # Create a web app.
    az appservice web create --name $webappname --resource-group myResourceGroup --plan $webappname

    # Set the account-level deployment credentials
    az appservice web deployment user set --user-name $username --password $password

    # Configure local Git and get deployment URL
    url=$(az appservice web source-control config-local-git --name $webappname \
    --resource-group myResourceGroup --query url --output tsv)

    # Add the Azure remote to your local Git respository and push your code
    cd $gitdirectory
    git remote add azure $url
    git push azure master

    # When prompted for password, use the value of $password that you specified

    # Browse to the deployed web app.
    az appservice web browse --name $webappname --resource-group myResourceGroup

[AZURE.INCLUDE [cli-script-clean-up](../../includes/cli-script-clean-up.md)]

## <a name="script-explanation"></a>脚本说明

此脚本使用以下命令。 表中的每条命令均链接到特定于命令的文档。

| 命令 | 说明 |
|---|---|
| [az group create](https://docs.microsoft.com/zh-cn/cli/azure/group#create) | 创建用于存储所有资源的资源组。 |
| [az appservice plan create](https://docs.microsoft.com/zh-cn/cli/azure/appservice/plan#create) | 创建应用服务计划。 |
| [az appservice web create](https://docs.microsoft.com/zh-cn/cli/azure/webapp#delete) | 创建 Azure Web 应用。 |
| [az appservice web deployment user set](https://docs.microsoft.com/zh-cn/cli/azure/webapp/deployment/user#set) | 为应用服务设置帐户级别部署凭据。 |
| [az appservice web source-control config-local-git](https://docs.microsoft.com/zh-cn/cli/azure/webapp/source-control#config-local-git) | 为本地 Git 存储库创建源控件配置。 |
| [az appservice web browse](https://docs.microsoft.com/zh-cn/cli/azure/webapp#browse) | 在浏览器中打开 Azure Web 应用。 |

## <a name="next-steps"></a>后续步骤

有关 Azure CLI 的详细信息，请参阅 [Azure CLI 文档](https://docs.microsoft.com/zh-cn/cli/azure/overview)。

可以在 [Azure 应用服务文档](/documentation/articles/app-service-cli-samples/)中找到其他应用服务 CLI 脚本示例。