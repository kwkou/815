<properties
	pageTitle="创建并上载 Linux VHD | Azure"
	description="使用包含 Linux 操作系统的经典部署模型创建并上载 Azure 虚拟硬盘 (VHD)。"
	services="virtual-machines-linux"
	documentationCenter=""
	authors="iainfoulds"
	manager="timlt"
	editor="tysonn"
	tags="azure-service-management"/>

<tags
	ms.service="virtual-machines-linux"
	ms.date="06/14/2016"
	wacn.date="07/11/2016"/>

# 创建并上载包含 Linux 操作系统的虚拟硬盘

> [AZURE.IMPORTANT] Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model/)。本文介绍使用经典部署模型。Azure 建议大多数新部署使用 Resource Manager 模型。

本文介绍如何创建和上载虚拟硬盘 (VHD)，以便可以使用它作为自己的映像在 Azure 中创建虚拟机。你将学习如何准备操作系统，以便使用它来根据该映像创建多个虚拟机。

**重要说明**：只有在使用某个认可的分发的时候也使用 [Azure上的 Linux 认可的分发](/documentation/articles/virtual-machines-linux-endorsed-distros/)中“支持的版本”下指定的配置详细信息时，Azure 平台 SLA 才适用于运行 Linux 操作系统的虚拟机。Azure 映像库中的所有 Linux 分发都是具有所需配置的认可的分发。


## 先决条件
本文假定你拥有以下项目：

- **安装在 .vhd 文件中的 Linux 操作系统** — 已将 [Azure 认可的 Linux 分发](/documentation/articles/virtual-machines-linux-endorsed-distros/)（或参阅[关于未认可分发的信息](/documentation/articles/virtual-machines-linux-create-upload-generic/)）安装在 VHD 格式的虚拟磁盘中。存在多种工具可创建 .vhd 文件，例如，可以使用虚拟化解决方案（例如 Hyper-V）创建 .vhd 文件并安装操作系统。有关说明，请参阅[安装 Hyper-V 角色和配置虚拟机](http://technet.microsoft.com/zh-cn/library/hh846766.aspx)。

	> [AZURE.NOTE] Azure 不支持更新的 VHDX 格式。可使用 Hyper-V 管理器或 `Convert-VHD` cmdlet 将 VHDX 转换为 VHD 格式。此外，Azure 不支持上载动态 VHD，因此，上载之前，你需要将此类磁盘转换为静态 VHD。可以使用诸如 [Azure VHD Utilities for GO](https://github.com/Microsoft/azure-vhd-utils-for-go) 之类的工具转换动态磁盘。

- **Azure 命令行接口** — 安装最新的 [Azure 命令行接口](/documentation/articles/virtual-machines-command-line-tools/)以上载 VHD。

<a id="prepimage"> </a>
## 步骤 1：准备要上载的映像

Azure 支持各种 Linux 分发（请参阅[认可的分发](/documentation/articles/virtual-machines-linux-endorsed-distros/)）。以下文章将指导你完成如何准备 Azure 上支持的各种 Linux 分发。按照以下指导中的步骤进行操作以后，回过头来查看此内容，你应该有了一个可以上载到 Azure 的 VHD 文件：

- **[基于 CentOS 的分发](/documentation/articles/virtual-machines-linux-create-upload-centos/)**
- **[Debian Linux](/documentation/articles/virtual-machines-linux-debian-create-upload-vhd/)**
- **[Oracle Linux](/documentation/articles/virtual-machines-linux-oracle-create-upload-vhd/)**
- **[Red Hat Enterprise Linux](/documentation/articles/virtual-machines-linux-redhat-create-upload-vhd/)**
- **[SLES 和 openSUSE](/documentation/articles/virtual-machines-linux-suse-create-upload-vhd/)**
- **[Ubuntu](/documentation/articles/virtual-machines-linux-create-upload-ubuntu/)**
- **[其他 - 非认可分发](/documentation/articles/virtual-machines-linux-create-upload-generic/)**

另请参阅 **[Linux 安装说明](/documentation/articles/virtual-machines-linux-create-upload-generic/#general-linux-installation-notes)**，以获取更多有关如何为 Azure 准备 Linux 映像的一般提示。


<a id="connect"> </a>
## 步骤 2：准备连接到 Azure

请确保在经典部署模型中使用 Azure CLI (`azure config mode asm`)，然后登录帐户：

	azure login -e AzureChinaCloud

<a id="upload"> </a>
## 步骤 3：向 Azure 上载映像

你需要一个存储帐户，以便向其上载你的 VHD 文件。你可以选择现有的存储帐户，也可以[创建一个新的](/documentation/articles/storage-create-storage-account/)。

在 Azure CLI 中使用以下命令来上载映像：

		azure vm image create <ImageName> --blob-url <BlobStorageURL>/<YourImagesFolder>/<VHDName> --os Linux <PathToVHDFile>

在上面的示例中：

- **BlobStorageURL** 是你打算使用的存储帐户的 URL
- **YourImagesFolder** 是你在 blob 存储中要用来存储映像的容器
- **VHDName** 是显示在门户中用于标识虚拟硬盘的标签。
- **PathToVHDFile** 是 .vhd 文件在计算机上的完整路径和名称。

有关详细信息，请参阅[用于 Azure 服务管理的 Azure CLI 参考](/documentation/articles/virtual-machines-command-line-tools/)。

[Step 1: Prepare the image to be uploaded]: #prepimage
[Step 2: Prepare the connection to Azure]: #connect
[Step 3: Upload the image to Azure]: #upload

<!---HONumber=Mooncake_0704_2016-->