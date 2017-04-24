<properties
    pageTitle="在 Azure 中不到 5 分钟创建你的第一个 Java Web 应用 | Azure"
    description="了解如何部署示例应用，轻松地在应用服务中运行 Web 应用。"
    services="app-service\web"
    documentationcenter=""
    author="cephalin"
    manager="wpickett"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid="8bacfe3e-7f0b-4394-959a-a88618cb31e1"
    ms.service="app-service-web"
    ms.workload="web"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="hero-article"
    ms.date="03/17/2017"
    wacn.date="04/24/2017"
    ms.author="cephalin"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="1fc45645f8cc055482281aab2aef022082d314c0"
    ms.lasthandoff="04/14/2017" />

# <a name="create-your-first-java-web-app-in-azure-in-five-minutes"></a>在 Azure 中不到 5 分钟创建你的第一个 Java Web 应用
[AZURE.INCLUDE [app-service-web-selector-get-started](../../includes/app-service-web-selector-get-started.md)]

本快速入门帮助你在数分钟内将你的第一个 Java Web 应用部署到 [Azure 应用服务](/documentation/articles/app-service-value-prop-what-is/)。

在开始之前，请确保已安装 Azure CLI。 有关详细信息，请参阅 [Azure CLI 安装指南](https://docs.microsoft.com/zh-cn/cli/azure/install-azure-cli)。

## <a name="log-in-to-azure"></a>登录 Azure
运行 `az login` 并按屏幕说明进行操作，以便登录到 Azure。

    az login

[AZURE.INCLUDE [azure-cli-2-azurechinacloud-environment-parameter](../../includes/azure-cli-2-azurechinacloud-environment-parameter.md)]

## <a name="create-a-resource-group"></a>创建资源组   
创建[资源组](/documentation/articles/resource-group-overview/)。 这是放置所有 Azure 资源（例如 Web 应用及其 SQL 数据库后端）的地方，这些资源需要集中进行管理。

    az group create --location "China North" --name myResourceGroup

若要查看适用于 `--location` 的可能值，请使用 `az appservice list-locations` Azure CLI 命令。

## <a name="create-an-app-service-plan"></a>创建应用服务计划
创建“免费”[应用服务计划](/documentation/articles/azure-web-sites-web-hosting-plans-in-depth-overview/)。 

    az appservice plan create --name my-free-appservice-plan --resource-group myResourceGroup --sku FREE

## <a name="create-a-web-app"></a>创建 Web 应用
使用 `<app_name>` 中的唯一名称创建 Web 应用。

    az appservice web create --name <app_name> --resource-group myResourceGroup --plan my-free-appservice-plan

## <a name="deploy-sample-application"></a>部署示例应用程序
从 GitHub 部署示例 Java 应用。

    az appservice web source-control config --name <app_name> --resource-group myResourceGroup \
    --repo-url "https://github.com/azure-appservice-samples/JavaCoffeeShopTemplate.git" --branch master --manual-integration 

## <a name="browse-to-web-app"></a>浏览到 Web 应用
若要查看应用在 Azure 中的实时运行情况，请运行此命令。

    az appservice web browse --name <app_name> --resource-group myResourceGroup

恭喜，你的第一个 Java Web 应用已在 Azure 应用服务中实时运行！

[AZURE.INCLUDE [cli-samples-clean-up](../../includes/cli-samples-clean-up.md)]

## <a name="next-steps"></a>后续步骤

浏览预先创建的 [Web 应用 CLI 脚本](/documentation/articles/app-service-cli-samples/)。

<!--Update_Description: wording update-->