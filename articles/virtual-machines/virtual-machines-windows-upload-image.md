<properties
    pageTitle="上载 Windows VHD 用于 Resource Manager | Azure"
    description="了解如何使用 Resource Manager 部署模型将 Windows 虚拟机 VHD 从本地上传到 Azure。 可以从通用或专用 VM 上传 VHD。"
    services="virtual-machines-windows"
    documentationcenter=""
    author="cynthn"
    manager="timlt"
    editor="tysonn"
    tags="azure-resource-manager"
    translationtype="Human Translation" />
<tags
    ms.assetid="192c8e5a-3411-48fe-9fc3-526e0296cf4c"
    ms.service="virtual-machines-windows"
    ms.workload="infrastructure-services"
    ms.tgt_pltfrm="vm-windows"
    ms.devlang="na"
    ms.topic="article"
    ms.date="03/06/2017"
    wacn.date="04/17/2017"
    ms.author="cynthn"
    ms.sourcegitcommit="e0e6e13098e42358a7eaf3a810930af750e724dd"
    ms.openlocfilehash="cda212f12074a5fecac7863205cecb7cf9968be8"
    ms.lasthandoff="04/06/2017" />

# <a name="upload-a-windows-vhd-from-an-on-premises-vm-to-azure"></a>将 Windows VHD 从本地 VM 上载到 Azure
本文介绍如何创建和上传用于创建 Azure VM 的 Windows 虚拟硬盘 (VHD)。 可以从通用 VM 或专用 VM 上载 VHD。 

有关如何使用非托管磁盘准备、上载和创建新 VM 的完整演练，请参阅[上载专用 VHD 以在 Azure 中创建 VM](/documentation/articles/virtual-machines-windows-upload-specialized/)。

有关 Azure 中的磁盘和 VHD 的更多详细信息，请参阅 [About disks and VHDs for virtual machines](/documentation/articles/storage-about-disks-and-vhds-windows/)（关于虚拟机的磁盘和 VHD）。

## <a name="prepare-the-vm"></a>准备 VM
可以将通用和专用 VHD 上传到 Azure。 每种类型都需要在开始之前准备 VM。

* **通用 VHD** - 通用 VHD 包含使用 Sysprep 删除所有个人帐户信息。 如果想要使用 VHD 作为映像来创建新的 VM，应该：

    * [准备好要上载到 Azure 的 Windows VHD](/documentation/articles/virtual-machines-windows-prepare-for-upload-vhd-image/)。 
    * [使用 Sysprep 通用化虚拟机](/documentation/articles/virtual-machines-windows-generalize-vhd/)。 
* **专用 VHD** - 专用 VHD 保留原始 VM 中的用户帐户、应用程序和其他状态数据。 如果想要使用当前 VHD 创建新 VM，请确保完成以下步骤。 

    * [准备好要上载到 Azure 的 Windows VHD](/documentation/articles/virtual-machines-windows-prepare-for-upload-vhd-image/)。 **不要**使用 Sysprep 通用化 VM。
    * 删除 VM 上安装的所有来宾虚拟化工具和代理（例如 VMware 工具）。
    * 确保 VM 配置为通过 DHCP 来提取其 IP 地址和 DNS 设置。 这确保服务器在启动时在 VNet 中获取 IP 地址。 

## <a name="log-in-to-azure"></a>登录到 Azure
如果尚未安装 Azure PowerShell 1.4 版或更高版本，请阅读 [如何安装和配置 Azure PowerShell](https://docs.microsoft.com/zh-cn/powershell/azureps-cmdlets-docs)。

1. 打开 Azure PowerShell 并登录到 Azure 帐户。 将打开一个弹出窗口，可以输入 Azure 帐户凭据。

        Login-AzureRmAccount -EnvironmentName AzureChinaCloud

2. 获取可用订阅的订阅 ID。

        Get-AzureRmSubscription

3. 使用订阅 ID 设置正确的订阅。 将 `<subscriptionID>` 替换为正确订阅的 ID。

        Select-AzureRmSubscription -SubscriptionId "<subscriptionID>"

## <a name="createstorage"></a> 获取存储帐户
需要在 Azure 中创建存储帐户来存储上传的 VM 映像。 可以使用现有存储帐户，也可以创建新存储帐户。 

显示可用的存储帐户，请键入：

    Get-AzureRmStorageAccount

如果要使用现有存储帐户，请转到 [上载 VM 映像](#upload-the-vm-vhd-to-your-storage-account) 部分。

若要创建存储帐户，请执行以下步骤：

1. 需要应在其中创建存储帐户的资源组的名称。 若要查找订阅中的所有资源组，请键入：

        Get-AzureRmResourceGroup

    若要在**中国北部**区域中创建名为 **myResourceGroup** 的资源组，请键入：

        New-AzureRmResourceGroup -Name myResourceGroup -Location "China North"

2. 使用 [New-AzureRmStorageAccount](https://msdn.microsoft.com/zh-cn/library/mt607148.aspx) cmdlet 在此资源组中创建名为 **mystorageaccount** 的存储帐户：

        New-AzureRmStorageAccount -ResourceGroupName myResourceGroup -Name mystorageaccount -Location "China North" `
            -SkuName "Standard_LRS" -Kind "Storage"

    -SkuName 的有效值为：

    * **Standard_LRS** - 本地冗余存储。 
    * **Standard_ZRS** - 区域冗余存储。
    * **Standard_GRS** - 异地冗余存储。 
    * **Standard_RAGRS** - 读取访问权限异地冗余存储。 
    * **Premium_LRS** - 高级本地冗余存储。 

## <a name="upload-the-vm-vhd-to-your-storage-account"></a> 将 VHD 上传到存储帐户
使用 [Add-AzureRmVhd](https://msdn.microsoft.com/zh-cn/library/mt603554.aspx) cmdlet 将映像上载到存储帐户中的容器： 本示例将文件 **myVHD.vhd** 从 `"C:\Users\Public\Documents\Virtual hard disks\"` 上载到 **myResourceGroup** 资源组中名为 **mystorageaccount** 的存储帐户。 该文件将放入名为 **mycontainer** 的容器，新文件名为 **myUploadedVHD.vhd**。

    $rgName = "myResourceGroup"
    $urlOfUploadedImageVhd = "https://mystorageaccount.blob.core.chinacloudapi.cn/mycontainer/myUploadedVHD.vhd"
    Add-AzureRmVhd -ResourceGroupName $rgName -Destination $urlOfUploadedImageVhd `
        -LocalFilePath "C:\Users\Public\Documents\Virtual hard disks\myVHD.vhd"

如果成功，将显示类似于下面的响应：

    MD5 hash is being calculated for the file C:\Users\Public\Documents\Virtual hard disks\myVHD.vhd.
    MD5 hash calculation is completed.
    Elapsed time for the operation: 00:03:35
    Creating new page blob of size 53687091712...
    Elapsed time for upload: 01:12:49

    LocalFilePath           DestinationUri
    -------------           --------------
    C:\Users\Public\Doc...  https://mystorageaccount.blob.core.chinacloudapi.cn/mycontainer/myUploadedVHD.vhd

完成执行此命令可能需要一段时间，具体取决于网络连接速度和 VHD 文件的大小

## <a name="next-steps"></a>后续步骤
* [基于通用 VHD 在 Azure 中创建 VM](/documentation/articles/virtual-machines-windows-create-vm-generalized/)
* 创建新 VM 时将专用 VHD 附加为 OS 磁盘，[基于专用 VHD 在 Azure 中创建 VM](/documentation/articles/virtual-machines-windows-create-vm-specialized/)。
<!--Update_Description: wording update-->