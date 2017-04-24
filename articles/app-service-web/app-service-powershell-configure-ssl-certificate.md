<properties
    pageTitle="Azure PowerShell 脚本示例 - 将自定义 SSL 证书绑定到 Web 应用 | Azure"
    description="Azure PowerShell 脚本示例 - 将自定义 SSL 证书绑定到 Web 应用"
    services="app-service\web"
    documentationcenter=""
    author="cephalin"
    manager="erikre"
    editor=""
    tags="azure-service-management"
    translationtype="Human Translation" />
<tags
    ms.assetid="23e83b74-614a-49a0-bc08-7542120eeec5"
    ms.service="app-service-web"
    ms.workload="web"
    ms.devlang="na"
    ms.topic="article"
    ms.date="03/20/2017"
    wacn.date="04/24/2017"
    ms.author="cephalin"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="d49d971c4bf9959da2307df1c85c41a2b04e1144"
    ms.lasthandoff="04/14/2017" />

# <a name="bind-a-custom-ssl-certificate-to-a-web-app"></a>将自定义 SSL 证书绑定到 Web 应用

此示例脚本在应用服务中创建一个 Web 应用及其相关资源，然后将自定义域名的 SSL 证书绑定到该应用。 

必要时，请使用 [Azure PowerShell 指南](https://docs.microsoft.com/zh-cn/powershell/azureps-cmdlets-docs/)中的说明安装 Azure PowerShell。 同时，请确保：

- 已使用 `az login` 命令创建与 Azure 的连接。
- 你可以访问域注册机构的 DNS 配置页。
- 你有要上载和绑定的 SSL 证书的有效 .PFX 文件及其密码。

[AZURE.INCLUDE [azure-cli-2-azurechinacloud-environment-parameter](../../includes/azure-cli-2-azurechinacloud-environment-parameter.md)]

## <a name="sample-script"></a>示例脚本

    $fqdn="<Replace with your custom domain name>"
    $pfxPath="<Replace with path to your .PFX file>"
    $pfxPassword="<Replace with your .PFX password>"
    $webappname="mywebapp$(Get-Random)"
    $location="China North"

    # Create a resource group.
    New-AzureRmResourceGroup -Name $webappname -Location $location

    # Create an App Service plan in Free tier.
    New-AzureRmAppServicePlan -Name $webappname -Location $location `
    -ResourceGroupName $webappname -Tier Free

    # Create a web app.
    New-AzureRmWebApp -Name $webappname -Location $location -AppServicePlan $webappname `
    -ResourceGroupName $webappname

    Write-Host "Configure a CNAME record that maps $fqdn to $webappname.chinacloudsites.cn"
    Read-Host "Press [Enter] key when ready ..."

    # Before continuing, go to your DNS configuration UI for your custom domain and follow the 
    # instructions at https://www.azure.cn/documentation/articles/web-sites-custom-domain-name/#step-2-create-the-dns-records to configure a CNAME record for the 
    # hostname "www" and point it your web app's default domain name.

    # Upgrade App Service plan to Basic tier (minimum required by custom SSL certificates)
    Set-AzureRmAppServicePlan -Name $webappname -ResourceGroupName $webappname `
    -Tier Basic

    # Add a custom domain name to the web app. 
    Set-AzureRmWebApp -Name $webappname -ResourceGroupName $webappname `
    -HostNames @($fqdn,"$webappname.chinacloudsites.cn")

    # Upload and bind the SSL certificate to the web app.
    New-AzureRmWebAppSSLBinding -WebAppName $webappname -ResourceGroupName $webappname -Name $fqdn `
    -CertificateFilePath $pfxPath -CertificatePassword $pfxPassword -SslState SniEnabled

## <a name="clean-up-deployment"></a>清理部署 

运行脚本示例后，可以使用以下命令删除资源组、Web 应用以及所有相关资源。

    Remove-AzureRmResourceGroup -Name myResourceGroup -Force

## <a name="script-explanation"></a>脚本说明

此脚本使用以下命令。 表中的每条命令均链接到特定于命令的文档。

| 命令 | 说明 |
|---|---|
| [New-AzureRmResourceGroup](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/AzureRM.Resources/v3.5.0/new-azurermresourcegroup) | 创建用于存储所有资源的资源组。 |
| [New-AzureRmAppServicePlan](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.websites/v2.5.0/new-azurermappserviceplan) | 创建应用服务计划。 |
| [New-AzureRmWebApp](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.websites/v2.5.0/new-azurermwebapp) | 创建 Web 应用。 |
| [Set-AzureRmAppServicePlan](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.websites/v2.5.0/set-azurermappserviceplan) | 修改应用服务计划以更改其定价层。 |
| [Set-AzureRmWebApp](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.websites/v2.5.0/set-azurermwebapp) | 修改 Web 应用的配置。 |
| [New-AzureRmWebAppSSLBinding](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.websites/v2.5.0/new-azurermwebappsslbinding) | 为 Web 应用创建 SSL 证书绑定。 |

## <a name="next-steps"></a>后续步骤

有关 Azure PowerShell 模块的详细信息，请参阅 [Azure PowerShell 文档](https://docs.microsoft.com/zh-cn/powershell/azureps-cmdlets-docs/)。

可以在 [Azure PowerShell 示例](/documentation/articles/app-service-powershell-samples/)中找到 Azure 应用服务 Web 应用的其他 Azure Powershell 示例。