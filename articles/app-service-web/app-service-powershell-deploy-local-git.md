<properties
    pageTitle="Azure PowerShell 脚本示例 - 从本地 Git 存储库创建 Web 应用并部署代码 | Azure"
    description="Azure PowerShell 脚本示例 - 从本地 Git 存储库创建 Web 应用并部署代码"
    services="app-service\web"
    documentationcenter=""
    author="cephalin"
    manager="erikre"
    editor=""
    tags="azure-service-management"
    translationtype="Human Translation" />
<tags
    ms.assetid="5a927f23-8e70-45fd-9aae-980d4e7a007d"
    ms.service="app-service-web"
    ms.workload="web"
    ms.devlang="na"
    ms.topic="article"
    ms.date="03/20/2017"
    wacn.date="04/24/2017"
    ms.author="cephalin"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="0adb0ec89b41f334a644734c9033599c80f00a06"
    ms.lasthandoff="04/14/2017" />

# <a name="create-a-web-app-and-deploy-code-from-a-local-git-repository"></a>从本地 Git 存储库创建 Web 应用并部署代码

此示例脚本使用其相关资源，在应用服务中创建 Web 应用，然后在本地 Git 存储库中部署 Web 应用代码。

必要时，请使用 [Azure PowerShell 指南](https://docs.microsoft.com/zh-cn/powershell/azureps-cmdlets-docs/)中的说明安装 Azure PowerShell，然后运行 `Login-AzureRmAccount -EnvironmentName AzureChinaCloud` 创建与 Azure 的连接。 此外，需将应用程序代码提交到本地 Git 存储库。

## <a name="sample-script"></a>示例脚本

    $gitdirectory="<Replace with path to local Git repo>"
    $webappname="mywebapp$(Get-Random)"
    $location="China North"

    # Create a resource group.
    New-AzureRmResourceGroup -Name myResourceGroup -Location $location

    # Create an App Service plan in `Free` tier.
    New-AzureRmAppServicePlan -Name $webappname -Location $location `
    -ResourceGroupName myResourceGroup -Tier Free

    # Create a web app.
    New-AzureRmWebApp -Name $webappname -Location $location -AppServicePlan $webappname `
    -ResourceGroupName myResourceGroup

    # Configure GitHub deployment from your GitHub repo and deploy once.
    $PropertiesObject = @{
        scmType = "LocalGit";
    }
    Set-AzureRmResource -PropertyObject $PropertiesObject -ResourceGroupName myResourceGroup `
    -ResourceType Microsoft.Web/sites/config -ResourceName $webappname/web `
    -ApiVersion 2015-08-01 -Force

    # Get app-level deployment credentials
    $xml = (Get-AzureRmWebAppPublishingProfile -Name $webappname -ResourceGroupName myResourceGroup `
    -OutputFile null)
    $username = $xml.SelectNodes("//publishProfile[@publishMethod=`"MSDeploy`"]/@userName").value
    $password = $xml.SelectNodes("//publishProfile[@publishMethod=`"MSDeploy`"]/@userPWD").value

    # Add the Azure remote to your local Git respository and push your code
    #### This method saves your password in the git remote. You can use a Git credential manager to secure your password instead.
    git remote add azure "https://${username}:$password@$webappname.scm.chinacloudsites.cn"
    git push azure master

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
| [Set-AzureRmResource](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.resources/v3.5.0/set-azurermresource) | 修改资源组中的资源。 |
| [Get-AzureRmWebAppPublishingProfile](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.websites/v2.5.0/get-azurermwebapppublishingprofile) | 获取 Web 应用的发布配置文件。 |

## <a name="next-steps"></a>后续步骤

有关 Azure PowerShell 模块的详细信息，请参阅 [Azure PowerShell 文档](https://docs.microsoft.com/zh-cn/powershell/azureps-cmdlets-docs/)。

可以在 [Azure PowerShell 示例](/documentation/articles/app-service-powershell-samples/)中找到 Azure 应用服务 Web 应用的其他 Azure Powershell 示例。