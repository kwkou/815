<properties
   pageTitle="Desired State Configuration Resource Manager 模板 | Azure"
   description="Azure 中 Desired State Configuration 的 Resource Manager 模板定义，提供示例和故障排除方法"
   services="virtual-machines-windows"
   documentationCenter=""
   authors="zjalexander"
   manager="timlt"
   editor=""
   tags="azure-service-management,azure-resource-manager"
   keywords=""/>  


<tags
   ms.service="virtual-machines-windows"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="vm-windows"
   ms.workload="na"
   ms.date="08/29/2016"
   wacn.date="10/17/2016"
   ms.author="zachal"/>  


# 在 Azure Resource Manager 模板中使用 Windows VMSS 和 Desired State Configuration
本文介绍 [Desired State Configuration 扩展处理程序](/documentation/articles/virtual-machines-windows-extensions-dsc-overview/)的 Resource Manager 模板。

## Windows VM 模板示例

将以下代码片段放入模板的 Resource 节。

			"name": "Microsoft.Powershell.DSC",
			"type": "extensions",
             "location": "[resourceGroup().location]",
             "apiVersion": "2015-06-15",
             "dependsOn": [
                  "[concat('Microsoft.Compute/virtualMachines/', variables('vmName'))]"
              ],
              "properties": {
                  "publisher": "Microsoft.Powershell",
                  "type": "DSC",
                  "typeHandlerVersion": "2.20",
                  "autoUpgradeMinorVersion": true,
                  "forceUpdateTag": "[parameters('dscExtensionUpdateTagVersion')]",
                  "settings": {
                      "configuration": {
                          "url": "[concat(parameters('_artifactsLocation'), '/', variables('dscExtensionArchiveFolder'), '/', variables('dscExtensionArchiveFileName'))]",
                          "script": "dscExtension.ps1",
                          "function": "Main"
                      },
                      "configurationArguments": {
                          "nodeName": "[variables('vmName')]"
                      }
                  },
                  "protectedSettings": {
                      "configurationUrlSasToken": "[parameters('_artifactsLocationSasToken')]"
                  }


## Windows VMSS 的模板示例

VMSS 节点具有“properties”节，其中包含“VirtualMachineProfile”和“extensionProfile”属性。DSC 添加在“extensions”下面。

    "extensionProfile": {
                "extensions": [
                    {
                        "name": "Microsoft.Powershell.DSC",
                        "properties": {
                            "publisher": "Microsoft.Powershell",
                            "type": "DSC",
                            "typeHandlerVersion": "2.20",
                            "autoUpgradeMinorVersion": true,
                            "forceUpdateTag": "[parameters('DscExtensionUpdateTagVersion')]",
                            "settings": {
                                "configuration": {
                                    "url": "[concat(parameters('_artifactsLocation'), '/', variables('DscExtensionArchiveFolder'), '/', variables('DscExtensionArchiveFileName'))]",
                                    "script": "DscExtension.ps1",
                                    "function": "Main"
                                },
                                "configurationArguments": {
                                    "nodeName": "localhost"
                                }
                            },
                            "protectedSettings": {
                                "configurationUrlSasToken": "[parameters('_artifactsLocationSasToken')]"
                            }
                        }
                    }
                ]

## 详细设置信息

以下架构用于 Azure Resource Manager 模板中 Azure DSC 扩展的 settings 部分。

    "settings": {
    "wmfVersion": "latest",
    "configuration": {
        "url": "http://validURLToConfigLocation",
        "script": "ConfigurationScript.ps1",
        "function": "ConfigurationFunction"
    },
    "configurationArguments": {
        "argument1": "Value1",
        "argument2": "Value2"
    },
    "configurationData": {
        "url": "https://foo.psd1"
    },
    "privacy": {
        "dataCollection": "enable"
    },
    "advancedOptions": {
        "downloadMappings": {
        "customWmfLocation": "http://myWMFlocation"
        }
    } 
    },
    "protectedSettings": {
    "configurationArguments": {
        "parameterOfTypePSCredential1": {
        "userName": "UsernameValue1",
        "password": "PasswordValue1"
        },
        "parameterOfTypePSCredential2": {
        "userName": "UsernameValue2",
        "password": "PasswordValue2"
        }
    },
    "configurationUrlSasToken": "?g!bber1sht0k3n",
    "configurationDataUrlSasToken": "?dataAcC355T0k3N"
    }

## 详细信息
| 属性名称 | 类型 | 说明 |
| --- | --- | --- |
| settings.wmfVersion | 字符串 | 指定应在 VM 上安装的 Windows Management Framework 版本。将此属性设置为“latest”可安装最新版本的 WMF。目前，此属性的可能值只有“4.0”、“5.0”、“5.0PP”和“latest”。这些可能值将来可能会更新。默认值为“latest”。|
| settings.configuration.url | 字符串 | 指定要从中下载 DSC 配置 zip 文件的 URL 位置。如果提供的 URL 需要 SAS 令牌才能访问，必须将 protectedSettings.configurationUrlSasToken 属性设置为 SAS 令牌的值。如果已定义 settings.configuration.script 和/或 settings.configuration.function，则需要此属性。 |
| settings.configuration.script | 字符串 | 指定包含 DSC 配置定义的脚本的文件名。此脚本必须位于从 configuration.url 属性所指定的 URL 下载的 zip 文件的根文件夹中。如果已定义 settings.configuration.url 和/或 settings.configuration.script，则需要此属性。 |
| settings.configuration.function | 字符串 | 指定 DSC 配置的名称。命名的配置必须包含在 configuration.script 定义的脚本中。如果已定义 settings.configuration.url 和/或 settings.configuration.function，则需要此属性。 |
| settings.configurationArguments | 集合 | 定义想要传递到 DSC 配置的任何参数。此属性未加密。 |
| settings.configurationData.url | 字符串 | 指定 URL，将从中下载配置数据 (.pds1) 文件用作 DSC 配置的输入。如果提供的 URL 需要 SAS 令牌才能访问，必须将 protectedSettings.configurationDataUrlSasToken 属性设置为 SAS 令牌的值。|
| settings.privacy.dataEnabled | 字符串 | 启用或禁用遥测数据收集。此属性的可能值只有“Enable”、“Disable”、'' 或 $null。将此属性留空或 null 可启用遥测。默认值为 ''。[更多信息](https://blogs.msdn.microsoft.com/powershell/2016/02/02/azure-dsc-extension-data-collection-2/) |
| settings.advancedOptions.downloadMappings | 集合 | 定义要从中下载 WMF 的备选位置。[更多信息](http://blogs.msdn.com/b/powershell/archive/2015/10/21/azure-dsc-extension-2-2-amp-how-to-map-downloads-of-the-extension-dependencies-to-your-own-location.aspx) |
| protectedSettings.configurationArguments | 集合 | 定义想要传递到 DSC 配置的任何参数。此属性已加密。 |
| protectedSettings.configurationUrlSasToken | 字符串 | 指定用于访问 configuration.url 所定义的 URL 的 SAS 令牌。此属性已加密。 |
| protectedSettings.configurationDataUrlSasToken | 字符串 | 指定用于访问 configurationData.url 所定义的 URL 的 SAS 令牌。此属性已加密。 |

## Settings 与ProtectedSettings
所有设置保存在 VM 上的 settings 文本文件中。
“settings”下面的属性是公共属性，因为它们未在 settings 文本文件中加密。
“protectedSettings”下面的属性已使用证书加密，因此不会在 VM 上以明文显示在此文件中。

如果配置需要凭据，可将凭据包含在 protectedSettings 中：

    "protectedSettings": {
        "configurationArguments": {
            "parameterOfTypePSCredential1": {
                "userName": "UsernameValue1",
                "password": "PasswordValue1"
            }
        }
    }

## 示例

以下示例摘自 [DSC Extension Handler Overview](/documentation/articles/virtual-machines-windows-extensions-dsc-overview/)（DSC 扩展处理程序概述）网页中的“Getting Started”（入门）部分。
此示例使用 Resource Manager 模板而不是cmdlet 来部署该扩展。 
保存“IisInstall.ps1”配置，将它放在 .ZIP 文件中，然后将该文件上载到可访问的 URL 中。此示例使用 Azure Blob 存储，但可以从任意位置下载 .ZIP 文件。

在 Azure Resource Manager 模板中，以下代码指示 VM 下载正确的文件并运行适当的 PowerShell 函数：

    "settings": {
        "configuration": {
            "url": "https://demo.blob.core.chinacloudapi.cn/",
            "script": "IisInstall.ps1",
            "function": "IISInstall"
        }
        } 
    },
    "protectedSettings": {
        "configurationUrlSasToken": "odLPL/U1p9lvcnp..."
    }

## 从以前的格式进行更新
以前格式（包含 ModulesUrl、ConfigurationFunction、SasToken 或 Properties 等公共属性）中的所有设置将自动调整为当前格式，并按以前的相同方式运行。

以前的 settings 架构如下所示：

    "settings": {
        "WMFVersion": "latest",
        "ModulesUrl": "https://UrlToZipContainingConfigurationScript.ps1.zip",
        "SasToken": "SAS Token if ModulesUrl points to private Azure Blob Storage",
        "ConfigurationFunction": "ConfigurationScript.ps1\\ConfigurationFunction",
        "Properties":  {
            "ParameterToConfigurationFunction1":  "Value1",
            "ParameterToConfigurationFunction2":  "Value2",
            "ParameterOfTypePSCredential1": {
                "UserName": "UsernameValue1",
                "Password": "PrivateSettingsRef:Key1" 
            },
            "ParameterOfTypePSCredential2": {
                "UserName": "UsernameValue2",
                "Password": "PrivateSettingsRef:Key2"
            }
        }
    },
    "protectedSettings": { 
        "Items": {
            "Key1": "PasswordValue1",
            "Key2": "PasswordValue2"
        },
        "DataBlobUri": "https://UrlToConfigurationDataWithOptionalSasToken.psd1"
    }

以前的格式调整为当前格式后的情况如下所示：

| 属性名称 | 以前架构的等效值 |
| --- | --- |
| settings.wmfVersion | settings.WMFVersion |
| settings.configuration.url | settings.ModulesUrl |
| settings.configuration.script | settings.ConfigurationFunction 的第 1 个部分（在 '\\\\' 前面） |
| settings.configuration.function | settings.ConfigurationFunction 的第 2 个部分（在 '\\\\' 后面） |
| settings.configurationArguments | settings.Properties |
| settings.configurationData.url | protectedSettings.DataBlobUri（没有 SAS 令牌） |
| settings.privacy.dataEnabled | settings.Privacy.DataEnabled |
| settings.advancedOptions.downloadMappings | settings.AdvancedOptions.DownloadMappings |
| protectedSettings.configurationArguments | protectedSettings.Properties |
| protectedSettings.configurationUrlSasToken | settings.SasToken |
| protectedSettings.configurationDataUrlSasToken | protectedSettings.DataBlobUri 中的 SAS 令牌 |


## 故障排除 - 错误代码 1100
错误代码 1100 指示 DSC 扩展中的用户输入有问题。
这些错误的文本是可变的，将来可能会更改。
下面是可能会遇到的一些错误及其解决方法。

### 无效值
“Privacy.dataCollection 为‘{0}’。可能的值只有 ''、'Enable' 和 'Disable'”。“WmfVersion 为‘{0}’。可能值只有 ... 和 'latest'”

问题：不允许使用提供的值。

解决方法：将无效值更改为有效值。请参阅“详细信息”部分中的表格。

### 无效的 URL
“ConfigurationData.url 为‘{0}’。这不是有效的 URL”。“DataBlobUri 为‘{0}’。这不是有效的 URL”。“Configuration.url 为‘{0}’。这不是有效的 URL”

问题：提供的 URL 无效。

解决方法：检查提供的所有 URL。确保所有 URL 都解析为扩展可在远程计算机上访问的有效位置。

### 无效的 ConfigurationArgument 类型
“无效的 configurationArguments 类型 {0}”

问题：ConfigurationArguments 属性无法解析为哈希表对象。

解决方法：将 ConfigurationArguments 属性设置为哈希表。遵循上述示例中提供的格式。请注意引号、逗号和括号。

### 重复的 ConfigurationArguments
“在公共和受保护的 configurationArguments 中找到重复的参数‘{0}’”

问题：公共设置中的 ConfigurationArguments 和受保护设置中的 ConfigurationArguments 包含同名属性。

解决方法：删除其中一个重复的属性。

### 缺少属性
“Configuration.function 要求指定 configuration.url 或 configuration.module”

“Configuration.url 要求指定 configuration.script”

“Configuration.script 要求指定 configuration.url”

“Configuration.url 要求指定 configuration.function”

“ConfigurationUrlSasToken 要求指定 configuration.url”

“ConfigurationDataUrlSasToken 要求指定 configurationData.url”

问题：定义的属性需要另一个缺少的属性。

解决方案：
- 提供缺少的属性。
- 删除需要缺失属性的属性。

<!---HONumber=Mooncake_1010_2016-->