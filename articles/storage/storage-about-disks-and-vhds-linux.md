<properties
    pageTitle="关于 Azure Linux VM 的磁盘和 VHD | Azure"
    description="了解 Azure 中 Linux 虚拟机磁盘和 VHD 的基础知识。"
    services="storage"
    documentationcenter=""
    author="robinsh"
    manager="timlt"
    editor="tysonn" />
<tags
    ms.assetid="7be8dd52-98f7-4187-9b78-55197915bc9b"
    ms.service="storage"
    ms.workload="storage"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="02/14/2017"
    wacn.date="03/20/2017"
    ms.author="robinsh" />  


# 关于 Azure Linux VM 的磁盘和 VHD
如其他任何计算机一样，Azure 中的虚拟机将磁盘用作存储操作系统、应用程序和数据的位置。所有 Azure 虚拟机都至少有两个磁盘，即 Linux 操作系统磁盘和临时磁盘。操作系统磁盘通过映像创建，操作系统磁盘和该映像实际上都存储在 Azure 存储帐户中的虚拟硬盘 (VHD)。虚拟机还可以有一个或多个数据磁盘，而这些磁盘也存储为 VHD。

在本文中，我们将讨论磁盘的不同用法，然后讨论可以创建和使用的不同磁盘类型。本文也适用于 [Windows 虚拟机](/documentation/articles/storage-about-disks-and-vhds-windows/)。

[AZURE.INCLUDE [了解部署模型](../../includes/learn-about-deployment-models-both-include.md)]

## VM 使用的磁盘

让我们来看看 VM 如何使用磁盘。

## 操作系统磁盘
每个虚拟机都附加了一个操作系统磁盘。默认情况下，它注册为 SATA 驱动器并标为 /dev/sda。此磁盘的最大容量为 1023 GB。

## 临时磁盘
临时磁盘是自动为你创建的。在 Linux 虚拟机上，此磁盘通常为 /dev/sdb，通过 Azure Linux 代理格式化和装入 /mnt/resource。

临时磁盘的大小因虚拟机的大小而异。有关详细信息，请参阅 [Linux 虚拟机的大小](/documentation/articles/virtual-machines-linux-sizes/)。

> [AZURE.WARNING]
>不要在临时磁盘上存储数据。该磁盘为应用程序和进程提供临时存储空间，只用于存储页面文件或交换文件等数据。
> 

有关 Azure 如何使用临时磁盘的详细信息，请参阅 [了解 Azure 虚拟机上的临时驱动器](https://blogs.msdn.microsoft.com/mast/2013/12/06/understanding-the-temporary-drive-on-windows-azure-virtual-machines/)

## 数据磁盘
数据磁盘是附加到虚拟机的 VHD，用于存储应用程序数据或其他需要保留的数据。数据磁盘注册为 SCSI 驱动器并且带有所选择的字母标记。每个数据磁盘的最大容量为 1023 GB。虚拟机的大小决定了可附加的磁盘数目，以及可用来托管磁盘的存储类型。

> [AZURE.NOTE]
>有关虚拟机容量的详细信息，请参阅 [Linux 虚拟机的大小](/documentation/articles/virtual-machines-linux-sizes/)。
> 

当你基于映像创建虚拟机时，Azure 将会创建操作系统磁盘。如果你使用包含数据磁盘的映像，则 Azure 还会在创建虚拟机时创建数据磁盘。否则，你需要在创建虚拟机后添加数据磁盘。

随时可以将数据磁盘添加到虚拟机，只需将该磁盘**附加**到虚拟机即可。可以使用已上传或复制到存储帐户的 VHD，也可以让 Azure 创建 VHD。附加数据磁盘会将 VHD 文件与 VM 关联，方法是在 VHD 上放置“租约”，因此在仍附加 VHD 时无法从存储中删除它。

[AZURE.INCLUDE [storage-about-vhds-and-disks-windows-and-linux](../../includes/storage-about-vhds-and-disks-windows-and-linux.md)]

## 故障排除
[AZURE.INCLUDE [virtual-machines-linux-lunzero](../../includes/virtual-machines-linux-lunzero.md)]

## 后续步骤
* [附加磁盘](/documentation/articles/virtual-machines-linux-add-disk/)可为 VM 添加额外的存储。
* [配置软件 RAID](/documentation/articles/virtual-machines-linux-configure-raid/) 以实现冗余。
* [捕获 Linux 虚拟机](/documentation/articles/virtual-machines-linux-classic-capture-image/) 以便可以快速部署更多 VM。

<!---HONumber=Mooncake_0313_2017-->