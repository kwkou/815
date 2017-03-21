<properties
    pageTitle="使用 Azure Resource Manager 模板创建服务总线授权规则 | Azure"
    description="使用 Azure Resource Manager 模板为命名空间和队列创建服务总线授权规则"
    services="service-bus"
    documentationCenter=".net"
    authors="sethmanheim"
    manager="timlt"
    editor=""/>  


<tags
    ms.service="service-bus"
    ms.devlang="tbd"
    ms.topic="article"
    ms.tgt_pltfrm="dotnet"
    ms.workload="na"
    ms.date="01/18/2017"
    ms.author="sethm;shvija"
    wacn.date="03/20/2017"/>  

# 使用 Azure Resource Manager 模板为命名空间和队列创建服务总线授权规则

本文介绍如何使用为服务总线命名空间和队列创建[授权规则](/documentation/articles/service-bus-authentication-and-authorization/#shared-access-signature-authentication)的 Azure Resource Manager 模板。你将了解如何定义要部署的资源以及如何定义执行部署时指定的参数。可将此模板用于自己的部署，或自定义此模板以满足要求。

有关创建模板的详细信息，请参阅[创作 Azure Resource Manager 模板][]。

有关完整的模板，请参阅 GitHub 上的[服务总线授权规则模板][]。

>[AZURE.NOTE] 以下 Azure Resource Manager 模板可供下载和部署。
>
> -  [创建服务总线命名空间](/documentation/articles/service-bus-resource-manager-namespace/)
> -  [创建包含队列的服务总线命名空间](/documentation/articles/service-bus-resource-manager-namespace-queue/)
> -  [创建包含主题和订阅的服务总线命名空间](/documentation/articles/service-bus-resource-manager-namespace-topic/)
> -  [创建包含主题、订阅和规则的服务总线命名空间](/documentation/articles/service-bus-resource-manager-namespace-topic-with-rule/)

## 你将部署什么内容？

利用此模板，你将为命名空间和消息传送实体（在此情况下为队列）部署服务总线授权规则。

此模板使用[共享访问签名 (SAS)](/documentation/articles/service-bus-sas-overview/) 进行身份验证。SAS 使应用程序能够使用在命名空间或在关联了特定权限的消息传送实体（队列或主题）上配置的访问密钥向服务总线进行身份验证。然后可以使用此密钥生成 SAS 令牌，客户端反过来可用它向服务总线进行身份验证。

若要自动运行部署，请单击以下按钮：

[![部署到 Azure](./media/service-bus-resource-manager-namespace-auth-rule/deploybutton.png)](https://portal.azure.cn/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F301-servicebus-create-authrule-namespace-and-queue%2Fazuredeploy.json)

## 参数

使用 Azure 资源管理器，可以定义在部署模板时想要指定的值的参数。该模板具有一个名为 `Parameters` 的部分，其中包含所有参数值。你应该为随着要部署的项目或要部署到的环境而变化的值定义参数。不要为永远保持不变的值定义参数。每个参数值可在模板中用来定义所部署的资源。

模板定义以下参数。

### serviceBusNamespaceName

要创建的服务总线命名空间的名称。

		"serviceBusNamespaceName": {
		"type": "string"
		}

### namespaceAuthorizationRuleName 

命名空间的授权规则的名称。

		"namespaceAuthorizationRuleName ": {
		"type": "string"
		}

### serviceBusQueueName

服务总线命名空间中的队列的名称。

		"serviceBusQueueName": {
		"type": "string"
		}

### serviceBusApiVersion

模板的服务总线 API 版本。

		"serviceBusApiVersion": {
		"type": "string"
		}
## 要部署的资源

创建**消息传送**类型的标准服务总线命名空间，以及命名空间和实体的服务总线授权规则。

		"resources": [
		        {
		            "apiVersion": "[variables('sbVersion')]",
		            "name": "[parameters('serviceBusNamespaceName')]",
		            "type": "Microsoft.ServiceBus/namespaces",
		            "location": "[variables('location')]",
		            "kind": "Messaging",
		            "sku": {
		                "name": "StandardSku",
		                "tier": "Standard"
		            },
		            "resources": [
		                {
		                    "apiVersion": "[variables('sbVersion')]",
		                    "name": "[parameters('serviceBusQueueName')]",
		                    "type": "Queues",
		                    "dependsOn": [
		                        "[concat('Microsoft.ServiceBus/namespaces/', parameters('serviceBusNamespaceName'))]"
		                    ],
		                    "properties": {
		                        "path": "[parameters('serviceBusQueueName')]"
		                    },
		                    "resources": [
		                        {
		                            "apiVersion": "[variables('sbVersion')]",
		                            "name": "[parameters('queueAuthorizationRuleName')]",
		                            "type": "authorizationRules",
		                            "dependsOn": [
		                                "[parameters('serviceBusQueueName')]"
		                            ],
		                            "properties": {
		                                "Rights": ["Listen"]
		                            }
		                        }
		                    ]
		                }
		            ]
		        }, {
		            "apiVersion": "[variables('sbVersion')]",
		            "name": "[variables('namespaceAuthRuleName')]",
		            "type": "Microsoft.ServiceBus/namespaces/authorizationRules",
		            "dependsOn": ["[concat('Microsoft.ServiceBus/namespaces/', parameters('serviceBusNamespaceName'))]"],
		            "location": "[resourceGroup().location]",
		            "properties": {
		                "Rights": ["Send"]
		            }
		        }
		    ]

## 运行部署的命令

[AZURE.INCLUDE [app-service-deploy-commands](../../includes/app-service-deploy-commands.md)]

### PowerShell

		New-AzureRmResourceGroupDeployment -ResourceGroupName <resource-group-name> -TemplateFile <https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/301-servicebus-create-authrule-namespace-and-queue/azuredeploy.json>

## Azure CLI

		azure config mode arm

		azure group deployment create <my-resource-group> <my-deployment-name> --template-uri <https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/301-servicebus-create-authrule-namespace-and-queue/azuredeploy.json>

## 后续步骤

现在，你已使用 Azure Resource Manager 创建并部署了资源，请通过查看以下文章了解如何管理这些资源：

- [使用 PowerShell 管理服务总线](/documentation/articles/service-bus-powershell-how-to-provision/)
- [使用服务总线资源管理器管理服务总线资源](https://code.msdn.microsoft.com/Service-Bus-Explorer-f2abca5a)
- [服务总线身份验证和授权](/documentation/articles/service-bus-authentication-and-authorization/)

  [创作 Azure Resource Manager 模板]: /documentation/articles/resource-group-authoring-templates/
  [Using Azure PowerShell with Azure Resource Manager]: /documentation/articles/powershell-azure-resource-manager/
  [Using the Azure CLI for Mac, Linux, and Windows with Azure Resource Management]: /documentation/articles/xplat-cli-azure-resource-manager/
  [服务总线授权规则模板]: https://github.com/Azure/azure-quickstart-templates/blob/master/301-servicebus-create-authrule-namespace-and-queue/

<!---HONumber=Mooncake_1219_2016-->