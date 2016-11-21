<properties
	pageTitle="创建 Windows VM 的不同方式 | Azure"
	description="列出使用资源管理器创建 Windows 虚拟机的不同方式。"
	services="virtual-machines-windows"
	documentationCenter=""
	authors="cynthn"
	manager="timlt"
	editor=""
	tags="azure-resource-manager,azure-service-management"/>

<tags
	ms.service="virtual-machines-windows"
	ms.devlang="na"
	ms.topic="article"
	ms.tgt_pltfrm="vm-windows"
	ms.workload="infrastructure-services"
	ms.date="09/27/2016"
	wacn.date="11/21/2016"
	ms.author="cynthn"/>

# 使用资源管理器创建 Windows 虚拟机的不同方式

Azure 提供不同方式来创建虚拟机，因为虚拟机适合于不同的用户和目的。这意味着，你需要在虚拟机及其创建方式上做出一些选择。本文提供了这些选项的摘要和说明链接。

## Azure 门户预览

使用 Azure 门户预览是一种试用虚拟机的简便方式，尤其是在你刚开始摸索 Azure 时。

[使用门户创建运行 Windows 的虚拟机](/documentation/articles/virtual-machines-windows-hero-tutorial/)

## 模板

虚拟机需要资源的组合（如可用性集和存储账户）。你无需单独部署和管理每个资源，而可以创建一个 Azure 资源管理器模板，以单个协调操作部署和设置所有资源。

- [使用资源管理器模板创建 Windows 虚拟机](/documentation/articles/virtual-machines-windows-ps-template/)


## Azure PowerShell

如果你喜欢在命令外壳中工作，可以使用 Azure PowerShell。

- [使用 PowerShell 创建 Windows VM](/documentation/articles/virtual-machines-windows-ps-create/)


## Visual Studio

结合使用 Visual Studio 和 Azure Tools for Visual Studio 以及 Azure SDK，生成、管理和部署 VM。

[Azure Tools for Visual Studio](https://www.visualstudio.com/features/azure-tools-vs)


<!---HONumber=Mooncake_0718_2016-->