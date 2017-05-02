<properties
    pageTitle="Azure CLI 脚本示例 - 将自定义域映射到 Web 应用 | Azure"
    description="Azure CLI 脚本示例 - 将自定义域映射到 Web 应用"
    services="app-service\web"
    documentationcenter=""
    author="cephalin"
    manager="erikre"
    editor=""
    tags="azure-service-management"
    translationtype="Human Translation" />
<tags
    ms.assetid="5ac4a680-cc73-4578-bcd6-8668c08802c2"
    ms.service="app-service-web"
    ms.workload="web"
    ms.devlang="azurecli"
    ms.tgt_pltfrm="na"
    ms.topic="article"
    ms.date="04/09/2017"
    wacn.date="05/02/2017"
    ms.author="cephalin"
    ms.sourcegitcommit="78da854d58905bc82228bcbff1de0fcfbc12d5ac"
    ms.openlocfilehash="f89aa33afee2524077c4cb3c6af1748146dfca82"
    ms.lasthandoff="04/22/2017" />

# <a name="map-a-custom-domain-to-a-web-app"></a>将自定义域映射到 Web 应用

此示例脚本使用其相关资源，在应用服务中创建 Web 应用，然后将 `www.<yourdomain>` 映射到它。

[AZURE.INCLUDE [sample-cli-install](../../includes/sample-cli-install.md)]

[AZURE.INCLUDE [azure-cli-2-azurechinacloud-environment-parameter](../../includes/azure-cli-2-azurechinacloud-environment-parameter.md)]

## <a name="sample-script"></a>示例脚本

    #!/bin/bash

    fqdn=<Replace with www.{yourdomain}>
    webappname=mywebapp$RANDOM

    # Create a resource group.
    az group create --location chinanorth --name myResourceGroup

    # Create an App Service plan in SHARED tier (minimum required by custom domains).
    az appservice plan create --name $webappname --resource-group myResourceGroup --sku SHARED

    # Create a web app.
    az appservice web create --name $webappname --resource-group myResourceGroup \
    --plan $webappname

    echo "Configure a CNAME record that maps $fqdn to $webappname.chinacloudsites.cn"
    read -p "Press [Enter] key when ready ..."

    # Before continuing, go to your DNS configuration UI for your custom domain and follow the 
    # instructions at https://www.azure.cn/documentation/articles/web-sites-custom-domain-name/#step-2-create-the-dns-records to configure a CNAME record for the 
    # hostname "www" and point it your web app's default domain name.

    # Map your prepared custom domain name to the web app.
    az appservice web config hostname add --webapp $webappname --resource-group myResourceGroup \
    --name $fqdn

    echo "You can now browse to http://$fqdn"


[AZURE.INCLUDE [cli-script-clean-up](../../includes/cli-script-clean-up.md)]

## <a name="script-explanation"></a>脚本说明

此脚本使用以下命令。 表中的每条命令均链接到特定于命令的文档。

| 命令 | 说明 |
|---|---|
| [az group create](https://docs.microsoft.com/zh-cn/cli/azure/group#create) | 创建用于存储所有资源的资源组。 |
| [az appservice plan create](https://docs.microsoft.com/zh-cn/cli/azure/appservice/plan#create) | 创建应用服务计划。 |
| [az appservice web create](https://docs.microsoft.com/zh-cn/cli/azure/appservice/web#delete) | 创建 Web 应用。 |
| [az appservice web config hostname add](https://docs.microsoft.com/zh-cn/cli/azure/appservice/web/config/hostname#add) | 将自定义域映射到 Web 应用。 |

## <a name="next-steps"></a>后续步骤

有关 Azure CLI 的详细信息，请参阅 [Azure CLI 文档](https://docs.microsoft.com/zh-cn/cli/azure/overview)。

可以在 [Azure 应用服务文档](/documentation/articles/app-service-cli-samples/)中找到其他应用服务 CLI 脚本示例。

<!--Update_Description: wording update-->