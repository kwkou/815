<properties
    pageTitle="标记的 Azure 资源策略 | Azure"
    description="提供用于管理资源的标记的资源策略示例"
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
    ms.date="02/09/2017"
    wacn.date="03/03/2017"
    ms.author="tomfitz" />  


# 对标记应用资源策略

本主题提供可应用以确保对资源一致使用标记的常见策略规则。

将标记策略应用于具有现有资源的资源组或订阅不会追溯性地将策略应用于这些资源。若要强制对这些资源实施策略，请触发对现有资源的更新，如[触发对现有资源的更新](#trigger-updates-to-existing-resources)中所示。

## 确保资源组中的所有资源都具有一个标记/值

一个常见要求是资源组中的所有资源都具有一个特定标记和值。要按部门跟踪成本，通常需要此要求。必须满足以下条件：

* 所需的标记和值将追加到新资源和没有任何现有标记的已更新资源。
* 所需的标记和值将追加到新资源和具有其他标记（但不是所需的标记和值）的已更新资源。
* 无法从任何现有的资源中删除所需的标记和值。

可以通过将以下三个策略应用于资源组来实现此要求：

* [追加标记](#append-tag)
* [在标记后面追加其他标记](#append-tag-with-other-tags)
* [需要标记和值](#require-tag-and-value)

<a id="append-tag"></a>
### 追加标记

不存在任何标记时，以下策略规则会使用预定义值追加 costCenter 标记：

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

<a id="append-tag-with-other-tags"></a>
### 在标记后面追加其他标记

存在标记但未定义 costCenter 标记时，以下策略规则会使用预定义值追加 costCenter 标记：

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

<a id="require-tag-and-value"></a>
### 需要标记和值

以下策略规则拒绝更新或创建不具有分配给预定义值的 costCenter 标记的资源。

    {
      "if": {
        "not": {
          "field": "tags.costCenter",
          "equals": "myDepartment"
        }
      },
      "then": {
        "effect": "deny"
      }
    }

## 要求对某种资源类型使用标记
以下示例演示如何嵌套逻辑运算符以仅要求对指定的资源类型（在本例中为存储帐户）使用应用程序标记。

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

## 需要标记
以下策略拒绝其标记不包含“costCenter”键（可以应用任何值）的请求：

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

## <a id="trigger-updates-to-existing-resources"></a> 触发对现有资源的更新

以下 PowerShell 脚本触发对现有资源的更新以强制实施已添加的标记策略。

    $group = Get-AzureRmResourceGroup -Name "ExampleGroup" 

    $resources = Find-AzureRmResource -ResourceGroupName $group.ResourceGroupName 

    foreach($r in $resources)
    {
        try{
            $r | Set-AzureRmResource -Tags ($a=if($_.Tags -eq $NULL) { @{}} else {$_.Tags}) -Force -UsePatchSemantics
        }
        catch{
            Write-Host  $r.ResourceId + "can't be updated"
        }
    }

## 后续步骤
* 定义策略规则（如前面的示例中所示）之后，需要创建策略定义并将其分配给作用域。作用域可以是订阅、资源组或资源。有关创建和分配策略的示例，请参阅[分配和管理策略](/documentation/articles/resource-manager-policy-create-assign/)。
* 有关资源策略的简介，请参阅[资源策略概述](/documentation/articles/resource-manager-policy/)。
* 如需了解企业如何使用 Resource Manager 对订阅进行有效管理，请参阅 [Azure 企业机架 - 规范性订阅管理](/documentation/articles/resource-manager-subscription-governance/)。

<!---HONumber=Mooncake_0227_2017-->
<!--Update_Description: New article about the resource policy on azure Tag-->