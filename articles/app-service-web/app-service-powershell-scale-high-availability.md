<properties
    pageTitle="Azure PowerShell 脚本示例 - 缩放具有高可用性体系结构的全球 Web 应用 | Azure"
    description="Azure PowerShell 脚本示例 - 缩放具有高可用性体系结构的全球 Web 应用"
    services="app-service\web"
    documentationcenter=""
    author="syntaxc4"
    manager="erikre"
    editor=""
    tags="azure-service-management"
    translationtype="Human Translation" />
<tags
    ms.assetid="470f0129-1efe-462c-a029-5c66e04158a8"
    ms.service="app-service"
    ms.devlang="multiple"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="web"
    ms.date="03/20/2017"
    wacn.date="04/24/2017"
    ms.author="cfowler"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="c0859923151489fd7de4c7c570299342e1ae4a04"
    ms.lasthandoff="04/14/2017" />

# <a name="scale-a-web-app-worldwide-with-a-high-availability-architecture"></a>缩放具有高可用性体系结构的全球 Web 应用

在此方案中，将创建一个资源组、两个应用服务计划、两个 Web 应用、一个流量管理器配置文件和两个流量管理器终结点。 完成本练习后，你将获得一个高可用性体系结构，它基于最低网络延迟提供 Web 应用的全局可用性。

必要时，请使用 [Azure PowerShell 指南](https://docs.microsoft.com/zh-cn/powershell/azureps-cmdlets-docs/)中的说明安装 Azure PowerShell，然后运行 `Login-AzureRmAccount -EnvironmentName AzureChinaCloud` 创建与 Azure 的连接。

## <a name="sample-script"></a>示例脚本

    # Generates a Random Value
    $Random=(New-Guid).ToString().Substring(0,8)

    # Variables
    $ResourceGroupName="myResourceGroup$Random"
    $App1Name="AppServiceTM1$Random"
    $App2Name="AppServiceTM2$Random"
    $Location1="ChinaNorth"
    $Location2="ChinaEast"

    # Create a Resource Group
    New-AzureRMResourceGroup -Name $ResourceGroupName -Location $Location1

    # Create Traffic Manager Profile
    New-AzureRMTrafficManagerProfile -Name "$ResourceGroupName-tmp" -ResourceGroupName $ResourceGroupName -TrafficRoutingMethod Performance -MonitorPath '/' -MonitorProtocol "HTTP" -RelativeDnsName $ResourceGroupName -Ttl 30 -MonitorPort 80

    # Create an App Service Plan
    New-AzureRMAppservicePlan -Name "$App1Name-Plan" -ResourceGroupName $ResourceGroupName -Location $Location1 -Tier Standard
    New-AzureRMAppservicePlan -Name "$App2Name-Plan" -ResourceGroupName $ResourceGroupName -Location $Location2 -Tier Standard

    # Create a Web App in the App Service Plan
    $App1ResourceId=(New-AzureRMWebApp -Name $App1Name -ResourceGroupName $ResourceGroupName -Location $Location1 -AppServicePlan "$App1Name-Plan").Id
    $App2ResourceId=(New-AzureRMWebApp -Name $App2Name -ResourceGroupName $ResourceGroupName -Location $Location2 -AppServicePlan "$App2Name-Plan").Id

    # Create Traffic Manager Endpoints for Web Apps
    New-AzureRMTrafficManagerEndpoint -Name "$App1Name-$Location1" -ResourceGroupName $ResourceGroupName -ProfileName "$ResourceGroupName-tmp" -Type AzureEndpoints -TargetResourceId $App1ResourceId -EndpointStatus "Enabled"
    New-AzureRMTrafficManagerEndpoint -Name "$App2Name-$Location2" -ResourceGroupName $ResourceGroupName -ProfileName "$ResourceGroupName-tmp" -Type AzureEndpoints -TargetResourceId $App2ResourceId -EndpointStatus "Enabled"

## <a name="clean-up-deployment"></a>清理部署 

运行脚本示例后，可以使用以下命令删除资源组、Web 应用以及所有相关资源。

    Remove-AzureRmResourceGroup -Name myResourceGroup -Force

## <a name="script-explanation"></a>脚本说明

此脚本使用以下命令。 表中的每条命令均链接到特定于命令的文档。

| 命令 | 说明 |
|---|---|
| [New-AzureRmResourceGroup](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/AzureRM.Resources/v3.5.0/new-azurermresourcegroup) | 创建用于存储所有资源的资源组。 |
| [New-AzureRMTrafficManagerProfile](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.trafficmanager/v2.5.0/new-azurermtrafficmanagerprofile) | 创建流量管理器配置文件。 |
| [New-AzureRmAppServicePlan](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.websites/v2.5.0/new-azurermappserviceplan) | 创建应用服务计划。 |
| [New-AzureRmWebApp](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.websites/v2.5.0/new-azurermwebapp) | 创建 Web 应用。 |
| [New-AzureRMTrafficManagerEndpoint](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.trafficmanager/v2.5.0/new-azurermtrafficmanagerendpoint) | 在流量管理器配置文件中创建终结点。 |

## <a name="next-steps"></a>后续步骤

有关 Azure PowerShell 模块的详细信息，请参阅 [Azure PowerShell 文档](https://docs.microsoft.com/zh-cn/powershell/azureps-cmdlets-docs/)。

可以在 [Azure PowerShell 示例](/documentation/articles/app-service-powershell-samples/)中找到 Azure 应用服务 Web 应用的其他 Azure Powershell 示例。