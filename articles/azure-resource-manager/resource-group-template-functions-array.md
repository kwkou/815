<properties
    pageTitle="Azure Resource Manager 模板函数 - 数组和对象 | Azure"
    description="介绍可在 Azure Resource Manager 模板中用来处理数组和对象的函数。"
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
    ms.openlocfilehash="ab1146f6f826c39af7e670c67a181cbdbf534e28"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/26/2017" />

# <a name="array-and-object-functions-for-azure-resource-manager-templates"></a>用于 Azure Resource Manager 模板的数组和对象函数 

Resource Manager 提供以下用于处理数组和对象的函数。

* [array](#array)
* [coalesce](#coalesce)
* [concat](#concat)
* [contains](#contains)
* [createArray](#createarray)
* [empty](#empty)
* [first](#first)
* [intersection](#intersection)
* [last](#last)
* [length](#length)
* [min](#min)
* [max](#max)
* [range](#range)
* [skip](#skip)
* [take](#take)
* [union](#union)

若要获取由某个值分隔的字符串值数组，请参阅 [split](/documentation/articles/resource-group-template-functions-string/#split)。

## <a id="array"></a> array
`array(convertToArray)`

将值转换为数组。

### <a name="parameters"></a>Parameters

| 参数 | 必选 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| convertToArray |是 |整数、字符串、数组或对象 |要转换为数组的值。 |

### <a name="examples"></a>示例

以下示例演示如何对不同的类型使用 array 函数。

    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "intToConvert": {
                "type": "int",
                "defaultValue": 1
            },
            "stringToConvert": {
                "type": "string",
                "defaultValue": "a"
            },
            "objectToConvert": {
                "type": "object",
                "defaultValue": {"a": "b", "c": "d"}
            }
        },
        "resources": [
        ],
        "outputs": {
            "intOutput": {
                "type": "array",
                "value": "[array(parameters('intToConvert'))]"
            },
            "stringOutput": {
                "type": "array",
                "value": "[array(parameters('stringToConvert'))]"
            },
            "objectOutput": {
                "type": "array",
                "value": "[array(parameters('objectToConvert'))]"
            }
        }
    }

### <a name="return-value"></a>返回值

一个数组。

## <a id="coalesce"></a> coalesce
`coalesce(arg1, arg2, arg3, ...)`

从参数中返回第一个非 null 值。 空字符串、空数组和空对象不为 null。

### <a name="parameters"></a>Parameters

| 参数 | 必选 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| arg1 |是 |整数、字符串、数组或对象 |要测试是否为 null 的第一个值。 |
| 其他参数 |否 |整数、字符串、数组或对象 |要测试是否为 null 的其他值。 |

### <a name="examples"></a>示例

以下示例显示 coalesce 不同用法的输出。

    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "objectToTest": {
                "type": "object",
                "defaultValue": {"first": null, "second": null}
            }
        },
        "resources": [
        ],
        "outputs": {
            "stringOutput": {
                "type": "string",
                "value": "[coalesce(parameters('objectToTest').first, parameters('objectToTest').second, 'fallback')]"
            },
            "intOutput": {
                "type": "int",
                "value": "[coalesce(parameters('objectToTest').first, parameters('objectToTest').second, 1)]"
            },
            "objectOutput": {
                "type": "object",
                "value": "[coalesce(parameters('objectToTest').first, parameters('objectToTest').second, parameters('objectToTest'))]"
            },
            "arrayOutput": {
                "type": "array",
                "value": "[coalesce(parameters('objectToTest').first, parameters('objectToTest').second, array(1))]"
            }
        }
    }

### <a name="return-value"></a>返回值

第一个非 null 参数的值，可以是字符串、整数、数组或对象。 如果所有参数都为 null，则为 null。 

## <a id="concat"></a> concat
`concat(arg1, arg2, arg3, ...)`

合并多个数组并返回串联的数组，或合并多个字符串值并返回串联的字符串。 

### <a name="parameters"></a>Parameters

| 参数 | 必选 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| arg1 |是 |数组或字符串 |要串联的第一个数组或字符串。 |
| 其他参数 |否 |数组或字符串 |按顺序排列的串联的其他数组或字符串。 |

此函数可以使用任意数量的参数，并可接受字符串或数组作为参数。

### <a name="examples"></a>示例

以下示例演示如何组合两个数组。

    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": { 
            "firstArray": { 
                "type": "array", 
                "defaultValue": [ 
                    "1-1", 
                    "1-2", 
                    "1-3" 
                ] 
            },
            "secondArray": {
                "type": "array", 
                "defaultValue": [ 
                    "2-1", 
                    "2-2",
                    "2-3" 
                ] 
            }
        },
        "resources": [
        ],
        "outputs": {
            "return": {
                "type": "array",
                "value": "[concat(parameters('firstArray'), parameters('secondArray'))]"
            }
        }
    }

以下示例演示如何组合两个字符串值并返回串联的字符串。

    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "prefix": {
                "type": "string",
                "defaultValue": "prefix"
            }
        },
        "resources": [],
        "outputs": {
            "concatOutput": {
                "value": "[concat(parameters('prefix'), uniqueString(resourceGroup().id))]",
                "type" : "string"
            }
        }
    }

### <a name="return-value"></a>返回值
由串联值构成的字符串或数组。

## <a id="contains"></a> contains
`contains(container, itemToFind)`

检查数组是否包含某个值、某个对象是否包含某个键，或者某个字符串是否包含某个子字符串。

### <a name="parameters"></a>Parameters

| 参数 | 必选 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| container |是 |数组、对象或字符串 |包含要查找的值的值。 |
| itemToFind |是 |字符串或整数 |要查找的值。 |

### <a name="examples"></a>示例

以下示例展示了如何对不同的类型使用 contains：

    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "stringToTest": {
                "type": "string",
                "defaultValue": "OneTwoThree"
            },
            "objectToTest": {
                "type": "object",
                "defaultValue": {"one": "a", "two": "b", "three": "c"}
            },
            "arrayToTest": {
                "type": "array",
                "defaultValue": ["one", "two", "three"]
            }
        },
        "resources": [
        ],
        "outputs": {
            "stringTrue": {
                "type": "bool",
                "value": "[contains(parameters('stringToTest'), 'e')]"
            },
            "stringFalse": {
                "type": "bool",
                "value": "[contains(parameters('stringToTest'), 'z')]"
            },
            "objectTrue": {
                "type": "bool",
                "value": "[contains(parameters('objectToTest'), 'one')]"
            },
            "objectFalse": {
                "type": "bool",
                "value": "[contains(parameters('objectToTest'), 'a')]"
            },
            "arrayTrue": {
                "type": "bool",
                "value": "[contains(parameters('arrayToTest'), 'three')]"
            },
            "arrayFalse": {
                "type": "bool",
                "value": "[contains(parameters('arrayToTest'), 'four')]"
            }
        }
    }

### <a name="return-value"></a>返回值

如果找到该项，则为 **True**；否则为 **False**。

## <a id="createarray"></a> createarray
`createArray (arg1, arg2, arg3, ...)`

从参数创建数组。

### <a name="parameters"></a>Parameters

| 参数 | 必选 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| arg1 |是 |字符串、整数、数组或对象 |数组中的第一个值。 |
| 其他参数 |否 |字符串、整数、数组或对象 |数组中的其他值。 |

### <a name="examples"></a>示例

以下示例演示如何对不同的类型使用 createArray：

    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "objectToTest": {
                "type": "object",
                "defaultValue": {"one": "a", "two": "b", "three": "c"}
            },
            "arrayToTest": {
                "type": "array",
                "defaultValue": ["one", "two", "three"]
            }
        },
        "resources": [
        ],
        "outputs": {
            "stringArray": {
                "type": "array",
                "value": "[createArray('a', 'b', 'c')]"
            },
            "intArray": {
                "type": "array",
                "value": "[createArray(1, 2, 3)]"
            },
            "objectArray": {
                "type": "array",
                "value": "[createArray(parameters('objectToTest'))]"
            },
            "arrayArray": {
                "type": "array",
                "value": "[createArray(parameters('arrayToTest'))]"
            }
        }
    }

### <a name="return-value"></a>返回值

一个数组。

## <a id="empty"></a> empty

`empty(itemToTest)`

确定数组、对象或字符串是否为空。

### <a name="parameters"></a>Parameters

| 参数 | 必选 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| itemToTest |是 |数组、对象或字符串 |要检查是否为空的值。 |

### <a name="examples"></a>示例

以下示例检查某个数组、对象和字符串是否为空。

    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "testArray": {
                "type": "array",
                "defaultValue": []
            },
            "testObject": {
                "type": "object",
                "defaultValue": {}
            },
            "testString": {
                "type": "string",
                "defaultValue": ""
            }
        },
        "resources": [
        ],
        "outputs": {
            "arrayEmpty": {
                "type": "bool",
                "value": "[empty(parameters('testArray'))]"
            },
            "objectEmpty": {
                "type": "bool",
                "value": "[empty(parameters('testObject'))]"
            },
            "stringEmpty": {
                "type": "bool",
                "value": "[empty(parameters('testString'))]"
            }
        }
    }

### <a name="return-value"></a>返回值

如果该值为空，则返回 **True**；否则返回 **False**。

## <a id="first"></a> first
`first(arg1)`

返回数组的第一个元素，或字符串的第一个字符。

### <a name="parameters"></a>Parameters

| 参数 | 必选 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| arg1 |是 |数组或字符串 |要检索第一个元素或字符的值。 |

### <a name="examples"></a>示例

以下示例展示了如何对不同的类型使用 first 函数。

    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "arrayToTest": {
                "type": "array",
                "defaultValue": ["one", "two", "three"]
            }
        },
        "resources": [
        ],
        "outputs": {
            "arrayOutput": {
                "type": "string",
                "value": "[first(parameters('arrayToTest'))]"
            },
            "stringOutput": {
                "type": "string",
                "value": "[first('One Two Three')]"
            }
        }
    }

### <a name="return-value"></a>返回值

数组中第一个元素或者字符串的第一个字符的类型（字符串、整数、数组或对象）。

## <a id="intersection"></a> intersection
`intersection(arg1, arg2, arg3, ...)`

返回包含参数中通用元素的单个数组或对象。

### <a name="parameters"></a>Parameters

| 参数 | 必选 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| arg1 |是 |数组或对象 |用于查找通用元素的第一个值。 |
| arg2 |是 |数组或对象 |用于查找通用元素的第二个值。 |
| 其他参数 |否 |数组或对象 |用于查找通用元素的其他值。 |

### <a name="examples"></a>示例

以下示例演示如何对数组和对象使用 intersection：

    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "firstObject": {
                "type": "object",
                "defaultValue": {"one": "a", "two": "b", "three": "c"}
            },
            "secondObject": {
                "type": "object",
                "defaultValue": {"one": "a", "two": "z", "three": "c"}
            },
            "firstArray": {
                "type": "array",
                "defaultValue": ["one", "two", "three"]
            },
            "secondArray": {
                "type": "array",
                "defaultValue": ["two", "three"]
            }
        },
        "resources": [
        ],
        "outputs": {
            "objectOutput": {
                "type": "object",
                "value": "[intersection(parameters('firstObject'), parameters('secondObject'))]"
            },
            "arrayOutput": {
                "type": "array",
                "value": "[intersection(parameters('firstArray'), parameters('secondArray'))]"
            }
        }
    }

### <a name="return-value"></a>返回值

包含通用元素的数组或对象。

## <a id="last"></a> last
`last (arg1)`

返回数组的最后一个元素，或字符串的最后一个字符。

### <a name="parameters"></a>Parameters

| 参数 | 必选 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| arg1 |是 |数组或字符串 |要检索最后一个元素或字符的值。 |

### <a name="examples"></a>示例

以下示例展示了如何对不同的类型使用 last 函数。

    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "arrayToTest": {
                "type": "array",
                "defaultValue": ["one", "two", "three"]
            }
        },
        "resources": [
        ],
        "outputs": {
            "arrayOutput": {
                "type": "string",
                "value": "[last(parameters('arrayToTest'))]"
            },
            "stringOutput": {
                "type": "string",
                "value": "[last('One Two Three')]"
            }
        }
    }

### <a name="return-value"></a>返回值

数组中最后一个元素或者字符串的最后一个字符的类型（字符串、整数、数组或对象）。

## <a id="length"></a> length
`length(arg1)`

返回数组中的元素数，或字符串中的字符数。

### <a name="parameters"></a>Parameters

| 参数 | 必选 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| arg1 |是 |数组或字符串 |用于获取元素数的数组，或用于获取字符数的字符串。 |

### <a name="examples"></a>示例

以下示例展示了如何对数组和字符串使用 length：

    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "arrayToTest": {
                "type": "array",
                "defaultValue": [
                    "one",
                    "two",
                    "three"
                ]
            },
            "stringToTest": {
                "type": "string",
                "defaultValue": "One Two Three"
            }
        },
        "resources": [],
        "outputs": {
            "arrayLength": {
                "type": "int",
                "value": "[length(parameters('arrayToTest'))]"
            },
            "stringLength": {
                "type": "int",
                "value": "[length(parameters('stringToTest'))]"
            }
        }
    }

创建资源时，可在数组中使用此函数指定迭代数。 在以下示例中，参数 **siteNames** 引用创建网站时要使用的名称数组。

    "copy": {
        "name": "websitescopy",
        "count": "[length(parameters('siteNames'))]"
    }

有关在数组中使用此函数的详细信息，请参阅 [Create multiple instances of resources in Azure Resource Manager](/documentation/articles/resource-group-create-multiple/)（在 Azure Resource Manager 中创建多个资源实例）。

### <a name="return-value"></a>返回值

一个整数。 

## <a id="min"></a> min
`min(arg1)`

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

表示最小值的整数。

## <a id="max"></a> max
`max(arg1)`

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

表示最大值的整数。

## <a id="range"></a> range
`range(startingInteger, numberOfElements)`

从起始整数创建整数数组并包含一些项。

### <a name="parameters"></a>Parameters

| 参数 | 必选 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| startingInteger |是 |int |数组中的第一个整数。 |
| numberofElements |是 |int |数组中的整数个数。 |

### <a name="examples"></a>示例

以下示例演示如何使用 range 函数：

    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "startingInt": {
                "type": "int",
                "defaultValue": 5
            },
            "numberOfElements": {
                "type": "int",
                "defaultValue": 3
            }
        },
        "resources": [],
        "outputs": {
            "rangeOutput": {
                "type": "array",
                "value": "[range(parameters('startingInt'),parameters('numberOfElements'))]"
            }
        }
    }

### <a name="return-value"></a>返回值

整数数组。

## <a id="skip"></a> skip
`skip(originalValue, numberToSkip)`

返回一个数组，其中包含数组中指定数字后面的所有元素；或返回一个字符串，其中包含字符串中指定数后面的所有字符。

### <a name="parameters"></a>Parameters

| 参数 | 必选 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| originalValue |是 |数组或字符串 |用于跳过的数组或字符串。 |
| numberToSkip |是 |int |要跳过的元素数或字符数。 如果此值小于或等于 0，则返回值中的所有元素或字符。 如果此值大于数组或字符串的长度，则返回空数组或字符串。 |

### <a name="examples"></a>示例

以下示例跳过数组中指定数目的元素，以及字符串中指定数目的字符。

    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "testArray": {
                "type": "array",
                "defaultValue": [
                    "one",
                    "two",
                    "three"
                ]
            },
            "elementsToSkip": {
                "type": "int",
                "defaultValue": 2
            },
            "testString": {
                "type": "string",
                "defaultValue": "one two three"
            },
            "charactersToSkip": {
                "type": "int",
                "defaultValue": 4
            }
        },
        "resources": [],
        "outputs": {
            "arrayOutput": {
                "type": "array",
                "value": "[skip(parameters('testArray'),parameters('elementsToSkip'))]"
            },
            "stringOutput": {
                "type": "string",
                "value": "[skip(parameters('testString'),parameters('charactersToSkip'))]"
            }
        }
    }

### <a name="return-value"></a>返回值

数组或字符串。

## <a id="take"></a> take
`take(originalValue, numberToTake)`

返回一个数组，其中包含从数组开头位置算起的指定数目的元素；或返回一个字符串，其中包含从字符串开头位置算起的指定数目的字符。

### <a name="parameters"></a>Parameters

| 参数 | 必选 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| originalValue |是 |数组或字符串 |要从中提取元素的数组或字符串。 |
| numberToTake |是 |int |要提取的元素或字符数。 如果此值为 0 或更小，则返回一个空数组或字符串。 如果此值大于给定数组或字符串的长度，则返回数组或字符串中的所有元素。 |

### <a name="examples"></a>示例

以下示例从数组中提取指定数目的元素，并从字符串中提取指定数目的字符。

    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "testArray": {
                "type": "array",
                "defaultValue": [
                    "one",
                    "two",
                    "three"
                ]
            },
            "elementsToTake": {
                "type": "int",
                "defaultValue": 2
            },
            "testString": {
                "type": "string",
                "defaultValue": "one two three"
            },
            "charactersToTake": {
                "type": "int",
                "defaultValue": 2
            }
        },
        "resources": [],
        "outputs": {
            "arrayOutput": {
                "type": "array",
                "value": "[take(parameters('testArray'),parameters('elementsToTake'))]"
            },
            "stringOutput": {
                "type": "string",
                "value": "[take(parameters('testString'),parameters('charactersToTake'))]"
            }
        }
    }

### <a name="return-value"></a>返回值

数组或字符串。

## <a id="union"></a> union
`union(arg1, arg2, arg3, ...)`

返回包含参数中所有元素的单个数组或对象。 重复的值或键仅包含一次。

### <a name="parameters"></a>Parameters

| 参数 | 必选 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| arg1 |是 |数组或对象 |用于联接元素的第一个值。 |
| arg2 |是 |数组或对象 |用于联接元素的第二个值。 |
| 其他参数 |否 |数组或对象 |用于联接元素的其他值。 |

### <a name="examples"></a>示例

以下示例演示如何对数组和对象使用 union：

    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "firstObject": {
                "type": "object",
                "defaultValue": {"one": "a", "two": "b", "three": "c"}
            },
            "secondObject": {
                "type": "object",
                "defaultValue": {"four": "d", "five": "e", "six": "f"}
            },
            "firstArray": {
                "type": "array",
                "defaultValue": ["one", "two", "three"]
            },
            "secondArray": {
                "type": "array",
                "defaultValue": ["four", "five"]
            }
        },
        "resources": [
        ],
        "outputs": {
            "objectOutput": {
                "type": "object",
                "value": "[union(parameters('firstObject'), parameters('secondObject'))]"
            },
            "arrayOutput": {
                "type": "array",
                "value": "[union(parameters('firstArray'), parameters('secondArray'))]"
            }
        }
    }

### <a name="return-value"></a>返回值

数组或对象。

## <a name="next-steps"></a>后续步骤
* 有关 Azure Resource Manager 模板中各部分的说明，请参阅[创作 Azure Resource Manager 模板](/documentation/articles/resource-group-authoring-templates/)。
* 若要合并多个模板，请参阅[将链接的模板与 Azure Resource Manager 配合使用](/documentation/articles/resource-group-linked-templates/)。
* 若要在创建资源类型时迭代指定的次数，请参阅[在 Azure Resource Manager 中创建多个资源实例](/documentation/articles/resource-group-create-multiple/)。
* 若要查看如何部署已创建的模板，请参阅[使用 Azure Resource Manager 模板部署应用程序](/documentation/articles/resource-group-template-deploy/)。

<!--Update_Description:new article about illustrate the array usage-->