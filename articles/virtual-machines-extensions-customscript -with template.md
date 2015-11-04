<properties
   pageTitle="在 VM 上使用模板自定义脚本 | Windows Azure"
   description="通过将自定义脚本扩展与资源管理器模板配合使用，自动执行 Windows 和 Linux 的 Azure VM 配置任务"
   services="virtual-machines"
   documentationCenter=""
   authors="kundanap"
   manager="timlt"
   editor=""
   tags="azure-resource-manager"/>

<tags
   ms.service="virtual-machines"
   ms.date="07/01/2015"
   wacn.date="11/02/2015"/>

# 将自定义脚本扩展与 Azure 资源管理器模板配合使用

本文概述如何使用自定义脚本扩展编写 Azure 资源管理器模板，用于引导 Linux 或 Windows VM 上的工作负荷。

有关自定义脚本扩展的概述，请参阅[此文](/documentation/articles/virtual-machines-extensions-customscript)。

[AZURE.INCLUDE [了解部署模型](../includes/learn-about-deployment-models-include.md)]本文介绍如何使用资源管理器部署模型创建资源。你还可以使用[经典部署模型](/documentation/articles/virtual-machines-extensions-customscript)创建资源。

自定义脚本扩展自推出后已广泛应用于在 Windows 和 Linux VM 上配置工作负荷。随着 Azure 资源管理器模板的引入，用户现在可以创建单个模板，该模板不只可以预配 VM，还可以配置 VM 上的工作负荷。

## Azure 资源管理器模板概述

Azure 资源管理器模板可让你通过定义资源之间的依赖关系，使用 JSON 语言以声明方式指定 Azure IaaS 基础结构。有关 Azure 资源管理器模板的详细概述，请参阅以下文章：

<a href="https://www.windowsazure.cn/documentation/articles/resource-group-overview/" target="_blank">资源组概述</a>。<br/><a href="https://www.windowsazure.cn/documentation/articles/virtual-machines-deploy-rmtemplates-azure-cli/" target="_blank">使用 Azure CLI 部署模板</a>。<br/><a href="https://www.windowsazure.cn/documentation/articles/virtual-machines-deploy-rmtemplates-powershell/" target="_blank">使用 Azure Powershell 部署模板。</a>

### 运行自定义脚本扩展的先决条件

1. 从<a href="https://www.windowsazure.cn/downloads" target="_blank">此处</a>安装最新的 Azure PowerShell Cmdlet 或 Azure CLI。
2. 如果脚本将在现有 VM 上运行，请确保已在该 VM 上启用了 VM 代理，如果没有启用，请根据<a href="https://msdn.microsoft.com/zh-cn/library/azure/dn832621.aspx" target="_blank">此文</a>来安装一个。
3. 将你要在 VM 上运行的脚本上载到 Azure 存储。脚本可以来自单个或多个存储容器。
4. 或者，也可以将脚本上载到 Github 帐户。
5. 脚本应当以下述方式编写：使用扩展启动入口脚本，然后入口脚本再调用其他脚本。

## 将自定义脚本扩展与模板配合使用的概述：

若要使用模板部署，我们需要使用 Azure 服务管理 API 中可用的同一版本的自定义脚本扩展。该扩展支持用于将文件上载到 Azure 存储帐户或 Github 位置的相同参数和方案。配合模板使用时，主要差异在于应该指定正确版本的扩展，而不是以 majorversion.* 格式指定版本。

## 用于 Linux VM 上自定义脚本扩展的模板代码段

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
      "fileUris": [ "https: //raw.githubusercontent.com/Azure/azure-quickstart-templates/master/mongodb-on-ubuntu/mongo-install-ubuntu.sh                        ],
      "commandToExecute": "shmongo-install-ubuntu.sh"
      }
    }
    }

## 用于 Windows VM 上自定义脚本扩展的模板代码段

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

创作模板之后，可以使用 Azure CLI 或 Azure Powershell 部署模板。

有关在 VM 上使用自定义脚本扩展配置应用程序的完整示例，请参阅以下示例。

<a href="https://github.com/Azure/azure-quickstart-templates/blob/b1908e74259da56a92800cace97350af1f1fc32b/mongodb-on-ubuntu/azuredeploy.json/" target="_blank">Linux VM 上的自定义脚本扩展</a>。
</br>
<a href="https://github.com/Azure/azure-quickstart-templates/blob/b1908e74259da56a92800cace97350af1f1fc32b/201-list-storage-keys-windows-vm/azuredeploy.json/" target="_blank">Windows VM 上的自定义脚本扩展</a>。

<!---HONumber=76-->