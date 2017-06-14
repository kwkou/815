<properties
    pageTitle="Azure 资源策略 | Azure"
    description="介绍如何使用 Azure Resource Manager 来确保在部署过程中设置一致的资源属性。 可在订阅或资源组中应用策略。"
    services="azure-resource-manager"
    documentationcenter="na"
    author="tfitzmac"
    manager="timlt"
    editor="tysonn"
    translationtype="Human Translation" />
<tags
    ms.assetid="abde0f73-c0fe-4e6d-a1ee-32a6fce52a2d"
    ms.service="azure-resource-manager"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="03/30/2017"
    wacn.date="06/05/2017"
    ms.author="tomfitz"
    ms.sourcegitcommit="78da854d58905bc82228bcbff1de0fcfbc12d5ac"
    ms.openlocfilehash="b5436e7de21f3358006a33f8f0439150f9fe7a43"
    ms.lasthandoff="04/22/2017" />

# <a name="resource-policy-overview"></a>资源策略概述
使用资源策略可在组织中建立资源约定。 通过定义约定，可以控制成本并更轻松地管理资源。 例如，可以指定仅允许某些类型的虚拟机，或要求所有资源都带有特定的标记。 策略由所有子资源继承。 因此，如果将策略应用到资源组，则其适用于该资源组中的所有资源。

了解策略的两个概念：

* 策略定义 - 描述何时强制执行策略，以及要采取的操作
* 策略分配 - 应用策略定义的范围（订阅或资源组）

<!-- Policy not supported on Azure.cn-->
<!--本主题重点介绍策略定义。有关策略分配的信息，请参阅[分配和管理策略](/documentation/articles/resource-manager-policy-create-assign/)。-->

Azure 提供一些内置的策略定义，可减少需要定义的策略数目。如果内置策略定义适用于你的方案，请在分配到范围时使用该定义。

将在创建和更新资源（PUT 和 PATCH 操作）时评估策略。

> [AZURE.NOTE]
> 当前，策略不对不支持标记、种类和位置的资源类型进行评估，例如 Microsoft.Resources/deployments 资源类型。 将来会添加此支持。 若要避免向后兼容问题，创作策略时应显式指定类型。 例如，未指定类型的标记策略应用于所有类型。 在此情况下，如果有嵌套资源不支持标记，并且部署资源类型已添加到策略评估中，则模板部署可能会失败。 
> 
> 

## <a name="how-is-it-different-from-rbac"></a>策略与 RBAC 有什么不同？
策略和基于角色的访问控制 (RBAC) 之间存在一些主要区别。 RBAC 关注不同范围内的**用户**操作。 例如，将你添加到所需范围的资源组的参与者角色后，你便可以对该资源组做出更改。 策略关注部署期间的 **资源** 属性。 例如，通过策略，你可以控制可预配的资源类型，或限制可以预配资源的位置。 不同于 RBAC，策略是默认的允许和明确拒绝系统。 

若要使用策略，用户必须通过 RBAC 进行身份验证。 具体而言，帐户需要：

* `Microsoft.Authorization/policydefinitions/write` 权限
* `Microsoft.Authorization/policyassignments/write` 权限 

**参与者** 角色不包括这些权限。

## <a name="policy-definition-structure"></a>策略定义结构
使用 JSON 创建策略定义。 策略定义包含以下各项的元素：

* 参数
* 显示名称
* description
* 策略规则
  * 逻辑求值
  * 效果

以下示例演示了一个限制资源部署位置的策略：

    {
      "properties": {
        "parameters": {
          "allowedLocations": {
            "type": "array",
            "metadata": {
              "description": "The list of locations that can be specified when deploying resources",
              "strongType": "location",
              "displayName": "Allowed locations"
            }
          }
        },
        "displayName": "Allowed locations",
        "description": "This policy enables you to restrict the locations your organization can specify when deploying resources.",
        "policyRule": {
          "if": {
            "not": {
              "field": "location",
              "in": "[parameters('allowedLocations')]"
            }
          },
          "then": {
            "effect": "deny"
          }
        }
      }
    }

## <a name="parameters"></a>参数
使用参数可减少策略定义的数量，有助于简化策略管理。 为资源属性定义策略（如限制资源部署的位置），并在定义中包含参数。 然后，通过在分配策略时传递不同的值（例如为订阅指定一组位置），针对不同的方案重复使用该策略定义。

在创建策略定义时声明参数。

    "parameters": {
      "allowedLocations": {
        "type": "array",
        "metadata": {
          "description": "The list of allowed locations for resources.",
          "displayName": "Allowed locations"
        }
      }
    }

参数类型可以是字符串，也可以是数组。 Azure 门户等工具使用元数据属性显示用户友好信息。 

在策略规则中，使用下列语法引用参数： 

    { 
        "field": "location",
        "in": "[parameters('allowedLocations')]"
    }

## <a name="display-name-and-description"></a>显示名称和说明

使用**显示名称**和**说明**来标识策略定义，并提供其使用情景。

## <a name="policy-rule"></a>策略规则

策略规则包括 **If** and **Then** 块。 在 **If** 块中，定义强制执行策略时指定的一个或多个条件。 可以对这些条件应用逻辑运算符，以精确定义策略的方案。 在 **Then** 块中，定义满足 **If** 条件时产生的效果。

    {
      "if" : {
          <condition> | <logical operator>
      },
      "then" : {
          "effect" : "deny | audit | append"
      }
    }

### <a name="logical-operators"></a>逻辑运算符
支持的逻辑运算符为：

* `"not": {condition or operator}`
* `"allOf": [{condition or operator},{condition or operator}]`
* `"anyOf": [{condition or operator},{condition or operator}]`

**not** 语法反转条件的结果。 **allOf** 语法（与逻辑 **And** 操作相似）要求所有条件为 true。 **anyOf** 语法（与逻辑 **Or** 操作相似）要求一个或多个条件为 true。

可以嵌套逻辑运算符。 以下示例显示了嵌套在 **And** 操作中的 **Not** 操作。 

    "if": {
      "allOf": [
        {
          "not": {
            "field": "tags",
            "containsKey": "application"
          }
        },
        {
          "field": "type",
          "equals": "Microsoft.Storage/storageAccounts"
        }
      ]
    },

### <a name="conditions"></a>条件
条件评估 **字段** 是否符合特定的准则。 支持的条件有：

* `"equals": "value"`
* `"like": "value"`
* `"contains": "value"`
* `"in": ["value1","value2"]`
* `"containsKey": "keyName"`
* `"exists": "bool"`  

在使用 **like** 条件时，可以在值中提供通配符 (*)。

### <a name="fields"></a>字段
使用字段构成条件。 字段显示用于描述资源状态的资源请求负载属性。  

支持以下字段：

* `name`
* `kind`
* `type`
* `location`
* `tags`
* `tags.*` 
* 属性别名

使用属性别名来访问资源类型的特定属性。 支持的别名为：

* Microsoft.CDN/profiles/sku.name
* Microsoft.Compute/virtualMachines/imageOffer
* Microsoft.Compute/virtualMachines/imagePublisher
* Microsoft.Compute/virtualMachines/sku.name
* Microsoft.Compute/virtualMachines/imageSku
* Microsoft.Compute/virtualMachines/imageVersion
* Microsoft.SQL/servers/databases/edition
* Microsoft.SQL/servers/databases/elasticPoolName
* Microsoft.SQL/servers/databases/requestedServiceObjectiveId
* Microsoft.SQL/servers/databases/requestedServiceObjectiveName
* Microsoft.SQL/servers/elasticPools/dtu
* Microsoft.SQL/servers/elasticPools/edition
* Microsoft.SQL/servers/version
* Microsoft.Storage/storageAccounts/accessTier
* Microsoft.Storage/storageAccounts/enableBlobEncryption
* Microsoft.Storage/storageAccounts/sku.name
* Microsoft.Web/serverFarms/sku.name

### <a name="effect"></a>效果
策略支持三种类型的效果 - `deny`、`audit` 和 `append`。 

* **Deny** 将在审核日志中生成一个事件，并使请求失败
* **Audit** 将在审核日志中生成一个警告事件，但不会使请求失败
* **Append** 会将定义的字段集添加到请求 

对于 **append**，必须提供以下详细信息：

    "effect": "append",
    "details": [
      {
        "field": "field name",
        "value": "value of the field"
      }
    ]

值可以是字符串或 JSON 格式对象。 

<!-- Policy unable on Azure.cn-->
<!-- ## <a name="policy-examples"></a>策略示例

以下主题包含策略示例：
* 有关标记策略的示例，请参阅[将资源策略应用于标记](/documentation/articles/resource-manager-policy-tags/)。
* 有关存储策略的示例，请参阅[将资源策略应用于存储帐户](/documentation/articles/resource-manager-policy-storage/)。
-->
### <a name="allowed-resource-locations"></a>允许的资源位置
若要指定允许的位置，请参阅[策略定义结构](#policy-definition-structure)部分中的示例。若要分配此策略定义，请使用带有资源 ID `/providers/Microsoft.Authorization/policyDefinitions/e56962a6-4747-49cd-b67b-bf8b01975c4c` 的内置策略。

### <a name="not-allowed-resource-locations"></a>不允许的资源位置
若要指定不允许的位置，请使用以下策略定义：

    {
      "properties": {
        "parameters": {
          "notAllowedLocations": {
            "type": "array",
            "metadata": {
              "description": "The list of locations that are not allowed when deploying resources",
              "strongType": "location",
              "displayName": "Not allowed locations"
            }
          }
        },
        "displayName": "Not allowed locations",
        "description": "This policy enables you to block locations that your organization can specify when deploying resources.",
        "policyRule": {
          "if": {
            "field": "location",
            "in": "[parameters('notAllowedLocations')]"
          },
          "then": {
            "effect": "deny"
          }
        }
      }
    }

### <a name="allowed-resource-types"></a>允许的资源类型
以下示例显示只允许对 Microsoft.Resources、Microsoft.Compute、Microsoft.Storage 和 Microsoft.Network 资源类型进行部署的策略。 将拒绝其他所有资源类型：

    {
      "if" : {
        "not" : {
          "anyOf" : [
            {
              "field" : "type",
              "like" : "Microsoft.Resources/*"
            },
            {
              "field" : "type",
              "like" : "Microsoft.Compute/*"
            },
            {
              "field" : "type",
              "like" : "Microsoft.Storage/*"
            },
            {
              "field" : "type",
              "like" : "Microsoft.Network/*"
            }
          ]
        }
      },
      "then" : {
        "effect" : "deny"
      }
    }

### <a name="set-naming-convention"></a>设置命名约定
以下示例演示如何使用 **like** 条件支持的通配符。 该条件指明，如果名称符合所述模式 (namePrefix\*nameSuffix)，则拒绝请求：

    {
      "if" : {
        "not" : {
          "field" : "name",
          "like" : "namePrefix*nameSuffix"
        }
      },
      "then" : {
        "effect" : "deny"
      }
    }

## <a name="next-steps"></a>后续步骤
* 如需了解企业如何使用 Resource Manager 对订阅进行有效管理，请参阅 [Azure 企业机架 - 规范性订阅管理](/documentation/articles/resource-manager-subscription-governance/)。
* 该策略架构已在 [http://schema.management.azure.com/schemas/2015-10-01-preview/policyDefinition.json](http://schema.management.azure.com/schemas/2015-10-01-preview/policyDefinition.json) 中发布。

<!-- Update_Description:update meta properties -->