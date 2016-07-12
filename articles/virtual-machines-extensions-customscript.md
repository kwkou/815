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
   ms.date="08/06/2015"
   wacn.date="11/12/2015"/>

# 适用于 Windows 虚拟机的自定义脚本扩展

本文概述了如何在 Windows VM 上通过 Azure Powershell cmdlet 来使用自定义脚本扩展。

虚拟机 (VM) 扩展由 Microsoft 和受信任的第三方发布者构建，用于扩展 VM 的功能。有关 VM 扩展的概述，请参阅 [Azure VM 扩展和功能](/documentation/articles/virtual-machines-windows-extensions-features/)。

> [AZURE.IMPORTANT]Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model/)。本文介绍使用经典部署模型。Azure 建议大多数新部署使用[资源管理器模型](/documentation/articles/virtual-machines-windows-classic-extensions-customscript/)。

## 自定义脚本扩展概述

适用于 Windows 的自定义脚本扩展允许你在不登录到远程 VM 的情况下在其上运行 Powershell 脚本。可以在预配 VM 后或者在 VM 的生命周期中的任何时间运行这些脚本，而不需要在 VM 上打开任何其他端口。自定义脚本扩展的最常见用例包括在预配 VM 后在 VM 上运行、安装和配置其他软件。

### 运行自定义脚本扩展的先决条件

1. 从<a href="/downloads" target="_blank">此处</a>安装 Azure PowerShell Cmdlet 版本 0.8.0 或更高版本。
2. 如果脚本将在现有 VM 上运行，请确保已在该 VM 上启用了 VM 代理。
3. 将你要在 VM 上运行的脚本上载到 Azure 存储。脚本可以来自单个容器或多个存储容器。
4. 脚本应当以下述方式编写：使用扩展启动入口脚本，然后入口脚本再启动其他脚本。

## 自定义脚本扩展方案

### 将文件上载到默认容器

如果你的脚本位于你的订阅的默认帐户的存储容器中，则下面的示例演示如何在 VM 上运行它们。ContainerName 是你将脚本上载到的位置。可以使用 **Get-AzureSubscription –Default** 命令验证默认存储帐户。

下面的示例将创建一个新 VM，但同一方案也可以在现有 VM 上运行。

    # create a new VM in Azure.
    $vm = New-AzureVMConfig -Name $name -InstanceSize Small -ImageName $imagename
    $vm = Add-AzureProvisioningConfig -VM $vm -Windows -AdminUsername $username -Password $password
    // Add Custom Script Extension to the VM. The container name refer to the storage container which contains the file.
    $vm = Set-AzureVMCustomScriptExtension -VM $vm -ContainerName $container -FileName 'start.ps1'
    New-AzureVM -ServiceName $servicename -Location $location -VMs $vm
    #  After the VM is created, the extension downloads the script from the storage location and executes it on the VM.

    # Viewing the  script execution output.
    $vm = Get-AzureVM -ServiceName $servicename -Name $name
    # Use the position of the extension in the output as index.
    $vm.ResourceExtensionStatusList[i].ExtensionSettingStatus.SubStatusList

### 将文件上载到非默认存储容器

此方案说明如何使用同一订阅中的或不同订阅中的非默认存储来上载脚本和文件。这次，我们将使用现有 VM，但可以在创建新 VM 时执行同样的操作。

        Get-AzureVM -Name $name -ServiceName $servicename | Set-AzureVMCustomScriptExtension -StorageAccountName $storageaccount -StorageAccountKey $storagekey -ContainerName $container -FileName 'file1.ps1','file2.ps1' -Run 'file.ps1' | Update-AzureVM

### 将脚本上载到不同存储帐户中的多个容器

  如果脚本文件存储在多个容器中，则要运行脚本，你必须提供这些文件的完整 SAS URL。

      Get-AzureVM -Name $name -ServiceName $servicename | Set-AzureVMCustomScriptExtension -StorageAccountName $storageaccount -StorageAccountKey $storagekey -ContainerName $container -FileUri $fileUrl1, $fileUrl2 -Run 'file.ps1' | Update-AzureVM

### 卸载自定义脚本扩展

可以使用以下命令从 VM 中卸载自定义脚本扩展。

      get-azureVM -ServiceName KPTRDemo -Name KPTRDemo | Set-AzureVMCustomScriptExtension -Uninstall | Update-AzureVM


<!--Image references-->
[5]: ./media/virtual-machines-windows-classic-extensions-customscript/addcse.png

<!---HONumber=79-->