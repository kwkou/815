<properties
   pageTitle="将 Desired State Configuration 与虚拟机规模集配合使用 | Azure"
   description="将虚拟机规模集与 Azure DSC 扩展配合使用"
   services="virtual-machines-windows"
   documentationCenter=""
   authors="zjalexander"
   manager="timlt"
   editor=""
   tags="azure-service-management,azure-resource-manager"
   keywords=""/>  


<tags
   ms.service="virtual-machines-scale-sets"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="vm-windows"
   ms.workload="na"
   ms.date="08/29/2016"
   wacn.date="10/17/2016"
   ms.author="zachal"/>  


# 将虚拟机规模集与 Azure DSC 扩展配合使用

[虚拟机规模集 (VMSS)](/documentation/articles/virtual-machines-windows-vmss-powershell-creating/) 可与 [Azure Desired State Configuration (DSC)](/documentation/articles/virtual-machines-windows-extensions-dsc-overview/) 扩展处理程序配合使用。VMSS 用于配置接受负载的虚拟机部署。VM 联机时，DSC 用于配置 VM，使它们能够运行生产软件。

## 部署到 VM 和部署到 VMSS 的差异

VMSS 的基础模板结构与单一 VM 略有不同。具体而言，单一 VM 将扩展部署在“virtualMachines”节点下面。有一个“extensions”类型的入口，DSC 将通过此处添加到模板中

    "resources": [
            {
                "name": "Microsoft.Powershell.DSC",
                "type": "extensions",
                "location": "[resourceGroup().location]",
                "apiVersion": "2015-06-15",
                "dependsOn": [
                    "[concat('Microsoft.Compute/virtualMachines/', variables('vmName'))]"
                ],
                "tags": {
                    "displayName": "dscExtension"
                },
                "properties": {
                    "publisher": "Microsoft.Powershell",
                    "type": "DSC",
                    "typeHandlerVersion": "2.20",
                    "autoUpgradeMinorVersion": false,
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
                }
            }
        ]

VMSS 节点有一个“properties”节，其中包含“VirtualMachineProfile”和“extensionProfile”属性。DSC 添加在“extensions”下面

    "extensionProfile": {
                "extensions": [
                    {
                        "name": "Microsoft.Powershell.DSC",
                        "properties": {
                            "publisher": "Microsoft.Powershell",
                            "type": "DSC",
                            "typeHandlerVersion": "2.9",
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

## VMSS 的行为

VMSS 的行为与单一 VM 的行为相同。创建新 VM 后，会自动使用 DSC 扩展对其进行预配。如果扩展需要更新的 WMF 版本，则 VM 会重新启动，然后联机。VM 联机后，将下载 DSC 配置 .zip 文件，并在 VM 上预配该文件。在 [Azure DSC Extension Overview](/documentation/articles/virtual-machines-windows-extensions-dsc-overview/)（Azure DSC 扩展概述）中可以找到详细信息。

<!---HONumber=Mooncake_1010_2016-->