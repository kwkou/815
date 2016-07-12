<properties
   pageTitle="密钥保管库中机密的 Resource Manager 模板 | Azure"
   description="介绍用于通过模板部署密钥保管库机密的Resource Manager 架构。"
   services="azure-resource-manager,key-vault"
   documentationCenter="na"
   authors="tfitzmac"
   manager="timlt"
   editor=""/>

<tags
   ms.service="azure-resource-manager"
   ms.date="04/05/2016"
   wacn.date="05/05/2016"/>

# 密钥保管库机密模板架构

创建存储于密钥保管库中的机密。此资源类型通常部署为[密钥保管库](/documentation/articles/resource-manager-template-keyvault/)的子资源。

## 架构格式

若要创建密钥保管库机密，请将以下架构添加到你的模板。可将机密定义为密钥保管库的子资源或定义为顶级资源。在同一个模板中部署密钥保管库时，可将机密定义为子资源。如果密钥保管库不是部署在同一个模板中，或者需要通过对资源类型执行循环来创建多个密码，则需要将机密定义为顶级资源。

    {
        "type": enum,
        "apiVersion": "2015-06-01",
        "name": string,
        "properties": {
            "value": string
        },
        "dependsOn": [ array values ]
    }

## 值

下表描述了需要在架构中设置的值。

| 名称 | 值 |
| ---- | ---- | 
| type | 枚举<br />必需<br />**secrets**（部署为密钥保管库的子资源时）或<br /> **Microsoft.KeyVault/vaults/secrets**（部署为顶级资源时）<br /><br />要创建的资源类型。 |
| apiVersion | 枚举<br />必需<br />**2015-06-01** 或 **2014-12-19-preview**<br /><br />要用于创建该资源的 API 版本。 | 
| name | 字符串<br />必需<br />一个单词（部署为密钥保管库的子资源时），采用格式 **{key-vault-name}/{secret-name}**（部署为要添加到现有密钥保管库的顶级资源时）。<br /><br />要创建的机密的名称。 |
| properties | 对象<br />必需<br />[properties 对象](#properties)<br /><br />一个对象，用于指定要创建的机密的值。 |
| dependsOn | 数组<br />可选<br />资源名称或资源唯一标识符的逗号分隔列表。<br /><br />此链接所依赖的资源的集合。如果将机密的密钥保管库部署在同一个模板中，请在此元素中包含密钥保管库的名称，以确保首先部署它。 |

<a id="properties"></a>
### 属性对象

| 名称 | 值 |
| ---- | ---- | 
| value | 字符串<br />必需<br /><br />要在密钥保管库中存储的机密值。传入此属性的值时，请使用类型为 **securestring** 的参数。 |

	
## 示例

第一个示例将机密部署为密钥保管库的子资源。

    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "keyVaultName": {
                "type": "string",
                "metadata": {
                    "description": "Name of the vault"
                }
            },
            "tenantId": {
                "type": "string",
                "metadata": {
                   "description": "Tenant Id for the subscription and use assigned access to the vault. Available from the Get-AzureRMSubscription PowerShell cmdlet"
                }
            },
            "objectId": {
                "type": "string",
                "metadata": {
                    "description": "Object Id of the AAD user or service principal that will have access to the vault. Available from the Get-AzureRMADUser or the Get-AzureRMADServicePrincipal cmdlets"
                }
            },
            "keysPermissions": {
                "type": "array",
                "defaultValue": [ "all" ],
                "metadata": {
                    "description": "Permissions to grant user to keys in the vault. Valid values are: all, create, import, update, get, list, delete, backup, restore, encrypt, decrypt, wrapkey, unwrapkey, sign, and verify."
                }
            },
            "secretsPermissions": {
                "type": "array",
                "defaultValue": [ "all" ],
                "metadata": {
                    "description": "Permissions to grant user to secrets in the vault. Valid values are: all, get, set, list, and delete."
                }
            },
            "vaultSku": {
                "type": "string",
                "defaultValue": "Standard",
                "allowedValues": [
                    "Standard",
                    "Premium"
                ],
                "metadata": {
                    "description": "SKU for the vault"
                }
            },
            "enabledForDeployment": {
                "type": "bool",
                "defaultValue": false,
                "metadata": {
                    "description": "Specifies if the vault is enabled for VM or Service Fabric deployment"
                }
            },
            "enabledForTemplateDeployment": {
                "type": "bool",
                "defaultValue": false,
                "metadata": {
                    "description": "Specifies if the vault is enabled for ARM template deployment"
                }
            },
            "enableVaultForVolumeEncryption": {
                "type": "bool",
                "defaultValue": false,
                "metadata": {
                    "description": "Specifies if the vault is enabled for volume encryption"
                }
            },
            "secretName": {
                "type": "string",
                "metadata": {
                    "description": "Name of the secret to store in the vault"
                }
            },
            "secretValue": {
                "type": "securestring",
                "metadata": {
                    "description": "Value of the secret to store in the vault"
                }
            }
        },
        "resources": [
        {
            "type": "Microsoft.KeyVault/vaults",
            "name": "[parameters('keyVaultName')]",
            "apiVersion": "2015-06-01",
            "location": "[resourceGroup().location]",
            "tags": {
                "displayName": "KeyVault"
            },
            "properties": {
                "enabledForDeployment": "[parameters('enabledForDeployment')]",
                "enabledForTemplateDeployment": "[parameters('enabledForTemplateDeployment')]",
                "enabledForVolumeEncryption": "[parameters('enableVaultForVolumeEncryption')]",
                "tenantId": "[parameters('tenantId')]",
                "accessPolicies": [
                {
                    "tenantId": "[parameters('tenantId')]",
                    "objectId": "[parameters('objectId')]",
                    "permissions": {
                        "keys": "[parameters('keysPermissions')]",
                        "secrets": "[parameters('secretsPermissions')]"
                    }
                }],
                "sku": {
                    "name": "[parameters('vaultSku')]",
                    "family": "A"
                }
            },
            "resources": [
            {
                "type": "secrets",
                "name": "[parameters('secretName')]",
                "apiVersion": "2015-06-01",
                "properties": {
                    "value": "[parameters('secretValue')]"
                },
                "dependsOn": [
                    "[concat('Microsoft.KeyVault/vaults/', parameters('keyVaultName'))]"
                ]
            }]
        }]
    }

第二个示例将机密部署为存储在现有密钥保管库中的顶级资源。

    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "keyVaultName": {
                "type": "string",
                "metadata": {
                    "description": "Name of the existing vault"
                }
            },
            "secretName": {
                "type": "string",
                "metadata": {
                    "description": "Name of the secret to store in the vault"
                }
            },
            "secretValue": {
                "type": "securestring",
                "metadata": {
                    "description": "Value of the secret to store in the vault"
                }
            }
        },
        "variables": {},
        "resources": [
            {
                "type": "Microsoft.KeyVault/vaults/secrets",
                "apiVersion": "2015-06-01",
                "name": "[concat(parameters('keyVaultName'), '/', parameters('secretName'))]",
                "properties": {
                    "value": "[parameters('secretValue')]"
                }
            }
        ],
        "outputs": {}
    }


## 后续步骤

- 有关密钥保管库的一般信息，请参阅 [Azure 密钥保管库入门](/documentation/articles/key-vault-get-started/)。
- 有关在部署模板时引用密钥保管库机密的示例，请参阅[在部署期间传递安全值](/documentation/articles/resource-manager-keyvault-parameter/)。



<!---HONumber=Mooncake_0425_2016-->