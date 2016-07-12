<!-- ARM: tested -->

<properties
   pageTitle="在 Windows VM 上使用模板自定义脚本 | Microsoft Azure"
   description="通过将自定义脚本扩展与资源管理器模板配合使用，自动执行 Windows 的 Azure VM 配置任务"
   services="virtual-machines-windows"
   documentationCenter=""
   authors="kundanap"
   manager="timlt"
   editor=""
   tags="azure-resource-manager"/>

<tags
   ms.service="virtual-machines-windows"
   ms.date="03/29/2016"
   wacn.date="06/29/2016"/>

# 将 Windows VM 的自定义脚本扩展与 Azure 资源管理器模板配合使用

本文概述如何使用自定义脚本扩展编写 Azure 资源管理器模板，用于引导 Linux 或 Windows VM 上的工作负荷。

> [AZURE.NOTE]Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model/)。这篇文章介绍如何使用资源管理器部署模型，Azure 建议大多数新部署使用资源管理器模型替代 [经典部署模型](/documentation/articles/virtual-machines-windows-classic-extensions-customscript/)。

[AZURE.INCLUDE [virtual-machines-common-extensions-customscript](../includes/virtual-machines-common-extensions-customscript.md)]

## Windows VM 模板示例

在模板的 Resource 节中定义以下资源

       {
       "type": "Microsoft.Compute/virtualMachines/extensions",
       "name": "MyCustomScriptExtension",
       "apiVersion": "2015-05-01-preview",
       "location": "[parameters('location')]",
       "dependsOn": [
           "[concat('Microsoft.Compute/virtualMachines/',parameters('vmName'))]"
       ],
       "properties": {
           "publisher": "Microsoft.Compute",
           "type": "CustomScriptExtension",
           "typeHandlerVersion": "1.4",
           "settings": {
               "fileUris": [
               "http://Yourstorageaccount.blob.core.windows.net/customscriptfiles/start.ps1"
           ],
           "commandToExecute": "powershell.exe -ExecutionPolicy Unrestricted -File start.ps1"
         }
       }
     }

在上述示例中，请将文件 URL 和文件名替换为你自己的设置。

创作模板之后，可以使用 Azure Powershell 部署模板。

有关在 VM 上使用自定义脚本扩展配置应用程序的完整示例，请参阅以下示例。

[Windows VM 上的自定义脚本扩展](https://github.com/Azure/azure-quickstart-templates/blob/b1908e74259da56a92800cace97350af1f1fc32b/201-list-storage-keys-windows-vm/azuredeploy.json/)

<!---HONumber=Mooncake_0118_2016-->