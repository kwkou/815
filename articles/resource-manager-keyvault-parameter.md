<!-- Remove best-practices-rm-security -->
<properties
   pageTitle="通过 Resource Manager 模板使用密钥保管库机密 | Azure"
   description="说明在部署期间如何以参数形式从密钥保管库传递机密。"
   services="azure-resource-manager,key-vault"
   documentationCenter="na"
   authors="tfitzmac"
   manager="wpickett"
   editor=""/>

<tags
   ms.service="azure-resource-manager"
   ms.date="05/16/2016"
   wacn.date="06/20/2016"/>

# 在部署期间传递安全值

当你需要在部署期间以参数形式传递安全值（例如密码）时，可以将该值存储为 [Azure 密钥保管库](/documentation/articles/key-vault-whatis/)中的机密，并在其他资源管理器模板中引用该值。你只会在模板中包含对机密的引用，因此该机密永远都不会公开，并且不需要每次部署资源时都手动输入该机密的值。指定哪些用户或服务主体可以访问该机密。

> [AZURE.NOTE] 目前，只有 Azure CLI 支持引用密钥保管库机密的功能。Azure PowerShell 中即将添加此功能。

## 部署密钥保管库和机密

若要创建可从其他资源管理器模板引用的密钥保管库，必须将 **enabledForTemplateDeployment** 属性设置为 **true**，并且必须向相应的用户或服务主体授予访问权限，使其能够执行引用机密的部署。

若要了解如何部署密钥保管库和机密，请参阅[密钥保管库架构](/documentation/articles/resource-manager-template-keyvault/)和[密钥保管库机密架构](/documentation/articles/resource-manager-template-keyvault-secret/)。

## 引用机密

从用于将值传递到模板的参数文件内部引用机密。你可以通过传递密钥保管库的资源标识符和机密的名称来引用机密。

    "parameters": {
      "adminPassword": {
        "reference": {
          "keyVault": {
            "id": "/subscriptions/{guid}/resourceGroups/{group-name}/providers/Microsoft.KeyVault/vaults/{vault-name}"
          }, 
          "secretName": "sqlAdminPassword" 
        } 
      }
    }

整个参数文件可能如下所示：

    {
      "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {
        "sqlsvrAdminLogin": {
          "value": ""
        },
        "sqlsvrAdminLoginPassword": {
          "reference": {
            "keyVault": {
              "id": "/subscriptions/{guid}/resourceGroups/{group-name}/providers/Microsoft.KeyVault/vaults/{vault-name}"
            },
            "secretName": "adminPassword"
          }
        }
      }
    }

接受机密的参数应是 **securestring**。以下示例显示了某个模板的相关部分，该模板将部署需要管理员密码的 SQL Server。

    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "sqlsvrAdminLogin": {
                "type": "string",
                "minLength": 4
            },
            "sqlsvrAdminLoginPassword": {
                "type": "securestring"
            },
            ...
        },
        "resources": [
        {
            "name": "[variables('sqlsvrName')]",
            "type": "Microsoft.Sql/servers",
            "location": "[resourceGroup().location]",
            "apiVersion": "2014-04-01-preview",
            "properties": {
                "administratorLogin": "[parameters('sqlsvrAdminLogin')]",
                "administratorLoginPassword": "[parameters('sqlsvrAdminLoginPassword')]"
            },
            ...
        }],
        "variables": {
            "sqlsvrName": "[concat('sqlsvr', uniqueString(resourceGroup().id))]"
        },
        "outputs": { }
    }




## 后续步骤

- 有关密钥保管库的一般信息，请参阅 [Azure 密钥保管库入门](/documentation/articles/key-vault-get-started/)。
<!-- - 有关对虚拟机使用密钥保管库的信息，请参阅 [Azure Resource Manager 的安全注意事项](/documentation/articles/best-practices-resource-manager-security/)。 -->
- 有关引用密钥机密的完整示例，请参阅[密钥保管库示例](https://github.com/rjmax/ArmExamples/tree/master/keyvaultexamples)。

<!---HONumber=Mooncake_0314_2016-->