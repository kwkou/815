<properties 
	pageTitle="使用 Redis 缓存预配 Web 应用" 
	description="使用 Azure 资源管理器模板部署包含 Redis 缓存的 Web 应用。" 
	services="app-service" 
	documentationCenter="" 
	authors="steved0x" 
	manager="erickson-doug" 
	editor=""/>

<tags
	ms.service="app-service"
	ms.date="03/04/2016"
	wacn.date="05/30/2016"/>

# 使用模板创建 Web 应用和 Redis 缓存

在本主题中，你将学习如何创建用于部署 Azure Web 应用和 Redis 缓存的 Azure 资源管理器模板。你将了解如何定义要部署的资源以及如何定义执行部署时指定的参数。可将此模板用于自己的部署，或自定义此模板以满足要求。

有关创建模板的详细信息，请参阅[创作 Azure 资源管理器模板](/documentation/articles/resource-group-authoring-templates/)。

有关完整的模板，请参阅[包含 Redis 缓存的 Web 应用模板](https://github.com/Azure/azure-quickstart-templates/blob/master/201-web-app-with-redis-cache/azuredeploy.json)。

## 将部署的内容

在此模板中，你将部署：

- Azure Web 应用
- Azure Redis 缓存。

## 要指定的参数

[AZURE.INCLUDE [app-service-web-deploy-web-parameters](../includes/app-service-web-deploy-web-parameters.md)]

[AZURE.INCLUDE [cache-deploy-parameters](../includes/cache-deploy-parameters.md)]

## 名称变量

此模板使用变量来构造资源的名称。它使用 [uniqueString](/documentation/articles/resource-group-template-functions/#uniquestring) 函数来构造基于资源组 ID 的值。

    "variables": {
      "hostingPlanName": "[concat('hostingplan', uniqueString(resourceGroup().id))]",
      "webSiteName": "[concat('webSite', uniqueString(resourceGroup().id))]",
      "cacheName": "[concat('cache', uniqueString(resourceGroup().id))]"
    },


## 要部署的资源

[AZURE.INCLUDE [app-service-web-deploy-web-host](../includes/app-service-web-deploy-web-host.md)]

### Redis Cache

创建用于 Web 应用的 Azure Redis 缓存。缓存的名称在 **cacheName** 变量中指定。

该模板将在资源组所在的同一位置中创建缓存。

    {
      "name": "[variables('cacheName')]",
      "type": "Microsoft.Cache/Redis",
      "location": "[resourceGroup().location]",
      "apiVersion": "2015-08-01",
      "dependsOn": [ ],
      "tags": {
        "displayName": "cache"
      },
      "properties": {
        "sku": {
          "name": "[parameters('cacheSKUName')]",
          "family": "[parameters('cacheSKUFamily')]",
          "capacity": "[parameters('cacheSKUCapacity')]"
        }
      }
    }


### Web 应用

使用 **webSiteName** 变量中指定的名称创建 Web 应用。

请注意，在 Web 应用中配置的应用设置属性使其可与 Redis 缓存配合工作。此应用设置是根据部署期间提供了值动态创建的。
        
    {
      "apiVersion": "2015-08-01",
      "name": "[variables('webSiteName')]",
      "type": "Microsoft.Web/sites",
      "location": "[resourceGroup().location]",
      "dependsOn": [
        "[concat('Microsoft.Web/serverFarms/', variables('hostingPlanName'))]",
        "[concat('Microsoft.Cache/Redis/', variables('cacheName'))]"
      ],
      "tags": {
        "[concat('hidden-related:', resourceGroup().id, '/providers/Microsoft.Web/serverfarms/', variables('hostingPlanName'))]": "empty",
        "displayName": "Website"
      },
      "properties": {
        "name": "[variables('webSiteName')]",
        "serverFarmId": "[resourceId('Microsoft.Web/serverfarms', variables('hostingPlanName'))]"
      },
      "resources": [
        {
          "apiVersion": "2015-08-01",
          "type": "config",
          "name": "appsettings",
          "dependsOn": [
            "[concat('Microsoft.Web/Sites/', variables('webSiteName'))]",
            "[concat('Microsoft.Cache/Redis/', variables('cacheName'))]"
          ],
          "properties": {
            "CacheConnection": "[concat(variables('cacheName'),'.redis.cache.chinacloudapi.cn,abortConnect=false,ssl=true,password=', listKeys(resourceId('Microsoft.Cache/Redis', variables('cacheName')), '2015-08-01').primaryKey)]"
          }
        }
      ]
    }

## 运行部署的命令

[AZURE.INCLUDE [app-service-deploy-commands](../includes/app-service-deploy-commands.md)]

### PowerShell

    New-AzureRmResourceGroupDeployment -TemplateUri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-web-app-with-redis-cache/azuredeploy.json -ResourceGroupName ExampleDeployGroup

### Azure CLI

    azure group deployment create --template-uri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-web-app-with-redis-cache/azuredeploy.json -g ExampleDeployGroup



<!---HONumber=Mooncake_0523_2016-->