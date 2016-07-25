<!-- ARM: tested -->

<properties
	pageTitle="创建 Linux VM 的副本 | Azure"
	description="了解如何通过创建一个 *专用映像*，在 Resource Manager 部署模型中创建运行 Linux 的 Azure 虚拟机。"
	services="virtual-machines-linux"
	documentationCenter=""
	authors="cynthn"
	manager="timlt"
	editor=""
	tags="azure-resource-manager"/>

<tags
	ms.service="virtual-machines-linux"
	ms.date="04/26/2016"
	wacn.date="07/25/2016"/>

# 在 Azure Resource Manager 部署模型中创建 Linux 虚拟机的副本


本文说明如何创建运行 Linux 的 Azure 虚拟机 (VM) 的副本。具体而言，它介绍如何使用 Azure CLI 和 Azure 门户预览在 Resource Manager 部署模型中执行此操作。其中说明了如何创建 Azure VM 的*专用*映像，此映像可维护用户帐户和其他来自原始 VM 的状态数据。使用专用映像可将 Linux VM 从经典部署模型移植到 Resource Manager 部署模型，或为创建于 Resource Manager 部署模型中的 Linux VM 创建备份副本。你可以使用此方法来通过操作系统和数据磁盘进行复制，然后设置网络资源以创建新虚拟机。

如果需要创建相似 Linux VM 的批量部署，则应该使用“通用”映像。相关信息请参阅[如何捕获 Linux 虚拟机](/documentation/articles/virtual-machines-linux-capture-image/)。



## 开始之前

在开始执行相关步骤前，请先确保符合以下先决条件：

- 你有使用经典或 Resource Manager 部署模型创建的运行 Linux 的 Azure 虚拟机。你已配置操作系统、附加数据磁盘和进行诸如安装所需应用程序之类的其他自定义设置。你将使用此 VM 创建副本。如需有关创建源 VM 的帮助，请参阅[在 Azure 中创建 Linux VM](/documentation/articles/virtual-machines-linux-quick-create-cli/)。

- 你已在计算机上下载和安装 Azure CLI，并已登录 Azure 订阅。有关详细信息，请参阅[如何安装 Azure CLI](/documentation/articles/xplat-cli-install/)。

- 你有资源组、存储帐户以及在该资源组中创建的用于将 VHD 复制到的 Blob 容器。有关创建存储帐户和 Blob 容器的详细信息，请参阅[将 Azure CLI 与 Azure 存储空间搭配使用](/documentation/articles/storage-azure-cli/)。

> [AZURE.NOTE] 对于使用两种部署模型之一作为源映像创建的 VM，可以采用类似的步骤。在适用的情况下，本文会注明细微的差异。


## 将 VHD 复制到 Resource Manager 存储帐户

[AZURE.INCLUDE [arm-api-version-cli](../includes/arm-api-version-cli.md)]

1. 执行以下两个选项之一，以释放源 VM 使用的 VHD：

	- 如果你想要复制源虚拟机，请停止并解除分配该虚拟机。在门户中，单击“浏览”>“虚拟机”或“虚拟机(经典)”> “你的 VM” >“停止”。对于在 Resource Manager 部署模型中创建的 VM，也可以使用 Azure CLI 命令 `azure vm stop <yourResourceGroup> <yourVmName>`，后接 `azure vm deallocate <yourResourceGroup> <yourVmName>`。请注意，门户中的 VM 状态将从“正在运行”更改为“已停止(已解除分配)”。

	- 如果你想要迁移源虚拟机，请删除该 VM 并使用剩余的 VHD。在[门户](https://portal.azure.cn)中浏览到虚拟机，然后单击“删除”。

1. 找到包含源 VHD 的存储帐户的访问密钥。有关访问密钥的详细信息，请参阅[关于 Azure 存储帐户](/documentation/articles/storage-create-storage-account/)。

	- 如果源 VM 是使用经典部署模型创建的，请单击“浏览”>“存储帐户(经典)”> “你的存储帐户” >“配置”>“密钥”。复制标记为 **PRIMARY ACCESS KEY** 的密钥。或者，在 Azure CLI 中使用 `azure config mode asm` 更改为经典模式，然后使用 `azure storage account keys list <yourSourceStorageAccountName>`。

	- 如果源 VM 是使用 Resource Manager 部署模型创建的，请单击“浏览”>“存储帐户”> “你的存储帐户” >“配置”>“访问密钥”。复制标记为 **key1** 的文本。或者，在 Azure CLI 中使用 `azure config mode arm` 确保处于 Resource Manager 模式，然后使用 `azure storage account keys list -g <yourDestinationResourceGroup> <yourDestinationStorageAccount>`。

1. 使用[适用于存储空间的 Azure CLI 命令](/documentation/articles/storage-azure-cli/)来复制 VHD 文件，如以下步骤中所述。或者，如果你偏向于使用用户界面，可以改用 [Azure 存储空间资源管理器](http://storageexplorer.com/)。
</br>
	1. 设置目标存储帐户的连接字符串。此连接字符串包含此存储帐户的访问密钥。

			$azure storage account connectionstring show -g <yourDestinationResourceGroup> <yourDestinationStorageAccount>
			$export AZURE_STORAGE_CONNECTION_STRING=<the_connectionstring_output_from_above_command>

	2. 为源存储帐户中的 VHD 文件创建[共享访问签名](/documentation/articles/storage-dotnet-shared-access-signature-part-1/)。记下以下命令的**共享访问 URL** 输出。

			$azure storage blob sas create  --account-name <yourSourceStorageAccountName> --account-key <SourceStorageAccessKey> --container <SourceStorageContainerName> --blob <FileNameOfTheVHDtoCopy> --permissions "r" --expiry <mm/dd/yyyy_when_you_want_theSAS_to_expire>

	3. 使用以下命令将 VHD 从源存储复制到目标位置。

			$azure storage blob copy start <SharedAccessURL_ofTheSourceVHD> <DestinationContainerName>

	4. 将以异步方式复制 VHD 文件。请使用以下命令查看进度。

			$azure storage blob copy show <DestinationContainerName> <FileNameOfTheVHDtoCopy>

</br>

>[AZURE.NOTE] 如前面所述，你应该分别复制操作系统和数据磁盘。


## 使用复制的 VHD 创建 VM

通过使用上述步骤中复制的 VHD，现在可以使用 Azure CLI 在新虚拟网络中创建基于 Resource Manager 的 Linux VM。VHD 应与要创建的新虚拟机位于同一存储帐户中。


使用类似于下面的脚本为新 VM 设置虚拟网络和 NIC。根据应用程序使用适当的变量值。

	$azure network vnet create <yourResourceGroup> <yourVnetName> -l <yourLocation>

	$azure network vnet subnet create <yourResourceGroup> <yourVnetName> <yourSubnetName>

	$azure network public-ip create <yourResourceGroup> <yourIpName> -l <yourLocation>

	$azure network nic create <yourResourceGroup> <yourNicName> -k <yourSubnetName> -m <yourVnetName> -p <yourIpName> -l <yourLocation>


通过运行以下命令，使用复制的 VHD 创建新虚拟机。</br>

	$azure vm create -g <yourResourceGroup> -n <yourVmName> -f <yourNicName> -d <UriOfYourOsDisk> -x <UriOfYourDataDisk> -e <DataDiskSizeGB> -Y -l <yourLocation> -y Linux -z "Standard_A1" -o <DestinationStorageAccountName> -R <DestinationStorageAccountBlobContainer>


数据磁盘和操作系统磁盘的 URL 类似于：`https://StorageAccountName.blob.core.chinacloudapi.cn/BlobContainerName/DiskName.vhd`。可通过以下方法在门户上找到此信息：浏览到存储容器，单击复制的操作系统或数据 VHD，然后复制 URL 的内容。


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

你应该会在 [Azure 门户预览](https://portal.azure.cn)的“浏览”>“虚拟机”下看到新建的 VM。

使用所选的 SSH 客户端连接到新虚拟机，然后使用原始虚拟机的帐户凭据（例如 `ssh OldAdminUser@<IPaddressOfYourNewVM>`）。有关详细信息，请参阅[如何在 Azure 中将 SSH 用于 Linux](/documentation/articles/virtual-machines-linux-ssh-from-linux/)。


## 后续步骤

若要了解如何使用 Azure CLI 来管理新虚拟机，请参阅 [Azure Resource Manager 的 Azure CLI 命令](/documentation/articles/azure-cli-arm-commands/)。

<!---HONumber=Mooncake_0718_2016-->