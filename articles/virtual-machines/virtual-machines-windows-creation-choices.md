<properties
	pageTitle="创建 Windows VM 的不同方法 | Azure"
	description="列出使用 Resource Manager 创建 Windows 虚拟机的不同方法。"
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
	wacn.date="12/12/2016"
	ms.author="cynthn"/>

# 使用 Resource Manager 创建 Windows 虚拟机的不同方法

由于虚拟机适合不同的用户和目的，因此 Azure 可提供创建虚拟机的不同方法。这意味着需要对虚拟机及其创建方法做出选择。本文提供了选择摘要和说明链接。

## Azure 门户预览

使用 Azure 门户预览是试用虚拟机的一种简单方法，尤其是在刚开始使用 Azure 时。

[使用门户创建运行 Windows 的虚拟机](/documentation/articles/virtual-machines-windows-hero-tutorial/)

## 模板

虚拟机需要使用各种资源（如可用性集和存储账户）的组合。无需单独部署和管理每个资源，可以创建 Azure Resource Manager 模板，用于在单个协调操作中部署和预配所有资源。

- [使用 Resource Manager 模板创建 Windows 虚拟机](/documentation/articles/virtual-machines-windows-ps-template/)


## Azure PowerShell

如果喜欢在命令外壳中工作，可以使用 Azure PowerShell。

- [使用 PowerShell 创建 Windows VM](/documentation/articles/virtual-machines-windows-ps-create/)


## Visual Studio

通过 Azure Tools for Visual Studio 和 Azure SDK，使用 Visual Studio 构建、管理和部署 VM。

[Azure Tools for Visual Studio](https://www.visualstudio.com/features/azure-tools-vs)

<!---HONumber=Mooncake_Quality_Review_1118_2016-->