<properties
	pageTitle="在 Azure 应用服务上进行高密度托管 | Azure"
	description="在 Azure 应用服务上进行高密度托管"
	authors="btardif"
	manager="wpickett"
	editor=""
	services="app-service\web"
	documentationCenter=""/>

<tags
	ms.service="app-service-web"
	ms.date="08/07/2016"
	wacn.date=""/>

# 在 Azure 应用服务上进行高密度托管#

使用 Azure 应用服务时，应用程序会与所分配的容量脱离关系，具体表现在两个概念：

- **应用程序：**代表应用及其运行时配置。例如，它包括运行时应该加载的 .NET 的版本、应用设置等。

- **应用服务计划：**定义容量的特征、可用功能集以及应用程序的地区性。例如，特征可能包括：大型（四核）机、四实例、中国东部的高级功能。

一个应用始终与一个应用服务计划相关联，但一个应用服务计划则可能为一个或多个应用提供容量。

这意味着，平台既可以让单个应用处于隔离状态，也可以让多个应用通过共享应用服务计划来共享资源，具有相当的灵活性。

但是，当多个应用共享一个应用服务计划时，该应用的实例会在该应用服务计划的每个实例上运行。

## 按应用缩放 ##
*按应用缩放* ：一项功能，在应用服务计划级别启用后即可用于每个应用程序。

按应用缩放在缩放应用时独立于所属的应用服务计划。可通过这种方式将应用服务计划配置为提供 10 个实例，而应用则可设置为仅缩放成其中 5 个的规模。

以下 Azure Resource Manager 模板所创建的应用服务计划将扩展成 10 个实例，而所创建的应用将配置为使用按应用缩放，仅缩放成 5 个实例。

为此，应用服务计划需将“按站点缩放”属性设置为 true (`"perSiteScaling": true`)，应用需将要使用的“辅助进程数”设置为 1 (`"properties": { "numberOfWorkers": "1" }`)。

    {
        "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters":{
            "appServicePlanName": { "type": "string" },
            "appName": { "type": "string" }
         },
        "resources": [
        {
            "comments": "App Service Plan with per site perSiteScaling = true",
            "type": "Microsoft.Web/serverFarms",
            "sku": {
                "name": "S1",
                "tier": "Standard",
                "size": "S1",
                "family": "S",
                "capacity": 10
                },
            "name": "[parameters('appServicePlanName')]",
            "apiVersion": "2015-08-01",
            "location": "China North",
            "properties": {
                "name": "[parameters('appServicePlanName')]",
                "perSiteScaling": true
            }
        },
        {
            "type": "Microsoft.Web/sites",
            "name": "[parameters('appName')]",
            "apiVersion": "2015-08-01-preview",
            "location": "China North",
            "dependsOn": [ "[resourceId('Microsoft.Web/serverFarms', parameters('appServicePlanName'))]" ],
            "properties": { "serverFarmId": "[resourceId('Microsoft.Web/serverFarms', parameters('appServicePlanName'))]" },
            "resources": [ {
                    "comments": "",
                    "type": "config",
                    "name": "web",
                    "apiVersion": "2015-08-01",
                    "location": "China North",
                    "dependsOn": [ "[resourceId('Microsoft.Web/Sites', parameters('appName'))]" ],
                    "properties": { "numberOfWorkers": "1" }
             } ]
         }]
    }


<!---HONumber=Mooncake_0919_2016-->