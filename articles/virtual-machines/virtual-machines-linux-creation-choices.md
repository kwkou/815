<properties
	pageTitle="创建 Linux VM 的不同方式 | Azure"
	description="列出在 Azure 上创建 Linux 虚拟机的不同方法，以及指向每种方法的工具和教程的链接。"
	services="virtual-machines-linux"
	documentationCenter=""
	authors="iainfoulds"
	manager="timlt"
	editor=""
	tags="azure-resource-manager"/>

<tags
	ms.service="virtual-machines-linux"
	ms.devlang="na"
	ms.topic="get-started-article"
	ms.tgt_pltfrm="vm-linux"
	ms.workload="infrastructure-services"
	ms.date="07/06/2016"
	wacn.date="08/15/2016"
	ms.author="iainfou"/>

# 使用 Resource Manager 创建 Linux 虚拟机的不同方式

Azure 提供不同的方法来使用 Resource Manager 部署模型创建 VM，以满足不同的用户和目的。本文汇总了这些方法的差异，以及在创建 Linux 虚拟机 (VM) 时可使用的选项。

## Azure CLI 

Azure CLI 可通过 npm 包、提供发行版的程序包或 Docker 容器跨平台使用。你可以阅读更多有关[如何安装和配置 Azure CLI](/documentation/articles/xplat-cli-install/) 的信息。以下教程提供了有关使用 Azure CLI 的示例。阅读下面每篇文章，了解更多有关所示的 CLI 快速启动命令的更多详细信息：

* [Create a Linux VM from the Azure CLI for dev and test](/documentation/articles/virtual-machines-linux-quick-create-cli/)（从 Azure CLI 创建用于开发和测试的 Linux VM）

		azure vm quick-create -M ~/.ssh/azure_id_rsa.pub -Q CoreOS

* [使用 Azure 模板创建受保护的 Linux VM](/documentation/articles/virtual-machines-linux-create-ssh-secured-vm-from-template/)

		azure group create --name TestRG --location ChinaNorth 
			--template-file /path/to/101-vm-sshkey/azuredeploy.json

* [使用 Azure CLI 从头开始创建 Linux VM](/documentation/articles/virtual-machines-linux-create-cli-complete/)

* [将磁盘添加到 Linux VM](/documentation/articles/virtual-machines-linux-add-disk/)

		azure vm disk attach-new --resource-group TestRG --vm-name TestVM <size-in-GB>

## Azure 门户预览

[Azure 门户预览](https://portal.azure.cn)的图形用户界面是一种试用 VM 的简便方式，尤其是在你刚开始摸索 Azure 时，因为无需在你的系统上安装任何内容。使用 Azure 门户预览创建 VM：

* [使用 Azure 门户预览创建 Linux VM](/documentation/articles/virtual-machines-linux-quick-create-portal/)
* [使用 Azure 门户预览附加磁盘](/documentation/articles/virtual-machines-linux-attach-disk-portal/)

## 操作系统和映像选项
创建 VM 时，可根据要运行的操作系统选择映像。Azure 及其合作伙伴提供了许多映像，其中一些映像包括预安装的应用程序和工具。或者，你也可以上载自己的映像之一（参见下文）。

### Azure 映像
可以使用 `azure vm image` CLI 命令，按发布者、发行版本和内部版本查看可用的内容。

列出可用的发布者：

	azure vm image list-publishers --location ChinaNorth

列出给定发布者的可用产品（服务）：

	azure vm image list-offers --location ChinaNorth --publisher Canonical

列出给定产品/服务的可用 SKU（发行版本）：

	azure vm image list-skus --location ChinaNorth --publisher Canonical --offer UbuntuServer

列出给定发行版的所有可用映像：

	azure vm image list --location ChinaNorth --publisher Canonical --offer UbuntuServer --sku 16.04.0-LTS

请参阅 [Navigate and select Azure virtual machine images with the Azure CLI](/documentation/articles/virtual-machines-linux-cli-ps-findimage/)（使用 Azure CLI 导航并选择 Azure 虚拟机映像），了解决有关浏览和使用可用映像的更多示例。

`azure vm quick-create` 和 `azure vm create` 命令还提供一些别名，可用于快速访问较常见的发行版及其最新版本。这比每次创建 VM 时都需要指定发布者、产品/服务、SKU 和版本要方便：

| 别名 | 发布者 | 产品 | SKU | 版本 |
|:----------|:----------|:-------------|:------------|:--------|
| CentOS | OpenLogic | Centos | 7\.2 | 最新 |
| CoreOS    | CoreOS    | CoreOS       | Stable      | 最新  |
| Debian    | credativ  | Debian       | 8           | 最新  |
| openSUSE | SUSE | openSUSE | 13\.2 | 最新 |
| SLES | SUSE | SLES | 12-SP1 | 最新 |
| UbuntuLTS | Canonical | UbuntuServer | 14\.04.3-LTS | 最新 |

### 使用你自己的映像

如果你需要特定的自定义，则可以通过*捕获*现有 Azure VM 来使用基于该 VM 的映像，或者上载你自己的存储在虚拟硬盘 (VHD) 中的映像。有关受支持的发行版以及如何使用你自己的映像的详细信息，请参阅以下文章：

* [Azure endorsed distributions（Azure 认可的分发版）](/documentation/articles/virtual-machines-linux-endorsed-distros/)

* [Information for non-endorsed distributions（有关未认可分发版的信息）](/documentation/articles/virtual-machines-linux-create-upload-generic/)

* [How to capture a Linux virtual machine as a Resource Manager template](/documentation/articles/virtual-machines-linux-capture-image/)（如何捕获用作 Resource Manager 模板的 Linux 虚拟机）。快速启动命令：

		azure vm deallocate --resource-group TestRG --vm-name TestVM
		azure vm generalize --resource-group TestRG --vm-name TestVM
		azure vm capture --resource-group TestRG --vm-name TestVM --vhd-name-prefix CapturedVM

## 后续步骤

* 尝试阅读以下教程之一，了解如何通过[门户](/documentation/articles/virtual-machines-linux-quick-create-portal/)、[CLI](/documentation/articles/virtual-machines-linux-quick-create-cli/) 或 Azure Resource Manager [模板](/documentation/articles/virtual-machines-linux-cli-deploy-templates/)创建 Linux VM。

* 创建 Linux VM 后，可以轻松[添加数据磁盘](/documentation/articles/virtual-machines-linux-add-disk/)。

* [重置密码或 SSH 密钥和管理用户](/documentation/articles/virtual-machines-linux-using-vmaccess-extension/)的快速步骤

<!---HONumber=Mooncake_0808_2016-->