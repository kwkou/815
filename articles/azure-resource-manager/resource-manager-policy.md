<properties
    pageTitle="Azure Resource Manager 策略 | Azure"
    description="介绍如何使用 Azure 资源管理器策略来防止订阅、资源组或单个资源等不同的范围发生冲突。"
    services="azure-resource-manager"
    documentationcenter="na"
    author="ravbhatnagar"
    manager="timlt"
    editor="tysonn" />
<tags
    ms.assetid="abde0f73-c0fe-4e6d-a1ee-32a6fce52a2d"
    ms.service="azure-resource-manager"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="12/07/2016"
    wacn.date="01/06/2017"
    ms.author="gauravbh;tomfitz" />

# 使用策略来管理资源和控制访问
借助 Azure Resource Manager，可通过自定义策略控制访问。使用策略可以防止组织中的用户违反管理组织资源所需的惯例。

你可以创建策略定义来描述会明确遭到拒绝的操作或资源。可以在所需范围（例如订阅、资源组或是单个资源）分配这些策略定义。策略由所有子资源继承。因此，如果将策略应用到资源组，则其适用于该资源组中的所有资源。

本文介绍可用于创建策略的策略定义语言的基本结构，并介绍如何在不同的范围应用这些策略。

## 策略与 RBAC 有什么不同？
策略与基于角色的访问控制之间有几个重要的差别，但首先你必须了解策略是和 RBAC 一起工作的。若要使用策略，用户必须通过 RBAC 进行身份验证。不同于 RBAC，策略是默认的允许和明确拒绝系统。

RBAC 着重于**用户**在不同的范围可执行的操作。例如，将特定用户添加到所需范围的资源组的参与者角色后，该用户便可对该资源组做出更改。

策略着重于各种范围的**资源**操作。例如，通过策略，你可以控制可预配的资源类型，或限制可以预配资源的位置。

## 常见方案
一个常见方案是为了费用分摊而要求提供部门标记。组织可能只想在关联相应的成本中心时允许操作，否则会拒绝请求。组织可以使用此策略通过相应的成本中心收取所执行操作的费用。

另一个常见方案是组织可能想要控制创建资源的位置。或者它们可能想要通过仅允许预配特定类型的资源，来控制对资源的访问。

同样地，组织可以控制服务类别或为资源强制运行所需的命名约定。

使用策略可轻松实现这些方案。

## 策略定义结构
策略定义是使用 JSON 创建的。它包含定义操作和效果的一个或多个条件/逻辑运算符，告诉你满足条件时产生的效果。该架构在 [http://schema.management.azure.com/schemas/2015-10-01-preview/policyDefinition.json](http://schema.management.azure.com/schemas/2015-10-01-preview/policyDefinition.json) 中发布。

以下示例说明可用于限制资源部署位置的策略：

    {
      "properties": {
        "parameters": {
          "listOfAllowedLocations": {
            "type": "array",
            "metadata": {
              "description": "An array of permitted locations for resources.",
              "strongType": "location",
              "displayName": "List of locations"
            }
          }
        },
        "displayName": "Geo-compliance policy template",
        "description": "This policy enables you to restrict the locations your organization can specify when deploying resources. Use to enforce your geo-compliance requirements.",
        "policyRule": {
          "if": {
            "not": {
              "field": "location",
              "in": "[parameters('listOfAllowedLocations')]"
            }
          },
          "then": {
            "effect": "deny"
          }
        }
      }
    }

基本而言，策略包含以下部分：

**参数**：分配策略时指定的值。

**条件/逻辑运算符**:可通过一组逻辑运算符操作的条件集。

**影响**：条件满足时会发生的情况 – 拒绝或审核。审核效果会发出警告事件服务日志。例如，管理员可以创建策略，即使有人创建大型 VM，此策略也会引发审核事件，然后管理员可以审查日志。

    {
      "if" : {
          <condition> | <logical operator>
      },
      "then" : {
          "effect" : "deny | audit | append"
      }
    }

## 策略评估
创建资源时会对策略进行评估。部署模板时，将在模板中的每个资源创建期间评估策略。

> [AZURE.NOTE]
当前，策略不对不支持标记、种类和位置的资源类型进行评估，例如 Microsoft.Resources/deployments 资源类型。将来会添加此支持。若要避免向后兼容问题，创作策略时应显式指定类型。例如，未指定类型的标记策略应用于所有类型。在此情况下，如果有嵌套资源不支持标记，并且部署资源类型已添加到策略评估中，则模板部署可能会失败。
> 
> 

## 参数
从 API 版本 2016-12-01 开始，可在策略定义中使用参数。使用参数可减少策略定义的数量，有助于简化策略管理。在分配策略时向参数提供值。

在创建策略定义时声明参数。

    "parameters": {
      "listOfLocations": {
        "type": "array",
        "metadata": {
          "description": "An array of permitted locations for resources.",
          "displayName": "List Of Locations"
        }
      }
    }

参数类型可以是字符串，也可以是数组。Azure 门户预览等工具使用元数据属性显示用户友好信息。

在策略规则中，可按照与模板类似的方式引用参数。例如：
        
    { 
        "field" : "location",
        "in" : "[parameters(listOfLocations)]"
    }

## 逻辑运算符
支持的逻辑运算符和语法包括：

| 运算符名称 | 语法 |
|:--- |:--- |
| Not |"not" : {&lt;condition or operator &gt;} |
| And |"allOf" : [ {&lt;condition or operator &gt;},{&lt;condition or operator &gt;}] |
| 或 |"anyOf" : [ {&lt;condition or operator &gt;},{&lt;condition or operator &gt;}] |

资源管理器可让你通过嵌套的运算符在策略中指定复杂逻辑。例如，你可以拒绝在指定资源类型的特定位置创建资源。本主题包含嵌套运算符的示例。

## <a name="conditions"></a> 条件
条件评估**字段**或**源**是否符合特定的准则。支持的条件名称和语法包括：

| 条件名称 | 语法 |
|:--- |:--- |
| 等于 |"equals" : "&lt;value&gt;" |
| Like |"like" : "&lt;value&gt;" |
| Contains |"contains" : "&lt;value&gt;" |
| In |"in" : [ "&lt;value1&gt;","&lt;value2&gt;" ] |
| ContainsKey |"containsKey" : "&lt;keyName&gt;" |
| Exists |"exists" : "&lt;bool&gt;" |

### 字段
条件是使用字段和源构成的。字段显示用于描述资源状态的资源请求负载属性。源表示请求本身的特征。

支持以下字段和源：

字段：**name**、**kind**、**type**、**location**、**tags**、**tags.*** 和 **property alias**。

### 属性别名
属性别名可在策略定义中用于访问资源类型特定属性，例如设置和 SKU。它适用于所有具有属性的 API 版本。可以通过 REST API 检索别名（将来会增加 Powershell 支持）：

    GET /subscriptions/{id}/providers?$expand=resourceTypes/aliases&api-version=2015-11-01

以下示例演示了别名的定义。如你所见，别名在不同的 API 版本中定义路径，无论属性名称是否更改。

    "aliases": [
        {
          "name": "Microsoft.Storage/storageAccounts/sku.name",
          "paths": [
            {
              "path": "properties.accountType",
              "apiVersions": [
                "2015-06-15",
                "2015-05-01-preview"
              ]
            },
            {
              "path": "sku.name",
              "apiVersions": [
                "2016-01-01"
              ]
            }
          ]
        }
    ]

目前，支持的别名为：

| 别名名称 | 说明 |
| --- | --- |
| {resourceType}/sku.name |支持的资源类型为：Microsoft.Compute/virtualMachines、<br />Microsoft.Storage/storageAccounts、<br />Microsoft.Web/serverFarms、<br />Microsoft.Scheduler/jobcollections、<br />Microsoft.DocumentDB/databaseAccounts、<br />Microsoft.Cache/Redis、<br />Microsoft.CDN/profiles |
| {resourceType}/sku.family |支持的资源类型为 Microsoft.Cache/Redis |
| {resourceType}/sku.capacity |支持的资源类型为 Microsoft.Cache/Redis |
| Microsoft.Compute/virtualMachines/imagePublisher | |
| Microsoft.Compute/virtualMachines/imageOffer | |
| Microsoft.Compute/virtualMachines/imageSku | |
| Microsoft.Compute/virtualMachines/imageVersion | |
| Microsoft.Storage/storageAccounts/accessTier | |
| Microsoft.Storage/storageAccounts/enableBlobEncryption | |
| Microsoft.Cache/Redis/enableNonSslPort | |
| Microsoft.Cache/Redis/shardCount | |
| Microsoft.SQL/servers/version | |
| Microsoft.SQL/servers/databases/requestedServiceObjectiveId | |
| Microsoft.SQL/servers/databases/requestedServiceObjectiveName | |
| Microsoft.SQL/servers/databases/edition | |
| Microsoft.SQL/servers/databases/elasticPoolName | |
| Microsoft.SQL/servers/elasticPools/dtu | |
| Microsoft.SQL/servers/elasticPools/edition | |


## 效果
策略支持三种类型的效果 - **deny**、**audit** 和 **append**。

* Deny 将在审核日志中生成一个事件，并使请求失败
* Audit 将在审核日志中生成一个事件，但不会使请求失败
* Append 会将定义的字段集添加到请求

对于 **append**，必须提供以下详细信息：

    "effect": "append",
    "details": [
      {
        "field": "field name",
        "value": "value of the field"
      }
    ]

值可以是字符串或 JSON 格式对象。


## 策略定义示例
现在，让我们看看如何定义策略以实现前述方案。

### 费用分摊：要求提供部门标记
以下策略拒绝其标记不包含“costCenter”键的请求。

    {
      "if": {
        "not" : {
          "field" : "tags",
          "containsKey" : "costCenter"
        }
      },
      "then" : {
        "effect" : "deny"
      }
    }

不存在任何标记时，以下策略会附加使用预定义值的 costCenter 标记。

    {
      "if": {
        "field": "tags",
        "exists": "false"
      },
      "then": {
        "effect": "append",
        "details": [
          {
            "field": "tags",
            "value": {"costCenter":"myDepartment" }
          }
        ]
      }
    }

不存在 costCenter 标记但存在其他标记时，以下策略会附加使用预定义值的 costCenter 标记。

    {
      "if": {
        "allOf": [
          {
            "field": "tags",
            "exists": "true"
          },
          {
            "field": "tags.costCenter",
            "exists": "false"
          }
        ]

      },
      "then": {
        "effect": "append",
        "details": [
          {
            "field": "tags.costCenter",
            "value": "myDepartment"
          }
        ]
      }
    }


### 遵循地区：确保资源位置
以下示例中的策略拒绝位置不是中国北部或西欧的请求。

    {
      "if" : {
        "not" : {
          "field" : "location",
          "in" : ["northeurope" , "westeurope"]
        }
      },
      "then" : {
        "effect" : "deny"
      }
    }

### 服务策展：选择服务目录
以下示例显示的策略只允许对 Microsoft.Resources/\*、Microsoft.Compute/\*、Microsoft.Storage/\*、Microsoft.Network/\* 类型的服务执行操作。其他类型的服务都会被拒绝。

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

### 使用批准的 SKU
以下示例演示如何使用属性别名限制 SKU。在示例中，只有 Standard\_LRS 和 Standard\_GRS 被批准用于存储帐户。

    {
      "if": {
        "allOf": [
          {
            "field": "type",
            "equals": "Microsoft.Storage/storageAccounts"
          },
          {
            "not": {
              "allof": [
                {
                  "field": "Microsoft.Storage/storageAccounts/sku.name",
                  "in": ["Standard_LRS", "Standard_GRS"]
                }
              ]
            }
          }
        ]
      },
      "then": {
        "effect": "deny"
      }
    }


### 命名约定
以下示例演示如何使用“like”条件支持的通配符。该条件指明，如果名称符合所述模式 (namePrefix*nameSuffix)，则拒绝请求。

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

### 只要求对存储资源使用标记
以下示例演示在只要求对存储资源使用应用程序标记的情况下，如何嵌套逻辑运算符。

    {
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
        "then": {
            "effect": "audit"
        }
    }

## <a name="create-and-assign-a-policy"></a> 创建并分配策略
应用策略时，需要先创建策略定义，然后将其应用到某个范围。

### REST API
你可以使用[用于策略定义的 REST API](https://docs.microsoft.com/rest/api/resources/policydefinitions) 来创建策略。REST API 可让你创建和删除策略定义，以及获取现有定义的信息。

若要创建策略，请运行：

    PUT https://management.chinacloudapi.cn/subscriptions/{subscription-id}/providers/Microsoft.authorization/policydefinitions/{policyDefinitionName}?api-version={api-version}

对于 api-version，请使用 *2016-04-01* 或 *2016-12-01* 。包括类似于以下示例的请求正文：

    {
      "properties": {
        "parameters": {
          "listOfAllowedLocations": {
            "type": "array",
            "metadata": {
              "description": "An array of permitted locations for resources.",
              "strongType": "location",
              "displayName": "List Of Locations"
            }
          }
        },
        "displayName": "Geo-compliance policy template",
        "description": "This policy enables you to restrict the locations your organization can specify when deploying resources. Use to enforce your geo-compliance requirements.",
        "policyRule": {
          "if": {
            "not": {
              "field": "location",
              "in": "[parameters('listOfAllowedLocations')]"
            }
          },
          "then": {
            "effect": "deny"
          }
        }
      }
    }

可以通过[用于策略分配的 REST API](https://docs.microsoft.com/rest/api/resources/policyassignments)，在所需范围内应用策略定义。REST API 可让你创建和删除策略分配，以及获取现有分配的信息。

若要创建策略分配，请运行：

    PUT https://management.chinacloudapi.cn /subscriptions/{subscription-id}/providers/Microsoft.authorization/policyassignments/{policyAssignmentName}?api-version={api-version}

{policy-assignment} 是策略分配的名称。对于 api-version，请使用 *2016-04-01* 或 *2016-12-01* （用于参数）。

使用类似于以下示例的请求正文：

    {
      "properties":{
        "displayName":"China North only policy assignment on the subscription ",
        "description":"Resources can only be provisioned in China North regions",
        "parameters": {
             "listOfAllowedLocations": { "value": ["China North", "China North 2"] }
         },
        "policyDefinitionId":"/subscriptions/########/providers/Microsoft.Authorization/policyDefinitions/testdefinition",
        "scope":"/subscriptions/########-####-####-####-############"
      },
    }

### PowerShell
可以使用 New-AzureRmPolicyDefinition cmdlet 创建策略定义。以下示例创建只允许使用中国北部和西欧资源的策略。

    $policy = New-AzureRmPolicyDefinition -Name regionPolicyDefinition -Description "Policy to allow resource creation only in certain regions" -Policy '{    
      "if" : {
        "not" : {
          "field" : "location",
          "in" : ["northeurope" , "westeurope"]
        }
      },
      "then" : {
        "effect" : "deny"
      }
    }'            

执行输出将存储在 $policy 对象中，稍后可在分配策略期间使用。对于策略参数，也可以提供包含策略的 .json 文件的路径，而不是指定内联策略。

    New-AzureRmPolicyDefinition -Name regionPolicyDefinition -Description "Policy to allow resource creation only in certain     regions" -Policy "path-to-policy-json-on-disk"

可以使用 New-AzureRmPolicyAssignment cmdlet 将策略应用到所需范围：

    New-AzureRmPolicyAssignment -Name regionPolicyAssignment -PolicyDefinition $policy -Scope    /subscriptions/########-####-####-####-############/resourceGroups/<resource-group-name>

此处的 $policy 是执行 New-AzureRmPolicyDefinition cmdlet 后返回的策略对象。此处的范围是指定的资源组的名称。

若要删除策略分配，请使用：

    Remove-AzureRmPolicyAssignment -Name regionPolicyAssignment -Scope /subscriptions/########-####-####-####-############/resourceGroups/<resource-group-name>

可以分别通过 Get-AzureRmPolicyDefinition、Set-AzureRmPolicyDefinition 和 Remove-AzureRmPolicyDefinition cmdlet 获取、更改或删除策略定义。

同样地，可以分别通过 Get-AzureRmPolicyAssignment、Set-AzureRmPolicyAssignment 和 Remove-AzureRmPolicyAssignment cmdlet 获取、更改或删除策略分配。

### Azure CLI
可以将 Azure CLI 与策略定义命令配合使用，创建策略定义。以下示例创建只允许使用中国北部和西欧资源的策略。

    azure policy definition create --name regionPolicyDefinition --description "Policy to allow resource creation only in certain regions" --policy-string '{    
      "if" : {
        "not" : {
          "field" : "location",
          "in" : ["northeurope" , "westeurope"]
        }
      },
      "then" : {
        "effect" : "deny"
      }
    }'    


可以指定包含策略的 .json 文件的路径，不必指定内联策略。

    azure policy definition create --name regionPolicyDefinition --description "Policy to allow resource creation only in certain regions" --policy "path-to-policy-json-on-disk"

可以使用策略分配命令将策略应用到所需范围：

    azure policy assignment create --name regionPolicyAssignment --policy-definition-id /subscriptions/########-####-####-####-############/providers/Microsoft.Authorization/policyDefinitions/<policy-name> --scope    /subscriptions/########-####-####-####-############/resourceGroups/<resource-group-name>

此处的范围是指定的资源组的名称。如果参数 policy-definition-id 的值未知，可通过 Azure CLI 获取。

    azure policy definition show <policy-name>

若要删除策略分配，请使用：

    azure policy assignment delete --name regionPolicyAssignment --scope /subscriptions/########-####-####-####-############/resourceGroups/<resource-group-name>

可以分别通过策略定义的 show、set 和 delete 命令获取、更改或删除策略定义。

类似地，可以分别通过策略分配的 show 和 delete 命令获取、更改或删除策略分配。

## 策略审核事件
在应用策略之后，即可看到与策略相关的事件。可以通过门户，PowerShell 或 Azure CLI 获取此数据。

### PowerShell
若要查看与拒绝效果相关的所有事件，可以使用以下 PowerShell 命令：

    Get-AzureRmLog | where {$_.OperationName -eq "Microsoft.Authorization/policies/deny/action"} 

若要查看与审核效果相关的所有事件，可以使用以下命令：

    Get-AzureRmLog | where {$_.OperationName -eq "Microsoft.Authorization/policies/audit/action"} 

### Azure CLI
若要查看资源组中与拒绝效果相关的所有事件，可以使用以下 CLI 命令：

    azure group log show ExampleGroup --json | jq ".[] | select(.operationName.value == "Microsoft.Authorization/policies/deny/action")"

若要查看与审核效果相关的所有事件，可以使用以下 CLI 命令：

    azure group log show ExampleGroup --json | jq ".[] | select(.operationName.value == "Microsoft.Authorization/policies/audit/action")"

## 查看策略
可以使用 PowerShell、Azure CLI 或 REST API 查看策略。可能需要在部署失败后查看策略，并需查看拒绝了部署的规则。错误消息包含策略定义的 ID。

### PowerShell
若要获取策略，请使用以下 cmdlet：

    (Get-AzureRmPolicyAssignment -Id "/subscriptions/{guid}/providers/Microsoft.Authorization/policyDefinitions/{definition-name}").Properties.policyRule | ConvertTo-Json

该 cmdlet 返回策略定义的 JSON。

### Azure CLI
若要获取策略，请使用以下命令：

    azure policy definition show {definition-name} --json

### REST API
若要获取策略，请使用[获取策略定义](https://docs.microsoft.com/rest/api/resources/policydefinitions#PolicyDefinitions_Get)操作。

## 后续步骤
* 如需了解企业如何使用 Resource Manager 对订阅进行有效管理，请参阅 [Azure 企业机架 - 规范性订阅管理](/documentation/articles/resource-manager-subscription-governance/)。

<!---HONumber=Mooncake_0103_2017-->