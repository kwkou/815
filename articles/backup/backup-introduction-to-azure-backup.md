<properties
    pageTitle="什么是 Azure 备份？ | Microsoft 文档"
    description="使用 Azure 备份，可以从 Windows Server、Windows 工作站、System Center DPM 服务器以及 Azure 虚拟机备份和还原数据与工作负荷。"
    services="backup"
    documentationcenter=""
    author="markgalioto"
    manager="carmonm"
    editor=""
    keywords="备份和还原;恢复服务;备份解决方案" />
<tags
    ms.assetid="0d2a7f08-8ade-443a-93af-440cbf7c36c4"
    ms.service="backup"
    ms.workload="storage-backup-recovery"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="get-started-article"
    ms.date="3/13/2017"
    ms.author="markgal;trinadhk; anuragm"
    ms.custom="H1Hack27Feb2017"
    wacn.date="05/22/2017"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="3ff18e6f95d8bbc27348658bc5fce50c3320cf0a"
    ms.openlocfilehash="70982c3cdecbe8694b4fa9995fb6b80509808b27"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/15/2017" />

# <a name="overview-of-the-features-in-azure-backup"></a>Azure 备份功能概述
Azure 备份是基于 Azure 的服务，可用于备份（或保护）和还原 Microsoft 云端数据。 Azure 备份取代了现有的本地或异地备份解决方案，并且是可靠、安全、高性价比的基于云的解决方案。 Azure 备份提供多个组件，可将其下载并部署到适当的计算机、服务器或云中。 可根据要保护的内容选择部署的组件或代理。 无论是保护本地数据还是云端数据，所有 Azure 备份组件均可用于将数据备份到 Azure 的备份保管库中。 请参阅本文稍后部分的 [Azure 备份组件表格](/documentation/articles/backup-introduction-to-azure-backup/#which-azure-backup-components-should-i-use/)，了解保护特定数据、应用程序或工作负荷所用的组件。

## <a name="why-use-azure-backup"></a>为何使用 Azure 备份？
传统的备份解决方案已演变成将云视为类似于磁盘/磁带的终结点或静态存储目标。 该方法很简单，但用途有限，不能充分利用基础云平台，由此变成了一种效率低的昂贵解决方案。 其他解决方案也很昂贵，因为你最终会为错误的存储类型或不需要的存储投资。 其他解决方案的效率通常不高，因为它们不会提供所需的存储类型/存储量，或者管理任务需要耗费太多时间。 与此相反，Azure 备份具有以下主要优势：

**自动存储管理** - 混合环境常常需要异类存储（部分在本地，部分在云）。 通过 Azure 备份，使用本地存储设备时无需付费。 Azure 备份会自动分配和管理备份存储，且采用即用即付模型。 即用即付是指只需为所用的存储付费。 有关详细详细，请参阅 [有关 Azure 定价的文章](/pricing/details/backup/)。

**无限缩放** - Azure 备份利用 Azure 云的基础功能和无限缩放功能实现高可用性 - 无需维护或监视开销。 可设置警报来获取相关事件信息，但无需担忧云数据的高可用性。

**多个存储选项** - 高可用性的一个方面是存储复制。 Azure 备份提供两种类型的复制：[本地冗余存储](/documentation/articles/storage-redundancy/#locally-redundant-storage/)和[异地冗余存储](/documentation/articles/storage-redundancy/#geo-redundant-storage/)。 根据需要选择备份存储选项：

- 本地冗余存储 (LRS) 将同一区域的配对数据中心内的数据复制三次（创建三个数据副本）。 LRS 是一个低成本选项，可在本地硬件故障时保护数据。

- 异地冗余存储 (GRS) 将数据复制到源数据主位置数英里之外的次要区域中。 GRS 的成本比 LRS 的高，但 GRS 可让数据更为持久，即使出现区域性中断也是如此。

**无限制的数据传输** - Azure 备份不会限制传输的入站或出站数据量。 Azure 备份也不会对传输的数据收费。 但如果使用 Azure 导入/导出服务来导入大量数据，则入站数据将产生相关费用。 有关此费用的详细信息，请参阅 [Azure 备份中的脱机备份工作流](/documentation/articles/backup-azure-backup-import-export/)。 出站数据是指还原操作期间从备份保管库传输的数据。

**数据加密** - 该服务允许在公有云中安全地传输和存储数据。 加密通行短语存储在本地，绝不会传输或存储到 Azure 中。 如有必要还原任何数据，只需具有加密通行短语或密码即可。

**应用程序一致的备份** - 无论是备份文件服务器、虚拟机还是 SQL 数据库，都需要知道恢复点具有还原备份副本所需的全部数据。 Azure 备份提供了应用程序一致的备份，确保了还原数据时无需额外的修补程序。 还原应用程序一致的数据可减少还原时间，使得可快速恢复到运行状态。

**长期保留** - 可使用 Azure 实现短期和长期保留，无需将备份副本从磁盘转到磁带中，也无需将磁带移到异地位置。 Azure 不会限制备份或恢复服务保管库中数据的保留时间长度。 可以根据需要设置数据在保管库中的保留时间。 Azure 备份的限制为每个受保护实例仅限 9999 个恢复点。 请参阅本文的[备份和保留](/documentation/articles/backup-introduction-to-azure-backup/#backup-and-retention/)部分，了解此限制对用户备份需求的影响。  

## <a name="which-azure-backup-components-should-i-use"></a>应使用哪些 Azure 备份组件？
如果不确定哪个 Azure 备份组件适合你的需求，请参阅下表了解每个组件可保护的内容。

| 组件 | 优点 | 限制 | 哪些内容受到保护？ | 备份存储在何处？ |
| --- | --- | --- | --- | --- |
| Azure 备份 (MARS) 代理 |<li>将文本和文件夹备份到物理或虚拟 Windows OS（VM 可以在本地或在 Azure 中）<li>无需单独的备份服务器。 |<li>每天备份三次 <li>不感知应用程序；仅支持文件、文件夹和卷级别的还原， <li>  不支持 Linux。 |<li>文件、 <li>文件夹 |Azure 备份保管库 |
| System Center DPM |<li>应用程序感知快照 (VSS)<li>在备份时间上完全灵活<li>恢复粒度（全部）<li>可使用 Azure 备份保管库<li>Hyper-V 和 VMware VM 对 Linux 的支持 <li>使用 DPM 2012 R2 备份和还原 VMware VM |无法备份 Oracle 工作负荷。|<li>文件、 <li>文件夹、<li> 卷、 <li>VM、<li> 应用程序、<li> 工作负荷 |<li>Azure 备份保管库、<li> 本地附加磁盘、<li>  磁带（仅限本地） |
| Azure 备份服务器 |<li>应用感知快照 (VSS)<li>在备份时间上完全灵活<li>恢复粒度（全部）<li>可使用 Azure 备份保管库<li>Hyper-V 和 VMware VM 对 Linux 的支持<li>备份和还原 VMware VM <li>不需要 System Center 许可证 |<li>无法备份 Oracle 工作负荷。<li>始终需要实时 Azure 订阅<li>不支持磁带备份 |<li>文件、 <li>文件夹、<li> 卷、 <li>VM、<li> 应用程序、<li> 工作负荷 |<li>Azure 备份保管库、<li> 本地附加磁盘 |
| Azure IaaS VM 备份 |<li>针对 Windows/Linux 的本地备份<li>无需安装特定代理<li>无需使用备份基础结构进行结构级备份 |<li>每天备份 VM 一次 <li>仅在磁盘级还原 VM<li>无法本地备份 |<li>VM、 <li>所有磁盘（使用 PowerShell） |<p>Azure 备份保管库</p> |

## <a name="what-are-the-deployment-scenarios-for-each-component"></a>每个组件适用于哪些部署方案？
| 组件 | 可以在 Azure 中部署吗？ | 可以在本地部署吗？ | 支持的目标存储 |
| --- | --- | --- | --- |
| Azure 备份 (MARS) 代理 |<p>**是**</p> <p>Azure 备份代理可在 Azure 中运行的任意 Windows Server VM 上进行部署。</p> |<p>**是**</p> <p>备份代理可在任意 Windows Server VM 或物理计算机上进行部署。</p> |<p>Azure 备份保管库</p> |
| System Center DPM |<p>**是**</p><p>深入了解[如何使用 System Center DPM 保护 Azure 中的工作负荷](/documentation/articles/backup-azure-dpm-introduction-classic/)。</p> |<p>**是**</p> <p>深入了解[如何保护数据中心内的工作负荷和 VM](https://technet.microsoft.com/en-us/system-center-docs/dpm/data-protection-manager)。</p> |<p>本地附加磁盘、</p> <p>Azure 备份保管库、</p> <p>磁带（仅限本地）</p> |
| Azure 备份服务器 |<p>**是**</p><p>深入了解[如何使用 Azure 备份服务器保护 Azure 中的工作负荷](/documentation/articles/backup-azure-microsoft-azure-backup-classic/)。</p> |<p>**是**</p> <p>深入了解[如何使用 Azure 备份服务器保护 Azure 中的工作负荷](/documentation/articles/backup-azure-microsoft-azure-backup-classic/)。</p> |<p>本地附加磁盘、</p> <p>Azure 备份保管库</p> |
| Azure IaaS VM 备份 |<p>**是**</p><p>Azure 结构的一部分</p><p>专用于[备份 Azure 基础结构即服务 (IaaS) 虚拟机](/documentation/articles/backup-azure-vms-introduction/)。</p> |<p>**否**</p> <p>使用 System Center DPM 备份数据中心内的虚拟机。</p> |<p>Azure 备份保管库</p> |

## <a name="which-applications-and-workloads-can-be-backed-up"></a>可以备份哪些应用程序和工作负荷？
下表提供了可使用 Azure 备份保护的数据和工作负荷的矩阵。 Azure 备份解决方案列具有该解决方案部署文档的链接。 每个 Azure 备份组件均可在经典（Service Manager 部署）或资源管理器部署模型环境中进行部署。

[AZURE.INCLUDE [learn-about-deployment-models](../../includes/learn-about-deployment-models-include.md)]

| 数据或工作负荷 | 源环境 | Azure 备份解决方案 |
| --- | --- | --- |
| 文件和文件夹 |Windows Server |<p>[Azure 备份代理](/documentation/articles/backup-configure-vault/)、</p> <p>[System Center DPM](/documentation/articles/backup-azure-dpm-introduction-classic/)（带 Azure 备份代理）、</p> <p>[Azure 备份服务器](/documentation/articles/backup-azure-microsoft-azure-backup-classic/)（带 Azure 备份代理）</p> |
| 文件和文件夹 |Windows 计算机 |<p>[Azure 备份代理](/documentation/articles/backup-configure-vault/)、</p> <p>[System Center DPM](/documentation/articles/backup-azure-dpm-introduction-classic/)（带 Azure 备份代理）、</p> <p>[Azure 备份服务器](/documentation/articles/backup-azure-microsoft-azure-backup-classic/)（带 Azure 备份代理）</p> |
| Hyper-V 虚拟机 (Windows) |Windows Server |<p>[System Center DPM](/documentation/articles/backup-azure-backup-sql/)（带 Azure 备份代理）、</p> <p>[Azure 备份服务器](/documentation/articles/backup-azure-microsoft-azure-backup-classic/)（带 Azure 备份代理）</p> |
| Hyper-V 虚拟机 (Linux) |Windows Server |<p>[System Center DPM](/documentation/articles/backup-azure-backup-sql/)（带 Azure 备份代理）、</p> <p>[Azure 备份服务器](/documentation/articles/backup-azure-microsoft-azure-backup-classic/)（带 Azure 备份代理）</p> |
| Microsoft SQL Server |Windows Server |<p>[System Center DPM](/documentation/articles/backup-azure-backup-sql/)（带 Azure 备份代理）、</p> <p>[Azure 备份服务器](/documentation/articles/backup-azure-microsoft-azure-backup-classic/)（带 Azure 备份代理）</p> |
| Microsoft SharePoint |Windows Server |<p>[System Center DPM](/documentation/articles/backup-azure-backup-sql/)（带 Azure 备份代理）、</p> <p>[Azure 备份服务器](/documentation/articles/backup-azure-microsoft-azure-backup-classic/)（带 Azure 备份代理）</p> |
| Microsoft Exchange |Windows Server |<p>[System Center DPM](/documentation/articles/backup-azure-backup-sql/)（带 Azure 备份代理）、</p> <p>[Azure 备份服务器](/documentation/articles/backup-azure-microsoft-azure-backup-classic/)（带 Azure 备份代理）</p> |
| Azure IaaS VM (Windows) |在 Azure 中运行 |[Azure 备份（VM 扩展）](/documentation/articles/backup-azure-vms-introduction/) |
| Azure IaaS VM (Linux) |在 Azure 中运行 |[Azure 备份（VM 扩展）](/documentation/articles/backup-azure-vms-introduction/) |

## <a name="linux-support"></a>Linux 支持
下表显示了支持 Linux 的 Azure 备份组件。  

| 组件 | Linux（Azure 认可）支持 |
| --- | --- |
| Azure 备份 (MARS) 代理 |否（仅限基于 Windows 的代理） |
| System Center DPM |在 Hyper-V 和 VMWare 上对 Linux 来宾 VM 进行文件一致性备份<br/> （不适用于 Azure VM）<br/> 对 Hyper-V 和 VMWare Linux 来宾 VM 进行 VM 还原 |
| Azure 备份服务器 |在 Hyper-V 和 VMWare 上对 Linux 来宾 VM 进行文件一致性备份<br/> （不适用于 Azure VM）<br/> 对 Hyper-V 和 VMWare Linux 来宾 VM 进行 VM 还原 |
| Azure IaaS VM 备份 |应用程序一致性备份，使用[前脚本和后脚本框架](/documentation/articles/backup-azure-linux-app-consistent/)<br/> [还原所有 VM 磁盘](/documentation/articles/backup-azure-restore-vms#restore-backed-up-disks/)<br/> [VM 还原](/documentation/articles/backup-azure-restore-vms#create-a-new-vm-from-restore-point/) |

## <a name="using-premium-storage-vms-with-azure-backup"></a>结合使用高级存储 VM 和 Azure 备份
Azure 备份可保护高级存储 VM。 Azure 高级存储是基于固态硬盘 (SSD) 的存储，用于支持 I/O 密集型工作负荷。 高级存储很适合虚拟机 (VM) 工作负荷。 有关高级存储的详细信息，请参阅[高级存储：Azure 虚拟机工作负荷的高性能存储](/documentation/articles/storage-premium-storage/)一文。

### <a name="back-up-premium-storage-vms"></a>备份高级存储 VM
在备份高级存储 VM 时，备份服务在高级存储帐户中创建名为“AzureBackup-”的临时暂存位置。 暂存位置与恢复点快照大小相同。 请确保存储帐户中存在适用于临时暂存位置的空间。 有关详细信息，请参阅[高级存储限制](/documentation/articles/storage-premium-storage/#scalability-and-performance-targets/)一文。 备份作业完成后，将删除暂存位置。 用于暂存位置的存储的价格与所有 [高级存储定价](/documentation/articles/storage-premium-storage/#pricing-and-billing/)一致。

> [AZURE.NOTE]
> 请不要修改或编辑暂存位置。
>
>

### <a name="restore-premium-storage-vms"></a>还原高级存储 VM
可以将高级存储 VM 还原为高级存储或常规存储。 将高级存储 VM 的恢复点还原到高级存储是典型的还原过程。 但是，将高级存储 VM 的恢复点还原到标准存储更符合成本效益。 如果需要 VM 中的一部分文件，可以使用这种还原类型。

## <a name="what-are-the-features-of-each-backup-component"></a>每个备份组件有哪些功能？
以下部分提供了相关表格，总结了每个 Azure 备份组件中各种功能是否可用或受支持。 请参阅各表格后的额外支持信息或详细信息。

### <a name="storage"></a>存储
| 功能 | Azure 备份代理 | System Center DPM | Azure 备份服务器 | Azure IaaS VM 备份 |
| --- | --- | --- | --- | --- |
| Azure 备份保管库 |![是][green] |![是][green] |![是][green] |![是][green] |
| 磁盘存储 | |![是][green] |![是][green] | |
| 表存储 | |![是][green] | | |
| 压缩 <br/>（在备份保管库中） |![是][green] |![是][green] |![是][green] | |
| 增量备份 |![是][green] |![是][green] |![是][green] |![是][green] |
| 磁盘重复数据删除 | |![部分][yellow] |![部分][yellow] | |

![表键](./media/backup-introduction-to-azure-backup/table-key.png)

备份保管库是所有组件中首选的存储目标。 System Center DPM 和 Azure 备份服务器还提供生成本地磁盘副本的选项。 但是，System Center DPM 提供将数据写入磁带存储设备的选项。

#### <a name="compression"></a>压缩
备份经过压缩以减少所需的存储空间。 唯一不进行压缩的组件为 VM 扩展。 VM 扩展可将所有备份数据从存储帐户复制到同一区域中的备份保管库。 传输数据时不使用压缩。 传输数据但不压缩会稍微增加所用的存储空间。 但是，存储数据而不压缩可加快还原，实现特定的恢复点目标。


#### <a name="disk-deduplication"></a>磁盘重复数据删除
在 [Hyper-V 虚拟机](http://blogs.technet.com/b/dpm/archive/2015/01/06/deduplication-of-dpm-storage-reduce-dpm-storage-consumption.aspx)上部署 System Center DPM 或 Azure 备份服务器时，可使用重复数据删除。 Windows Server 会在以备份存储形式附加到虚拟机的虚拟硬盘 (VHD) 上执行主机级别的重复数据删除。

> [AZURE.NOTE]
> 重复数据删除不一定适用于 Azure 中的所有备份组件。 如果 System Center DPM 和备份服务器部署在 Azure 中，则附加到 VM 的存储磁盘无法进行重复数据删除。
>
>

### <a name="incremental-backup-explained"></a>增量备份说明
无论目标存储（磁盘、磁带、备份保管库）如何，每个 Azure 备份组件都支持增量备份。 增量备份仅传输自上次备份以来所做的更改，从而可以确保备份在存储空间和时间方面高效。

#### <a name="comparing-full-differential-and-incremental-backup"></a>比较完整备份、差异备份和增量备份

存储消耗、恢复时间目标 (RTO) 和网络消耗因每种备份方法而异。 为了降低备份总拥有成本 (TCO)，需要了解如何选择最佳备份解决方案。 下图对完整备份、差异备份和增量备份进行了比较。 在图中，数据源 A 由 10 个每月备份的存储块 A1-A10 组成。 第一个月，存储块 A2、A3、A4 和 A9 变化，第二个月，存储块 A5 变化。

![备份方法比较图](./media/backup-introduction-to-azure-backup/backup-method-comparison.png)

借助**完整备份**，每个备份副本包含整个数据源。 完整备份将占用大量的网络带宽和存储，每次传输一份备份副本。

**差异备份**仅存储自初始完整备份后发生变化的数据块，这将占用较少的网络、消耗较少的存储。 差异备份不保留无变化数据的冗余副本。 但是，由于会传输并存储后续备份之间保持不变的数据块，所以差异备份的效率比较低。 第二个月，对已更改的存储块 A2、A3、A4 和 A9 进行备份。 第三个月，会再次备份这些相同的存储块，以及已更改的存储块 A5。 下次进行完整备份之前，将继续对已更改的存储块进行备份。

**增量备份**通过仅存储上次备份后更改的数据块，从而实现高存储效率和高网络效率。 采用增量备份，没有必要进行定期的完整备份。 在示例中，第一个月进行完整备份后，已更改存储块 A2、A3、A4 和 A9 将标记为“已更改”，在第二个月进行传输。 在第三个月，仅标记已更改的存储块 A5，并进行传输。 移动较少的数据可以节省存储和网络资源，从而降低 TCO。   

### <a name="security"></a>“安全”
| 功能 | Azure 备份代理 | System Center DPM | Azure 备份服务器 | Azure IaaS VM 备份 |
| --- | --- | --- | --- | --- |
| 网络安全<br/> （到 Azure） |![是][green] |![是][green] |![是][green] |![部分][yellow] |
| 数据安全<br/> （在 Azure 中） |![是][green] |![是][green] |![是][green] |![部分][yellow] |

![表键](./media/backup-introduction-to-azure-backup/table-key.png)

#### <a name="network-security"></a>网络安全性
从服务器到备份保管库的所有备份流量均通过高级加密标准 256 进行加密。 备份数据通过安全的 HTTPS 链接发送。 备份数据还会以加密格式存储在备份保管库中。 只有 Azure 客户具有解锁此数据的通行短语。 Microsoft 无法解密任何位置的备份数据。

> [AZURE.WARNING]
> 建立备份保管库后，只有你才能访问加密密钥。 Microsoft 绝不会保留加密密钥副本，且没有访问该密钥的权限。 如果客户丢失了密钥，Microsoft 无法恢复备份数据。
>
>

#### <a name="data-security"></a>数据安全性
备份 Azure VM 时，需要在虚拟机 *内部* 设置加密。 在 Windows 虚拟机上使用 BitLocker，在 Linux 虚拟机上使用 **dm-crypt** 。 Azure 备份不会自动加密来自此路径的备份数据。

### <a name="network"></a>网络
| 功能 | Azure 备份代理 | System Center DPM | Azure 备份服务器 | Azure IaaS VM 备份 |
| --- | --- | --- | --- | --- |
| 网络压缩 <br/>（到**备份服务器**） | |![是][green] |![是][green] | |
| 网络压缩 <br/>（到**备份保管库**） |![是][green] |![是][green] |![是][green] | |
| 网络协议 <br/>（到**备份服务器**） | |TCP |TCP | |
| 网络协议 <br/>（到**备份保管库**） |HTTPS |HTTPS |HTTPS |HTTPS |

![表键](./media/backup-introduction-to-azure-backup/table-key-2.png)

IaaS VM 上的 VM 扩展会通过存储网络直接读取 Azure 存储帐户中的数据，因此无需优化此流量。

若要将数据备份到 System Center DPM 或 Azure 备份服务器，则需压缩从主服务器流到备份服务器的数据。 在备份到 DPM 或 Azure 备份服务器之前压缩数据可以节省带宽。

#### <a name="network-throttling"></a>网络限制
Azure 备份代理提供网络限制功能，可用于控制数据传输期间的网络带宽使用方式。 如果需要在上班时间内备份数据，但不希望备份程序干扰其他 Internet 流量，限制会很有帮助。 数据传输的限制适用于备份和还原活动。

## <a name="backup-and-retention"></a>备份和保留

Azure 备份针对每个受保护实例实施 9999 个恢复点（也称为备份副本或快照）的限制。 受保护的实例是计算机、服务器（物理或虚拟）或配置为向 Azure 备份数据的工作负荷。 有关详细信息，请参阅[什么是受保护实例](/documentation/articles/backup-introduction-to-azure-backup/#what-is-a-protected-instance/)部分。 保存数据的备份副本时，将保护实例。 数据的备份副本就是保障。 如果源数据丢失或损坏，备份副本可还原源数据。 下表显示每个组件的最大备份频率。 备份策略配置决定使用恢复点的速度。 例如，如果每天创建恢复点，那么恢复点可保留 27 年才到期。 如果每月创建恢复点，那么恢复点可保留 833 年才到期。 备份服务不会为恢复点设置到期时间限制。

|  | Azure 备份代理 | System Center DPM | Azure 备份服务器 | Azure IaaS VM 备份 |
| --- | --- | --- | --- | --- |
| 备份频率<br/> （到备份保管库） |每天三次备份 |每天备份两次 |每天备份两次 |每天一次备份 |
| 备份频率<br/> （到磁盘） |不适用 |<li>SQL Server 每隔 15 分钟 <li>其他工作负荷每隔 1 小时 |<li>SQL Server 每隔 15 分钟 <li>其他工作负荷每隔 1 小时</p> |不适用 |
| 保留期选项 |每日、每周、每月、每年 |每日、每周、每月、每年 |每日、每周、每月、每年 |每日、每周、每月、每年 |
| 每个受保护实例的最大恢复点数 |9999|9999|9999|9999|
| 最大保留期 |取决于备份频率 |取决于备份频率 |取决于备份频率 |取决于备份频率 |
| 本地磁盘上的恢复点 |不适用 |<li>对于文件服务器为 64，<li>对于应用程序服务器为 448 |<li>对于文件服务器为 64，<li>对于应用程序服务器为 448 |不适用 |
| 磁带上的恢复点 |不适用 |不受限制 |不适用 |不适用 |

## <a name="what-is-a-protected-instance"></a>受保护实例是什么
受保护实例通常指配置为备份到 Azure 的 Windows 计算机、物理或虚拟服务器，或者 SQL 数据库。 为计算机、服务器或数据库设置备份策略，并为数据创建副本后，实例即受到保护。 该受保护实例的备份数据后续副本（称为恢复点）将增加使用的存储量。 可为每个受保护实例创建最多 9999 个恢复点。 如果从存储中删除恢复点，则不计入 9999 个总恢复点数。
受保护实例的一些常见示例包括运行 Windows 操作系统的虚拟机、应用程序服务器、数据库和个人计算机。 例如：

- 运行 Hyper-V 或 Azure IaaS 虚拟机监控程序结构的虚拟机。 此虚拟机的来宾操作系统可以是 Windows Server 或 Linux。
- 应用程序服务器：它可为运行 Windows Server 和工作负荷（具有需备份的数据）的物理或虚拟机。 常见的工作负荷有 Microsoft SQL Server、Microsoft Exchange 服务器、Microsoft SharePoint 服务器和 Windows Server 上的文件服务器角色。 若要备份这些工作负荷，需要 System Center Data Protection Manager (DPM) 或 Azure 备份服务器。
- 运行 Windows 操作系统的个人计算机、工作站或笔记本电脑。


## <a name="what-is-the-vault-credential-file"></a>什么是保管库凭据文件？
保管库凭据文件是门户为每个备份保管库生成的证书。 然后，门户会将公钥上载到访问控制服务 (ACS)。 下载凭据时，会向你提供私钥。 可用它来注册要保护的计算机。 私钥可用于对要将备份数据发送到特定备份保管库的服务器或计算机进行身份验证。

只能使用保管库凭据来注册服务器或计算机。 但是，请保存好保管库凭据，如果它丢失或由他人获取，这些凭据可能会用于针对相同的保管库注册其他计算机。 由于备份数据使用通行短语进行了加密，因此只有你可访问，现有的备份数据不会泄露。 保管库凭据会在 48 小时后过期。 虽然可随时下载备份保管库的保管库凭据，但只有最新的凭据可用于注册。

## <a name="how-does-azure-backup-differ-from-azure-site-recovery"></a>Azure 备份与 Azure Site Recovery 有何不同？
Azure 备份和 Azure Site Recovery 均备份数据，均可还原数据。 但两者的价值主张不同。

Azure 备份保护本地和云端的数据。 Azure Site Recovery 就虚拟机和物理服务器的复制、故障转移和故障恢复进行协调。 这两个服务都很重要，因为灾难恢复解决方案需要让数据保持安全且可恢复（备份）， *同时* 在服务中断时使工作负荷保持可用 (Site Recovery)。

以下概念可帮助你做出有关备份和灾难恢复的重要决策。

| 概念 | 详细信息 | 备份 | 灾难恢复 (DR) |
| --- | --- | --- | --- |
| 恢复点目标 (RPO) |在需要执行恢复的情况下可接受的数据丢失量。 |备份解决方案的可接受 RPO 存在很大差异。 虚拟机备份的 RPO 通常为一天，而数据库备份的 RPO 只有 15 分钟。 |灾难恢复解决方案的 RPO 较低。 DR 复制可以落后几秒钟或几分钟时间。 |
| 恢复时间目标 (RTO) |完成恢复或还原所需的时间量。 |由于 RPO 较大，备份解决方案需要处理的数据量通常更多，这会导致 RTO 较长。 例如，根据从异地转送磁带所需的时间，从磁带还原数据可能需要数天的时间。 |由于灾难恢复解决方案与源之间的同步程度更高，因此其 RTO 更小， 需要处理的更改也更少。 |
| 保留 |数据需要存储多久 |对于需要进行操作恢复的情况（数据损坏、文件意外删除、OS 故障），备份数据通常会保留 30 天或更短时间。<br>从合规性角度来看，数据可能需要存储数月甚至数年。 在这种情况下，备份数据非常适合存档。 |灾难恢复只需操作性恢复数据，通常只需几个小时或最多一天的数据。 由于 DR 解决方案中使用了精细数据捕获，因此不建议长期保留 DR 数据。 |

## <a name="next-steps"></a>后续步骤
请参阅下述某个教程，详细了解在 Windows Server 上保护数据或在 Azure 中保护虚拟机 (VM) 的分步说明。

- [备份文件和文件夹](/documentation/articles/backup-try-azure-backup-in-10-mins/)
- [备份 Azure 虚拟机](/documentation/articles/backup-azure-vms-first-look/)

有关保护其他工作负荷的详细信息，请尝试阅读以下文章之一：

- [备份 Windows Server](/documentation/articles/backup-configure-vault/)
- [备份应用程序工作负荷](/documentation/articles/backup-azure-microsoft-azure-backup-classic/)
- [Backup Azure IaaS VMs（备份 Azure IaaS VM）](/documentation/articles/backup-azure-vms-prepare/)

[green]: ./media/backup-introduction-to-azure-backup/green.png
[yellow]: ./media/backup-introduction-to-azure-backup/yellow.png
[red]: ./media/backup-introduction-to-azure-backup/red.png

<!---Update_Description: wording update -->