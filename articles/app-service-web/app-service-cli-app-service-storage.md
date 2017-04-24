<properties
    pageTitle="Azure CLI 脚本示例 - 将 Web 应用连接到存储帐户 | Azure"
    description="Azure CLI 脚本示例 - 将 Web 应用连接到存储帐户"
    services="appservice"
    documentationcenter="appservice"
    author="syntaxc4"
    manager="erikre"
    editor=""
    tags="azure-service-management"
    translationtype="Human Translation" />
<tags
    ms.assetid="bc8345b2-8487-40c6-a91f-77414e8688e6"
    ms.service="app-service"
    ms.devlang="multiple"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="web"
    ms.date="03/20/2017"
    wacn.date="04/24/2017"
    ms.author="cfowler"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="9cf5bb25a1a117bf4f2488b84c41f3c6309ceaa4"
    ms.lasthandoff="04/14/2017" />

# <a name="connect-a-web-app-to-a-storage-account"></a>将 Web 应用连接到存储帐户

在此方案中，你将了解如何创建 Azure 存储帐户和 Azure Web 应用。 然后，将使用应用设置将存储帐户链接到 Web 应用。

必要时，请使用 [Azure CLI 安装指南](https://docs.microsoft.com/zh-cn/cli/azure/install-azure-cli)中的说明安装 Azure CLI，然后运行 `az login` 创建与 Azure 的连接。

[AZURE.INCLUDE [azure-cli-2-azurechinacloud-environment-parameter](../../includes/azure-cli-2-azurechinacloud-environment-parameter.md)]

此示例在 Bash shell 中正常工作。 有关在 Windows 客户端上运行 Azure CLI 脚本的选项，请参阅[在 Windows 中运行 Azure CLI](/documentation/articles/virtual-machines-windows-cli-options/)。

## <a name="sample-script"></a>示例脚本

    #/bin/bash

    # Variables
    appName="webappwithstorage$random"
    storageName="webappstorage$random"
    location="ChinaNorth"

    # Create a Resource Group 
    az group create --name myResourceGroup --location $location

    # Create an App Service Plan
    az appservice plan create --name WebAppWithStoragePlan --resource-group myResourceGroup --location $location

    # Create a Web App
    az appservice web create --name $appName --plan WebAppWithStoragePlan --resource-group myResourceGroup 

    # Create a Storage Account
    az storage account create --name $storageName --resource-group myResourceGroup --location $location --sku Standard_LRS

    # Retreive the Storage Account connection string 
    connstr=$(az storage account show-connection-string --name $storageName --resource-group myResourceGroup --query connectionString --output tsv)

    # Assign the connection string to an App Setting in the Web App
    az appservice web config appsettings update --settings "STORAGE_CONNSTR=$connstr" --name $appName --resource-group myResourceGroup

[AZURE.INCLUDE [cli-script-clean-up](../../includes/cli-script-clean-up.md)]

## <a name="script-explanation"></a>脚本说明

此脚本使用以下命令创建资源组、Web 应用、存储帐户和所有相关资源。 表中的每条命令均链接到特定于命令的文档。

| 命令 | 说明 |
|---|---|
| [az group create](https://docs.microsoft.com/zh-cn/cli/azure/group#create) | 创建用于存储所有资源的资源组。 |
| [az appservice plan create](https://docs.microsoft.com/zh-cn/cli/azure/appservice/plan#create) | 创建应用服务计划。 这与 Azure Web 应用的服务器场类似。 |
| [az appservice web create](https://docs.microsoft.com/zh-cn/cli/azure/appservice/web#create) | 创建应用服务计划中的 Azure Web 应用。 |
| [az storage account create](https://docs.microsoft.com/zh-cn/cli/azure/storage/account#create) | 创建存储帐户。 此帐户将存储静态资产。 |
| [az storage account show-connection-string](https://docs.microsoft.com/zh-cn/cli/azure/storage/account#show-connection-string) | |
| [az appservice web config appsetings update](https://docs.microsoft.com/zh-cn/cli/azure/appservice/web/config/appsettings#update) | 创建或更新 Azure Web 应用的应用设置。 应用设置将作为应用的环境变量公开。 |

## <a name="next-steps"></a>后续步骤

有关 Azure CLI 的详细信息，请参阅 [Azure CLI 文档](https://docs.microsoft.com/zh-cn/cli/azure/overview)。

可以在 [Azure 应用服务文档](/documentation/articles/app-service-cli-samples/)中找到其他应用服务 CLI 脚本示例。