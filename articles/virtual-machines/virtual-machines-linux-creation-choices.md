<properties
	pageTitle="创建 Linux VM 的不同方式 | Azure"
	description="介绍在 Azure 上创建 Linux 虚拟机的不同方法，并提供每种方法的工具和教程的链接。"
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
	ms.date="09/27/2016"
	wacn.date="01/05/2017"
	ms.author="iainfou"/>  


# 在 Azure 中创建 Linux 虚拟机的不同方式

在 Azure 中，可以使用习惯的工具和工作流灵活创建 Linux 虚拟机 (VM)。本文汇总了这些方法的差异，并举例说明如何创建 Linux VM。


## Azure CLI 

Azure CLI 可通过 npm 包、提供发行版的程序包或 Docker 容器跨平台使用。你可以阅读更多有关[如何安装和配置 Azure CLI](/documentation/articles/xplat-cli-install/) 的信息。以下教程提供了有关使用 Azure CLI 的示例。阅读下面每篇文章，了解更多有关所示的 CLI 快速启动命令的更多详细信息：

- [Create a Linux VM from the Azure CLI for dev and test](/documentation/articles/virtual-machines-linux-quick-create-cli/)（从 Azure CLI 创建用于开发和测试的 Linux VM）
	- 以下示例使用名为 `azure_id_rsa.pub` 的公钥创建 CoreOS VM：

			azure vm quick-create -ssh-publickey-file ~/.ssh/azure_id_rsa.pub \
				--image-urn CoreOS

- [使用 Azure 模板创建受保护的 Linux VM](/documentation/articles/virtual-machines-linux-create-ssh-secured-vm-from-template/)
	- 以下示例使用 GitHub 上存储的模板创建 VM：

            azure group create --name myResourceGroup --location ChinaNorth \
              --template-file /path/to/101-vm-sshkey/azuredeploy.json

- [使用 Azure CLI 创建完整的 Linux 环境](/documentation/articles/virtual-machines-linux-create-cli-complete/)
	- 包括在可用性集中创建负载均衡器和多个 VM。

- [将磁盘添加到 Linux VM](/documentation/articles/virtual-machines-linux-add-disk/)
	- 以下示例将一个 5Gb 磁盘添加到名为 `TestVM` 的现有 VM：

            azure vm disk attach-new --resource-group myResourceGroup  --vm-name myVM \
              --size-in-GB 5

## Azure 门户预览

在 [Azure 门户预览](https://portal.azure.cn)中可以快速创建 VM，因为不需要在系统上安装任何组件。使用 Azure 门户预览创建 VM：

* [使用 Azure 门户预览创建 Linux VM](/documentation/articles/virtual-machines-linux-quick-create-portal/)
* [使用 Azure 门户预览附加磁盘](/documentation/articles/virtual-machines-linux-attach-disk-portal/)


## 操作系统和映像选项
创建 VM 时，可根据要运行的操作系统选择映像。Azure 及其合作伙伴提供了许多映像，其中一些映像包括预安装的应用程序和工具。你也可以上载自己的某个映像（请参阅[下一部分](#use-your-own-image)）。

### Azure 映像
使用 `azure vm image` CLI 命令可按发布者、发行版本和内部版本查看可用内容。

列出可用的发布者，如下所示：

	azure vm image list-publishers --location ChinaNorth

列出特定发布者的可用产品，如下所示：

	azure vm image list-offers --location ChinaNorth --publisher Canonical

列出给定产品的可用 SKU （分发版），如下所示：

	azure vm image list-skus --location ChinaNorth --publisher Canonical --offer UbuntuServer

列出给定版本的所有可用映像，如下所示：

	azure vm image list --location ChinaNorth --publisher Canonical --offer UbuntuServer --sku 16.04.0-LTS

有关浏览和使用可用映像的更多示例，请参阅[使用 Azure CLI 导航并选择 Azure 虚拟机映像](/documentation/articles/virtual-machines-linux-cli-ps-findimage/)。

`azure vm quick-create` 和 `azure vm create` 命令具有一些别名，可用于快速访问其他常用分发版及其最新版本。使用别名通常比每次创建 VM 时指定发布者、产品、SKU 和版本更加快捷：

| 别名 | 发布者 | 产品 | SKU | 版本 |
|:----------|:----------|:-------------|:------------|:--------|
| CentOS | OpenLogic | Centos | 7\.2 | 最新 |
| CoreOS | CoreOS | CoreOS | Stable | 最新 |
| Debian | credativ | Debian | 8 | 最新 |
| openSUSE | SUSE | openSUSE | 13\.2 | 最新 |
| SLES | SUSE | SLES | 12-SP1 | 最新 |
| UbuntuLTS | Canonical | UbuntuServer | 14\.04.3-LTS | 最新 |

### <a name="use-your-own-image"></a>使用自己的映像

若要进行具体的自定义，可以通过*捕获*现有 Azure VM 来使用基于该 VM 的映像。也可以上载本地创建的映像。有关受支持的发行版以及如何使用你自己的映像的详细信息，请参阅以下文章：

- [Azure endorsed distributions（Azure 认可的分发版）](/documentation/articles/virtual-machines-linux-endorsed-distros/)

- [Information for non-endorsed distributions（有关未认可分发版的信息）](/documentation/articles/virtual-machines-linux-create-upload-generic/)

- [How to capture a Linux virtual machine as a Resource Manager template](/documentation/articles/virtual-machines-linux-capture-image/)（如何捕获用作 Resource Manager 模板的 Linux 虚拟机）。
	- 用于捕获现有 VM 的快速入门示例命令：

            azure vm deallocate --resource-group myResourceGroup --vm-name myVM
            azure vm generalize --resource-group myResourceGroup --vm-name myVM
            azure vm capture --resource-group myResourceGroup --vm-name myVM --vhd-name-prefix myCapturedVM

## 后续步骤

- 通过[门户](/documentation/articles/virtual-machines-linux-quick-create-portal/)、[CLI](/documentation/articles/virtual-machines-linux-quick-create-cli/) 或 [Azure Resource Manager 模板](/documentation/articles/virtual-machines-linux-cli-deploy-templates/)创建 Linux VM。

- 创建 Linux VM 后[添加数据磁盘](/documentation/articles/virtual-machines-linux-add-disk/)。

- [重置密码或 SSH 密钥和管理用户](/documentation/articles/virtual-machines-linux-using-vmaccess-extension/)的快速步骤

<!---HONumber=Mooncake_1114_2016-->