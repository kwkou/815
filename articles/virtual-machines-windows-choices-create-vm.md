<properties
	pageTitle="创建 Windows VM 的不同方式 | Azure"
	description="列出使用资源管理器创建 Windows 虚拟机的不同方式。"
	services="virtual-machines"
	documentationCenter=""
	authors="cynthn"
	manager="timlt"
	editor=""
	tags="azure-resource-manager"/>

<tags
	ms.service="virtual-machines"
	ms.date="10/22/2015"
	wacn.date="02/17/2016"/>

# 创建 Windows 虚拟机的不同方式

Azure 提供不同方式来创建虚拟机，因为虚拟机适合于不同的用户和目的。这意味着，你需要在虚拟机及其创建方式上做出一些选择。本文提供了这些选项的摘要和说明链接。

## 工具选项

### GUI：Azure 门户

Azure 门户的图形用户界面是一种试用虚拟机的简便方式，尤其是在你刚开始摸索 Azure 时。使用 Azure 门户创建虚拟机：

[创建运行 Windows 的虚拟机][]

### 命令 Shell：Azure CLI 或 Azure PowerShell

如果你喜欢使用命令行解释器，请选择适用于 Mac 和 Linux 用户的 Azure 命令行界面 (CLI) 或 Azure PowerShell，后者提供 cmdlet for Azure 和自定义控制台。

有关 Azure CLI，请参阅：

- [将适用于 Mac、Linux 和 Windows 的 Azure CLI 与 Azure 服务管理配合使用](/documentation/articles/virtual-machines-command-line-tools).

有关 Azure PowerShell，请参阅：

- [在 Azure 中创建 SQL Server 虚拟机 (PowerShell)](/documentation/articles/virtual-machines-sql-server-create-vm-with-powershell)
- [使用 PowerShell 来部署和管理虚拟机][]

## 操作系统和映像选项

根据要运行的操作系统选择映像。Azure 及其合作伙伴提供了许多映像，其中一些映像包括应用程序和工具。或者，使用你自己的某一映像。使用此文中的信息查找你的应用程序所需的平台映像：[使用 Windows PowerShell 和 Azure CLI 来浏览和选择 Azure 虚拟机映像][]。

<!-- LINKS -->
[概述]: /documentation/articles/resource-group-overview

[创建运行 Windows 的虚拟机]: /documentation/articles/virtual-machines-windows-tutorial-classic-portal

[适合使用针对 Mac、Linux 和 Windows 的 Azure CLI 进行虚拟机操作的等效资源管理器和服务管理命令]: /documentation/articles/xplat-cli-azure-manage-vm-asm-arm

[使用 PowerShell 来部署和管理虚拟机]: /documentation/articles/virtual-machines-manage-vms-powershell


[使用 Windows PowerShell 和 Azure CLI 来浏览和选择 Azure 虚拟机映像]: /documentation/articles/resource-groups-vm-searching

[Sign in to the virtual machine]: /documentation/articles/virtual-machines-log-on-windows-server

[Base configuration test environment]: /documentation/articles/virtual-machines-base-configuration-test-environment

[Azure hybrid cloud test environments]: /documentation/articles/virtual-machines-hybrid-cloud-test-environments

<!---HONumber=Mooncake_1221_2015-->