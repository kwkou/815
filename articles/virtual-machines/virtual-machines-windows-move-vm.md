<properties
	pageTitle="移动 Windows VM | Azure"
	description="在 Resource Manager 部署模型中将 Windows VM 移到其他 Azure 订阅或资源组。"
	services="virtual-machines-windows"
	documentationCenter=""
	authors="cynthn"
	manager="timlt"
	editor=""
	tags="azure-resource-manager"/>

<tags
	ms.service="virtual-machines-windows"
	ms.workload="infrastructure-services"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="08/08/2016"
	wacn.date="12/26/2016"
	ms.author="cynthn"/>  


	


# 将 Windows VM 移到其他 Azure 订阅或资源组 

本文逐步说明如何在资源组或订阅之间移动 Windows VM。如果最初在个人订阅中创建了 VM，现在想要将其移到公司的订阅以继续工作，则在订阅之间移动 VM 会很方便。

> [AZURE.NOTE] 在移动过程中将创建新的资源 ID。移动 VM 后，需要更新工具和脚本以使用新的资源 ID。


[AZURE.INCLUDE [virtual-machines-common-move-vm](../../includes/virtual-machines-common-move-vm.md)]


## 使用 PowerShell 移动 VM

若要将虚拟机移到其他资源组，需确保同时移动所有依赖资源。若要使用 Move-AzureRMResource cmdlet，需要资源的名称和类型。可以通过 Find-AzureRMResource cmdlet 获取这两项信息。

	Find-AzureRMResource -ResourceGroupNameContains "<sourceResourceGroupName>"
	

若要移动 VM，需要移动多个资源。只需分别创建每个资源的变量，然后列出这些变量即可。本示例包括 VM 的大多数基本资源，但可以根据需要添加更多资源。

	$sourceRG = "<sourceResourceGroupName>"
	$destinationRG = "<destinationResourceGroupName>"
    
	$vm = Get-AzureRmResource -ResourceGroupName $sourceRG -ResourceType "Microsoft.Compute/virtualMachines" -ResourceName "<vmName>"
    $storageAccount = Get-AzureRmResource -ResourceGroupName $sourceRG -ResourceType "Microsoft.Storage/storageAccounts" -ResourceName "<storageAccountName>"
	$diagStorageAccount = Get-AzureRmResource -ResourceGroupName $sourceRG -ResourceType "Microsoft.Storage/storageAccounts" -ResourceName "<diagnosticStorageAccountName>"
	$vNet = Get-AzureRmResource -ResourceGroupName $sourceRG -ResourceType "Microsoft.Network/virtualNetworks" -ResourceName "<vNetName>"
	$nic = Get-AzureRmResource -ResourceGroupName $sourceRG -ResourceType "Microsoft.Network/networkInterfaces" -ResourceName "<nicName>"
	$ip = Get-AzureRmResource -ResourceGroupName $sourceRG -ResourceType "Microsoft.Network/publicIPAddresses" -ResourceName "<ipName>"
	$nsg = Get-AzureRmResource -ResourceGroupName $sourceRG -ResourceType "Microsoft.Network/networkSecurityGroups" -ResourceName "<nsgName>"
	
	Move-AzureRmResource -DestinationResourceGroupName $destinationRG -ResourceId $vm.ResourceId, $storageAccount.ResourceId, $diagStorageAccount.ResourceId, $vNet.ResourceId, $nic.ResourceId, $ip.ResourceId, $nsg.ResourceId

若要将资源移到其他订阅，请添加 **-DestinationSubscriptionId** 参数。

	Move-AzureRmResource -DestinationSubscriptionId "<destinationSubscriptionID>" -DestinationResourceGroupName $destinationRG -ResourceId $vm.ResourceId, $storageAccount.ResourceId, $diagStorageAccount.ResourceId, $vNet.ResourceId, $nic.ResourceId, $ip.ResourceId, $nsg.ResourceId



系统会要求你确认你需要移动指定资源。请键入 **Y** 确认要移动资源。

  
## 后续步骤

可以在资源组和订阅之间移动许多不同类型的资源。有关详细信息，请参阅[将资源移到新的资源组或订阅](/documentation/articles/resource-group-move-resources/)。

<!---HONumber=Mooncake_Quality_Review_1215_2016-->