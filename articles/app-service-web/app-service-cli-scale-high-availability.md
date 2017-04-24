<properties
    pageTitle="Azure CLI 脚本示例 - 缩放具有高可用性体系结构的全球 Web 应用 | Azure"
    description="Azure CLI 脚本示例 - 缩放具有高可用性体系结构的全球 Web 应用"
    services="appservice"
    documentationcenter="appservice"
    author="syntaxc4"
    manager="erikre"
    editor=""
    tags="azure-service-management"
    translationtype="Human Translation" />
<tags
    ms.assetid="e4033a50-0e05-4505-8ce8-c876204b2acc"
    ms.service="app-service"
    ms.devlang="multiple"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="web"
    ms.date="03/20/2017"
    wacn.date="04/24/2017"
    ms.author="cfowler"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="78bab88e64b19993bbb58c52276020ebfaed717c"
    ms.lasthandoff="04/14/2017" />

# <a name="scale-a-web-app-worldwide-with-a-high-availability-architecture"></a>缩放具有高可用性体系结构的全球 Web 应用

在此方案中，将创建一个资源组、两个应用服务计划、两个 Web 应用、一个流量管理器配置文件和两个流量管理器终结点。 完成本练习后，你将获得一个高可用性体系结构，它基于最低网络延迟提供 Web 应用的全局可用性。

必要时，请使用 [Azure CLI 安装指南](https://docs.microsoft.com/zh-cn/cli/azure/install-azure-cli)中的说明安装 Azure CLI，然后运行 `az login` 创建与 Azure 的连接。

[AZURE.INCLUDE [azure-cli-2-azurechinacloud-environment-parameter](../../includes/azure-cli-2-azurechinacloud-environment-parameter.md)]

此示例在 Bash shell 中正常工作。 有关在 Windows 客户端上运行 Azure CLI 脚本的选项，请参阅[在 Windows 中运行 Azure CLI](/documentation/articles/virtual-machines-windows-cli-options/)。

## <a name="sample-script"></a>示例脚本

    #/bin/bash

    # Variables
    trafficManagerDnsName="myDnsName$random"
    app1Name="AppServiceTM1$random"
    app2Name="AppServiceTM2$random"
    location1="ChinaNorth"
    location2="ChinaEast"

    # Create a Resource Group
    az group create --name myResourceGroup --location $location1

    # Create a Traffic Manager Profile
    az network traffic-manager profile create --name $trafficManagerDnsName-tmp --resource-group myResourceGroup --routing-method Performance --unique-dns-name $trafficManagerDnsName

    # Create App Service Plans in two Regions
    az appservice plan create --name $app1Name-Plan --resource-group myResourceGroup --location $location1 --sku S1
    az appservice plan create --name $app2Name-Plan --resource-group myResourceGroup --location $location2 --sku S1

    # Add a Web App to each App Service Plan
    site1=$(az appservice web create --name $app1Name --plan $app1Name-Plan --resource-group myResourceGroup --query id --output tsv)
    site2=$(az appservice web create --name $app2Name --plan $app2Name-Plan --resource-group myResourceGroup --query id --output tsv)

    # Assign each Web App as an Endpoint for high-availabilty
    az network traffic-manager endpoint create -n $app1Name-$location1 --profile-name $trafficManagerDnsName-tmp -g myResourceGroup --type azureEndpoints --target-resource-id $site1
    az network traffic-manager endpoint create -n $app2Name-$location2 --profile-name $trafficManagerDnsName-tmp -g myResourceGroup --type azureEndpoints --target-resource-id $site2

[AZURE.INCLUDE [cli-script-clean-up](../../includes/cli-script-clean-up.md)]

## <a name="script-explanation"></a>脚本说明

此脚本使用以下命令创建资源组、Web 应用、流量管理器配置文件和所有相关资源。 表中的每条命令均链接到特定于命令的文档。

| 命令 | 说明 |
|---|---|
| [az group create](https://docs.microsoft.com/zh-cn/cli/azure/group#create) | 创建用于存储所有资源的资源组。 |
| [az appservice plan create](https://docs.microsoft.com/zh-cn/cli/azure/appservice/plan#create) | 创建应用服务计划。 这与 Azure Web 应用的服务器场类似。 |
| [az appservice web create](https://docs.microsoft.com/zh-cn/cli/azure/appservice/web#create) | 创建应用服务计划中的 Azure Web 应用。 |
| [az network traffic-manager profile create](https://docs.microsoft.com/zh-cn/cli/azure/network/traffic-manager/profile#create) | 创建 Azure 流量管理器配置文件。 |
| [az network traffic-manager endpoint create](https://docs.microsoft.com/zh-cn/cli/azure/network/traffic-manager/endpoint#create) | 将终结点添加到 Azure 流量管理器配置文件。 |

## <a name="next-steps"></a>后续步骤

有关 Azure CLI 的详细信息，请参阅 [Azure CLI 文档](https://docs.microsoft.com/zh-cn/cli/azure/overview)。

可以在 [Azure 应用服务文档](/documentation/articles/app-service-cli-samples/)中找到其他应用服务 CLI 脚本示例。