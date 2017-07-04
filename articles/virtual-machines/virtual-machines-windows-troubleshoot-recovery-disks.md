<properties
    pageTitle="将 Windows 故障排除 VM 与 Azure PowerShell 联合使用 | Azure"
    description="了解如何通过 Azure PowerShell 将 OS 磁盘连接到恢复 VM，以便排查 Azure 中的 Windows VM 问题。"
    services="virtual-machines-windows"
    documentationCenter=""
    authors="iainfoulds"
    manager="timlt"
    editor="" />
<tags
    ms.service="virtual-machines-windows"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="vm-windows"
    ms.workload="infrastructure"
    ms.date="12/13/2016"
    wacn.date="03/28/2017"
    ms.author="iainfou" />  


# 通过使用 Azure PowerShell 将 OS 磁盘附加到恢复 VM 来对 Windows VM 进行故障排除
如果 Windows 虚拟机 (VM) 在 Azure 中遇到启动或磁盘错误，可能需要对虚拟硬盘本身执行故障排除步骤。一个常见示例是应用程序更新失败，使 VM 无法成功启动。本文详细介绍如何使用 Azure PowerShell 将虚拟硬盘连接到另一个 Windows VM 来修复所有错误，然后重新创建原始 VM。


## 恢复过程概述
故障排除过程如下：

1. 删除遇到问题的 VM，保留虚拟硬盘。
2. 将虚拟硬盘附加并装入到另一个 Windows VM，以便进行故障排除。
3. 连接到故障排除 VM。编辑文件或运行任何工具以修复原始虚拟硬盘上的问题。
4. 从故障排除 VM 卸载并分离虚拟硬盘。
5. 使用原始虚拟硬盘创建 VM。

确保已安装[最新 Azure PowerShell](https://docs.microsoft.com/powershell/azureps-cmdlets-docs) 并登录到订阅：

    Login-AzureRMAccount -EnvironmentName AzureChinaCloud

在以下示例中，请将参数名称替换为你自己的值。示例参数名称包括 `myResourceGroup`、`mystorageaccount` 和 `myVM`。


## 确定启动问题
用户可以通过查看 Azure 中 VM 的屏幕快照来排查启动问题。此屏幕快照有助于确定为何 VM 无法启动。以下示例从名为 `myResourceGroup` 的资源组中名为 `myVM` 的 Windows VM 获取屏幕快照：

    Get-AzureRmVMBootDiagnosticsData -ResourceGroupName myResourceGroup `
        -Name myVM -Windows -LocalPath C:\Users\ops\

检查屏幕快照，确定 VM 无法启动的原因。请注意所提供的任何特定错误消息或错误代码。


## 查看现有虚拟硬盘的详细信息
在将虚拟硬盘附加到另一个 VM 之前，需要标识虚拟硬盘 (VHD) 的名称。

以下示例从名为 `myResourceGroup` 的资源组中获取名为 `myVM` 的 VM 的信息：

    Get-AzureRmVM -ResourceGroupName "myResourceGroup" -Name "myVM"

在前一个命令的输出的 `StorageProfile` 部分查找 `Vhd URI`。下面这个截断的示例输出显示了代码块末尾的 `Vhd URI`：

    RequestId                     : 8a134642-2f01-4e08-bb12-d89b5b81a0a0
    StatusCode                    : OK
    ResourceGroupName             : myResourceGroup
    Id                            : /subscriptions/guid/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM
    Name                          : myVM
    Type                          : Microsoft.Compute/virtualMachines
    ...
    StorageProfile                :
      ImageReference              :
        Publisher                 : MicrosoftWindowsServer
        Offer                     : WindowsServer
        Sku                       : 2016-Datacenter
        Version                   : latest
      OsDisk                      :
        OsType                    : Windows
        Name                      : myVM
        Vhd                       :
          Uri                     : https://mystorageaccount.blob.core.chinacloudapi.cn/vhds/myVM.vhd
        Caching                   : ReadWrite
        CreateOption              : FromImage

## 删除现有 VM
虚拟硬盘和 VM 在 Azure 中是两个不同的资源。虚拟硬盘是操作系统本身，存储应用程序和配置。VM 本身只是定义大小或位置的元数据，引用虚拟硬盘或虚拟网络接口卡 (NIC) 等资源。每个虚拟硬盘在附加到 VM 时分配有一个租约。尽管 VM 正在运行时也可以附加和分离数据磁盘，但是，若要分离 OS 磁盘，则必须删除 VM 资源。即使 VM 处于停止和解除分配状态，租约也继续将 OS 磁盘与 VM 相关联。

恢复 VM 的第一步是删除 VM 资源本身。删除 VM 时会将虚拟硬盘留在存储帐户中。删除 VM 后，可将虚拟硬盘附加到另一个 VM，以进行故障排除和解决这些错误。

以下示例从名为 `myResourceGroup` 的资源组中删除名为 `myVM` 的 VM：

    Remove-AzureRmVM -ResourceGroupName "myResourceGroup" -Name "myVM"

等到 VM 已完成删除，然后再将虚拟硬盘附加到另一个 VM。虚拟硬盘上将其与 VM 关联的租约需要释放，然后才能将虚拟硬盘附加到另一个 VM。


## 将现有虚拟硬盘附加到另一个 VM
在后续几个步骤中，将使用另一个 VM 进行故障排除。将现有虚拟硬盘附加到此故障排除 VM，以浏览和编辑磁盘的内容。例如，此过程允许用户更正任何配置错误或者查看其他应用程序或系统日志文件。选择或创建另一个 VM 以用于故障排除。

附加现有的虚拟硬盘时，指定在前面的 `Get-AzureRmVM` 命令中获取的磁盘 URL。以下示例将现有的虚拟硬盘附加到名为 `myResourceGroup` 的资源组中名为 `myVMRecovery` 的故障排除 VM：

    $myVM = Get-AzureRmVM -ResourceGroupName "myResourceGroup" -Name "myVMRecovery"
    Add-AzureRmVMDataDisk -VM $myVM -CreateOption "Attach" -Name "DataDisk" -DiskSizeInGB $null `
        -VhdUri "https://mystorageaccount.blob.core.chinacloudapi.cn/vhds/myVM.vhd"
    Update-AzureRmVM -ResourceGroup "myResourceGroup" -VM $myVM

> [AZURE.NOTE]
添加磁盘时需指定磁盘的大小。我们在附加现有磁盘时，会将 `-DiskSizeInGB` 指定为 `$null`。此值可确保正确附加数据磁盘，无需确定数据磁盘的实际大小。


## 装载附加的数据磁盘

1. 使用相应的凭据通过 RDP 连接到故障排除 VM。以下示例为名为 `myResourceGroup` 的资源组中名为 `myVMRecovery` 的 VM 下载 RDP 连接文件，该文件下载到 `C:\Users\ops\Documents`。

        Get-AzureRMRemoteDesktopFile -ResourceGroupName "myResourceGroup" -Name "myVMRecovery" `
            -LocalPath "C:\Users\ops\Documents\myVMRecovery.rdp"

2. 系统会自动检测并附加数据磁盘。查看已附加卷的列表以确定驱动器号，如下所示：

        Get-Disk

    以下示例输出显示虚拟硬盘连接了磁盘 **2**。（也可使用 `Get-Volume` 查看驱动器号）：

        Number   Friendly Name   Serial Number   HealthStatus   OperationalStatus   Total Size   Partition
                                                                                                 Style
        ------   -------------   -------------   ------------   -----------------   ----------   ----------
        0        Virtual HD                                     Healthy             Online       127 GB MBR
        1        Virtual HD                                     Healthy             Online       50 GB MBR
        2        Msft Virtu...                                  Healthy             Online       127 GB MBR

## 修复原始虚拟硬盘上的问题
装载现有虚拟硬盘后，可以根据需要执行任何维护和故障排除步骤。解决问题后，请继续执行以下步骤。


## 卸载并分离原始虚拟硬盘
解决错误后，可从故障排除 VM 中卸载并分离现有虚拟硬盘。在将虚拟硬盘附加到故障排除 VM 的租约释放前，不能将该虚拟硬盘用于任何其他 VM。

1. 从 RDP 会话卸载恢复 VM 上的数据磁盘。需要以前的 `Get-Disk` cmdlet 中的磁盘编号。然后，使用 `Set-Disk` 将磁盘设置为脱机：

        Set-Disk -Number 2 -IsOffline $True

    再次使用 `Get-Disk` 确认磁盘现已设置为脱机。下面的示例输出显示磁盘现已设置为脱机：

        Number   Friendly Name   Serial Number   HealthStatus   OperationalStatus   Total Size   Partition
                                                                                                 Style
        ------   -------------   -------------   ------------   -----------------   ----------   ----------
        0        Virtual HD                                     Healthy             Online       127 GB MBR
        1        Virtual HD                                     Healthy             Online       50 GB MBR
        2        Msft Virtu...                                  Healthy             Offline      127 GB MBR

2. 退出 RDP 会话。通过 Azure PowerShell 会话删除故障排除 VM 中的虚拟硬盘。

        $myVM = Get-AzureRmVM -ResourceGroupName "myResourceGroup" -Name "myVMRecovery"
        Remove-AzureRmVMDataDisk -VM $myVM -Name "DataDisk"
        Update-AzureRmVM -ResourceGroup "myResourceGroup" -VM $myVM

## 从原始硬盘创建 VM
若要从原始虚拟硬盘创建 VM，请使用[此 Azure Resource Manager 模板](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vm-specialized-vhd-existing-vnet)。实际的 JSON 模板位于以下链接中：

- https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-vm-specialized-vhd-existing-vnet/azuredeploy.json  


该模板使用先前命令中的 VHD URL 将 VM 部署到现有虚拟网络中。以下示例将模板部署到名为 `myResourceGroup` 的资源组：

    New-AzureRmResourceGroupDeployment -Name myDeployment -ResourceGroupName myResourceGroup `
      -TemplateUri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/201-vm-specialized-vhd-existing-vnet/azuredeploy.json

响应模板的提示，例如 VM 名称、OS 类型和 VM 大小。`osDiskVhdUri` 与前面将现有虚拟硬盘附加到故障排除 VM 时使用的相同。


## 重新启用启动诊断

从现有虚拟硬盘创建 VM 时，启动诊断可能不会自动启用。以下示例在名为 `myResourceGroup` 的资源组中名为 `myVMDeployed` 的 VM 上启用诊断扩展：

    $myVM = Get-AzureRmVM -ResourceGroupName "myResourceGroup" -Name "myVMDeployed"
    Set-AzureRmVMBootDiagnostics -ResourceGroupName myResourceGroup -VM $myVM -enable
    Update-AzureRmVM -ResourceGroup "myResourceGroup" -VM $myVM

## 后续步骤
如果在连接到 VM 时遇到问题，请参阅[排查 Azure VM 的 RDP 连接问题](/documentation/articles/virtual-machines-windows-troubleshoot-rdp-connection/)。如果在访问 VM 上运行的应用时遇到问题，请参阅[排查 Windows VM 上的应用程序连接问题](/documentation/articles/virtual-machines-windows-troubleshoot-app-connection/)。

有关资源组的详细信息，请参阅 [Azure Resource Manager 概述](/documentation/articles/resource-group-overview/)。

<!---HONumber=Mooncake_0109_2017-->