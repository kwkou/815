<properties 
	pageTitle="高级存储：适用于 Azure 虚拟机工作负荷的高性能存储 | Azure"
	description="高级存储为 Azure 虚拟机上运行的 I/O 密集型工作负载提供高性能、低延迟的磁盘支持。Azure DS 系列和 GS 系列 VM 支持高级存储。"
	services="storage"
	documentationCenter=""
	authors="ms-prkhad"
	manager=""
	editor="tysonn"/>

<tags 
	ms.service="storage"
	ms.date="12/04/2015"
	wacn.date="01/14/2016"/>


# 高级存储：适用于 Azure 虚拟机工作负荷的高性能存储

## 概述

Azure 高级存储为运行 I/O 密集型工作负荷的虚拟机提供高性能、低延迟的磁盘支持。在固态硬盘 (SSD) 上使用高级存储存储数据的虚拟机 (VM) 磁盘。可以将应用程序的 VM 磁盘迁移到 Azure 高级存储，以充分利用这些磁盘的速度和性能。


Azure VM 支持附加多个高级存储磁盘，使你的应用程序可以具有每个 VM 多达 64 TB 的存储空间。借助高级存储，应用程序对于每个 VM 可以实现 80,000 IOPS（每秒输入/输出操作数）和每秒 2000 MB 的磁盘吞吐量，并且读取操作的延迟非常低。

使用高级存储，Azure 提供的功能可真正将要求苛刻的企业应用程序（如 Dynamics AX、Dynamics CRM、Exchange Server、SharePoint 场和 SAP Business Suite）转移到云。你可以运行各种需要高级存储的一致高性能和低延迟的性能密集型数据库工作负荷，如 SQL Server、Oracle、MongoDB、MySQL、Redis。

>[AZURE.NOTE]建议将任何需要高 IOPS 的虚拟机磁盘迁移到 Azure 高级存储，以便你的应用程序实现最佳性能。如果你的磁盘不需要高 IOPS，你可以通过在标准存储（将虚拟机磁盘数据存储在硬盘驱动器 (HDD) 上而不是 SSD 上）中对其进行维护来限制成本。

若要开始使用 Azure 高级存储，请访问[开始免费试用](/zh-cn/pricing/1rmb-trial/)页。有关将现有的虚拟机迁移到高级存储的信息，请参阅[迁移到 Azure 高级存储](/documentation/articles/storage-migration-to-premium-storage)。

## 有关高级存储的重要事项

以下是使用高级存储之前或使用高级存储时需考虑的重要事项列表：

- 若要使用高级存储，你必须有一个高级存储帐户。若要了解如何创建高级存储帐户，请参阅[为磁盘创建和使用高级存储帐户](#create-and-use-a-premium-storage-account-for-a-virtual-machine-data-disk)。

- 高级存储已在 [Azure 门户](https://manage.windowsazure.cn)中中提供并可通过以下 SDK 库来访问：[存储 REST API](http://msdn.microsoft.com/zh-cn/library/azure/dd179355.aspx) 版本 2014-02-14 或更高版本；[服务管理 REST API](http://msdn.microsoft.com/zh-cn/library/azure/ee460799.aspx) 版本 2014-10-01 或更高版本；[Azure PowerShell](/documentation/articles/powershell-install-configure) 版本 0.8.10 或更高版本。

- 以下地区提供受限的高级存储：中国东部。

- 高级存储仅支持 Azure 页 Blob，用于保存适用于 Azure 虚拟机 (VM) 的持久性磁盘。有关 Azure 页 Blob 的信息，请参阅[了解块 Blob 和页 Blob](http://msdn.microsoft.com/zh-cn/library/azure/ee691964.aspx)。高级存储不支持 Azure 块 Blob、Azure 文件、Azure 表或 Azure 队列。

- 高级存储帐户是本地冗余 (LRS) 帐户，在单一区域内会保留三份数据。有关使用高级存储时的异地复制注意事项，请参阅本文中的[使用高级存储时的快照与复制 Blob](#snapshots-and-copy-blob-when-using-premium-storage) 部分。

- 如果你希望 VM 磁盘使用高级存储帐户，你必须使用 DS 或 GS 系列的 VM。DS 或 GS 系列的 VM 可同时使用标准和高级存储磁盘。非 DS 或非 GS 系列的 VM 无法使用高级存储磁盘。有关可用 Azure VM 磁盘类型和大小的详细信息，请参阅 [Azure 的虚拟机和云服务大小](/documentation/articles/virtual-machines-size-specs)。

- 为 VM 设置高级存储磁盘的过程，与设置标准存储磁盘的过程类似。你需要为 Azure 磁盘和 VM 选择最适合的高级存储选项。根据高级产品的性能特征，VM 大小应适合工作负载。有关详细信息，请参阅[使用高级存储时的缩放性和性能目标](#scalability-and-performance-targets-when-using-premium-storage)。

- 高级存储帐户无法映射到自定义域名。

- 存储分析目前不支持高级存储。若要使用高级存储帐户上的磁盘分析 VM 的性能度量值，请使用基于操作系统的工具，如 [Windows 性能监视器](https://technet.microsoft.com/zh-cn/library/cc749249.aspx)（对于 Windows VM）和 [IOSTAT](http://linux.die.net/man/1/iostat)（对于 Linux VM）。你还可以在 Azure 管理门户上启用 Azure VM 诊断。有关详细信息，请参阅[使用 Azure Diagnostics 扩展监视 Azure 虚拟机](http://azure.microsoft.com/blog/2014/09/02/windows-azure-virtual-machine-monitoring-with-wad-extension/)。

## 使用高级存储磁盘
可通过两种方式使用高级存储磁盘：

- 首先创建新的高级存储帐户，然后在创建 VM 时使用它。
- 创建新的 DS 或 GS 系列 VM。创建 VM 时，你可以选择以前创建的高级存储帐户、创建新的高级存储帐户，或者让 Azure 门户创建默认的高级帐户。

Azure 使用存储帐户作为操作系统和数据磁盘的容器。换句话说，如果你创建 Azure DS 系列或 GS 系列的 VM 并选择 Azure 高级存储帐户，操作系统和数据磁盘会存储在该存储帐户中。


有关将现有的虚拟机迁移到高级存储的信息，请参阅[迁移到 Azure 高级存储](/documentation/articles/storage-migration-to-premium-storage)。
为充分利用高级存储的优点，请先使用帐户类型 *Premium_LRS* 创建一个高级存储帐户。为此，可以使用 [Azure 门户](https://manage.windowsazure.cn)、[Azure PowerShell](/documentation/articles/powershell-install-configure) 或[服务管理 REST API](http://msdn.microsoft.com/zh-cn/library/azure/ee460799.aspx)。有关分步说明，请参阅[为磁盘创建和使用高级存储帐户](#create-and-use-a-premium-storage-account-for-a-virtual-machine-data-disk)。

### 重要说明：

- 有关高级存储帐户的容量和带宽限制的详细信息，请参阅[使用高级存储时的缩放性和性能目标](#scalability-and-performance-targets-when-using-premium-storage)部分。如果你的应用程序的需求超过了单个存储帐户的可伸缩性目标，则在生成应用程序时请让它使用多个存储帐户，并将数据分布在这些存储帐户中。例如，如果要将 51 TB 的磁盘附加到多个 VM，请将它们分布在两个存储帐户中，因为 35 TB 是单个高级存储帐户的限制。请确保单个高级存储帐户永远不会超过 35 TB 的设置磁盘。
- 默认情况下，所有高级数据磁盘的磁盘缓存策略都是“只读的”，所有附加到 VM 的高级操作系统都是“读写的”。为使应用程序的 I/O 达到最佳性能，建议使用此配置设置。对于频繁写入或只写的磁盘（例如 SQL Server 日志文件），禁用磁盘缓存可获得更佳的应用程序性能。
- 确保 VM 上有足够的带宽来驱动磁盘通信。例如，STANDARD_DS1 VM 为高级存储磁盘通信提供每秒 32 MB 的专用带宽。这意味着，附加到此 VM 的 P10 高级存储磁盘最高只能达到每秒 32 MB，而不能像 P10 磁盘那样最高达到每秒 100 MB。同样，STANDARD_DS13 VM 可跨所有磁盘最高达到每秒 256 MB。目前，DS 系列上的最大 VM 是 STANDARD_DS14，它可以跨所有磁盘最高提供每秒 512 MB。GS 系列上的最大 VM 是 STANDARD_GS5，它可以跨所有磁盘最高提供每秒 2000 MB。

	请注意，这些限制只适用于磁盘流量，而不包括缓存命中和网络流量。VM 网络通信可以使用单独的带宽，这不同于高级存储磁盘的专用带宽。
	
	有关 DS 系列和 GS 系列 VM 的最大 IOPS 与吞吐量（带宽）的最新信息，请参阅 [Azure 的虚拟机和云服务大小](/documentation/articles/virtual-machines-size-specs)。若要了解高级存储磁盘及其 IOPS 和吞吐量限制，请参阅本文的[使用高级存储时的可伸缩性和性能目标](#scalability-and-performance-targets-when-using-premium-storage)部分中的表格。

> [AZURE.NOTE]缓存命中数不受到磁盘配置 IOPS/吞吐量的限制。也就是说，当你在 DS 系列或 GS 系列 VM 上使用具有 ReadOnly 缓存设置的数据磁盘时，缓存提供的读取数不受高级存储磁盘限制的约束。因此，如果工作负荷以读取为主，可以从磁盘获得极高的吞吐量。请注意，缓存根据 VM 大小受到 VM 级别不同的 IOPS / 吞吐量的限制。DS 系列 VM 大约有 4000 IOPS，缓存与本地 SSD IO 是每个核心 33 MB/秒。

- 可以在同一个 DS 系列或 GS 系列 VM 中同时使用高级和标准存储磁盘。
- 使用高级存储时，可以设置 DS 系列 VM 并将多个持久性数据磁盘附加到 VM。如有需要，可以跨磁盘条带化，以增加卷的容量与性能。如果你使用[存储空间](http://technet.microsoft.com/zh-cn/library/hh831739.aspx)来条带化高级存储数据磁盘，应该以使用的每个磁盘一个列的方式来配置它。否则，条带化卷的整体性能可能会低于预期，因为磁盘之间的通信分配不平均。默认情况下，服务器管理器用户界面 (UI) 可让你设置最多包含 8 个磁盘的列。但如果磁盘超过 8 个，则你必须使用 PowerShell 来创建卷，并手动指定列数。否则，即使你有更多磁盘，服务器管理器 UI 仍会继续使用 8 个列。例如，如果在一个条带集中有 32 个磁盘，则你应该指定 32 列。可以使用 [New-VirtualDisk](http://technet.microsoft.com/zh-cn/library/hh848643.aspx) PowerShell cmdlet 的 *NumberOfColumns* 参数来指定虚拟磁盘使用的列数。有关详细信息，请参阅[存储空间概述](http://technet.microsoft.com/zh-cn/library/jj822938.aspx)和[存储空间常见问题](http://social.technet.microsoft.com/wiki/contents/articles/11382.storage-spaces-frequently-asked-questions-faq.aspx)。
- 避免将 DS 系列虚拟机添加到包含非 DS 系列 VM 的现有云服务。可能的解决方法是将现有 VHD 迁移到只运行 DS 系列 VM 的新云服务。如果想要保留托管 DS 系列 VM 的新云服务的相同虚拟 IP 地址 (VIP)，请使用[保留 IP 地址](/documentation/articles/virtual-networks-configure-vnet-to-vnet-connection)功能。GS 系列 VM 只能添加到运行 G 系列 VM 的现有云服务。
- 可以将 DS 系列 Azure 虚拟机配置为使用标准存储帐户或高级存储帐户上托管的操作系统 (OS) 磁盘。如果 OS 磁盘只是用于引导，则你可以考虑使用基于标准存储的 OS 磁盘。这样既可以提高性价比，又可以在引导后提供类似于高级存储的性能。如果在除引导以外的 OS 磁盘上执行任何其他任务，请使用高级存储，因为它提供更好的性能。例如，如果你的应用程序要与 OS 磁盘相互读/写数据，则使用基于高级存储的 OS 磁盘可为 VM 提供更好的性能。
- 可以对高级存储使用 [Azure 命令行界面 (Azure CLI)](/documentation/articles/xplat-cli-install)。若要使用 Azure CLI 更改某个磁盘上的缓存策略，请运行以下命令：

	`$ azure vm disk attach -h ReadOnly <VM-Name> <Disk-Name>`

	请注意，缓存策略选项可以是 ReadOnly、None 或 ReadWrite。有关更多选项，请通过运行以下命令查看帮助：

	`azure vm disk attach --help`


##<a id="scalability-and-performance-targets-when-using-premium-storage"></a> 使用高级存储时的可伸缩性和性能目标

当你为某个高级存储帐户设置磁盘时，其每秒的输入/输出操作次数 (IOPS) 和吞吐量（带宽）取决于磁盘大小。目前有三种类型的高级存储磁盘：P10、P20 和 P30。每种类型各有特定的 IOPS 和吞吐量限制，如下表所示：

<table border="1" cellspacing="0" cellpadding="5" style="border: 1px solid #000000;">
<tbody>
<tr>
	<td><strong>高级存储磁盘类型</strong></td>
	<td><strong>P10</strong></td>
	<td><strong>P20</strong></td>
	<td><strong>P30</strong></td>
</tr>
<tr>
	<td><strong>磁盘大小</strong></td>
	<td>128 GB</td>
	<td>512 GB</td>
	<td>1024 GB (1 TB)</td>
</tr>
<tr>
	<td><strong>每个磁盘的 IOPS</strong></td>
	<td>500</td>
	<td>2300</td>
	<td>5000</td>
</tr>
<tr>
	<td><strong>每个磁盘的吞吐量</strong></td>
	<td>每秒 100 MB *</td>
	<td>每秒 150 MB *</td>
	<td>每秒 200 MB *</td>
</tr>
</tbody>
</table>

> [AZURE.NOTE]请确保 VM 上有足够的带宽可用来驱动磁盘通信，如本文前面的[使用高级存储磁盘](#using-premium-storage-for-disks)部分中所述。否则，将会根据 VM 限制而不是上表中提到的磁盘限制，将磁盘吞吐量和 IOPS 约束为较小值。

Azure 会将磁盘大小映射（向上舍入）至表中指定的最接近高级存储磁盘选项。例如，大小为 100 GB 的磁盘会分类为 P10 选项，每秒最多可执行 500 个 IO 单位，每秒吞吐量可达 100 MB。同样地，大小为 400 GB 的磁盘会分类为 P20 选项，每秒最多可执行 2300 个 IO 单位，每秒吞吐量可达 150 MB。

输入/输出 (I/O) 单位大小为 256 KB。如果要传送的数据少于 256 KB，会视为单个 I/O 单位。较大的 I/O 大小则会视为大小是 256 KB 的多个 I/O。例如，1100 KB 的 I/O 会视为五个 I/O 单位。

吞吐量限制包括磁盘写入，以及不是从缓存进行的磁盘读取。例如，P10 磁盘的非缓存读取和写入总和应小于或等于每秒 100 MB。同样地，单个 P10 磁盘可有每秒 100 MB 的非缓存读取或每秒 100 MB 的写入吞吐量，或是每秒 60 MB 的读取和每秒 40 MB 的写入吞吐量。

当你在 Azure 创建磁盘后，请根据应用程序的容量、性能、缩放性和高峰负载需求来选择最适合的高级存储磁盘产品。

> [AZURE.NOTE]你可以轻松增加现有磁盘的大小。例如，如果你想要将 30 GB 大小的磁盘增加到 128 GB 或 1 TB。或者，如果想要将 P20 磁盘转换为 P30 磁盘，因为需要更多容量或更多的 IOPS 和吞吐量。可以使用“Update-AzureDisk”PowerShell 命令配合“-ResizedSizeInGB”属性来扩展磁盘。若要执行此操作，需要先从 VM 分离磁盘或停止 VM。

下表描述高级存储帐户的缩放性目标：

<table border="1" cellspacing="0" cellpadding="5" style="border: 1px solid #000000;">
<tbody>
<tr>
	<td><strong>总帐户容量</strong></td>
	<td><strong>本地冗余存储帐户的总带宽</strong></td>
</tr>
<tr>
	<td>
	<ul>
       <li type=round>磁盘容量：35 TB</li>
       <li type=round>快照容量：10 TB</li>
    </ul>
	</td>
	<td>入站 + 出站最高每秒 50 Gbps</td>
</tr>
</tbody>
</table>

- 入站是指发送到存储帐户的所有数据（请求）。
- 出站是指从存储帐户接收的所有数据（响应）。

有关详细信息，请参阅 [Azure 存储服务缩放性和性能目标](http://msdn.microsoft.com/zh-cn/library/azure/dn249410.aspx)。

## 使用高级存储时的限制
如果应用程序的 IOPS 或吞吐量超出了分配的高级存储磁盘限制，或者 VM 上所有磁盘的总磁盘通信超出了 VM 可用的磁盘带宽限制，则可能会有限制情况。若要避免限制，建议根据设置的磁盘缩放性和性能目标，以及 VM 可用的磁盘带宽，来限制磁盘的挂起 I/O 请求数。

当应用程序设计为避免限制情况时，可以达到最低延迟。另一方面，如果磁盘的挂起 I/O 请求数过小，应用程序便无法利用磁盘可用的最大 IOPS 和吞吐量级别。

以下示例演示了如何计算限制级别：

### 示例 1：
在 P10 磁盘上，应用程序一秒内有 495 个 16 KB 磁盘大小的 I/O（相当于 495 个 I/O 单位）。如果你在该秒内尝试 2 MB 的 I/O，I/O 单位的总数会等于 495 + 8。这是因为当 I/O 单位大小是 256 KB 时，2 MB 的 I/O 会产生 2048 KB / 256 KB = 8 个 I/O 单位。因为 495 + 8 的总和超出磁盘 500 的 IOPS 限制，因此会发生限制情况。

### 注意： 
所有计算都是基于 256 KB 的 I/O 单位大小。

### 示例 2：
在 P10 磁盘上，应用程序有 400 个 256 KB 磁盘大小的 I/O。使用的总带宽为 (400 * 256) / 1024 = 100 MB/秒。请注意，每秒 100 MB 是 P10 磁盘的吞吐量限制。如果应用程序尝试在该秒内执行更多 I/O 便会发生限制情况，因为会超出分配的限制。

### 示例 3：
某个 DS4 VM 附加了两个 P30 磁盘。每个 P30 磁盘的吞吐量可达到每秒 200 MB。但是，DS4 VM 的总磁盘带宽容量为每秒 256 MB。因此，你无法在此 DS4 VM 上同时以最大吞吐量驱动附加的磁盘。若要解决此问题，你可以在一个磁盘上保持每秒 200 MB 的通信，在另一个磁盘上保持每秒 56 MB。如果磁盘通信的总和超出每秒 256 MB，磁盘通信将受到限制。

### 注意：
如果磁盘通信大多包含小型 I/O，应用程序在达到吞吐量限制前很可能会先达到 IOPS 限制。另一方面，如果磁盘通信主要包含大型 I/O，应用程序可能会达到吞吐量限制，而非 IOPS 限制。可以使用最佳的 I/O 大小和限制磁盘的挂起 I/O 请求数，来最大化应用程序的 IOPS 和吞吐量容量。

## 使用高级存储时的快照与复制 Blob
可以像使用标准存储时创建快照的方式来为高级存储创建快照。由于高级存储是本地冗余存储，因此建议你创建快照，并将那些快照复制到地域冗余的标准存储帐户。有关详细信息，请参阅 [Azure 存储冗余选项](http://msdn.microsoft.com/zh-cn/library/azure/dn727290.aspx)。

如果磁盘已附加到 VM，在备份磁盘的页 Blob 上不允许某些 API 操作。例如，只要磁盘附加到 VM，你就无法在该 Blob 上执行[复制 Blob](http://msdn.microsoft.com/zh-cn/library/azure/dd894037.aspx) 操作。此时，你必须先使用[快照 Blob](http://msdn.microsoft.com/zh-cn/library/azure/ee691971.aspx) REST API 方法创建该 Blob 的快照，然后对该快照执行[复制 Blob](http://msdn.microsoft.com/zh-cn/library/azure/dd894037.aspx) 以复制附加的磁盘。或者，可以中断附加磁盘，然后在基础 Blob 上执行任何必要的操作。

### 重要说明：

- 单个 Blob 的快照数限制为 100。每 10 分钟最多可获取一个快照。
- 每个高级存储帐户的最大快照容量为 10 TB。请注意，快照容量仅指快照中的数据总量，而不包括基本 Blob 中的数据。
- 若要维护快照的异地冗余副本，你可以使用 AzCopy 或“复制 Blob”将高级存储帐户中的快照复制到异地冗余的标准存储帐户。有关详细信息，请参阅[如何对 Azure 存储空间使用 AzCopy](/documentation/articles/storage-use-azcopy) 和[复制 Blob](http://msdn.microsoft.com/zh-cn/library/azure/dd894037.aspx)。
- 有关对高级存储帐户中的页 Blob 执行 REST 操作的详细信息，请参阅 MSDN 库中的[对 Azure 高级存储使用 Blob 服务操作](https://msdn.microsoft.com/zh-cn/library/dn889922.aspx)。

## 在高级存储中使用 Linux VM
请参考以下重要说明，以了解如何在高级存储上配置 Linux VM：

- 对于缓存设置为“ReadOnly”或“None”的所有高级存储磁盘，必须在装入文件系统时禁用“屏障”，以实现高级存储的伸放性目标。对于这种情况你不需要屏障，因为写入高级存储支持的磁盘对于这些缓存设置是持久的。在成功完成写入请求时，数据已写入到持久存储。请根据你的文件系统，使用以下方法来禁用“屏障”：
	- 如果你使用的是 **reiserFS**，请使用装入选项“barrier=none”禁用屏障（要启用屏障，请使用“barrier=flush”）
	- 如果你使用的是 **ext3/ext4**，请使用装入选项“barrier=0”禁用屏障（要启用屏障，请使用“barrier=1”）
	- 如果你使用的是 **XFS**，请使用装入选项“nobarrier”禁用屏障（要启用屏障，请使用“barrier”）

- 对于缓存设置为“ReadWrite”的高级存储磁盘，应该启用屏障以实现写入持久性。
- 要在重新启动 VM 后保留卷标，你必须使用对磁盘的 UUID 引用来更新 /etc/fstab。另请参阅 [如何将数据磁盘附加到 Linux 虚拟机。](/documentation/articles/virtual-machines-linux-how-to-attach-disk)

以下是我们使用高级存储验证过的 Linux 分发版。我们建议将 VM 升级到其中至少一个版本（或更新版本），以改进高级存储的性能和稳定性。此外，某些版本需要最新的 LIS（适用于 Azure 的 Linux Integration Services v4.0）。请使用下面提供的链接进行下载和安装。在我们完成其他验证后，将陆续在列表中添加更多映像。请注意，我们的验证表明，性能根据映像而有所不同，并且还取决于工作负荷特征和映像上的设置。不同的映像已针对不同种类的工作负荷进行优化。
<table border="1" cellspacing="0" cellpadding="5" style="border: 1px solid #000000;">
<tbody>
<tr>
	<td><strong>分发</strong></td>
	<td><strong>版本</strong></td>
	<td><strong>支持的内核</strong></td>
	<td><strong>支持的映像</strong></td>
</tr>
<tr>
	<td rowspan="4"><strong>Ubuntu</strong></td>
	<td>12.04</td>
	<td>3.2.0-75.110</td>
	<td>Ubuntu-12_04_5-LTS-amd64-server-20150119-zh-CN-30GB</td>
</tr>
<tr>
	<td>14.04</td>
	<td>3.13.0-44.73</td>
	<td>Ubuntu-14_04_1-LTS-amd64-server-20150123-zh-CN-30GB</td>
</tr>
<tr>
	<td>14.10</td>
	<td>3.16.0-29.39</td>
	<td>Ubuntu-14_10-amd64-server-20150202-zh-CN-30GB</td>
</tr>
<tr>
	<td>15.04</td>
	<td>3.19.0-15</td>
	<td>Ubuntu-15_04-amd64-server-20150422-zh-CN-30GB</td>
</tr>
<tr>
	<td><strong>SUSE</strong></td>
	<td>SLES 12</td>
	<td>3.12.36-38.1</td>
	<td>suse-sles-12-priority-v20150213<br>suse-sles-12-v20150213</td>
</tr>
<tr>
	<td><strong>CoreOS</strong></td>
	<td>584.0.0</td>
	<td>3.18.4</td>
	<td>CoreOS 584.0.0</td>
</tr>
<tr>
	<td rowspan="2"><strong>CentOS</strong></td>
	<td>6.5, 6.6, 6.7, 7.0</td>
	<td></td>
	<td>
		<a href="http://go.microsoft.com/fwlink/?LinkID=403033&clcid=0x409"> 需要 LIS 4.0 </a> </br>
		*请参阅以下注释
	</td>
</tr>
<tr>
	<td>7.1</td>
	<td>3.10.0-229.1.2.el7</td>
	<td>
		<a href="http://go.microsoft.com/fwlink/?LinkID=403033&clcid=0x409"> 建议使用 LIS 4.0 </a> <br/>
		*请参阅以下注释
	</td>
</tr>

<tr>
	<td rowspan="2"><strong>Oracle</strong></td>
	<td>6.4</td>
	<td></td>
	<td><a href="http://go.microsoft.com/fwlink/?LinkID=403033&clcid=0x409"> 需要 LIS 4.0 </a></td>
</tr>
<tr>
	<td>7.0</td>
	<td></td>
	<td>联系支持人员以获取详细信息</td>
</tr>
</tbody>
</table>


### Openlogic CentOS 的 LIS 驱动程序

运行 OpenLogic CentOS VM 的客户应该运行以下命令来安装最新的驱动程序：

	sudo rpm -e hypervkvpd  ## (may return error if not installed, that's OK)
	sudo yum install microsoft-hyper-v

需要重新启动才能激活新的驱动程序。



## 使用高级存储时的定价和计费方式
使用高级存储时，请注意以下计费方式：

- 高级存储磁盘的计费根据是磁盘的设置大小。Azure 会将磁盘大小（向上舍入）映射到[使用高级存储时的缩放性和性能目标](#scalability-and-performance-targets-when-using-premium-storage)部分的表中指定的最接近高级存储磁盘。任何已设置的磁盘都是按每月的高级存储优惠价格以每小时的方式计费。例如，如果你在设置完 P10 磁盘的 20 小时后删除它，则会以 20 小时计算 P10 解决方案的费用。这与写入磁盘的实际数据量或使用的 IOPS/吞吐量无关。
- 高级存储上的快照会因为使用的额外容量而产生费用。有关快照的详细信息，请参阅[创建 Blob 的快照](http://msdn.microsoft.com/zh-cn/library/azure/hh488361.aspx)。
- [出站数据传输](/pricing/details/data-transfer/)（Azure 数据中心送出的数据）会产生带宽使用费。

有关高级存储与 DS 和 GS 系列 VM 定价的详细信息，请参阅：

- [Azure 存储定价](/home/features/storage/#price)
- [虚拟机定价](/home/features/virtual-machines#price)

##<a id="create-and-use-a-premium-storage-account-for-a-virtual-machine-data-disk"></a> 为虚拟机数据磁盘创建和使用高级存储帐户
本部分演示如何使用Azure PowerShell 和 Azure 命令行界面 (Azure CLI) 创建高级存储帐户。另外，还演示了高级存储帐户的示例用例：在使用高级存储帐户时创建虚拟机，并将数据磁盘附加到虚拟机。

### 通过 Azure PowerShell 使用高级存储创建 Azure 虚拟机
本 PowerShell 示例演示如何创建新的高级存储帐户并将使用该帐户的数据磁盘附加到新的 Azure 虚拟机。

1. 根据[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure) 中提供的步骤设置 PowerShell 环境。
2. 启动 PowerShell 控制台，连接到订阅，并在控制台窗口中运行以下 PowerShell cmdlet。如此 PowerShell 语句中所示，当你创建高级存储帐户时，必须将 **Type** 参数指定为 **Premium_LRS**。

		New-AzureStorageAccount -StorageAccountName "yourpremiumaccount" -Location "China East" -Type "Premium_LRS"

3. 接下来，请创建新的 DS 系列 VM，并在控制台窗口中运行以下 PowerShell cmdlet 以指定你要使用高级存储。可以使用相同的步骤创建 GS 系列 VM。在命令中指定适当的 VM 大小。例如，Standard_GS2：

    	$storageAccount = "yourpremiumaccount"
    	$adminName = "youradmin"
    	$adminPassword = "yourpassword"
    	$vmName ="yourVM"
    	$location = "China East"
    	$imageName = "55bc2b193643443bb879a78bda516fc8__Windows-Server-2012-R2-20150726-en.us-127GB.vhd"
    	$vmSize ="Standard_DS2"
    	$OSDiskPath = "https://" + $storageAccount + ".blob.core.chinacloudapi.cn/vhds/" + $vmName + "_OS_PIO.vhd"
    	$vm = New-AzureVMConfig -Name $vmName -ImageName $imageName -InstanceSize $vmSize -MediaLocation $OSDiskPath
    	Add-AzureProvisioningConfig -Windows -VM $vm -AdminUsername $adminName -Password $adminPassword
    	New-AzureVM -ServiceName $vmName -VMs $VM -Location $location

4. 如果希望 VM 有更多的磁盘空间，请在创建虚拟机后于控制台窗口中运行以下 PowerShell cmdlet 以将新的数据磁盘附加到现有 DS 系列或 GS 系列 VM。

    	$storageAccount = "yourpremiumaccount"
    	$vmName ="yourVM"
    	$vm = Get-AzureVM -ServiceName $vmName -Name $vmName
    	$LunNo = 1
    	$path = "http://" + $storageAccount + ".blob.core.chinacloudapi.cn/vhds/" + "myDataDisk_" + $LunNo + "_PIO.vhd"
    	$label = "Disk " + $LunNo
    	Add-AzureDataDisk -CreateNew -MediaLocation $path -DiskSizeInGB 128 -DiskLabel $label -LUN $LunNo -HostCaching ReadOnly -VM $vm | Update-AzureVm

### 通过 Azure 命令行界面使用高级存储创建 Azure 虚拟机

[Azure 命令行界面](/documentation/articles/xplat-cli-install) (Azure CLI) 提供一组可在 Azure 平台上运行的开放源代码跨平台命令。以下示例演示如何使用 Azure CLI（0.8.14 和更高版本）创建高级存储帐户、新的虚拟机，以及从高级存储帐户附加新的数据磁盘。

#### 创建高级存储帐户

	azure storage account create "premiumtestaccount" -l "china east" --type PLRS


#### 创建 DS 系列虚拟机

	azure vm create -z "Standard_DS2" -l "china east" -e 22 "premium-test-vm"
		"b39f27a8b8c64d52b05eac6a62ebad85__Ubuntu-14_10-amd64-server-20150202-zh-CN-30GB" -u "myusername" -p "passwd@123"

#### 显示有关虚拟机的信息

	azure vm show premium-test-vm

#### 附加新的数据磁盘

	azure vm disk attach-new premium-test-vm 20 https://premiumstorageaccount.blob.core.chinacloudapi.cn/vhd-store/data1.vhd

#### 显示有关新数据磁盘的信息

	azure vm disk show premium-test-vm-premium-test-vm-0-201502210429470316

## 后续步骤

[对 Azure 高级存储使用 Blob 服务操作](https://msdn.microsoft.com/zh-cn/library/dn889922.aspx)

- [迁移到 Azure 高级存储](/documentation/articles/storage-migration-to-premium-storage)。[创建运行 Windows 的虚拟机](/documentation/articles/virtual-machines-windows-tutorial-classic-portal)

[Azure 的虚拟机和云服务大小](/documentation/articles/virtual-machines-size-specs)

[存储服务文档](/documentation/services/storage)

[Image1]: ./media/storage-premium-storage-preview-portal/Azure_pricing_tier.png

<!---HONumber=Mooncake_0104_2016-->