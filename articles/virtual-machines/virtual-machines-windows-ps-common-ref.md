<properties
    pageTitle="适用于 Azure 虚拟机的常用 PowerShell 命令 | Azure"
    description="帮助你开始在 Azure 中创建和管理 Windows VM 的常用 PowerShell 命令"
    services="virtual-machines-windows"
    documentationcenter=""
    author="davidmu1"
    manager="timlt"
    editor="tysonn"
    tags="azure-resource-manager"
    translationtype="Human Translation" />
<tags
    ms.assetid="ba3839a2-f3d5-4e19-a5de-95bfb1c0e61e"
    ms.service="virtual-machines-windows"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="vm-windows"
    ms.workload="infrastructure-services"
    ms.date="03/02/2017"
    wacn.date="04/17/2017"
    ms.author="davidmu"
    ms.sourcegitcommit="e0e6e13098e42358a7eaf3a810930af750e724dd"
    ms.openlocfilehash="1546a528c8664fc7fce8d3b6bf6e014f09a198d7"
    ms.lasthandoff="04/06/2017" />

# <a name="common-powershell-commands-for-creating-and-managing-azure-virtual-machines"></a>用于创建和管理 Azure 虚拟机的常用 PowerShell 命令

本文介绍一些可用于在 Azure 订阅中创建和管理虚拟机的 Azure PowerShell 命令。  若要获取特定命令行开关和选项的详细帮助，可以使用 **Get-Help** *命令*。

有关安装最新版 Azure PowerShell、选择订阅和登录到帐户的信息，请参阅[如何安装和配置 Azure PowerShell](https://docs.microsoft.com/zh-cn/powershell/azureps-cmdlets-docs)。

如果在本文运行多个命令，以下变量可能对你有用：

- $location - 虚拟机的位置。 可以使用 [Get-AzureRmLocation](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.resources/v3.5.0/get-azurermlocation) 查找适合你的地理区域。
- $myResourceGroup - 包含虚拟机的资源组的名称。
- $myVM - 虚拟机的名称。

## <a name="create-a-vm"></a>创建 VM

| 任务 | 命令 |
| ---- | ------- |
| 创建 VM 配置 |$vm = [New-AzureRmVMConfig](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.compute/v2.5.0/New-AzureRmVMConfig) -VMName $myVM -VMSize "Standard_D1_v1"<BR></BR><BR></BR>VM 配置用于定义或更新 VM 的设置。 使用 VM 的名称及其[大小](/documentation/articles/virtual-machines-windows-sizes/)对配置进行初始化。 |
| 添加配置设置 |$vm = [Set-AzureRmVMOperatingSystem](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.compute/v2.5.0/Set-AzureRmVMOperatingSystem) -VM $vm -Windows -ComputerName $myVM -Credential $cred -ProvisionVMAgent -EnableAutoUpdate<BR></BR><BR></BR>包括[凭据](https://technet.microsoft.com/zh-cn/library/hh849815.aspx)的操作系统设置将添加到以前使用 New-AzureRmVMConfig 创建的配置对象。 |
| 添加网络接口 |$vm = [Add-AzureRmVMNetworkInterface](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.compute/v2.5.0/Add-AzureRmVMNetworkInterface) -VM $vm -Id $nic.Id<BR></BR><BR></BR>VM 必须使用[网络接口](/documentation/articles/virtual-machines-windows-quick-create-powershell/)在虚拟网络中通信。 还可使用 [Get-AzureRmNetworkInterface](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/AzureRM.Network/v3.4.0/Get-AzureRmNetworkInterface) 检索现有网络接口对象。 |
| 指定平台映像 |$vm = [Set-AzureRmVMSourceImage](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.compute/v2.5.0/Set-AzureRmVMSourceImage) -VM $vm -PublisherName "publisher_name" -Offer "publisher_offer" -Skus "product_sku" -Version "latest"<BR></BR><BR></BR>[映像信息](/documentation/articles/virtual-machines-windows-cli-ps-findimage/)将添加到以前使用 New-AzureRmVMConfig 创建的配置对象。 仅当将操作系统磁盘设置为使用平台映像时，才使用此命令返回的对象。 |
| 设置要使用平台映像的操作系统磁盘 |$vm = [Set-AzureRmVMOSDisk](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.compute/v2.5.0/Set-AzureRmVMOSDisk) -VM $vm -Name "myOSDisk" -VhdUri "http://mystore1.blob.core.chinacloudapi.cn/vhds/myOSDisk.vhd" -CreateOption FromImage<BR></BR><BR></BR>操作系统磁盘的名称以及它在[存储](/documentation/articles/storage-powershell-guide-full/)中的位置将添加到以前创建的配置对象。 |
| 设置要使用一般化映像的操作系统磁盘 |$vm = Set-AzureRmVMOSDisk -VM $vm -Name "myOSDisk" -SourceImageUri "https://mystore1.blob.core.chinacloudapi.cn/system/Microsoft.Compute/Images/myimages/myprefix-osDisk.{guid}.vhd" -VhdUri "https://mystore1.blob.core.chinacloudapi.cn/vhds/disk_name.vhd" -CreateOption FromImage -Windows<BR></BR><BR></BR>操作系统磁盘的名称、源映像的位置，以及磁盘在[存储](/documentation/articles/storage-powershell-guide-full/)中的位置将添加到以前创建的配置对象。 |
| 设置要使用特殊化映像的操作系统磁盘 |$vm = Set-AzureRmVMOSDisk -VM $vm -Name "myOSDisk" -VhdUri "http://mystore1.blob.core.chinacloudapi.cn/vhds/" -CreateOption Attach -Windows |
| 创建 VM |[New-AzureRmVM](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.compute/v2.4.0/new-azurermvm) -ResourceGroupName $myResourceGroup -Location $location -VM $vm<BR></BR><BR></BR>所有资源在[资源组](/documentation/articles/powershell-azure-resource-manager/)中创建。 运行此命令之前，请运行 New-AzureRmVMConfig、Set-AzureRmVMOperatingSystem、Set-AzureRmVMSourceImage、Add-AzureRmVMNetworkInterface 和 Set-AzureRmVMOSDisk。 |

## <a name="get-information-about-vms"></a>获取有关 VM 的信息

| 任务 | 命令 |
| ---- | ------- |
| 列出订阅中的 VM |[Get-AzureRmVM](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.compute/v2.5.0/Get-AzureRmVM) |
| 列出资源组中的 VM |Get-AzureRmVM -ResourceGroupName $myResourceGroup<BR></BR><BR></BR>若要获取订阅中的资源组列表，请使用 [Get-AzureRmResourceGroup](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.resources/v3.5.0/get-azurermresourcegroup)。 |
| 获取有关 VM 的信息 |Get-AzureRmVM -ResourceGroupName $myResourceGroup -Name $myVM |

## <a name="manage-vms"></a>管理 VM
| 任务 | 命令 |
| --- | --- |
| 启动 VM |[Start-AzureRmVM](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.compute/v2.5.0/Start-AzureRmVM) -ResourceGroupName $myResourceGroup -Name $myVM |
| 停止 VM |[Stop-AzureRmVM](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.compute/v2.5.0/Stop-AzureRmVM) -ResourceGroupName $myResourceGroup -Name $myVM |
| 重启正在运行的 VM |[Restart-AzureRmVM](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.compute/v2.5.0/Restart-AzureRmVM) -ResourceGroupName $myResourceGroup -Name $myVM |
| 删除 VM |[Remove-AzureRmVM](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.compute/v2.5.0/Remove-AzureRmVM) -ResourceGroupName $myResourceGroup -Name $myVM |
| 对 VM 进行一般化 |[Set-AzureRmVm](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.compute/v2.5.0/Set-AzureRmVM) -ResourceGroupName $myResourceGroup -Name $myVM -Generalized<BR></BR><BR></BR>在运行 Save-AzureRmVMImage 之前运行此命令。 |
| 捕获 VM |[Save-AzureRmVMImage](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.compute/v2.5.0/Save-AzureRmVMImage) -ResourceGroupName $myResourceGroup -VMName $myVM -DestinationContainerName "myImageContainer" -VHDNamePrefix "myImagePrefix" -Path "C:\filepath\filename.json"<BR></BR><BR></BR>虚拟机必须[关闭并通用化](/documentation/articles/virtual-machines-windows-generalize-vhd/)才能用于创建映像。 运行此命令之前，请运行 Set-AzureRmVm。 |
| 更新 VM |[Update-AzureRmVM](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.compute/v2.5.0/Update-AzureRmVM) -ResourceGroupName $myResourceGroup -VM $vm<BR></BR><BR></BR>使用 Get-AzureRmVM 获取当前 VM 配置，更改 VM 对象上的配置设置，然后运行此命令。 |
| 将数据磁盘添加到 VM |[Add-AzureRmVMDataDisk](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.compute/v2.5.0/Add-AzureRmVMDataDisk) -VM $vm -Name "myDataDisk" -VhdUri "https://mystore1.blob.core.chinacloudapi.cn/vhds/myDataDisk.vhd" -LUN # -Caching ReadWrite -DiskSizeinGB # -CreateOption Empty<BR></BR><BR></BR>使用 Get-AzureRmVM 获取 VM 对象。 指定 LUN 号和磁盘大小。 运行 Update-AzureRmVM 将配置更改应用到 VM。 添加的磁盘未进行初始化。 有关添加磁盘时初始化磁盘的信息，请参阅[使用 Resource Manager 与 PowerShell 来管理 Azure 虚拟机](/documentation/articles/virtual-machines-windows-ps-manage/)。 |
| 从 VM 中删除数据磁盘 |[Remove-AzureRmVMDataDisk](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.compute/v2.5.0/Remove-AzureRmVMDataDisk) -VM $vm -Name "myDataDisk"<BR></BR><BR></BR>使用 Get-AzureRmVM 获取 VM 对象。 运行 Update-AzureRmVM 将配置更改应用到 VM。 |
| 将扩展添加到 VM |[Set-AzureRmVMExtension](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.compute/v2.5.0/Set-AzureRmVMExtension) -ResourceGroupName $myResourceGroup -Location $location -VMName $myVM -Name "extensionName" -Publisher "publisherName" -Type "extensionType" -TypeHandlerVersion "#.#" -Settings $Settings -ProtectedSettings $ProtectedSettings<BR></BR><BR></BR>使用要安装的扩展的相应[配置信息](/documentation/articles/virtual-machines-windows-extensions-configuration-samples/)运行此命令。 |
| 删除 VM 扩展 |[Remove-AzureRmVMExtension](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.compute/v2.5.0/Remove-AzureRmVMExtension) -ResourceGroupName $myResourceGroup -Name "extensionName" -VMName $myVM |

## <a name="next-steps"></a>后续步骤
* 请参阅[使用 Resource Manager 和 PowerShell 创建 Windows VM](/documentation/articles/virtual-machines-windows-quick-create-powershell/) 中有关创建虚拟机的基本步骤。
<!--Update_Description: wording update-->