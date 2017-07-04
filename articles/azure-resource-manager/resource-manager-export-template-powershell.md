<properties
    pageTitle="使用 Azure PowerShell 导出 Resource Manager 模板 | Azure"
    description="使用 Azure Resource Manager 和 Azure PowerShell 从资源组导出模板。"
    services="azure-resource-manager"
    documentationcenter="na"
    author="tfitzmac"
    manager="timlt"
    editor="tysonn" />
<tags
    ms.service="azure-resource-manager"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="05/01/2017"
    wacn.date="06/05/2017"
    ms.author="v-yeche"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="08618ee31568db24eba7a7d9a5fc3b079cf34577"
    ms.openlocfilehash="946a141c779ef5dc54534a7f25c1e1f7638fac3f"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/26/2017" />

# <a name="export-azure-resource-manager-templates-with-powershell"></a>使用 PowerShell 导出 Azure Resource Manager 模板

使用 Resource Manager 可从订阅中的现有资源导出 Resource Manager 模板。 你可以使用该生成的模板了解模板语法，或根据需要自动重新部署解决方案。

必须注意的是，可以使用两种不同的方式来导出模板：

* 可以导出已用于部署的实际模板。 导出的模板中包括的所有参数和变量与原始模板中显示的完全一样。 如果需要检索模板，此方法非常有用。
* 你可以导出表示资源组当前状态的模板。 导出的模板不基于任何已用于部署的模板。 与之相反，它所创建的模板为资源组的快照。 导出的模板会有许多硬编码的值，其参数可能没有定义的那么多。 如果已修改资源组，则可以使用此方法。 现在，需要捕获资源组作为模板。

本主题演示这两种方法。

## <a name="deploy-a-solution"></a>部署解决方案

为了说明用于导出模板的这两种方法，让我们首先将解决方案部署到订阅。 如果订阅中已有要导出的资源组，则无需部署此解决方案。 但是，本文的剩余部分指的是此解决方案的模板。 该示例脚本可部署存储帐户。

    New-AzureRmResourceGroup -Name ExampleGroup -Location "China East"
    New-AzureRmResourceGroupDeployment -ResourceGroupName ExampleGroup `
      -DeploymentName NewStorage
      -TemplateUri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-storage-account-create/azuredeploy.json

## <a name="save-template-from-deployment-history"></a>从部署历史记录保存模板

可以使用 [Save-AzureRmResourceGroupDeploymentTemplate](https://docs.microsoft.com/zh-cn/powershell/module/azurerm.resources/save-azurermresourcegroupdeploymenttemplate) 命令从部署历史记录中检索模板。 以下示例保存以前部署的模板：

    Save-AzureRmResourceGroupDeploymentTemplate -ResourceGroupName ExampleGroup -DeploymentName NewStorage

它将返回模板的位置。

    Path
    ----
    C:\Users\exampleuser\NewStorage.json

打开该文件，并请注意，它就是用于部署的模板。 参数和变量与 GitHub 中的模板匹配。 可以重新部署此模板。

## <a name="export-resource-group-as-template"></a>将资源组导出为模板

可以使用 [Export-AzureRmResourceGroup](https://docs.microsoft.com/zh-cn/powershell/module/azurerm.resources/export-azurermresourcegroup) 命令检索表示资源组当前状态的模板，而不是从部署历史记录中检索模板。 如果对资源组进行了许多更改，并且现有模板未表示所有更改，则可以使用此命令。

    Export-AzureRmResourceGroup -ResourceGroupName ExampleGroup

它将返回模板的位置。

    Path
    ----
    C:\Users\exampleuser\ExampleGroup.json

打开该文件，并请注意，它与 GitHub 中的模板不同。 它具有不同参数，并且没有变量。 存储 SKU 和位置均硬编码为值。 以下示例演示导出的模板，但你的模板可以使用略有不同的参数名称：

    {
      "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {
        "storageAccounts_nf3mvst4nqb36standardsa_name": {
          "defaultValue": null,
          "type": "String"
        }
      },
      "variables": {},
      "resources": [
        {
          "type": "Microsoft.Storage/storageAccounts",
          "sku": {
            "name": "Standard_LRS",
            "tier": "Standard"
          },
          "kind": "Storage",
          "name": "[parameters('storageAccounts_nf3mvst4nqb36standardsa_name')]",
          "apiVersion": "2016-01-01",
          "location": "chinaeast",
          "tags": {},
          "properties": {},
          "dependsOn": []
        }
      ]
    }

可以重新部署此模板，但需要猜测存储帐户的唯一名称。 参数名称可以略有不同。

    New-AzureRmResourceGroupDeployment -ResourceGroupName ExampleGroup `
      -TemplateFile C:\Users\exampleuser\ExampleGroup.json `
      -storageAccounts_nf3mvst4nqb36standardsa_name tfnewstorage0501

## <a name="customize-exported-template"></a>自定义导出的模板

可以修改此模板，使其更易用且更灵活。 若要允许多个位置，请更改 location 属性以使用与资源组相同的位置：

    "location": "[resourceGroup().location]",

若要避免必须猜测存储帐户的唯一名称，请删除存储帐户名称的参数。 为存储名称后缀及存储 SKU 添加参数：

    "parameters": {
        "storageSuffix": {
          "type": "string",
          "defaultValue": "standardsa"
        },
        "storageAccountType": {
          "defaultValue": "Standard_LRS",
          "allowedValues": [
            "Standard_LRS",
            "Standard_GRS",
            "Standard_ZRS"
          ],
          "type": "string",
          "metadata": {
            "description": "Storage Account type"
          }
        }
    },

使用 uniqueString 函数添加用于构造存储帐户名称的变量：

    "variables": {
        "storageAccountName": "[concat(uniquestring(resourceGroup().id), 'standardsa')]"
      },

将存储帐户名称设置为变量：

    "name": "[variables('storageAccountName')]",

将 SKU 设置为参数：

    "sku": {
        "name": "[parameters('storageAccountType')]",
        "tier": "Standard"
    },

模板现在如下所示：

    {
      "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {
        "storageSuffix": {
          "type": "string",
          "defaultValue": "standardsa"
        },
        "storageAccountType": {
          "defaultValue": "Standard_LRS",
          "allowedValues": [
            "Standard_LRS",
            "Standard_GRS",
            "Standard_ZRS"
          ],
          "type": "string",
          "metadata": {
            "description": "Storage Account type"
          }
        }
      },
      "variables": {
        "storageAccountName": "[concat(uniquestring(resourceGroup().id), parameters('storageSuffix'))]"
      },
      "resources": [
        {
          "type": "Microsoft.Storage/storageAccounts",
          "sku": {
            "name": "[parameters('storageAccountType')]",
            "tier": "Standard"
          },
          "kind": "Storage",
          "name": "[variables('storageAccountName')]",
          "apiVersion": "2016-01-01",
          "location": "[resourceGroup().location]",
          "tags": {},
          "properties": {},
          "dependsOn": []
        }
      ]
    }

重新部署已修改的模板。

## <a name="next-steps"></a>后续步骤
* 有关使用门户导出模板的信息，请参阅[从现有资源导出 Azure Resource Manager 模板](/documentation/articles/resource-manager-export-template/)。
* 若要在模板中定义参数，请参阅[创作模板](/documentation/articles/resource-group-authoring-templates/#parameters)。
* 有关解决常见部署错误的提示，请参阅[排查使用 Azure Resource Manager 时的常见 Azure 部署错误](/documentation/articles/resource-manager-common-deployment-errors/)。

<!--Update_Description:new article about illustrating how to export template with powershell -->
