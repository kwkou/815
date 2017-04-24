<properties
    pageTitle="在 Azure 中扩展附加到 Windows VM 的数据磁盘 | Azure"
    description="使用 PowerShell 扩展附加到 Windows 虚拟机的数据磁盘的大小。"
    services="virtual-machines-windows"
    documentationcenter="na"
    author="cynthn"
    manager="timlt"
    editor=""
    tags=""
    translationtype="Human Translation" />
<tags
    ms.assetid=""
    ms.service="virtual-machines-windows"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="infrastructure-services"
    ms.date="03/02/2017"
    wacn.date="04/17/2017"
    ms.author="cynthn"
    ms.sourcegitcommit="e0e6e13098e42358a7eaf3a810930af750e724dd"
    ms.openlocfilehash="cadc7c639f4a332b33550007db0409fe7759448e"
    ms.lasthandoff="04/06/2017" />

# <a name="increase-the-size-of-a-data-disk-attached-to-a-windows-vm"></a>增加附加到 Windows VM 的数据磁盘的大小

若要增加附加到虚拟机的数据磁盘的大小，可以使用 PowerShell 增加大小。 在 Azure VM 设置中增加数据磁盘的大小后，还需在 VM 中分配新的磁盘空间。

## <a name="use-powershell-to-increase-the-size-of-an-unmanaged-data-disk"></a>使用 PowerShell 增加非托管数据磁盘的大小

若要在存储帐户中增加非托管数据磁盘的大小，请使用以下 PowerShell cmdlet：

|                                                                    |                                                            |
|--------------------------------------------------------------------|------------------------------------------------------------|
| [Get-AzureRMStorageAccount](https://docs.microsoft.com/zh-cn/powershell/module/azurerm.storage/get-azurermstorageaccount?view=azurermps-3.8.0) | [Get-AzureRMVM](https://docs.microsoft.com/zh-cn/powershell/module/azurerm.compute/get-azurermvm?view=azurermps-3.8.0)                  |
| [Stop-AzureRMVM](https://docs.microsoft.com/zh-cn/powershell/module/azurerm.compute/stop-azurermvm?view=azurermps-3.8.0)                       | [Set-AzureRmVMDataDisk](https://docs.microsoft.com/zh-cn/powershell/module/azurerm.compute/set-azurermvmdatadisk?view=azurermps-3.8.0) |
| [Update-AzureRmVM](https://docs.microsoft.com/zh-cn/powershell/module/azurerm.compute/update-azurermvm?view=azurermps-3.8.0)                   | [Start-AzureRmVM](https://docs.microsoft.com/zh-cn/powershell/module/azurerm.compute/start-azurermvm?view=azurermps-3.8.0)             |

<br>

以下脚本将引导你获取 VM 和存储帐户信息、选择数据磁盘和指定新的大小。

    # Select Azure Storage Account

        $storageAccount =
            Get-AzureRMStorageAccount | Out-GridView `
                -Title "Select Azure Storage Account" `
                -PassThru

        $rgName = $storageAccount.ResourceGroupName

    # Select the VM

        $vm = Get-AzureRMVM `
        -ResourceGroupName $rgName | Out-GridView `
                -Title "Select a VM …" `
                -PassThru

    # Select Data Disk to resize

        $disk =
            $vm.DataDiskNames | Out-GridView `
                -Title "Select a data disk to resize" `
                -PassThru

    # Specify a larger data disk size 

        $size =  Read-Host `
            -Prompt "New size in GB"

    # Stop and Deallocate VM prior to resizing data disk

        $vm | Stop-AzureRMVM -Force

    # Set the new disk size

        Set-AzureRmVMDataDisk -VM $vm -Name "$disk" `
            -DiskSizeInGB $size

    # Update the configuration in Azure

        Update-AzureRmVM -VM $vm -ResourceGroupName $rgName

    # Start the VM
        Start-AzureRmVM -ResourceGroupName $rgName `
        -VMName $vm.name
    

## <a name="allocate-the-unallocated-disk-space"></a>分配未分配的磁盘空间 

增加驱动器的大小后，需要从 VM 中分配新的未分配空间。 若要分配空间，可使用磁盘管理 (diskmgmt.msc) 连接到 VM。 或者，如果在创建 VM 时在其上启用了 WinRM 和证书，则可以使用远程 PowerShell 初始化该磁盘。 还可以使用自定义脚本扩展： 

        $location = "location-name"
        $scriptName = "script-name"
        $fileName = "script-file-name"
        Set-AzureRmVMCustomScriptExtension -ResourceGroupName $rgName -Location $locName -VMName $vmName -Name $scriptName -TypeHandlerVersion "1.4" -StorageAccountName "mystore1" -StorageAccountKey "primary-key" -FileName $fileName -ContainerName "scripts"

脚本文件可能包含类似于此代码的内容，以将驱动器分配增加到磁盘的最大大小：

    $driveLetter= "F"

    $MaxSize = (Get-PartitionSupportedSize -DriveLetter $driveLetter).sizeMax

    Resize-Partition -DriveLetter $driveLetter -Size $MaxSize

## <a name="next-steps"></a>后续步骤
- [深入了解磁盘和 VHD](/documentation/articles/storage-about-disks-and-vhds-windows/)