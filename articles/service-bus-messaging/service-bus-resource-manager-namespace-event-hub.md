<properties
    pageTitle="使用 Azure Resource Manager 模板创建包含事件中心和使用者组的服务总线命名空间 | Azure"
    description="使用 Azure Resource Manager 模板创建包含事件中心和使用者组的服务总线命名空间"
    services="service-bus"
    documentationCenter=".net"
    authors="sethmanheim"
    manager="timlt"
    editor=""/>

<tags
    ms.service="service-bus"
    ms.date="07/11/2016"
    wacn.date="08/15/2016"/>

# 使用 Azure Resource Manager 模板创建包含事件中心和使用者组的服务总线命名空间

本文介绍如何使用 Azure Resource Manager 模板创建包含事件中心和使用者组的服务总线命名空间。你将了解如何定义要部署的资源以及如何定义执行部署时指定的参数。可将此模板用于自己的部署，或自定义此模板以满足要求

有关创建模板的详细信息，请参阅[创作 Azure Resource Manager 模板][]。


>[AZURE.NOTE] 以下 Azure Resource Manager 模板可供下载和部署。
>
>-    [创建包含队列和授权规则的服务总线命名空间](/documentation/articles/service-bus-resource-manager-namespace-auth-rule/)
>-    [创建包含队列的服务总线命名空间](/documentation/articles/service-bus-resource-manager-namespace-queue/)
>-    [创建包含主题和订阅的服务总线命名空间](/documentation/articles/service-bus-resource-manager-namespace-topic/)
>-    [创建服务总线命名空间](/documentation/articles/service-bus-resource-manager-namespace/)
>
>若要检查最新模板，请访问 [Azure 快速入门模板][] 库并搜索服务总线。

## 你将部署什么内容？

使用此模板，你将部署包含事件中心和使用者组的服务总线命名空间。

[事件中心](/documentation/articles/event-hubs-what-is-event-hubs/)是一种事件处理服务，用于向 Azure 提供大规模的事件与遥测数据输入，并且具有较低的延迟和较高的可靠性。

若要自动运行部署，请单击以下按钮：

[![部署到 Azure](./media/service-bus-resource-manager-namespace-event-hub/deploybutton.png)](https://portal.azure.cn/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F201-servicebus-create-eventhub-and-consumergroup%2Fazuredeploy.json)

## Parameters

使用 Azure 资源管理器，可以定义在部署模板时想要指定的值的参数。该模板具有一个名为 `Parameters` 的部分，其中包含所有参数值。你应该为随着要部署的项目或要部署到的环境而变化的值定义参数。不要为永远保持不变的值定义参数。每个参数值可在模板中用来定义所部署的资源。

模板定义以下参数。

### serviceBusNamespaceName

要创建的服务总线命名空间的名称。


    "serviceBusNamespaceName": {
    "type": "string"
    }


### serviceBusEventHubName

在服务总线命名空间中创建的事件中心的名称。


    "serviceBusEventHubName": {
    "type": "string"
    }


### serviceBusConsumerGroupName

为服务总线命名空间中的事件中心创建的使用者组的名称。


    "serviceBusConsumerGroupName": {
    "type": "string"
    }


### serviceBusApiVersion

模板的服务总线 API 版本。


    "serviceBusApiVersion": {
    "type": "string"
    }


## 要部署的资源

创建包含事件中心和使用者组的**事件中心**类型的服务总线命名空间。


    "resources": [
            {
                "apiVersion": "[variables('ehVersion')]",
                "name": "[parameters('serviceBusNamespaceName')]",
                "type": "Microsoft.ServiceBus/Namespaces",
                "location": "[variables('location')]",
                "kind": "EventHub",
                "sku": {
                    "name": "StandardSku",
                    "tier": "Standard"
                },
                "resources": [
                    {
                        "apiVersion": "[variables('ehVersion')]",
                        "name": "[parameters('serviceBusEventHubName')]",
                        "type": "EventHubs",
                        "dependsOn": [
                            "[concat('Microsoft.ServiceBus/namespaces/', parameters('serviceBusNamespaceName'))]"
                        ],
                        "properties": {
                            "path": "[parameters('serviceBusEventHubName')]"
                        },
                        "resources": [
                            {
                                "apiVersion": "[variables('ehVersion')]",
                                "name": "[parameters('serviceBusConsumerGroupName')]",
                                "type": "ConsumerGroups",
                                "dependsOn": [
                                    "[parameters('serviceBusEventHubName')]"
                                ],
                                "properties": {
                                }
                            }
                        ]
                    }
                ]
            }
        ]


## 运行部署的命令

[AZURE.INCLUDE [app-service-deploy-commands](../../includes/app-service-deploy-commands.md)]

## PowerShell


    New-AzureRmResourceGroupDeployment -ResourceGroupName \<resource-group-name\> -TemplateFile https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/201-servicebus-create-eventhub-and-consumergroup/azuredeploy.json


## Azure CLI


    azure config mode arm
    
    azure group deployment create \<my-resource-group\> \<my-deployment-name\> --template-uri [https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/201-servicebus-create-eventhub-and-consumergroup/azuredeploy.json][]


## 后续步骤

现在，你已使用 Azure Resource Manager 创建并部署了资源，请通过查看以下文章了解如何管理这些资源：

- [使用 Azure 自动化管理 Azure Service Bus](/documentation/articles/service-bus-management-libraries/)
- [使用 PowerShell 管理事件中心](https://docs.microsoft.com/en-us/powershell/module/azurerm.servicebus/?view=azurermps-3.8.0)
- [使用服务总线资源管理器管理事件中心资源](https://code.msdn.microsoft.com/Service-Bus-Explorer-f2abca5a)

  [创作 Azure Resource Manager 模板]: /documentation/articles/resource-group-authoring-templates/
  [Using Azure PowerShell with Azure Resource Manager]: /documentation/articles/powershell-azure-resource-manager/
  [Using the Azure CLI for Mac, Linux, and Windows with Azure Resource Management]: /documentation/articles/xplat-cli-azure-resource-manager/
  [服务总线事件中心和使用者组模板]: https://github.com/Azure/azure-quickstart-templates/blob/master/201-servicebus-create-eventhub-and-consumergroup/

<!---HONumber=Mooncake_0808_2016-->