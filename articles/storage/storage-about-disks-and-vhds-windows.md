<properties
    pageTitle="关于 Azure Windows VM 的磁盘和 VHD | Azure"
    description="了解 Azure 中 Windows 虚拟机磁盘和 VHD 的基础知识。"
    services="storage"
    documentationcenter=""
    author="robinsh"
    manager="timlt"
    editor="tysonn" />
<tags
    ms.assetid="0142c64d-5e8c-4d62-aa6f-06d6261f485a"
    ms.service="storage"
    ms.workload="storage"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="02/06/2017"
    wacn.date="03/20/2017"
    ms.author="robinsh" />  


# 关于 Azure Windows VM 的磁盘和 VHD
如其他任何计算机一样，Azure 中的虚拟机将磁盘用作存储操作系统、应用程序和数据的位置。所有 Azure 虚拟机都至少有两个磁盘，即 Windows 操作系统磁盘和临时磁盘。操作系统磁盘通过映像创建，操作系统磁盘和该映像都存储在 Azure 存储帐户中的虚拟硬盘 (VHD)。虚拟机还可以有一个或多个数据磁盘，而这些磁盘也存储为 VHD。

在本文中，我们将讨论磁盘的不同用法，然后讨论可以创建和使用的不同磁盘类型。本文也适用于 [Linux 虚拟机](/documentation/articles/storage-about-disks-and-vhds-linux/)。

[AZURE.INCLUDE [了解部署模型](../../includes/learn-about-deployment-models-both-include.md)]

## VM 使用的磁盘

让我们来看看 VM 如何使用磁盘。

### 操作系统磁盘
每个虚拟机都附加了一个操作系统磁盘。默认情况下，它注册为 SATA 驱动器并标为 C: 盘。此磁盘的最大容量为 1023 GB。

### 临时磁盘
临时磁盘是自动为你创建的。临时磁盘默认标记为 D: 盘，用于存储 pagefile.sys。

临时磁盘的大小因虚拟机的大小而异。有关详细信息，请参阅 [Windows 虚拟机的大小](/documentation/articles/virtual-machines-windows-sizes/)。

> [AZURE.WARNING]
不要在临时磁盘上存储数据。该磁盘为应用程序和进程提供临时存储空间，只用于存储页面文件或交换文件等数据。若要将此磁盘重新映射到其他驱动器号，请参阅 [更改 Windows 临时磁盘的驱动器号](/documentation/articles/virtual-machines-windows-classic-change-drive-letter/)。
> 

有关 Azure 如何使用临时磁盘的详细信息，请参阅 [了解 Azure 虚拟机上的临时驱动器](https://blogs.msdn.microsoft.com/mast/2013/12/06/understanding-the-temporary-drive-on-windows-azure-virtual-machines/)

### 数据磁盘
数据磁盘是附加到虚拟机的 VHD，用于存储应用程序数据或其他需要保留的数据。数据磁盘注册为 SCSI 驱动器并且带有所选择的字母标记。每个数据磁盘的最大容量为 1023 GB。虚拟机的大小决定可附加的磁盘数量，以及可用于托管磁盘的存储类型。

> [AZURE.NOTE]
有关虚拟机容量的详细信息，请参阅 [Windows 虚拟机的大小](/documentation/articles/virtual-machines-windows-sizes/)。
> 

在通过映像创建虚拟机时，Azure 将会创建操作系统磁盘。如果你使用包含数据磁盘的映像，则 Azure 还会在创建虚拟机时创建数据磁盘。否则，你需要在创建虚拟机后添加数据磁盘。

随时可以将数据磁盘添加到虚拟机，只需将该磁盘**附加**到虚拟机即可。可以使用已上传或复制到存储帐户的 VHD，也可以让 Azure 创建 VHD。附加数据磁盘会将 VHD 文件与 VM 关联，方法是在 VHD 上放置“租约”，因此在仍附加 VHD 时无法从存储中删除它。


[AZURE.INCLUDE [storage-about-vhds-and-disks-windows-and-linux](../../includes/storage-about-vhds-and-disks-windows-and-linux.md)]

## 最后一个建议：对非托管标准磁盘使用 TRIM 

如果使用非托管标准磁盘 (HDD)，应启用 TRIM。TRIM 会丢弃磁盘上未使用的块，使用户只需为实际使用的存储付费。如果创建了较大的文件，然后将其删除，这样可以节省成本。

可以运行此命令来检查 TRIM 设置。在 Windows VM 上打开命令提示符，然后键入：


	fsutil behavior query DisableDeleteNotify


如果该命令返回 0，则表示正确启用了 TRIM。如果返回 1，请运行以下命令启用 TRIM：


	fsutil behavior set DisableDeleteNotify 0


<!-- Might want to match next-steps from overview of managed disks -->

## 后续步骤
* [附加磁盘](/documentation/articles/virtual-machines-windows-attach-disk-portal/)可为 VM 添加额外的存储。
* [将 Windows VM 映像上载到 Azure](/documentation/articles/virtual-machines-windows-upload-image/)，以便在创建新的 VM 时使用。
* [更改 Windows 临时磁盘的驱动器号](/documentation/articles/virtual-machines-windows-classic-change-drive-letter/)，使应用程序能够将 D: 盘用于数据。

<!---HONumber=Mooncake_0313_2017-->