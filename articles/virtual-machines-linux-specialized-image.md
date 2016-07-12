<!-- Ibiza Portal -->

<properties
	pageTitle="创建 Linux VM 的副本 | Azure"
	description="了解如何通过创建一个 *专用映像*，在 Resource Manager 部署模型中创建运行 Linux 的 Azure 虚拟机。"
	services="virtual-machines-linux"
	documentationCenter=""
	authors="dsk-2015"
	manager="timlt"
	editor=""
	tags="azure-resource-manager"/>

<tags
	ms.service="virtual-machines-linux"
	ms.date="04/26/2016"
	wacn.date="06/20/2016"/>

# 如何在 Resource Manager 部署模型中创建 Linux 虚拟机的副本



本文说明如何使用 Azure CLI 与 Azure 门户预览在 Resource Manager 部署模型中创建运行 Linux 的 Azure 虚拟机 (VM) 副本。其中说明如何创建 Azure VM 的**_专用_**映像，此映像可维护用户帐户和其他来自原始 VM 的状态数据。使用专用映像可将 Linux VM 从经典部署模型移植到 Resource Manager 部署模型，或为创建于 Resource Manager 部署模型中的 Linux VM 创建备份副本。你可以使用此方法来通过 OS 和数据磁盘进行复制，然后设置网络资源以创建新虚拟机。

如果需要创建类似于 Linux VM 的批量部署，应该使用*通用*映像。有关信息请参阅 [How to capture a Linux virtual machine（如何捕获 Linux 虚拟机）](/documentation/articles/virtual-machines-linux-capture-image/)。



## 开始前请检查以下先决条件

本文假定你具备以下条件：

1. 经典或 Resource Manager 部署模型中**运行 Linux 的 Azure 虚拟机**，其上已配置操作系统、已附加数据磁盘并且安装了所需的应用程序。如需有关创建此 VM 的帮助，请阅读 [Create a Linux VM in Azure（在 Azure 中创建 Linux VM）](/documentation/articles/virtual-machines-linux-quick-create-cli/)。 

1. 已在计算机上安装 **Azure CLI** 并且你已登录到 Azure 订阅。有关详细信息，请阅读 [How to install Azure CLI（如何安装 Azure CLI）](/documentation/articles/xplat-cli-install/)。

1. 一个**资源组**，其中创建了**存储帐户**和 **Blob 容器**。VHD 将复制到该资源组。有关详细信息，请阅读 [Using the Azure CLI with Azure Storage（将 Azure CLI 用于 Azure 存储空间）](/documentation/articles/storage-azure-cli/)。



> [AZURE.NOTE] 对于使用两种部署模型之一作为源映像创建的 VM 可以应用类似的步骤。在适当的情况下，本文会注明细微的差别。


## 将 VHD 复制到 Resource Manager 存储帐户

[AZURE.INCLUDE [arm-api-version-cli](../includes/arm-api-version-cli.md)]

1. 先执行以下两个选项之一，以释放源 VM 使用的 VHD：

	- 如果你想要**_复制_**源虚拟机，请**停止**并**解除分配**该虚拟机。在门户预览中，单击“浏览”>“虚拟机”或“虚拟机(经典)”> *你的 VM* >“停止”。对于在 Resource Manager 部署模型中创建的 VM，也可以使用 Azure CLI 命令 `azure vm stop <yourResourceGroup> <yourVmName>` 后接 `azure vm deallocate <yourResourceGroup> <yourVmName>`。请注意，门户预览中的 VM“状态”将从“正在运行”更改为“已停止(已解除分配)”。
	
	- 如果你想要**_迁移_**源虚拟机，请**删除**该 VM 并使用剩余的 VHD。在[门户预览](https://portal.azure.cn)中**浏览**到虚拟机并单击“删除”。
	
1. 找到包含源 VHD 的存储帐户的访问密钥。有关访问密钥的详细信息，请阅读 [About Azure storage accounts（关于 Azure 存储帐户）](/documentation/articles/storage-create-storage-account/)。

	- 如果源 VM 是使用经典部署模型创建的，请单击“浏览”>“存储帐户(经典)”> *你的存储帐户* >“配置”>“密钥”，然后复制标签为“主访问密钥”的密钥。或者，在 Azure CLI 中使用 `azure config mode asm` 更改为经典模式，然后使用 `azure storage account keys list <yourSourceStorageAccountName>`。

	- 对于使用 Resource Manager 部署模型所创建的源 VM，请单击“浏览”>“存储帐户”> *你的存储帐户* >“配置”>“访问密钥”，然后复制标签为“key1”的文本。或者，在 Azure CLI 中使用 `azure config mode arm` 确保处于 Resource Manager 模式，然后使用 `azure storage account keys list -g <yourDestinationResourceGroup> <yourDestinationStorageAccount>`。

1. 使用[适用于存储空间的 Azure CLI 命令](/documentation/articles/storage-azure-cli/)来复制 VHD 文件，如以下步骤中所述。或者，如果你偏向于使用 UI 来实现相同的效果，可以改用 [Azure 存储空间资源管理器](http://storageexplorer.com/)。
</br>
	1. 设置目标存储帐户的连接字符串。此连接字符串包含此存储帐户的访问密钥。
	
			$azure storage account connectionstring show -g <yourDestinationResourceGroup> <yourDestinationStorageAccount>
			$export AZURE_STORAGE_CONNECTION_STRING=<the_connectionstring_output_from_above_command>
	
	2. 为源存储帐户中的 VHD 文件创建[共享访问签名](/documentation/articles/storage-dotnet-shared-access-signature-part-1/)。记下以下命令的**共享访问 URL** 输出。
	
			$azure storage blob sas create  --account-name <yourSourceStorageAccountName> --account-key <SourceStorageAccessKey> --container <SourceStorageContainerName> --blob <FileNameOfTheVHDtoCopy> --permissions "r" --expiry <mm/dd/yyyy_when_you_want_theSAS_to_expire>
	
	3. 使用以下命令将 VHD 从源存储复制到目标。
	
			$azure storage blob copy start <SharedAccessURL_ofTheSourceVHD> <DestinationContainerName>
	
	4. 将以异步方式复制 VHD 文件。可以使用以下命令查看进度。
	
			$azure storage blob copy show <DestinationContainerName> <FileNameOfTheVHDtoCopy>
		
</br>

>[AZURE.NOTE] 如前所述，需要分别复制 OS 磁盘和数据磁盘。


## 使用复制的 VHD 创建 VM

以下步骤说明如何使用上述步骤中复制的 VHD 通过 Azure CLI 在新虚拟网络中创建基于 Resource Manager 的 Linux VM。VHD 应与要创建的新虚拟机位于同一存储帐户中。


首先，使用类似于下面的脚本为新 VM 设置虚拟网络和 NIC。根据应用程序使用适当的变量值。

	$azure network vnet create <yourResourceGroup> <yourVnetName> -l <yourLocation>

	$azure network vnet subnet create <yourResourceGroup> <yourVnetName> <yourSubnetName>

	$azure network public-ip create <yourResourceGroup> <yourIpName> -l <yourLocation>

	$azure network nic create <yourResourceGroup> <yourNicName> -k <yourSubnetName> -m <yourVnetName> -p <yourIpName> -l <yourLocation>


现在，请运行以下命令，使用复制的 VHD 创建新虚拟机。</br>

	$azure vm create -g <yourResourceGroup> -n <yourVmName> -f <yourNicName> -d <UriOfYourOsDisk> -x <UriOfYourDataDisk> -e <DataDiskSizeGB> -Y -l <yourLocation> -y Linux -z "Standard_A1" -o <DestinationStorageAccountName> -R <DestinationStorageAccountBlobContainer>

	
数据磁盘和 OS 磁盘的 URL 类似于：`https://StorageAccountName.blob.core.chinacloudapi.cn/BlobContainerName/DiskName.vhd`。可通过以下方法在门户预览上找到此信息：浏览到存储容器，单击复制的 OS 或数据 VHD，然后复制 **URL** 的内容。
	
	
如果此命令成功，你会看到类似于下面的输出：

	$azure vm create -g "testcopyRG" -n "redhatcopy" -f "LinCopyNic" -d https://testcopystore.blob.core.chinacloudapi.cn/testcopyblob/RedHat201631816334.vhd -x https://testcopystore.blob.core.chinacloudapi.cn/testcopyblob/RedHat-data.vhd -e 10 -Y -l "China North" -y Linux -z "Standard_A1" -o "testcopystore" -R "testcopyblob"
	info:    Executing command vm create
	+ Looking up the VM "redhatcopy"
	info:    Using the VM Size "Standard_A1"
	+ Looking up the NIC "LinCopyNic"
	info:    Found an existing NIC "LinCopyNic"
	info:    Found an IP configuration with virtual network subnet id "/subscriptions/b8e6a92b-d6b7-4dbb-87a8-3c8dfe248156/resourceGroups/testcopyRG/providers/Microsoft.Network/virtualNetworks/LinCopyVnet/subnets/LinCopySub" in the NIC "LinCopyNic"
	info:    This NIC IP configuration has a public ip already configured "/subscriptions/b8e6a92b-d6b7-4dbb-87a8-3c8dfe248156/resourcegroups/testcopyrg/providers/microsoft.network/publicipaddresses/lincopyip", any public ip parameters if provided, will be ignored.
	+ Looking up the storage account testcopystore
	info:    The storage URI 'https://testcopystore.blob.core.chinacloudapi.cn/' will be used for boot diagnostics settings, and it can be overwritten by the parameter input of '--boot-diagnostics-storage-uri'.
	+ Creating VM "redhatcopy"
	info:    vm create command OK

你应会在 [Azure 门户预览](https://portal.azure.cn)的“浏览”>“虚拟机”下看到新建的 VM。

使用所选的 SSH 客户端连接到新虚拟机，然后使用原始虚拟机的帐户凭据（例如 `ssh OldAdminUser@<IPaddressOfYourNewVM>`）。有关通过 SSH 连接到 Linux VM 的详细信息，请阅读 [How to use SSH with Linux on Azure（如何使用 SSH 连接 Azure 中的 Linux）](/documentation/articles/virtual-machines-linux-ssh-from-linux/)。


## 后续步骤

若要了解如何使用 Azure CLI 来管理新虚拟机，请参阅 [Azure CLI commands for the Azure Resource Manager（Azure Resource Manager 的 Azure CLI 命令）](/documentation/articles/azure-cli-arm-commands/)。

<!---HONumber=Mooncake_0613_2016-->