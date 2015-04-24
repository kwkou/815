<properties
   pageTitle="Windows 上的自定义脚本扩展"
   description="在 Windows 上使用自定义脚本扩展自动执行 Azure 虚拟机配置任务 "
   services="virtual-machines"
   documentationCenter=""
   authors="kundanap"
   manager="madhana"
   editor=""/>
<tags ms.service="virtual-machines"
   ms.date="02/19/2015"
   wacn.date="04/15/2015"/>


# 适用于 Windows 的自定义脚本扩展

本文概述了如何在 Windows 上使用 Azure Powershell cmdlet 来使用自定义脚本扩展。


虚拟机 (VM) 扩展由 Microsoft 和受信任的第三方发布者构建，用于扩展 VM 的功能。有关 VM 扩展的详细概述，请参阅 <a href="https://msdn.microsoft.com/zh-cn/library/azure/dn606311.aspx" target="_blank">MSDN 文档</a>。

## 自定义脚本扩展概述

适用于 Windows 的自定义脚本扩展允许你在不登录到远程虚拟机的情况下在其上执行 Powershell 脚本。可以在设置 VM 后或者在 VM 生命周期中的任何时间执行这些脚本，不需要在 VM 上打开额外的端口。自定义脚本的最常见用例包括在进行设置后在 VM 上执行其他软件的安装和配置。

### 运行自定义脚本扩展的先决条件

1. 从<a href="/downloads" target="_blank">此处</a>安装 Azure PowerShell Cmdlets V0.8.0 或更高版本。
2. 如果脚本将在现有 VM 上运行，请确保已在该 VM 上启用了 VM 代理，如果没有启用，请根据本<a href="https://msdn.microsoft.com/zh-cn/library/azure/dn832621.aspx" target="_blank">文章</a>来安装一个。
3. 将你要在 VM 上运行的脚本上载到 Azure 存储。脚本可以来自单个或多个存储容器。
4. 脚本应当以下述方式编写：使用扩展启动入口脚本，然后入口脚本再调用其他脚本。

## 自定义脚本扩展方案：

### 将文件上载到默认容器：
如果你的脚本位于你的订阅的默认帐户的存储容器中，则下面的 cmdlet 代码片段展示了如何在 VM 上运行它们。下面的示例中的 ContainerName 是你将脚本上载到的位置。可以使用 cmdlet"Get-AzureSubscription -Default"验证默认存储帐户。

注意：此用例创建一个新的 VM，但也可以在现有的 VM 上执行同样的操作。

    # create a new VM in Azure.
    $vm = New-AzureVMConfig -Name $name -InstanceSize Small -ImageName $imagename
    $vm = Add-AzureProvisioningConfig -VM $vm -Windows -AdminUsername $username -Password $password
    #Add Custom Script Extension to the VM. The container name refer to the storage container which contains the file.
    $vm = Set-AzureVMCustomScriptExtension -VM $vm -ContainerName $container -FileName 'start.ps1'
    New-AzureVM -ServiceName $servicename -Location $location -VMs $vm
    #  After the VM is created, the extension downloads the script from the storage location and executes it on the VM.

    # Viewing the  script execution output.
    $vm = Get-AzureVM -ServiceName $servicename -Name $name
    # Use the position of the extension in the output as index.
    $vm.ResourceExtensionStatusList[i].ExtensionSettingStatus.SubStatusList

### 将文件上载到非默认存储容器。

此用例展示了如何使用同一订阅中的或不同订阅中的非默认存储来上载脚本/文件。这次，我们将使用现有 VM，但可以在创建新 VM 时执行同样的操作。

        Get-AzureVM -Name $name -ServiceName $servicename | Set-AzureVMCustomScriptExtension -StorageAccountName $storageaccount -StorageAccountKey $storagekey -ContainerName $container -FileName 'file1.ps1','file2.ps1' -Run 'file.ps1' | Update-AzureVM
### 将脚本上载到不同存储帐户中的多个容器中。
  如果脚本文件存储在多个容器中，则当前要运行这些文件，你必须提供这些文件的完整 SAS URL。

      Get-AzureVM -Name $name -ServiceName $servicename | Set-AzureVMCustomScriptExtension -StorageAccountName $storageaccount -StorageAccountKey $storagekey -ContainerName $container -FileUri $fileUrl1, $fileUrl2 -Run 'file.ps1' | Update-AzureVM


### 从门户中添加自定义脚本扩展。
在 <a href="https://management.windowsazure.cn/ " target="_blank">Azure 门户</a>中浏览找到虚拟机，并通过指定要运行的脚本文件添加扩展。
  ![][5]

### 卸载自定义脚本扩展。

可以使用下面的 cmdlet 从 VM 中卸载自定义脚本扩展

      get-azureVM -ServiceName KPTRDemo -Name KPTRDemo | Set-AzureVMCustomScriptExtension -Uninstall | Update-AzureVM

### 即将推出

我们很快将添加适用于 Linux 的自定义脚本的用法和示例，敬请关注。

<!--Image references-->
[5]: ./media/virtual-machines-extensions-customscript/addcse.png

<!--HONumber=50-->