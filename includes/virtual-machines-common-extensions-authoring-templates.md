
## Azure 资源管理器模板概述。

>[AZURE.NOTE] 你从 GitHub 仓库 "azure-quickstart-templates" 中下载的模板，需要做一些修改才能适用于 Azure 中国云环境。例如，替换一些终结点 -- "blob.core.windows.net" 替换成 "blob.core.chinacloudapi.cn"，"cloudapp.azure.com" 替换成 "chinacloudapp.cn"；改掉一些不支持的 VM 映像，还有，改掉一些不支持的 VM 大小。

Azure 资源管理器模板可让你通过定义资源之间的依赖关系，使用 JSON 语言以声明方式指定 Azure IaaS 基础结构。有关 Azure 资源管理器模板的详细概述，请参阅以下文章：

[资源组概述](/documentation/articles/resource-group-overview/)

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
