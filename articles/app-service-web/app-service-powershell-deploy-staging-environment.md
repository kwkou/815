<properties
    pageTitle="Azure PowerShell 脚本示例 - 创建 Web 应用并将代码部署到过渡环境 | Azure"
    description="Azure PowerShell 脚本示例 - 创建 Web 应用并将代码部署到过渡环境"
    services="app-service\web"
    documentationcenter=""
    author="cephalin"
    manager="erikre"
    editor=""
    tags="azure-service-management"
    translationtype="Human Translation" />
<tags
    ms.assetid="27cf0680-c3a9-4a58-9f71-6dec09f6b874"
    ms.service="app-service-web"
    ms.workload="web"
    ms.devlang="na"
    ms.topic="article"
    ms.date="03/20/2017"
    wacn.date="04/24/2017"
    ms.author="cephalin"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="8289bf6e357c54599956c063612c43debde69a81"
    ms.lasthandoff="04/14/2017" />

# <a name="create-a-web-app-and-deploy-code-to-a-staging-environment"></a>创建 Web 应用并将代码部署到过渡环境

此示例脚本使用名为“过渡”的附加部署槽在应用服务中创建 Web 应用，然后将示例应用部署到“过渡”槽。

必要时，请使用 [Azure PowerShell 指南](https://docs.microsoft.com/zh-cn/powershell/azureps-cmdlets-docs/)中的说明安装 Azure PowerShell，然后运行 `Login-AzureRmAccount -EnvironmentName AzureChinaCloud` 创建与 Azure 的连接。

## <a name="sample-script"></a>示例脚本

    $gitrepo="<Replace with your GitHub repo URL>"
    $webappname="mywebapp$(Get-Random)"
    $location="China North"

    # Create a resource group.
    New-AzureRmResourceGroup -Name myResourceGroup -Location $location

    # Create an App Service plan in Free tier.
    New-AzureRmAppServicePlan -Name $webappname -Location $location `
    -ResourceGroupName myResourceGroup -Tier Free

    # Create a web app.
    New-AzureRmWebApp -Name $webappname -Location $location `
    -AppServicePlan $webappname -ResourceGroupName myResourceGroup

    # Upgrade App Service plan to Standard tier (minimum required by deployment slots)
    Set-AzureRmAppServicePlan -Name $webappname -ResourceGroupName myResourceGroup `
    -Tier Standard

    #Create a deployment slot with the name "staging".
    New-AzureRmWebAppSlot -Name $webappname -ResourceGroupName myResourceGroup `
    -Slot staging

    # Configure GitHub deployment to the staging slot from your GitHub repo and deploy once.
    $PropertiesObject = @{
        repoUrl = "$gitrepo";
        branch = "master";
    }
    Set-AzureRmResource -PropertyObject $PropertiesObject -ResourceGroupName myResourceGroup `
    -ResourceType Microsoft.Web/sites/slots/sourcecontrols `
    -ResourceName $webappname/staging/web -ApiVersion 2015-08-01 -Force

    # Swap the verified/warmed up staging slot into production.
    Switch-​Azure​Rm​Web​App​Slot -Name $webappname -ResourceGroupName myResourceGroup `
    -SourceSlotName staging -DestinationSlotName production

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
| [New-AzureRmWebAppSlot](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.websites/v2.5.0/new-azurermwebappslot) | 为 Web 应用创建部署槽。 |
| [Set-AzureRmResource](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.resources/v3.5.0/set-azurermresource) | 修改资源组中的资源。 |
| [Switch-​Azure​Rm​Web​App​Slot](https://docs.microsoft.com/zh-cn/powershell/module/azurerm.websites/switch-azurermwebappslot) | 将 Web 应用的部署槽交换到生产环境。 |

## <a name="next-steps"></a>后续步骤

有关 Azure PowerShell 模块的详细信息，请参阅 [Azure PowerShell 文档](https://docs.microsoft.com/zh-cn/powershell/azureps-cmdlets-docs/)。

可以在 [Azure PowerShell 示例](/documentation/articles/app-service-powershell-samples/)中找到 Azure 应用服务 Web 应用的其他 Azure Powershell 示例。