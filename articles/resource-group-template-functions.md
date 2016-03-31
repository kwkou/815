<properties
   pageTitle="资源管理器模板表达式 | Azure"
   description="介绍在 Azure 资源管理器模板中检索值、处理字符串和数字以及检索部署信息时所用的函数。"
   services="azure-resource-manager"
   documentationCenter="na"
   authors="tfitzmac"
   manager="wpickett"
   editor=""/>

<tags
   ms.service="azure-resource-manager"
   ms.date="12/07/2015"
   wacn.date="01/14/2016"/>

# Azure 资源管理器模板表达式

本主题介绍你可以在 Azure 资源管理器模板中使用的所有表达式。

模板表达式及其参数不区分大小写。例如，资源管理器将 **variables('var1')** 和 **VARIABLES('VAR1')** 视为相同。在求值时，除非表达式明确修改大小与（例如，使用 toUpper 或 toLower），否则表达式将保留大小写。某些资源类型可能会提出大小写要求，而不考虑表达式求值方式。

## 数字表达式

资源管理器提供以下表达式用于处理整数：

- [添加](#add)
- [copyIndex](#copyindex)
- [div](#div)
- [int](#int)
- [length](#length)
- [mod](#mod)
- [mul](#mul)
- [sub](#sub)


<a id="add" />
### 添加

**add(operand1, operand2)**

返回提供的两个整数的总和。

| 参数 | 必选 | 说明
| :--------------------------------: | :------: | :----------
| operand1 | 是 | 要使用的第一个操作数。
| operand2 | 是 | 要使用的第二个操作数。


<a id="copyindex" />
### copyIndex

**copyIndex(offset)**

返回一个迭代循环的当前索引。

此表达式始终配合 **copy** 对象使用。有关使用 **copyIndex** 的示例，请参阅[在 Azure 资源管理器中创建多个资源实例](/documentation/articles/resource-group-create-multiple)。


<a id="div" />
### div

**div(operand1, operand2)**

返回提供的两个整数在整除后的商。

| 参数 | 必选 | 说明
| :--------------------------------: | :------: | :----------
| operand1 | 是 | 被除数。
| operand2 | 是 | 除数，不得为 0。


<a id="int" />
### int

**int(valueToConvert)**

将指定的值转换为整数。

| 参数 | 必选 | 说明
| :--------------------------------: | :------: | :----------
| valueToConvert | 是 | 要转换为整数的值。值的类型只能是字符串或整数。

以下示例将用户提供的参数值转换为整数。

    "parameters": {
        "appId": { "type": "string" }
    },
    "variables": { 
        "intValue": "[int(parameters('appId'))]"
    }


<a id="length" />
### length

**length(array 或 string)**

返回数组中的元素数或字符串中的字符数。创建资源时，可在数组中使用此函数指定迭代数。在以下示例中，参数 **siteNames** 引用创建网站时要使用的名称数组。

    "copy": {
        "name": "websitescopy",
        "count": "[length(parameters('siteNames'))]"
    }

有关在数组中使用此函数的详细信息，请参阅[在 Azure 资源管理器中创建多个资源实例](resource-group-create-multiple.md)。

或者，你可以在字符串中使用：

    "parameters": {
        "appName": { "type": "string" }
    },
    "variables": { 
        "nameLength": "[length(parameters('appName'))]"
    }


<a id="mod" />
### mod

**mod(operand1, operand2)**

返回使用提供的两个整数整除后的余数。

| 参数 | 必选 | 说明
| :--------------------------------: | :------: | :----------
| operand1 | 是 | 被除数。
| operand2 | 是 | 除数，不得为 0。



<a id="mul" />
### mul

**mul(operand1, operand2)**

返回提供的两个整数的积。

| 参数 | 必选 | 说明
| :--------------------------------: | :------: | :----------
| operand1 | 是 | 要使用的第一个操作数。
| operand2 | 是 | 要使用的第二个操作数。


<a id="sub" />
### sub

**sub(operand1, operand2)**

返回提供的两个整数在相减后的结果。

| 参数 | 必选 | 说明
| :--------------------------------: | :------: | :----------
| operand1 | 是 | 减数。
| operand2 | 是 | 被减数。


## 字符串表达式

资源管理器提供以下表达式用于处理字符串：

- [base64](#base64)
- [concat](#concat)
- [padLeft](#padleft)
- [replace](#replace)
- [split](#split)
- [字符串](#string)
- [toLower](#tolower)
- [toUpper](#toupper)
- [trim](#trim)
- [uniqueString](#uniquestring)
- [uri](#uri)

若要获取字符串或数组中的字符数，请参阅 [length](#length)。

<a id="base64" />
### base64

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

<a id="concat" />
### concat

**concat (arg1, arg2, arg3, ...)**

组合多个字符串值，并返回生成的字符串值。此函数可以结合任意数目的参数。

以下示例演示了如何组合多个值以返回一个值。

    "outputs": {
        "siteUri": {
          "type": "string",
          "value": "[concat('http://',reference(resourceId('Microsoft.Web/sites', parameters('siteName'))).hostNames[0])]"
        }
    }

<a id="padleft" />
### padLeft

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

<a id="replace" />
### replace

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

<a id="split" />
### split

**split(inputString, delimiter)**
**split(inputString, [delimiters])**

返回包含输入字符串的子字符串的字符串数组，其中的子字符串使用发送的分隔符进行分隔。

| 参数 | 必选 | 说明
| :--------------------------------: | :------: | :----------
| inputString | 是 | 要拆分的字符串。
| delimiter | 是 | 要使用的分隔符，可以是单个字符串，也可以是字符串数组。

以下示例使用逗号拆分输入字符串。

    "parameters": {
        "inputString": { "type": "string" }
    },
    "variables": { 
        "stringPieces": "[split(parameters('inputString'), ',')]"
    }

<a id="string" />
### 字符串

**string(valueToConvert)**

将指定的值转换为字符串。

| 参数 | 必选 | 说明
| :--------------------------------: | :------: | :----------
| valueToConvert | 是 | 要转换为字符串的值。该值的类型只能是布尔值、整数或字符串。

以下示例将用户提供的参数值转换为字符串。

    "parameters": {
        "appId": { "type": "int" }
    },
    "variables": { 
        "stringValue": "[string(parameters('appId'))]"
    }

<a id="tolower" />
### toLower

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

<a id="toupper" />
### toUpper

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

<a id="trim" />
### trim

**trim (stringToTrim)**

从指定的字符串中删除所有前导和尾随空白字符。

| 参数 | 必选 | 说明
| :--------------------------------: | :------: | :----------
| stringToTrim | 是 | 要裁剪的字符串。

以下示例将裁剪掉用户提供的参数值中的空白字符。

    "parameters": {
        "appName": { "type": "string" }
    },
    "variables": { 
        "trimAppName": "[trim(parameters('appName'))]"
    }

<a id="uniquestring" />
### uniqueString

**uniqueString (stringForCreatingUniqueString, ...)**

执行所提供字符串的 64 位哈希来创建唯一的字符串。当你需要创建资源的唯一名称时，此函数很有帮助。提供表示结果唯一性级别的参数值。可以指定该名称对于你的订阅、资源组或部署是否唯一。

| 参数 | 必选 | 说明
| :--------------------------------: | :------: | :----------
| stringForCreatingUniqueString | 是 | 哈希函数中用于创建唯一字符串的基本字符串。
| 根据需要使用其他参数 | 否 | 你可以添加任意数目的字符串，以创建指定唯一性级别的值。

返回的值不是完全随机字符串，而是哈希函数的结果。返回的值长度为 13 个字符。不保证是全局唯一。你可以根据命名约定使用前缀来组合值，以创建更友好的名称。

以下示例演示显示如何使用 uniqueString 来创建不同的通用级唯一值。

根据订阅保持唯一

    "[uniqueString(subscription().subscriptionId)]"

根据资源组保持唯一

    "[uniqueString(resourceGroup().id)]"

根据资源组的部署保持唯一

    "[uniqueString(resourceGroup().id, deployment().name)]"
    
以下示例演示显示如何根据资源组创建存储帐户的唯一名称。

    "resources": [{ 
        "name": "[concat('ContosoStorage', uniqueString(resourceGroup().id))]", 
        "type": "Microsoft.Storage/storageAccounts", 
        ...

<a id="uri" />
### uri

**uri (baseUri, relativeUri)**

通过组合 baseUri 和 relativeUri 字符串来创建绝对 URI。

| 参数 | 必选 | 说明
| :--------------------------------: | :------: | :----------
| baseUri | 是 | 基本 uri 字符串。
| relativeUri | 是 | 要添加到基本 uri 字符串的相对 uri 字符串。

**baseUri** 参数的值可包含特定文件，但在构造 URI 时，只使用基路径。例如，将 **http://contoso.com/resources/azuredeploy.json** 作为 baseUri 参数传递会生成 **http://contoso.com/resources/** 的基 URI。

以下示例演示如何根据父模板的值构造嵌套模板的链接。

    "templateLink": "[uri(deployment().properties.templateLink.uri, 'nested/azuredeploy.json')]"



## 部署值表达式

资源管理器提供以下表达式，用于从与部署相关的模板和值部分获取值：

- [部署](#deployment)
- [参数](#parameters)
- [variables](#variables)

若要从资源、资源组或订阅获取值，请参阅[资源表达式](#resource-expressions)。

<a id="deployment" />
### 部署

**deployment()**

返回有关当前部署操作的信息。

此表达式返回部署期间传递的对象。根据部署对象是作为链接还是内联对象传递的，所返回对象中的属性将有所不同。如果部署对象是以内联形式传递的（例如使用 Azure PowerShell 中的 **-TemplateFile** 参数指向本地文件时），所返回的对象采用以下格式：

    {
        "name": "",
        "properties": {
            "template": {
                "$schema": "",
                "contentVersion": "",
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
                "uri": "",
                "contentVersion": ""
            },
            "mode": "",
            "provisioningState": ""
        }
    }

以下示例演示如何根据父模板的 URI，使用 deployment() 链接到另一个模板。

    "variables": {  
        "sharedTemplateUrl": "[uri(deployment().properties.templateLink.uri, 'shared-resources.json')]"  
    }  


<a id="parameters" />
### 参数

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

<a id="variables" />
### variables

**variables (variableName)**

返回变量的值。指定的变量名称必须已在模板的 variables 节中定义。

| 参数 | 必选 | 说明
| :--------------------------------: | :------: | :----------
| variableName | 是 | 要返回的变量名称。



## 资源表达式

资源管理器提供以下表达式用于获取资源值：

- [listkeys](#listkeys)
- [providers](#providers)
- [reference](#reference)
- [resourceGroup](#resourcegroup)
- [resourceId](#resourceid)
- [订阅](#subscription)

若要从参数、变量或当前部署获取值，请参阅[部署值表达式](#deployment-value-expressions)。

<a id="listkeys" />
### listKeys

**listKeys (resourceName or resourceIdentifier, apiVersion)**

返回存储帐户的密钥。可以使用 [resourceId 函数](./#resourceid)或使用格式 **providerNamespace/resourceType/resourceName** 指定 resourceId。可以使用该函数来获取 primaryKey 和 secondaryKey。
  
| 参数 | 必选 | 说明
| :--------------------------------: | :------: | :----------
| resourceName 或 resourceIdentifier | 是 | 存储帐户的唯一标识符。
| apiVersion | 是 | 资源运行时状态的 API 版本。

以下示例演示了如何从 outputs 节中的存储帐户返回密钥。

    "outputs": { 
      "exampleOutput": { 
        "value": "[listKeys(resourceId('Microsoft.Storage/storageAccounts', parameters('storageAccountName')), '2015-05-01-preview')]", 
        "type" : "object" 
      } 
    } 

<a id="providers" />
### providers

**providers (providerNamespace, [resourceType])**

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

<a id="reference" />
### reference

**reference (resourceName or resourceIdentifier, [apiVersion])**

使表达式能够从另一个资源的运行时状态派生其值。

| 参数 | 必选 | 说明
| :--------------------------------: | :------: | :----------
| resourceName 或 resourceIdentifier | 是 | 资源的名称或唯一标识符。
| apiVersion | 否 | 指定的资源的 API 版本。如果未在同一模板内预配资源，则必须包含此参数。

**reference** 函数从运行时状态派生其值，因此不能在 variables 节中使用。可以在模板的 outputs 节中使用它。

如果在相同的模板内设置了引用的资源，则可使用 reference 表达式来隐式声明一个资源依赖于另一个资源。另外，不需要使用 **dependsOn** 属性。只有当引用的资源已完成部署后，才会对表达式进行计算。

以下示例引用同一模板中部署的存储帐户。

    "outputs": {
		"NewStorage": {
			"value": "[reference(parameters('storageAccountName'))]",
			"type" : "object"
		}
	}

以下示例引用未部署在此模板中，但是当部署资源时存在于同一资源组内的存储帐户。

    "outputs": {
		"ExistingStorage": {
			"value": "[reference(concat('Microsoft.Storage/storageAccounts/', parameters('storageAccountName')), '2015-06-15')]",
			"type" : "object"
		}
	}

可以从返回的对象（例如 Blob 终结点 URI）检索特定的值，如下所示。

    "outputs": {
		"BlobUri": {
			"value": "[reference(concat('Microsoft.Storage/storageAccounts/', parameters('storageAccountName')), '2015-06-15').primaryEndpoints.blob]",
			"type" : "string"
		}
	}

如果你现在想要直接在模板中指定 API 版本，可以使用 **providers** 表达式并检索一个值（例如最新版本），如下所示。

    "outputs": {
		"BlobUri": {
			"value": "[reference(concat('Microsoft.Storage/storageAccounts/', parameters('storageAccountName')), providers('Microsoft.Storage', 'storageAccounts').apiVersions[0]).primaryEndpoints.blob]",
			"type" : "string"
		}
	}

以下示例引用不同资源组中的存储帐户。

    "outputs": {
		"BlobUri": {
			"value": "[reference(resourceId(parameters('relatedGroup'), 'Microsoft.Storage/storageAccounts/', parameters('storageAccountName')), '2015-06-15').primaryEndpoints.blob]",
			"type" : "string"
		}
	}

<a id="resourcegroup" />
### resourceGroup

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

<a id="resourceid" />
### resourceId

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

<a id="subscription" />
### 订阅

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


## 后续步骤
- 有关 Azure 资源管理器模板中对各部分的说明，请参阅[创作 Azure 资源管理器模板](/documentation/articles/resource-group-authoring-templates)
- 若要合并多个模板，请参阅[将已链接的模板与 Azure 资源管理器配合使用](/documentation/articles/resource-group-linked-templates)
- 若要在创建资源类型时迭代指定的次数，请参阅[在 Azure 资源管理器中创建多个资源实例](/documentation/articles/resource-group-create-multiple)
- 若要查看如何部署已创建的模板，请参阅[使用 Azure 资源管理器模板部署应用程序](/documentation/articles/resource-group-template-deploy)

<!---HONumber=Mooncake_0104_2016-->