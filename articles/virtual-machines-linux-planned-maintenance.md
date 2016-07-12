<properties
	pageTitle="Linux VM 的计划内维护 | Azure"
	description="了解什么是 Azure 计划内维护以及它如何影响正在 Azure 中运行的 Linux 虚拟机。"
	services="virtual-machines-linux"
	documentationCenter=""
	authors="drewm"
	manager="timlt"
	editor=""
	tags="azure-service-management,azure-resource-manager"/>

<tags
	ms.service="virtual-machines-linux"
	ms.date="01/05/2016"
	wacn.date="03/28/2016"/>


# Azure 中 Linux 虚拟机的计划内维护

了解什么事 Azure 计划内维护，以及它怎么影响你的 Linux 虚拟机的可用性。这篇文章同样适用于 [Windows 虚拟机](/documentation/articles/virtual-machines-windows-planned-maintenance/)。

[AZURE.INCLUDE [了解部署模型](../includes/learn-about-deployment-models-both-include.md)]

## Azure 为何要执行计划内维护

Azure 在全球范围内定期执行更新，以提高虚拟机所基于的主机基础结构的可靠性、性能及安全性。其中许多更新在执行时不会对虚拟机或云服务产生任何影响，其中包括内存保留更新。

不过，有些更新就需要重新启动虚拟机，以便向基础结构应用所需的更新。虚拟机会在我们修补基础结构时关闭，之后再重新启动。

请注意，有两类维护可能会影响虚拟机的可用性：计划内维护和计划外维护。本页介绍 Azure 如何执行计划内维护。有关计划外维护的详细信息，请参阅[了解计划内与计划外维护](/documentation/articles/virtual-machines-linux-manage-availability/)。

[AZURE.INCLUDE [virtual-machines-common-planned-maintenance](../includes/virtual-machines-common-planned-maintenance.md)]

<!---HONumber=Mooncake_0321_2016-->