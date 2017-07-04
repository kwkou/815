<properties
    pageTitle="调整 Windows VM 的大小 | Azure"
    description="使用 Azure Powershell 调整在 Resource Manager 部署模型中创建的 Windows 虚拟机的大小。"
    services="virtual-machines-windows"
    documentationcenter=""
    author="Drewm3"
    manager="timlt"
    editor=""
    tags="azure-resource-manager" />
<tags 
    ms.assetid="057ff274-6dad-415e-891c-58f8eea9ed78"
    ms.service="virtual-machines-windows"
    ms.workload="na"
    ms.tgt_pltfrm="vm-windows"
    ms.devlang="na"
    ms.topic="article"
    ms.date="10/19/2016"
    wacn.date="03/06/2017"
    ms.author="drewm" />

# 调整 Windows VM 的大小
本文说明如何使用 Azure Powershell 调整在 Resource Manager 部署模型中创建的 Windows VM 的大小。

创建虚拟机 (VM) 后，可以通过更改 VM 大小来扩展或缩减 VM。在某些情况下，必须先解除分配 VM。如果新大小在当前托管 VM 的硬件群集上不可用，则可能会出现这种情况。

## 调整不在可用性集中的 Windows VM 的大小
1. 列出托管 VM 的硬件群集上可用的 VM 大小。

        Get-AzureRmVMSize -ResourceGroupName <resourceGroupName> -VMName <vmName> 

2. 如果列出了所需大小，请运行以下命令来调整 VM 的大小。如果未列出所需大小，请转到步骤 3。

        $vm = Get-AzureRmVM -ResourceGroupName <resourceGroupName> -VMName <vmName>
        $vm.HardwareProfile.VmSize = "<newVMsize>"
        Update-AzureRmVM -VM $vm -ResourceGroupName <resourceGroupName>

3. 如果未列出所需大小，请运行以下命令来解除分配 VM、调整其大小，然后将它重新启动。

        $rgname = "<resourceGroupName>"
        $vmname = "<vmName>"
        Stop-AzureRmVM -ResourceGroupName $rgname -VMName $vmname -Force
        $vm = Get-AzureRmVM -ResourceGroupName $rgname -VMName $vmname
        $vm.HardwareProfile.VmSize = "<newVMSize>"
        Update-AzureRmVM -VM $vm -ResourceGroupName $rgname
        Start-AzureRmVM -ResourceGroupName $rgname -Name $vmname

> [AZURE.WARNING]
解除分配 VM 会释放分配给该 VM 的所有动态 IP 地址。OS 和数据磁盘不受影响。
> 
> 

## 调整可用性集中的 Windows VM 的大小
如果可用性集中 VM 的新大小在当前托管 VM 的硬件群集上不可用，则将需要解除分配可用性集中的所有 VM 以调整 VM 大小。已调整一个 VM 的大小后，可能还需要更新可用性集中其他 VM 的大小。若要调整可用性集中 VM 的大小，请执行以下步骤。

1. 列出托管 VM 的硬件群集上可用的 VM 大小。

        Get-AzureRmVMSize -ResourceGroupName <resourceGroupName> -VMName <vmName>

2. 如果列出了所需大小，请运行以下命令来调整 VM 的大小。如果未列出所需大小，请转到步骤 3。

        $vm = Get-AzureRmVM -ResourceGroupName <resourceGroupName> -VMName <vmName>
        $vm.HardwareProfile.VmSize = "<newVmSize>"
        Update-AzureRmVM -VM $vm -ResourceGroupName <resourceGroupName>

3. 如果未列出所需大小，则继续执行以下步骤以解除分配可用性集中的所有 VM、调整 VM 大小，然后重新启动 VM。
4. 停止可用性集中的所有 VM。

        $rg = "<resourceGroupName>"
        $as = Get-AzureRmAvailabilitySet -ResourceGroupName $rg
        $vmIds = $as.VirtualMachinesReferences
        foreach ($vmId in $vmIDs){
            $string = $vmID.Id.Split("/")
            $vmName = $string[8]
            Stop-AzureRmVM -ResourceGroupName $rg -Name $vmName -Force
        } 

5. 调整可用性集中 VM 的大小并重新启动 VM。

        $rg = "<resourceGroupName>"
        $newSize = "<newVmSize>"
        $as = Get-AzureRmAvailabilitySet -ResourceGroupName $rg
        $vmIds = $as.VirtualMachinesReferences
        foreach ($vmId in $vmIDs){
            $string = $vmID.Id.Split("/")
            $vmName = $string[8]
            $vm = Get-AzureRmVM -ResourceGroupName $rg -Name $vmName
            $vm.HardwareProfile.VmSize = $newSize
            Update-AzureRmVM -ResourceGroupName $rg -VM $vm
            Start-AzureRmVM -ResourceGroupName $rg -Name $vmName
        }

<!---HONumber=Mooncake_1212_2016-->