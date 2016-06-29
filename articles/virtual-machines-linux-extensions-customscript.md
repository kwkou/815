<!-- ARM: tested -->

<properties
   pageTitle="在 Linux VM 上使用模板自定义脚本 | Microsoft Azure"
   description="通过将自定义脚本扩展与资源管理器模板配合使用，自动执行 Linux 的 Azure VM 配置任务"
   services="virtual-machines-linux"
   documentationCenter=""
   authors="kundanap"
   manager="timlt"
   editor=""
   tags="azure-resource-manager"/>

<tags
   ms.service="virtual-machines-linux"
   ms.date="03/29/2016"
   wacn.date="06/29/2016"/>

# 将 Linux VM 的自定义脚本扩展与 Azure 资源管理器模板配合使用

>[AZURE.NOTE] 你从 GitHub 仓库 "azure-quickstart-templates" 中下载的模板，需要做一些修改才能适用于 Azure 中国云环境。例如，替换一些终结点 -- "blob.core.windows.net" 替换成 "blob.core.chinacloudapi.cn"，"cloudapp.azure.com" 替换成 "chinacloudapp.cn"；改掉一些不支持的 VM 映像，还有，改掉一些不支持的 VM 大小。

本文概述如何使用自定义脚本扩展编写 Azure 资源管理器模板，用于引导 Linux 或 Windows VM 上的工作负荷。

[AZURE.INCLUDE [virtual-machines-common-extensions-customscript](../includes/virtual-machines-common-extensions-customscript.md)]

## Linux VM 模板示例

在模板的 Resource 节中定义以下扩展资源

      {
    "type": "Microsoft.Compute/virtualMachines/extensions",
    "name": "MyCustomScriptExtension",
    "apiVersion": "2015-05-01-preview",
    "location": "[parameters('location')]",
    "dependsOn": ["[concat('Microsoft.Compute/virtualMachines/',parameters('vmName'))]"],
    "properties":
    {
      "publisher": "Microsoft.OSTCExtensions",
      "type": "CustomScriptForLinux",
      "typeHandlerVersion": "1.2",
      "settings": {
      "fileUris": [ "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/mongodb-on-ubuntu/mongo-install-ubuntu.sh"],
      "commandToExecute": "sh mongo-install-ubuntu.sh"
      }
    }
    }

在上述示例中，请将文件 URL 和文件名替换为你自己的设置。

创作模板之后，可以使用 Azure CLI 部署模板。

有关在 VM 上使用自定义脚本扩展配置应用程序的完整示例，请参阅以下示例。

[Linux VM 上的自定义脚本扩展](https://github.com/Azure/azure-quickstart-templates/blob/b1908e74259da56a92800cace97350af1f1fc32b/mongodb-on-ubuntu/azuredeploy.json/)

<!---HONumber=Mooncake_0118_2016-->