<properties
    pageTitle="在 Azure 应用服务上进行高密度托管 | Azure"
    description="在 Azure 应用服务上进行高密度托管"
    author="btardif"
    manager="wpickett"
    editor=""
    services="app-service\web"
    documentationcenter="" />
<tags
    ms.assetid="a903cb78-4927-47b0-8427-56412c4e3e64"
    ms.service="app-service-web"
    ms.workload="web"
    ms.tgt_pltfrm="na"
    ms.devlang="multiple"
    ms.topic="article"
    ms.date="01/11/2017"
    wacn.date="02/24/2017"
    ms.author="byvinyal" />  


# 在 Azure 应用服务上进行高密度托管
使用应用服务时，你的应用程序通过两个概念与分配给它的容量解耦：

* **应用程序：**代表应用及其运行时配置。例如，它包括运行时应该加载的 .NET 的版本、应用设置等。
* **应用服务计划：**定义容量的特征、可用功能集以及应用程序的地区性。例如，特征可能包括：大型（四核）机、四实例、中国东部的高级功能。

一个应用始终与一个应用服务计划相关联，但一个应用服务计划则可能为一个或多个应用提供容量。

因此，平台既可以让单个应用处于隔离状态，也可以让多个应用通过共享应用服务计划来共享资源，具有相当的灵活性。

但是，当多个应用共享一个应用服务计划时，该应用的实例会在该应用服务计划的每个实例上运行。

## 按应用缩放
*按应用缩放*：一项功能，在应用服务计划级别启用后即可用于每个应用程序。

按应用缩放在缩放应用时独立于所属的应用服务计划。可通过这种方式将应用服务计划配置为提供 10 个实例，而应用则可设置为仅缩放成其中 5 个的规模。

>[AZURE.NOTE]
>“按应用扩展”仅可用于**高级** SKU 应用服务计划
>

### 使用 PowerShell 进行按应用缩放

可以创建一个新计划并将其配置为*按应用缩放* 计划，方法是将 ```-perSiteScaling $true``` 属性传递给
```New-AzureRmAppServicePlan``` commandlet

    New-AzureRmAppServicePlan -ResourceGroupName $ResourceGroup -Name $AppServicePlan `
                                -Location $Location `
                                -Tier Premium -WorkerSize Small `
                                -NumberofWorkers 5 -PerSiteScaling $true

如果要更新现有应用服务计划以使用此功能：

- 请获取目标计划 ```Get-AzureRmAppServicePlan```
- 在本地修改属性 ```$newASP.PerSiteScaling = $true```
- 将所做的更改发布回 Azure ```Set-AzureRmAppServicePlan```

        # Get the new App Service Plan and modify the "PerSiteScaling" property.
        $newASP = Get-AzureRmAppServicePlan -ResourceGroupName $ResourceGroup -Name $AppServicePlan
        $newASP

        #Modify the local copy to use "PerSiteScaling" property.
        $newASP.PerSiteScaling = $true
        $newASP
    
        #Post updated app service plan back to azure
        Set-AzureRmAppServicePlan $newASP

配置好计划以后，即可设置每个应用的最大实例数。

在下面的示例中，应用的最大实例数为 2，不管基础应用服务计划要扩展到多少个实例。

        # Get the app we want to configure to use "PerSiteScaling"
        $newapp = Get-AzureRmWebApp -ResourceGroupName $ResourceGroup -Name $webapp
    
        # Modify the NumberOfWorkers setting to the desired value.
        $newapp.SiteConfig.NumberOfWorkers = 2
    
        # Post updated app back to azure
        Set-AzureRmWebApp $newapp

### 使用 Azure Resource Manager 进行按应用缩放

以下 *Azure Resource Manager 模板*创建：

- 扩展到 10 个实例的应用服务计划
- 已配置为最多可扩展到 5 个实例的应用。

该应用服务计划将 **PerSiteScaling** 属性设置为 true ```"perSiteScaling": true```。该应用将要使用的**辅助角色数**设置为 5 ```"properties": { "numberOfWorkers": "5" }```。

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
                    "name": "P1",
                    "tier": "Premium",
                    "size": "P1",
                    "family": "P",
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
                        "properties": { "numberOfWorkers": "5" }
                 } ]
             }]
        }

<!---HONumber=Mooncake_0220_2017-->
<!--Update_Description: add solution for PowerShell-->