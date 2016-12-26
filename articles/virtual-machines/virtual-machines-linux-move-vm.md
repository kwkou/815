<properties
	pageTitle="移动 Linux VM | Azure"
	description="在 Resource Manager 部署模型中将 Linux VM 移到其他 Azure 订阅或资源组。"
	services="virtual-machines-linux"
	documentationCenter=""
	authors="cynthn"
	manager="timlt"
	editor=""
	tags="azure-resource-manager"/>  


<tags
	ms.service="virtual-machines-linux"
	ms.workload="infrastructure-services"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="08/08/2016"
	wacn.date="12/26/2016"
	ms.author="cynthn"/>

	


# 将 Linux VM 移到其他订阅或资源组

本文逐步说明如何在资源组或订阅之间移动 Linux VM。如果在个人订阅中创建了 VM，现在想要将其移到公司的订阅，则在订阅之间移动 VM 会很方便。

> [AZURE.NOTE] 在移动过程中将创建新的资源 ID。移动 VM 后，需要更新工具和脚本以使用新的资源 ID。


## 使用 Azure CLI 移动 VM 

若要成功移动 VM，需要移动 VM 及其所有支持资源。使用 **azure group show** 命令列出资源组中的所有资源及其 ID。这有助于通过管道将此命令的输出发送到文件，以便将 ID 复制并粘贴到后续命令中。

	azure group show <resourceGroupName>

若要将 VM 及其资源移到其他资源组，请使用 **azure resource move** CLI 命令。以下示例说明如何移动 VM 及其所需的大多数通用资源。我们使用 **-i** 参数，并传入要移动的资源的逗号分隔 ID 列表（不包含空格）。

	
    vm=/subscriptions/<sourceSubscriptionID>/resourceGroups/<sourceResourceGroup>/providers/Microsoft.Compute/virtualMachines/<vmName>
	nic=/subscriptions/<sourceSubscriptionID>/resourceGroups/<sourceResourceGroup>/providers/Microsoft.Network/networkInterfaces/<nicName>
	nsg=/subscriptions/<sourceSubscriptionID>/resourceGroups/<sourceResourceGroup>/providers/Microsoft.Network/networkSecurityGroups/<nsgName>
	pip=/subscriptions/<sourceSubscriptionID>/resourceGroups/<sourceResourceGroup>/providers/Microsoft.Network/publicIPAddresses/<publicIPName>
	vnet=/subscriptions/<sourceSubscriptionID>/resourceGroups/<sourceResourceGroup>/providers/Microsoft.Network/virtualNetworks/<vnetName>
	diag=/subscriptions/<sourceSubscriptionID>/resourceGroups/<sourceResourceGroup>/providers/Microsoft.Storage/storageAccounts/<diagnosticStorageAccountName>
	storage=/subscriptions/<sourceSubscriptionID>/resourceGroups/<sourceResourceGroup>/providers/Microsoft.Storage/storageAccounts/<storageAcountName>  	
	
	azure resource move --ids $vm,$nic,$nsg,$pip,$vnet,$storage,$diag -d "<destinationResourceGroup>"
	
如果要将 VM 及其资源移到其他订阅，请添加 **--destination-subscriptionId &#60;destinationSubscriptionID&#62;** 参数来指定目标订阅。

如果从 Windows 计算机上的命令提示符操作，需要在声明变量名称时在其前面添加 **$**。在 Linux 中不需要这样做。

系统将要求确认是否想要移动指定的资源。请键入 **Y** 确认要移动资源。
	

[AZURE.INCLUDE [virtual-machines-common-move-vm](../../includes/virtual-machines-common-move-vm.md)]

## 后续步骤

可以在资源组和订阅之间移动许多不同类型的资源。有关详细信息，请参阅[将资源移到新的资源组或订阅](/documentation/articles/resource-group-move-resources/)。

<!---HONumber=Mooncake_Quality_Review_1215_2016-->