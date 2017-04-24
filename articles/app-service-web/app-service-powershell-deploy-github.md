<properties
    pageTitle="Azure PowerShell 脚本示例 - 从 GitHub 创建 Web 应用并部署代码 | Azure"
    description="Azure PowerShell 脚本示例 - 从 GitHub 创建 Web 应用并部署代码"
    services="app-service\web"
    documentationcenter=""
    author="cephalin"
    manager="erikre"
    editor=""
    tags="azure-service-management"
    translationtype="Human Translation" />
<tags
    ms.assetid="0f9c8bc5-3789-4eb3-8deb-ae6e2200795a"
    ms.service="app-service-web"
    ms.workload="web"
    ms.devlang="na"
    ms.topic="article"
    ms.date="03/20/2017"
    wacn.date="04/24/2017"
    ms.author="cephalin"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="121076b74d6b8bdd6452a69b09819e986fc15514"
    ms.lasthandoff="04/14/2017" />

# <a name="create-a-web-app-and-deploy-code-from-github"></a>从 GitHub 创建 Web 应用并部署代码

此示例脚本使用其相关资源，在应用服务中创建 Web 应用，然后从公共 GitHub 存储库部署 Web 应用代码（不进行连续部署）。 有关不进行连续部署的 GitHub 部署，请参阅[从 GitHub 使用连续部署创建 Web 应用](/documentation/articles/app-service-continuous-deployment/)。

必要时，请使用 [Azure PowerShell 指南](https://docs.microsoft.com/zh-cn/powershell/azureps-cmdlets-docs/)中的说明安装 Azure PowerShell。 此外，还需要指向含 web 应用代码的 GitHub 存储库的链接。

## <a name="sample-script"></a>示例脚本

    $gitrepo="<Replace with your GitHub repo URL>"
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

    # Configure GitHub deployment from your GitHub repo and deploy once.
    $PropertiesObject = @{
        repoUrl = "$gitrepo";
        branch = "master";
    }
    Set-AzureRmResource -PropertyObject $PropertiesObject -ResourceGroupName $webappname `
    -ResourceType Microsoft.Web/sites/sourcecontrols -ResourceName $webappname/web `
    -ApiVersion 2015-08-01 -Force

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

## <a name="next-steps"></a>后续步骤

有关 Azure PowerShell 模块的详细信息，请参阅 [Azure PowerShell 文档](https://docs.microsoft.com/zh-cn/powershell/azureps-cmdlets-docs/)。

可以在 [Azure PowerShell 示例](/documentation/articles/app-service-powershell-samples/)中找到 Azure 应用服务 Web 应用的其他 Azure Powershell 示例。