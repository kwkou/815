<properties
    pageTitle="存储帐户的 Azure 资源策略 | Azure"
    description="介绍用于管理存储帐户的部署的 Azure Resource Manager 策略。"
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


# 将资源策略应用于存储帐户
本主题显示了可以应用于 Azure 存储帐户的几个[资源策略](/documentation/articles/resource-manager-policy/)。这些策略可确保组织中部署的存储帐户的一致性。

## 定义允许的存储帐户类型

以下策略限制可以部署哪些[存储帐户类型](/documentation/articles/storage-redundancy/)：

    {
      "if": {
        "allOf": [
          {
            "field": "type",
            "equals": "Microsoft.Storage/storageAccounts"
          },
          {
            "not": {
              "field": "Microsoft.Storage/storageAccounts/sku.name",
              "in": [
                "Standard_LRS",
                "Standard_GRS"
              ]
            }
          }
        ]
      },
      "then": {
        "effect": "deny"
      }
    }

具有用于接受允许的 SKU 的参数的类似策略规则以内置策略定义的形式提供。内置策略具有资源 ID `/providers/Microsoft.Authorization/policyDefinitions/7433c107-6db4-4ad1-b57a-a76dce0154a1`。

## 定义允许的访问层

以下策略指定可以为存储帐户指定的[访问层](/documentation/articles/storage-blob-storage-tiers/)类型：

    {
      "if": {
        "allOf": [
          {
            "field": "type",
            "equals": "Microsoft.Storage/storageAccounts"
          },
          {
            "field": "kind",
            "equals": "BlobStorage"
          },
          {
            "not": {
              "field": "Microsoft.Storage/storageAccounts/accessTier",
              "equals": "cool"
            }
          }
        ]
      },
      "then": {
        "effect": "deny"
      }
    }

## 确保已启用加密

以下策略要求所有存储帐户都启用[存储服务加密](/documentation/articles/storage-service-encryption/)：

    {
      "if": {
        "allOf": [
          {
            "field": "type",
            "equals": "Microsoft.Storage/storageAccounts"
          },
          {
            "not": {
              "field": "Microsoft.Storage/storageAccounts/enableBlobEncryption",
              "equals": "true"
            }
          }
        ]
      },
      "then": {
        "effect": "deny"
      }
    }

此策略规则也以资源 ID 为 `/providers/Microsoft.Authorization/policyDefinitions/7c5a74bf-ae94-4a74-8fcf-644d1e0e6e6f` 的内置策略定义的形式提供。

## 后续步骤
* 定义策略规则（如前面的示例中所示）之后，需要创建策略定义并将其分配给作用域。作用域可以是订阅、资源组或资源。有关创建和分配策略的示例，请参阅[分配和管理策略](/documentation/articles/resource-manager-policy-create-assign/)。
* 如需了解企业如何使用 Resource Manager 对订阅进行有效管理，请参阅 [Azure 企业机架 - 规范性订阅管理](/documentation/articles/resource-manager-subscription-governance/)。

<!---HONumber=Mooncake_0227_2017-->
<!--Update_Description: new article about the strategy policy on managing the storage account in azure resource manager-->