<properties
    pageTitle="Azure Resource Manager 模板函数 - 资源 | Azure"
    description="介绍可在 Azure Resource Manager 模板中用于检索资源相关值的函数。"
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
    ms.author="tomfitz"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="08618ee31568db24eba7a7d9a5fc3b079cf34577"
    ms.openlocfilehash="c5bf28c1be7eada22f8d31a932ecead1e9c4f73b"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/26/2017" />

# <a name="resource-functions-for-azure-resource-manager-templates"></a>用于 Azure Resource Manager 模板的资源函数

Resource Manager 提供以下用于获取资源值的函数：

* [listKeys 和 list{Value}](#listkeys)
* [providers](#providers)
* [reference](#reference)
* [resourceGroup](#resourcegroup)
* [resourceId](#resourceid)
* [订阅](#subscription)

若要从参数、变量或当前部署获取值，请参阅 [Deployment value functions](/documentation/articles/resource-group-template-functions-deployment/)（部署值函数）。

<a id="listkeys" />
## <a id="list"></a> listKeys 和 list{Value}
`listKeys(resourceName or resourceIdentifier, apiVersion)`

`list{Value}(resourceName or resourceIdentifier, apiVersion)`

返回支持 list 操作的任何资源类型的值。 最常见的用法是 `listKeys`。 

### <a name="parameters"></a>Parameters

| 参数 | 必选 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| resourceName 或 resourceIdentifier |是 |字符串 |资源的唯一标识符。 |
| apiVersion |是 |字符串 |资源运行时状态的 API 版本。 通常采用 **yyyy-mm-dd**格式。 |

### <a name="remarks"></a>备注

以 **list** 开头的任何操作都可用作模板中的函数。 可用操作不仅包括 listKeys，也包括 `list`、`listAdminKeys` 和 `listStatus` 等操作。 若要确定哪些资源类型具有列表操作，请使用以下选项：

* 查看资源提供程序的 [REST API 操作](https://docs.microsoft.com/zh-cn/rest/api/)，并查找列表操作。 例如，存储帐户具有 [listKeys 操作](https://docs.microsoft.com/zh-cn/rest/api/storagerp/storageaccounts#StorageAccounts_ListKeys)。
* 使用 [Get-AzureRmProviderOperation](https://docs.microsoft.com/zh-cn/powershell/module/azurerm.resources/get-azurermprovideroperation) PowerShell cmdlet。 以下示例获取存储帐户的所有列表操作：

      Get-AzureRmProviderOperation -OperationSearchString "Microsoft.Storage/*" | where {$_.Operation -like "*list*"} | FT Operation

* 使用以下 Azure CLI 命令和 JSON 实用工具 [jq](http://stedolan.github.io/jq/download/) 来仅筛选列表操作：

      azure provider operations show --operationSearchString */apiapps/* --json | jq ".[] | select (.operation | contains(\"list\"))"

使用 [resourceId 函数](#resourceid)或格式 `{providerNamespace}/{resourceType}/{resourceName}` 指定资源。

### <a name="examples"></a>示例

以下示例演示如何从 outputs 节中的存储帐户返回主密钥和辅助密钥。

    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "storageAccountId": {
                "type": "string"
            }
        },
        "resources": [],
        "outputs": {
            "storageKeysOutput": {
                "value": "[listKeys(parameters('storageAccountId'), '2016-01-01')]",
                "type" : "object"
            }
        }
    }

### <a name="return-value"></a>返回值

ListKeys 返回的对象采用以下格式：

    {
      "keys": [
        {
          "keyName": "key1",
          "permissions": "Full",
          "value": "{value}"
        },
        {
          "keyName": "key2",
          "permissions": "Full",
          "value": "{value}"
        }
      ]
    }

其他列表函数具有不同的返回格式。 若要查看函数的格式，请将其包含在 outputs 节中，如示例模板所示。 

## <a id="providers"></a> providers
`providers(providerNamespace, [resourceType])`

返回有关资源提供程序及其支持的资源类型的信息。 如果未提供资源类型，该函数将返回资源提供程序支持的所有类型。

### <a name="parameters"></a>Parameters

| 参数 | 必选 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| providerNamespace |是 |字符串 |提供程序的命名空间 |
| resourceType |否 |字符串 |指定的命名空间中的资源类型。 |

### <a name="examples"></a>示例

以下示例演示了如何使用 provider 函数：

    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "resources": [],
        "outputs": {
            "providerOutput": {
                "value": "[providers('Microsoft.Storage', 'storageAccounts')]",
                "type" : "object"
            }
        }
    }

### <a name="return-value"></a>返回值

将使用以下格式返回支持的每个类型： 

    {
        "resourceType": "{name of resource type}",
        "locations": [ all supported locations ],
        "apiVersions": [ all supported API versions ]
    }

不保证返回值的数组排序。

## <a id="reference"></a> reference
`reference(resourceName or resourceIdentifier, [apiVersion])`

返回表示资源的运行时状态的对象。

### <a name="parameters"></a>Parameters

| 参数 | 必选 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| resourceName 或 resourceIdentifier |是 |字符串 |资源的名称或唯一标识符。 |
| apiVersion |否 |字符串 |指定的资源的 API 版本。 如果资源不是在同一模板中预配的，请包含此参数。 通常采用 **yyyy-mm-dd**格式。 |

### <a name="remarks"></a>备注

reference 函数从运行时状态派生其值，因此不能在 variables 节中使用。 可以在模板的 outputs 节中使用它。 

如果在相同的模板内设置了引用的资源，则可使用 reference 函数来隐式声明一个资源依赖于另一个资源。 不需要同时使用 dependsOn 属性。 只有当引用的资源已完成部署后，才会对函数求值。

若要查看资源类型的属性名称和值，请创建一个模板，该模板返回 outputs 节中的对象。 如果有现有的该类型的资源，则模板只返回对象而不部署任何新资源。 

### <a name="examples"></a>示例

以下示例引用未部署在此模板中，但存在于同一资源组中的存储帐户。

    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "storageAccountName": {
                "type": "string"
            }
        },
        "resources": [],
        "outputs": {
            "ExistingStorage": {
                "value": "[reference(concat('Microsoft.Storage/storageAccounts/', parameters('storageAccountName')), '2016-01-01')]",
                "type" : "object"
            }
        }
    }

或者，可以部署并引用同一模板中的资源。

    {
      "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {
          "storageAccountName": { 
              "type": "string"
          }
      },
      "resources": [
        {
          "name": "[parameters('storageAccountName')]",
          "type": "Microsoft.Storage/storageAccounts",
          "apiVersion": "2016-12-01",
          "sku": {
            "name": "Standard_LRS"
          },
          "kind": "Storage",
          "location": "[resourceGroup().location]",
          "tags": {},
          "properties": {
          }
        }
      ],
      "outputs": {
          "referenceOutput": {
              "type": "object",
              "value": "[reference(parameters('storageAccountName'))]"
          }
        }
    }

通常情况下，可以使用 reference 函数返回对象的特定值，例如 Blob 终结点 URI 或完全限定域名。

    "outputs": {
        "BlobUri": {
            "value": "[reference(concat('Microsoft.Storage/storageAccounts/', parameters('storageAccountName')), '2016-01-01').primaryEndpoints.blob]",
            "type" : "string"
        },
        "FQDN": {
            "value": "[reference(concat('Microsoft.Network/publicIPAddresses/', parameters('ipAddressName')), '2016-03-30').dnsSettings.fqdn]",
            "type" : "string"
        }
    }

### <a name="return-value"></a>返回值

每种资源类型返回 reference 函数的不同属性。 该函数不返回单个预定义的格式。 若要查看资源类型的属性，请返回 outputs 节中的对象，如示例所示。

## <a id="resourcegroup"></a> resourceGroup
`resourceGroup()`

返回表示当前资源组的对象。 

### <a name="examples"></a>示例

以下模板返回资源组的属性。

    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "resources": [],
        "outputs": {
            "subscriptionOutput": {
                "value": "[resourceGroup()]",
                "type" : "object"
            }
        }
    }

resourceGroup 函数的一个常见用途是在与资源组相同的位置中创建资源。 以下示例使用资源组位置来分配网站的位置。

    "resources": [
       {
          "apiVersion": "2014-06-01",
          "type": "Microsoft.Web/sites",
          "name": "[parameters('siteName')]",
          "location": "[resourceGroup().location]",
          ...
       }
    ]

### <a name="return-value"></a>返回值

返回的对象采用以下格式：

    {
      "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}",
      "name": "{resourceGroupName}",
      "location": "{resourceGroupLocation}",
      "tags": {
      },
      "properties": {
        "provisioningState": "{status}"
      }
    }

## <a id="resourceid"></a> resourceId
`resourceId([subscriptionId], [resourceGroupName], resourceType, resourceName1, [resourceName2]...)`

返回资源的唯一标识符。 如果资源名称不确定或未设置在相同的模板内，请使用此函数。 

### <a name="parameters"></a>Parameters

| 参数 | 必选 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| subscriptionId |否 |字符串（GUID 格式） |默认值为当前订阅。 如果需要检索另一个订阅中的资源，请指定此值。 |
| resourceGroupName |否 |字符串 |默认值为当前资源组。 如果需要检索另一个资源组中的资源，请指定此值。 |
| resourceType |是 |字符串 |资源类型，包括资源提供程序命名空间。 |
| resourceName1 |是 |字符串 |资源的名称。 |
| resourceName2 |否 |字符串 |下一个资源名称段（如果资源是嵌套的）。 |

### <a name="examples"></a>示例

以下示例返回资源组中存储帐户的资源 ID：

    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "resources": [],
        "outputs": {
            "resourceIdOutput": {
                "value": "[resourceId('Microsoft.Storage/storageAccounts','examplestorage')]",
                "type" : "string"
            }
        }
    }

以下示例演示如何检索不同资源组中网站的资源 ID 和不同资源组中的数据库：

    [resourceId('otherResourceGroup', 'Microsoft.Web/sites', parameters('siteName'))]
    [resourceId('otherResourceGroup', 'Microsoft.SQL/servers/databases', parameters('serverName'), parameters('databaseName'))]

通常，在替代资源组中使用存储帐户或虚拟网络时，需要使用此函数。 存储帐户或虚拟网络可能用于多个资源组中；因此，你不想要在删除单个资源组时删除它们。 以下示例演示了如何轻松使用外部资源组中的资源：

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

### <a name="return-value"></a>返回值

将使用以下格式返回标识符：

    /subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/{resourceProviderNamespace}/{resourceType}/{resourceName}

## <a id="subscription"></a> 订阅
`subscription()`

返回有关当前部署的订阅的详细信息。 

### <a name="examples"></a>示例

以下示例显示了在 outputs 节中调用的 subscription 函数。 

    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "resources": [],
        "outputs": {
            "subscriptionOutput": {
                "value": "[subscription()]",
                "type" : "object"
            }
        }
    }

### <a name="return-value"></a>返回值

该函数返回以下格式：

    {
        "id": "/subscriptions/{subscription-id}",
        "subscriptionId": "{subscription-id}",
        "tenantId": "{tenant-id}",
        "displayName": "{name-of-subscription}"
    }

## <a name="next-steps"></a>后续步骤
* 有关 Azure Resource Manager 模板中各部分的说明，请参阅[创作 Azure Resource Manager 模板](/documentation/articles/resource-group-authoring-templates/)。
* 若要合并多个模板，请参阅[将链接的模板与 Azure Resource Manager 配合使用](/documentation/articles/resource-group-linked-templates/)。
* 若要在创建资源类型时迭代指定的次数，请参阅[在 Azure Resource Manager 中创建多个资源实例](/documentation/articles/resource-group-create-multiple/)。
* 若要查看如何部署已创建的模板，请参阅[使用 Azure Resource Manager 模板部署应用程序](/documentation/articles/resource-group-template-deploy/)。

<!--Update_Description:new article about illustrating resource usage -->