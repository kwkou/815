<properties
    pageTitle="Azure Resource Manager 模板函数 - 部署 | Azure"
    description="介绍可在 Azure Resource Manager 模板中使用的用于检索部署信息的函数。"
    services="azure-resource-manager"
    documentationcenter="na"
    author="tfitzmac"
    manager="timlt"
    editor="tysonn" />
<tags
    ms.assetid=""
    ms.service="azure-resource-manager"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="04/26/2017"
    wacn.date="06/05/2017"
    ms.author="v-yeche"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="08618ee31568db24eba7a7d9a5fc3b079cf34577"
    ms.openlocfilehash="27cc3e3e0a2aba522da9b1f20796a49aaa3b2a40"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/26/2017" />

# <a name="deployment-functions-for-azure-resource-manager-templates"></a>用于 Azure Resource Manager 模板的部署函数 

Resource Manager 提供以下函数，用于从与部署相关的模板和值部分获取值：

* [部署](#deployment)
* [参数](#parameters)
* [variables](#variables)

若要从资源、资源组或订阅获取值，请参阅 [Resource functions](/documentation/articles/resource-group-template-functions-resource/)（资源函数）。

## <a id="deployment"></a> 部署
`deployment()`

返回有关当前部署操作的信息。

### <a name="examples"></a>示例

下面的示例返回部署对象：

    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "resources": [],
        "outputs": {
            "subscriptionOutput": {
                "value": "[deployment()]",
                "type" : "object"
            }
        }
    }

以下示例演示如何根据父模板的 URI，使用 deployment() 链接到另一个模板。

    "variables": {  
        "sharedTemplateUrl": "[uri(deployment().properties.templateLink.uri, 'shared-resources.json')]"  
    }

### <a name="return-value"></a>返回值

此函数返回部署期间传递的对象。 根据部署对象是作为链接还是内联对象传递，所返回对象中的属性将有所不同。 如果部署对象是以内联形式传递的（例如使用 Azure PowerShell 中的 **-TemplateFile** 参数指向本地文件时），所返回的对象采用以下格式：

    {
        "name": "",
        "properties": {
            "template": {
                "$schema": "",
                "contentVersion": "",
                "parameters": {},
                "variables": {},
                "resources": [
                ],
                "outputs": {}
            },
            "parameters": {},
            "mode": "",
            "provisioningState": ""
        }
    }

如果对象是以链接形式传递的（例如使用 **-TemplateUri** 参数指向远程文件时），所返回的对象采用以下格式： 

    {
        "name": "",
        "properties": {
            "templateLink": {
                "uri": ""
            },
            "template": {
                "$schema": "",
                "contentVersion": "",
                "parameters": {},
                "variables": {},
                "resources": [],
                "outputs": {}
            },
            "parameters": {},
            "mode": "",
            "provisioningState": ""
        }
    }

## <a id="parameters"></a> 参数
`parameters(parameterName)`

返回一个参数值。 指定的参数名称必须已在模板的 parameters 节中定义。

### <a name="parameters"></a>Parameters

| 参数 | 必选 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| parameterName |是 |字符串 |要返回的参数名称。 |

### <a name="examples"></a>示例

以下示例演示了 parameters 函数的简化用法。

    "parameters": { 
      "siteName": {
          "type": "string"
      }
    },
    "resources": [
       {
          "apiVersion": "2014-06-01",
          "name": "[parameters('siteName')]",
          "type": "Microsoft.Web/Sites",
          ...
       }
    ]

### <a name="return-value"></a>返回值

参数的类型。

## <a id="variables"></a> variables
`variables(variableName)`

返回变量的值。 指定的变量名称必须已在模板的 variables 节中定义。

### <a name="parameters"></a>Parameters

| 参数 | 必选 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| variableName |是 |字符串 |要返回的变量名称。 |

### <a name="examples"></a>示例

以下示例使用变量值。

    "variables": {
      "storageName": "[concat('storage', uniqueString(resourceGroup().id))]"
    },
    "resources": [
      {
        "type": "Microsoft.Storage/storageAccounts",
        "name": "[variables('storageName')]",
        ...
      }
    ],

### <a name="return-value"></a>返回值

变量的类型。

## <a name="next-steps"></a>后续步骤
* 有关 Azure Resource Manager 模板中各部分的说明，请参阅[创作 Azure Resource Manager 模板](/documentation/articles/resource-group-authoring-templates/)。
* 若要合并多个模板，请参阅[将链接的模板与 Azure Resource Manager 配合使用](/documentation/articles/resource-group-linked-templates/)。
* 若要在创建资源类型时迭代指定的次数，请参阅[在 Azure Resource Manager 中创建多个资源实例](/documentation/articles/resource-group-create-multiple/)。
* 若要查看如何部署已创建的模板，请参阅[使用 Azure Resource Manager 模板部署应用程序](/documentation/articles/resource-group-template-deploy/)。

<!--Update_Description:new article about illustrating the deployment usage -->