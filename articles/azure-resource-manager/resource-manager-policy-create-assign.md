<properties
    pageTitle="分配和管理 Azure 资源策略 | Azure"
    description="介绍如何将 Azure 资源策略应用于订阅和资源组，以及如何查看资源策略。"
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
    ms.date="02/10/2017"
    wacn.date="03/03/2017"
    ms.author="tomfitz" />  


# 分配和管理资源策略

若要实施策略，必须执行三个步骤：

1. 使用 JSON 定义策略规则。
2. 基于在上一步中创建的 JSON，在订阅中创建策略定义。此步骤使策略可用于分配，但不会将规则应用于订阅。
3. 将策略分配给作用域（例如，订阅或资源组）。现在将强制执行策略的规则。

Azure 提供了一些预定义的策略，可减少用户必须定义的策略数。如果预定义的策略适用于你的方案，请跳过前两个步骤，并将预定义的策略分配给作用域。

本主题重点介绍有关创建策略定义并将该定义分配给作用域的步骤。它不侧重于用于创建策略定义的语法。有关策略语法的信息，请参阅[资源策略概述](/documentation/articles/resource-manager-policy/)。

## REST API

### 创建策略定义

你可以使用[用于策略定义的 REST API](https://docs.microsoft.com/rest/api/resources/policydefinitions) 来创建策略。REST API 可让你创建和删除策略定义，以及获取现有定义的信息。

若要创建策略定义，请运行：

    PUT https://management.chinacloudapi.cn/subscriptions/{subscription-id}/providers/Microsoft.authorization/policydefinitions/{policyDefinitionName}?api-version={api-version}

包括类似于以下示例的请求正文：

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

### 分配策略

可以通过[用于策略分配的 REST API](https://docs.microsoft.com/rest/api/resources/policyassignments)，在所需范围内应用策略定义。REST API 可让你创建和删除策略分配，以及获取现有分配的信息。

若要创建策略分配，请运行：

    PUT https://management.chinacloudapi.cn/subscriptions/{subscription-id}/providers/Microsoft.authorization/policyassignments/{policyAssignmentName}?api-version={api-version}

{policy-assignment} 是策略分配的名称。

包括类似于以下示例的请求正文：

    {
      "properties":{
        "displayName":"China North only policy assignment on the subscription ",
        "description":"Resources can only be provisioned in China North regions",
        "parameters": {
          "listOfAllowedLocations": { "value": ["China North", "China East"] }
         },
        "policyDefinitionId":"/subscriptions/{subscription-id}/providers/Microsoft.Authorization/policyDefinitions/{definition-name}",
          "scope":"/subscriptions/{subscription-id}"
      },
    }

### 查看策略
若要获取策略，请使用[获取策略定义](https://docs.microsoft.com/rest/api/resources/policydefinitions#PolicyDefinitions_Get)操作。

### 获取别名
可以通过 REST API 检索别名：

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

## PowerShell

### 创建策略定义
可以使用 `New-AzureRmPolicyDefinition` cmdlet 创建策略定义。以下示例创建只允许使用中国北部资源的策略定义。

    $policy = New-AzureRmPolicyDefinition -Name regionPolicyDefinition -Description "Policy to allow resource creation only in certain regions" -Policy '{    
      "if" : {
        "not" : {
          "field" : "location",
          "in" : ["chinaeast" , "chinanorth"]
        }
      },
      "then" : {
        "effect" : "deny"
      }
    }'

输出存储在 `$policy` 对象中，将在策略分配过程中使用该对象。

不是以参数形式指定 JSON，而可以提供包含策略规则的 .json 文件的路径。

    $policy = New-AzureRmPolicyDefinition -Name regionPolicyDefinition -Description "Policy to allow resource creation only in certain regions" -Policy "c:\policies\storageskupolicy.json"

### 分配策略

使用 `New-AzureRmPolicyAssignment` cmdlet 在所需的作用域内应用策略：

    New-AzureRmPolicyAssignment -Name regionPolicyAssignment -PolicyDefinition $policy -Scope /subscriptions/{subscription-id}/resourceGroups/{resource-group-name}

### 查看策略分配

若要获取策略，请使用以下 cmdlet：

    (Get-AzureRmPolicyAssignment -Id "/subscriptions/{guid}/providers/Microsoft.Authorization/policyDefinitions/{definition-name}").Properties.policyRule | ConvertTo-Json

该 cmdlet 返回策略定义的 JSON。

### 删除策略分配 

若要删除策略分配，请使用：

    Remove-AzureRmPolicyAssignment -Name regionPolicyAssignment -Scope /subscriptions/{subscription-id}/resourceGroups/{resource-group-name}

## Azure CLI 2.0（预览版）

### 创建策略定义

可以将 Azure CLI 2.0（预览版）与策略定义命令配合使用，创建策略定义。以下示例创建只允许使用中国北部资源的策略。

    az policy definition create --name regionPolicyDefinition --description "Policy to allow resource creation only in certain regions" --rules '{    
      "if" : {
        "not" : {
          "field" : "location",
          "in" : ["chinaeast" , "chinanorth"]
        }
      },
      "then" : {
        "effect" : "deny"
      }
    }'    

### 分配策略

可以使用策略分配命令将策略应用到所需范围：

    az policy assignment create --name regionPolicyAssignment --policy regionPolicyDefinition --scope /subscriptions/{subscription-id}/resourceGroups/{resource-group-name}

### 查看策略定义
若要获取策略定义，请使用以下命令：

    az policy definition show --name regionPolicyAssignment

### 删除策略分配 

若要删除策略分配，请使用：

    az policy assignment delete --name regionPolicyAssignment --scope /subscriptions/{subscription-id}/resourceGroups/{resource-group-name}

## Azure CLI 1.0

### 创建策略定义

可以将 Azure CLI 与策略定义命令配合使用，创建策略定义。以下示例创建只允许使用中国北部资源的策略。

    azure policy definition create --name regionPolicyDefinition --description "Policy to allow resource creation only in certain regions" --policy-string '{    
      "if" : {
        "not" : {
          "field" : "location",
          "in" : ["chinaeast" , "chinanorth"]
        }
      },
      "then" : {
        "effect" : "deny"
      }
    }'    

可以指定包含策略的 .json 文件的路径，不必指定内联策略。

    azure policy definition create --name regionPolicyDefinition --description "Policy to allow resource creation only in certain regions" --policy "path-to-policy-json-on-disk"

### 分配策略

可以使用策略分配命令将策略应用到所需范围：

    azure policy assignment create --name regionPolicyAssignment --policy-definition-id /subscriptions/{subscription-id}/providers/Microsoft.Authorization/policyDefinitions/{policy-name} --scope    /subscriptions/{subscription-id}/resourceGroups/{resource-group-name}

此处的范围是指定的资源组的名称。如果参数 policy-definition-id 的值未知，可通过 Azure CLI 获取。

    azure policy definition show {policy-name}

### 查看策略
若要获取策略，请使用以下命令：

    azure policy definition show {definition-name} --json

### 删除策略分配 

若要删除策略分配，请使用：

    azure policy assignment delete --name regionPolicyAssignment --scope /subscriptions/{subscription-id}/resourceGroups/{resource-group-name}

## 后续步骤
* 如需了解企业如何使用 Resource Manager 对订阅进行有效管理，请参阅 [Azure 企业机架 - 规范性订阅管理](/documentation/articles/resource-manager-subscription-governance/)。

<!---HONumber=Mooncake_0227_2017-->
<!--Update_Description: New article about the policy on distributing and managing azure resource-->