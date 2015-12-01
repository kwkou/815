<properties
   pageTitle="使用 Azure VM 扩展创作模板 | Windows Azure"
   description="详细了解如何使用扩展创建模板"
   services="virtual-machines"
   documentationCenter=""
   authors="kundanap"
   manager="timlt"
   editor=""
   tags="azure-resource-manager"/>

<tags
   ms.service="virtual-machines"
   ms.date="09/01/2015"
   wacn.date="11/12/2015"/>

# 使用 VM 扩展创作 Azure 资源管理器模板。

[AZURE.INCLUDE [了解部署模型](../includes/learn-about-deployment-models-include.md)]本文介绍如何使用资源管理器部署模型。

## Azure 资源管理器模板概述。

Azure 资源管理器模板可让你通过定义资源之间的依赖关系，使用 JSON 语言以声明方式指定 Azure IaaS 基础结构。有关 Azure 资源管理器模板的详细概述，请参阅以下文章：

[资源组概述](/documentation/articles/resource-group-overview)

## VM 扩展的示例模板代码段。
将 VM 扩展部署为 Azure 资源管理器模板的一部分要求你在模板中以声明方式指定扩展配置。以下是指定扩展配置的格式。

      {
      "type": "Microsoft.Compute/virtualMachines/extensions",
      "name": "MyExtension",
      "apiVersion": "2015-05-01-preview",
      "location": "[parameters('location')]",
      "dependsOn": ["[concat('Microsoft.Compute/virtualMachines/',parameters('vmName'))]"],
      "properties":
      {
      "publisher": "Publisher Namespace",
      "type": "extension Name",
      "typeHandlerVersion": "extension version",
      "settings": {
      // Extension specific configuration goes in here.
      }
      }
      }

如上所示，扩展模板包含两个主要部分：

1. 扩展名称、发布者和版本。
2. 扩展配置。

## 确定任何扩展的发布者、类型和 typeHandlerVersion。

Azure VM 扩展由 Microsoft 和受信任的第三方发布者发布，每个扩展都通过其发布者、类型和 typeHandlerVersion 进行唯一标识。可通过以下方式确定这些内容：

从 Azure PowerShell，运行以下 Azure Powershell cmdlet：

      Get-AzureVMAvailableExtension

从 Azure CLI，运行以下 Azure Powershell cmdlet：

      Azure VM Extension list

此 cmdlet 会返回发布者名称、扩展名称和版本，如下所示：

       Publisher                   : Microsoft.Azure.Extensions  
      ExtensionName               : DockerExtension
      Version                     : 1.0

这三个属性分别映射到上述模板代码段中的“发布者”、“类型”和“typeHandlerVersion”。注意：始终建议使用最新的扩展版本以获取最新功能。

## 确定扩展配置参数的架构

创作扩展模板的下一个步骤是确定提供配置参数的格式。每个扩展均支持其自己的参数集。

若要查看 Windows 扩展的示例配置，请单击文档 [Windows 扩展示例](/documentation/articles/virtual-machines-extensions-configuration-samples-windows)。

若要查看 Linux 扩展的示例配置，请单击文档 [Linux 扩展示例](/documentation/articles/virtual-machines-extensions-configuration-samples-linux)。

请参阅以下 VM 模板链接，以获取完整的 VM 扩展模板。

[Windows VM 上的自定义脚本扩展](https://github.com/Azure/azure-quickstart-templates/blob/b1908e74259da56a92800cace97350af1f1fc32b/201-list-storage-keys-windows-vm/azuredeploy.json/)

[Linux VM 上的自定义脚本扩展](https://github.com/Azure/azure-quickstart-templates/blob/b1908e74259da56a92800cace97350af1f1fc32b/mongodb-on-ubuntu/azuredeploy.json/)

创作模板之后，可以使用 Azure CLI 或 Azure Powershell 部署模板。

<!---HONumber=79-->