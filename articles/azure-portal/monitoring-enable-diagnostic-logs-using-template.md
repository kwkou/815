<properties
	pageTitle="使用 Resource Manager 模板自动启用诊断设置 | Azure"
	description="了解如何使用 Resource Manager 模板创建诊断设置，以便将诊断日志流式传输到事件中心，或者将其存储在存储帐户中。"
	authors="johnkemnetz"
	manager="rboucher"
	editor=""
	services="monitoring-and-diagnostics"
	documentationCenter="monitoring-and-diagnostics"/>

<tags
	ms.service="monitoring-and-diagnostics"
	ms.workload="na"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="08/17/2016"
	ms.author="johnkem"
	wacn.date="10/17/2016"/>  


# 在创建资源时使用 Resource Manager 模板自动启用诊断设置
本文介绍如何使用 [Azure Resource Manager 模板](/documentation/articles/resource-group-authoring-templates/)在创建资源时配置资源的诊断设置。这样可以让用户在创建资源时自动将诊断日志和指标流式传输到事件中心，或者将其存档到存储帐户中。

通过 Resource Manager 模板启用诊断日志时，所用方法取决于资源类型。

- **非计算**资源（例如，网络安全组、逻辑应用、自动化）使用[此文中描述的诊断设置](/documentation/articles/monitoring-overview-of-diagnostic-logs/#diagnostic-settings)。
- **计算**（基于 WAD/LAD）资源使用[此文中描述的 WAD/LAD 配置文件](/documentation/articles/vs-azure-tools-diagnostics-for-cloud-services-and-virtual-machines/)。

本文介绍如何使用其中一种方法配置诊断。

基本步骤如下所示：

1. 以 JSON 文件格式创建一个模板，说明如何创建资源和启用诊断。
2. [使用任意部署方法部署模板](/documentation/articles/resource-group-template-deploy/)。

下面是一个需为非计算资源和计算资源生成的模板 JSON 文件的示例。

## 非计算资源模板
对于非计算资源，需要做两件事：

1. 根据存储帐户名称和服务总线规则 ID 向参数 blob 添加参数（允许在存储帐户中存档诊断日志，并/或将日志流式传输到事件中心）。

    ```
    "storageAccountName": {
      "type": "string",
      "metadata": {
        "description": "Name of the Storage Account in which Diagnostic Logs should be saved."
      }
    },
    "serviceBusRuleId": {
      "type": "string",
      "metadata": {
        "description": "Service Bus Rule Id for the Service Bus Namespace in which the Event Hub should be created or streamed to."
      }
    }
    ```
2. 在资源数组（包含需启用诊断日志的资源）中，添加类型为 `[resource namespace]/providers/diagnosticSettings` 的资源。

    ```
    "resources": [
      {
        "type": "providers/diagnosticSettings",
        "name": "Microsoft.Insights/service",
        "dependsOn": [
          "[/*resource Id for which Diagnostic Logs will be enabled>*/]"
        ],
        "apiVersion": "2015-07-01",
        "properties": {
          "storageAccountId": "[resourceId('Microsoft.Storage/storageAccounts', parameters('storageAccountName'))]",
          "serviceBusRuleId": "[parameters('serviceBusRuleId')]",
          "logs": [ 
            {
              "category": "/* log category name */",
              "enabled": true,
              "retentionPolicy": {
                "days": 0,
                "enabled": false
              }
            }
          ]
        }
      }
    ]
    ```

诊断设置的属性 blob 遵循[此文所述的格式](https://msdn.microsoft.com/zh-cn/library/azure/dn931931.aspx)。

下面是一个完整的示例，说明了如何创建网络安全组，以及如何启用流式传输到事件中心和在存储帐户中进行存储的功能。

```
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "nsgName": {
            "type": "string",
			"metadata": {
				"description": "Name of the NSG that will be created."
			}
        },
		"storageAccountName": {
			"type": "string",
			"metadata": {
				"description":"Name of the Storage Account in which Diagnostic Logs should be saved."
			}
		},
		"serviceBusRuleId": {
			"type": "string",
			"metadata": {
				"description":"Service Bus Rule Id for the Service Bus Namespace in which the Event Hub should be created or streamed to."
			}
		}
    },
    "variables": {},
    "resources": [
        {
            "type": "Microsoft.Network/networkSecurityGroups",
            "name": "[parameters('nsgName')]",
            "apiVersion": "2016-03-30",
            "location": "westus",
            "properties": {
                "securityRules": []
            },
            "resources": [
				{
					"type": "providers/diagnosticSettings",
					"name": "Microsoft.Insights/service",
					"dependsOn": [
						"[resourceId('Microsoft.Network/networkSecurityGroups', parameters('nsgName'))]"
					],
					"apiVersion": "2015-07-01",
					"properties": {
						"storageAccountId": "[resourceId('Microsoft.Storage/storageAccounts', parameters('storageAccountName'))]",
                        "serviceBusRuleId": "[parameters('serviceBusRuleId')]",
						"logs": [
							{
								"category": "NetworkSecurityGroupEvent",
								"enabled": true,
								"retentionPolicy": {
									"days": 0,
									"enabled": false
								}
							},
                            {
								"category": "NetworkSecurityGroupRuleCounter",
								"enabled": true,
								"retentionPolicy": {
									"days": 0,
									"enabled": false
								}
							}
						]
					}
				}
			],
            "dependsOn": []
        }
    ]
}
```

## 计算资源模板
若要允许对计算资源（例如虚拟机或 Service Fabric 群集）进行诊断，需执行以下操作：

1. 将 Azure 诊断扩展添加到 VM 资源定义中。
2. 以参数形式指定存储帐户和/或事件中心。
3. 将 WADCfg XML 文件的内容添加到 XMLCfg 属性中，对所有 XML 字符进行适当的转义。

> [AZURE.WARNING] 这最后一步操作起来比较复杂。请[参阅此文](/documentation/articles/virtual-machines-windows-extensions-diagnostics-template/#diagnostics-configuration-variables)获取相关示例，了解如何将诊断配置架构拆分成进行了正确转义和格式化操作的变量。

整个过程（包括示例）详见[此文档](/documentation/articles/virtual-machines-windows-extensions-diagnostics-template/)。


## 后续步骤
- [详细了解 Azure 诊断日志](/documentation/articles/monitoring-overview-of-diagnostic-logs/)
- [将 Azure 诊断日志流式传输到事件中心](/documentation/articles/monitoring-stream-diagnostic-logs-to-event-hubs/)

<!---HONumber=Mooncake_1010_2016-->