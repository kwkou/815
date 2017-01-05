<properties
	pageTitle="关于 Linux VM 的磁盘和 VHD | Azure"
	description="了解 Azure 中 Linux 虚拟机磁盘和 VHD 的基础知识。"
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
	ms.date="06/16/2016"
	wacn.date="01/05/2017"
	ms.author="cynthn"/>  


# 关于 Azure 虚拟机的磁盘和 VHD

就像其他任何计算机一样，Azure 中的虚拟机将磁盘用作存储操作系统、应用程序和数据的位置。所有 Azure 虚拟机都至少有两个磁盘，即 Linux 操作系统磁盘（对于 Linux VM）和临时磁盘。操作系统磁盘基于映像创建，操作系统磁盘和该映像实际上都存储在 Azure 存储帐户中的虚拟硬盘 (VHD) 内。虚拟机还可以有一个或多个数据磁盘，而这些磁盘也存储为 VHD。本文也适用于 [Windows 虚拟机](/documentation/articles/virtual-machines-windows-about-disks-vhds/)。

[AZURE.INCLUDE [了解部署模型](../../includes/learn-about-deployment-models-both-include.md)]

## 操作系统磁盘

每个虚拟机都附加了一个操作系统磁盘。默认情况下，它注册为 SATA 驱动器并标为 /dev/sda。此磁盘的最大容量为 1023 GB。

##临时磁盘

临时磁盘是自动为你创建的。在 Linux 虚拟机上，此磁盘通常为 /dev/sdb，通过 Azure Linux 代理格式化和装入 /mnt/resource。

临时磁盘的大小因虚拟机的大小而异。有关详细信息，请参阅 [Linux 虚拟机的大小](/documentation/articles/virtual-machines-linux-sizes/)。

>[AZURE.WARNING] 不要在临时磁盘上存储数据。该磁盘为应用程序和进程提供临时存储空间，只用于存储页面文件或交换文件等数据。

有关 Azure 如何使用临时磁盘的详细信息，请参阅 [Understanding the temporary drive on Azure Virtual Machines](https://blogs.msdn.microsoft.com/mast/2013/12/06/understanding-the-temporary-drive-on-windows-azure-virtual-machines/)（了解 Azure 虚拟机上的临时驱动器）

## 数据磁盘

数据磁盘是附加到虚拟机的 VHD，用于存储应用程序数据或其他需要保留的数据。数据磁盘注册为 SCSI 驱动器并且带有所选择的字母标记。每个数据磁盘的最大容量为 1023 GB。虚拟机的大小决定了可附加的磁盘数目，以及可用来托管磁盘的存储类型。

>[AZURE.NOTE] 有关虚拟机容量的详细信息，请参阅 [Linux 虚拟机的大小](/documentation/articles/virtual-machines-linux-sizes/)。

当你基于映像创建虚拟机时，Azure 将会创建操作系统磁盘。如果你使用包含数据磁盘的映像，则 Azure 还会在创建虚拟机时创建数据磁盘。否则，你需要在创建虚拟机后添加数据磁盘。

随时可以将数据磁盘添加到虚拟机，只需将该磁盘**附加**到虚拟机即可。你可以使用已上载或复制到存储帐户的 VHD，也可以让 Azure 为你创建 VHD。附加数据磁盘会将存储帐户中的 VHD 文件与虚拟机关联，方法是在 VHD 上放置“租约”，因此在仍附加 VHD 时无法从存储中删除它。

## 关于 VHD

Azure 中使用的 VHD 是在 Azure 的标准或高级存储帐户中作为页 Blob 存储的 .vhd 文件。有关页 blob 的详细信息，请参阅[了解块 blob 和页 blob](https://msdn.microsoft.com/zh-cn/library/ee691964.aspx)。有关高级存储的详细信息，请参阅[高级存储：适用于 Azure 虚拟机工作负荷的高性能存储](/documentation/articles/storage-premium-storage/)。

Azure 支持固定的磁盘 VHD 格式。固定格式在文件内对逻辑磁盘以线性方式布局，使磁盘偏移量 X 存储在 Blob 偏移量 X 的位置。在 Blob 末尾有一小段脚注，描述了 VHD 的属性。通常，由于大多数磁盘中都有较大的未使用区域，因此固定格式会浪费空间。不过，Azure 以稀疏格式存储 .vhd 文件，因此可兼获固定和动态格式磁盘的优点。有关更多详细信息，请参阅[虚拟硬盘入门](https://technet.microsoft.com/zh-cn/library/dd979539.aspx)。

Azure 中所有要用作磁盘或映像创建来源的 .vhd 文件都是只读文件。当你创建磁盘或映像时，Azure 将生成 .vhd 文件的副本。这些副本可以是只读文件，也可以是读写文件，具体取决于你使用 VHD 的方式。

当你基于映像创建虚拟机时，Azure 将为虚拟机创建磁盘，该磁盘是源 .vhd 文件的副本。为防止意外删除，Azure 对任何用于创建映像、操作系统磁盘或数据磁盘的源 .vhd 文件都设置了租约。

在删除源 .vhd 文件之前，需要先通过删除磁盘或映像来解除租约。若要删除由虚拟机当前用作操作系统磁盘的 .vhd 文件，可以通过删除虚拟机并删除所有关联的磁盘，一次性删除虚拟机、操作系统磁盘和源 .vhd 文件。但是，删除用作数据磁盘来源的 .vhd 文件需要按一定顺序执行几个步骤：先从虚拟机分离该磁盘，再删除该磁盘，然后才能删除 .vhd 文件。

>[AZURE.WARNING] 如果你从存储空间删除了源 .vhd 文件或删除了你的存储帐户，Microsoft 无法为你恢复数据。


## 故障排除
[AZURE.INCLUDE [virtual-machines-linux-lunzero](../../includes/virtual-machines-linux-lunzero.md)]

## 后续步骤

-  [附加磁盘](/documentation/articles/virtual-machines-linux-add-disk/)可为 VM 添加额外的存储。
-  [配置软件 RAID](/documentation/articles/virtual-machines-linux-configure-raid/) 以实现冗余。
-  [捕获 Linux 虚拟机](/documentation/articles/virtual-machines-linux-classic-capture-image/) 以便可以快速部署更多 VM。

<!---HONumber=Mooncake_1017_2016-->