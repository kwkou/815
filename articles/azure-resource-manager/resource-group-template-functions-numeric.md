<properties
    pageTitle="Azure Resource Manager 模板函数 - 数值 | Azure"
    description="介绍可在 Azure Resource Manager 模板中使用的用于处理数值的函数。"
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
    ms.date="05/08/2017"
    wacn.date="06/05/2017"
    ms.author="v-yeche"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="08618ee31568db24eba7a7d9a5fc3b079cf34577"
    ms.openlocfilehash="d70e119169690b52b1ccd2b244f98d234677aa7d"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/26/2017" />

# <a name="numeric-functions-for-azure-resource-manager-templates"></a>用于 Azure Resource Manager 模板的数值函数

Resource Manager 提供以下用于处理整数的函数：

* [添加](#add)
* [copyIndex](#copyindex)
* [div](#div)
* [float](#float)
* [int](#int)
* [min](#min)
* [max](#max)
* [mod](#mod)
* [mul](#mul)
* [sub](#sub)

## <a id="add"></a> 添加
`add(operand1, operand2)`

返回提供的两个整数的总和。

### <a name="parameters"></a>Parameters

| 参数 | 必选 | 类型 | 说明 |
|:--- |:--- |:--- |:--- | 
|operand1 |是 |int |被加数。 |
|operand2 |是 |int |加数。 |

### <a name="examples"></a>示例

以下示例将添加两个参数。

    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "first": {
                "type": "int",
                "metadata": {
                    "description": "First integer to add"
                }
            },
            "second": {
                "type": "int",
                "metadata": {
                    "description": "Second integer to add"
                }
            }
        },
        "resources": [
        ],
        "outputs": {
            "addResult": {
                "type": "int",
                "value": "[add(parameters('first'), parameters('second'))]"
            }
        }
    }

### <a name="return-value"></a>返回值

一个整数，包含参数的总和。

## <a id="copyindex"></a> copyIndex
`copyIndex(loopName, offset)`

返回迭代循环的索引。 

### <a name="parameters"></a>Parameters

| 参数 | 必选 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| loopName | 否 | 字符串 | 用于获取迭代的循环的名称。 |
| offset |否 |int |要添加到的从零开始的迭代值的数字。 |

### <a name="remarks"></a>备注

此函数始终用于 **copy** 对象。 如果没有提供任何值作为 **偏移量**，则返回当前迭代值。 迭代值从零开始。

**loopName** 属性可用于指定 copyIndex 是引用资源迭代还是引用属性迭代。 如果没有为 **loopName** 提供值，则将使用当前的资源类型迭代。 在属性上迭代时，请为 **loopName** 提供值。 

有关如何使用 **copyIndex** 的完整说明，请参阅 [Create multiple instances of resources in Azure Resource Manager](/documentation/articles/resource-group-create-multiple/)（在 Azure Resource Manager 中创建多个资源实例）。

### <a name="examples"></a>示例

以下示例显示名称中包含 copy 循环和索引值。 

    "resources": [ 
      { 
        "name": "[concat('examplecopy-', copyIndex())]", 
        "type": "Microsoft.Web/sites", 
        "copy": { 
          "name": "websitescopy", 
          "count": "[parameters('count')]" 
        }, 
        ...
      }
    ]

### <a name="return-value"></a>返回值

一个表示迭代的当前索引的整数。

## <a id="div"></a> div
`div(operand1, operand2)`

返回提供的两个整数在整除后的商。

### <a name="parameters"></a>Parameters

| 参数 | 必选 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| operand1 |是 |int |被除数。 |
| operand2 |是 |int |除数。 不能为 0。 |

### <a name="examples"></a>示例

以下示例将一个参数除以另一个参数。

    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "first": {
                "type": "int",
                "metadata": {
                    "description": "Integer being divided"
                }
            },
            "second": {
                "type": "int",
                "metadata": {
                    "description": "Integer used to divide"
                }
            }
        },
        "resources": [
        ],
        "outputs": {
            "divResult": {
                "type": "int",
                "value": "[div(parameters('first'), parameters('second'))]"
            }
        }
    }

### <a name="return-value"></a>返回值

一个表示商的整数。

## <a id="float"></a> float
`float(arg1)`

将值转换为浮点数。 仅当将自定义参数传递给应用程序（例如，逻辑应用）时，才使用此函数。

### <a name="parameters"></a>Parameters

| 参数 | 必选 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| arg1 |是 |字符串或整数 |要转换为浮点数的值。 |

### <a name="examples"></a>示例

以下示例演示如何使用 float 将参数传递给逻辑应用：

    {
        "type": "Microsoft.Logic/workflows",
        "properties": {
            ...
            "parameters": {
            "custom1": {
                "value": "[float('3.0')]"
            },
            "custom2": {
                "value": "[float(3)]"
            },

### <a name="return-value"></a>返回值
一个浮点数。

## <a id="int"></a> int
`int(valueToConvert)`

将指定的值转换为整数。

### <a name="parameters"></a>Parameters

| 参数 | 必选 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| valueToConvert |是 |字符串或整数 |要转换为整数的值。 |

### <a name="examples"></a>示例

以下示例将用户提供的参数值转换为整数。

    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "appId": { "type": "string" }
        },
        "variables": { 
            "intValue": "[int(parameters('appId'))]"
        },
        "resources": [
        ],
        "outputs": {
            "divResult": {
                "type": "int",
                "value": "[variables('intValue')]"
            }
        }
    }

### <a name="return-value"></a>返回值

一个整数。

## <a id="min"></a> min
`min (arg1)`

返回整数数组或逗号分隔的整数列表中的最小值。

### <a name="parameters"></a>Parameters

| 参数 | 必选 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| arg1 |是 |整数数组或逗号分隔的整数列表 |要获取最小值的集合。 |

### <a name="examples"></a>示例

以下示例演示如何对整数数组和整数列表使用 min：

    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "arrayToTest": {
                "type": "array",
                "defaultValue": [0,3,2,5,4]
            }
        },
        "resources": [],
        "outputs": {
            "arrayOutput": {
                "type": "int",
                "value": "[min(parameters('arrayToTest'))]"
            },
            "intOutput": {
                "type": "int",
                "value": "[min(0,3,2,5,4)]"
            }
        }
    }

### <a name="return-value"></a>返回值

一个整数，表示集合中的最小值。

## <a id="max"></a> max
`max (arg1)`

返回整数数组或逗号分隔的整数列表中的最大值。

### <a name="parameters"></a>Parameters

| 参数 | 必选 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| arg1 |是 |整数数组或逗号分隔的整数列表 |要获取最大值的集合。 |

### <a name="examples"></a>示例

以下示例演示如何对整数数组和整数列表使用 max：

    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "arrayToTest": {
                "type": "array",
                "defaultValue": [0,3,2,5,4]
            }
        },
        "resources": [],
        "outputs": {
            "arrayOutput": {
                "type": "int",
                "value": "[max(parameters('arrayToTest'))]"
            },
            "intOutput": {
                "type": "int",
                "value": "[max(0,3,2,5,4)]"
            }
        }
    }

### <a name="return-value"></a>返回值

一个整数，表示集合中的最大值。

## <a id="mod"></a> mod
`mod(operand1, operand2)`

返回使用提供的两个整数整除后的余数。

### <a name="parameters"></a>Parameters

| 参数 | 必选 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| operand1 |是 |int |被除数。 |
| operand2 |是 |int |除数，不能为 0。 |

### <a name="examples"></a>示例

以下示例将返回一个参数除以另一个参数后所得的余数。

    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "first": {
                "type": "int",
                "metadata": {
                    "description": "Integer being divided"
                }
            },
            "second": {
                "type": "int",
                "metadata": {
                    "description": "Integer used to divide"
                }
            }
        },
        "resources": [
        ],
        "outputs": {
            "modResult": {
                "type": "int",
                "value": "[mod(parameters('first'), parameters('second'))]"
            }
        }
    }

### <a name="return-value"></a>返回值
一个表示余数的整数。

## <a id="mul"></a> mul
`mul(operand1, operand2)`

返回提供的两个整数的积。

### <a name="parameters"></a>Parameters

| 参数 | 必选 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| operand1 |是 |int |被乘数。 |
| operand2 |是 |int |乘数。 |

### <a name="examples"></a>示例

以下示例将一个参数乘以另一个参数。

    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "first": {
                "type": "int",
                "metadata": {
                    "description": "First integer to multiply"
                }
            },
            "second": {
                "type": "int",
                "metadata": {
                    "description": "Second integer to multiply"
                }
            }
        },
        "resources": [
        ],
        "outputs": {
            "mulResult": {
                "type": "int",
                "value": "[mul(parameters('first'), parameters('second'))]"
            }
        }
    }

### <a name="return-value"></a>返回值

一个表示积的整数。

## <a id="sub"></a> sub
`sub(operand1, operand2)`

返回提供的两个整数在相减后的结果。

### <a name="parameters"></a>Parameters

| 参数 | 必选 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| operand1 |是 |int |被减数。 |
| operand2 |是 |int |减数。 |

### <a name="examples"></a>示例

以下示例将一个参数减另一个参数。

    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "first": {
                "type": "int",
                "metadata": {
                    "description": "Integer subtracted from"
                }
            },
            "second": {
                "type": "int",
                "metadata": {
                    "description": "Integer to subtract"
                }
            }
        },
        "resources": [
        ],
        "outputs": {
            "subResult": {
                "type": "int",
                "value": "[sub(parameters('first'), parameters('second'))]"
            }
        }
    }

### <a name="return-value"></a>返回值
一个表示减后结果的整数。

## <a name="next-steps"></a>后续步骤
* 有关 Azure Resource Manager 模板中各部分的说明，请参阅[创作 Azure Resource Manager 模板](/documentation/articles/resource-group-authoring-templates/)。
* 若要合并多个模板，请参阅[将链接的模板与 Azure Resource Manager 配合使用](/documentation/articles/resource-group-linked-templates/)。
* 若要在创建资源类型时迭代指定的次数，请参阅[在 Azure Resource Manager 中创建多个资源实例](/documentation/articles/resource-group-create-multiple/)。
* 若要查看如何部署已创建的模板，请参阅[使用 Azure Resource Manager 模板部署应用程序](/documentation/articles/resource-group-template-deploy/)。

<!--Update_Description:new article about illustrating numerics usage -->