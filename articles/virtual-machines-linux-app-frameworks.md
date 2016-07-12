<!-- not suitable for Mooncake -->

<properties
   pageTitle="使用模板部署常用应用程序框架 | Azure"
   description="使用 Azure Resource Manager 模板在 Linux 虚拟机创建常用应用程序框架，以便安装 Active Directory、Docker，等等。"
   services="virtual-machines"
   documentationCenter="virtual-machines-linux"
   authors="squillace"
   manager="timlt"
   editor=""
   tags="azure-resource-manager" />

<tags
	ms.service="virtual-machines-linux"
	ms.date="02/03/2016"
	wacn.date="06/07/2016"/>

# 使用 Azure Resource Manager 模板部署常用应用程序框架

工作负荷通常要求多项资源按设计运行。利用 Azure Resource Manager 模板，你不仅可以定义应用程序的配置方式，而且可以定义资源的部署方式，以便为配置的应用程序提供支持。本文介绍库中最常用的模板以及如何使用 、Azure PowerShell 或 Azure CLI 来部署它们。你也可以[这个主题的 Windows 版本](/documentation/articles/virtual-machines-windows-app-frameworks/)。

> [AZURE.NOTE]Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model/)。这篇文章介绍如何使用资源管理器部署模型，Azure 建议大多数新部署使用管理器部署模型代替经典部署模型。

[AZURE.INCLUDE [virtual-machines-common-app-frameworks](../includes/virtual-machines-common-app-frameworks.md)]

<!---HONumber=Mooncake_0411_2016-->