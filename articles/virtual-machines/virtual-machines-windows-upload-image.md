<properties
	pageTitle="上载 Windows VHD 用于 Resource Manager | Azure"
	description="了解如何使用 Resource Manager 部署模型将 Windows 虚拟机 VHD 从本地上传到 Azure。可以从通用或专用 VM 上传 VHD。"
	services="virtual-machines-windows"
	documentationCenter=""
	authors="cynthn"
	manager="timlt"
	editor="tysonn"
	tags="azure-resource-manager"/>  


<tags
	ms.service="virtual-machines-windows"
	ms.workload="infrastructure-services"
	ms.tgt_pltfrm="vm-windows"
	ms.devlang="na"
	ms.topic="article"
	ms.date="10/10/2016"
	wacn.date="01/05/2017"
	ms.author="cynthn"/>  


# 从本地 VM 将 Windows VHD 上传到 Azure 


本文介绍如何创建和上传用于创建 Azure VM 的 Windows 虚拟硬盘 (VHD)。可以从通用 VM 或专用 VM 上传 VHD。

有关 Azure 中的磁盘和 VHD 的更多详细信息，请参阅 [About disks and VHDs for virtual machines](/documentation/articles/virtual-machines-linux-about-disks-vhds/)（关于虚拟机的磁盘和 VHD）。


## 准备 VM 

可以将通用和专用 VHD 上传到 Azure。每种类型都需要在开始之前准备 VM。

- **通用 VHD** - 通用 VHD 包含使用 Sysprep 删除所有个人帐户信息。如果想要使用 VHD 作为映像来创建新的 VM，应该：
	- [准备好要上传到 Azure 的 Windows VHD](/documentation/articles/virtual-machines-windows-prepare-for-upload-vhd-image/)。
	- [使用 Sysprep 通用化虚拟机](/documentation/articles/virtual-machines-windows-generalize-vhd/)。

- **专用 VHD** - 专用 VHD 保留原始 VM 中的用户帐户、应用程序和其他状态数据。如果想要使用当前 VHD 创建新 VM，请确保完成以下步骤。
	- [准备好要上传到 Azure 的 Windows VHD](/documentation/articles/virtual-machines-windows-prepare-for-upload-vhd-image/)。**不要**使用 Sysprep 通用化 VM。
	- 删除安装于 VM 上的任何来宾虚拟化工具和代理，即 VMware 工具。
	- 确保 VM 配置为通过 DHCP 来提取其 IP 地址和 DNS 设置。这确保服务器在启动时在 VNet 中获取 IP 地址。

## 登录到 Azure

如果尚未安装 Azure PowerShell 1.4 版或更高版本，请阅读[如何安装和配置 Azure PowerShell](https://docs.microsoft.com/powershell/azureps-cmdlets-docs)。

1. 打开 Azure PowerShell 并登录到 Azure 帐户。将打开一个弹出窗口，可以输入 Azure 帐户凭据。

		Login-AzureRmAccount -EnvironmentName AzureChinaCloud

2. 获取可用订阅的订阅 ID。

		Get-AzureRmSubscription

3. 使用订阅 ID 设置正确的订阅。将 `<subscriptionID>` 替换为正确订阅的 ID。

		Select-AzureRmSubscription -SubscriptionId "<subscriptionID>"

## <a name="createstorage"></a> 获取存储帐户

需要在 Azure 中创建存储帐户来存储上传的 VM 映像。可以使用现有存储帐户，也可以创建新存储帐户。

显示可用的存储帐户，请键入：

	Get-AzureRmStorageAccount

如果要使用现有存储帐户，请转到[上载 VM 映像](#upload-the-vm-vhd-to-your-storage-account)部分。

若要创建存储帐户，请执行以下步骤：

1. 需要应在其中创建存储帐户的资源组的名称。若要查找订阅中的所有资源组，请键入：

		Get-AzureRmResourceGroup

	若要在**中国北部**区域中创建名称为 **myResourceGroup** 的资源组，请键入：

		New-AzureRmResourceGroup -Name myResourceGroup -Location "China North"

2. 使用 [New-AzureRmStorageAccount](https://msdn.microsoft.com/zh-cn/library/mt607148.aspx) cmdlet 在此资源组中创建名称为 **mystorageaccount** 存储帐户：

		New-AzureRmStorageAccount -ResourceGroupName myResourceGroup -Name mystorageaccount -Location "China North" -SkuName "Standard_LRS" -Kind "Storage"
			
	-SkuName 的有效值为：

	- **Standard\_LRS** - 本地冗余存储。
	- **Standard\_ZRS** - 区域冗余存储。
	- **Standard\_GRS** - 异地冗余存储。
	- **Standard\_RAGRS** - 读取访问权限异地冗余存储。
	- **Premium\_LRS** - 高级本地冗余存储。



## <a name="upload-the-vm-vhd-to-your-storage-account"></a> 将 VHD 上传到存储帐户

使用 [Add-AzureRmVhd](https://msdn.microsoft.com/zh-cn/library/mt603554.aspx) cmdlet 将映像上传到存储帐户中的容器。此示例将文件 **myVHD.vhd** 从 `"C:\Users\Public\Documents\Virtual hard disks"` 上传到 **myResourceGroup** 资源组中名为 **mystorageaccount** 的存储帐户。该文件将放到名为 **mycontainer** 的容器，新的文件名将是 **myUploadedVHD.vhd**。

```powershell
$rgName = "myResourceGroup"
$urlOfUploadedImageVhd = "https://mystorageaccount.blob.core.chinacloudapi.cn/mycontainer/myUploadedVHD.vhd"
Add-AzureRmVhd -ResourceGroupName $rgName -Destination $urlOfUploadedImageVhd -LocalFilePath "C:\Users\Public\Documents\Virtual hard disks\myVHD.vhd"
```


如果成功，显示如下所示的响应：

	  C:\> Add-AzureRmVhd -ResourceGroupName myResourceGroup -Destination https://mystorageaccount.blob.core.chinacloudapi.cn/mycontainer/myUploadedVHD.vhd -LocalFilePath "C:\Users\Public\Documents\Virtual hard disks\myVHD.vhd"
	  MD5 hash is being calculated for the file C:\Users\Public\Documents\Virtual hard disks\myVHD.vhd.
	  MD5 hash calculation is completed.
	  Elapsed time for the operation: 00:03:35
	  Creating new page blob of size 53687091712...
	  Elapsed time for upload: 01:12:49

	  LocalFilePath           DestinationUri
	  -------------           --------------
	  C:\Users\Public\Doc...  https://mystorageaccount.blob.core.chinacloudapi.cn/mycontainer/myUploadedVHD.vhd

完成执行此命令可能需要一段时间，具体取决于网络连接速度和 VHD 文件的大小


## 后续步骤

- [在 Azure 中从通用 VHD 映像创建 VM](/documentation/articles/virtual-machines-windows-create-vm-generalized/)
- 创建新 VM 时，通过将其作为 OS 磁盘进行附加从而[在 Azure 中从专用 VHD 创建 VM](/documentation/articles/virtual-machines-windows-create-vm-specialized/)。

<!---HONumber=Mooncake_1121_2016-->