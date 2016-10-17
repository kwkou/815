<properties
	pageTitle="更改 VM 可用性集 | Azure"
	description="了解如何使用 Azure PowerShell 和 Resource Manager 部署模型更改虚拟机的可用性集。"
	keywords=""
	services="virtual-machines-windows"
	documentationCenter=""
	authors="Drewm3"
	manager="timlt"
	editor=""
	tags="azure-resource-manager"/>  

<tags
	ms.service="virtual-machines-windows"
	ms.workload="infrastructure-services"
	ms.tgt_pltfrm="vm-windows"
	ms.devlang="na"
	ms.topic="article"
	ms.date="09/15/2016"
	wacn.date="10/17/2016"
	ms.author="drewm"/>  




# 更改 Windows VM 的可用性集

以下步骤说明如何使用 Azure PowerShell 来更改 VM 的可用性集。只能在创建 VM 时将 VM 添加到可用性集。若要更改可用性集，必须将虚拟机删除，然后重新创建虚拟机。

## 使用 PowerShell 更改可用性集

1. 从要修改的 VM 中捕获以下重要详细信息。

	VM 的名称

		$vm = Get-AzureRmVM -ResourceGroupName <Name-of-resource-group> -Name <name-of-VM>
		$vm.Name

	VM 大小

		$vm.HardwareProfile.VmSize

	网络主要网络接口和可选的网络接口（如果在 VM 上存在）

		$vm.NetworkProfile.NetworkInterfaces[0].Id

	OS 磁盘配置文件

		$vm.StorageProfile.OsDisk.OsType
		$vm.StorageProfile.OsDisk.Name
		$vm.StorageProfile.OsDisk.Vhd.Uri

	每个数据磁盘的磁盘配置文件

		$vm.StorageProfile.DataDisks[<index>].Lun
		$vm.StorageProfile.DataDisks[<index>].Vhd.Uri

	已安装的 VM 扩展

		$vm.Extensions

2. 删除 VM 但不删除任何磁盘或网络接口。

		Remove-AzureRmVM -ResourceGroupName <resourceGroupName> -Name <vmName> 

3. 创建可用性集（如果尚不存在）

		New-AzureRmAvailabilitySet -ResourceGroupName <resourceGroupName> -Name <availabilitySetName> -Location "<location>" 

4. 使用新可用性集重新创建 VM

		$vm2 = New-AzureRmVMConfig -VMName <VM-name> -VMSize <vm-size> -AvailabilitySetId <availability-set-id>

		Set-AzureRmVMOSDisk -CreateOption "Attach" -VM <vmConfig> -VhdUri <osDiskURI> -Name <osDiskName> [-Windows | -Linux]

		Add-AzureRmVMNetworkInterface -VM <vmConfig> -Id  <nicId> 

		New-AzureRmVM -ResourceGroupName <resourceGroupName> -Location <location> -VM <vmConfig>

5. 添加数据磁盘和扩展。有关详细信息，请参阅 [Attach Data Disk to VM](/documentation/articles/virtual-machines-windows-attach-disk-portal/)（将数据磁盘附加到 VM）和 [Extension Configuration Samples](/documentation/articles/virtual-machines-windows-extensions-configuration-samples/)（扩展配置示例）。可以使用 PowerShell 或 Azure CLI 将数据磁盘和扩展添加到 VM。

## 示例脚本

以下脚本提供一个示例，该示例收集所需的信息、删除原始 VM，然后在新可用性集中重新创建 VM。

	#set variables
	$rg = "demo-resource-group"
	$vmName = "demo-vm"
	$newAvailSetName = "demo-as"
	$outFile = "C:\temp\outfile.txt"

	#Get VM Details
	$OriginalVM = get-azurermvm -ResourceGroupName $rg -Name $vmName

	#Output VM details to file
	"VM Name: " | Out-File -FilePath $outFile 
	$OriginalVM.Name | Out-File -FilePath $outFile -Append

	"Extensions: " | Out-File -FilePath $outFile -Append
	$OriginalVM.Extensions | Out-File -FilePath $outFile -Append

	"VMSize: " | Out-File -FilePath $outFile -Append
	$OriginalVM.HardwareProfile.VmSize | Out-File -FilePath $outFile -Append

	"NIC: " | Out-File -FilePath $outFile -Append
	$OriginalVM.NetworkProfile.NetworkInterfaces[0].Id | Out-File -FilePath $outFile -Append

	"OSType: " | Out-File -FilePath $outFile -Append
	$OriginalVM.StorageProfile.OsDisk.OsType | Out-File -FilePath $outFile -Append

	"OS Disk: " | Out-File -FilePath $outFile -Append
	$OriginalVM.StorageProfile.OsDisk.Vhd.Uri | Out-File -FilePath $outFile -Append

	if ($OriginalVM.StorageProfile.DataDisks) {
    "Data Disk(s): " | Out-File -FilePath $outFile -Append
    $OriginalVM.StorageProfile.DataDisks | Out-File -FilePath $outFile -Append
	}

	#Remove the original VM
	Remove-AzureRmVM -ResourceGroupName $rg -Name $vmName

	#Create new availability set if it does not exist
	$availSet = Get-AzureRmAvailabilitySet -ResourceGroupName $rg -Name $newAvailSetName -ErrorAction Ignore
	if (-Not $availSet) {
    $availset = New-AzureRmAvailabilitySet -ResourceGroupName $rg -Name $newAvailSetName -Location $OriginalVM.Location
	}

	#Create the basic configuration for the replacement VM
	$newVM = New-AzureRmVMConfig -VMName $OriginalVM.Name -VMSize $OriginalVM.HardwareProfile.VmSize -AvailabilitySetId $availSet.Id
	Set-AzureRmVMOSDisk -VM $NewVM -VhdUri $OriginalVM.StorageProfile.OsDisk.Vhd.Uri  -Name $OriginalVM.Name -CreateOption Attach -Windows

	#Add Data Disks
	foreach ($disk in $OriginalVM.StorageProfile.DataDisks ) { 
    Add-AzureRmVMDataDisk -VM $newVM -Name $disk.Name -VhdUri $disk.Vhd.Uri -Caching $disk.Caching -Lun $disk.Lun -CreateOption Attach -DiskSizeInGB $disk.DiskSizeGB
	}

	#Add NIC(s)
	foreach ($nic in $OriginalVM.NetworkInterfaceIDs) {
		Add-AzureRmVMNetworkInterface -VM $NewVM -Id $nic
	}

	#Create the VM
	New-AzureRmVM -ResourceGroupName $rg -Location $OriginalVM.Location -VM $NewVM -DisableBginfoExtension

## 后续步骤

通过添加附加[数据磁盘](/documentation/articles/virtual-machines-windows-attach-disk-portal/)，向 VM 添加附加存储。

<!---HONumber=Mooncake_1010_2016-->