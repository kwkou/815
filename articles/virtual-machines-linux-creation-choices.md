<properties
	pageTitle="创建 Linux VM 的不同方式 | Azure"
	description="列出在 Azure 上创建 Linux 虚拟机的不同方式，并提供进一步说明链接。"
	services="virtual-machines"
	documentationCenter=""
	authors="dsk-2015"
	manager="timlt"
	editor=""
	tags="azure-service-management,azure-resource-manager"/>

<tags
	ms.service="virtual-machines-linux"
	ms.date="01/20/2016"
	wacn.date="03/21/2016"/>

# 创建 Linux 虚拟机的不同方式

[AZURE.INCLUDE [了解部署模型](../includes/learn-about-deployment-models-both-include.md)]

Azure 提供不同的方法来创建 VM，以符合不同的用户和目的。本文汇总了这些方法的差异，以及在创建 Linux 虚拟机时可使用的选项。

## 工具选项

### GUI：Azure 管理门户 

Azure 管理门户的图形用户界面是一种试用虚拟机的简便方式，尤其是在你刚开始摸索 Azure 时。使用 [Azure 管理门户](https://manage.windowsazure.cn)创建 VM。如需一般性说明，请参阅[创建自定义虚拟机][]，然后从“库”中选择任意 Linux 映像。

### 命令 Shell：Azure CLI 或 Azure PowerShell

如果你喜欢使用命令行界面，请选择适用于 Mac、Linux 和 Windows 用户的 Azure 命令行界面 (CLI) 或 Azure PowerShell，后者提供适用于 Azure 的 Windows PowerShell cmdlet 和自定义控制台。

对于经典部署模型，请参阅[使用 Azure CLI 创建自定义 Linux 虚拟机](/documentation/articles/virtual-machines-linux-classic-create-custom)和[使用 Azure PowerShell 创建和预配置基于 Linux 的虚拟机][]。


### 开发环境：Visual Studio

Visual Studio 也支持创建 Azure 虚拟机。对于经典部署模型，请阅读[使用 Visual Studio 创建用于网站的虚拟机][]。

## 操作系统和映像选项

根据要运行的操作系统选择映像。Azure 及其合作伙伴提供了许多映像，其中一些映像包括应用程序和工具。或者，使用你自己的某一映像。


### Azure 映像

在所有上述文章中，你可以轻松使用现有 Azure 映像来创建虚拟机，并针对联网、负载平衡等目的自定义该虚拟机。门户将提供一个库，或者为 Azure 中的映像提供映像列表。你可以使用命令行获取类似的列表。例如，在 Azure CLI 中，运行 `azure vm image list` 可按位置和发布者获取所有可用映像的列表。


### 使用你自己的映像

通过 *捕获* 现有 Azure 虚拟机来使用基于该 VM 的映像，或者上载你自己的存储在虚拟硬盘 (VHD) 中的映像。对于经典部署模型，请参阅[如何使用 CLI 捕获将用作模板的 Linux 虚拟机][]和[创建并上载包含 Linux 操作系统的虚拟硬盘][]。

## 后续步骤

[登录到虚拟机][]

[附加数据磁盘][]

## 其他资源

[基本配置测试环境][]

<!-- LINKS -->
[overview]: /documentation/articles/resource-group-overview

[Create a Virtual Machine Running Windows]: /documentation/articles/virtual-machines-windows-hero-tutorial
[Create a Virtual Machine Running Linux]: /documentation/articles/virtual-machines-linux-quick-create-cli

[Equivalent Resource Manager and Service Management Commands for VM Operations with the Azure CLI for Mac, Linux, and Windows]: /documentation/articles/virtual-machines-windows-cli-manage
[Deploy and Manage Virtual Machines using Azure Resource Manager Templates and the Azure CLI]: /documentation/articles/virtual-machines-linux-cli-deploy-templates
[Deploy and Manage Virtual Machines using Azure Resource Manager Templates and PowerShell]: /documentation/articles/virtual-machines-windows-ps-manage
[使用 Azure PowerShell 创建和预配置基于 Linux 的虚拟机]: /documentation/articles/virtual-machines-linux-classic-createpowershell

[How to Create a Custom Virtual Machine Running Linux in Azure]: /documentation/articles/virtual-machines-linux-classic-create-custom
[如何使用 CLI 捕获将用作模板的 Linux 虚拟机]: /documentation/articles/virtual-machines-linux-classic-capture-image

[创建并上载包含 Linux 操作系统的虚拟硬盘]: /documentation/articles/virtual-machines-linux-classic-create-upload-vhd

[使用 Visual Studio 创建用于网站的虚拟机]: /documentation/articles/virtual-machines-windows-classic-web-app-visual-studio
[Deploy Azure Resources Using the Compute, Network, and Storage .NET Libraries]: /documentation/articles/virtual-machines-windows-csharp

[登录到虚拟机]: /documentation/articles/virtual-machines-linux-classic-log-on

[附加数据磁盘]: /documentation/articles/virtual-machines-linux-classic-attach-disk

[基本配置测试环境]: /documentation/articles/virtual-machines-windows-classic-test-config-env
[Azure 混合云测试环境]: /documentation/articles/virtual-machines-windows-classic-hybrid-test-env

[Create a Virtual Machine Running Linux]: /documentation/articles/virtual-machines-linux-quick-create-cli
[创建自定义虚拟机]: /documentation/articles/virtual-machines-windows-classic-createportal

<!---HONumber=Mooncake_0314_2016-->