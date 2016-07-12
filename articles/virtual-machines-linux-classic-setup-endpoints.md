<properties
	pageTitle="在经典 Linux 虚拟机上设置终结点 | Azure"
	description="了解如何在 Azure 经典管理门户中设置终结点以允许与 Azure 中的 Linux 虚拟机通信。"
	services="virtual-machines-linux"
	documentationCenter=""
	authors="cynthn"
	manager="timlt"
	editor=""
	tags="azure-service-management"/>

<tags
	ms.service="virtual-machines-linux"
	ms.date="04/19/2016"
	wacn.date="06/29/2016"/>

# 如何在经典 Azure 虚拟机上设置终结点

在 Azure 中使用经典部署模型创建的所有虚拟机都可以通过专用网络通道与同一云服务或虚拟网络中的其他虚拟机自动通信。但是，Internet 上的计算机或其他虚拟网络需要终结点才能定向虚拟机的入站网络流量。这篇文章也适用于 [Windows 虚拟机](/documentation/articles/virtual-machines-windows-classic-setup-endpoints/)。

> [AZURE.IMPORTANT]Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model/)。本文介绍使用经典部署模型。Azure 建议大多数新部署使用资源管理器模型。

在 Azure 经典管理门户中创建 Linux 虚拟机时，通常会为你自动创建用于安全外壳 (SSH) 的终结点，具体取决于你选择的操作系统。你可以在创建虚拟机时或之后根据需要配置其他终结点。

[AZURE.INCLUDE [virtual-machines-common-classic-setup-endpoints](../includes/virtual-machines-common-classic-setup-endpoints.md)]

## 下一步

* 你也可以在[服务管理模式](/documentation/articles/virtual-machines-command-line-tools/)下使用 Azure 命令行创建 VM 终结点。运行 **azure vm endpoint create** 命令。

<!---HONumber=Mooncake_0215_2016-->