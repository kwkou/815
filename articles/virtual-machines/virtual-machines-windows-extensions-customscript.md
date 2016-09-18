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
	ms.date="03/29/2016"
	wacn.date="09/12/2016"/>  


# 将 Windows VM 的自定义脚本扩展与 Azure Resource Manager 模板配合使用

[AZURE.INCLUDE [virtual-machines-common-extensions-customscript](../../includes/virtual-machines-common-extensions-customscript.md)]

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

在上述示例中，请将文件 URL 和文件名替换为自己的设置。
创作模板之后，可以使用 Azure Powershell 部署模板。

在许多情况下，客户想要将脚本 URL 和参数保留为专用。为此，只需将脚本 URL 保留为专用，以便只能使用存储帐户名和密钥（以受保护设置的形式发送）访问它。此外，提供脚本参数时，也可以使用 1.7 版或更高版本的 Windows 自定义脚本扩展，以受保护设置的形式提供。

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

有关最新版自定义脚本扩展的架构的信息，请参阅[此文档](/documentation/articles/virtual-machines-windows-extensions-configuration-samples/)

有关在 VM 上使用自定义脚本扩展配置应用程序的完整示例，请参阅以下示例。

* [Windows VM 上的自定义脚本扩展](https://github.com/Azure/azure-quickstart-templates/blob/b1908e74259da56a92800cace97350af1f1fc32b/201-list-storage-keys-windows-vm/azuredeploy.json/)

<!---HONumber=Mooncake_0905_2016-->