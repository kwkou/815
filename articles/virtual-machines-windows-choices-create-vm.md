<properties
	pageTitle="创建 Windows VM 的不同方式 | Windows Azure"
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
	wacn.date="12/31/2015"/>

# 使用资源管理器创建 Windows 虚拟机的不同方式

Azure 提供不同方式来创建虚拟机，因为虚拟机适合于不同的用户和目的。这意味着，你需要在虚拟机及其创建方式上做出一些选择。本文提供了这些选项的摘要和说明链接。

[AZURE.INCLUDE [了解部署模型](../includes/learn-about-deployment-models-rm-include.md)]经典部署模型。

最近引入了 Azure 资源管理器模板作为一种创建和管理虚拟机的方法，而将其不同资源作为一个逻辑部署单元。此方法的说明包含如下（如果可用）。若要了解有关 Azure 资源管理器以及如何将资源作为一个单元管理的详细信息，请参阅[概述][]。

## 工具选项

### GUI：Azure 门户

Azure 门户的图形用户界面是一种试用虚拟机的简便方式，尤其是在你刚开始摸索 Azure 时。使用 Azure 门户创建虚拟机：

[创建运行 Windows 的虚拟机][]

### Azure PowerShell

如果你喜欢使用命令行界面，请选择Azure PowerShell，它提供 Windows PowerShell cmdlets for Azure 和自定义控制台。

有关 Azure PowerShell，请参阅[使用 Azure PowerShell 创建和预配置 Windows 虚拟机][]。若要在服务管理堆栈中创建虚拟机，请参阅[使用 Azure PowerShell 创建和预配置 Windows 虚拟机][]。

有关 Azure PowerShell，请参阅：

- [使用资源管理器和 PowerShell 创建 Windows VM][]
- [使用资源管理器和 Azure PowerShell 创建并预配置 Windows 虚拟机][]
- [使用 Azure 资源管理器模板与 PowerShell 来部署和管理虚拟机][]
- [使用资源管理器模板和 PowerShell 创建 Windows 虚拟机][]

### 开发环境：Visual Studio

[使用计算、网络和存储 .NET 库部署 Azure 资源][]

## 操作系统和映像选项

根据要运行的操作系统选择映像。Azure 及其合作伙伴提供了许多映像，其中一些映像包括应用程序和工具。或者，使用你自己的某一映像。使用此文中的信息查找你的应用程序所需的平台映像：[使用 Windows PowerShell 和 Azure CLI 来浏览和选择 Azure 虚拟机映像][]。

<!-- LINKS -->
[概述]: /documentation/articles/resource-group-overview

[创建运行 Windows 的虚拟机]: /documentation/articles/virtual-machines-windows-tutorial-classic-portal

[适合使用针对 Mac、Linux 和 Windows 的 Azure CLI 进行虚拟机操作的等效资源管理器和服务管理命令]: /documentation/articles/xplat-cli-azure-manage-vm-asm-arm
[使用 Azure 资源管理器模板和 Azure CLI 部署和管理虚拟机]: /documentation/articles/virtual-machines-deploy-rmtemplates-azure-cli
[使用资源管理器和 Azure PowerShell 创建并预配置 Windows 虚拟机]: /documentation/articles/virtual-machines-ps-create-preconfigure-windows-resource-manager-vms
[使用 Azure 资源管理器模板与 PowerShell 来部署和管理虚拟机]: /documentation/articles/virtual-machines-deploy-rmtemplates-powershell
[使用资源管理器和 PowerShell 创建 Windows VM]: /documentation/articles/virtual-machines-create-windows-powershell-resource-manager
[使用资源管理器模板和 PowerShell 创建 Windows 虚拟机]: /documentation/articles/virtual-machines-create-windows-powershell-resource-manager-template-simple


[使用 Windows PowerShell 和 Azure CLI 来浏览和选择 Azure 虚拟机映像]: /documentation/articles/resource-groups-vm-searching
[使用计算、网络和存储 .NET 库部署 Azure 资源]: /documentation/articles/virtual-machines-arm-deployment

[Sign in to the virtual machine]: /documentation/articles/virtual-machines-log-on-windows-server

[Base configuration test environment]: /documentation/articles/virtual-machines-base-configuration-test-environment

[Azure hybrid cloud test environments]: /documentation/articles/virtual-machines-hybrid-cloud-test-environments

<!---HONumber=Mooncake_1221_2015-->