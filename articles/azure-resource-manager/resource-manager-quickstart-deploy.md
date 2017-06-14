<properties
    pageTitle="将资源部署到 Azure | Azure"
    description="使用 Azure PowerShell 或 Azure CLI 将资源部署到 Azure。 资源在 Resource Manager 模板中定义。"
    services="azure-resource-manager"
    documentationcenter="na"
    author="tfitzmac"
    manager="timlt"
    editor="tysonn" />
<tags
    ms.assetid=""
    ms.service="azure-resource-manager"
    ms.devlang="na"
    ms.topic="get-started-article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="02/16/2017"
    wacn.date="06/05/2017"
    ms.author="v-yeche"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="08618ee31568db24eba7a7d9a5fc3b079cf34577"
    ms.openlocfilehash="3f2165d3bc88deb64de662295530213fa3dd899a"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/26/2017" />

# <a name="deploy-resources-to-azure"></a>将资源部署到 Azure

本主题演示如何将资源部署到 Azure 订阅。 可以使用 Azure PowerShell 或 Azure CLI 来部署用于定义解决方案的基础结构的 Resource Manager 模板。

有关 Resource Manager 概念的介绍，请参阅 [Azure Resource Manager 概述](/documentation/articles/resource-group-overview/)。

## <a name="steps-for-deployment"></a>部署步骤

本主题假定你要部署本主题中的[示例存储模板](#example-storage-template)。你可以使用其他模板，但是传递的参数不同于本主题中所示的参数。

创建模板后，用于部署模板的常规步骤如下：

1. 登录到你的帐户
2. 选择要使用的订阅（仅当有多个订阅且要使用的订阅不是默认订阅时，才有必要）
3. 创建资源组
4. 部署模板
5. 查看部署状态

以下部分说明如何使用 [PowerShell](#powershell) 或 [Azure CLI](#azure-cli) 来执行这些步骤。

## <a name="powershell"></a>PowerShell

1. 若要安装 Azure PowerShell，请参阅 [Azure PowerShell cmdlet 入门](https://docs.microsoft.com/zh-cn/powershell/azure/overview)。

2. 若要快速开始进行部署，请使用以下 cmdlet：

        Login-AzureRmAccount -EnvironmentName AzureChinaCloud
        Set-AzureRmContext -SubscriptionID {subscription-id}

        New-AzureRmResourceGroup -Name ExampleGroup -Location "China North"
        New-AzureRmResourceGroupDeployment -Name ExampleDeployment -ResourceGroupName ExampleGroup -TemplateFile c:\MyTemplates\azuredeploy.json 

    仅当要使用非默认订阅时，才需要 `Set-AzureRmContext` cmdlet。 若要查看所有订阅及其 ID，请使用：

        Get-AzureRmSubscription

3. 部署可能需要几分钟才能完成。 完成之后，将看到如下消息：

        DeploymentName          : ExampleDeployment
        CorrelationId           : 07b3b233-8367-4a0b-b9bc-7aa189065665
        ResourceGroupName       : ExampleGroup
        ProvisioningState       : Succeeded
        ...

4. 若要查看资源组和存储帐户是否已部署到订阅，请使用：

        Get-AzureRmResourceGroup -Name ExampleGroup
        Find-AzureRmResource -ResourceGroupNameEquals ExampleGroup

5. 部署模板时，可以指定模板参数作为 PowerShell 参数。 前面的示例未包括任何模板参数，因此使用了模板中的默认值。 若要部署其他存储帐户，并为存储名称前缀和存储帐户 SKU 提供参数值，请使用：

        New-AzureRmResourceGroupDeployment -Name ExampleDeployment2 -ResourceGroupName ExampleGroup -TemplateFile c:\MyTemplates\azuredeploy.json -storageNamePrefix "contoso" -storageSKU "Standard_GRS"

    现在，资源组中已有两个存储帐户。 

## <a name="azure-cli"></a>Azure CLI

1. 若要安装 Azure CLI，请参阅 [安装 Azure CLI 2.0](https://docs.microsoft.com/zh-cn/cli/azure/install-az-cli2)。

2. 若要快速开始进行部署，请使用以下命令：

        az login
        az account set --subscription {subscription-id}

        az group create --name ExampleGroup --location "China North"
        az group deployment create --name ExampleDeployment --resource-group ExampleGroup --template-file c:\MyTemplates\azuredeploy.json

    仅当要使用非默认订阅时，才需要 `az account set` 命令。 若要查看所有订阅及其 ID，请使用：

        az account list

3. 部署可能需要几分钟才能完成。 完成之后，将看到如下消息：

        "provisioningState": "Succeeded",

4. 若要查看资源组和存储帐户是否已部署到订阅，请使用：

        az group show --name ExampleGroup
        az resource list --resource-group ExampleGroup

5. 部署模板时，可以指定模板参数作为 PowerShell 参数。 前面的示例未包括任何模板参数，因此使用了模板中的默认值。 若要部署其他存储帐户，并为存储名称前缀和存储帐户 SKU 提供参数值，请使用：

        az group deployment create --name ExampleDeployment2 --resource-group ExampleGroup --template-file c:\MyTemplates\azuredeploy.json --parameters '{"storageNamePrefix":{"value":"contoso"},"storageSKU":{"value":"Standard_GRS"}}'

    现在，资源组中已有两个存储帐户。 

## <a name="example-storage-template"></a>示例存储模板

使用以下示例模板将存储帐户部署到订阅：

    {
      "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {
        "storageNamePrefix": {
          "type": "string",
          "maxLength": 11,
          "defaultValue": "storage",
          "metadata": {
            "description": "The value to use for starting the storage account name."
          }
        },
        "storageSKU": {
          "type": "string",
          "allowedValues": [
            "Standard_LRS",
            "Standard_ZRS",
            "Standard_GRS",
            "Standard_RAGRS",
            "Premium_LRS"
          ],
          "defaultValue": "Standard_LRS",
          "metadata": {
            "description": "The type of replication to use for the storage account."
          }
        }
      },
      "variables": {
        "storageName": "[concat(parameters('storageNamePrefix'), uniqueString(resourceGroup().id))]"
      },
      "resources": [
        {
          "name": "[variables('storageName')]",
          "type": "Microsoft.Storage/storageAccounts",
          "apiVersion": "2016-05-01",
          "sku": {
            "name": "[parameters('storageSKU')]"
          },
          "kind": "Storage",
          "location": "[resourceGroup().location]",
          "tags": {},
          "properties": {
          }
        }
      ],
      "outputs": {  }
    }

## <a name="next-steps"></a>后续步骤

* 有关使用 PowerShell 部署模板的详细信息，请参阅[使用 Resource Manager 模板和 Azure PowerShell 部署资源](/documentation/articles/resource-group-template-deploy/)。
* 有关使用 Azure CLI 部署模板的详细信息，请参阅[使用 Resource Manager 模板和 Azure CLI 部署资源](/documentation/articles/resource-group-template-deploy-cli/)。

<!--Update_Description:update meta properties; wording update; update link reference -->