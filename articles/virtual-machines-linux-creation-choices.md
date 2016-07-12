<properties
	pageTitle="创建 Linux VM 的不同方式 | Azure"
	description="列出在 Azure 上创建 Linux 虚拟机的不同方式，并提供其他说明的链接"
	services="virtual-machines-linux"
	documentationCenter=""
	authors="iainfoulds"
	manager="timlt"
	editor=""
	tags="azure-resource-manager"/>

<tags
	ms.service="virtual-machines-linux"
	ms.date="04/12/2016"
	wacn.date="06/07/2016"/>

# 使用 Resource Manager 创建 Linux 虚拟机的不同方式

Azure 提供不同的方法来创建 VM，以符合不同的用户和目的。本文汇总了这些方法的差异，以及在创建 Linux 虚拟机时可使用的选项。

## Azure CLI 

通过 CLI 使用 Azure 命令行界面。请参阅以下有关使用 Azure CLI 的教程：

* [Create a Linux VM from the CLI for dev and test（从 CLI 创建用于开发和测试的 Linux VM）](/documentation/articles/virtual-machines-linux-quick-create-cli/) 

* [使用 Azure 模板创建受保护的 Linux VM](/documentation/articles/virtual-machines-linux-create-ssh-secured-vm-from-template/)

## Azure 门户预览

[Azure 门户预览](https://portal.azure.cn)的图形用户界面是一种试用虚拟机的简便方式，尤其是在你刚开始摸索 Azure 时。使用 Azure 门户预览创建 VM：

* [使用 Azure 门户预览创建运行 Linux 的虚拟机](/documentation/articles/virtual-machines-linux-quick-create-portal/) 

## 操作系统和映像选项

根据要运行的操作系统选择映像。Azure 及其合作伙伴提供了许多映像，其中一些映像包括应用程序和工具。或者，使用你自己的某一映像。

### Azure 映像

在所有上述文章中，你可以轻松使用现有 Azure 映像来创建虚拟机，并针对联网、负载平衡等目的自定义该虚拟机。门户预览提供 Azure 应用市场，其中包含 Azure 提供的映像。你可以使用命令行获取类似的列表。例如，在 Azure CLI 中，运行 `azure vm image list` 可按位置和发布者获取所有可用映像的列表。请参阅 [Navigate and select Azure virtual machine images with the Azure CLI（使用 Azure CLI 导航并选择 Azure 虚拟机映像）](/documentation/articles/virtual-machines-linux-cli-ps-findimage/)。

### 使用你自己的映像

通过捕获现有 Azure 虚拟机来使用基于该 VM 的映像，或者上载你自己的存储在虚拟硬盘 (VHD) 中的映像。有关详细信息，请参阅：

* [Azure endorsed distributions（Azure 认可的分发版）](/documentation/articles/virtual-machines-linux-endorsed-distros/)

* [Information for non-endorsed distributions（有关未认可分发版的信息）](/documentation/articles/virtual-machines-linux-create-upload-generic/)

* [How to capture a Linux virtual machine as a Resource Manager template（如何捕获用作 Resource Manager 模板的 Linux 虚拟机）](/documentation/articles/virtual-machines-linux-capture-image/)

## 后续步骤

* 尝试阅读以下教程之一，了解如何通过[门户预览](/documentation/articles/virtual-machines-linux-quick-create-portal/)、[CLI](/documentation/articles/virtual-machines-linux-quick-create-cli/) 或 Azure Resource Manager [模板](/documentation/articles/virtual-machines-linux-cli-deploy-templates/)创建 Linux VM。

* 创建 Linux VM 后，可以轻松[添加数据磁盘](/documentation/articles/virtual-machines-linux-add-disk/)。

<!---HONumber=Mooncake_0503_2016-->