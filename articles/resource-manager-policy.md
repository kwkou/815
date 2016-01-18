<properties
	pageTitle="Azure 资源管理器策略 | Windows Azure"
	description="介绍如何使用 Azure 资源管理器策略来防止订阅、资源组或单个资源等不同的范围发生冲突。"
	services="azure-resource-manager"
	documentationCenter="na"
	authors="ravbhatnagar"
	manager="ryjones"
	editor=""/>

<tags
	ms.service="azure-resource-manager"
	ms.date="12/18/2015"
	wacn.date="01/14/2016"/>

# 使用策略来管理资源和控制访问

Azure 资源管理器现在可让你通过自定义策略来控制访问。使用策略可以防止组织中的用户违反管理组织资源所需的惯例。

你可以创建策略定义来描述会明确遭到拒绝的操作或资源。你可以在所需范围（例如订阅、资源组或是单个资源）分配这些策略定义。

在本文中，我们将说明可用于创建策略的策略定义语言的基本结构。然后说明如何在不同范围应用这些策略，最后演示有关如何通过 REST API 实现此目的的一些示例。

策略当前以预览版提供。

## 策略与 RBAC 有什么不同？

策略与基于角色的访问控制之间有几个重要的差别，但首先你必须了解策略是和 RBAC 一起工作的。若要使用策略，用户必须通过 RBAC 完成身份验证。不同于 RBAC，策略是默认的允许和明确拒绝系统。

RBAC 着重于**用户**在不同的范围可执行的操作。例如，将特定用户添加到所需范围的资源组的参与者角色后，该用户便可对该资源组做出更改。

策略着重于各种范围的**资源**操作。例如，通过策略，你可以控制可预配的资源类型，或限制可以预配资源的位置。

## 常见方案

一个常见方案是为了费用分摊而要求提供部门标记。组织可能只想在有关联的适当成本中心时允许操作，否则将拒绝请求。这将帮助它们向适当的成本中心对所执行的操作收费。

另一个常见方案是组织可能想要控制创建资源的位置。或者它们可能想要通过仅允许预配特定类型的资源，来控制对资源的访问。

同样地，组织可以控制服务类别或为资源强制运行所需的命名约定。

使用策略可轻松实现这些方案，如下所述。

## 策略定义结构

策略定义是使用 JSON 创建的。它包含定义操作和效果的一个或多个条件/逻辑运算符，告诉你满足条件时产生的效果。

简单而言，策略包含以下各项：

**条件/逻辑运算符：**包含可通过一组逻辑运算符操作的条件集。

**效果：**描述当满足条件时产生的效果 – 拒绝或审核。审核效果将发出警告事件服务日志。例如，管理员可以创建策略，如果有任何人创建大型 VM，则此策略会引发审核，然后管理员可以审查日志。

    {
      "if" : {
        <condition> | <logical operator>
      },
      "then" : {
        "effect" : "deny | audit"
      }
    }


## 逻辑运算符

下面列出了支持的逻辑运算符和语法：

| 运算符名称 | 语法 |
| :------------- | :------------- |
| Not | "not" : {&lt;condition or operator &gt;} |
| And | "allOf" : [ {&lt;condition or operator &gt;},{&lt;condition or operator &gt;}] |
| 或 | "anyOf" : [ {&lt;condition or operator &gt;},{&lt;condition or operator &gt;}] |

资源管理器可让你通过嵌套的运算符在策略中指定复杂逻辑。例如，你可以拒绝在指定资源类型的特定位置创建资源。下面显示了嵌套运算符的示例。

## 条件

条件评估**字段**或**源**是否符合特定的准则。下面列出了支持的条件名称和语法：

| 条件名称 | 语法 |
| :------------- | :------------- |
| 等于 | "equals" : "&lt;value&gt;" |
| Like | "like" : "&lt;value&gt;" |
| Contains | "contains" : "&lt;value&gt;"|
| In | "in" : [ "&lt;value1&gt;","&lt;value2&gt;" ]|
| ContainsKey | "containsKey" : "&lt;keyName&gt;" |


## 字段和源

条件是使用字段和源构成的。字段表示资源请求负载中的属性。源表示请求本身的特征。

支持以下字段和源：

字段：**name**、**kind**、**type**、**location**、**tags**、**tags.***。

源：**action**


## 策略定义示例

现在让我们看一下如何定义策略，以实现以上所列的方案。

### 费用分摊：要求提供部门标记

以下策略将拒绝不包含“costCenter”键标记的所有请求。

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


### 地区法规遵循：确保资源位置

以下示例显示的策略将拒绝位置不是欧洲北部或欧洲西部的所有请求。

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

以下示例演示如何使用源。它显示只允许对 Microsoft.Resources/、Microsoft.Compute/*、Microsoft.Storage/*、Microsoft.Network/ 类型的服务执行的操作。其他任何请求将被拒绝。

    {
      "if" : {
        "not" : {
          "anyOf" : [
            {
              "source" : "action",
              "like" : "Microsoft.Resources/*"
            },
            {
              "source" : "action",
              "like" : "Microsoft.Compute/*"
            },
            {
              "source" : "action",
              "like" : "Microsoft.Storage/*"
            },
            {
              "source" : "action",
              "like" : "Microsoft.Network/*"
            }
          ]
        }
      },
      "then" : {
        "effect" : "deny"
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
    
### 只要求存储资源的标记

以下示例演示如何嵌套逻辑运算符，以便要求只对存储资源使用应用程序标记。

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
                "source": "action",
                "like": "Microsoft.Storage/*"
              }
            ]
        },
        "then": {
            "effect": "audit"
        }
    }

## 策略分配

策略可在不同的范围应用，如订阅、资源组和单个资源。策略由所有子资源继承。因此，如果将策略应用到资源组，则会将它应用到该资源组中的所有资源。

## 创建策略

本部分提供有关如何使用 REST API 创建策略的详细信息。

### 使用 REST API 创建策略定义

你可以使用[用于策略定义的 REST API](https://msdn.microsoft.com/zh-cn/library/azure/mt588471.aspx) 来创建策略。REST API 可让你创建和删除策略定义，以及获取现有定义的信息。

若要创建新策略，请运行：

    PUT https://manage.windowsazure.cn/subscriptions/{subscription-id}/providers/Microsoft.authorization/policydefinitions/{policyDefinitionName}?api-version={api-version}

使用类似于下面的请求正文：

    {
      "properties":{
        "policyType":"Custom",
        "description":"Test Policy",
        "policyRule":{
          "if" : {
            "not" : {
              "field" : "tags",
              "containsKey" : "costCenter"
            }
          },
          "then" : {
            "effect" : "deny"
          }
        }
      },
      "id":"/subscriptions/########-####-####-####-############/providers/Microsoft.Authorization/policyDefinitions/testdefinition",
      "type":"Microsoft.Authorization/policyDefinitions",
      "name":"testdefinition"
    }


策略定义可以定义为如上所示的示例之一。对于 api-version，请使用 *2015-10-01-preview*。有关示例与其他详细信息，请参阅[用于策略定义的 REST API](https://msdn.microsoft.com/zh-cn/library/azure/mt588471.aspx)。

### 使用 PowerShell 创建策略定义

可以使用 New-AzureRmPolicyDefinition cmdlet 创建新的策略定义，如下所示。以下示例将创建一个策略，只允许欧洲北部和欧洲西部的资源。

    $policy = New-AzureRmPolicyDefinition -Name regionPolicyDefinition -Description "Policy to allow resource creation onlyin certain regions" -Policy '{	"if" : {
    	    			    "not" : {
    	      			    	"field" : "location",
    	      			    		"in" : ["northeurope" , "westeurope"]
    	    			    	}
    	    		          },
    	      		    		"then" : {
    	    			    		"effect" : "deny"
    	      			    		}
    	    		    	}'    		

执行输出将存储在 $policy 对象中，以便稍后可在分配策略期间使用。对于策略参数，也可以提供包含策略的 .json 文件的路径，而不是指定内联策略，如下所示。

    New-AzureRmPolicyDefinition -Name regionPolicyDefinition -Description "Policy to allow resource creation only in certain 	regions" -Policy "path-to-policy-json-on-disk"


## 应用策略

### 使用 REST API 分配策略

可以通过[用于策略分配的 REST API](https://msdn.microsoft.com/zh-cn/library/azure/mt588466.aspx)，在所需范围内应用策略定义。REST API 可让你创建和删除策略分配，以及获取现有分配的信息。

若要创建新的策略分配，请运行：

    PUT https://management.azure.com /subscriptions/{subscription-id}/providers/Microsoft.authorization/policyassignments/{policyAssignmentName}?api-version={api-version}

{policy-assignment} 是策略分配的名称。对于 api-version，请使用 *2015-10-01-preview*。

使用类似于下面的请求正文：

    {
      "properties":{
        "displayName":"VM_Policy_Assignment",
        "policyDefinitionId":"/subscriptions/########/providers/Microsoft.Authorization/policyDefinitions/testdefinition",
        "scope":"/subscriptions/########-####-####-####-############"
      },
      "id":"/subscriptions/########-####-####-####-############/providers/Microsoft.Authorization/policyAssignments/VMPolicyAssignment",
      "type":"Microsoft.Authorization/policyAssignments",
      "name":"VMPolicyAssignment"
    }

有关示例与其他详细信息，请参阅[用于策略分配的 REST API](https://msdn.microsoft.com/zh-cn/library/azure/mt588466.aspx)。

### 使用 PowerShell 分配策略

你可以使用 New-AzureRmPolicyAssignment cmdlet 将上面通过 PowerShell 创建的策略应用到所需的范围，如下所示：

    New-AzureRmPolicyAssignment -Name regionPolicyAssignment -PolicyDefinition $policy -Scope    /subscriptions/########-####-####-####-############/resourceGroups/<resource-group-name>
        
此处的 $policy 是执行 New-AzureRmPolicyDefinition cmdlet 后返回的策略对象，如下所示。此处的范围是指定的资源组的名称。

如果你要删除上述策略分配，可执行以下操作：

    Remove-AzureRmPolicyAssignment -Name regionPolicyAssignment -Scope /subscriptions/########-####-####-####-############/resourceGroups/<resource-group-name>

可以分别通过 Get-AzureRmPolicyDefinition、Set-AzureRmPolicyDefinition 和 Remove-AzureRmPolicyDefinition cmdlet 来获取、更改或删除策略定义。

同样地，可以分别通过 Get-AzureRmPolicyAssignment、Set-AzureRmPolicyAssignment 和 Remove-AzureRmPolicyAssignment 来获取、更改或删除策略分配。

##策略审核事件

在应用策略之后，即可看到与策略相关的事件。你可以转到门户，或使用 PowerShell 来获取此数据。

若要查看拒绝效果相关的所有事件，可以使用以下命令。

    Get-AzureRmLog | where {$_.subStatus -eq "Forbidden"}     

若要查看审核效果相关的所有事件，可以使用以下命令。

    Get-AzureRmLog | where {$_.OperationName -eq "Microsoft.Authorization/policies/audit/action"} 
    

<!---HONumber=Mooncake_0104_2016-->