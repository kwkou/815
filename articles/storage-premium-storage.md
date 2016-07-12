<properties
	pageTitle="高级存储：适用于 Azure 虚拟机工作负荷的高性能存储 | Azure"
	description="高级存储为 Azure 虚拟机上运行的 I/O 密集型工作负载提供高性能、低延迟的磁盘支持。Azure DS 系列、DSv2 系列 VM 支持高级存储。"
	services="storage"
	documentationCenter=""
	authors="ms-prkhad"
	manager=""
	editor="tysonn"/>

<tags
	ms.service="storage"
	ms.date="04/26/2016"
	wacn.date="06/20/2016"/>


# 高级存储：适用于 Azure 虚拟机工作负荷的高性能存储

## 概述

Azure 高级存储为运行 I/O 密集型工作负荷的虚拟机提供高性能、低延迟的磁盘支持。在固态硬盘 (SSD) 上使用高级存储存储数据的虚拟机 (VM) 磁盘。可以将应用程序的 VM 磁盘迁移到 Azure 高级存储，以充分利用这些磁盘的速度和性能。

Azure VM 支持附加多个高级存储磁盘，使你的应用程序可以具有每个 VM 多达 64 TB 的存储空间。借助高级存储，应用程序对于每个 VM 可以实现 80,000 IOPS（每秒输入/输出操作数）和每秒 2000 MB 的磁盘吞吐量，并且读取操作的延迟非常低。

使用高级存储，Azure 提供的功能可真正将要求苛刻的企业应用程序（如 Dynamics AX、Dynamics CRM、Exchange Server、SharePoint 场和 SAP Business Suite）转移到云。你可以运行各种需要高级存储的一致高性能和低延迟的性能密集型数据库工作负荷，如 SQL Server、Oracle、MongoDB、MySQL、Redis。

>[AZURE.NOTE] 建议将任何需要高 IOPS 的虚拟机磁盘迁移到 Azure 高级存储，以便你的应用程序实现最佳性能。如果你的磁盘不需要高 IOPS，你可以通过在标准存储（将虚拟机磁盘数据存储在硬盘驱动器 (HDD) 上而不是 SSD 上）中对其进行维护来限制成本。

若要开始使用 Azure 高级存储，请访问[开始免费试用](/pricing/1rmb-trial/)页。有关将现有的虚拟机迁移到高级存储的信息，请参阅[迁移到 Azure 高级存储](/documentation/articles/storage-migration-to-premium-storage/)。


## 高级存储功能

**高级存储磁盘**：Azure 高级存储支持可连接到 DS、DSv2 系列 Azure VM 的 VM 磁盘。使用高级存储时，可以选择三种磁盘大小（即 P10 (128GiB)、P20 (512GiB) 和 P30 (1024GiB)），每种大小都有自身的性能规范。根据应用程序的要求，可以将一个或多个此类磁盘连接到 DS、DSv2 系列 VM。在下一部分[高级存储的可伸缩性和性能目标](#premium-storage-scalability-and-performance-targets)中，我们将详细地介绍规范。

**高级页 Blob**：高级存储支持 Azure 页 Blob（用于保存 Azure 虚拟机 (VM) 的永久性磁盘）。高级存储目前不支持 Azure 块 Blob、Azure 追加 Blob、Azure 文件、Azure 表或 Azure 队列。

**高级存储帐户**：若要开始使用高级存储，必须创建一个高级存储帐户。使用以下 SDK 库来创建“Premium_LRS”类型的存储帐户：[存储 REST API](http://msdn.microsoft.com/zh-cn/library/azure/dd179355.aspx) 版本 2014-02-14 或更高版本；[服务管理 REST API](http://msdn.microsoft.com/zh-cn/library/azure/ee460799.aspx) 版本 2014-10-01 或更高版本（经典部署）；[Azure 存储空间资源提供程序 REST API 参考](http://msdn.microsoft.com/zh-cn/library/azure/mt163683.aspx)（ARM 部署）；[Azure PowerShell](/documentation/articles/powershell-install-configure/) 版本 0.8.10 或更高版本。在以下有关[高级存储的可缩放性和性能目标](#premium-storage-scalability-and-performance-targets)的部分中了解高级存储帐户限制。

**高级本地冗余存储**：高级存储帐户仅支持使用本地冗余存储 (LRS) 作为复制选项，并在单个区域中保留三个数据副本。有关使用高级存储时的异地复制注意事项，请参阅本文中的[快照与复制 Blob](#snapshots-and-copy-blob) 部分。

Azure 使用存储帐户作为操作系统和数据磁盘的容器。如果你创建 Azure DS、DSv2 VM 并选择 Azure 高级存储帐户，操作系统和数据磁盘会存储在该存储帐户中。

可通过两种方式使用高级存储磁盘：
- 首先，创建新的高级存储帐户。接下来，在创建新的 DS、DSv2 VM 时选择存储配置设置中的高级存储帐户。或者，
- 在创建新的 DS、DSv2 VM 时，在存储配置设置中创建新的高级存储帐户，或者让 Azure 门户创建默认的高级存储帐户。

有关分步说明，请参阅本文后面的[快速入门](#quick-start)部分。

>[AZURE.NOTE] 高级存储帐户无法映射到自定义域名。

## DS、DSv2 系列 VM

高级存储支持 DS 系列、DSv2 系列 Azure 虚拟机 (VM)。DS 系列、DSv2 系列的 VM 可同时使用标准和高级存储磁盘。非 DS 的 VM 无法使用高级存储磁盘。有关可用 Azure VM 类型和大小的详细信息，请参阅 [Sizes for Virtual Machines（虚拟机大小）](/documentation/articles/virtual-machines-linux-sizes/)。以下是 DS、DSv2 VM 的一些功能。

有关可用 Azure VM 类型和 Windows VM 大小的信息，请参阅 [Windows VM sizes（Windows VM 大小）](/documentation/articles/virtual-machines-windows-sizes/)。有关 Linux VM 的 VM 类型和大小的信息，请参阅 [Linux VM sizes（Linux VM 大小）](/documentation/articles/virtual-machines-linux-sizes/)。

以下是 DS、DSv2 和 GS 系列 VM 的一些功能：

**云服务**：可以将 DS 系列 VM 添加到仅包含 DS 系列 VM 的云服务。请不要将 DS 系列虚拟机添加到包含非 DS 系列 VM 的现有云服务。你可以将现有 VHD 迁移到只运行 DS 系列 VM 的新云服务。如果想要保留托管 DS 系列 VM 的新云服务的相同虚拟 IP 地址 (VIP)，请使用[保留 IP 地址](../virtual-network/virtual-networks-instance-level-public-ip.md)。

**操作系统磁盘**：可以将 DS、DSv2 系列 Azure 虚拟机配置为使用标准存储帐户或高级存储帐户上托管的操作系统 (OS) 磁盘。如果 OS 磁盘只是用于引导，则你可以考虑使用基于标准存储的 OS 磁盘。这样既可以提高性价比，又可以在引导后提供类似于高级存储的性能。如果在除引导以外的 OS 磁盘上执行任何其他任务，请使用高级存储，因为它提供更好的性能。例如，如果你的应用程序要与 OS 磁盘相互读/写数据，则使用基于高级存储的 OS 磁盘可为 VM 提供更好的性能。

**数据磁盘**：可以在同一个 DS 系列、DSv2 系列 VM 中同时使用高级和标准存储磁盘。使用高级存储时，可以设置 DS、DSv2 系列 VM 并将多个持久性数据磁盘附加到 VM。如有需要，可以跨磁盘条带化，以增加卷的容量与性能。

> [AZURE.NOTE] 如果你使用[存储空间](http://technet.microsoft.com/zh-cn/library/hh831739.aspx)来条带化高级存储数据磁盘，应该以使用的每个磁盘一个列的方式来配置它。否则，条带化卷的整体性能可能会低于预期，因为磁盘之间的通信分配不平均。默认情况下，服务器管理器用户界面 (UI) 可让你设置最多包含 8 个磁盘的列。但如果磁盘超过 8 个，则你必须使用 PowerShell 来创建卷，并手动指定列数。否则，即使你有更多磁盘，服务器管理器 UI 仍会继续使用 8 个列。例如，如果在一个条带集中有 32 个磁盘，则你应该指定 32 列。可以使用 [New-VirtualDisk](http://technet.microsoft.com/zh-cn/library/hh848643.aspx) PowerShell cmdlet 的 NumberOfColumns 参数来指定虚拟磁盘使用的列数。有关详细信息，请参阅[存储空间概述](http://technet.microsoft.com/zh-cn/library/hh831739.aspx)和[存储空间常见问题](http://social.technet.microsoft.com/wiki/contents/articles/11382.storage-spaces-frequently-asked-questions-faq.aspx)。

**缓存**：DS、DSv2 系列 VM 都有独特的缓存功能，可让你获取超过基础高级存储磁盘性能的高级别吞吐量和延迟时间。可以在高级存储磁盘上将磁盘缓存策略配置为 ReadOnly、ReadWrite 或 None。所有高级数据磁盘的默认磁盘缓存策略都是 ReadOnly，而操作系统磁盘的磁盘缓存策略则是 ReadWrite。请使用正确的配置设置，以达到应用程序的最佳性能。例如，对于读取频繁或只读数据磁盘（如 SQL Server 数据文件），将磁盘缓存策略设置为“ReadOnly”。例如，对于写入频繁或只写数据磁盘（如 SQL Server 日志文件），将磁盘缓存策略设置为“None”。在 [Design for Performance with Premium Storage（使用高级存储器针对性能进行设计）](/documentation/articles/storage-premium-storage-performance/)中深入了解如何优化高级存储的设计。

**分析**：若要分析使用高级存储帐户磁盘的 VM 性能，可以在 Azure 门户中启用 Azure VM 诊断。有关详细信息，请参阅 [Azure Virtual Machine Monitoring with Azure Diagnostics Extension（使用 Azure Diagnostics 扩展监视 Azure 虚拟机）](https://azure.microsoft.com/blog/2014/09/02/windows-azure-virtual-machine-monitoring-with-wad-extension/)。若要查看磁盘性能，请使用操作系统工具，例如适用于 Windows VM 的 [Windows 性能监视器](https://technet.microsoft.com/zh-cn/library/cc749249.aspx)和适用于 Linux VM 的 [IOSTAT](http://linux.die.net/man/1/iostat)。

**VM 缩放限制和性能**：每个 DS 系列、DSv2 系列的 VM 大小都有 IOPS、带宽和每个 VM 可连接的磁盘数目的缩放限制和性能规范。使用高级存储磁盘配合 DS、DSv2 系列 VM 时，请确保 VM 上有足够的 IOPS 和带宽可用于驱动磁盘流量。
例如，STANDARD_DS1 VM 为高级存储磁盘通信提供每秒 32 MB 的专用带宽。P10 高级存储磁盘可以提供每秒 100 MB 的带宽。附加到此 VM 的 P10 高级存储磁盘最高只能达到每秒 32 MB，而不能像 P10 磁盘那样最高达到每秒 100 MB。

目前，DS 系列上的最大 VM 是 STANDARD_DS14，它可以跨所有磁盘最高提供每秒 512 MB。
请注意，这些限制只适用于磁盘流量，而不包括缓存命中和网络流量。VM 网络通信可以使用单独的带宽，这不同于高级存储磁盘的专用带宽。

有关 DS 系列、DSv2 系列 VM 的最大 IOPS 与吞吐量（带宽）的最新信息，请参阅 [Windows VM sizes（Windows VM 大小）](/documentation/articles/virtual-machines-windows-sizes/)或 [Linux VM sizes（Linux VM 大小）](/documentation/articles/virtual-machines-linux-sizes/)。

若要了解高级存储磁盘及其 IOPS 和吞吐量限制，请参阅本文的[使用高级存储时的可伸缩性和性能目标](#scalability-and-performance-targets-when-using-premium-storage)部分中的表格。

##<a id="scalability-and-performance-targets-when-using-premium-storage"></a>高级存储的可伸缩性和性能目标

在本部分，我们将说明在使用高级存储时必须考虑的可缩放性和性能目标。

### 高级存储帐户限制

高级存储帐户有以下可缩放性目标：

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

有关详细信息，请参阅 [Azure 存储空间缩放性和性能目标](/documentation/articles/storage-scalability-targets/)。

如果你的应用程序的需求超过了单个存储帐户的可伸缩性目标，则在生成应用程序时请让它使用多个存储帐户，并将数据分布在这些存储帐户中。例如，如果要将 51 TB 的磁盘附加到多个 VM，请将它们分布在两个存储帐户中，因为 35 TB 是单个高级存储帐户的限制。请确保单个高级存储帐户永远不会超过 35 TB 的设置磁盘。

### 高级存储磁盘限制

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
	<td>128 GiB</td>
	<td>512 GiB</td>
	<td>1024 GiB (1 TB)</td>
</tr>
<tr>
	<td><strong>每个磁盘的 IOPS</strong></td>
	<td>500</td>
	<td>2300</td>
	<td>5000</td>
</tr>
<tr>
	<td><strong>每个磁盘的吞吐量</strong></td>
	<td>每秒 100 MB </td>
	<td>每秒 150 MB </td>
	<td>每秒 200 MB </td>
</tr>
</tbody>
</table>

> [AZURE.NOTE] 请确保 VM 上有足够的带宽可用来驱动磁盘通信，如本文前面的 [DS、DSv2 系列 VM](#ds-dsv2-and-gs-series-vms) 部分中所述。否则，将会根据 VM 限制而不是上表中提到的磁盘限制，将磁盘吞吐量和 IOPS 约束为较小值。

以下是在高级存储可缩放性和性能目标方面必须知道的一些重要事项：

- **预配的容量和性能**：当预配高级存储磁盘时，不同于标准存储的是，你可以获得该磁盘的容量、IOPS 和吞吐量保证。例如，如果你要创建 P30 磁盘，Azure 将为该磁盘预配 1024 GB 存储容量、5000 IOPS 和每秒 200 MB 的吞吐量。应用程序可以使用全部或部分容量与性能。

- **磁盘大小**：Azure 会将磁盘大小映射（向上舍入）至表中指定的最接近高级存储磁盘选项。例如，大小为 100 GiB 的磁盘会分类为 P10 选项，每秒最多可执行 500 个 IO 单位，每秒吞吐量可达 100 MB。同样地，大小为 400 GiB 的磁盘会分类为 P20 选项，每秒最多可执行 2300 个 IO 单位，每秒吞吐量可达 150 MB。

	> [AZURE.NOTE] 你可以轻松增加现有磁盘的大小。例如，如果你想要将 30 GB 大小的磁盘增加到 128 GB 或 1 TB。或者，如果想要将 P20 磁盘转换为 P30 磁盘，因为需要更多容量或更多的 IOPS 和吞吐量。可以使用“Update-AzureDisk”PowerShell 命令配合“-ResizedSizeInGB”属性来扩展磁盘。若要执行此操作，需要先从 VM 分离磁盘或停止 VM。

- **IO 大小**：输入/输出 (I/O) 单位大小为 256 KB。如果要传送的数据少于 256 KB，会视为单个 I/O 单位。较大的 I/O 大小则会视为大小是 256 KB 的多个 I/O。例如，1100 KB 的 I/O 会视为五个 I/O 单位。

- **吞吐量**：吞吐量限制包括磁盘写入，以及不是从缓存进行的磁盘读取。例如，P10 磁盘有每秒 100 MB 的每磁盘吞吐量。P10 磁盘的有效吞吐量示例如下：
<table border="1" cellspacing="0" cellpadding="5" style="border: 1px solid #000000;">
<tbody>
<tr>
	<td><strong>每个 P10 磁盘的吞吐量上限</strong></td>
	<td><strong>从磁盘的非缓存读取</strong></td>
	<td><strong>对磁盘的非缓存写入</strong></td>
</tr>
<tr>
	<td>每秒 100 MB</td>
	<td>每秒 100 MB</td>
	<td>0</td>
</tr>
<tr>
	<td>每秒 100 MB</td>
	<td>0</td>
	<td>每秒 100 MB</td>
</tr>
<tr>
	<td>每秒 100 MB </td>
	<td>每秒 60 MB </td>
	<td>40 MB/秒 </td>
</tr>
</tbody>
</table>

- **缓存命中数**：缓存命中数不受到磁盘配置 IOPS/吞吐量的限制。例如，当你在 DS 系列、DSv2 系列 VM 上使用具有 ReadOnly 缓存设置的数据磁盘时，缓存提供的读取数不受高级存储磁盘限制的约束。因此，如果工作负荷以读取为主，可以从磁盘获得极高的吞吐量。请注意，缓存根据 VM 大小受到 VM 级别不同的 IOPS / 吞吐量的限制。DS 系列 VM 大约有 4000 IOPS，缓存与本地 SSD IO 是每个核心 33 MB/秒。

## 限制
如果应用程序的 IOPS 或吞吐量超出了分配的高级存储磁盘限制，或者 VM 上所有磁盘的总磁盘通信超出了 VM 可用的磁盘带宽限制，则可能会有限制情况。若要避免限制，建议根据设置的磁盘缩放性和性能目标，以及 VM 可用的磁盘带宽，来限制磁盘的挂起 I/O 请求数。

当应用程序设计为避免限制情况时，可以达到最低延迟。另一方面，如果磁盘的挂起 I/O 请求数过小，应用程序便无法利用磁盘可用的最大 IOPS 和吞吐量级别。

以下示例演示了如何计算限制级别。所有计算都是基于 256 KB 的 I/O 单位大小：

### 示例 1：
在 P10 磁盘上，应用程序一秒内有 495 个 16 KB 大小的 I/O 单位。这些被视为每秒 495 个 I/O 单位 (IOPS)。如果你在该秒内尝试 2 MB 的 I/O，I/O 单位的总数会等于 495 + 8。这是因为当 I/O 单位大小是 256 KB 时，2 MB 的 I/O 会产生 2048 KB / 256 KB = 8 个 I/O 单位。因为 495 + 8 的总和超出磁盘 500 的 IOPS 限制，因此会发生限制情况。

### 示例 2：
在 P10 磁盘上，应用程序有 400 个 256 KB 大小的 I/O 单位。消耗的总带宽是 (400 * 256) / 1024 = 100 MB/秒。P10 磁盘的吞吐量限制为每秒 100 MB。如果应用程序尝试在该秒内执行更多 I/O 便会发生限制情况，因为会超出分配的限制。

### 示例 3：
某个 DS4 VM 附加了两个 P30 磁盘。每个 P30 磁盘的吞吐量可达到每秒 200 MB。但是，DS4 VM 的总磁盘带宽容量为每秒 256 MB。因此，你无法在此 DS4 VM 上同时以最大吞吐量驱动附加的磁盘。若要解决此问题，你可以在一个磁盘上保持每秒 200 MB 的通信，在另一个磁盘上保持每秒 56 MB。如果磁盘通信的总和超出每秒 256 MB，磁盘通信将受到限制。

>[AZURE.NOTE] 如果磁盘通信大多包含小型 I/O，应用程序在达到吞吐量限制前很可能会先达到 IOPS 限制。另一方面，如果磁盘通信主要包含大型 I/O，应用程序可能会达到吞吐量限制，而非 IOPS 限制。可以使用最佳的 I/O 大小和限制磁盘的挂起 I/O 请求数，来最大化应用程序的 IOPS 和吞吐量容量。

若要了解如何使用高级存储设计高性能，请参阅 [Design for Performance with Premium Storage（使用高级存储器针对性能进行设计）](/documentation/articles/storage-premium-storage-performance/)一文。

##<a id="snapshots-and-copy-blob-when-using-premium-storage"></a> 快照和复制 Blob
可以像使用标准存储时创建快照的方式来为高级存储创建快照。由于高级存储仅支持使用本地冗余存储 (LRS) 作为复制选项，因此建议你创建快照，并将那些快照复制到地域冗余的标准存储帐户。有关详细信息，请参阅 [Azure 存储冗余选项](/documentation/articles/storage-redundancy/)。

如果磁盘已附加到 VM，在备份磁盘的页 Blob 上不允许某些 API 操作。例如，只要磁盘附加到 VM，你就无法在该 Blob 上执行[复制 Blob](http://msdn.microsoft.com/zh-cn/library/azure/dd894037.aspx) 操作。此时，你必须先使用[快照 Blob](http://msdn.microsoft.com/zh-cn/library/azure/ee691971.aspx) REST API 方法创建该 Blob 的快照，然后对该快照执行[复制 Blob](http://msdn.microsoft.com/zh-cn/library/azure/dd894037.aspx) 以复制附加的磁盘。或者，可以中断附加磁盘，然后在基础 Blob 上执行任何必要的操作。

以下限制适用于高级存储 Blob 快照：
<table border="1" cellspacing="0" cellpadding="5" style="border: 1px solid #000000;">
<tbody>
<tr>
	<td><strong>高级存储限制</strong></td>
	<td><strong>值</strong></td>
</tr>
<tr>
	<td>每个 Blob 的快照数上限</td>
	<td>100</td>
</tr>
<tr>
	<td>快照的存储帐户容量（只包括快照中的数据，不包括基本 Blob 中的数据）。</td>
	<td>10 TB</td>
</tr>
<tr>
	<td>连续快照的间隔时间下限</td>
	<td>10 分钟</td>
</tr>
</tbody>
</table>

若要维护快照的异地冗余副本，你可以使用 AzCopy 或“复制 Blob”将高级存储帐户中的快照复制到异地冗余的标准存储帐户。有关详细信息，请参阅 [Transfer data with the AzCopy Command-Line Utility（使用 AzCopy 命令行实用工具传输数据）](/documentation/articles/storage-use-azcopy/) 和 [Copy Blob（复制 Blob）](http://msdn.microsoft.com/zh-cn/library/azure/dd894037.aspx)。

有关对高级存储帐户中的页 Blob 执行 REST 操作的详细信息，请参阅 MSDN 库中的[对 Azure 高级存储使用 Blob 服务操作](https://msdn.microsoft.com/zh-cn/library/dn889922.aspx)。

## 在高级存储中使用 Linux VM
请参考以下重要说明，以了解如何在高级存储上配置 Linux VM：

- 对于缓存设置为“ReadOnly”或“None”的所有高级存储磁盘，必须在装入文件系统时禁用“屏障”，以实现高级存储的伸放性目标。对于这种情况你不需要屏障，因为写入高级存储支持的磁盘对于这些缓存设置是持久的。在成功完成写入请求时，数据已写入到持久存储。请根据你的文件系统，使用以下方法来禁用“屏障”：
	- 如果你使用的是 **reiserFS**，请使用装入选项“barrier=none”禁用屏障（要启用屏障，请使用“barrier=flush”）
	- 如果你使用的是 **ext3/ext4**，请使用装入选项“barrier=0”禁用屏障（要启用屏障，请使用“barrier=1”）
	- 如果你使用的是 **XFS**，请使用装入选项“nobarrier”禁用屏障（要启用屏障，请使用“barrier”）

- 对于缓存设置为“ReadWrite”的高级存储磁盘，应该启用屏障以实现写入持久性。
- 若要在重新启动 VM 后保留卷标，你必须使用对磁盘的 UUID 引用来更新 /etc/fstab。另请参阅 [How to Attach a Data Disk to a Linux Virtual Machine（如何将数据磁盘附加到 Linux 虚拟机）](/documentation/articles/virtual-machines-linux-classic-attach-disk/)

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
	<td>Ubuntu-12_04_5-LTS-amd64-server-20150119-en-us-30GB</td>
</tr>
<tr>
	<td>14.04</td>
	<td>3.13.0-44.73</td>
	<td>Ubuntu-14_04_1-LTS-amd64-server-20150123-en-us-30GB</td>
</tr>
<tr>
	<td>14.10</td>
	<td>3.16.0-29.39</td>
	<td>Ubuntu-14_10-amd64-server-20150202-en-us-30GB</td>
</tr>
<tr>
	<td>15.04</td>
	<td>3.19.0-15</td>
	<td>Ubuntu-15_04-amd64-server-20150422-en-us-30GB</td>
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
	<td>6.5、6.6、6.7、7.0</td>
	<td></td>
	<td>
		<a href="http://go.microsoft.com/fwlink/?LinkID=403033&clcid=0x409"> 需要 LIS 4.0 </a> </br>
		* 参阅以下注释
	</td>
</tr>
<tr>
	<td>7.1</td>
	<td>3.10.0-229.1.2.el7</td>
	<td>
		<a href="http://go.microsoft.com/fwlink/?LinkID=403033&clcid=0x409">建议使用 LIS 4.0</a> <br/>
		* 参阅以下注释
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

## 定价和计费
使用高级存储时，请注意以下计费方式：
- 高级存储磁盘大小
- 高级存储快照
- 出站数据传输

**高级存储磁盘大小**：高级存储磁盘的计费根据是磁盘的设置大小。Azure 会将磁盘大小（向上舍入）映射到[使用高级存储时的缩放性和性能目标](#scalability-and-performance-targets-when-using-premium-storage)部分的表中指定的最接近高级存储磁盘。任何已设置的磁盘都是按每月的高级存储优惠价格以每小时的方式计费。例如，如果你在设置完 P10 磁盘的 20 小时后删除它，则会以 20 小时计算 P10 解决方案的费用。这与写入磁盘的实际数据量或使用的 IOPS/吞吐量无关。

**高级存储快照**：高级存储上的快照会因为使用的额外容量而产生费用。有关快照的详细信息，请参阅[创建 Blob 的快照](http://msdn.microsoft.com/zh-cn/library/azure/hh488361.aspx)。

**出站数据传输**：[出站数据传输](/pricing/details/data-transfers/)（Azure 数据中心送出的数据）会产生带宽使用费。

有关高级存储、DS 系列、DSv2 系列 VM 定价的详细信息，请参阅：

- [Azure 存储定价](/home/features/storage/pricing/)
- [虚拟机定价](/home/features/virtual-machines/pricing/)

##<a id="quick-start"></a> 快速启动

##<a id="create-and-use-a-premium-storage-account-for-a-virtual-machine-data-disk"></a> 为虚拟机器数据磁盘创建和使用高级存储帐户

在本部分，我们将使用Azure PowerShell 和 Azure CLI 演示以下方案：
- 如何创建高级存储帐户。
- 如何在使用高级存储时创建虚拟机并将数据磁盘附加到该虚拟机。
- 如何更改已附加到虚拟机的数据磁盘的磁盘缓存策略：




### 通过 Azure PowerShell 使用高级存储创建 Azure 虚拟机

#### I.在 Azure PowerShell 中创建高级存储帐户

本 PowerShell 示例演示如何创建新的高级存储帐户并将使用该帐户的数据磁盘附加到新的 Azure 虚拟机。

1. 根据[如何安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure/) 中提供的步骤设置 PowerShell 环境。
2. 启动 PowerShell 控制台，连接到订阅，并在控制台窗口中运行以下 PowerShell cmdlet。如此 PowerShell 语句中所示，当你创建高级存储帐户时，必须将 **Type** 参数指定为 **Premium_LRS**。

		New-AzureStorageAccount -StorageAccountName "yourpremiumaccount" -Location "China East" -Type "Premium_LRS"

#### II.通过 Azure PowerShell 创建 Azure 虚拟机

接下来，请创建新的 DS 系列 VM，并在控制台窗口中运行以下 PowerShell cmdlet 以指定要使用高级存储：

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

#### III.通过 Azure PowerShell 附加高级存储数据磁盘

如果希望 VM 有更多的磁盘空间，请在创建虚拟机后于控制台窗口中运行以下 PowerShell cmdlet 以将新的数据磁盘附加到现有 DS 系列、DSv2 系列 VM。

    	$storageAccount = "yourpremiumaccount"
    	$vmName ="yourVM"
    	$vm = Get-AzureVM -ServiceName $vmName -Name $vmName
    	$LunNo = 1
    	$path = "http://" + $storageAccount + ".blob.core.chinacloudapi.cn/vhds/" + "myDataDisk_" + $LunNo + "_PIO.vhd"
    	$label = "Disk " + $LunNo
    	Add-AzureDataDisk -CreateNew -MediaLocation $path -DiskSizeInGB 128 -DiskLabel $label -LUN $LunNo -HostCaching ReadOnly -VM $vm | Update-AzureVm

#### IV.通过 Azure PowerShell 更改磁盘缓存策略

若要更新磁盘缓存策略，请记下附加的数据磁盘的 LUN 编号。运行以下命令，将附加到 LUN 编号 2 的数据磁盘更新为 ReadOnly。

		Get-AzureVM "myservice" -name "MyVM" | Set-AzureDataDisk -LUN 2 -HostCaching ReadOnly | Update-AzureVM

### 通过 Azure 命令行界面使用高级存储创建 Azure 虚拟机

[Azure 命令行接口](/documentation/articles/xplat-cli-install/) (Azure CLI) 提供一组可在 Azure 平台上运行的开放源代码跨平台命令。以下示例演示如何使用 Azure CLI（0.8.14 和更高版本）创建高级存储帐户、新的虚拟机，以及从高级存储帐户附加新的数据磁盘。

#### I.通过 Azure CLI 创建高级存储帐户

````
azure storage account create "premiumtestaccount" -l "China East" --type PLRS
````

#### II.通过 Azure CLI 创建 DS 系列虚拟机

	azure vm create -z "Standard_DS2" -l "China East" -e 22 "premium-test-vm"
		"b549f4301d0b4295b8e76ceb65df47d4__Ubuntu-14_10-amd64-server-20150202-en-us-30GB" -u "myusername" -p "passwd@123"

显示有关虚拟机的信息

	azure vm show premium-test-vm

#### III.通过 Azure CLI 附加新的高级数据磁盘

	azure vm disk attach-new premium-test-vm 20 https://premiumstorageaccount.blob.core.chinacloudapi.cn/vhd-store/data1.vhd

显示有关新数据磁盘的信息

	azure vm disk show premium-test-vm-premium-test-vm-0-201502210429470316

#### IV.更改磁盘缓存策略

若要使用 Azure CLI 更改某个磁盘上的缓存策略，请运行以下命令：

		$ azure vm disk attach -h ReadOnly <VM-Name> <Disk-Name>

请注意，缓存策略选项可以是 ReadOnly、None 或 ReadWrite。有关更多选项，请通过运行以下命令查看帮助：

		azure vm disk attach --help

## 常见问题

1. **是否可以同时将高级和标准数据磁盘附加到 DS、DSv2 系列 VM？**

	是的。可以同时将高级和标准数据磁盘附加到 DS、DSv2 系列 VM。

2. **是否可以同时将高级和标准数据磁盘附加到 D、Dv2 或 G 系列 VM？**

	不可以。只能将标准数据磁盘附加到非 DS、DSv2 系列的所有 VM。

3. **如果我从现有的 VHD（大小为 80 GB）创建高级数据磁盘，需要多少费用？**

	从 80 GB VHD 创建的高级数据磁盘被视为下一个可用的高级磁盘大小（P10 磁盘）。我们将根据 P10 磁盘价格收费。

4. **使用高级存储是否产生任何事务成本？**

	每个磁盘大小都有固定成本，其随着特定数量的 IOPS 和吞吐量预配。其他成本包括输出带宽和快照容量（如果适用）。有关详细信息，请参阅 [Azure 存储空间定价](/home/features/storage)。

5. **可以在何处存储 DS、DSv2 系列 VM 的引导诊断信息？**

	请创建标准存储帐户用于存储 DS、DSv2 系列 VM 的引导诊断信息。

6. **我可以从磁盘缓存获取多少 IOPS 和吞吐量？**

	DS 系列的缓存和本地 SSD 合并限制是每个核心 4000 IOPS，以及每个核心每秒 33 MB。

7. **DS、DSv2 系列 VM 中的本地 SSD 是什么？**

	本地 SSD 是 DS、DSv2 系列 VM 随附的临时存储。临时存储不需要额外的成本。建议不要使用此临时存储或本地 SSD 来存储应用程序数据，因为这些数据不会永久保存在 Azure Blob 存储中。

8. **是否可以将标准存储帐户转换成高级存储帐户？**

	无法将标准存储帐户转换成高级存储帐户（反之亦然）。必须使用所需的类型创建新的存储帐户，并将数据复制到新的存储帐户（如果适用）。

9. **如何将 D 系列 VM 转换成 DS 系列 VM？**

	请参阅迁移指南 [Migrating to Azure Premium Storage（迁移到 Azure 高级存储）](/documentation/articles/storage-migration-to-premium-storage/)，将工作负荷从使用标准存储帐户的 D 系列 VM 迁移到使用高级存储帐户的 DS 系列 VM。

## 后续步骤

有关 Azure 高级存储的详细信息，请参阅以下文章。

### 使用 Azure 高级存储进行设计和实现

- [Design for Performance with Premium Storage（使用高级存储针对性能进行设计）](/documentation/articles/storage-premium-storage-performance/)
- [对 Azure 高级存储使用 Blob 服务操作](https://msdn.microsoft.com/zh-cn/library/dn889922.aspx)

### 操作指南

- [迁移到 Azure 高级存储](/documentation/articles/storage-migration-to-premium-storage/)

### 博客文章

- [Azure Premium Storage Generally Available（正式推出 Azure 高级存储）](https://azure.microsoft.com/blog/azure-premium-storage-now-generally-available-2/)


[Image1]: ./media/storage-premium-storage/Azure_attach_premium_disk.png

<!---HONumber=Mooncake_0606_2016-->