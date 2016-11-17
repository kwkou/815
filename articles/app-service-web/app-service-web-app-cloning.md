<properties
	pageTitle="使用 PowerShell 克隆 Web 应用"
	description="了解如何使用 PowerShell 将 Web 应用克隆到新的 Web 应用。"
	services="app-service\web"
	documentationCenter=""
	authors="ahmedelnably"
	manager="stefsch"
	editor=""/>

<tags
	ms.service="app-service-web"
	ms.workload="web"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="01/13/2016"
	wacn.date="09/26/2016"
	ms.author="ahmedelnably"/>  


# 使用 PowerShell 克隆 Azure App Service 应用#

在发行的 Azure PowerShell 版本 1.1.0 中，为 New-AzureRMWebApp 添加了新选项，可让用户将现有的 Web 应用克隆到不同区域或相同区域中新建的应用。这样，客户就可以跨不同区域部署许多应用。

应用克隆目前仅支持高级层 App Service 计划。新功能使用与 Web Apps 备份功能相同的限制，具体请参阅[在 Azure App Service 中备份 Web 应用](/documentation/articles/web-sites-backup/)。

[AZURE.INCLUDE [app-service-web-to-api-and-mobile](../../includes/app-service-web-to-api-and-mobile.md)]

若要了解如何使用基于 Azure Resource Manager 的 Azure PowerShell cmdlet 来管理 Web Apps，请查看[适用于 Azure Web 应用的基于 Azure Resource Manager 的 PowerShell 命令](/documentation/articles/app-service-web-app-azure-resource-manager-powershell/)

## 克隆现有应用 ##

方案：用户想要将位于中国东部区域的现有 Web 应用内容克隆到位于中国北部区域的新 Web 应用。结合 -SourceWebApp 选项使用 Azure Resource Manager 版本的 PowerShell cmdlet 来创建新的 Web 应用，即可实现此目的。

如果知道包含源 Web 应用的资源组名称，我们就可以使用以下 PowerShell 命令来获取源 Web 应用的信息（在本例中，该应用名为 source-webapp）：

    $srcapp = Get-AzureRmWebApp -ResourceGroupName SourceAzureResourceGroup -Name source-webapp

若要创建新的 App Service 计划，我们可以按以下示例中所示使用 New-AzureRmAppServicePlan 命令

	New-AzureRmAppServicePlan -Location "China East" -ResourceGroupName DestinationAzureResourceGroup -Name NewAppServicePlan -Tier Premium

使用 New-AzureRmWebApp 命令，我们可以在中国北部区域创建新的 Web 应用，并将它连接到现有的高级层 App Service 计划，而且我们可以使用相同的资源组作为源 Web 应用，或定义新的资源组，如下所示：

    $destapp = New-AzureRmWebApp -ResourceGroupName DestinationAzureResourceGroup -Name dest-webapp -Location "China North" -AppServicePlan DestinationAppServicePlan -SourceWebApp $srcapp

若要克隆现有的 Web 应用（包括所有关联的部署槽），用户必须使用 IncludeSourceWebAppSlots 参数。以下 PowerShell 命令演示了如何在 New-AzureRmWebApp 命令中配合使用该参数：

    $destapp = New-AzureRmWebApp -ResourceGroupName DestinationAzureResourceGroup -Name dest-webapp -Location "China North" -AppServicePlan DestinationAppServicePlan -SourceWebApp $srcapp -IncludeSourceWebAppSlots $true

若要在同一区域中克隆现有的 Web 应用，用户必须在同一区域中创建新资源组和新的 App Service 计划，然后使用以下 PowerShell 命令来克隆 Web 应用

    $destapp = New-AzureRmWebApp -ResourceGroupName NewAzureResourceGroup -Name dest-webapp -Location "China East" -AppServicePlan NewAppServicePlan -SourceWebApp $srcap

## 克隆现有的应用槽 ##

方案：用户想要将现有 Web 应用槽克隆到新的 Web 应用或新的 Web 应用槽。新的 Web 应用可与原始 Web 应用槽位于相同或不同的区域中。

如果知道包含源 Web 应用的资源组名称，我们就可以使用以下 PowerShell 命令来获取绑定到 Web 应用 source-webapp 的源 Web 应用槽信息（在本例中，该应用槽名为 source-webappslot）：

    $srcappslot = Get-AzureRmWebAppSlot -ResourceGroupName SourceAzureResourceGroup -Name source-webapp -Slot source-webappslot

下面演示了如何在新 Web 应用中创建源 Web 应用的克隆：

    $destapp = New-AzureRmWebApp -ResourceGroupName DestinationAzureResourceGroup -Name dest-webapp -Location "China North" -AppServicePlan DestinationAppServicePlan -SourceWebApp $srcappslot

## 在克隆应用时配置流量管理器 ##

创建多区域 Web 应用并配置 Azure 流量管理器以将流量路由到所有这些 Web 应用是一种确保客户应用高可用性的重要方案，当克隆现有的 Web 应用时，你可以选择将两个 Web 应用连接到新的流量管理器配置文件或现有的配置文件 — 请注意，仅支持 Azure Resource Manager 版本的流量管理器。

### 在克隆应用时创建新的流量管理器配置文件 ###

方案：用户想要将 Web 应用克隆到另一个区域，同时配置包含两个 Web 应用的 Azure Resource Manager 流量管理器配置文件。下面演示了如何在新 Web 应用中创建源 Web 应用的克隆，同时配置新的流量管理器配置文件：

    $destapp = New-AzureRmWebApp -ResourceGroupName DestinationAzureResourceGroup -Name dest-webapp -Location "China East" -AppServicePlan DestinationAppServicePlan -SourceWebApp $srcapp -TrafficManagerProfileName newTrafficManagerProfile

### 将新克隆的 Web 应用添加到现有的流量管理器配置文件 ###

方案：用户已有一个 Azure Resource Manager 流量管理器配置文件，现在想要将两个 Web 应用添加为终结点。为此，我们首先需要组合现有的流量管理器配置文件 ID，并需要订阅 ID、资源组名称和现有的流量管理器配置文件名称。

    $TMProfileID = "/subscriptions/<Your subscription ID goes here>/resourceGroups/<Your resource group name goes here>/providers/Microsoft.TrafficManagerProfiles/ExistingTrafficManagerProfileName"

获取流量管理器 ID 之后，根据以下演示在新 Web 应用中创建源 Web 应用的克隆，同时将它们添加到现有的流量管理器配置文件：

	$destapp = New-AzureRmWebApp -ResourceGroupName <Resource group name> -Name dest-webapp -Location "China East" -AppServicePlan DestinationAppServicePlan -SourceWebApp $srcapp -TrafficManagerProfileId $TMProfileID

## 当前限制 ##

此功能目前以预览版提供，我们正在不断地添加新功能。以下是当前版本的应用克隆已知存在的限制：

- 不会克隆自动缩放设置
- 不会克隆备份计划设置
- 不会克隆 VNET 设置
- 不会自动在目标 Web 应用上设置 App Insights
- 不会克隆简易身份验证设置
- 不会克隆 Kudu 扩展
- 不会克隆 TiP 规则
- 将不会克隆数据库内容


### 参考 ###
- [适用于 Azure Web 应用的基于 Azure Resource Manager 的 PowerShell 命令](/documentation/articles/app-service-web-app-azure-resource-manager-powershell/)
- [使用 Azure 门户预览克隆 Web 应用](/documentation/articles/app-service-web-app-cloning-portal/)
- [在 Azure App Service 中备份 Web 应用](/documentation/articles/web-sites-backup/)
- [Azure 流量管理器预览版对 Azure 资源管理器的支持](/documentation/articles/traffic-manager-powershell-arm/)
- [将 Azure PowerShell 与 Azure 资源管理器配合使用](/documentation/articles/powershell-azure-resource-manager/)

<!---HONumber=Mooncake_0815_2016-->