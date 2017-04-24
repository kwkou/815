<properties
    pageTitle="在 Azure 上优化 Linux VM | Azure"
    description="了解一些优化提示，以确保正确设置你的 Linux VM，从而在 Azure 上获得最佳性能"
    keywords="linux 虚拟机,虚拟机 linux,ubuntu 虚拟机"
    services="virtual-machines-linux"
    documentationcenter=""
    author="rickstercdn"
    manager="timlt"
    editor="tysonn"
    tags="azure-resource-manager"
    translationtype="Human Translation" />
<tags
    ms.assetid="8baa30c8-d40e-41ac-93d0-74e96fe18d4c"
    ms.service="virtual-machines-linux"
    ms.workload="infrastructure-services"
    ms.tgt_pltfrm="vm-linux"
    ms.devlang="na"
    ms.topic="article"
    ms.date="09/06/2016"
    wacn.date="04/24/2017"
    ms.author="rclaus"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="02d4d82b35ad891a2cc981a2af1369f8403311d9"
    ms.lasthandoff="04/14/2017" />

# <a name="optimize-your-linux-vm-on-azure"></a>在 Azure 上优化 Linux VM
通过命令行或门户创建运行 Linux 虚拟机 (VM) 是一项很简单的操作。 本教程说明如何在 Azure 平台上设置 VM 以确保优化其性能。 本主题使用 Ubuntu Server VM，不过你也可以[将自己的映像作为模板](/documentation/articles/virtual-machines-linux-create-upload-generic/)来创建 Linux 虚拟机。  

## <a name="prerequisites"></a>先决条件
本主题假设你已有一个有效的 Azure 订阅（[注册试用版](/pricing/1rmb-trial/)），并已在 Azure 订阅中预配 VM。 请确保已安装最新的 [Azure CLI 2.0](https://docs.microsoft.com/zh-cn/cli/azure/install-az-cli2) 并使用 [az login](https://docs.microsoft.com/zh-cn/cli/azure/#login) 登录到 Azure 订阅，然后[创建 VM](/documentation/articles/virtual-machines-linux-quick-create-cli/)。

[AZURE.INCLUDE [azure-cli-2-azurechinacloud-environment-parameter](../../includes/azure-cli-2-azurechinacloud-environment-parameter.md)]

## <a name="azure-os-disk"></a>Azure OS 磁盘
在 Azure 中创建 Linux VM 后，它将具有两个与之关联的磁盘。 **/dev/sda** 是 OS 磁盘，**/dev/sdb** 是临时磁盘。  请勿将主要 OS 磁盘 (**/dev/sda**) 用于操作系统以外的用途，因为它已针对快速启动 VM 进行优化，无法为工作负荷提供良好的性能。 若要获得持久且经过优化的数据存储，可以将一个或多个磁盘附加到 VM。 

## <a name="adding-disks-for-size-and-performance-targets"></a>添加磁盘以实现大小和性能目标
根据 VM 大小，可以分别在 A 系列、D 系列和 G 系列计算机上额外附加最多 16 个、32 个和 64 个磁盘，每个磁盘最大可为 1 TB。 可以根据空间和 IOps 要求以及自己的需要添加额外的磁盘。 标准存储的每个磁盘的性能目标为 500 IOps，高级存储的每个磁盘的性能目标最高为 5000 IOps。  有关高级存储磁盘的详细信息，请参阅[高级存储：适用于 Azure VM 的高性能存储](/documentation/articles/storage-premium-storage/)

对于缓存设置为“ReadOnly”或“None”的高级存储磁盘，必须在 Linux 中装入文件系统时禁用“barrier”（屏障）才能达到最高 IOps。 你不需要屏障，因为写入高级存储支持的磁盘对于这些缓存设置是持久的。

* 如果使用的是 **reiserFS**，请使用装入选项 `barrier=none` 禁用屏障（若要启用屏障，请使用 `barrier=flush`）
* 如果使用的是 **ext3/ext4**，请使用装入选项 `barrier=0` 禁用屏障（若要启用屏障，请使用 `barrier=1`）
* 如果使用的是 **XFS**，请使用装入选项 `nobarrier` 禁用屏障（若要启用屏障，请使用 `barrier`）

## <a name="unmanaged-storage-account-considerations"></a>非托管存储帐户注意事项

如果使用非托管磁盘创建 VM，请务必从区域与 VM 相同的存储帐户附加磁盘，以确保高度邻近性并降低网络延迟。  每个标准存储帐户最多有 20k IOps 和 500 TB 大小的容量。  此限制大约相当于 40 个重度使用的磁盘，包括 OS 磁盘和用户创建的任何数据磁盘。 高级存储帐户没有 IOps 上限，但有 32 TB 的大小限制。 

在处理 IOps 很高的工作负荷时，如果为磁盘选择了标准存储，则可能需要将磁盘拆分到多个存储帐户才能避免达到标准存储帐户 20,000 IOps 的限制。 VM 中可以混合来自不同存储帐户和不同存储帐户类型的磁盘，以实现最佳配置。

## <a name="your-vm-temporary-drive"></a>VM 临时驱动器
默认情况下，创建 VM 时，Azure 将提供 OS 磁盘 (**/dev/sda**) 和临时磁盘 (**/dev/sdb**)。  额外添加的所有磁盘显示为 **/dev/sdc**、**/dev/sdd**、**/dev/sde**，依此类推。 临时磁盘 (**/dev/sdb**) 上的所有数据均不具有持久性，因此当发生 VM 调整大小、重新部署或维护等特定事件，从而迫使 VM 重新启动时，数据可能会丢失。  临时磁盘的类型和大小与在部署时选择的 VM 大小相关。 所有高级大小的 VM（DS、G 和 DS_V2 系列），临时驱动器均由本地 SSD 提供支持，因此可以实现最高 48k IOps 的附加性能。 

## <a name="linux-swap-file"></a>Linux 交换文件
如果你的 Azure VM 来自 Ubuntu 或 CoreOS 映像，则可以使用 CustomData 将 cloud-config 发送到 cloud-init。 如果已[上载使用 cloud-init 的自定义 Linux 映像](/documentation/articles/virtual-machines-linux-upload-vhd/)，则还可以使用 cloud-init 配置交换分区。

在 Ubuntu 云映像上，必须使用 cloud-init 配置交换分区。 有关详细信息，请参阅 [AzureSwapPartitions](https://wiki.ubuntu.com/AzureSwapPartitions)。

对于不带 cloud-init 支持的映像，从 Azure 应用商店部署的 VM 映像具有与 OS 集成的 VM Linux 代理。 此代理使 VM 可以与各种 Azure 服务进行交互。 假设你从 Azure 应用商店部署了标准映像，需要执行以下操作来正确配置 Linux 交换文件设置：

找出并修改 **/etc/waagent.conf** 文件中的两个条目。 这些条目控制专用交换文件的存在状态和交换文件的大小。 要修改的参数是 `ResourceDisk.EnableSwap=N` 和 `ResourceDisk.SwapSizeMB=0` 

将参数更改为以下设置：

* ResourceDisk.EnableSwap=Y
* ResourceDisk.SwapSizeMB={符合需要的大小，以 MB 为单位} 

做出更改后，需要重新启动 waagent 或重新启动 Linux VM 才能反映这些更改。  使用 `free` 命令查看可用空间时，即可知道已实现更改并已创建交换文件。 以下示例在修改 **waagent.conf** 文件后创建了 512MB 的交换文件：

    azuseruser@myVM:~$ free
                total       used       free     shared    buffers     cached
    Mem:       3525156     804168    2720988        408       8428     633192
    -/+ buffers/cache:     162548    3362608
    Swap:       524284          0     524284

## <a name="io-scheduling-algorithm-for-premium-storage"></a>高级存储的 I/O 调度算法
随着 2.6.18 Linux 内核的推出，默认 I/O 调度算法已从 Deadline 更改为 CFQ（完全公平的队列算法）。 对于随机访问 I/O 模式，CFQ 与 Deadline 之间的性能差异可忽略不计。  对于磁盘 I/O 模式以循序为主的基于 SSD 的磁盘，切换回到 NOOP 或 Deadline 算法可以实现更好的 I/O 性能。

### <a name="view-the-current-io-scheduler"></a>查看当前的 I/O 调度器
请使用以下命令：  

    cat /sys/block/sda/queue/scheduler

用户会看到以下输出，指示当前的计划程序。  

    noop [deadline] cfq

### <a name="change-the-current-device-devsda-of-io-scheduling-algorithm"></a>更改当前设备 (/dev/sda) 的 I/O 计划算法
使用以下命令：  

    azureuser@myVM:~$ sudo su -
    root@myVM:~# echo "noop" >/sys/block/sda/queue/scheduler
    root@myVM:~# sed -i 's/GRUB_CMDLINE_LINUX=""/GRUB_CMDLINE_LINUX_DEFAULT="quiet splash elevator=noop"/g' /etc/default/grub
    root@myVM:~# update-grub

> [AZURE.NOTE]
> 为 **/dev/sda** 单独应用此设置毫无用处。 在有序 I/O 支配 I/O 模式的所有数据磁盘上设置。  

用户应该会看到以下输出，指示已成功重新生成 **grub.cfg** 并且默认计划程序已更新为 NOOP。  

    Generating grub configuration file ...
    Found linux image: /boot/vmlinuz-3.13.0-34-generic
    Found initrd image: /boot/initrd.img-3.13.0-34-generic
    Found linux image: /boot/vmlinuz-3.13.0-32-generic
    Found initrd image: /boot/initrd.img-3.13.0-32-generic
    Found memtest86+ image: /memtest86+.elf
    Found memtest86+ image: /memtest86+.bin
    done

对于 Redhat 分发系列，只需以下命令：   

    echo 'echo noop >/sys/block/sda/queue/scheduler' >> /etc/rc.local

## <a name="using-software-raid-to-achieve-higher-iops"></a>使用软件 RAID 来实现更高的 I/Ops
如果工作负荷所需的 IOps 超过单个磁盘的极限，则需要使用包含多个磁盘的软件 RAID 配置。 由于 Azure 已在本地结构层执行磁盘复原，因此可以通过 RAID-0 条带化配置获得最高级别的性能。  在 Azure 环境中预配和创建磁盘，将这些磁盘附加到 Linux VM，然后分区、格式化并装入驱动器。  有关在 Azure 中针对 Linux VM 配置软件 RAID 设置的详细信息，请参阅 **[Configuring Software RAID on Linux](/documentation/articles/virtual-machines-linux-configure-raid/)**（在 Linux 上配置软件 RAID）文档。

## <a name="next-steps"></a>后续步骤
请记住，如同有关优化的所有文章中所述，需要在每次更改之前和之后执行测试，以衡量更改所造成的影响。  优化是一个逐序渐进的过程，在环境中不同的计算机上会产生不同的效果。  对某一项配置有用的做法不一定适用于其他配置。

其他有用资源的链接： 

* [高级存储：适用于 Azure 虚拟机工作负荷的高性能存储](/documentation/articles/storage-premium-storage/)
* [Azure Linux 代理用户指南](/documentation/articles/virtual-machines-linux-agent-user-guide/)
* [优化 Azure Linux VM 上的 MySQL 性能](/documentation/articles/virtual-machines-linux-classic-optimize-mysql/)
* [在 Linux 上配置软件 RAID](/documentation/articles/virtual-machines-linux-configure-raid/)
<!--Update_Description: wording update-->