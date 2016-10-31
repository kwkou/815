<properties
   pageTitle="Windows VM 上的自定义脚本扩展 | Azure"
   description="通过使用自定义脚本扩展在远程 Windows VM 上运行 PowerShell 脚本自动执行 Azure VM 配置任务"
   services="virtual-machines-windows"
   documentationCenter=""
   authors="kundanap"
   manager="timlt"
   editor=""
   tags="azure-service-management"/>  


<tags
   ms.service="virtual-machines-windows"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="vm-windows"
   ms.workload="infrastructure-services"
   ms.date="08/06/2015"
   wacn.date="10/24/2016"
   ms.author="kundanap"/>

# 适用于 Windows 虚拟机的自定义脚本扩展

本文概述了如何在 Windows VM 上通过 Azure PowerShell cmdlet 和 Azure 服务管理 API 使用自定义脚本扩展。

虚拟机 (VM) 扩展由 Microsoft 和受信任的第三方发布者构建，用于扩展 VM 的功能。有关 VM 扩展的概述，请参阅 [Azure VM 扩展和功能](/documentation/articles/virtual-machines-windows-extensions-features/)。

> [AZURE.IMPORTANT] Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model/)。本文介绍使用经典部署模型。Azure 建议大多数新部署使用 Resource Manager 模型。了解如何[使用 Resource Manager 模型执行这些步骤](/documentation/articles/virtual-machines-windows-extensions-customscript/)。

## 自定义脚本扩展概述

使用 Windows 的自定义脚本扩展，用户可以在不登录远程 VM 的情况下在其上运行 PowerShell 脚本。可以在预配 VM 后或者在 VM 生命周期中的任何时间运行这些脚本，不需要打开任何其他端口。运行自定义脚本扩展时，最常见用例包括在预配 VM 后在 VM 上运行、安装和配置其他软件。

### 运行自定义脚本扩展的先决条件

1. 安装 <a href="/downloads" target="_blank">Azure PowerShell cmdlet</a> 0.8.0 或更高版本。
2. 如果希望脚本在现有 VM 上运行，请确保已在该 VM 上启用 VM 代理。如果尚未安装，请执行以下[步骤](/documentation/articles/virtual-machines-windows-classic-agents-and-extensions/)。如果 VM 是从 Azure 门户预览创建的，则会默认安装 VM 代理。
3. 将你要在 VM 上运行的脚本上载到 Azure 存储。脚本可以来自单个容器或多个存储容器。
4. 脚本应当以下述方式编写：使用扩展启动入口脚本，然后入口脚本再启动其他脚本。

## 自定义脚本扩展方案

### 将文件上载到默认容器

下面的示例演示如何在 VM 上运行脚本，前提是脚本位于订阅的默认帐户的存储容器中。可将脚本上载到 ContainerName。可以使用 **Get-AzureSubscription –Default** 命令验证默认存储帐户。

下面的示例将创建一个 VM，但同一方案也可在现有 VM 上运行。

    # Create a new VM in Azure.
    $vm = New-AzureVMConfig -Name $name -InstanceSize Small -ImageName $imagename
    $vm = Add-AzureProvisioningConfig -VM $vm -Windows -AdminUsername $username -Password $password
    // Add Custom Script extension to the VM. The container name refers to the storage container that contains the file.
    $vm = Set-AzureVMCustomScriptExtension -VM $vm -ContainerName $container -FileName 'start.ps1'
    New-AzureVM -ServiceName $servicename -Location $location -VMs $vm
    #  After the VM is created, the extension downloads the script from the storage location and executes it on the VM.

    # Viewing the  script execution output.
    $vm = Get-AzureVM -ServiceName $servicename -Name $name
    # Use the position of the extension in the output as index.
    $vm.ResourceExtensionStatusList[i].ExtensionSettingStatus.SubStatusList

### 将文件上载到非默认存储容器

此方案说明如何使用同一订阅中的或不同订阅中的非默认存储容器来上载脚本和文件。此示例演示的是现有 VM，但创建 VM 时也可执行同样的操作。

        Get-AzureVM -Name $name -ServiceName $servicename | Set-AzureVMCustomScriptExtension -StorageAccountName $storageaccount -StorageAccountKey $storagekey -ContainerName $container -FileName 'file1.ps1','file2.ps1' -Run 'file.ps1' | Update-AzureVM

### 将脚本上载到不同存储帐户中的多个容器

  如果脚本文件存储在多个容器中，则要运行脚本，必须提供这些文件的完整共享访问签名 (SAS) URL。

      Get-AzureVM -Name $name -ServiceName $servicename | Set-AzureVMCustomScriptExtension -StorageAccountName $storageaccount -StorageAccountKey $storagekey -ContainerName $container -FileUri $fileUrl1, $fileUrl2 -Run 'file.ps1' | Update-AzureVM


### 从 Azure 门户预览添加自定义脚本扩展

在 <a href="https://portal.azure.cn/ " target="_blank">Azure 门户预览</a>中转到 VM，并通过指定要运行的脚本文件添加扩展。

  ![指定脚本文件][5]  



### 卸载自定义脚本扩展

可以使用以下命令从 VM 中卸载自定义脚本扩展。

      get-azureVM -ServiceName KPTRDemo -Name KPTRDemo | Set-AzureVMCustomScriptExtension -Uninstall | Update-AzureVM

### 将自定义脚本扩展与模板配合使用

若要了解如何将自定义脚本扩展与 Azure Resource Manager 模板配合使用，请参阅 [Using the Custom Script extension for Windows VMs with Azure Resource Manager templates](/documentation/articles/virtual-machines-windows-extensions-customscript/)（将 Windows VM 的自定义脚本扩展与 Azure Resource Manager 模板配合使用）。

<!--Image references-->

[5]: ./media/virtual-machines-windows-classic-extensions-customscript/addcse.png

<!---HONumber=Mooncake_1017_2016-->