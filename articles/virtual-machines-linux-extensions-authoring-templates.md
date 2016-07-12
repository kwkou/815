<!-- ARM: tested -->

<properties
   pageTitle="使用 Linux VM 扩展创作模板 | Azure"
   description="详细了解如何为 Linux VM 使用扩展创建 Azure 资源管理器模板"
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

# 使用 VM 扩展创作 Linux 资源管理器模板。

[AZURE.INCLUDE [arm-api-version-cli](../includes/arm-api-version-cli.md)]

> [AZURE.NOTE]Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model/)。这篇文章介绍如何使用资源管理器部署模型，Azure 建议大多数新部署使用资源管理器模型替代经典部署模型。

[AZURE.INCLUDE [virtual-machines-common-extensions-authoring-templates](../includes/virtual-machines-common-extensions-authoring-templates.md)]

从 Azure CLI，运行以下命令：

      Azure VM Extension list

此命令会返回发布者名称、扩展名称和版本，如下所示：

       Publisher                   : Microsoft.Azure.Extensions  
      ExtensionName               : DockerExtension
      Version                     : 1.0

这三个属性分别映射到上述模板代码段中的“发布者”、“类型”和“typeHandlerVersion”。

>[AZURE.NOTE] 始终建议使用最新的扩展版本以获取最新功能。

## 确定扩展配置参数的架构

创作扩展模板的下一个步骤是确定提供配置参数的格式。每个扩展均支持其自己的参数集。

若要查看 Linux 扩展的示例配置，请单击文档 [Linux 扩展示例](/documentation/articles/virtual-machines-linux-extensions-configuration-samples/)。

请参阅以下 VM 模板链接，以获取完整的 VM 扩展模板。

[Linux VM 上的自定义脚本扩展](https://github.com/Azure/azure-quickstart-templates/blob/b1908e74259da56a92800cace97350af1f1fc32b/mongodb-on-ubuntu/azuredeploy.json/)

创作模板之后，可以使用 Azure CLI 部署模板。

<!---HONumber=79-->