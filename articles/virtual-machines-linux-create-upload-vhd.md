<properties
	pageTitle="创建和上载 Linux VHD | Azure"
	description="使用包含 Linux 操作系统的经典部署模型创建并上载 Azure 虚拟硬盘 (VHD)。"
	services="virtual-machines"
	documentationCenter=""
	authors="dsk-2015"
	manager="timlt"
	editor="tysonn"
	tags="azure-service-management"/>

<tags
	ms.service="virtual-machines"
	ms.date="01/22/2016"
	wacn.date="03/21/2016"/>

# 创建并上载包含 Linux 操作系统的虚拟硬盘

[AZURE.INCLUDE [了解部署模型](../includes/learn-about-deployment-models-classic-include.md)]


本文介绍如何创建和上载虚拟硬盘 (VHD)，以便可以使用它作为自己的映像在 Azure 中创建虚拟机。你将学习如何准备操作系统，以便使用它来根据该映像创建多个虚拟机。

[AZURE.INCLUDE [free-trial-note](../includes/create-account-note.md)]

Azure 虚拟机在基于创建过程中选择的映像的操作系统上运行。这些映像以 VHD 格式和 .vhd 文件的形式存储在存储帐户中。有关详细信息，请参阅 [Azure 中的磁盘](/documentation/articles/virtual-machines-disks-vhds)和 [Azure 中的映像](/documentation/articles/virtual-machines-images)。

当你创建虚拟机时，你可以自定义部分操作系统设置，使之适合于要运行的应用程序。有关说明，请参阅[如何创建自定义虚拟机](/documentation/articles/virtual-machines-create-custom)。

**重要说明**：只有在使用某个认可的分发的时候也使用 [Azure上的 Linux 认可的分发](/documentation/articles/virtual-machines-linux-endorsed-distributions)中“支持的版本”下指定的配置详细信息时，Azure 平台 SLA 才适用于运行 Linux 操作系统的虚拟机。Azure 映像库中的所有 Linux 分发都是具有所需配置的认可的分发。


## 先决条件
本文假定你拥有以下项目：

- **管理证书** - 你已为要为其上载 VHD 的订阅创建一个管理证书，并且已将该证书导出到 .cer 文件。有关创建证书的详细信息，请参阅[适用于 Azure 的证书概述](/documentation/articles/cloud-services-certs-create)。

- **安装在 .vhd 文件中的 Linux 操作系统** - 你已将受支持的 Linux 操作系统安装到虚拟硬盘。存在多种工具可创建 .vhd 文件，例如，可以使用虚拟化解决方案（例如 Hyper-V）创建 .vhd 文件并安装操作系统。有关说明，请参阅[安装 Hyper-V 角色和配置虚拟机](http://technet.microsoft.com/zh-cn/library/hh846766.aspx)。

	**重要说明**：Azure 不支持更新的 VHDX 格式。可使用 Hyper-V 管理器或 convert-vhd cmdlet 将磁盘转换为 VHD 格式。

	有关认可分发的列表，请参阅 [Azure 认可的分发中的 Linux](/documentation/articles/virtual-machines-linux-endorsed-distributions)。有关 Linux 分发的常规列表，请参阅[有关非认可分发的信息](/documentation/articles/virtual-machines-linux-create-upload-vhd-generic)。

- **Azure 命令行界面** - 如果你使用 Linux 操作系统来创建映像，则使用 [Azure 命令行界面](/documentation/articles/virtual-machines-command-line-tools)来上载 VHD。

- **Azure Powershell 工具** - 还可以使用 `Add-AzureVhd` cmdlet 来上载 VHD。若要下载 Azure Powershell cmdlet，请参阅 [Azure 下载](/downloads/)。有关引用信息，请参阅 [Add-AzureVhd](https://msdn.microsoft.com/zh-cn/library/azure/dn495173.aspx)（可能为英文页面）。

<a id="prepimage"> </a>
## 步骤 1：准备要上载的映像

Azure 支持各种 Linux 分发（请参阅[认可的分发](/documentation/articles/virtual-machines-linux-endorsed-distributions)）。以下文章将指导你完成如何准备 Azure 上支持的各种 Linux 分发：

- **[基于 CentOS 的分发](/documentation/articles/virtual-machines-linux-create-upload-vhd-centos)**
- **[Debian Linux](/documentation/articles/virtual-machines-linux-create-upload-vhd-debian)**
- **[Oracle Linux](/documentation/articles/virtual-machines-linux-create-upload-vhd-oracle)**
- **[Red Hat Enterprise Linux](/documentation/articles/virtual-machines-linux-create-upload-vhd-redhat)**
- **[SLES 和 openSUSE](/documentation/articles/virtual-machines-linux-create-upload-vhd-suse)**
- **[Ubuntu](/documentation/articles/virtual-machines-linux-create-upload-vhd-ubuntu)**
- **[其他 - 非认可分发](/documentation/articles/virtual-machines-linux-create-upload-vhd-generic)**

另请参阅 **[Linux 安装说明](/documentation/articles/virtual-machines-linux-create-upload-vhd-generic#linuxinstall)**，以获取更多有关如何为 Azure 准备 Linux 映像的提示。

按照上述指导中的步骤进行操作以后，你应该有了一个可以上载到 Azure 中的 VHD 文件。

<a id="connect"> </a>
## 步骤 2：准备连接到 Azure

你首先需要在计算机和 Azure 中的订阅之间建立一个安全连接，然后才能上载 .vhd 文件。


### 如果使用 Azure CLI

最新的 Azure CLI 会默认到资源管理器部署模型中，以确保使用以下命令使你处于经典部署模型中：

		azure config mode asm  

接下来，使用以下任意登录方法连接到你的 Azure 订阅。

使用 Azure AD 方法登录：

1. 打开 Azure CLI 窗口

2. 键入：

	`azure login -e AzureChinaCloud -u <username>`

	出现提示时，键入你的用户名和密码。

**或者**，改用 PublishSettings 文件：

1. 打开 Azure CLI 窗口

2. 键入：

	`azure account download -e AzureChinaCloud`

	此命令将打开浏览器窗口，并自动下载包含信息的 .publishsettings 文件和 Azure 订阅的证书。

3. 保存 .publishsettings 文件

4. 键入：

	`azure account import <PathToFile>`

	其中 `<PathToFile>` 是 .publishsettings 文件的完整路径。

	有关详细信息，请阅读[从 Azure CLI 连接到 Azure](/documentation/articles/xplat-cli-connect)。


### 如果使用 Azure PowerShell

使用 Azure AD 方法登录：

1. 打开 Azure PowerShell 窗口。

2. 键入：

	`Add-AzureAccount -Environment AzureChinaCloud`

	出现提示时，输入你组织提供的用户 ID 和密码。

**或者**，改用 PublishSettings 文件：

1. 打开 Azure PowerShell 窗口。

2. 键入：

	`Get-AzurePublishSettingsFile -Environment AzureChinaCloud`

	此命令将打开浏览器窗口，并自动下载包含信息的 .publishsettings 文件和 Azure 订阅的证书。

3. 保存 .publishsettings 文件。

4. 键入：

	`Import-AzurePublishSettingsFile -Environment AzureChinaCloud <PathToFile>`

	其中 `<PathToFile>` 是 .publishsettings 文件的完整路径。

	有关详细信息，请参阅[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure)。

> [AZURE.NOTE] 我们建议你使用更新的 Azure Active Directory 方法登录到你的 Azure 订阅，不管是通过 Azure CLI 还是通过 Azure PowerShell。

<a id="upload"> </a>
## 步骤 3：向 Azure 上载映像

你需要一个存储帐户，以便向其上载你的 VHD 文件。你可以选择现有的存储帐户，也可以创建一个新的。若要创建存储帐户，请参阅[创建存储帐户](/documentation/articles/storage-create-storage-account)

在上载 .vhd 文件时，你可以将 .vhd 文件放置在 Blob 存储中的任何位置。在以下命令示例中，**BlobStorageURL** 是你计划使用的存储帐户的 URL，**YourImagesFolder** 是要在其中存储映像的 Blob 存储中的容器。VHDName 是显示在 [Azure 管理门户](http://manage.windowsazure.cn)或 [Azure 管理门户](http://manage.windowsazure.cn)中用于标识虚拟硬盘的标签。**PathToVHDFile** 是 .vhd 文件在计算机上的完整路径和名称。


### 如果使用 Azure CLI

在 Azure CLI 中使用以下命令来上载映像：

		azure vm image create <ImageName> --blob-url <BlobStorageURL>/<YourImagesFolder>/<VHDName> --os Linux <PathToVHDFile>

有关详细信息，请参阅[用于 Azure 服务管理的 Azure CLI 参考](/documentation/articles/virtual-machines-command-line-tools)。


### 如果使用 PowerShell

从你在上一步中使用的 Azure PowerShell 窗口中，键入：

		Add-AzureVhd -Destination <BlobStorageURL>/<YourImagesFolder>/<VHDName> -LocalFilePath <PathToVHDFile>

有关详细信息，请参阅 [Add-AzureVhd](https://msdn.microsoft.com/zh-cn/library/azure/dn495173.aspx)。


[Step 1: Prepare the image to be uploaded]: #prepimage
[Step 2: Prepare the connection to Azure]: #connect
[Step 3: Upload the image to Azure]: #upload

<!---HONumber=Mooncake_0314_2016-->