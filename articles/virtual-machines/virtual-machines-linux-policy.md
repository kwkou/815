<properties
	pageTitle="将策略应用到 Azure Resource Manager 虚拟机 | Azure"
	description="如何将策略应用到 Azure Resource Manager Linux 虚拟机"
	services="virtual-machines-linux"
	documentationCenter=""
	authors="singhkay"
	manager="drewm"
	editor=""
	tags="azure-resource-manager"/>

<tags
	ms.service="virtual-machines-linux"
	ms.date="04/13/2016"
	wacn.date="06/07/2016"/>

# 将策略应用到 Azure Resource Manager 虚拟机

> [AZURE.NOTE]Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model/)。这篇文章介绍如何使用资源管理器部署模型，Azure 建议大多数新部署使用资源管理器模型替代经典部署模型

通过使用策略，组织可以在整个企业中强制实施各种约定和规则。强制实施所需行为有助于消除风险，同时为组织的成功做出贡献。在本文中，我们将介绍如何使用 Azure Resource Manager 策略来为组织中的虚拟机定义所需行为。

下面概述了实现此目的的步骤

1. Azure Resource Manager 策略 101
2. 为虚拟机定义策略
3. 创建策略
4. 应用策略

## 为虚拟机定义策略

企业中常用的一种方案可能是，只允许其用户在经测试可与 LOB 应用程序兼容的特定操作系统中创建虚拟机。使用 Azure Resource Manager 策略可以通过几个步骤完成此任务。
在此策略示例中，我们将只允许创建 Ubuntu 14.04.2-LTS 虚拟机。策略定义如下所示

	"if": {
	  "allOf": [
	    {
	      "field": "type",
	      "equals": "Microsoft.Compute/virtualMachines"
	    },
	    {
	      "not": {
	        "allOf": [
	          {
	            "field": "Microsoft.Compute/virtualMachines/imagePublisher",
	            "equals": "Canonical"
	          },
	          {
	            "field": "Microsoft.Compute/virtualMachines/imageOffer",
	            "equals": "UbuntuServer"
	          },
	          {
	            "field": "Microsoft.Compute/virtualMachines/imageSku",
	            "equals": "14.04.2-LTS"
	          }
	        ]
	      }
	    }
	  ]
	},
	"then": {
	  "effect": "deny"
	}

可以轻松修改上述策略，以允许在虚拟机部署中使用经过以下更改的任何 Ubuntu LTS 映像

	{
	  "field": "Microsoft.Compute/virtualMachines/imageSku",
	  "like": "*LTS"
	}

#### 虚拟机属性字段

下表描述了可在策略定义中用作字段的虚拟机属性。

| 字段名称 | 说明 |
|----------------|----------------------------------------------------|
| imagePublisher | 指定映像的发布者 |
| imageOffer | 指定所选映像发布者的产品 |
| imageSku | 指定所选产品的 SKU |
| imageVersion | 指定所选 SKU 的映像版本 |

## 创建策略

可以直接使用 REST API 或 PowerShell cmdlet 轻松创建策略。

## 应用策略

创建策略后，需要根据定义的范围来应用它。范围可以是订阅、资源组甚至资源。

<!---HONumber=Mooncake_0425_2016-->