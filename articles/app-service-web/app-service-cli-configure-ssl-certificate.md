<properties
    pageTitle="Azure CLI 脚本示例 - 将自定义 SSL 证书绑定到 Web 应用 | Azure"
    description="Azure CLI 脚本示例 - 将自定义 SSL 证书绑定到 Web 应用"
    services="app-service\web"
    documentationcenter=""
    author="cephalin"
    manager="erikre"
    editor=""
    tags="azure-service-management"
    translationtype="Human Translation" />
<tags
    ms.assetid="eb95d350-81ea-4145-a1e2-6eea3b7469b2"
    ms.service="app-service-web"
    ms.workload="web"
    ms.devlang="na"
    ms.topic="article"
    ms.date="03/20/2017"
    wacn.date="04/24/2017"
    ms.author="cephalin"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="ba51de270fb8d6765371130be3dfaf191f63d440"
    ms.lasthandoff="04/14/2017" />

# <a name="bind-a-custom-ssl-certificate-to-a-web-app"></a>将自定义 SSL 证书绑定到 Web 应用

此示例脚本在应用服务中创建一个 Web 应用及其相关资源，然后将自定义域名的 SSL 证书绑定到该应用。 

必要时，请使用 [Azure CLI 安装指南](https://docs.microsoft.com/zh-cn/cli/azure/install-azure-cli)中的指令安装 Azure CLI。 同时，请确保：

- 已使用 `az login` 命令创建与 Azure 的连接。
- 你可以访问域注册机构的 DNS 配置页。
- 你有要上载和绑定的 SSL 证书的有效 .PFX 文件及其密码。

[AZURE.INCLUDE [azure-cli-2-azurechinacloud-environment-parameter](../../includes/azure-cli-2-azurechinacloud-environment-parameter.md)]

此示例在 Bash shell 中正常工作。 有关在 Windows 客户端上运行 Azure CLI 脚本的选项，请参阅[在 Windows 中运行 Azure CLI](/documentation/articles/virtual-machines-windows-cli-options/)。

## <a name="sample-script"></a>示例脚本

    #!/bin/bash

    fqdn=<Replace with www.{yourdomain}>
    pfxPath=<Replace with path to your .PFX file>
    pfxPassword=<Replace with your .PFX password>
    webappname=mywebapp$RANDOM

    # Create a resource group.
    az group create --location chinanorth --name myResourceGroup

    # Create an App Service plan in Basic tier (minimum required by custom domains).
    az appservice plan create --name $webappname --resource-group myResourceGroup --sku B1

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

    # Upload the SSL certificate and get the thumbprint.
    thumprint=$(az appservice web config ssl upload --certificate-file $pfxPath \
    --certificate-password $pfxPassword --name $webappname --resource-group myResourceGroup \
    --query thumbprint --output tsv)

    # Binds the uploaded SSL certificate to the web app.
    az appservice web config ssl bind --certificate-thumbprint $thumbprint --ssl-type SNI \
    --name $webappname --resource-group myResourceGroup

    echo "You can now browse to https://$fqdn"

[AZURE.INCLUDE [cli-script-clean-up](../../includes/cli-script-clean-up.md)]

## <a name="script-explanation"></a>脚本说明

此脚本使用以下命令。 表中的每条命令均链接到特定于命令的文档。

| 命令 | 说明 |
|---|---|
| [az group create](https://docs.microsoft.com/zh-cn/cli/azure/group#create) | 创建用于存储所有资源的资源组。 |
| [az appservice plan create](https://docs.microsoft.com/zh-cn/cli/azure/appservice/plan#create) | 创建应用服务计划。 |
| [az appservice web create](https://docs.microsoft.com/zh-cn/cli/azure/appservice/web#delete) | 创建 Azure Web 应用。 |
| [az appservice web config hostname add](https://docs.microsoft.com/zh-cn/cli/azure/appservice/web/config/hostname#add) | 更新应用服务计划以缩放其定价层。 |
| [az appservice web config ssl upload](https://docs.microsoft.com/zh-cn/cli/azure/appservice/web/config/ssl#upload) | 将 SSL 证书上载到 Web 应用。 |
| [az appservice web config ssl bind](https://docs.microsoft.com/zh-cn/cli/azure/appservice/web/config/ssl#bind) | 将上载的 SSL 证书绑定到 Web 应用。 |

## <a name="next-steps"></a>后续步骤

有关 Azure CLI 的详细信息，请参阅 [Azure CLI 文档](https://docs.microsoft.com/zh-cn/cli/azure/overview)。

可以在 [Azure 应用服务文档](/documentation/articles/app-service-cli-samples/)中找到其他应用服务 CLI 脚本示例。