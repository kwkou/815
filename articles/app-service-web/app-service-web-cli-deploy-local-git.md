<properties
    pageTitle="Azure CLI 脚本示例 - 从本地 Git 存储库部署 Web 应用 | Azure"
    description="Azure CLI 脚本示例 - 从本地 Git 存储库部署 Web 应用"
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
    ms.openlocfilehash="c91c44f6df6c447703d3abd03aadc5a34bdfcb34"
    ms.lasthandoff="04/14/2017" />

# <a name="deploy-a-web-app-from-a-local-git-repository"></a>从本地 Git 存储库部署 Web 应用

此示例脚本使用 Azure CLI 2.0 执行以下操作： 

* 在中国北部 Azure 区域的 Azure 应用服务中创建 Web 应用。
* 从本地 Git 存储库部署 Web 应用代码。
* 在浏览器中显示已部署的 Web 应用。

## <a name="prerequisites"></a>先决条件

* 运行 `az login` 登录到 Azure。
* 将 Web 应用代码提交到本地 Git 存储库。

[AZURE.INCLUDE [azure-cli-2-azurechinacloud-environment-parameter](../../includes/azure-cli-2-azurechinacloud-environment-parameter.md)]

## <a name="create-vm-sample"></a>创建 VM 示例

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


## <a name="clean-up-deployment"></a>清理部署 

运行脚本示例后，可以使用以下命令删除资源组、Web 应用以及所有相关资源。

    az group delete --name $webappname

## <a name="script-explanation"></a>脚本说明

此脚本使用以下命令。 表中的每条命令均链接到特定于命令的文档。

| 命令 | 说明 |
|---|---|
| [az group create](https://docs.microsoft.com/zh-cn/cli/azure/group#create) | 创建用于存储所有资源的资源组。 |
| [az appservice plan create](https://docs.microsoft.com/zh-cn/cli/azure/appservice/plan#create) | 创建应用服务计划。 |
| [az appservice web create](https://docs.microsoft.com/zh-cn/cli/azure/webapp#delete) | 创建 Web 应用 |
| [az appservice web source-control config-local-git](https://docs.microsoft.com/zh-cn/cli/azure/webapp/source-control#config-local-git) | 为本地 Git 存储库创建源控件配置。 |
| [az appservice web browse](https://docs.microsoft.com/zh-cn/cli/azure/webapp#browse) | 在浏览器中打开 Web 应用。 |

## <a name="next-steps"></a>后续步骤

有关 Azure CLI 的详细信息，请参阅 [Azure CLI 文档](https://docs.microsoft.com/zh-cn/cli/azure/overview)。

可以在 [Azure CLI 示例](https://github.com/Azure/azure-docs-cli-python-samples)中找到 Azure 应用服务 Web 应用的其他 CLI 脚本示例。