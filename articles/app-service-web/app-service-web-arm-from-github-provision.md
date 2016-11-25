<properties 
	pageTitle="部署链接到 GitHub 存储库的网站" 
	description="使用 Azure 资源管理器模板来部署包含 GitHub 存储库中项目的网站。" 
	services="app-service" 
	documentationCenter="" 
	authors="cephalin" 
	manager="wpickett" 
	editor=""/>

<tags 
	ms.service="app-service" 
	ms.workload="na" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="04/27/2016" 
	wacn.date="11/25/2016" 
	ms.author="cephalin"/>

# 部署链接到 GitHub 存储库的网站

[AZURE.INCLUDE [azure-sdk-developer-differences](../../includes/azure-sdk-developer-differences.md)]

在本主题中，你将学习如何创建 Azure 资源管理器模板，该模板部署链接到 GitHub 存储库中项目的网站。你将了解如何定义要部署的资源以及如何定义执行部署时指定的参数。可将此模板用于自己的部署，或自定义此模板以满足要求。

有关创建模板的详细信息，请参阅[创作 Azure 资源管理器模板](/documentation/articles/resource-group-authoring-templates/)。

有关完整的模板，请参阅[链接到 GitHub 的网站模板](https://github.com/Azure/azure-quickstart-templates/blob/master/201-web-app-github-deploy/azuredeploy.json)。

[AZURE.INCLUDE [app-service-web-to-api-and-mobile](../../includes/app-service-web-to-api-and-mobile.md)] 

## 将部署的内容

使用此模板，你将部署包含 GitHub 中项目代码的网站。

## Parameters

[AZURE.INCLUDE [app-service-web-deploy-web-parameters](../../includes/app-service-web-deploy-web-parameters.md)]

### repoURL

包含要部署项目的 GitHub 存储库的 URL。此参数包含默认值，但此值仅用于向你展示如何提供存储库的 URL。测试模板时可以使用此值，但使用模板时你将需要为自己的存储库提供 URL。

    "repoURL": {
        "type": "string",
        "defaultValue": "https://github.com/davidebbo-test/Mvc52Application.git"
    }

### branch

部署应用程序时要使用的存储库的分支。默认值为 master，但也可输入你想要部署的存储库中任何分支的名称。

    "branch": {
        "type": "string",
        "defaultValue": "master"
    }
    
## 要部署的资源

[AZURE.INCLUDE [app-service-web-deploy-web-host](../../includes/app-service-web-deploy-web-host.md)]

### 网站

创建链接到 GitHub 中项目的网站。

通过 **siteName** 参数指定网站的名称，通过 **siteLocation** 参数指定网站的位置。在 **dependsOn** 元素中，该模板将网站定义为依赖服务托管计划。因为它依赖托管计划，所以只有当托管计划创建完成后才会创建网站。**dependsOn** 元素仅用于指定部署顺序。如果未将网站标记为依赖托管计划，Azure 资源管理器将尝试同时创建两个资源，而如果在创建托管计划之前创建了网站，则可能会接收到错误。

网站还具有一个子资源，在以下**资源**部分中对其进行定义。此子资源为使用网站部署的项目定义源代码管理。在此模板中，源代码管理链接到特定的 GitHub 存储库。使用代码 **"RepoUrl":"https://github.com/davidebbo-test/Mvc52Application.git"** 定义 GitHub 存储库。如果要使用最少的参数创建可重复部署单个项目的模板，可以对存储库 URL 进行硬编码。
如果不对存储库 URL 进行硬编码，可为存储库 URL 添加一个参数，并将该值用于 **RepoUrl** 属性。

    {
      "apiVersion": "2015-08-01",
      "name": "[parameters('siteName')]",
      "type": "Microsoft.Web/sites",
      "location": "[resourceGroup().location]",
      "dependsOn": [
        "[resourceId('Microsoft.Web/serverfarms', parameters('hostingPlanName'))]"
      ],
      "properties": {
        "serverFarmId": "[parameters('hostingPlanName')]"
      },
      "resources": [
        {
          "apiVersion": "2015-08-01",
          "name": "web",
          "type": "sourcecontrols",
          "dependsOn": [
            "[resourceId('Microsoft.Web/Sites', parameters('siteName'))]"
          ],
          "properties": {
            "RepoUrl": "[parameters('repoURL')]",
            "branch": "[parameters('branch')]",
            "IsManualIntegration": true
          }
        }
      ]
    }

>[AZURE.NOTE] 在 Azure 中国区，我们暂时无法在新门户设置 GitHub 凭据，因此`IsManualIntegration` 必须设置为 true。

## 运行部署的命令

[AZURE.INCLUDE [app-service-deploy-commands](../../includes/app-service-deploy-commands.md)]

### PowerShell

    New-AzureRmResourceGroupDeployment -TemplateFile path/to/azuredeploy.json -siteName ExampleSite -hostingPlanName ExamplePlan -siteLocation "China North" -ResourceGroupName ExampleDeployGroup

### Azure CLI

    azure group deployment create --template-file path/to/azuredeploy.json


 

<!---HONumber=Mooncake_0118_2016-->