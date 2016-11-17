<properties
	pageTitle="使用 Powershell 创建和上载 VM 映像 | Azure"
	description="了解如何使用经典部署模型和 Azure Powershell 创建并上载通用化 Windows Server 映像 (VHD)。"
	services="virtual-machines-windows"
	documentationCenter=""
	authors="cynthn"
	manager="timlt"
	editor="tysonn"
	tags="azure-service-management"/>

<tags
	ms.service="virtual-machines-windows"
	ms.workload="infrastructure-services"
	ms.tgt_pltfrm="vm-windows"
	ms.devlang="na"
	ms.topic="article"
	ms.date="07/21/2016"
	wacn.date="09/12/2016"
	ms.author="cynthn"/>  


# 创建 Windows Server VHD 并将其上载到 Azure

本文说明如何上载自己的通用化 VM 映像作为虚拟硬盘 (VHD)，以便使用它来创建虚拟机。有关 Azure 中的磁盘和 VHD 的更多详细信息，请参阅 [About Disks and VHDs for Virtual Machines](/documentation/articles/virtual-machines-linux-about-disks-vhds/)（关于虚拟机的磁盘和 VHD）。


> [AZURE.IMPORTANT] Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model/)。本文介绍使用经典部署模型。Azure 建议大多数新部署使用 Resource Manager 模型。也可以使用 Resource Manager 模型来[捕获](/documentation/articles/virtual-machines-windows-capture-image/)和[上载](/documentation/articles/virtual-machines-windows-upload-image/)虚拟机。

## 先决条件

本文假定你具备以下条件：

- **Azure 订阅** - 如果没有，可以[建立一个 Azure 帐户](/pricing/1rmb-trial/?WT.mc_id=A261C142F)。

- **[Azure PowerShell](/documentation/articles/powershell-install-configure/)** - 已安装 Azure PowerShell 模块并将其配置为使用用户的订阅。

- **.VHD 文件** - 存储在 .vhd 文件中并附加到虚拟机的受支持 Windows 操作系统。还应该检查 sysprep 是否支持 VHD 上运行的服务器角色。有关详细信息，请参阅 [Sysprep Support for Server Roles](https://msdn.microsoft.com/windows/hardware/commercialize/manufacture/desktop/sysprep-support-for-server-roles)（Sysprep 对服务器角色的支持）。

> [AZURE.IMPORTANT] Azure 不支持 VHDX 格式。可使用 Hyper-V 管理器或 [Convert-VHD cmdlet](http://technet.microsoft.com/zh-cn/library/hh848454.aspx) 将磁盘转换为 VHD 格式。有关详细信息，请参阅此 [blogpost](http://blogs.msdn.com/b/virtual_pc_guy/archive/2012/10/03/using-powershell-to-convert-a-vhd-to-a-vhdx.aspx)。

## 步骤 1：准备 VHD 

在将 VHD 上载到 Azure 之前，需要使用 Sysprep 工具对 VHD 进行一般化。这样就可以将 VHD 作为映像使用。有关 Sysprep 的详细信息，请参阅[如何使用 Sysprep：简介](http://technet.microsoft.com/zh-cn/library/bb457073.aspx)。

在操作系统已安装到的虚拟机中，完成以下过程：

1. 登录到操作系统。

2. 以管理员身份打开“命令提示符”窗口。将目录更改为 **%windir%\\system32\\sysprep**，然后运行 `sysprep.exe`。

	![打开“命令提示符”窗口](./media/virtual-machines-windows-classic-createupload-vhd/sysprep_commandprompt.png)  


3.	此时会显示**“系统准备工具”**对话框。

	![启动 Sysprep](./media/virtual-machines-windows-classic-createupload-vhd/sysprepgeneral.png)

4.  在**“系统准备工具”**中，选择**“进入系统全新体验(OOBE)”**并确保选中**“一般化”**。

5.  在**“关机选项”**中选择**“关机”**。

6.  单击**“确定”**。

## 步骤 2：创建存储帐户和容器

你需要一个 Azure 存储帐户，以便有一个可向其上载 .vhd 文件的位置。此步骤演示了如何创建一个帐户，或如何从现有帐户获取需要的信息。将 &lsaquo; 和 &rsaquo; 括号中的变量替换为自己的信息。

1. 登录

		Add-AzureAccount -Environment AzureChinaCloud

1. 设置你的 Azure 订阅。

    	Select-AzureSubscription -SubscriptionName <SubscriptionName> 

2. 新建存储帐户。存储帐户的名称应该唯一且包含 3-24 个字符。名称可以是字母和数字的任意组合。还需要指定一个位置，例如“中国东部”。
    	
		New-AzureStorageAccount -StorageAccountName <StorageAccountName> -Location <Location>

3. 将新存储帐户设置为默认值。
    	
		Set-AzureSubscription -CurrentStorageAccountName <StorageAccountName> -SubscriptionName <SubscriptionName>

4. 创建新容器。

    	New-AzureStorageContainer -Name <ContainerName> -Permission Off

 

## 步骤 3：上载 .vhd 文件

使用 [Add-AzureVhd](http://msdn.microsoft.com/zh-cn/library/dn495173.aspx) 上载 VHD。

在上一步骤使用的 Azure PowerShell 窗口中键入以下命令，并将 &lsaquo; 和 &rsaquo; 括号中的变量替换为自己的信息。

		Add-AzureVhd -Destination "https://<StorageAccountName>.blob.core.chinacloudapi.cn/<ContainerName>/<vhdName>.vhd" -LocalFilePath <LocalPathtoVHDFile>


## 步骤 4：将映像添加到自定义映像列表

使用 [Add-AzureVMImage])(https://msdn.microsoft.com/zh-cn/library/mt589167.aspx) cmdlet 将映像添加到自定义映像列表。

		Add-AzureVMImage -ImageName <ImageName> -MediaLocation "https://<StorageAccountName>.blob.core.chinacloudapi.cn/<ContainerName>/<vhdName>.vhd" -OS "Windows"


## 后续步骤

现在可以使用上载的映像来[创建自定义的 VM](/documentation/articles/virtual-machines-windows-classic-createportal/)。

<!---HONumber=Mooncake_0905_2016-->