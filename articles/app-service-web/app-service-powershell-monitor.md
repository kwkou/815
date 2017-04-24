<properties
    pageTitle="Azure PowerShell 脚本示例 - 使用 Web 服务器日志监视 Web 应用 | Azure"
    description="Azure PowerShell 脚本示例 - 使用 Web 服务器日志监视 Web 应用"
    services="app-service\web"
    documentationcenter=""
    author="syntaxc4"
    manager="erikre"
    editor=""
    tags="azure-service-management"
    translationtype="Human Translation" />
<tags
    ms.assetid="5805d7cd-9e56-4eba-bd85-75b013690ff5"
    ms.service="app-service"
    ms.devlang="multiple"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="web"
    ms.date="03/20/2017"
    wacn.date="04/24/2017"
    ms.author="cfowler"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="5e30e98c13ba67aff41302a48edd8d1d281c64cd"
    ms.lasthandoff="04/14/2017" />

# <a name="monitor-a-web-app-with-web-server-logs"></a>使用 Web 服务器日志监视 Web 应用

在此方案中，将创建资源组、应用服务计划、Web 应用，并配置 Web 应用以启用 Web 服务器日志。 然后，将下载日志文件以供查看。

必要时，请使用 [Azure PowerShell 指南](https://docs.microsoft.com/zh-cn/powershell/azureps-cmdlets-docs/)中的说明安装 Azure PowerShell，然后运行 `Login-AzureRmAccount -EnvironmentName AzureChinaCloud` 创建与 Azure 的连接。

## <a name="sample-script"></a>示例脚本

    # Generates a Random Value
    $Random=(New-Guid).ToString().Substring(0,8)

    # Variables
    $ResourceGroupName="myResourceGroup$Random"
    $AppName="AppServiceMonitor$Random"
    $Location="ChinaNorth"

    # Create a Resource Group
    New-AzureRMResourceGroup -Name $ResourceGroupName -Location $Location

    # Create an App Service Plan
    New-AzureRMAppservicePlan -Name AppServiceMonitorPlan -ResourceGroupName $ResourceGroupName -Location $Location -Tier Basic

    # Create a Web App in the App Service Plan
    New-AzureRMWebApp -Name $AppName -ResourceGroupName $ResourceGroupName -Location $Location -AppServicePlan AppServiceMonitorPlan

    # Enable Logs
    Set-AzureRMWebApp -RequestTracingEnabled $True -HttpLoggingEnabled $True -DetailedErrorLoggingEnabled $True -ResourceGroupName $ResourceGroupName -Name $AppName

    # Make a Request
    Invoke-WebRequest -Method "Get" -Uri https://$AppName.chinacloudsites.cn/404 -ErrorAction SilentlyContinue

    # Download the Web App Logs
    #Get-AzureRMWebAppMetrics -ResourceGroupName $ResourceGroupName -Name $AppName -Metrics

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
| [Set-AzureRmWebApp](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.websites/v2.5.0/set-azurermwebapp) | 修改 Web 应用的配置。 |
| [Get-AzureRMWebAppMetrics](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.websites/v2.5.0/get-azurermwebappmetrics) | 获取 Web 应用的指标。 |

## <a name="next-steps"></a>后续步骤

有关 Azure PowerShell 模块的详细信息，请参阅 [Azure PowerShell 文档](https://docs.microsoft.com/zh-cn/powershell/azureps-cmdlets-docs/)。

可以在 [Azure PowerShell 示例](/documentation/articles/app-service-powershell-samples/)中找到 Azure 应用服务 Web 应用的其他 Azure Powershell 示例。