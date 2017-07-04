<properties
	pageTitle="管理 Linux VM 的可用性 | Azure"
	description="了解如何使用多个虚拟机来确保 Linux 应用程序在 Azure 中的高可用性"
	services="virtual-machines-linux"
	documentationCenter=""
	authors="cynthn"
	manager="timlt"
	editor="tysonn"
	tags="azure-resource-manager,azure-service-management"/>

<tags
	ms.service="virtual-machines-linux"
	ms.workload="infrastructure-services"
	ms.tgt_pltfrm="vm-linux"
	ms.devlang="na"
	ms.topic="article"
	ms.date="03/21/2017"
	wacn.date="05/31/2017"
	ms.author="cynthn"/>

# 管理虚拟机的可用性

了解如何设置和管理多个虚拟机，以确保 Linux 应用程序在 Azure 中的高可用性。你也可以[管理 Windows 虚拟机的可用性](/documentation/articles/virtual-machines-windows-manage-availability/)。

[AZURE.INCLUDE [learn-about-deployment-models](../../includes/learn-about-deployment-models-both-include.md)]

有关在资源管理器部署模型中使用 CLI 创建可用性集的说明，请参阅 [azure availset：用于管理可用性集的命令](/documentation/articles/azure-cli-arm-commands/#azure-availset-commands-to-manage-your-availability-sets)。

[AZURE.INCLUDE [virtual-machines-common-manage-availability](../../includes/virtual-machines-common-manage-availability.md)]

## 后续步骤

若要了解有关对虚拟机进行负载均衡的详细信息，请参阅[对虚拟机进行负载均衡](/documentation/articles/load-balancer-overview/)。

<!---HONumber=Mooncake_Quality_Review_1202_2016-->