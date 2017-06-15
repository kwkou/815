<properties
    pageTitle="Azure CLI 脚本示例 - 从 GitHub 连续部署 Web 应用 | Azure"
    description="Azure CLI 脚本示例 - 从 GitHub 连续部署 Web 应用"
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
    ms.openlocfilehash="e187eb3d0197adebc2fd95a8c1af76669ad2ffa3"
    ms.lasthandoff="04/14/2017" />

# <a name="continuously-deploy-web-app-from-github"></a>从 GitHub 连续部署 Web 应用

此示例脚本使用 Azure CLI 2.0 执行以下操作： 

* 在中国北部 Azure 区域的 Azure 应用服务中创建 Web 应用。 
* 从 GitHub 部署 Web 应用代码。
* 在浏览器中显示已部署的 Web 应用。

## <a name="prerequisites"></a>先决条件

* 运行 `az login` 登录到 Azure。
* 将 Web 应用代码放入你拥有的 GitHub 存储库中。

[AZURE.INCLUDE [azure-cli-2-azurechinacloud-environment-parameter](../../includes/azure-cli-2-azurechinacloud-environment-parameter.md)]

> [AZURE.NOTE]
> 若要设置连续部署，请参阅[连续部署到 Azure 应用服务](/documentation/articles/app-service-continuous-deployment/)。 在 Azure 中国区，需使用 KUDU 进行此设置。
>
>

## <a name="create-vm-sample"></a>创建 VM 示例

    #!/bin/bash

    gitrepo=<Replace with a public GitHub repo URL. e.g. https://github.com/Azure-Samples/app-service-web-dotnet-get-started.git>
    webappname=mywebapp$RANDOM

    # Create a resource group.
    az group create --location chinanorth --name myResourceGroup

    # Create an App Service plan in `FREE` tier.
    az appservice plan create --name $webappname --resource-group myResourceGroup --sku FREE

    # Create a web app.
    az appservice web create --name $webappname --resource-group myResourceGroup --plan $webappname

    # Deploy code from a public GitHub repository. 
    az appservice web source-control config --name $webappname --resource-group myResourceGroup \
    --repo-url $gitrepo --branch master --manual-integration

    # Browse to the web app.
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
| [az appservice web source-control config](https://docs.microsoft.com/zh-cn/cli/azure/webapp/source-control#config) | 将 Web 应用与 Git 或 Mercurial 存储库相关联。 |
| [az appservice web browse](https://docs.microsoft.com/zh-cn/cli/azure/webapp#browse) | 在浏览器中打开 Web 应用。 |

## <a name="next-steps"></a>后续步骤

有关 Azure CLI 的详细信息，请参阅 [Azure CLI 文档](https://docs.microsoft.com/zh-cn/cli/azure/overview)。

可以在 [Azure CLI 示例](https://github.com/Azure/azure-docs-cli-python-samples)中找到 Azure 应用服务 Web 应用的其他 CLI 脚本示例。