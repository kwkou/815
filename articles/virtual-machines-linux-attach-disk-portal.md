<!-- Ibiza Portal: tested -->

<properties
	pageTitle="给 Linux 虚拟机附加数据磁盘 | Azure"
	description="如何使用资源管理器部署模型在 Azure 门户预览中将新磁盘或现有数据磁盘附加到 Linux VM。"
	services="virtual-machines-linux"
	documentationCenter=""
	authors="cynthn"
	manager="timlt"
	editor=""
	tags="azure-resource-manager"/>

<tags
	ms.service="virtual-machines-linux"
	ms.date="03/25/2016"
	wacn.date="06/29/2016"/>

# 如何在 Azure 门户预览中给 Linux 虚拟机附加数据磁盘

本文向你介绍如何通过 Azure 门户预览将新磁盘和现有磁盘附加到虚拟机。你也可以[在 Azure 门户预览中给 Windows 虚拟机附加数据磁盘](/documentation/articles/virtual-machines-windows-attach-disk-portal/)。在开始之前，请查看以下提示：

- 虚拟机的大小决定了可以附加多少个磁盘。有关详细信息，请参阅[虚拟机大小](/documentation/articles/virtual-machines-linux-sizes/)。
- 要使用高级存储，需要使用 DS 系列虚拟机。可以从高级存储帐户和标准存储帐户通过这些虚拟机使用磁盘。有关详细信息，请参阅[高级存储：适用于 Azure 虚拟机工作负荷的高性能存储](/documentation/articles/storage-premium-storage/)。
- 附加到虚拟机的磁盘实际上是 Azure 存储帐户中的 .vhd 文件。有关详细信息，请参阅[关于虚拟机的磁盘和 VHD](/documentation/articles/virtual-machines-linux-about-disks-vhds/)。
- 对于新磁盘，你不必首先创建它，因为 Azure 将在附加磁盘时创建该磁盘。
- 对于现有磁盘，Azure 存储帐户中必须要有可用的 .vhd 文件。你可以使用已经存在的 .vhd（如果该磁盘没有附加到另一虚拟机），或者将你自己的 .vhd 文件上载到存储帐户。

[AZURE.INCLUDE [virtual-machines-common-attach-disk-portal](../includes/virtual-machines-common-attach-disk-portal.md)]

## 后续步骤

在添加磁盘之后，需要准备磁盘以便在虚拟机的操作系统中使用：对于 Linux，请参阅[本文](/documentation/articles/virtual-machines-linux-classic-attach-disk/#how-to-initialize-a-new-data-disk-in-linux)中的“如何：在 Linux 中初始化新的数据磁盘”。

<!---HONumber=Mooncake_0411_2016-->