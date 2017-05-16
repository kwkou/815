<properties
    pageTitle="在 Azure 中规划 VM 备份基础结构 | Azure"
    description="规划在 Azure 中备份虚拟机时的重要注意事项"
    services="backup"
    documentationcenter=""
    author="markgalioto"
    manager="carmonm"
    editor=""
    keywords="备份 vm, 备份虚拟机" />
<tags
    ms.assetid="19d2cf82-1f60-43e1-b089-9238042887a9"
    ms.service="backup"
    ms.workload="storage-backup-recovery"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="4/5/2017"
    ms.author="markgal;trinadhk"
    wacn.date="05/15/2017"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="3ff18e6f95d8bbc27348658bc5fce50c3320cf0a"
    ms.openlocfilehash="423ab952e7f2406f0e003517bc3ad87561ea58d3"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/15/2017" />

# <a name="plan-your-vm-backup-infrastructure-in-azure"></a>在 Azure 中计划 VM 备份基础结构
本文提供性能和资源建议，帮助规划 VM 备份基础结构。 文中还定义了备份服务的主要方面；这些方面对于决定体系结构、容量规划和计划安排至关重要。 如果已[准备好环境](/documentation/articles/backup-azure-vms-prepare/)，请首先进行此规划，然后再开始[备份 VM](/documentation/articles/backup-azure-vms/)。 如需有关 Azure 虚拟机的详细信息，请参阅[虚拟机文档](/documentation/services/virtual-machines/)。

## <a name="how-does-azure-back-up-virtual-machines"></a>Azure 虚拟机备份原理
当 Azure 备份服务在计划的时间启动备份作业时，它将触发进行时间点快照拍摄所需的备份扩展。 Azure 备份服务在 Windows 中使用 _VMSnapshot_ 扩展，在 Linux 中使用 _VMSnapshotLinux_ 扩展。 在第一个 VM 备份期间安装扩展。 若要安装扩展，VM 必须处于运行状态。 如果 VM 未运行，备份服务将创建基础存储的快照（因为在 VM 停止时不会发生任何应用程序写入）。

创建 Windows VM 快照时，备份服务与卷影复制服务 (VSS) 互相配合，来获取虚拟机磁盘的一致性快照。 如果要备份 Linux VM，可以编写自定义脚本来确保创建 VM 快照时的一致性。 本文的后面部分将提供有关调用这些脚本的详细信息。

Azure 备份服务创建快照后，数据将传输到保管库。 为最大限度地提高效率，服务仅标识和传输自上次备份以后已更改的数据块。

![Azure 虚拟机备份体系结构](./media/backup-azure-vms-introduction/vmbackup-architecture.png)

数据传输完成后，将会删除快照并创建恢复点。

> [AZURE.NOTE]
> 1. 在备份过程中，Azure 备份不包括附加到虚拟机的临时磁盘。 有关详细信息，请参阅[临时存储](https://blogs.msdn.microsoft.com/mast/2013/12/06/understanding-the-temporary-drive-on-windows-azure-virtual-machines/)博客。
> 2. 由于 Azure 备份创建存储级快照，并将该快照传输到保管库，因此在备份作业完成前，不要更改存储帐户密钥。
>

### <a name="data-consistency"></a>数据一致性
备份和还原业务关键数据十分复杂，因为必须在生成数据的应用程序仍在运行时备份业务关键数据。 为了解决此问题，Azure 备份对 Windows 和 Linux VM 均支持应用程序一致的备份

#### <a name="windows-vm"></a>Windows VM
Azure 备份将在 Windows VM 上创建 VSS 完整备份（深入了解 [VSS 完整备份](http://blogs.technet.com/b/filecab/archive/2008/05/21/what-is-the-difference-between-vss-full-backup-and-vss-copy-backup-in-windows-server-2008.aspx)）。 若要启用 VSS 复制备份，需要在 VM 上设置以下注册表项。

    [HKEY_LOCAL_MACHINE\SOFTWARE\MICROSOFT\BCDRAGENT]
    "USEVSSCOPYBACKUP"="TRUE"

#### <a name="linux-vms"></a>Linux VM
Azure 备份提供脚本框架。 若要确保备份 Linux VM 时的应用程序一致性，请创建自定义操作前脚本和操作后脚本，用于控制备份工作流和环境。 Azure 备份会在创建 VM 快照前调用操作前脚本，而在 VM 快照创建作业完成后调用操作后脚本。 有关更多详细信息，请参阅[使用操作前脚本和操作后脚本的应用程序一致性 VM 备份](/documentation/articles/backup-azure-linux-app-consistent/)。
> [AZURE.NOTE]
> Azure 备份只调用客户编写的操作前脚本和操作后脚本。 如果操作前脚本和操作后脚本成功执行，Azure 备份会将恢复点标记为应用程序一致。 但是，客户最终为使用自定义脚本时的应用程序一致性负责。
>


此表介绍了一致性类型，以及在 Azure VM 备份和还原过程中，这些一致性类型会在何种条件下出现。

| 一致性 | 基于 VSS | 解释和详细信息 |
| --- | --- | --- |
| 应用程序一致性 |是（对于 Windows）|应用程序一致性非常适合工作负荷，因为它可确保：<ol><li> VM *启动*。 <li>无数据损坏。 <li>无数据丢失。<li> 对于使用数据的应用程序，数据将保持一致，因为备份时会使用 VSS 或前/后脚本将应用程序纳入考虑。</ol> <li>*Windows VM* - 大多数 Microsoft 工作负荷都有 VSS 写入器，负责执行与数据一致性相关的工作负荷特定操作。 例如，Microsoft SQL Server 的 VSS 编写器可确保正确写入事务日志文件和数据库。 对于 Azure Windows VM 备份，若要创建应用程序一致恢复点，备份扩展必须调用 VSS 工作流且需在创建 VM 快照前完成调用。 若要确保 Azure VM 快照准确性，则也必须完成所有 Azure VM 应用程序的 VSS 编写器。 （了解 [VSS 基本信息](http://blogs.technet.com/b/josebda/archive/2007/10/10/the-basics-of-the-volume-shadow-copy-service-vss.aspx)，并深入了解其[工作原理](https://technet.microsoft.com/zh-cn/library/cc785914%28v=ws.10%29.aspx)详细信息）。 </li> <li> *Linux VM* - 客户可以执行[自定义操作前脚本和操作后脚本，以确保应用程序一致性](/documentation/articles/backup-azure-linux-app-consistent/)。 </li> |
| 文件系统一致性 |是 - 对于基于 Windows 的计算机 |在两种情况下，恢复点可做到文件系统一致：<ul><li>在没有前脚本/后脚本或前脚本/后脚本失败时，Azure 中 Linux VM 的备份。 <li>在 Azure 中备份 Windows VM 时出现 VSS 故障。</li></ul> 在这两种情况下，最佳做法是确保： <ol><li> VM *启动*。 <li>无数据损坏。<li>无数据丢失。</ol> 应用程序需要对还原的数据实施自己的“修复”机制。 |
| 崩溃一致性 |否 |这种情况相当于虚拟机“崩溃”（通过软重置或硬重置）。 崩溃一致性通常出现在 Azure 虚拟机在备份期间关闭时。 无论是从操作系统还是应用程序角度而言，崩溃一致性恢复点皆无法保证存储媒体上数据的一致性。 仅会捕获和备份备份时磁盘上已存在的数据。 <br/> <br/> 尽管并无保证，但通常情况下，会启动操作系统，并在之后进行 chkdsk 等磁盘检查过程来修复任何损坏错误。 任何未传输到磁盘的内存中数据或写入都将丢失。 如果需要执行数据回滚，应用程序通常会接着执行其自身的验证机制。 <br><br>例如，如果事务日志中的条目不在数据库中，则数据库软件将执行回滚，直到数据一致。 当数据分散在多个虚拟磁盘上时（例如跨区卷），崩溃一致恢复点不保证数据的正确性。 |

## <a name="performance-and-resource-utilization"></a>性能和资源利用率
与本地部署的备份软件一样，在 Azure 中备份 VM 时，应在容量和资源利用率方面进行需求规划。 [Azure 存储限制](/documentation/articles/azure-subscription-service-limits/#storage-limits/)定义如何构建 VM 部署，以获得最大性能并对运行中工作负荷造成最小影响。

规划备份性能时，请注意以下 Azure 存储限制：

- 每个存储帐户的最大传出
- 每个存储帐户的总请求速率

### <a name="storage-account-limits"></a>存储帐户限制
对于从存储帐户复制的备份数据，会添加到该存储帐户的每秒输入/输出操作数 (IOPS) 和出口（或吞吐量）度量值。 同时，虚拟机也会消耗 IOPS 与吞吐量。 目的是确保备份和虚拟机流量不会超过存储帐户限制。

### <a name="number-of-disks"></a>磁盘数目
备份过程会尝试尽可能快地完成备份作业。 在此情况下，它会占用尽可能多的资源。 但是， *单个 Blob 的目标吞吐量*实施 I/O 操作总次数限制为每秒 60 MB。 为了最大限度加快速度，备份过程会尝试并行备份每个 VM 磁盘。 如果 VM 有 4 个磁盘，则服务会尝试并行备份所有 4 个磁盘。 决定存储帐户备份流量的最重要因素是备份的**磁盘数**。

### <a name="backup-schedule"></a>备份计划
影响性能的另一因素是 **备份计划**。 如果配置策略以同时备份所有 VM，则会导致流量拥堵。 备份过程会尝试并行备份所有磁盘。 若要减少存储帐户的备份流量，请在一天的不同时间备份不同 VM，并且时间不重叠。

## <a name="capacity-planning"></a>容量计划
综合上述因素来看，用户需规划存储帐户使用需求。 请下载 [VM 备份容量规划 Excel 电子表格](https://gallery.technet.microsoft.com/Azure-Backup-Storage-a46d7e33)，以了解磁盘和备份计划选择所造成的影响。

### <a name="backup-throughput"></a>备份吞吐量
对于每个要备份的磁盘，Azure 备份将读取磁盘上的块，并只存储更改的数据（增量备份）。 下表显示了备份服务吞吐量平均值。 使用以下数据可以估算备份给定大小的磁盘所需的时间。

| 备份操作 | 最优吞吐量 |
| --- | --- |
| 初始备份 |160 Mbps |
| 增量备份 (DR) |640 Mbps <br><br> 如果更改数据（需要进行备份的数据）分散在磁盘，则吞吐量会显著下降。|

## <a name="total-vm-backup-time"></a>VM 备份总时间
尽管大部分备份时间花在读取和复制数据上，但其他一些操作也会影响到备份 VM 所需的总时间：

- 花费在[安装或更新备份扩展](/documentation/articles/backup-azure-vms/)上的时间。
- 快照时间，即触发某个快照所花费的时间。 接近计划的备份时间时触发快照。
- 队列等待时间。 由于备份服务要处理来自多个客户的备份，可能不会立即将备份数据从快照复制到备份或恢复服务保管库。 在负载高峰期，由于要处理的备份数过多，等待时间可能会长达 8 小时。 但是，每日备份策略规定的 VM 备份总时间不会超过 24 小时。
- 数据传输时间，备份服务计算上一备份中的增量更改并将更改传输到保管库存储所需的时间。

### <a name="why-am-i-observing-longer12-hours-backup-time"></a>为什么会看到较长的备份时间（超过 12 个小时）？
备份包含两个阶段：获取快照和将快照传输到保管库。 备份服务针对存储进行相关优化。 将快照数据传输到保管库时，服务仅传输上一个快照的增量更改。  为确定增量更改，服务会计算块的校验和。 如果一个块发生更改，则该块会被标识为要发送到保管库的块。 然后服务进一步钻取到每个已标识块，寻找机会尽量减少要传输的数据。 评估所有已更改块后，服务将联合更改并将其发送到保管库。 在一些旧版应用程序中，存储不适合小的分段写入。 如果快照包含很多小的分段写入，则服务会花费额外时间处理应用程序写入的数据。 对于在 VM 内部运行的应用程序，Azure 中建议的应用程序写入块的最小值是 8 KB。 如果应用程序使用大小小于 8 KB 的块，则备份性能会受影响。 有关调整应用程序以提高备份性能的帮助信息，请参阅[调整应用程序以实现 Azure 存储的最佳性能](/documentation/articles/storage-premium-storage-performance/)。 尽管有关备份性能的本文使用了高级存储示例，但是本指南同样适用于标准存储磁盘。

## <a name="total-restore-time"></a>总还原时间
还原操作包括两个主要的子任务：将数据从保管库复制回所选的客户存储帐户和创建虚拟机。 从保管库复制回数据取决于 Azure 中内部存储备份的位置以及存储客户存储帐户的位置。 复制数据所花的时间取决于：
- 队列等待时间 - 由于服务同时处理来自多个客户的还原作业，因此还原请求会排入队列中。
- 数据复制时间 - 将数据从保管库复制到客户存储帐户。 还原时间取决于 Azure 备份服务在所选客户存储帐户上获取的 IOPS 和吞吐量。 若要减少还原过程期间的复制时间，请选择一个未加载其他应用程序写入和读取的存储帐户。

## <a name="best-practices"></a>最佳实践
我们建议在为虚拟机配置备份时遵循以下做法：

- 请勿计划同时备份同一云服务中的 10 个以上经典 VM。 如果要备份同一云服务中的多个 VM，建议将备份开始时间错开一小时。
- 请勿计划同时备份 40 个以上 VM。
- 将 VM 备份安排在非高峰时间进行。 这样备份服务会使用 IOPS 将数据从客户存储帐户传输到保管库。
- 确保策略在分布于不同存储帐户的 VM 上应用。 建议不要使用同一备份计划保护单个存储帐户中总数超过 20 个的磁盘。 如果存储帐户中的磁盘超过 20 个，可将这些 VM 分散到多个策略中，以便在备份过程的传输阶段能够获得所需的 IOPS。
- 不要将运行在高级存储上的 VM 还原到同一存储帐户。 如果还原操作过程与备份操作一致，则会减少备份的可用 IOPS。
- 建议将每个高级 VM 运行在不同的高级存储帐户上，确保优化备份性能。

## <a name="data-encryption"></a>数据加密
在备份过程中，Azure 备份不会加密数据。 但是，可以在 VM 中加密数据，然后无缝备份保护的数据（阅读有关[加密数据备份](/documentation/articles/backup-azure-vms-encryption/)的详细信息）。

## <a name="calculating-the-cost-of-protected-instances"></a>计算受保护实例的成本
通过 Azure 备份进行备份的 Azure 虚拟机的收费依据 [Azure 备份定价](/pricing/details/backup/)。 受保护的实例计算基于虚拟机的实际大小，即虚拟机中除“资源磁盘”外的所有数据之和。

VM 备份定价*并非*基于附加到虚拟机的每个数据磁盘的最大支持大小。 定价基于数据磁盘中存储的实际数据。 与此类似，备份存储的收费是基于 Azure 备份中存储的数据量，即每个恢复点中实际数据之和。

以 A2 标准大小的虚拟机为例，该虚拟机有两个额外的数据磁盘，总大小为 1TB。 下表提供了其中每个磁盘上存储的实际数据：

| 磁盘类型 | 最大大小 | 实际存在的数据 |
| --- | --- | --- |
| 操作系统磁盘 |1023 GB |17 GB |
| 本地磁盘/资源磁盘 |135 GB |5 GB（不包括在备份中） |
| 数据磁盘 1 |1023 GB |30 GB |
| 数据磁盘 2 |1023 GB |0 GB |

此示例中，虚拟机的实际大小为 17 GB + 30 GB + 0 GB = 47 GB。 此受保护实例大小 (47 GB) 成为按月计费的基础。 随着虚拟机中数据量的增长，用于计费的受保护实例大小也会相应变化。

第一个备份成功完成后才会开始计费。 存储和受保护的实例也会在此同时开始计费。 只要针对虚拟机的*任何备份数据存储在保管库中*就会持续计费。 如果在虚拟机上停止保护，但在保管库中仍然存在虚拟机备份数据，仍将继续计费。

针对特定虚拟机的计费仅在停止保护*并且*删除全部备份数据后才会停止。 当停止保护并且没有活动的备份作业时，最后一个成功的 VM 备份的大小将成为用于每月帐单的受保护实例大小。

## <a name="questions"></a>有疑问？
如果你有疑问，或者希望包含某种功能，请 [给我们反馈](http://aka.ms/azurebackup_feedback)。

## <a name="next-steps"></a>后续步骤
- [备份虚拟机](/documentation/articles/backup-azure-vms/)
- [管理虚拟机备份](/documentation/articles/backup-azure-manage-vms-classic/)
- [恢复虚拟机](/documentation/articles/backup-azure-restore-vms/)
- [解决 VM 备份问题](/documentation/articles/backup-azure-vms-troubleshoot/)

<!---Update_Description: wording update -->