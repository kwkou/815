<properties
    pageTitle="Azure CLI 脚本示例 - 使用 Azure CLI 2.0 手动缩放 Web 应用 | Azure"
    description="Azure CLI 脚本示例 - 使用 Azure CLI 2.0 手动缩放 Web 应用"
    services="appservice"
    documentationcenter="appservice"
    author="syntaxc4"
    manager="erikre"
    editor=""
    tags="azure-service-management"
    translationtype="Human Translation" />
<tags
    ms.assetid="251d9074-8fff-4121-ad16-9eca9556ac96"
    ms.service="app-service"
    ms.devlang="multiple"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="web"
    ms.date="03/20/2017"
    wacn.date="04/24/2017"
    ms.author="cfowler"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="72c991f9c3ba36c66dc7c5a8a4aada7be6cbe1b0"
    ms.lasthandoff="04/14/2017" />

# <a name="scale-a-web-app-manually"></a>手动缩放 Web 应用

在此方案中，你将了解如何创建资源组、应用服务计划和 Web 应用。 然后，将应用服务计划从单个实例扩展到多个实例。

[!INCLUDE [sample-cli-install](../../includes/sample-cli-install.md)]

[AZURE.INCLUDE [azure-cli-2-azurechinacloud-environment-parameter](../../includes/azure-cli-2-azurechinacloud-environment-parameter.md)]

## <a name="sample-script"></a>示例脚本

    #/bin/bash

    # Variables
    appName="AppServiceManualScale$random"
    location="ChinaNorth"

    # Create a Resource Group
    az group create --name myResourceGroup --location $location

    # Create App Service Plans
    az appservice plan create --name AppServiceManualScalePlan --resource-group myResourceGroup --location $location --sku B1

    # Add a Web App
    az appservice web create --name $appName --plan AppServiceManualScalePlan --resource-group myResourceGroup

    # Scale Web App to 2 Workers
    az appservice plan update --number-of-workers 2 --name AppServiceManualScalePlan --resource-group myResourceGroup

[AZURE.INCLUDE [cli-script-clean-up](../../includes/cli-script-clean-up.md)]

## <a name="script-explanation"></a>脚本说明

此脚本使用以下命令创建资源组、Web 应用和所有相关资源。 表中的每条命令均链接到特定于命令的文档。

| 命令 | 说明 |
|---|---|
| [az group create](https://docs.microsoft.com/zh-cn/cli/azure/group#create) | 创建用于存储所有资源的资源组。 |
| [az appservice plan create](https://docs.microsoft.com/zh-cn/cli/azure/appservice/plan#create) | 创建应用服务计划。 这与 Azure Web 应用的服务器场类似。 |
| [az appservice web create](https://docs.microsoft.com/zh-cn/cli/azure/webapp#create) | 创建应用服务计划中的 Azure Web 应用。 |
| [az appservice plan update](https://docs.microsoft.com/zh-cn/cli/azure/appservice/plan#update) | 更新应用服务计划的属性。 |

## <a name="next-steps"></a>后续步骤

有关 Azure CLI 的详细信息，请参阅 [Azure CLI 文档](https://docs.microsoft.com/zh-cn/cli/azure/overview)。

可以在 [Azure 应用服务文档](/documentation/articles/app-service-cli-samples/)中找到其他应用服务 CLI 脚本示例。