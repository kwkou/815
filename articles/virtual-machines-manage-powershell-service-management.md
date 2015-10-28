<properties pageTitle="在 Azure PowerShell 中使用服务管理模式的常见 VM 任务" description="为你提供在 Azure PowerShell 中使用服务管理模式的常见 VM 任务的命令" services="virtual-machines" documentationCenter="windows" authors="KBDAzure" manager="timlt" editor=""/>

<tags ms.service="virtual-machines" ms.date="04/22/2015" wacn.date="06/26/2015"/>

# 在服务管理模式下使用 Azure PowerShell 的常见 VM 任务

> [AZURE.SELECTOR]
> - [Resource Manager](/documentation/articles/virtual-machines-manage-powershell-resource-management)
> - [Service Management](/documentation/articles/virtual-machines-manage-powershell-service-management)

每天执行的许多管理 VM 的任务都可通过在服务管理模式下使用 Azure PowerShell cmdlet 自动执行。本文提供较简单任务的示例命令，并提供演示更复杂任务的命令的文章链接。

>[AZURE.NOTE]如果你尚未安装和配置 Azure PowerShell，则可以在[此处](/documentation/articles/install-configure-powershell)获取相关说明。

## 如何使用示例命令
你需要将命令中的一些文本替换为适合你的环境的文本。< and > 符号指示需要替换的文本。替换文本时，请删除符号，但将引号保留在原处。

## 获取 VM
这是你将经常使用的基本任务。使用它可获取有关 VM 的信息、对 VM 执行任务或获取输出以存储在变量中。

若要获取有关 VM 的信息，请运行此命令，并替换引号内的所有内容（包括 < and > 字符）：

     Get-AzureVM -ServiceName "<cloud service name>" -Name "<virtual machine name>"

若要将输出存储在 $vm 变量中，请运行：

    $vm = Get-AzureVM -ServiceName "<cloud service name>" -Name "<virtual machine name>"

## 登录到基于 Windows 的虚拟机

运行以下命令：

>[AZURE.NOTE]可以从显示的 **Get-AzureVM** 命令中获取虚拟机和云服务名称。
>
	$svcName="<cloud service name>"
	$vmName="<virtual machine name>"
	$localPath="<drive and folder location to store the downloaded RDP file, example: c:\temp >"
	$localFile=$localPath + "" + $vmname + ".rdp"
	Get-AzureRemoteDesktopFile -ServiceName $svcName -Name $vmName -LocalPath $localFile -Launch

## 停止 VM

运行以下命令：

    Stop-AzureVM -ServiceName "<cloud service name>" -Name "<virtual machine name>"

>[AZURE.IMPORTANT]如果此 VM 是云服务中的最后一个 VM，则使用 **StayProvisioned** 参数可以保留该云服务的虚拟 IP (VIP)。如果你使用此参数，则仍要支付 VM 的费用。

## 启动 VM

运行以下命令：

    Start-AzureVM -ServiceName "<cloud service name>" -Name "<virtual machine name>"

## 附加数据磁盘
此任务需要几个步骤。首先，使用 ****Add-AzureDataDisk**** cmdlet 将磁盘添加到 $vm 对象，然后使用 Update-AzureVM cmdlet 更新 VM 的配置。

你还需要确定是要附加新的磁盘还是附加包含数据的磁盘。对于新磁盘，此命令将创建 .vhd 文件，然后将它附加在同一个命令中。

若要附加新磁盘，请运行以下命令：

    Add-AzureDataDisk -CreateNew -DiskSizeInGB 128 -DiskLabel "<main>" -LUN <0> -VM <$vm> `
              | Update-AzureVM

若要附加现有数据磁盘，请运行以下命令：

    Add-AzureDataDisk -Import -DiskName "<MyExistingDisk>" -LUN <0> `
              | Update-AzureVM

若要从 blob 存储中现有的 .vhd 文件附加数据磁盘，请运行以下命令：

    Add-AzureDataDisk -ImportFrom -MediaLocation `
              "<https://mystorage.blob.core.chinacloudapi.cn/mycontainer/MyExistingDisk.vhd>" `
              -DiskLabel "<main>" -LUN <0> `
              | Update-AzureVM

## 创建 Windows VM

若要在 Azure 中创建新的基于 Windows 的虚拟机，请使用[使用 Azure PowerShell 创建和预配置基于 Windows 的虚拟机](/documentation/articles/virtual-machines-ps-create-preconfigure-windows-vms)中的说明。本主题指导你逐步完成创建 PowerShell 命令集，以创建可预先配置以下项的 Windows 虚拟机：

- Active Directory 域成员身份
- 附加磁盘
- 作为现有负载平衡集的成员
- 静态 IP 地址

<!---HONumber=61-->