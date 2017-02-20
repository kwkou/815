<properties
    pageTitle="Windows VM 上的自定义脚本扩展 | Azure"
    description="通过使用自定义脚本扩展在远程 Windows VM 上运行 PowerShell 脚本自动执行 Azure VM 配置任务"
    services="virtual-machines-windows"
    documentationcenter=""
    author="neilpeterson"
    manager="timlt"
    editor=""
    tags="azure-service-management" />
<tags
    ms.assetid="ebb7340a-8f61-4d3c-a290-d7bf8de2d0bd"
    ms.service="virtual-machines-windows"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="vm-windows"
    ms.workload="infrastructure-services"
    ms.date="01/17/2017"
    wacn.date="02/20/2017"
    ms.author="nepeters" />  


# 使用经典部署模型完成的适用于 Windows 的自定义脚本扩展

> [AZURE.IMPORTANT] 
Azure 提供两个不同的部署模型用于创建和处理资源：[Resource Manager 模型和经典模型](/documentation/articles/resource-manager-deployment-model/)。本文介绍如何使用经典部署模型。Azure 建议大多数新部署使用 Resource Manager 模型。了解如何[使用 Resource Manager 模型执行这些步骤](/documentation/articles/virtual-machines-windows-extensions-customscript/)。

自定义脚本扩展在 Azure 虚拟机上下载并执行脚本。此扩展适用于部署后配置、软件安装或其他任何配置/管理任务。可以从 Azure 存储或 GitHub 下载脚本，也可以在扩展运行时将脚本提供给 Azure 门户预览。自定义脚本扩展与 Azure Resource Manager 模板集成，也可以使用 Azure CLI、PowerShell、Azure 门户预览或 Azure 虚拟机 REST API 来运行它。

本文档详细说明了如何通过 Azure PowerShell 模块和 Azure Resource Manager 模板使用自定义脚本扩展，同时详细说明了 Windows 系统上的故障排除步骤。

## 先决条件

### 操作系统

可以针对 Windows Server 2008 R2、2012、2012 R2 和 2016 版本运行适用于 Windows 的自定义脚本扩展。

### 脚本位置

该脚本需存储在 Azure 存储中，或者存储在可以通过有效 URL 访问的任何其他位置。

### Internet 连接

适用于 Windows 的自定义脚本扩展要求目标虚拟机已连接到 Internet。

## 扩展架构

以下 JSON 显示自定义脚本扩展的架构。扩展需要脚本位置（Azure 存储或其他具有有效 URL 的位置）以及命令才能执行。如果使用 Azure 存储作为脚本源，则需 Azure 存储帐户名称和帐户密钥。这些项目应视为敏感数据，并在扩展的受保护设置配置中指定。Azure VM 扩展的受保护设置数据已加密，并且只能在目标虚拟机上解密。

    {
        "name": "config-app",
        "type": "Microsoft.ClassicCompute/virtualMachines/extensions",
        "location": "[resourceGroup().location]",
        "apiVersion": "2015-06-01",
        "properties": {
            "publisher": "Microsoft.Compute",
            "extension": "CustomScriptExtension",
            "version": "1.8",
            "parameters": {
                "public": {
                    "fileUris": "[myScriptLocation]"
                },
                "private": {
                    "commandToExecute": "[myExecutionString]"
                }
            }
        }
    }

### 属性值

| 名称 | 值/示例 |
| ---- | ---- |
| apiVersion | 2015-06-15 |
| publisher | Microsoft.Compute |
| 扩展 | CustomScriptExtension |
| typeHandlerVersion | 1\.8 |
| fileUris（示例） | https://raw.githubusercontent.com/Microsoft/dotnet-core-sample-templates/master/dotnet-core-music-windows/scripts/configure-music-app.ps1  
 |
| commandToExecute（示例） | powershell -ExecutionPolicy Unrestricted -File configure-music-app.ps1 |

## 模板部署

可使用 Azure Resource Manager 模板部署 Azure VM 扩展。可以将上一部分详述的 JSON 架构用在 Azure Resource Manager 模板中，以便在 Azure Resource Manager 模板部署期间运行自定义脚本扩展。若需包含自定义脚本扩展的示例模板，可访问 [GitHub](https://github.com/Microsoft/dotnet-core-sample-templates/tree/master/dotnet-core-music-windows)。

## PowerShell 部署

可以使用 `Set-AzureVMCustomScriptExtension` 命令将自定义脚本扩展添加到现有的虚拟机。有关详细信息，请参阅 [Set-AzureRmVMCustomScriptExtension ](https://docs.microsoft.com/powershell/resourcemanager/azurerm.compute/v2.1.0/set-azurermvmcustomscriptextension)。

    # create vm object
    $vm = Get-AzureVM -Name 2016clas -ServiceName 2016clas1313

    # set extension
    Set-AzureVMCustomScriptExtension -VM $vm -FileUri myFileUri -Run 'create-file.ps1'

    # update vm
    $vm | Update-AzureVM

## 故障排除和支持

### 故障排除

有关扩展部署状态的数据可以通过 Azure 门户预览和 Azure PowerShell 模块进行检索。若要查看给定 VM 的扩展部署状态，请运行以下命令。

    Get-AzureVMExtension -ResourceGroupName myResourceGroup -VMName myVM -Name myExtensionName

扩展执行输出将记录到在目标虚拟机的以下目录中发现的文件。

    C:\WindowsAzure\Logs\Plugins\Microsoft.Compute.CustomScriptExtension

脚本本身下载到目标虚拟机的以下目录中。

    C:\Packages\Plugins\Microsoft.Compute.CustomScriptExtension\1.*\Downloads

### 支持

如果对本文中的任何观点存在疑问，可以联系 [MSDN Azure 和 CSDN Azure](/support/forums/) 上的 Azure 专家。或者，也可以提交 Azure 支持事件。请转到 [Azure 支持站点](/support/contact/)并选择“获取支持”。有关使用 Azure 支持的信息，请阅读 [Azure 支持常见问题](/support/faq/)。

<!---HONumber=Mooncake_0213_2017-->