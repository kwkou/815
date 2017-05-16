<properties
    pageTitle="使用 Azure PowerShell 管理 Windows 虚拟机 | Azure"
    description="教程 - 使用 Azure PowerShell 管理 Windows 虚拟机"
    services="virtual-machines-windows"
    documentationcenter="virtual-machines"
    author="davidmu1"
    manager="timlt"
    editor="tysonn"
    tags="azure-resource-manager" />
<tags
    ms.assetid=""
    ms.service="virtual-machines-windows"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="vm-windows"
    ms.workload="infrastructure"
    ms.date="03/21/2017"
    wacn.date="05/15/2017"
    ms.author="davidmu"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="457fc748a9a2d66d7a2906b988e127b09ee11e18"
    ms.openlocfilehash="f1dd2a2d0f7502d0181b5603e99cdb5a84c8f16d"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/05/2017" />

# <a name="manage-windows-virtual-machines-with-azure-powershell"></a>使用 Azure PowerShell 管理 Windows 虚拟机

本教程将创建一个虚拟机并执行常见的管理任务，例如添加磁盘、自动化软件安装、管理防火墙规则以及创建虚拟机快照。

若要完成本教程，请确保已安装最新的 [Azure PowerShell](https://docs.microsoft.com/zh-cn/powershell/azureps-cmdlets-docs/) 模块。

## <a name="step-1---log-in-to-azure"></a>步骤 1 - 登录 Azure

首先，使用 Login-AzureRmAccount 命令登录到 Azure 订阅，并按照屏幕上的说明进行操作。

    Login-AzureRmAccount -EnvironmentName AzureChinaCloud

## <a name="step-2---create-resource-group"></a>步骤 2 - 创建资源组

Azure 资源组是在其中部署和管理 Azure 资源的逻辑容器。 必须在创建虚拟机前创建资源组。

使用 [New-AzureRmResourceGroup](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/AzureRM.Resources/v2.0.3/new-azurermresourcegroup) 创建资源组。  在此示例中，在 `chinanorth` 区域中创建了名为 `myResourceGroup` 的资源组：

    New-AzureRmResourceGroup -ResourceGroupName myResourceGroup -Location chinanorth

## <a name="step-3---create-virtual-machine"></a>步骤 3 - 创建虚拟机

虚拟机必须连接到虚拟网络。 可以通过网络接口卡使用公共 IP 地址与虚拟机通信。

### <a name="create-virtual-network"></a>创建虚拟网络

使用 [New-AzureRmVirtualNetworkSubnetConfig](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.network/v3.6.0/new-azurermvirtualnetworksubnetconfig) 创建子网：

    $subnetConfig = New-AzureRmVirtualNetworkSubnetConfig `
      -Name mySubnet `
      -AddressPrefix 192.168.1.0/24

使用 [New-AzureRmVirtualNetwork](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.network/v3.6.0/new-azurermvirtualnetwork) 创建虚拟网络：

    $vnet = New-AzureRmVirtualNetwork `
      -ResourceGroupName myResourceGroup `
      -Location chinanorth `
      -Name myVnet `
      -AddressPrefix 192.168.0.0/16 `
      -Subnet $subnetConfig

### <a name="create-public-ip-address"></a>创建公共 IP 地址

使用 [New-AzureRmPublicIpAddress](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.network/v3.6.0/new-azurermpublicipaddress) 创建一个公共 IP 地址：

    $pip = New-AzureRmPublicIpAddress `
      -ResourceGroupName myResourceGroup `
      -Location chinanorth `
      -AllocationMethod Static `
      -Name myPublicIPAddress

### <a name="create-network-interface-card"></a>创建网络接口卡

使用 [New-AzureRmNetworkInterface](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.network/v3.6.0/new-azurermnetworkinterface) 创建网络接口卡：

    $nic = New-AzureRmNetworkInterface `
      -ResourceGroupName myResourceGroup `
      -Location chinanorth `
      -Name myNic `
      -SubnetId $vnet.Subnets[0].Id `
      -PublicIpAddressId $pip.Id

### <a name="create-network-security-group"></a>创建网络安全组

Azure [网络安全组](/documentation/articles/virtual-networks-nsg/) (NSG) 控制一个或多个虚拟机的入站和出站流量。 网络安全组规则允许或拒绝特定端口上或端口范围内的网络流量。 这些规则还可包括源地址前缀，这样只有源自预定义源的流量才可与虚拟机进行通信。 若要访问正安装的 IIS web 服务器，必须添加入站 NSG 规则。

若要创建入站 NSG 规则，请使用 [Add-AzureRmNetworkSecurityRuleConfig](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.network/v3.6.0/add-azurermnetworksecurityruleconfig)。 以下示例创建名为 `myNSGRule` 的 NSG 规则，为虚拟机打开端口 `80`：

    $nsgRule = New-AzureRmNetworkSecurityRuleConfig `
      -Name myNSGRule `
      -Protocol Tcp `
      -Direction Inbound `
      -Priority 1000 `
      -SourceAddressPrefix * `
      -SourcePortRange * `
      -DestinationAddressPrefix * `
      -DestinationPortRange 80 `
      -Access Allow

将 `myNSGRule` 与 [New-AzureRmNetworkSecurityGroup](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.network/v3.6.0/new-azurermnetworksecuritygroup) 结合使用，创建 NSG：

    $nsg = New-AzureRmNetworkSecurityGroup `
      -ResourceGroupName myResourceGroup `
      -Location chinanorth `
      -Name myNetworkSecurityGroup `
      -SecurityRules $nsgRule

使用 [Set-AzureRmVirtualNetworkSubnetConfig](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.network/v3.6.0/set-azurermvirtualnetworksubnetconfig) 将 NSG 添加到虚拟网络中的子网：

    Set-AzureRmVirtualNetworkSubnetConfig -Name mySubnet `
      -VirtualNetwork $vnet `
      -NetworkSecurityGroup $nsg `
      -AddressPrefix 192.168.1.0/24

使用 [Set-AzureRmVirtualNetwork](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.network/v3.6.0/set-azurermvirtualnetwork) 更新虚拟网络：

    Set-AzureRmVirtualNetwork -VirtualNetwork $vnet

### <a name="create-virtual-machine"></a>创建虚拟机

创建虚拟机时，可使用多个选项，例如操作系统映像、磁盘大小调整和管理凭据。 在此示例中，将创建名为 `myVM` 的虚拟机，运行最新版本的 Windows Server 2016 Datacenter。

使用 [Get-Credential](https://msdn.microsoft.com/powershell/reference/5.1/microsoft.powershell.security/Get-Credential) 设置虚拟机上管理员帐户所需的用户名和密码：

    $cred = Get-Credential

使用 [New-AzureRmVMConfig](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.compute/v2.8.0/new-azurermvmconfig) 为虚拟机创建初始配置：

    $vm = New-AzureRmVMConfig -VMName myVM -VMSize Standard_D1

使用 [Set-AzureRmVMOperatingSystem](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.compute/v2.8.0/set-azurermvmoperatingsystem) 向虚拟机配置添加操作系统信息：

    $vm = Set-AzureRmVMOperatingSystem -VM $vm `
      -Windows `
      -ComputerName myVM `
      -Credential $cred `
      -ProvisionVMAgent `
      -EnableAutoUpdate

使用 [Set-AzureRmVMSourceImage](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.compute/v2.8.0/set-azurermvmsourceimage) 向虚拟机配置添加映像信息：

    $vm = Set-AzureRmVMSourceImage -VM $vm `
      -PublisherName MicrosoftWindowsServer `
      -Offer WindowsServer `
      -Skus 2016-Datacenter `
      -Version latest

使用 [Set-AzureRmVMOSDisk](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.compute/v2.8.0/set-azurermvmosdisk) 向虚拟机配置添加操作系统磁盘设置：

    $vm = Set-AzureRmVMOSDisk -VM $vm `
      -Name myOsDisk `
      -DiskSizeInGB 128 `
      -CreateOption FromImage `
      -Caching ReadWrite

使用 [Add-AzureRmVMNetworkInterface](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.compute/v2.8.0/add-azurermvmnetworkinterface) 向虚拟机配置添加以前创建的网络接口卡：

    $vm = Add-AzureRmVMNetworkInterface -VM $vm -Id $nic.Id

使用 [New-AzureRmVM](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.compute/v2.8.0/new-azurermvm) 创建虚拟机：

    New-AzureRmVM -ResourceGroupName myResourceGroup -Location chinanorth -VM $vm

## <a name="step-4---add-data-disk"></a>步骤 4 - 添加数据磁盘

默认情况下，使用单个操作系统磁盘即可创建 Azure 虚拟机。 可以添加更多磁盘，以便进行多磁盘存储配置、应用程序安装和数据存储。

### <a name="create-disk"></a>创建磁盘

使用 [New-AzureRmDiskConfig](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.compute/v2.8.0/new-azurermdiskconfig) 创建数据磁盘的初始配置。 以下示例创建名为 `myDataDisk` 且大小为 50 GB 的磁盘：

    $diskConfig = New-AzureRmDiskConfig `
      -Location chinanorth `
      -CreateOption Empty `
      -DiskSizeGB 50

使用 [New-AzureRmDisk](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.compute/v2.8.0/new-azurermdisk) 创建数据磁盘：

    $dataDisk = New-AzureRmDisk `
      -ResourceGroupName myResourceGroup `
      -DiskName myDataDisk `
      -Disk $diskConfig

使用 [Get-AzureRmVM](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.compute/v2.8.0/get-azurermvm) 获取要向其添加数据磁盘的虚拟机：

    $vm = Get-AzureRmVM -ResourceGroupName myResourceGroup -Name myVM

使用 [Add-AzureRmVMDataDisk](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.compute/v2.8.0/add-azurermvmdatadisk) 向虚拟机配置添加数据磁盘：

    $vm = Add-AzureRmVMDataDisk `
      -VM $vm `
      -Name myDataDisk `
      -CreateOption Attach `
      -ManagedDiskId $dataDisk.Id `
      -Lun 1

使用 [Update-AzureRmVM](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.compute/v2.8.0/update-azurermvm) 更新虚拟机：

    Update-AzureRmVM -ResourceGroupName myResourceGroup -VM $vm

## <a name="step-5---automate-configuration"></a>步骤 5 - 自动配置

Azure 虚拟机扩展用于自动执行虚拟机配置任务（例如安装应用程序和配置操作系统）。 [适用于 Windows 的自定义脚本扩展](/documentation/articles/virtual-machines-windows-extensions-customscript/)用于在虚拟机上运行任何 PowerShell 脚本。 此脚本可存储于 Azure 存储、任何可访问的 HTTP 终结点或者嵌入自定义脚本扩展配置中。 在本教程中，将实现两个操作的自动化：

- 安装 IIS
- 格式化 VM 上的数据磁盘

由于扩展在 VM 部署时运行，需先定义 **install-iis-format-disk.ps1** 文件，然后才能创建虚拟机。 就本教程来说，该文件位于 Azure PowerShell 脚本存储库中。 该文件包含 `Add-WindowsFeature Web-Server` 命令和 `Get-Disk | Where partitionstyle -eq 'raw' | Initialize-Disk -PartitionStyle MBR -PassThru | New-Partition -AssignDriveLetter -UseMaximumSize | Format-Volume -FileSystem NTFS -NewFileSystemLabel "myDataDisk" -Confirm:$false**` 命令。

使用 [Set-AzureRmVMExtension](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.compute/v2.8.0/set-azurermvmextension) 添加扩展：

    Set-AzureRmVMExtension -Name myScript `
      -ResourceGroupName myResourceGroup `
      -Location chinanorth `
      -VMName myVM `
      -Publisher Microsoft.Compute `
      -ExtensionType CustomScriptExtension `
      -TypeHandlerVersion 1.4 `
      -SettingString '{"fileUris":["https://raw.githubusercontent.com/Azure/azure-docs-powershell-samples/master/virtual-machine/install-iis-format-disk/install-iis-format-disk.ps1?token=ABlGkLjzVTdZGoCRbu1pCXjvSDRMHnUlks5Y5nbZwA%3D%3D"], "commandToExecute":"powershell.exe -ExecutionPolicy Unrestricted -File install-iis-format-disk.ps1" }'

使用 [Get-AzureRmPublicIPAddress](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.network/v3.6.0/get-azurermpublicipaddress) 获取虚拟机的公共 IP 地址：

    Get-AzureRmPublicIPAddress -ResourceGroupName myResourceGroup -Name myPublicIPAddress | select IpAddress

现在浏览到虚拟机的公共 IP 地址。 如果具有 NSG 规则，将显示默认 IIS 网站。

## <a name="step-6---snapshot-virtual-machine"></a>步骤 6 - 虚拟机快照

拍摄虚拟机的快照即可创建虚拟机操作系统磁盘的只读时间点副本。 通过快照，可将虚拟机还原到特定状态，或者可使用快照创建具有相同状态的新虚拟机。

### <a name="create-snapshot"></a>创建快照

创建虚拟机磁盘快照之前，需提供虚拟机操作系统磁盘的 ID。

使用 [Get-AzureRmDisk](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.compute/v2.8.0/get-azurermdisk) 获取操作系统磁盘：

    $vmOSDisk = Get-AzureRmDisk -ResourceGroupName myResourceGroup -Name myOSDisk

使用 New-AzureRmSnapshotConfig 创建快照的配置：

    $snapshotConfig = New-AzureRmSnapshotConfig -Location chinanorth -CreateOption Copy -SourceResourceId $vmOSDisk.id

使用 New-AzureRmSnapshot 创建快照：

    $snapshot = New-AzureRmSnapshot -ResourceGroupName myResourceGroup -SnapshotName mySnapshot -Snapshot $snapshotConfig

### <a name="create-disk-from-snapshot"></a>从快照创建磁盘

然后，可将此快照转换为可用于重新创建虚拟机的磁盘。

使用 [New-AzureRmDiskConfig](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.compute/v2.8.0/new-azurermdiskconfig) 创建磁盘的配置：

    $diskConfig = New-AzureRmDiskConfig -Location chinanorth -CreateOption Copy -SourceResourceId $snapshot.id

使用 [New-AzureRmDisk](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.compute/v2.8.0/new-azurermdisk) 创建磁盘：

    $disk = New-AzureRmDisk -ResourceGroupName myResourceGroup -DiskName myOSDiskFromSnapshot -Disk $diskConfig

### <a name="restore-virtual-machine-from-snapshot"></a>从快照还原虚拟机

若要演示如何还原虚拟机，请删除现有虚拟机。

    Remove-AzureRmVM -ResourceGroupName myResourceGroup -Name myVM -Force

从快照磁盘创建新虚拟机。 在此示例中，将指定现有的网络接口。 此配置将以前创建的所有 NSG 规则应用到新的虚拟机。

使用 [New-AzureRmVMConfig](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.compute/v2.8.0/new-azurermvmconfig) 为虚拟机创建初始配置：

    $vm = New-AzureRmVMConfig -VMName myVM -VMSize Standard_D1

使用 [Set-AzureRmVMOSDisk](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.compute/v2.8.0/set-azurermvmosdisk) 添加操作系统磁盘：

    $vm = Set-AzureRmVMOSDisk -VM $vm -CreateOption Attach -ManagedDiskId $disk.Id -Windows

使用 [Add-AzureRmVMNetworkInterface](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.compute/v2.8.0/add-azurermvmnetworkinterface) 添加网络接口卡：

    $vm = Add-AzureRmVMNetworkInterface -VM $vm -Id $nic.Id

使用 [New-AzureRmVM](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.compute/v2.8.0/new-azurermvm) 创建虚拟机：

    New-AzureRmVM -ResourceGroupName myResourceGroup -VM $vm -Location chinanorth

将虚拟机的公共 IP 地址输入 Internet 浏览器的地址栏中。 应会看到该 IIS 正在还原的虚拟机中运行。

## <a name="step-7---management-tasks"></a>步骤 7 - 管理任务

在虚拟机生命周期中，你可能需要运行管理任务，例如启动、停止或删除虚拟机。 此外，可能还需要创建脚本来自动执行重复或复杂的任务。 使用 Azure PowerShell，可从命令行或脚本运行许多常见的管理任务。

### <a name="stop-virtual-machine"></a>停止虚拟机

使用 [Stop-AzureRmVM](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.compute/v2.8.0/stop-azurermvm) 停止并解除分配虚拟机：

    Stop-AzureRmVM -ResourceGroupName myResourceGroup -Name myVM -Force

若要让虚拟机保持已预配状态，请使用 -StayProvisioned 参数。

### <a name="resize-virtual-machine"></a>调整虚拟机大小

若要调整已停止并解除分配的虚拟机的大小，你需要所选 Azure 区域中可用大小的名称。 使用 [Get-AzureRmVMSize](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.compute/v2.8.0/get-azurermvmsize) 可查找这些大小：

    Get-AzureRmVMSize -Location chinanorth

使用 [Get-AzureRmDisk](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.compute/v2.8.0/get-azurermdisk) 获取要调整大小的虚拟机：

    $vm = Get-AzureRmVM -Name myVM -ResourceGroupName myResourceGroup

使用所选大小设置 vmSize 属性，以便设置虚拟机的新大小。

    $vm.HardwareProfile.vmSize = "Standard_F4s"

使用 [Update-AzureRmVM](https://docs.microsoft.com/zh-cn/powershell/resourcemanager/azurerm.compute/v2.8.0/update-azurermvm) 更新虚拟机：

    Update-AzureRmVM -ResourceGroupName myResourceGroup -VM $vm

### <a name="start-virtual-machine"></a>启动虚拟机

    Start-AzureRmVM -ResourceGroupName myResourceGroup -Name myVM

### <a name="delete-resource-group"></a>删除资源组

删除资源组会删除其包含的所有资源。

    Remove-AzureRmResourceGroup -Name myResourceGroup -Force

## <a name="next-steps"></a>后续步骤

示例 - [Azure 虚拟机 PowerShell 示例脚本](/documentation/articles/virtual-machines-windows-powershell-samples/)