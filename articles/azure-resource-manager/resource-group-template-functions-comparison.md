<properties
    pageTitle="Azure Resource Manager 模板函数 - 比较 | Azure"
    description="介绍可在 Azure Resource Manager 模板中使用的用于比较值的函数。"
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
    ms.openlocfilehash="cdd679a0cb57ba1a8424c11b27da2469731a9544"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/26/2017" />

# <a name="comparison-functions-for-azure-resource-manager-templates"></a>用于 Azure Resource Manager 模板的比较函数

Resource Manager 提供了多个用于在模板中进行比较的函数。

* [equals](#equals)
* [less](#less)
* [lessOrEquals](#lessorequals)
* [greater](#greater)
* [greaterOrEquals](#greaterorequals)

## <a id="equals"></a> equals
`equals(arg1, arg2)`

检查两个值是否相等。

### <a name="parameters"></a>Parameters

| 参数 | 必选 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| arg1 |是 |int、string、array 或 object |要检查是否相等的第一个值。 |
| arg2 |是 |int、string、array 或 object |要检查是否相等的第二个值。 |

### <a name="examples"></a>示例

示例模板检查不同类型的值是否相等。 所有默认值都返回 True。

    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "firstInt": {
                "type": "int",
                "defaultValue": 1
            },
            "secondInt": {
                "type": "int",
                "defaultValue": 1
            },
            "firstString": {
                "type": "string",
                "defaultValue": "a"
            },
            "secondString": {
                "type": "string",
                "defaultValue": "a"
            },
            "firstArray": {
                "type": "array",
                "defaultValue": ["a", "b"]
            },
            "secondArray": {
                "type": "array",
                "defaultValue": ["a", "b"]
            },
            "firstObject": {
                "type": "object",
                "defaultValue": {"a": "b"}
            },
            "secondObject": {
                "type": "object",
                "defaultValue": {"a": "b"}
            }
        },
        "resources": [
        ],
        "outputs": {
            "checkInts": {
                "type": "bool",
                "value": "[equals(parameters('firstInt'), parameters('secondInt') )]"
            },
            "checkStrings": {
                "type": "bool",
                "value": "[equals(parameters('firstString'), parameters('secondString'))]"
            },
            "checkArrays": {
                "type": "bool",
                "value": "[equals(parameters('firstArray'), parameters('secondArray'))]"
            },
            "checkObjects": {
                "type": "bool",
                "value": "[equals(parameters('firstObject'), parameters('secondObject'))]"
            }
        }
    }

### <a name="return-value"></a>返回值

如果值相等，返回 **True**；否则返回 **False**。

## <a id="less"></a> less
`less(arg1, arg2)`

检查第一个值是否小于第二个值。

### <a name="parameters"></a>Parameters

| 参数 | 必选 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| arg1 |是 |int 或 string |用于小于比较的第一个值。 |
| arg2 |是 |int 或 string |用于小于比较的第二个值。 |

### <a name="examples"></a>示例

示例模板检查一个值是否小于另一个值。

    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "firstInt": {
                "type": "int"
            },
            "secondInt": {
                "type": "int"
            },
            "firstString": {
                "type": "string"
            },
            "secondString": {
                "type": "string"
            }
        },
        "resources": [
        ],
        "outputs": {
            "checkInts": {
                "type": "bool",
                "value": "[less(parameters('firstInt'), parameters('secondInt') )]"
            },
            "checkStrings": {
                "type": "bool",
                "value": "[less(parameters('firstString'), parameters('secondString'))]"
            }
        }
    }

### <a name="return-value"></a>返回值

如果第一个值小于第二个值，返回 **True**；否则返回 **False**。

## <a id="lessorequals"></a> lessOrEquals
`lessOrEquals(arg1, arg2)`

检查第一个值是否小于或等于第二个值。

### <a name="parameters"></a>Parameters

| 参数 | 必选 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| arg1 |是 |int 或 string |用于小于或等于比较的第一个值。 |
| arg2 |是 |int 或 string |用于小于或等于比较的第二个值。 |

### <a name="examples"></a>示例

示例模板检查一个值是否小于或等于另一个值。

    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "firstInt": {
                "type": "int"
            },
            "secondInt": {
                "type": "int"
            },
            "firstString": {
                "type": "string"
            },
            "secondString": {
                "type": "string"
            }
        },
        "resources": [
        ],
        "outputs": {
            "checkInts": {
                "type": "bool",
                "value": "[lessOrEquals(parameters('firstInt'), parameters('secondInt') )]"
            },
            "checkStrings": {
                "type": "bool",
                "value": "[lessOrEquals(parameters('firstString'), parameters('secondString'))]"
            }
        }
    }

### <a name="return-value"></a>返回值

如果第一个值小于或等于第二个值，返回 **True**；否则返回 **False**。

## <a id="greater"></a> greater
`greater(arg1, arg2)`

检查第一个值是否大于第二个值。

### <a name="parameters"></a>Parameters

| 参数 | 必选 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| arg1 |是 |int 或 string |用于大于比较的第一个值。 |
| arg2 |是 |int 或 string |用于大于比较的第二个值。 |

### <a name="examples"></a>示例

示例模板检查一个值是否大于另一个值。

    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "firstInt": {
                "type": "int"
            },
            "secondInt": {
                "type": "int"
            },
            "firstString": {
                "type": "string"
            },
            "secondString": {
                "type": "string"
            }
        },
        "resources": [
        ],
        "outputs": {
            "checkInts": {
                "type": "bool",
                "value": "[greater(parameters('firstInt'), parameters('secondInt') )]"
            },
            "checkStrings": {
                "type": "bool",
                "value": "[greater(parameters('firstString'), parameters('secondString'))]"
            }
        }
    }

### <a name="return-value"></a>返回值

如果第一个值大于第二个值，返回 **True**；否则返回 **False**。

## <a id="greaterorequals"></a> greaterOrEquals
`greaterOrEquals(arg1, arg2)`

检查第一个值是否大于或等于第二个值。

### <a name="parameters"></a>Parameters

| 参数 | 必选 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| arg1 |是 |int 或 string |用于大于或等于比较的第一个值。 |
| arg2 |是 |int 或 string |用于大于或等于比较的第二个值。 |

### <a name="examples"></a>示例

示例模板检查一个值是否大于或等于另一个值。

    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "firstInt": {
                "type": "int"
            },
            "secondInt": {
                "type": "int"
            },
            "firstString": {
                "type": "string"
            },
            "secondString": {
                "type": "string"
            }
        },
        "resources": [
        ],
        "outputs": {
            "checkInts": {
                "type": "bool",
                "value": "[greaterOrEquals(parameters('firstInt'), parameters('secondInt') )]"
            },
            "checkStrings": {
                "type": "bool",
                "value": "[greaterOrEquals(parameters('firstString'), parameters('secondString'))]"
            }
        }
    }

### <a name="return-value"></a>返回值

如果第一个值大于或等于第二个值，返回 **True**；否则返回 **False**。

## <a name="next-steps"></a>后续步骤
* 有关 Azure Resource Manager 模板中各部分的说明，请参阅[创作 Azure Resource Manager 模板](/documentation/articles/resource-group-authoring-templates/)。
* 若要合并多个模板，请参阅[将链接的模板与 Azure Resource Manager 配合使用](/documentation/articles/resource-group-linked-templates/)。
* 若要在创建资源类型时迭代指定的次数，请参阅[在 Azure Resource Manager 中创建多个资源实例](/documentation/articles/resource-group-create-multiple/)。
* 若要查看如何部署已创建的模板，请参阅[使用 Azure Resource Manager 模板部署应用程序](/documentation/articles/resource-group-template-deploy/)。

<!--Update_Description:new article about illustrate the comparison usage -->