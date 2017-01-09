<properties
    pageTitle="Resource Manager 模板函数 | Azure"
    description="介绍在 Azure 资源管理器模板中检索值、处理字符串和数字以及检索部署信息时所用的函数。"
    services="azure-resource-manager"
    documentationcenter="na"
    author="tfitzmac"
    manager="timlt"
    editor="tysonn" />
<tags
    ms.assetid="0644abe1-abaa-443d-820d-1966d7d26bfd"
    ms.service="azure-resource-manager"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="11/22/2016"
    wacn.date="01/06/2017"
    ms.author="tomfitz" />

# Azure 资源管理器模板函数
本主题介绍可以在 Azure Resource Manager 模板中使用的所有函数。

模板函数及其参数不区分大小写。例如，资源管理器将 **variables('var1')** 和 **VARIABLES('VAR1')** 视为相同。在求值时，除非函数明确修改大小写（例如，使用 toUpper 或 toLower 进行修改），否则函数将保留大小写。某些资源类型可能会提出大小写要求，而不考虑函数求值方式。

## 数值函数
资源管理器提供以下用于处理整数的函数：

* [添加](#add)
* [copyIndex](#copyindex)
* [div](#div)
* [int](#int)
* [mod](#mod)
* [mul](#mul)
* [sub](#sub)

### <a id="add"></a> add
`add(operand1, operand2)`  


返回提供的两个整数的总和。

| 参数 | 必选 | 类型 | 说明 |
|:--- |:--- |:--- |:--- | 
|operand1 |是 |Integer |被加数。 |
|operand2 |是 |Integer |加数。 |

以下示例将添加两个参数。

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
    ...
    "outputs": {
      "addResult": {
        "type": "int",
        "value": "[add(parameters('first'), parameters('second'))]"
      }
    }

### <a id="copyindex"></a> copyIndex
`copyIndex(offset)`  


返回迭代循环的索引。

| 参数 | 必选 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| offset |否 |Integer |要添加到的从零开始的迭代值的数字。 |

此函数始终用于 **copy** 对象。如果没有提供任何值作为**偏移量**，则返回当前迭代值。迭代值从零开始。有关如何使用 **copyIndex** 的完整说明，请参阅 [Create multiple instances of resources in Azure Resource Manager](/documentation/articles/resource-group-create-multiple/)（在 Azure Resource Manager 中创建多个资源实例）。

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


### <a id="div"></a> div
`div(operand1, operand2)`  


返回提供的两个整数在整除后的商。

| 参数 | 必选 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| operand1 |是 |Integer |被除数。 |
| operand2 |是 |Integer |除数。不能为 0。 |

以下示例将一个参数除以另一个参数。

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
    ...
    "outputs": {
      "divResult": {
        "type": "int",
        "value": "[div(parameters('first'), parameters('second'))]"
      }
    }

### <a id="int"></a> int
`int(valueToConvert)`  


将指定的值转换为整数。

| 参数 | 必选 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| valueToConvert |是 |字符串或整数 |要转换为整数的值。 |

以下示例将用户提供的参数值转换为整数。

    "parameters": {
        "appId": { "type": "string" }
    },
    "variables": { 
        "intValue": "[int(parameters('appId'))]"
    }


### <a id="mod"></a> mod
`mod(operand1, operand2)`  


返回使用提供的两个整数整除后的余数。

| 参数 | 必选 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| operand1 |是 |Integer |被除数。 |
| operand2 |是 |Integer |除数，不能为 0。 |

以下示例将返回一个参数除以另一个参数后所得的余数。

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
    ...
    "outputs": {
      "modResult": {
        "type": "int",
        "value": "[mod(parameters('first'), parameters('second'))]"
      }
    }

### <a id="mul"></a> mul
`mul(operand1, operand2)`  


返回提供的两个整数的积。

| 参数 | 必选 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| operand1 |是 |Integer |被乘数。 |
| operand2 |是 |Integer |乘数。 |

以下示例将一个参数乘以另一个参数。

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
    ...
    "outputs": {
      "mulResult": {
        "type": "int",
        "value": "[mul(parameters('first'), parameters('second'))]"
      }
    }

### <a id="sub"></a> sub
`sub(operand1, operand2)`  


返回提供的两个整数在相减后的结果。

| 参数 | 必选 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| operand1 |是 |Integer |被减数。 |
| operand2 |是 |Integer |减数。 |

以下示例将一个参数减另一个参数。

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
    ...
    "outputs": {
      "subResult": {
        "type": "int",
        "value": "[sub(parameters('first'), parameters('second'))]"
      }
    }

## 字符串函数
资源管理器提供以下用于处理字符串的函数：

* [base64](#base64)
* [concat](#concat)
* [length](#lengthstring)
* [padLeft](#padleft)
* [replace](#replace)
* [skip](#skipstring)
* [split](#split)
* [string](#string)
* [substring](#substring)
* [take](#takestring)
* [toLower](#tolower)
* [toUpper](#toupper)
* [trim](#trim)
* [uniqueString](#uniquestring)
* [uri](#uri)

### <a id="base64"></a> base64
`base64 (inputString)`  


返回输入字符串的 base64 表示形式。

| 参数 | 必选 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| inputString |是 |String |要以 base64 表示形式返回的值。 |

以下示例演示如何使用 base64 函数。

    "variables": {
      "usernameAndPassword": "[concat(parameters('username'), ':', parameters('password'))]",
      "authorizationHeader": "[concat('Basic ', base64(variables('usernameAndPassword')))]"
    }

### <a id="concat"></a> concat - string
`concat (string1, string2, string3, ...)`  


组合多个字符串值并返回串联的字符串。

| 参数 | 必选 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| string1 |是 |String |串联的第一个值。 |
| 其他字符串 |否 |字符串 |要按顺序串联的其他值。 |

此函数可以使用任意数量的参数，并可接受字符串或数组作为参数。有关串联数组的示例，请参阅 [concat - 数组](#concatarray)。

以下示例演示了如何组合多个字符串值以返回串联的字符串。

    "outputs": {
        "siteUri": {
          "type": "string",
          "value": "[concat('http://', reference(resourceId('Microsoft.Web/sites', parameters('siteName'))).hostNames[0])]"
        }
    }


### <a id="lengthstring"></a> length - string
`length(string)`  


返回字符串中的字符数。

| 参数 | 必选 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| 字符串 |是 |String |用于获取字符数的值。 |

有关对数组使用 length 的示例，请参阅 [length - 数组](#length)。

以下示例返回字符串中的字符数。

    "parameters": {
        "appName": { "type": "string" }
    },
    "variables": { 
        "nameLength": "[length(parameters('appName'))]"
    }


### <a id="padleft"></a> padLeft
`padLeft(valueToPad, totalLength, paddingCharacter)`  


通过向左侧添加字符直至到达指定的总长度返回右对齐的字符串。

| 参数 | 必选 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| valueToPad |是 |字符串或整数 |要右对齐的值。 |
| totalLength |是 |Integer |返回字符串中的字符总数。 |
| paddingCharacter |否 |单个字符 |要用于向左填充直到达到总长度的字符。默认值为空格。 |

以下示例演示如何通过添加零字符直到字符串达到 10 个字符进而填充该用户提供的参数值。如果原始的参数值超过 10 个字符，将不会添加任何字符。

    "parameters": {
        "appName": { "type": "string" }
    },
    "variables": { 
        "paddedAppName": "[padLeft(parameters('appName'),10,'0')]"
    }

### <a id="replace"></a> replace
`replace(originalString, oldCharacter, newCharacter)`  


如果在所有实例中特定字符串中的一个字符已被替换为另一个字符时，则返回新的字符串。

| 参数 | 必选 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| originalString |是 |String |包含将某一个字符替换为另一个字符的所有实例的值。 |
| oldCharacter |是 |String |要从原始字符串中删除的字符。 |
| newCharacter |是 |String |要添加到已删除字符的位置的字符。 |

以下示例演示如何从用户提供的字符串中删除所有的短划线。

    "parameters": {
        "identifier": { "type": "string" }
    },
    "variables": { 
        "newidentifier": "[replace(parameters('identifier'),'-','')]"
    }

### <a id="skipstring"></a> skip - string
`skip(originalValue, numberToSkip)`  


返回一个字符串，其中包含字符串中指定数字后面的所有字符。

| 参数 | 必选 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| originalValue |是 |字符串 |用于跳过的字符串。 |
| numberToSkip |是 |Integer |要跳过的字符数。如果此值为 0 或更小，则返回字符串中的所有字符。如果此值大于字符串的长度，则返回空字符串。 |

有关对数组使用 skip 的示例，请参阅 [skip - 数组](#skip)。

以下示例将跳过字符串中指定数目的字符。

    "parameters": {
      "first": {
        "type": "string",
        "metadata": {
          "description": "Value to use for skipping"
        }
      },
      "second": {
        "type": "int",
        "metadata": {
          "description": "Number of characters to skip"
        }
      }
    },
    "resources": [
    ],
    "outputs": {
      "return": {
        "type": "string",
        "value": "[skip(parameters('first'),parameters('second'))]"
      }
    }


### <a id="split"></a> split
`split(inputString, delimiterString)`  


`split(inputString, delimiterArray)`  


返回包含输入字符串的子字符串的字符串数组，其中的子字符串使用指定的分隔符进行分隔。

| 参数 | 必选 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| inputString |是 |String |要拆分的字符串。 |
| delimiter |是 |字符串或字符串数组 |用于拆分字符串的分隔符。 |

以下示例使用逗号拆分输入字符串。

    "parameters": {
        "inputString": { "type": "string" }
    },
    "variables": { 
        "stringPieces": "[split(parameters('inputString'), ',')]"
    }

以下示例使用逗号或分号拆分输入字符串。

    "variables": {
      "stringToSplit": "test1,test2;test3",
      "delimiters": [ ",", ";" ]
    },
    "resources": [ ],
    "outputs": {
      "exampleOutput": {
        "value": "[split(variables('stringToSplit'), variables('delimiters'))]",
        "type": "array"
      }
    }

### <a id="string"></a> string
`string(valueToConvert)`  


将指定的值转换为字符串。

| 参数 | 必选 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| valueToConvert |是 | 任意 |要转换为字符串的值。可以转换任何类型的值，包括对象和数组。 |

以下示例将用户提供的参数值转换为字符串。

    "parameters": {
      "jsonObject": {
        "type": "object",
        "defaultValue": {
          "valueA": 10,
          "valueB": "Example Text"
        }
      },
      "jsonArray": {
        "type": "array",
        "defaultValue": [ "a", "b", "c" ]
      },
      "jsonInt": {
        "type": "int",
        "defaultValue": 5
      }
    },
    "variables": { 
      "objectString": "[string(parameters('jsonObject'))]",
      "arrayString": "[string(parameters('jsonArray'))]",
      "intString": "[string(parameters('jsonInt'))]"
    }

### <a id="substring"></a> substring
`substring(stringToParse, startIndex, length)`  


返回从指定的字符位置开始且包含指定数量的字符的子字符串。

| 参数 | 必选 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| stringToParse |是 |字符串 |从中提取子字符串的原始字符串。 |
| startIndex |否 |Integer |子字符串的从零开始的字符位置。 |
| length |否 |Integer |子字符串的字符数。 |

以下示例提取参数中的前三个字符。

    "parameters": {
        "inputString": { "type": "string" }
    },
    "variables": { 
        "prefix": "[substring(parameters('inputString'), 0, 3)]"
    }

### <a id="takestring"></a> take - string
`take(originalValue, numberToTake)`  


返回一个字符串，其中包含从字符串开头算起的指定数目的字符。

| 参数 | 必选 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| originalValue |是 |String |要从中获取字符的值。 |
| numberToTake |是 |Integer |要提取的字符数。如果此值为 0 或更小，则返回空字符串。如果此值大于给定字符串的长度，则返回字符串中的所有字符。 |

有关对数组使用 take 的示例，请参阅 [take - 数组](#take)。

以下示例将从字符串中提取指定数目的字符。

    "parameters": {
      "first": {
        "type": "string",
        "metadata": {
          "description": "Value to use for taking"
        }
      },
      "second": {
        "type": "int",
        "metadata": {
          "description": "Number of characters to take"
        }
      }
    },
    "resources": [
    ],
    "outputs": {
      "return": {
        "type": "string",
        "value": "[take(parameters('first'), parameters('second'))]"
      }
    }

### <a id="tolower"></a> toLower
`toLower(stringToChange)`  


将指定的字符串转换为小写。

| 参数 | 必选 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| stringToChange |是 |字符串 |要转换为小写的值。 |

以下示例将用户提供的参数值转换为小写。

    "parameters": {
        "appName": { "type": "string" }
    },
    "variables": { 
        "lowerCaseAppName": "[toLower(parameters('appName'))]"
    }

### <a id="toupper"></a> toUpper
`toUpper(stringToChange)`  


将指定的字符串转换为大写。

| 参数 | 必选 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| stringToChange |是 |String |要转换为大写的值。 |

以下示例将用户提供的参数值转换为大写。

    "parameters": {
        "appName": { "type": "string" }
    },
    "variables": { 
        "upperCaseAppName": "[toUpper(parameters('appName'))]"
    }

### <a id="trim"></a> trim
`trim (stringToTrim)`  


从指定的字符串中删除所有前导和尾随空白字符。

| 参数 | 必选 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| stringToTrim |是 |字符串 |要剪裁的值。 |

以下示例将裁剪掉用户提供的参数值中的空白字符。

    "parameters": {
        "appName": { "type": "string" }
    },
    "variables": { 
        "trimAppName": "[trim(parameters('appName'))]"
    }

### <a id="uniquestring"></a> uniqueString
`uniqueString (baseString, ...)`  


根据作为参数提供的值创建确定性哈希字符串。

| 参数 | 必选 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| baseString |是 |String |哈希函数中用于创建唯一字符串的值。 |
| 根据需要使用其他参数 |否 |字符串 |你可以添加任意数目的字符串，以创建指定唯一性级别的值。 |

当你需要创建资源的唯一名称时，此函数很有帮助。提供参数值，这些值用于限制结果的唯一性范围。可以指定该名称对于订阅、资源组或部署是否唯一。

返回的值不是随机字符串，而是哈希函数的结果。返回的值长度为 13 个字符。并非全局唯一。可能需要根据命名约定使用前缀来组合值，以创建有意义的名称。以下示例显示了返回值的格式。实际值取决于提供的参数。

    tcvhiyu5h2o5o

以下示例演示如何使用 uniqueString 创建通用级别唯一值。

仅对订阅唯一

    "[uniqueString(subscription().subscriptionId)]"

仅对资源组唯一

    "[uniqueString(resourceGroup().id)]"

仅对资源组的部署唯一

    "[uniqueString(resourceGroup().id, deployment().name)]"

以下示例演示显示如何根据资源组创建存储帐户的唯一名称。在资源组中，如果构造方式相同，则名称不唯一。

    "resources": [{ 
        "name": "[concat('storage', uniqueString(resourceGroup().id))]", 
        "type": "Microsoft.Storage/storageAccounts", 
        ...



### <a id="uri"></a> uri
`uri (baseUri, relativeUri)`  


通过组合 baseUri 和 relativeUri 字符串来创建绝对 URI。

| 参数 | 必选 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| baseUri |是 |String |基本 uri 字符串。 |
| relativeUri |是 |String |要添加到基本 uri 字符串的相对 uri 字符串。 |

**baseUri** 参数的值可包含特定文件，但在构造 URI 时，只使用基路径。例如，将 `http://contoso.com/resources/azuredeploy.json` 作为 baseUri 参数传递会生成 `http://contoso.com/resources/` 的基 URI。

以下示例演示如何根据父模板的值构造嵌套模板的链接。

    "templateLink": "[uri(deployment().properties.templateLink.uri, 'nested/azuredeploy.json')]"

## 数组函数
资源管理器提供以下用于处理数组值的函数。

* [concat](#concatarray)
* [length](#length)
* [skip](#skip)
* [take](#take)

若要获取由某个值分隔的字符串值数组，请参阅 [split](#split)。

### <a id="concatarray"></a> concat - array
`concat (array1, array2, array3, ...)`  


组合多个数组并返回串联的数组。

| 参数 | 必选 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| array1 |是 |Array |串联的第一个数组。 |
| 其他数组 |否 |Array |要按顺序串联的其他数组。 |

此函数可以使用任意数量的参数，并可接受字符串或数组作为参数。有关串联字符串值的示例，请参阅 [concat - 字符串](#concat)。

以下示例演示如何组合两个数组。

    "parameters": {
        "firstarray": {
            type: "array"
        }
        "secondarray": {
            type: "array"
        }
     },
     "variables": {
         "combinedarray": "[concat(parameters('firstarray'), parameters('secondarray'))]
     }


### <a id="length"></a> length - array
`length(array)`  


返回数组中的元素数。

| 参数 | 必选 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| 数组 |是 |Array |用于获取元素数目的数组。 |

创建资源时，可在数组中使用此函数指定迭代数。在以下示例中，参数 **siteNames** 引用创建网站时要使用的名称数组。

    "copy": {
        "name": "websitescopy",
        "count": "[length(parameters('siteNames'))]"
    }

有关在数组中使用此函数的详细信息，请参阅[在 Azure 资源管理器中创建多个资源实例](/documentation/articles/resource-group-create-multiple/)。

有关对字符串值使用 length 的示例，请参阅 [length - 字符串](#lengthstring)。

### <a id="skip"></a> skip - array
`skip(originalValue, numberToSkip)`  


返回一个数组，其中包含数组中指定数字后面的所有元素。

| 参数 | 必选 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| originalValue |是 |Array |用于跳过的数组。 |
| numberToSkip |是 |Integer |要跳过的元素数目。如果此值为 0 或更小，则返回数组中的所有元素。如果此值大于数组的长度，则返回空数组。 |

有关对字符串使用 skip 的示例，请参阅 [skip - 字符串](#skipstring)。

下面的示例将跳过数组中指定数目的元素。

    "parameters": {
      "first": {
        "type": "array",
        "metadata": {
          "description": "Values to use for skipping"
        },
        "defaultValue": [ "one", "two", "three" ]
      },
      "second": {
        "type": "int",
        "metadata": {
          "description": "Number of elements to skip"
        }
      }
    },
    "resources": [
    ],
    "outputs": {
      "return": {
        "type": "array",
        "value": "[skip(parameters('first'), parameters('second'))]"
      }
    }

### <a id="take"></a> take - array
`take(originalValue, numberToTake)`  


返回一个数组，其中包含从数组开头算起的指定数目的元素。

| 参数 | 必选 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| originalValue |是 |Array |要从中提取元素的数组。 |
| numberToTake |是 |Integer |要提取的元素数目。如果此值为 0 或更小，则返回空数组。如果此值大于给定数组的长度，则返回数组中的所有元素。 |

有关对字符串使用 take 的示例，请参阅 [take - 字符串](#takestring)。

下面的示例将从数组中获取指定个数的元素。

    "parameters": {
      "first": {
        "type": "array",
        "metadata": {
          "description": "Values to use for taking"
        },
        "defaultValue": [ "one", "two", "three" ]
      },
      "second": {
        "type": "int",
        "metadata": {
          "description": "Number of elements to take"
        }
      }
    },
    "resources": [
    ],
    "outputs": {
      "return": {
        "type": "array",
        "value": "[take(parameters('first'),parameters('second'))]"
      }
    }

## <a name="deployment-value-functions"></a> 部署值函数
资源管理器提供以下函数，用于从与部署相关的模板和值部分获取值：

* [部署](#deployment)
* [参数](#parameters)
* [variables](#variables)

若要从资源、资源组或订阅获取值，请参阅 [Resource functions](#resource-functions)（资源函数）。

### <a id="deployment"></a> deployment
`deployment()`  


返回有关当前部署操作的信息。

此函数返回部署期间传递的对象。根据部署对象是作为链接还是内联对象传递，所返回对象中的属性将有所不同。

如果部署对象是以内联形式传递的（例如使用 Azure PowerShell 中的 **-TemplateFile** 参数指向本地文件时），所返回的对象采用以下格式：

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

以下示例演示如何根据父模板的 URI，使用 deployment() 链接到另一个模板。

    "variables": {  
        "sharedTemplateUrl": "[uri(deployment().properties.templateLink.uri, 'shared-resources.json')]"  
    }  

### <a id="parameters"></a> parameters
`parameters (parameterName)`  


返回一个参数值。指定的参数名称必须已在模板的 parameters 节中定义。

| 参数 | 必选 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| parameterName |是 |String |要返回的参数名称。 |

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

### <a id="variables"></a> variables
`variables (variableName)`  


返回变量的值。指定的变量名称必须已在模板的 variables 节中定义。

| 参数 | 必选 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| variableName |是 |String |要返回的变量名称。 |

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

## <a name="resource-functions"></a> 资源函数
资源管理器提供以下用于获取资源值的函数：

* [listKeys 和 list{Value}](#listkeys)
* [providers](#providers)
* [reference](#reference)
* [resourceGroup](#resourcegroup)
* [resourceId](#resourceid)
* [订阅](#subscription)

若要从参数、变量或当前部署获取值，请参阅 [Deployment value functions](#deployment-value-functions)（部署值函数）。

### <a id="list" name="listkeys"></a> listKeys and list{Value}
`listKeys (resourceName or resourceIdentifier, apiVersion)`  


`list{Value} (resourceName or resourceIdentifier, apiVersion)`  


返回支持 list 操作的任何资源类型的值。最常见的用法是 **listKeys**。

| 参数 | 必选 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| resourceName 或 resourceIdentifier |是 |String |资源的唯一标识符。 |
| apiVersion |是 |String |资源运行时状态的 API 版本。通常采用 **yyyy-mm-dd** 格式。 |

以 **list** 开头的任何操作都可用作模板中的函数。可用操作不仅包括 **listKeys**，也包括 **list**、**listAdminKeys** 和 **listStatus** 等操作。若要确定有列表操作的资源类型，请使用以下 PowerShell 命令：

    Get-AzureRmProviderOperation -OperationSearchString *  | where {$_.Operation -like "*list*"} | FT Operation

或者，使用 Azure CLI 来检索列表。以下示例将检索 **apiapps** 的所有操作，并使用 JSON 实用工具 [jq](http://stedolan.github.io/jq/download/) 来只筛选出 list 操作。

    azure provider operations show --operationSearchString */apiapps/* --json | jq ".[] | select (.operation | contains("list"))"

可以使用 [resourceId 函数](#resourceid)或使用格式 **{providerNamespace}/{resourceType}/{resourceName}** 指定 resourceId。

以下示例演示如何从 outputs 节中的存储帐户返回主密钥和辅助密钥。

    "outputs": { 
      "listKeysOutput": { 
        "value": "[listKeys(resourceId('Microsoft.Storage/storageAccounts', parameters('storageAccountName')), '2016-01-01')]", 
        "type" : "object" 
      } 
    } 

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

### <a id="providers"></a> providers
`providers (providerNamespace, [resourceType])`  


返回有关资源提供程序及其支持的资源类型的信息。如果未提供资源类型，该函数将返回资源提供程序支持的所有类型。

| 参数 | 必选 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| providerNamespace |是 |字符串 |提供程序的命名空间 |
| resourceType |否 |String |指定的命名空间中的资源类型。 |

将使用以下格式返回支持的每个类型。不保证数组按顺序排列。

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

### <a id="reference"></a> reference
`reference (resourceName or resourceIdentifier, [apiVersion])`  


返回表示另一个资源的运行时状态的对象。

| 参数 | 必选 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| resourceName 或 resourceIdentifier |是 |String |资源的名称或唯一标识符。 |
| apiVersion |否 |String |指定的资源的 API 版本。如果资源不是在同一模板中预配的，请包含此参数。通常采用 **yyyy-mm-dd** 格式。 |

**reference** 函数从运行时状态派生其值，因此不能在 variables 节中使用。可以在模板的 outputs 节中使用它。

如果在相同的模板内设置了引用的资源，则可使用 reference 函数来隐式声明一个资源依赖于另一个资源。另外，不需要使用 **dependsOn** 属性。只有当引用的资源已完成部署后，才会对函数求值。

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
            "value": "[reference(concat('Microsoft.Storage/storageAccounts/', parameters('storageAccountName')), '2016-01-01')]",
            "type" : "object"
        }
    }

可从返回对象（例如 blob 终结点 URI）中检索特定值，如以下示例所示：

    "outputs": {
        "BlobUri": {
            "value": "[reference(concat('Microsoft.Storage/storageAccounts/', parameters('storageAccountName')), '2016-01-01').primaryEndpoints.blob]",
            "type" : "string"
        }
    }

以下示例引用不同资源组中的存储帐户。

    "outputs": {
        "BlobUri": {
            "value": "[reference(resourceId(parameters('relatedGroup'), 'Microsoft.Storage/storageAccounts/', parameters('storageAccountName')), '2016-01-01').primaryEndpoints.blob]",
            "type" : "string"
        }
    }

从 **reference** 函数中返回的对象上的属性因资源类型而异。若要查看资源类型的属性名称和值，请创建一个简单模板，该模板返回 **outputs** 部分中的对象。如果有现有的该类型的资源，则模板只返回对象而不部署任何新资源。如果没有现有的该类型的资源，则模板只部署该类型并返回对象。然后，将这些属性添加到需要在部署期间动态检索值的其他模板。

### <a id="resourcegroup"></a> resourceGroup
`resourceGroup()`  


返回表示当前资源组的对象。

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

### <a id="resourceid"></a> resourceId
`resourceId ([subscriptionId], [resourceGroupName], resourceType, resourceName1, [resourceName2]...)`  


返回资源的唯一标识符。

| 参数 | 必选 | 类型 | 说明 |
|:--- |:--- |:--- |:--- |
| subscriptionId |否 |字符串（GUID 格式） |默认值为当前订阅。如果需要检索另一个订阅中的资源，请指定此值。 |
| resourceGroupName |否 |字符串 |默认值为当前资源组。如果需要检索另一个资源组中的资源，请指定此值。 |
| resourceType |是 |字符串 |资源类型，包括资源提供程序命名空间。 |
| resourceName1 |是 |String |资源的名称。 |
| resourceName2 |否 |字符串 |下一个资源名称段（如果资源是嵌套的）。 |

如果资源名称不确定或未设置在相同的模板内，请使用此函数。将使用以下格式返回标识符：

    /subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/{resourceProviderNamespace}/{resourceType}/{resourceName}

以下示例演示了如何检索网站和数据库的资源 ID。网站存在于名为 **myWebsitesGroup** 的资源组中，而数据库存在于此模板的当前资源组中。

    [resourceId('myWebsitesGroup', 'Microsoft.Web/sites', parameters('siteName'))]
    [resourceId('Microsoft.SQL/servers/databases', parameters('serverName'), parameters('databaseName'))]

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

### <a id="subscription"></a> subscription
`subscription()`  


将使用以下格式返回有关订阅的详细信息：

    {
        "id": "/subscriptions/#####",
        "subscriptionId": "#####",
        "tenantId": "#####"
    }

以下示例显示了在 outputs 节中调用的 subscription 函数。

    "outputs": { 
      "exampleOutput": { 
          "value": "[subscription()]", 
          "type" : "object" 
      } 
    } 


## 后续步骤
* 有关 Azure 资源管理器模板中对各部分的说明，请参阅[创作 Azure 资源管理器模板](/documentation/articles/resource-group-authoring-templates/)
* 若要合并多个模板，请参阅[将已链接的模板与 Azure 资源管理器配合使用](/documentation/articles/resource-group-linked-templates/)
* 若要在创建资源类型时迭代指定的次数，请参阅[在 Azure 资源管理器中创建多个资源实例](/documentation/articles/resource-group-create-multiple/)
* 若要查看如何部署已创建的模板，请参阅[使用 Azure 资源管理器模板部署应用程序](/documentation/articles/resource-group-template-deploy/)

<!---HONumber=Mooncake_0103_2017-->