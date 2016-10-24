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
	ms.workload="infrastructure-services"
	ms.tgt_pltfrm="vm-linux"
	ms.devlang="na"
	ms.topic="article"
	ms.date="09/01/2016"
	wacn.date="10/24/2016"
	ms.author="iainfou"/>  


# 创建并上载包含 Linux 操作系统的虚拟硬盘

> [AZURE.IMPORTANT]Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model/)。本文介绍使用经典部署模型。Azure 建议大多数新部署使用资源管理器模型。还可以[使用 Azure Resource Manager 上载自定义磁盘映像](/documentation/articles/virtual-machines-linux-upload-vhd/)。

本文介绍如何创建和上载虚拟硬盘 (VHD)，以便可以使用它作为自己的映像在 Azure 中创建虚拟机。学习如何准备操作系统，以便使用它来基于该映像创建多个虚拟机。

>  [AZURE.NOTE] 如果有时间，请参加这个有关体验的[快速调查](https://aka.ms/linuxdocsurvey)，帮助我们改进 Azure Linux VM 文档。每个回答都将帮助我们帮助你完成工作。

## 先决条件
本文假定你拥有以下项目：

- **安装在 .vhd 文件中的 Linux 操作系统** - 已将 [Azure 认可的 Linux 分发](/documentation/articles/virtual-machines-linux-endorsed-distros/)（或参阅[关于未认可分发的信息](/documentation/articles/virtual-machines-linux-create-upload-generic/)）安装在 VHD 格式的虚拟磁盘中。可使用多种工具创建 VM 和 VHD：
	- 安装并配置 [QEMU](https://en.wikibooks.org/wiki/QEMU/Installing_QEMU) 或 [KVM](http://www.linux-kvm.org/page/RunningKVM)，并小心使用 VHD 作为你的映像格式。如有需要，可以使用 `qemu-img convert` [转换映像](https://en.wikibooks.org/wiki/QEMU/Images#Converting_image_formats)。
	- 也可以在 [Windows 10](https://msdn.microsoft.com/virtualization/hyperv_on_windows/quick_start/walkthrough_install) 或 [Windows Server 2012/2012 R2](https://technet.microsoft.com/zh-cn/library/hh846766.aspx) 上使用 Hyper-V。

> [AZURE.NOTE] Azure 不支持更新的 VHDX 格式。创建 VM 时，请将 VHD 指定为映像格式。如有需要，可以使用 [`qemu-img convert`](https://en.wikibooks.org/wiki/QEMU/Images#Converting_image_formats) 或 [`Convert-VHD`](https://technet.microsoft.com/zh-cn/library/hh848454.aspx) PowerShell cmdlet 将 VHDX 磁盘转换为 VHD。此外，Azure 不支持上载动态 VHD，因此，上载之前，你需要将此类磁盘转换为静态 VHD。可以使用 [Azure VHD Utilities for GO](https://github.com/Microsoft/azure-vhd-utils-for-go) 等工具在上载到 Azure 的过程中转换动态磁盘。

- **Azure 命令行接口** - 安装最新的 [Azure 命令行接口](/documentation/articles/virtual-machines-command-line-tools/)以上载 VHD。

<a id="prepimage"> </a>
## 步骤 1：准备要上载的映像

Azure 支持各种 Linux 分发版（请参阅 [Endorsed Distributions](/documentation/articles/virtual-machines-linux-endorsed-distros/)（认可的分发版））。以下文章将指导用户如何准备 Azure 上支持的各种 Linux 分发。完成以下指南中的步骤后，返回到此处，你应该有了一个可以上载到 Azure 的 VHD 文件：

- **[基于 CentOS 的分发](/documentation/articles/virtual-machines-linux-create-upload-centos/)**
- **[Debian Linux](/documentation/articles/virtual-machines-linux-debian-create-upload-vhd/)**
- **[Oracle Linux](/documentation/articles/virtual-machines-linux-oracle-create-upload-vhd/)**
- **[Red Hat Enterprise Linux](/documentation/articles/virtual-machines-linux-redhat-create-upload-vhd/)**
- **[SLES 和 openSUSE](/documentation/articles/virtual-machines-linux-suse-create-upload-vhd/)**
- **[Ubuntu](/documentation/articles/virtual-machines-linux-create-upload-ubuntu/)**
- **[其他 - 非认可分发](/documentation/articles/virtual-machines-linux-create-upload-generic/)**

> [AZURE.NOTE] 只有在使用某个认可的分发的时候也使用 [Azure 认可的分发中的 Linux](/documentation/articles/virtual-machines-linux-endorsed-distros/) 中“支持的版本”下指定的配置详细信息时，Azure 平台 SLA 才适用于运行 Linux 操作系统的虚拟机。Azure 映像库中的所有 Linux 分发都是具有所需配置的认可的分发。

另请参阅 **[Linux 安装说明](/documentation/articles/virtual-machines-linux-create-upload-generic/#general-linux-installation-notes)**，以获取更多有关如何为 Azure 准备 Linux 映像的一般提示。


<a id="connect"> </a>
## 步骤 2：准备连接到 Azure

请确保在经典部署模型中使用 Azure CLI (`azure config mode asm`)，然后登录帐户：

	azure login -e AzureChinaCloud

<a id="upload"> </a>
## 步骤 3：向 Azure 上载映像

需要一个存储帐户，以便向其上载 VHD 文件。可以选取现有存储帐户，也可以[创建新的存储帐户](/documentation/articles/storage-create-storage-account/)。

在 Azure CLI 中使用以下命令来上载映像：

	azure vm image create <ImageName> `
		--blob-url <BlobStorageURL>/<YourImagesFolder>/<VHDName> `
		--os Linux <PathToVHDFile>

在上面的示例中：

- **BlobStorageURL** 是打算使用的存储帐户的 URL
- **YourImagesFolder** 是在 Blob 存储中要用来存储映像的容器
- **VHDName** 是显示在门户中用于标识虚拟硬盘的标签。
- **PathToVHDFile** 是 .vhd 文件在计算机上的完整路径和名称。

下面显示一个完整示例：

	azure vm image create UbuntuLTS `
		--blob-url https://teststorage.blob.core.chinacloudapi.cn/vhds/UbuntuLTS.vhd `
		--os Linux /home/ahmet/UbuntuLTS.vhd

## 步骤 4：从映像创建 VM
使用 `azure vm create` 以与创建常规 VM 相同的方式创建 VM。指定在上一步中为映像提供的名称。在以下示例中，使用上一步中指定的映像名称 **UbuntuLTS**：

	azure vm create --userName ops --password P@ssw0rd! --vm-size Small --ssh `
		--location "China North" "DeployedUbuntu" UbuntuLTS

若要创建自己的 VM，请提供自己的用户名 + 密码、位置、DNS 名称和映像名称。

## 后续步骤

有关详细信息，请参阅[用于 Azure 经典部署模型的 Azure CLI 参考](/documentation/articles/virtual-machines-command-line-tools/)。

[Step 1: Prepare the image to be uploaded]: #prepimage
[Step 2: Prepare the connection to Azure]: #connect
[Step 3: Upload the image to Azure]: #upload

<!---HONumber=Mooncake_1017_2016-->