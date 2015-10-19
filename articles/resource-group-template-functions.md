<properties
   pageTitle="Azure 资源管理器模板函数"
   description="介绍在 Azure 资源管理器模板中检索值、格式字符串和检索部署信息时所使用的函数。"
   services="azure-resource-manager"
   documentationCenter="na"
   authors="tfitzmac"
   manager="wpickett"
   editor=""/>

<tags
   ms.service="azure-resource-manager"
   ms.date="07/27/2015"
   wacn.date="10/3/2015"/>

# Azure 资源管理器模板函数

本主题介绍你可以在 Azure 资源管理器模板中使用的所有函数。

## base64

**base64 (inputString)**

返回输入字符串的 base64 表示形式。

| 参数 | 必选 | 说明
| :--------------------------------: | :------: | :----------
| inputString | 是 | 要以 base64 表示形式返回的字符串值。

以下示例演示了如何使用 base64 函数。

    "variables": {
      "usernameAndPassword": "[concat('parameters('username'), ':', parameters('password'))]",
      "authorizationHeader": "[concat('Basic ', base64(variables('usernameAndPassword')))]"
    }

## concat

**concat (arg1, arg2, arg3, ...)**

组合多个字符串值，并返回生成的字符串值。此函数可以结合任意数目的参数。

以下示例演示了如何组合多个值以返回一个值。

    "outputs": {
        "siteUri": {
          "type": "string",
          "value": "[concat('http://',reference(resourceId('Microsoft.Web/sites', parameters('siteName'))).hostNames[0])]"
        }
    }

## copyIndex

**copyIndex(offset)**

返回一个迭代循环的当前索引。有关使用此函数的示例，请参阅[在 Azure 资源管理器中创建多个资源实例](/documentation/articles/resource-group-create-multiple)。

## 部署

**deployment()**

返回有关当前部署操作的信息。

将返回有关部署的信息，作为具有以下属性的对象：

    {
      "name": "",
      "properties": {
        "template": {},
        "parameters": {},
        "mode": "",
        "provisioningState": ""
      }
    }

以下示例演示了如何返回 outputs 部分中的部署信息。

    "outputs": {
      "exampleOutput": {
        "value": "[deployment()]",
        "type" : "object"
      }
    }

## listKeys

**listKeys (resourceName or resourceIdentifier, [apiVersion])**

返回存储帐户的密钥。可以使用 [resourceId 函数](#resourceid)或使用格式 **providerNamespace/resourceType/resourceName** 指定 resourceId。可以使用该函数来获取 primaryKey 和 secondaryKey。
  
| 参数 | 必选 | 说明
| :--------------------------------: | :------: | :----------
| resourceName 或 resourceIdentifier | 是 | 存储帐户的唯一标识符。
| apiVersion | 是 | 资源运行时状态的 API 版本。

以下示例演示了如何从 outputs 节中的存储帐户返回密钥。

    "outputs": { 
      "exampleOutput": { 
        "value": "[listKeys(resourceId('Microsoft.Storage/storageAccounts', parameters('storageAccountName')), providers('Microsoft.Storage', 'storageAccounts').apiVersions[0])]", 
        "type" : "object" 
      } 
    } 

## padLeft

**padLeft(stringToPad, totalLength, paddingCharacter)**

通过向左侧添加字符直至到达指定的总长度返回右对齐的字符串。
  
| 参数 | 必选 | 说明
| :--------------------------------: | :------: | :----------
| stringToPad | 是 | 要右对齐的字符串。
| totalLength | 是 | 返回字符串中的字符总数。
| paddingCharacter | 是 | 要用于向左填充直到达到总长度的字符。

以下示例演示如何通过添加零字符直到字符串达到 10 个字符进而填充该用户提供的参数值。如果原始的参数值超过 10 个字符，将不会添加任何字符。

    "parameters": {
        "appName": { "type": "string" }
    },
    "variables": { 
        "paddedAppName": "[padLeft(parameters('appName'),10,'0')]"
    }


## 参数

**parameters (parameterName)**

返回一个参数值。指定的参数名称必须已在模板的 parameters 节中定义。

| 参数 | 必选 | 说明
| :--------------------------------: | :------: | :----------
| parameterName | 是 | 要返回的参数名称。

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

## provider

**provider (providerNamespace, [resourceType])**

返回有关资源提供程序及其支持的资源类型的信息。如果未提供类型，则会返回所有支持的类型。

| 参数 | 必选 | 说明
| :--------------------------------: | :------: | :----------
| providerNamespace | 是 | 提供程序的命名空间
| resourceType | 否 | 指定的命名空间中的资源类型。

将使用以下格式返回支持的每个类型：

    {
        "resourceType": "",
        "locations": [ ],
        "apiVersions": [ ]
    }

以下示例演示了如何使用 provider 函数：

    "outputs": {
	    "exampleOutput": {
		    "value": "[providers('Microsoft.Storage', 'storageAccounts')]",
		    "type" : "object"
	    }
    }

## reference

**reference (resourceName or resourceIdentifier, [apiVersion])**

使表达式能够从另一个资源的运行时状态派生其值。

| 参数 | 必选 | 说明
| :--------------------------------: | :------: | :----------
| resourceName 或 resourceIdentifier | 是 | 资源的名称或唯一标识符。
| apiVersion | 否 | 资源运行时状态的 API 版本。如果在相同的模板内未设置资源，则应使用参数。

**reference** 函数从运行时状态派生其值，因此不能在 variables 节中使用。可以在模板的 outputs 节中使用它。

如果在相同的模板内设置了引用的资源，则可使用 reference 表达式来隐式声明一个资源依赖于另一个资源。另外，不需要使用 **dependsOn** 属性。只有当引用的资源已完成部署后，才会对表达式进行计算。

    "outputs": {
      "siteUri": {
          "type": "string",
          "value": "[concat('http://',reference(resourceId('Microsoft.Web/sites', parameters('siteName'))).hostNames[0])]"
      }
    }

## replace

**replace(originalString, oldCharacter, newCharacter)**

如果在所有实例中特定字符串中的一个字符已被替换为另一个字符时，则返回新的字符串。

| 参数 | 必选 | 说明
| :--------------------------------: | :------: | :----------
| originalString | 是 | 包含将某一个字符替换为另一个字符的所有实例的字符串。
| oldCharacter | 是 | 要从原始字符串中删除的字符。
| newCharacter | 是 | 要添加到已删除字符的位置的字符。

以下示例演示如何从用户提供的字符串中删除所有的短划线。

    "parameters": {
        "identifier": { "type": "string" }
    },
    "variables": { 
        "newidentifier": "[replace(parameters('identifier'),'-','')]"
    }

## resourceGroup

**resourceGroup()**

返回表示当前资源组的结构化对象。该对象的格式如下：

    {
      "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}",
      "name": "{resourceGroupName}",
      "location": "{resourceGroupLocation}",
    }

以下示例使用资源组位置来分配网站的位置。

    "resources": [
       {
          "apiVersion": "2014-06-01",
          "type": "Microsoft.Web/sites",
          "name": "[parameters('siteName')]",
          "location": "[resourceGroup().location]",
          ...
       }
    ]

## resourceId

**resourceId ([resourceGroupName], resourceType, resourceName1, [resourceName2]...)**

返回资源的唯一标识符。如果资源名称不确定或未设置在相同的模板内，请使用此函数。将使用以下格式返回标识符：

    /subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/{resourceProviderNamespace}/{resourceType}/{resourceName}
      
| 参数 | 必选 | 说明
| :---------------: | :------: | :----------
| resourceGroupName | 否 | 可选的资源组名称。默认值为当前资源组。如果要检索另一个资源组中的资源，请指定此值。
| resourceType | 是 | 资源类型，包括资源提供程序命名空间。
| resourceName1 | 是 | 资源的名称。
| resourceName2 | 否 | 下一个资源名称段（如果资源是嵌套的）。

以下示例演示了如何检索网站和数据库的资源 ID。网站存在于名为 **myWebsitesGroup** 的资源组中，而数据库存在于此模板的当前资源组中。

    [resourceId('myWebsitesGroup', 'Microsoft.Web/sites', parameters('siteName'))]
    [resourceId('Microsoft.SQL/servers/databases', parameters('serverName'),parameters('databaseName'))]
    
通常，在替代资源组中使用存储帐户或虚拟网络时，需要使用此函数。存储帐户或虚拟网络可能用于多个资源组中；因此，你不想要在删除单个资源组时删除它们。以下示例演示了如何轻松使用外部资源组中的资源：

    {
      "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {
          "virtualNetworkName": {
              "type": "string"
          },
          "virtualNetworkResourceGroup": {
              "type": "string"
          },
          "subnet1Name": {
              "type": "string"
          },
          "nicName": {
              "type": "string"
          }
      },
      "variables": {
          "vnetID": "[resourceId(parameters('virtualNetworkResourceGroup'), 'Microsoft.Network/virtualNetworks', parameters('virtualNetworkName'))]",
          "subnet1Ref": "[concat(variables('vnetID'),'/subnets/', parameters('subnet1Name'))]"
      },
      "resources": [
      {
          "apiVersion": "2015-05-01-preview",
          "type": "Microsoft.Network/networkInterfaces",
          "name": "[parameters('nicName')]",
          "location": "[parameters('location')]",
          "properties": {
              "ipConfigurations": [{
                  "name": "ipconfig1",
                  "properties": {
                      "privateIPAllocationMethod": "Dynamic",
                      "subnet": {
                          "id": "[variables('subnet1Ref')]"
                      }
                  }
              }]
           }
      }]
    }


## 订阅

**subscription()**

将使用以下格式返回有关订阅的详细信息。

    {
        "id": "/subscriptions/#####"
        "subscriptionId": "#####"
    }

以下示例演示了在 outputs 节中调用的 subscription 函数。

    "outputs": { 
      "exampleOutput": { 
          "value": "[subscription()]", 
          "type" : "object" 
      } 
    } 

## toLower

**toLower(stringToChange)**

将指定的字符串转换为小写。

| 参数 | 必选 | 说明
| :--------------------------------: | :------: | :----------
| stringToChange | 是 | 将转换为小写的字符串。

以下示例将用户提供的参数值转换为小写。

    "parameters": {
        "appName": { "type": "string" }
    },
    "variables": { 
        "lowerCaseAppName": "[toLower(parameters('appName'))]"
    }

## toUpper

**toUpper(stringToChange)**

将指定的字符串转换为大写。

| 参数 | 必选 | 说明
| :--------------------------------: | :------: | :----------
| stringToChange | 是 | 将转换为大写的字符串。

以下示例将用户提供的参数值转换为大写。

    "parameters": {
        "appName": { "type": "string" }
    },
    "variables": { 
        "upperCaseAppName": "[toUpper(parameters('appName'))]"
    }


## variables

**variables (variableName)**

返回变量的值。指定的变量名称必须已在模板的 variables 节中定义。

| 参数 | 必选 | 说明
| :--------------------------------: | :------: | :----------
| variableName | 是 | 要返回的变量名称。


## 后续步骤
- 有关 Azure 资源管理器模板中对各部分的说明，请参阅[创作 Azure 资源管理器模板](/documentation/articles/resource-group-authoring-templates)
- 若要合并多个模版，请参阅[将已链接的模版与 Azure 资源管理器配合使用](/documentation/articles/resource-group-linked-templates)
- 若要在创建资源类型时迭代指定的次数，请参阅[在 Azure 资源管理器中创建多个资源实例](/documentation/articles/resource-group-create-multiple)
<!-- - 若要查看如何部署已创建的模板，请参阅[使用 Azure 资源管理器模板部署应用程序](/documentation/articles/resource-group-template-deploy)-->

<!---HONumber=71-->