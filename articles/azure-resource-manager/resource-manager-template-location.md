<properties
    pageTitle="模板中的 Azure 资源位置 | Azure"
    description="演示如何在 Azure Resource Manager 模板中设置资源位置"
    services="azure-resource-manager"
    documentationcenter=""
    author="tfitzmac"
    manager="timlt"
    editor="tysonn" />
<tags
    ms.assetid=""
    ms.service="azure-resource-manager"
    ms.workload="na"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="02/03/2017"
    wacn.date="03/03/2017"
    ms.author="tomfitz" />  


# 在 Azure Resource Manager 模板中设置资源位置
部署模板时，必须提供每个资源的位置。本主题说明如何确定可用于每种资源类型的订阅的位置。

## 确定支持的位置

<!-- regions/services not supported on Azure.cn-->
<!--有关每种资源类型支持的位置的完整列表，请参阅[可用产品（按区域）](https://azure.microsoft.com/regions/services/)。但是，你的订阅可能无法访问该列表中的所有位置。-->
若要查看可用于订阅的位置的自定义列表，请使用 Azure PowerShell 或 Azure CLI。

以下示例使用 PowerShell 获取 `Microsoft.Web\sites` 资源类型的位置：

    ((Get-AzureRmResourceProvider -ProviderNamespace Microsoft.Web).ResourceTypes | Where-Object ResourceTypeName -eq sites).Locations

以下示例使用 Azure CLI 2.0（预览版）获取 `Microsoft.Web\sites` 资源类型的位置：

    az provider show -n Microsoft.Web --query "resourceTypes[?resourceType=='sites'].locations"

## 在模板中设置位置

确定资源支持的位置后，需要在模板中设置该位置。设置此值的最简单方法是在支持资源类型的位置中创建资源组，并将每个位置设置为 `[resourceGroup().location]`。可以将模板重新部署到不同位置中的资源组，而不更改模板或参数中的任何值。

以下示例显示了部署到资源组所在位置的存储帐户：

    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "variables": {
          "storageName": "[concat('storage', uniqueString(resourceGroup().id))]"
        },
        "resources": [
        {
          "apiVersion": "2016-01-01",
          "type": "Microsoft.Storage/storageAccounts",
          "name": "[variables('storageName')]",
          "location": "[resourceGroup().location]",
          "tags": {
            "Dept": "Finance",
            "Environment": "Production"
          },
          "sku": {
            "name": "Standard_LRS"
          },
          "kind": "Storage",
          "properties": { }
        }
        ]
    }

如果需要在模板中对位置进行硬编码，请提供其中一个支持的区域名称。以下示例显示了始终部署到中国北部的存储帐户：

    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "resources": [
        {
          "apiVersion": "2016-01-01",
          "type": "Microsoft.Storage/storageAccounts",
          "name": "[concat('storageloc', uniqueString(resourceGroup().id))]",
          "location": "China North",
          "tags": {
            "Dept": "Finance",
            "Environment": "Production"
          },
          "sku": {
            "name": "Standard_LRS"
          },
          "kind": "Storage",
          "properties": { }
        }
        ]
    }

## 后续步骤
* 有关如何创建模板的建议，请参阅 [Best practices for creating Azure Resource Manager templates](/documentation/articles/resource-manager-template-best-practices/)（创建 Azure Resource Manager 模板的最佳做法）。

<!---HONumber=Mooncake_0227_2017-->
<!-- Update_Description: new articles about how to set the location in the template of azure resource manager-->