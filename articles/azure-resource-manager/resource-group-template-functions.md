<properties
    pageTitle="Resource Manager 模板函数 | Azure"
    description="介绍在 Azure Resource Manager 模板中检索值、处理字符串和数字以及检索部署信息时所用的函数。"
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
    ms.date="04/26/2017"
    wacn.date="06/05/2017"
    ms.author="v-yeche"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="78da854d58905bc82228bcbff1de0fcfbc12d5ac"
    ms.openlocfilehash="ec60a13abe94fdaa15afc0e53973ac4d632f1f13"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/26/2017" />

# <a name="azure-resource-manager-template-functions"></a>Azure Resource Manager 模板函数
本主题介绍可以在 Azure Resource Manager 模板中使用的所有函数。

模板函数及其参数不区分大小写。 例如，Resource Manager 将 **variables('var1')** 和 **VARIABLES('VAR1')** 视为相同。 在求值时，除非函数明确修改大小写（例如，使用 toUpper 或 toLower 进行修改），否则函数将保留大小写。 某些资源类型可能会提出大小写要求，而不考虑函数求值方式。

<a id="array" />
<a id="coalesce" />
<a id="concatarray" />
<a id="contains" />
<a id="createarray" />
<a id="empty" />
<a id="first" />
<a id="intersection" />
<a id="last" />
<a id="length" />
<a id="min" />
<a id="max" />
<a id="range" />
<a id="skip" />
<a id="take" />
## <a id="union"></a> 数组和对象函数
Resource Manager 提供以下用于处理数组和对象的函数。

* [array](/documentation/articles/resource-group-template-functions-array/#array)
* [coalesce](/documentation/articles/resource-group-template-functions-array/#coalesce)
* [concat](/documentation/articles/resource-group-template-functions-array/#concat)
* [contains](/documentation/articles/resource-group-template-functions-array/#contains)
* [createArray](/documentation/articles/resource-group-template-functions-array/#createarray)
* [empty](/documentation/articles/resource-group-template-functions-array/#empty)
* [first](/documentation/articles/resource-group-template-functions-array/#first)
* [intersection](/documentation/articles/resource-group-template-functions-array/#intersection)
* [last](/documentation/articles/resource-group-template-functions-array/#last)
* [length](/documentation/articles/resource-group-template-functions-array/#length)
* [min](/documentation/articles/resource-group-template-functions-array/#min)
* [max](/documentation/articles/resource-group-template-functions-array/#max)
* [range](/documentation/articles/resource-group-template-functions-array/#range)
* [skip](/documentation/articles/resource-group-template-functions-array/#skip)
* [take](/documentation/articles/resource-group-template-functions-array/#take)
* [union](/documentation/articles/resource-group-template-functions-array/#union)

<a id="equals" />
<a id="less" />
<a id="lessorequals" />
<a id="greater" />
## <a id="greaterorequals"></a> 比较函数
Resource Manager 提供了多个用于在模板中进行比较的函数。

* [equals](/documentation/articles/resource-group-template-functions-comparison/#equals)
* [less](/documentation/articles/resource-group-template-functions-comparison/#less)
* [lessOrEquals](/documentation/articles/resource-group-template-functions-comparison/#lessorequals)
* [greater](/documentation/articles/resource-group-template-functions-comparison/#greater)
* [greaterOrEquals](/documentation/articles/resource-group-template-functions-comparison/#greaterorequals)

<a id="deployment" />
<a id="parameters" />
## <a id="variables"></a> 部署值函数
Resource Manager 提供以下函数，用于从与部署相关的模板和值部分获取值：

* [部署](/documentation/articles/resource-group-template-functions-deployment/#deployment)
* [参数](/documentation/articles/resource-group-template-functions-deployment/#parameters)
* [variables](/documentation/articles/resource-group-template-functions-deployment/#variables)

<a id="add" />
<a id="copyindex" />
<a id="div" />
<a id="float" />
<a id="int" />
<a id="minint" />
<a id="maxint" />
<a id="mod" />
<a id="mul" />
## <a id="sub"></a> 数值函数
Resource Manager 提供以下用于处理整数的函数：

* [添加](/documentation/articles/resource-group-template-functions-numeric/#add)
* [copyIndex](/documentation/articles/resource-group-template-functions-numeric/#copyindex)
* [div](/documentation/articles/resource-group-template-functions-numeric/#div)
* [float](/documentation/articles/resource-group-template-functions-numeric/#float)
* [int](/documentation/articles/resource-group-template-functions-numeric/#int)
* [min](/documentation/articles/resource-group-template-functions-numeric/#min)
* [max](/documentation/articles/resource-group-template-functions-numeric/#max)
* [mod](/documentation/articles/resource-group-template-functions-numeric/#mod)
* [mul](/documentation/articles/resource-group-template-functions-numeric/#mul)
* [sub](/documentation/articles/resource-group-template-functions-numeric/#sub)

<a id="listkeys" />
<a id="list" />
<a id="providers" />
<a id="reference" />
<a id="resourcegroup" />
<a id="resourceid" />
## <a id="subscription"></a> 资源函数
Resource Manager 提供以下用于获取资源值的函数：

* [listKeys 和 list{Value}](/documentation/articles/resource-group-template-functions-resource/#listkeys)
* [providers](/documentation/articles/resource-group-template-functions-resource/#providers)
* [reference](/documentation/articles/resource-group-template-functions-resource/#reference)
* [resourceGroup](/documentation/articles/resource-group-template-functions-resource/#resourcegroup)
* [resourceId](/documentation/articles/resource-group-template-functions-resource/#resourceid)
* [subscription](/documentation/articles/resource-group-template-functions-resource/#subscription)

<a id="base64" />
<a id="base64tojson" />
<a id="base64tostring" />
<a id="bool" />
<a id="concat" />
<a id="containsstring" />
<a id="datauri" />
<a id="datauritostring" />
<a id="emptystring" />
<a id="endswith" />
<a id="firststring" />
<a id="indexof" />
<a id="laststring" />
<a id="lastindexof" />
<a id="lengthstring" />
<a id="padleft" />
<a id="replace" />
<a id="skipstring" />
<a id="split" />
<a id="startswith" />
<a id="string" />
<a id="substring" />
<a id="takestring" />
<a id="tolower" />
<a id="toupper" />
<a id="trim" />
<a id="uniquestring" />
<a id="uri" />
<a id="uricomponent" />
## <a id="uricomponenttostring"></a> 字符串函数
Resource Manager 提供以下用于处理字符串的函数：

* [base64](/documentation/articles/resource-group-template-functions-string/#base64)
* [base64ToJson](/documentation/articles/resource-group-template-functions-string/#base64tojson)
* [base64ToString](/documentation/articles/resource-group-template-functions-string/#base64tostring)
* [bool](/documentation/articles/resource-group-template-functions-string/#bool)
* [concat](/documentation/articles/resource-group-template-functions-string/#concat)
* [contains](/documentation/articles/resource-group-template-functions-string/#contains)
* [dataUri](/documentation/articles/resource-group-template-functions-string/#datauri)
* [dataUriToString](/documentation/articles/resource-group-template-functions-string/#datauritostring)
* [empty](/documentation/articles/resource-group-template-functions-string/#empty)
* [endsWith](/documentation/articles/resource-group-template-functions-string/#endswith)
* [first](/documentation/articles/resource-group-template-functions-string/#first)
* [indexOf](/documentation/articles/resource-group-template-functions-string/#indexof)
* [last](/documentation/articles/resource-group-template-functions-string/#last)
* [lastIndexOf](/documentation/articles/resource-group-template-functions-string/#lastindexof)
* [length](/documentation/articles/resource-group-template-functions-string/#length)
* [padLeft](/documentation/articles/resource-group-template-functions-string/#padleft)
* [replace](/documentation/articles/resource-group-template-functions-string/#replace)
* [skip](/documentation/articles/resource-group-template-functions-string/#skip)
* [split](/documentation/articles/resource-group-template-functions-string/#split)
* [startsWith](/documentation/articles/resource-group-template-functions-string/#startswith)
* [字符串](/documentation/articles/resource-group-template-functions-string/#string)
* [substring](/documentation/articles/resource-group-template-functions-string/#substring)
* [take](/documentation/articles/resource-group-template-functions-string/#take)
* [toLower](/documentation/articles/resource-group-template-functions-string/#tolower)
* [toUpper](/documentation/articles/resource-group-template-functions-string/#toupper)
* [trim](/documentation/articles/resource-group-template-functions-string/#trim)
* [uniqueString](/documentation/articles/resource-group-template-functions-string/#uniquestring)
* [uri](/documentation/articles/resource-group-template-functions-string/#uri)
* [uriComponent](/documentation/articles/resource-group-template-functions-string/#uricomponent)
* [uriComponentToString](/documentation/articles/resource-group-template-functions-string/#uricomponenttostring)

## <a name="next-steps"></a>后续步骤
* 有关 Azure Resource Manager 模板中各部分的说明，请参阅 [Authoring Azure Resource Manager templates](/documentation/articles/resource-group-authoring-templates/)（创作 Azure Resource Manager 模板）
* 若要合并多个模板，请参阅 [Using linked templates with Azure Resource Manager](/documentation/articles/resource-group-linked-templates/)（将链接的模板与 Azure Resource Manager 配合使用）
* 若要在创建资源类型时迭代指定的次数，请参阅 [Create multiple instances of resources in Azure Resource Manager](/documentation/articles/resource-group-create-multiple/)（在 Azure Resource Manager 中创建多个资源实例）
* 若要查看如何部署已创建的模板，请参阅 [Deploy an application with Azure Resource Manager template](/documentation/articles/resource-group-template-deploy/)（使用 Azure Resource Manager 模板部署应用程序）

<!--Update_Description:update meta properties; update article format to replace code block with link reference-->