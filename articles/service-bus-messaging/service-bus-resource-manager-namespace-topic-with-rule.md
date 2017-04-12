<properties
	pageTitle="使用模板创建 Azure 服务总线主题订阅和规则"
	description="使用 Azure Resource Manager 模板创建包含主题、订阅和规则的服务总线命名空间"
	services=" service-bus"
	documentationcenter=" .net"
	author="sethmanheim"
	manager="timlt"
	editor="" />  


<tags
	ms.date="01/18/2017"
	ms.service = "service-bus"
	wacn.date="03/20/2017"/>  

	

# 使用 Azure Resource Manager 模板创建包含主题、订阅和规则的服务总线命名空间
本文介绍如何使用 Azure Resource Manager 模板创建包含主题、订阅和规则（筛选器）的服务总线命名空间。你将了解如何定义要部署的资源以及如何定义执行部署时指定的参数。可将此模板用于自己的部署，或自定义此模板以满足要求

有关创建模板的详细信息，请参阅[创作 Azure Resource Manager 模板][Authoring Azure Resource Manager templates]。

有关 Azure 资源命名约定的实践和模式的详细信息，请参阅 [Azure 资源的建议命名约定][Azure Resources Naming Conventions]。

有关完整的模板，请参阅[包含主题、订阅和规则的服务总线命名空间][Service Bus namespace with topic, subscription, and rule]模板。

> [AZURE.NOTE]
以下 Azure Resource Manager 模板可供下载和部署。
> 
>-  [创建包含队列和授权规则的服务总线命名空间](/documentation/articles/service-bus-resource-manager-namespace-auth-rule/)
>-  [创建包含队列的服务总线命名空间](/documentation/articles/service-bus-resource-manager-namespace-queue/)
>-  [创建服务总线命名空间](/documentation/articles/service-bus-resource-manager-namespace/)
>-  [创建包含主题和订阅的服务总线命名空间](/documentation/articles/service-bus-resource-manager-namespace-topic/)

## 你将部署什么内容？
使用此模板，可以部署包含主题、订阅和规则（筛选器）的服务总线命名空间。

[服务总线主题和订阅](/documentation/articles/service-bus-queues-topics-subscriptions/#topics-and-subscriptions)以“发布/订阅”模式提供一对多的通信形式。使用主题和订阅时，分布式应用程序的组件不直接相互通信，而是通过充当中介的主题交换消息。对主题的订阅类似于接收发送到主题的消息副本的虚拟队列。针对订阅的筛选器可以指定发送到主题的哪些消息应该在特定主题订阅中显示。

## 什么是规则（筛选器）？
在许多情况下，必须以不同方式处理具有特定特征的消息。若要启用此功能，你可以将订阅配置为查找具有所需属性的消息，然后执行这些属性的部分修改操作。虽然服务总线订阅可以看到发送到主题的所有消息，但你仅可以将这些消息的一个子集复制到虚拟订阅队列。使用订阅筛选器完成此操作。若要了解有关规则（筛选器）的详细信息，请参阅[服务总线队列、主题和订阅][Service Bus queues, topics, and subscriptions]。

若要自动运行部署，请单击以下按钮：

[![部署到 Azure](./media/service-bus-resource-manager-namespace-topic/deploybutton.png)](https://portal.azure.cn/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F201-servicebus-create-topic-subscription-rule%2Fazuredeploy.json)

## 参数
使用 Azure Resource Manager，可以定义在部署模板时想要指定的值的参数。该模板具有一个名为 `Parameters` 的部分，其中包含所有参数值。你应该为随着要部署的项目或要部署到的环境而变化的值定义参数。不要为永远保持不变的值定义参数。每个参数值可在模板中用来定义所部署的资源。

模板定义以下参数：

### serviceBusNamespaceName
要创建的服务总线命名空间的名称。

		"serviceBusNamespaceName": {
		"type": "string"
		}

### serviceBusTopicName
在服务总线命名空间中创建的主题的名称。

		"serviceBusTopicName": {
		"type": "string"
		}

### serviceBusSubscriptionName
在服务总线命名空间中创建的订阅的名称。

		"serviceBusSubscriptionName": {
		"type": "string"
		}
### serviceBusRuleName
在服务总线命名空间中创建的规则（筛选器）的名称。

		"serviceBusRuleName": {
		"type": "string",
		}
### serviceBusApiVersion
模板的服务总线 API 版本。


		"serviceBusApiVersion": {
		"type": "string"
		}

## 要部署的资源
创建类型为 **Messaging** 的包含主题、订阅和规则的标准服务总线命名空间。


		 "resources": [{
		        "apiVersion": "[variables('sbVersion')]",
		        "name": "[parameters('serviceBusNamespaceName')]",
		        "type": "Microsoft.ServiceBus/Namespaces",
		        "location": "[variables('location')]",
		        "sku": {
		            "name": "Standard",
		            "tier": "Standard"
		        },
		        "resources": [{
		            "apiVersion": "[variables('sbVersion')]",
		            "name": "[parameters('serviceBusTopicName')]",
		            "type": "Topics",
		            "dependsOn": [
		                "[concat('Microsoft.ServiceBus/namespaces/', parameters('serviceBusNamespaceName'))]"
		            ],
		            "properties": {
		                "path": "[parameters('serviceBusTopicName')]"
		            },
		            "resources": [{
		                "apiVersion": "[variables('sbVersion')]",
		                "name": "[parameters('serviceBusSubscriptionName')]",
		                "type": "Subscriptions",
		                "dependsOn": [
		                    "[parameters('serviceBusTopicName')]"
		                ],
		                "properties": {},
		                "resources": [{
		                    "apiVersion": "[variables('sbVersion')]",
		                    "name": "[parameters('serviceBusRuleName')]",
		                    "type": "Rules",
		                    "dependsOn": [
		                        "[parameters('serviceBusSubscriptionName')]"
		                    ],
		                    "properties": {
		                        "filter": {
		                            "sqlExpression": "StoreName = 'Store1'"
		                        },
		                        "action": {
		                            "sqlExpression": "set FilterTag = 'true'"
		                        }
		                    }
		                }]
		            }]
		        }]
		    }]


## 运行部署的命令
[AZURE.INCLUDE [app-service-deploy-commands](../../includes/app-service-deploy-commands.md)]

## PowerShell
		New-AzureResourceGroupDeployment -Name <deployment-name> -ResourceGroupName <resource-group-name> -TemplateUri <https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/201-servicebus-create-topic-subscription-rule/azuredeploy.json>

## Azure CLI
		azure config mode arm

		azure group deployment create <my-resource-group> <my-deployment-name> --template-uri <https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/201-servicebus-create-topic-subscription-rule/azuredeploy.json>

## 后续步骤
现在，你已使用 Azure Resource Manager 创建并部署了资源，请通过查看以下文章了解如何管理这些资源：

* [使用 Azure 自动化管理 Azure Service Bus](/documentation/articles/service-bus-management-libraries/)
* [使用 PowerShell 管理服务总线](https://docs.microsoft.com/en-us/powershell/resourcemanager/azurerm.servicebus/v0.0.2/azurerm.servicebus/)
* [使用服务总线资源管理器管理服务总线资源](https://github.com/paolosalvatori/ServiceBusExplorer/releases)

[Authoring Azure Resource Manager templates]: /documentation/articles/resource-group-authoring-templates/
[Learn more about Service Bus topics and subscriptions]: /documentation/articles/service-bus-queues-topics-subscriptions/
[Using Azure PowerShell with Azure Resource Manager]: /documentation/articles/powershell-azure-resource-manager/
[Using the Azure CLI for Mac, Linux, and Windows with Azure Resource Management]: /documentation/articles/xplat-cli-azure-resource-manager/
[Azure Resources Naming Conventions]: https://docs.microsoft.com/en-us/azure/architecture/best-practices/naming-conventions/
[Service Bus namespace with topic, subscription, and rule]: https://github.com/Azure/azure-quickstart-templates/blob/master/201-servicebus-create-topic-subscription-rule/
[Service Bus queues, topics, and subscriptions]: /documentation/articles/service-bus-queues-topics-subscriptions/

<!---HONumber=Mooncake_0313_2017-->