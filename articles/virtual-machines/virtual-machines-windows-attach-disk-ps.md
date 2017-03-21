<properties
    pageTitle="使用 PowerShell 将数据磁盘附加到 Azure 中的 Windows VM | Azure"
    description="如何通过 Resource Manager 部署模型使用 PowerShell 将新磁盘或现有数据磁盘附加到 Windows VM。"
    services="virtual-machines-windows"
    documentationcenter=""
    author="cynthn"
    manager="timlt"
    editor=""
    tags="azure-resource-manager" />
<tags
    ms.assetid=""
    ms.service="virtual-machines-windows"
    ms.workload="infrastructure-services"
    ms.tgt_pltfrm="vm-windows"
    ms.devlang="na"
    ms.topic="article"
    ms.date="02/07/2017"
    wacn.date="03/20/2017"
    ms.author="cynthn" />  


# 使用 PowerShell 将数据磁盘附加到 Windows VM

本文介绍如何使用 PowerShell 将新磁盘和现有磁盘附加到 Windows 虚拟机。还可以将非托管数据磁盘附加到使用存储帐户中非托管磁盘的 VM。

>[AZURE.NOTE] Azure 中国区尚无法使用 Azure 托管磁盘。

在开始之前，请查看以下提示：
* 虚拟机的大小决定了可以附加多少个磁盘。有关详细信息，请参阅[虚拟机大小](/documentation/articles/virtual-machines-windows-sizes/)。
* 若要使用高级存储，需要支持高级存储的 VM 大小，如 DS 系列或 GS 系列虚拟机。可以用高级存储帐户和标准存储帐户将磁盘用于这些虚拟机。高级存储只在某些区域可用。有关详细信息，请参阅[高级存储：适用于 Azure 虚拟机工作负荷的高性能存储](/documentation/articles/storage-premium-storage/)。

## 开始之前
如果使用 PowerShell，请确保使用的是最新版本的 AzureRM.Compute PowerShell 模块。运行以下命令来安装该模块。

    Install-Module AzureRM.Compute -RequiredVersion 2.6.0

有关详细信息，请参阅 [Azure PowerShell 版本控制](https://docs.microsoft.com/powershell/azureps-cmdlets-docs/#azure-powershell-versioning)。

## 将空数据磁盘添加到虚拟机

此示例演示了如何将空数据磁盘添加到现有的虚拟机。

### 使用存储帐户中的非托管磁盘

        $vm = Get-AzureRmVM -ResourceGroupName $rgName -Name $vmName
        Add-AzureRmVMDataDisk -VM $vm -Name "disk-name" -VhdUri "https://mystore1.blob.core.chinacloudapi.cn/vhds/datadisk1.vhd" -LUN 0 -Caching ReadWrite -DiskSizeinGB 1 -CreateOption Empty
        Update-AzureRmVM -ResourceGroupName $rgName -VM $vm

### 初始化磁盘

添加空磁盘后，需要对其进行初始化。若要初始化磁盘，可以登录到 VM，然后使用磁盘管理进行初始化。如果在创建 VM 时在其上启用了 WinRM 和证书，则可以通过远程 PowerShell 初始化该磁盘。还可以使用自定义脚本扩展：

        $location = "location-name"
        $scriptName = "script-name"
        $fileName = "script-file-name"
        Set-AzureRmVMCustomScriptExtension -ResourceGroupName $rgName -Location $locName -VMName $vmName -Name $scriptName -TypeHandlerVersion "1.4" -StorageAccountName "mystore1" -StorageAccountKey "primary-key" -FileName $fileName -ContainerName "scripts"

脚本文件可以包含类似如下所示代码初始化磁盘：

        $disks = Get-Disk | Where partitionstyle -eq 'raw' | sort number

        $letters = 70..89 | ForEach-Object { [char]$_ }
        $count = 0
        $labels = "data1","data2"

        foreach ($disk in $disks) {
            $driveLetter = $letters[$count].ToString()
            $disk | 
            Initialize-Disk -PartitionStyle MBR -PassThru |
            New-Partition -UseMaximumSize -DriveLetter $driveLetter |
            Format-Volume -FileSystem NTFS -NewFileSystemLabel $labels[$count] -Confirm:$false -Force
        $count++
        }

## 将现有数据磁盘附加到 VM

还可以将现有 VHD 作为托管数据磁盘附加到虚拟机。

### 使用非托管磁盘

        $vm = Get-AzureRmVM -ResourceGroupName $rgName -Name $vmName
        Add-AzureRmVMDataDisk -VM $vm -Name "disk-name" -VhdUri "https://mystore1.blob.core.chinacloudapi.cn/vhds/datadisk1.vhd" -LUN 0 -Caching ReadWrite -DiskSizeinGB 1 -CreateOption Attach
        Update-AzureRmVM -ResourceGroupName $rgName -VM $vm

<!---HONumber=Mooncake_0313_2017-->