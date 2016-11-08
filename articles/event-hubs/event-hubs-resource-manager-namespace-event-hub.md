<properties
    pageTitle="使用 Azure Resource Manager 模板创建包含事件中心和使用者组的事件中心命名空间 | Azure"
    description="使用 Azure Resource Manager 模板创建包含事件中心和使用者组的事件中心命名空间"
    services="event-hubs"
    documentationCenter=".net"
    authors="sethmanheim"
    manager="timlt"
    editor=""/>  


<tags
    ms.service="event-hubs"
    ms.devlang="tbd"
    ms.topic="article"
    ms.tgt_pltfrm="dotnet"
    ms.workload="na"
    ms.date="08/31/2016"
    wacn.date="11/08/2016"/>  


# 使用 Azure Resource Manager 模板创建包含事件中心和使用者组的事件中心命名空间

本文介绍如何使用 Azure Resource Manager 模板创建包含事件中心和使用者组的事件中心命名空间。你将了解如何定义要部署的资源以及如何定义执行部署时指定的参数。可将此模板用于自己的部署，或自定义此模板以满足要求

有关创建模板的详细信息，请参阅[创作 Azure Resource Manager 模板][]。

有关完整的模板，请参阅 GitHub 上的[事件中心和使用者组模板][]。

>[AZURE.NOTE]
若要检查最新模板，请访问 [Azure 快速启动模板][]库并搜索事件中心。

## 你将部署什么内容？

使用此模板，你将部署包含事件中心和使用者组的事件中心命名空间。

[事件中心](/documentation/articles/event-hubs-what-is-event-hubs/)是一种事件处理服务，用于向 Azure 提供大规模的事件与遥测数据入口，并且具有较低的延迟和较高的可靠性。

## 参数

使用 Azure 资源管理器，可以定义在部署模板时想要指定的值的参数。该模板具有一个名为 `Parameters` 的部分，其中包含所有参数值。你应该为随着要部署的项目或要部署到的环境而变化的值定义参数。不要为永远保持不变的值定义参数。每个参数值可在模板中用来定义所部署的资源。

模板定义以下参数。

### eventHubNamespaceName

要创建的事件中心命名空间的名称。

```
"eventHubNamespaceName": {
"type": "string"
}
```

### eventHubName

在事件中心命名空间中创建的事件中心的名称。

```
"eventHubName": {
"type": "string"
}
```

### eventHubConsumerGroupName

为事件中心创建的使用者组的名称。

```
"eventHubConsumerGroupName": {
"type": "string"
}
```

### apiVersion

模板的 API 版本。

```
"apiVersion": {
"type": "string"
}
```

## 要部署的资源

创建包含事件中心和使用者组的 **EventHubs** 类型的命名空间。

```
"resources":[  
      {  
         "apiVersion":"[variables('ehVersion')]",
         "name":"[parameters('namespaceName')]",
         "type":"Microsoft.EventHub/Namespaces",
         "location":"[variables('location')]",
         "sku":{  
            "name":"Standard",
            "tier":"Standard"
         },
         "resources":[  
            {  
               "apiVersion":"[variables('ehVersion')]",
               "name":"[parameters('eventHubName')]",
               "type":"EventHubs",
               "dependsOn":[  
                  "[concat('Microsoft.EventHub/namespaces/', parameters('namespaceName'))]"
               ],
               "properties":{  
                  "path":"[parameters('eventHubName')]"
               },
               "resources":[  
                  {  
                     "apiVersion":"[variables('ehVersion')]",
                     "name":"[parameters('consumerGroupName')]",
                     "type":"ConsumerGroups",
                     "dependsOn":[  
                        "[parameters('eventHubName')]"
                     ],
                     "properties":{  

                     }
                  }
               ]
            }
         ]
      }
   ],
```

## 运行部署的命令

[AZURE.INCLUDE [app-service-deploy-commands](../../includes/app-service-deploy-commands.md)]

## PowerShell

```
New-AzureRmResourceGroupDeployment -ResourceGroupName \<resource-group-name\> -TemplateFile https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/201-event-hubs-create-event-hub-and-consumer-group/azuredeploy.json
```

## Azure CLI

```
azure config mode arm

azure group deployment create \<my-resource-group\> \<my-deployment-name\> --template-uri [https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/201-event-hubs-create-event-hub-and-consumer-group/azuredeploy.json][]
```

[创作 Azure Resource Manager 模板]: /documentation/articles/resource-group-authoring-templates
[将 Azure PowerShell 与 Azure 资源管理器配合使用]: /documentation/articles/powershell-azure-resource-manager
[使用 Azure CLI 管理 Azure 资源和资源组]: /documentation/articles/xplat-cli-azure-resource-manager
[事件中心和使用者组模板]: https://github.com/Azure/azure-quickstart-templates/blob/master/201-event-hubs-create-event-hub-and-consumer-group/

<!---HONumber=Mooncake_1031_2016-->