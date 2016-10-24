<properties
   pageTitle="将 Windows VM 上的自定义脚本与模板配合使用 | Azure"
   description="通过将自定义脚本扩展与 Resource Manager 模板配合使用，自动完成 Windows VM 配置任务"
   services="virtual-machines-windows"
   documentationCenter=""
   authors="kundanap"
   manager="timlt"
   editor=""
   tags="azure-resource-manager"/>  


<tags
   ms.service="virtual-machines-windows"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="vm-windows"
   ms.workload="infrastructure-services"
   ms.date="03/29/2016"
   wacn.date=""
   ms.author="kundanap"/>  


# 使用 Azure Resource Manager 模板的 Windows VM 自定义脚本扩展

[AZURE.INCLUDE [virtual-machines-common-extensions-customscript](../../includes/virtual-machines-common-extensions-customscript.md)]

## Windows VM 模板示例

在模板的 Resource 节中定义以下资源。

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
           "typeHandlerVersion": "1.7",
           "autoUpgradeMinorVersion":true,
           "settings": {
               "fileUris": [
               "http://Yourstorageaccount.blob.core.chinacloudapi.cn/customscriptfiles/start.ps1"
           ],
           "commandToExecute": "powershell.exe -ExecutionPolicy Unrestricted -File start.ps1"
         }
       }
     }

在前述示例中，请将文件 URL 和文件名替换为自己的设置。
创作模板之后，可以使用 Azure PowerShell 部署模板。

若要让脚本 URL 和参数仅供私人专用，可将脚本 URL 设置为“专用”。。如果将脚本 URL 设置为“专用”，则只能使用作为受保护设置发送的存储帐户名称和密钥进行访问。对于 1.7 版或更高版本的自定义脚本扩展，也可以受保护设置的形式提供脚本参数。

## 使用受保护设置的 Windows VM 模板示例

        {
        "publisher": "Microsoft.Compute",
        "type": "CustomScriptExtension",
        "typeHandlerVersion": "1.7",
        "settings": {
        "fileUris": [
        "http: //Yourstorageaccount.blob.core.chinacloudapi.cn/customscriptfiles/start.ps1"
        ]
        },
        "protectedSettings": {
        "commandToExecute": "powershell.exe -ExecutionPolicy Unrestricted -start.ps1",
        "storageAccountName": "yourStorageAccountName",
        "storageAccountKey": "yourStorageAccountKey"
        }
        }
若要了解最新版自定义脚本扩展的架构，请参阅 [Azure Windows VM extension configuration samples](/documentation/articles/virtual-machines-windows-extensions-configuration-samples/)（Azure Windows VM 扩展配置示例）。

有关使用自定义脚本扩展的 VM 上的应用程序配置的示例，请参阅 [Custom Script extension on a Windows VM](https://github.com/Azure/azure-quickstart-templates/blob/b1908e74259da56a92800cace97350af1f1fc32b/201-list-storage-keys-windows-vm/azuredeploy.json/)（Windows VM 上的自定义脚本扩展）。

<!---HONumber=Mooncake_1017_2016-->