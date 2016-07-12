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
	ms.date="03/11/2016"
	wacn.date="06/07/2016"/>

# 使用资源管理器创建 Windows 虚拟机的不同方式

Azure 提供不同方式来创建虚拟机，因为虚拟机适合于不同的用户和目的。这意味着，你需要在虚拟机及其创建方式上做出一些选择。本文提供了这些选项的摘要和说明链接。

> [AZURE.NOTE] Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model/)。本文介绍如何使用 Resource Manager 部署模型。Azure 建议对大多数新的部署使用该模型，而不是经典部署模型。


## Azure 门户预览

Azure 门户预览的图形用户界面是一种试用虚拟机的简便方式，尤其是在你刚开始摸索 Azure 时。使用 Azure 门户预览创建虚拟机：

[创建运行 Windows 的虚拟机](/documentation/articles/virtual-machines-windows-hero-tutorial/)

>[AZURE.NOTE] 经典管理门户只能创建管理经典部署模型的虚拟机

## Azure PowerShell

如果你喜欢在命令外壳中工作，可以使用 Azure PowerShell。

- [使用 PowerShell 创建 Windows VM](/documentation/articles/virtual-machines-windows-ps-create/)
- [使用资源管理器模板创建 Windows 虚拟机](/documentation/articles/virtual-machines-windows-ps-template/)

## 模板

虚拟机需要资源的组合（如可用性集和存储账户）。你无需单独部署和管理每个资源，而可以创建一个 Azure 资源管理器模板，以单个协调操作部署和设置所有资源。

- [使用资源管理器模板创建 Windows 虚拟机](/documentation/articles/virtual-machines-windows-ps-template/)

## Visual Studio

[使用计算、网络和存储 .NET 库部署 Azure 资源](/documentation/articles/virtual-machines-windows-csharp/)


<!---HONumber=Mooncake_0509_2016-->