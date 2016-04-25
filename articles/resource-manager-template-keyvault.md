<properties
   pageTitle="密钥保管库的 Resource Manager 模板 | Azure"
   description="介绍用于通过模板部署密钥保管库的 Resource Manager 架构。"
   services="azure-resource-manager,key-vault"
   documentationCenter="na"
   authors="tfitzmac"
   manager="wpickett"
   editor=""/>

<tags
   ms.service="azure-resource-manager"
   ms.date="02/23/2016"
   wacn.date="04/25/2016"/>

# 密钥保管库模板架构

创建密钥保管库。

## 架构格式

若要创建密钥保管库，请将以下架构添加到模板的 resources 节中。

    {
        "type": "Microsoft.KeyVault/vaults",
        "apiVersion": "2015-06-01",
        "name": string,
        "location": string,
        "properties": {
            "enabledForDeployment": bool,
            "enabledForTemplateDeployment": bool,
            "enabledForVolumeEncryption": bool,
            "tenantId": string,
            "accessPolicies": [
                {
                    "tenantId": string,
                    "objectId": string,
                    "permissions": {
                        "keys": [ keys permissions ],
                        "secrets": [ secrets permissions ]
                    }
                }
            ],
            "sku": {
                "name": enum,
                "family": "A"
            }
        },
        "resources": [
             child resources
        ]
    }

## 值

下表描述了需要在架构中设置的值。

| 名称 | 类型 | 必选 | 允许的值 | 说明 |
| ---- | ---- | -------- | ---------------- | ----------- |
| type | 枚举 | 是 | **Microsoft.KeyVault/vaults** | 要创建的资源类型。 |
| apiVersion | 枚举 | 是 | **2015-06-01** <br /> **2014-12-19-preview** | 要用于创建该资源的 API 版本。 | 
| name | 字符串 | 是 | | 要创建的密钥保管库的名称。该名称必须在全 Azure 中唯一。请考虑在你的命名约定中使用 [uniqueString](/documentation/articles/resource-group-template-functions#uniquestring) 函数，如下面的示例所示。 |
| location | 字符串 | 是 | 若要确定有效的区域，请参阅[支持的区域](/documentation/articles/resource-manager-supported-services#supported-regions)。 | 托管密钥保管库的区域。 |
| properties | 对象 | 是 | （[如下所示](#properties)） | 一个对象，用于指定要创建的密钥保管库的类型。 |
| resources | 数组 | 否 | [密钥保管库机密](/documentation/articles/resource-manager-template-keyvault-secret) | 密钥保管库的子资源。 |

<a id="properties"></a>
### 属性对象

| 名称 | 类型 | 必选 | 允许的值 | 说明 |
| ---- | ---- | -------- | ---------------- | ----------- |
| enabledForDeployment | 布尔值 | 否 | **true** 或 **false** | 指定是否已为虚拟机或 Service Fabric 部署启用了保管库。 |
| enabledForTemplateDeployment | 布尔值 | 否 | **true** 或 **false** | 指定是否在资源管理器模板部署中启用了保管库。有关详细信息，请参阅[在部署期间传递安全值](/documentation/articles/resource-manager-keyvault-parameter) |
| enabledForVolumeEncryption | 布尔值 | 否 | **true** 或 **false** | 指定是否为卷加密启用了保管库。 |
| tenantId | 字符串 | 是 | 全局唯一标识符 | 订阅的租户标识符。可以使用 **Get-AzureRMSubscription** PowerShell cmdlet 检索该值。 |
| accessPolicies | 数组 | 是 | （[如下所示](#accesspolicies)） | 最多包含 16 个对象的数组，指定用户或服务主体的权限。 |
| sku | 对象 | 是 | （[如下所示](#sku)） | 密钥保管库的 SKU。 |

<a id="accesspolicies"></a>
### properties.accessPolicies 对象

| 名称 | 类型 | 必选 | 允许的值 | 说明 |
| ---- | ---- | -------- | ---------------- | ----------- |
| tenantId | 字符串 | 是 | 全局唯一标识符 | 包含此访问策略中 **objectId** 的 Azure Active Directory 租户的租户标识符 |
| objectId | 字符串 | 是 | 全局唯一标识符 | 将有权访问保管库的 AAD 用户或服务主体的对象标识符。可以通过 **Get-AzureRMADUser** 或 **Get-AzureRMADServicePrincipal** cmdlet 检索该值。 |
| permissions | 对象 | 是 | （[如下所示](#permissions)） | 向 Active Directory 对象授予的对此保管库的权限。 |

<a id="permissions"></a>
### properties.accessPolicies.permissions 对象

| 名称 | 类型 | 必选 | 允许的值 | 说明 |
| ---- | ---- | -------- | ---------------- | ----------- |
| keys | 数组 | 是 | 以下值的逗号分隔列表：<br />**all**<br />**backup**<br />**create**<br />**decrypt**<br />**delete**<br />**encrypt**<br />**get**<br />**import**<br />**list**<br />**restore**<br />**sign**<br />**unwrapkey**<br/>**update**<br />**verify**<br />**wrapkey** | 向此 Active Directory 对象授予的，对此保管库中密钥的权限。必须将此值指定为允许值的数组。 |
| secrets | 数组 | 是 | 以下值的逗号分隔列表：<br />**all**<br />**delete**<br />**get**<br />**list**<br />**set** | 向此 Active Directory 对象授予的，对此保管库中机密的权限。必须将此值指定为允许值的数组。 |

<a id="sku"></a>
### properties.sku 对象

| 名称 | 类型 | 必选 | 允许的值 | 说明 |
| ---- | ---- | -------- | ---------------- | ----------- |
| name | 枚举 | 是 | **standard**<br />**premium** | 要使用的 KeyVault 服务层。标准支持机密和软件保护的密钥。高级版添加了对 HSM 保护密钥的支持。 |
| family | 枚举 | 是 | **A** | 要使用的 SKU 系列。 
 
	
## 示例

以下示例将部署密钥保管库和机密。

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

## 快速入门模板

以下快速入门模板将部署密钥保管库。

- [创建密钥保管库](https://github.com/Azure/azure-quickstart-templates/tree/master/101-key-vault-create)


## 后续步骤

- 有关密钥保管库的一般信息，请参阅 [Azure 密钥保管库入门](/documentation/articles/key-vault-get-started)。
- 有关在部署模板时引用密钥保管库机密的示例，请参阅[在部署期间传递安全值](/documentation/articles/resource-manager-keyvault-parameter)。


<!---HONumber=Mooncake_0418_2016-->