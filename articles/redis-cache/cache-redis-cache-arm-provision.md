<properties 
	pageTitle="预配 Redis 缓存" 
	description="使用 Azure 资源管理器模板部署 Azure Redis 缓存。" 
	services="app-service" 
	documentationCenter="" 
	authors="tfitzmac" 
	manager="wpickett" 
	editor=""/>

<tags
	ms.service="cache"
	ms.date="12/16/2015"
	wacn.date="08/29/2016"/>

# 使用模板创建 Redis 缓存

在本主题中，你将学习如何创建用于部署 Azure Redis 缓存的 Azure 资源管理器模板。你将了解如何定义要部署的资源以及如何定义执行部署时指定的参数。可将此模板用于自己的部署，或自定义此模板以满足要求。

有关创建模板的详细信息，请参阅[创作 Azure 资源管理器模板](/documentation/articles/resource-group-authoring-templates/)。

有关完整的模板，请参阅 [Redis 缓存模板](https://github.com/Azure/azure-quickstart-templates/blob/master/101-redis-cache/azuredeploy.json)。

>[AZURE.NOTE]适用于新[高级层](/documentation/articles/cache-premium-tier-intro/)的 ARM 模板现已推出。
>
>
>若要检查最新模板，请参阅 [Azure 快速入门模板](https://azure.microsoft.com/documentation/templates/)并搜索 `Redis Cache`。

## 将部署的内容

在此模板中，你将部署 Azure Redis 缓存。

## Parameters

使用 Azure 资源管理器，可以定义在部署模板时想要指定的值的参数。该模板具有一个名为 Parameters 的部分，其中包含所有参数值。你应该为随着要部署的项目或要部署到的环境而变化的值定义参数。不要为永远保持不变的值定义参数。每个参数值可在模板中用来定义所部署的资源。

下面介绍模板中的每个参数。

[AZURE.INCLUDE [app-service-web-deploy-redis-parameters](../../includes/cache-deploy-parameters.md)]

### redisCacheLocation

Redics 缓存的位置。为获得最佳性能，请使用要与缓存配合使用的应用所在的同一位置。

    "redisCacheLocation": {
      "type": "string"
    }

### enableNonSslPort

一个布尔值，该值指示是否允许通过非 SSL 端口访问。

    "enableNonSslPort": {
      "type": "bool"
    }
    
## 要部署的资源

### Redis Cache

创建 Azure Redis 缓存

    {
      "apiVersion": "2015-08-01",
      "name": "[parameters('redisCacheName')]",
      "type": "Microsoft.Cache/Redis",
      "location": "[parameters('redisCacheLocation')]",
      "properties": {
        "enableNonSslPort": "[parameters('enableNonSslPort')]",
        "sku": {
          "capacity": "[parameters('redisCacheCapacity')]",
          "family": "[parameters('redisCacheFamily')]",
          "name": "[parameters('redisCacheSKU')]"
        }
      },
        "resources": [
      ]
    }

## 运行部署的命令

[AZURE.INCLUDE [app-service-deploy-commands](../../includes/app-service-deploy-commands.md)]

### PowerShell

    New-AzureRmResourceGroupDeployment -TemplateFile path/to/azuredeploy.json -ResourceGroupName ExampleDeployGroup -redisCacheName ExampleCache -redisCacheLocation "China North"

### Azure CLI

    azure group deployment create --template-file path/to/azuredeploy.json -g ExampleDeployGroup

<!---HONumber=Mooncake_0104_2016-->