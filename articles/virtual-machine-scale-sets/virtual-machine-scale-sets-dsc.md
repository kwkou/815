<properties
   pageTitle="将 Desired State Configuration 与虚拟机规模集配合使用 | Azure"
   description="将虚拟机规模集与 Azure DSC 扩展配合使用"
   services="virtual-machine-scale-sets"
   documentationCenter=""
   authors="zjalexander"
   manager="timlt"
   editor=""
   tags="azure-service-management,azure-resource-manager"
   keywords=""/>  


<tags
   ms.service="virtual-machine-scale-sets"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="vm-windows"
   ms.workload="na"
   ms.date="09/15/2016"
   wacn.date="10/31/2016"
   ms.author="zachal"/>  


# 将虚拟机规模集与 Azure DSC 扩展配合使用

[虚拟机规模集 (VMSS)](/documentation/articles/virtual-machine-scale-sets-overview/) 可与 [Azure Desired State Configuration (DSC)](/documentation/articles/virtual-machines-windows-extensions-dsc-overview/) 扩展处理程序配合使用。VMSS 提供部署和管理大量虚拟机的方法，并且可根据负载情况实现弹性扩大和缩小。VM 联机时，DSC 用于配置 VM，使它们能够运行生产软件。

## 部署到 VM 和部署到 VMSS 之间的差异

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
                            "script": "DscExtension.ps1",
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
                            "typeHandlerVersion": "2.20",
                            "autoUpgradeMinorVersion": false,
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

## 后续步骤 ##
检查 [Azure Resource Manager template for the DSC extension](/documentation/articles/virtual-machines-windows-extensions-dsc-template/)（适用于 DSC 扩展的 Azure Resource Manager 模板）。

了解[DSC 扩展安全处理凭据](/documentation/articles/virtual-machines-windows-extensions-dsc-credentials/)的方法。

有关 Azure DSC 扩展处理程序的详细信息，请参阅 [Introduction to the Azure Desired State Configuration extension handler](/documentation/articles/virtual-machines-windows-extensions-dsc-overview/)（Azure Desired State Configuration 扩展处理程序简介）。

有关 PowerShell DSC 的详细信息，请[访问 PowerShell 文档中心](https://msdn.microsoft.com/powershell/dsc/overview)。

<!---HONumber=Mooncake_1024_2016-->