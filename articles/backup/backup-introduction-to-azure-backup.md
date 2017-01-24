<properties
    pageTitle="什么是 Azure 备份？| Azure"
    description="使用 Azure 备份和恢复服务，可从 Windows Server、Windows 客户端计算机、System Center DPM 服务器和 Azure 虚拟机备份和还原数据与应用程序。"
    services="backup"
    documentationcenter=""
    author="markgalioto"
    manager="cfreeman"
    editor="keywords: backup and restore; recovery services; backup solutions" />
<tags
    ms.assetid="0d2a7f08-8ade-443a-93af-440cbf7c36c4"
    ms.service="backup"
    ms.workload="storage-backup-recovery"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="get-started-article"
    ms.date="12/7/2016"
    wacn.date="01/24/2017"
    ms.author="jimpark; trinadhk" />

# 什么是 Azure 备份？
Azure 备份是基于 Azure 的服务，可用于备份（或保护）和还原 Microsoft 云中的数据。Azure 备份取代了现有的本地或异地备份解决方案，并且是可靠、安全、高性价比的基于云的解决方案。Azure 备份提供多个组件，可将其下载并部署到适当的计算机、服务器或云中。可根据要保护的内容选择部署的组件或代理。无论是保护本地数据还是云数据，所有 Azure 备份组件均可用于将数据备份到 Azure 的备份保管库中。请参阅本文稍后提供的 [Azure 备份组件表格](/documentation/articles/backup-introduction-to-azure-backup/#which-azure-backup-components-should-i-use/)，了解保护特定数据、应用程序或工作负荷所用的组件。

## 为何使用 Azure 备份？
传统的备份解决方案已演变成将云视为类似于磁盘/磁带的终结点或静态存储目标。该方法很简单，但用途有限，不能充分利用基础云平台，由此变成了一种效率低的昂贵解决方案。其他解决方案也很昂贵，因为你最终会为错误的存储类型或不需要的存储投资。其他解决方案的效率通常不高，因为它们不会提供所需的存储类型/存储量，或者管理任务需要耗费太多时间。与此相反，Azure 备份具有以下主要优势：

**自动存储管理** - 混合环境常常需要异类存储（部分在本地，部分在云）。通过 Azure 备份，使用本地存储设备时无需付费。Azure 备份会自动分配和管理备份存储，且采用即用即付模型。即用即付是指只需为所用的存储付费。有关详细详细，请参阅[有关 Azure 定价的文章](/pricing/details/backup/)。

**无限缩放** - Azure 备份利用 Azure 云的基础功能和无限缩放功能实现高可用性 - 无需维护或监视开销。可设置警报来获取相关事件信息，但无需担忧云数据的高可用性。

**多种存储选项** - 高可用性的一个方面是存储复制。Azure 备份提供 2 种类型的复制：[本地冗余存储](/documentation/articles/storage-redundancy/#locally-redundant-storage/)和[异地冗余存储](/documentation/articles/storage-redundancy/#geo-redundant-storage/)。根据需要选择备份存储选项：

- 本地冗余存储 (LRS) 将同一区域的配对数据中心内的数据复制三次（创建三个数据副本）。LRS 是一个低成本选项，可在本地硬件故障时保护数据。

- 异地冗余存储 (GRS) 将数据复制到源数据主位置数英里之外的次要区域中。GRS 的成本比 LRS 的高，但 GRS 可让数据更为持久，即使出现区域性中断也是如此。

**无限制的数据传输** - Azure 备份不会限制传输的入站或出站数据量。Azure 备份也不会对传输的数据收费。但如果使用 Azure 导入/导出服务来导入大量数据，则入站数据将产生相关费用。有关此成本的详细信息，请参阅 [Azure 备份中的脱机备份工作流](/documentation/articles/backup-azure-backup-import-export/)。出站数据是指还原操作期间从备份保管库传输的数据。

**数据加密** - 该服务允许在公有云中安全地传输和存储数据。加密通行短语存储在本地，绝不会传输或存储到 Azure 中。如有必要还原任何数据，只需具有加密通行短语或密码即可。

**应用程序一致的备份** - 无论是备份文件服务器、虚拟机还是 SQL 数据库，都需要知道恢复点具有还原备份副本所需的全部数据。Azure 备份提供了应用程序一致的备份，确保了还原数据时无需额外的修补程序。还原应用程序一致的数据可减少还原时间，使得可快速恢复到运行状态。

**长期保留** - 可以在 Azure 中备份数据 99 年。可使用 Azure 实现短期和长期保留，无需将备份副本从磁盘转到磁带中，再将磁带移到异地位置进行长期存储。

## 应使用哪些 Azure 备份组件？ <a name="which-azure-backup-components-should-i-use"></a>
如果不确定哪个 Azure 备份组件适合你的需求，请参阅下表了解每个组件可保护的内容。

| 组件 | 优点 | 限制 | 哪些内容受到保护？ | 备份存储在何处？ |
| --- | --- | --- | --- | --- |
| Azure 备份 (MARS) 代理 |<li>将文本和文件夹备份到物理或虚拟 Windows OS（VM 可以在本地或在 Azure 中）<li>无需单独的备份服务器。 |<li>每天备份三次<li>不感知应用程序；仅支持文件、文件夹和卷级别的还原，<li>不支持 Linux。 |<li>文件、<li>文件夹 |Azure 备份保管库 |
| System Center DPM |<li>应用感知的快照 (VSS)<li>创建备份时提供充分的弹性<li>恢复粒度（全部）<li>可使用 Azure 备份保管库<li>Hyper-V 和 VMware VM 上的 Linux 支持<li>使用 DPM 2012 R2 保护 VMware VM |无法备份 Oracle 工作负荷。|<li>文件、<li>文件夹、<li>卷、<li>VM、<li>应用程序、<li>工作负荷 |<li>Azure 备份保管库、<li>本地附加的磁盘、<li>磁带（仅限本地） |
| Azure 备份服务器 |<li>应用感知的快照 (VSS)<li>创建备份时提供充分的弹性<li>恢复粒度（全部）<li>可使用 Azure 备份保管库<li>Linux 支持（如果托管在 Hyper-V 上）<li>使用 DPM 2012 R2 保护 VMware VM<li>无需 System Center 许可证 |<li>无法备份 Oracle 工作负荷。<li>始终需要实时 Azure 订阅<li>不支持磁带备份 |<li>文件、<li>文件夹、<li>卷、<li>VM、<li>应用程序、<li>工作负荷 |<li>Azure 备份保管库、<li>本地附加的磁盘 |
| Azure IaaS VM 备份 |<li>针对 Windows/Linux 的本机备份<li>不需要安装特定代理<li>无需使用备份基础结构进行结构级备份 |<li>每天备份 VM 一次<li>仅在磁盘级还原 VM<li>无法本地备份 |<li>VM、<li>所有磁盘（使用 PowerShell） |<p>Azure 备份保管库</p> |

## 每个组件适用于哪些部署方案？
| 组件 | 可以在 Azure 中部署吗？ | 可以在本地部署吗？ | 支持的目标存储 |
| --- | --- | --- | --- |
| Azure 备份 (MARS) 代理 |<p>**是**</p> <p>Azure 备份代理可以部署在 Azure 中运行的任何 Windows Server VM 上。</p> |<p>**是**</p> <p>该备份代理可以部署在任何 Windows Server VM 或物理计算机上。</p> |<p>Azure 备份保管库</p> |
| System Center DPM |<p>**是**</p><p>详细了解[如何使用 System Center DPM 保护 Azure 中的工作负荷](/documentation/articles/backup-azure-dpm-introduction-classic/)。</p> |<p>**是**</p> <p>详细了解[如何保护数据中心的工作负荷和 VM](https://technet.microsoft.com/zh-cn/system-center-docs/dpm/data-protection-manager)。</p> |<p>本地附加的磁盘、</p><p>Azure 备份保管库、</p><p>磁带（仅限本地）</p> |
| Azure 备份服务器 |<p>**是**</p><p>详细了解如何使用 Azure 备份服务器保护 Azure 中的工作负荷。</p> |<p>**是**</p><p>详细了解如何使用 Azure 备份服务器保护 Azure 中的工作负荷。</p> |<p>本地附加的磁盘、</p><p>Azure 备份保管库</p> |
| Azure IaaS VM 备份 |<p>**是**</p><p>Azure 结构的一部分</p><p>专门用于[备份 Azure 基础结构即服务 (IaaS) 虚拟机](/documentation/articles/backup-azure-vms-introduction/)。</p> |<p>**否**</p> <p>使用 System Center DPM 备份数据中心的虚拟机。</p> |<p>Azure 备份保管库</p> |

## 可以备份哪些应用程序和工作负荷？
下表提供了可使用 Azure 备份保护的数据和工作负荷的矩阵。Azure 备份解决方案列具有该解决方案部署文档的链接。每个 Azure 备份组件均可在经典（Service Manager 部署）或资源管理器部署模型环境中部署。

[AZURE.INCLUDE [了解部署模型](../../includes/learn-about-deployment-models-include.md)]

| 数据或工作负荷 | 源环境 | Azure 备份解决方案 |
| --- | --- | --- |
| 文件和文件夹 |Windows Server |<p>[Azure 备份代理](/documentation/articles/backup-configure-vault/)、</p><p>[System Center DPM](/documentation/articles/backup-azure-dpm-introduction-classic/)（+ Azure 备份代理）、</p><p>Azure 备份服务器（包括 Azure 备份代理）</p> |
| 文件和文件夹 |Windows 计算机 |<p>[Azure 备份代理](/documentation/articles/backup-configure-vault/)、</p><p>[System Center DPM](/documentation/articles/backup-azure-dpm-introduction-classic/)（+ Azure 备份代理）、</p><p>Azure 备份服务器（包括 Azure 备份代理）</p> |
| Hyper-V 虚拟机 (Windows) |Windows Server |<p>[System Center DPM](/documentation/articles/backup-azure-backup-sql/)（+ Azure 备份代理）、</p><p>Azure 备份服务器（包括 Azure 备份代理）</p> |
| Hyper-V 虚拟机 (Linux) |Windows Server |<p>[System Center DPM](/documentation/articles/backup-azure-backup-sql/)（+ Azure 备份代理）、</p><p>Azure 备份服务器（包括 Azure 备份代理）</p> |
| Microsoft SQL Server |Windows Server |<p>[System Center DPM](/documentation/articles/backup-azure-backup-sql/)（+ Azure 备份代理）、</p><p>Azure 备份服务器（包括 Azure 备份代理）</p> |
| Microsoft SharePoint |Windows Server |<p>[System Center DPM](/documentation/articles/backup-azure-backup-sql/)（+ Azure 备份代理）、</p><p>Azure 备份服务器（包括 Azure 备份代理）</p> |
| Microsoft Exchange |Windows Server |<p>[System Center DPM](/documentation/articles/backup-azure-backup-sql/)（+ Azure 备份代理）、</p><p>Azure 备份服务器（包括 Azure 备份代理）</p> |
| Azure IaaS VM (Windows) |在 Azure 中运行 |[Azure 备份（VM 扩展）](/documentation/articles/backup-azure-vms-introduction/) |
| Azure IaaS VM (Linux) |在 Azure 中运行 |[Azure 备份（VM 扩展）](/documentation/articles/backup-azure-vms-introduction/) |

## Linux 支持
下表显示了支持 Linux 的 Azure 备份组件。

| 组件 | Linux（Azure 认可）支持 |
| --- | --- |
| Azure 备份 (MARS) 代理 |否（仅限基于 Windows 的代理） |
| System Center DPM |文件一致性备份仅支持 Hyper-V<br/>（不适用于 Azure VM） |
| Azure 备份服务器 |文件一致性备份仅支持 Hyper-V<br/>（不适用于 Azure VM） |
| Azure IaaS VM 备份 |是 |

## 结合使用高级存储 VM 和 Azure 备份
Azure 备份可保护高级存储 VM。Azure 高级存储是基于固态硬盘 (SSD) 的存储，用于支持 I/O 密集型工作负荷。高级存储很适合虚拟机 (VM) 工作负荷。有关高级存储的详细信息，请参阅[高级存储：Azure 虚拟机工作负荷的高性能存储](/documentation/articles/storage-premium-storage/)一文

### 备份高级存储 VM <a name="back-up-and-restore-premium-storage-vms"></a>
在备份高级存储 VM 时，备份服务在高级存储帐户中创建临时暂存位置。名为“AzureBackup-”的暂存位置相当于连接到 VM 的高级磁盘的数据大小总计。检查存储帐户中的临时暂存位置是否有足够可用空间。有关其他信息，请参阅[高级存储限制](/documentation/articles/storage-premium-storage/#premium-storage-scalability-and-performance-targets/)一文。

> [AZURE.NOTE]
请不要修改或编辑暂存位置。
>
>

备份作业完成后，将删除暂存位置。用于暂存位置的存储的价格与所有[高级存储定价](/documentation/articles/storage-premium-storage/#pricing-and-billing/)一致。

### 还原高级存储 VM
可以将高级存储 VM 还原为高级存储或常规存储。将高级存储 VM 的恢复点还原到高级存储是典型的还原过程。但是，将高级存储 VM 的恢复点还原到标准存储更符合成本效益。如果需要 VM 中的一部分文件，可以使用这种还原类型。

## 每个备份组件有哪些功能？
以下部分提供了相关表格，总结了每个 Azure 备份组件中各种功能是否可用或受支持。请参阅各表格后的额外支持信息或详细信息。

### 存储
| 功能 | Azure 备份代理 | System Center DPM | Azure 备份服务器 | Azure IaaS VM 备份 |
| --- | --- | --- | --- | --- |
| Azure 备份保管库 |![是][green] |![是][green] |![是][green] |![是][green] |
| 磁盘存储 | |![是][green]  |![是][green] | |
| 表存储 | |![是][green] | | |
| 压缩<br/>（在备份保管库中） |![是][green]|![是][green] |![是][green] | |
| 增量备份 |![是][green] |![是][green] |![是][green] |![是][green] |
| 磁盘重复数据删除 | |![部分][yellow] |![部分][yellow] | |

![表键](./media/backup-introduction-to-azure-backup/table-key.png)

备份保管库是所有组件中首选的存储目标。System Center DPM 和 Azure 备份服务器还提供生成本地磁盘副本的选项。但是，System Center DPM 提供将数据写入磁带存储设备的选项。

#### 压缩
备份经过压缩以减少所需的存储空间。唯一不进行压缩的组件为 VM 扩展。VM 扩展可将所有备份数据从存储帐户复制到同一区域中的备份保管库。传输数据时不使用压缩。传输数据但不压缩会稍微增加所用的存储空间。但是，存储数据而不压缩可加快还原，实现特定的恢复点目标。

#### 增量备份
无论目标存储（磁盘、磁带、备份保管库）如何，每个组件都支持增量备份。增量备份仅传输自上次备份以来所做的更改，从而可以确保备份在存储空间和时间方面高效。

#### 磁盘重复数据删除
[在 Hyper-V 虚拟机上](http://blogs.technet.com/b/dpm/archive/2015/01/06/deduplication-of-dpm-storage-reduce-dpm-storage-consumption.aspx)部署 System Center DPM 或 Azure 备份服务器时，可以利用重复数据删除。Windows Server 将在以备份存储形式附加到虚拟机的虚拟硬盘 (VHD) 上执行主机级别的重复数据删除。

> [AZURE.NOTE]
重复数据删除不一定适用于 Azure 中的所有备份组件。如果 System Center DPM 和备份服务器部署在 Azure 中，则附加到 VM 的存储磁盘无法进行重复数据删除。
>
>

### 安全
| 功能 | Azure 备份代理 | System Center DPM | Azure 备份服务器 | Azure IaaS VM 备份 |
| --- | --- | --- | --- | --- |
| 网络安全性<br/>（到 Azure） |![是][green] |![是][green] |![是][green] |![部分][yellow] |
| 数据安全性<br/>（Azure 中） |![是][green] |![是][green] |![是][green] |![部分][yellow] |

![表键](./media/backup-introduction-to-azure-backup/table-key.png)  


#### 网络安全
从服务器到备份保管库的所有备份流量均通过高级加密标准 256 进行加密。备份数据通过安全的 HTTPS 链接发送。备份数据还会以加密格式存储在备份保管库中。只有 Azure 客户具有解锁此数据的通行短语。Microsoft 无法解密任何位置的备份数据。

> [AZURE.WARNING]
建立备份保管库后，只有你才能访问加密密钥。Microsoft 绝不会保留加密密钥副本，且没有访问该密钥的权限。如果客户丢失了密钥，Microsoft 无法恢复备份数据。
>
>

#### 数据安全性
备份 Azure VM 时，需要在虚拟机*内部*设置加密。在 Windows 虚拟机上使用 BitLocker，在 Linux 虚拟机上使用 **dm-crypt**。Azure 备份不会自动加密来自此路径的备份数据。

### 网络
| 功能 | Azure 备份代理 | System Center DPM | Azure 备份服务器 | Azure IaaS VM 备份 |
| --- | --- | --- | --- | --- |
| 网络压缩<br/>（到**备份服务器**） | |![是][green] |![是][green] | |
| 网络压缩<br/>（到**备份保管库**） |![是][green] |![是][green] |![是][green] | |
| 网络协议<br/>（到**备份服务器**） | |TCP |TCP | |
| 网络协议<br/>（到**备份保管库**） |HTTPS |HTTPS |HTTPS |HTTPS |

![表键](./media/backup-introduction-to-azure-backup/table-key-2.png)  


IaaS VM 上的 VM 扩展会通过存储网络直接读取 Azure 存储帐户中的数据，因此无需优化此流量。

若要将数据备份到 System Center DPM 或 Azure 备份服务器，需要压缩从主服务器传送到备份服务器的数据。在备份到 DPM 或 Azure 备份服务器之前压缩数据可以节省带宽。

#### 网络限制
Azure 备份代理提供网络限制功能，可用于控制数据传输期间的网络带宽使用方式。如果需要在上班时间内备份数据，但不希望备份程序干扰其他 Internet 流量，限制会很有帮助。数据传输的限制适用于备份和还原活动。

### 备份和保留

针对每个受保护实例，Azure 备份的恢复点数（也称为备份副本或快照）限制为 9999。受保护实例指配置为将数据备份到 Azure 的计算机、物理或虚拟服务器，或工作负荷。有关详细信息，请参阅[受保护实例是什么](/documentation/articles/backup-introduction-to-azure-backup/#what-is-a-protected-instance/)一节。保存数据的备份副本后，实例将受保护。数据的备份副本就是保障。如果源数据丢失或损坏，备份副本可还原源数据。下表显示每个组件的最大备份频率。备份策略配置决定使用恢复点的速度。例如，如果每天创建恢复点，那么恢复点可保留 27 年才到期。如果每月创建恢复点，那么恢复点可保留 833 年才到期。备份服务不会为恢复点设置到期时间限制。

| | Azure 备份代理 | System Center DPM | Azure 备份服务器 | Azure IaaS VM 备份 |
| --- | --- | --- | --- | --- |
| 备份频率<br/>（到备份保管库） |每天三次备份 |每天备份两次 |每天备份两次 |每天一次备份 |
| 备份频率<br/>（到磁盘） |不适用 |<li>SQL Server 每 15 分钟一次<li>其他工作负荷每小时一次 |<li>SQL Server 每 15 分钟一次<li>其他工作负荷每小时一次</p> |不适用 |
| 保留期选项 |每日、每周、每月、每年 |每日、每周、每月、每年 |每日、每周、每月、每年 |每日、每周、每月、每年 |
| 每个受保护实例的最大恢复点数 |9999|9999|9999|9999|
| 最大保留期 |取决于备份频率 |取决于备份频率 |取决于备份频率 |取决于备份频率 |
| 本地磁盘上的恢复点 |不适用 |<li>对于文件服务器为 64，<li>对于应用程序服务器为 448 |<li>对于文件服务器为 64，<li>对于应用程序服务器为 448 |不适用 |
| 磁带上的恢复点 |不适用 |不受限制 |不适用 |不适用 |

## 受保护实例是什么
受保护实例通常指配置为备份到 Azure 的 Windows 计算机、物理或虚拟服务器，或者 SQL 数据库。为计算机、服务器或数据库设置备份策略，并为数据创建副本后，实例即受到保护。该受保护实例的备份数据后续副本（称为恢复点）将增加使用的存储量。可为每个受保护实例创建最多 9999 个恢复点。如果从存储中删除恢复点，则不计入 9999 个总恢复点数。受保护实例的一些常见示例包括运行 Windows 操作系统的虚拟机、应用程序服务器、数据库和个人计算机。例如：

- 运行 Hyper-V 或 Azure IaaS 虚拟机监控程序结构的虚拟机。此虚拟机的来宾操作系统可以是 Windows Server 或 Linux。
- 应用程序服务器：应用程序服务器可以是运行 Windows Server 的物理或虚拟机，也可以是包含要备份的数据的工作负荷。常见工作负荷有 Microsoft SQL Server、Microsoft Exchange 服务器、Microsoft SharePoint 服务器、Microsoft Dynamics 和 Windows Server 的文件服务器角色。备份这些工作负荷需要 System Center Data Protection Manager (DPM) 或 Azure 备份服务器。
- 运行 Windows 操作系统的个人计算机或笔记本电脑。


## 什么是保管库凭据文件？ <a name="what-is-the-vault-credential-file"></a>
保管库凭据文件是门户为每个备份保管库生成的证书。然后，门户会将公钥上载到访问控制服务 (ACS)。下载凭据时，会向你提供私钥。可用它来注册要保护的计算机。私钥可用于对要将备份数据发送到特定备份保管库的服务器或计算机进行身份验证。

只能使用保管库凭据来注册服务器或计算机。但是，请保存好保管库凭据，如果它丢失或由他人获取，这些凭据可能会用于针对相同的保管库注册其他计算机。由于备份数据使用通行短语进行了加密，因此只有你可访问，现有的备份数据不会泄露。保管库凭据会在 48 小时后过期。虽然可随时下载备份保管库的保管库凭据，但只有最新的凭据可用于注册。

## Azure 备份与 Azure Site Recovery 有何不同？
Azure 备份和 Azure Site Recovery 在这两种服务的备份数据方面相关并且可还原该数据，但两者的核心价值主张不同。

Azure 备份保护本地和云中的数据。Azure Site Recovery 可以协调虚拟机和物理服务器的复制、故障转移和故障回复。这两个服务都很重要，因为灾难恢复解决方案需要让数据保持安全且可恢复（备份），*同时*在服务中断时使工作负荷保持可用 (Site Recovery)。

以下概念可帮助你做出有关备份和灾难恢复的重要决策。

| 概念 | 详细信息 | 备份 | 灾难恢复 (DR) |
| --- | --- | --- | --- |
| 恢复点目标 (RPO) |在需要执行恢复的情况下可接受的数据丢失量。 |备份解决方案的可接受 RPO 存在很大差异。虚拟机备份的 RPO 通常为一天，而数据库备份的 RPO 只有 15 分钟。 |灾难恢复解决方案的 RPO 较低。DR 复制可以落后几秒钟或几分钟时间。 |
| 恢复时间目标 (RTO) |完成恢复或还原所需的时间量。 |由于 RPO 较大，备份解决方案需要处理的数据量通常更多，这会导致 RTO 较长。例如，根据从异地转送磁带所需的时间，从磁带还原数据可能需要数天的时间。 |由于灾难恢复解决方案与源之间的同步程度更高，因此其 RTO 更小，需要处理的更改也更少。 |
| 保留 |数据需要存储多久 |对于需要进行操作恢复的情况（数据损坏、意外的文件删除、OS 故障），备份数据通常会保留 30 天或更短。<br>从合规性的角度来看，数据可能需要存储数月甚至数年。在这种情况下，备份数据非常适合存档。 |灾难恢复只需操作性恢复数据，通常只需几个小时或最多一天的数据。由于 DR 解决方案中使用了精细数据捕获，因此不建议长期保留 DR 数据。 |

## 后续步骤
请参阅下述某个教程，详细了解在 Windows Server 上保护数据或在 Azure 中保护虚拟机 (VM) 的分步说明：

- [备份文件和文件夹](/documentation/articles/backup-try-azure-backup-in-10-mins/)

有关保护其他工作负荷的详细信息，请尝试阅读以下文章之一：

- [备份 Windows Server](/documentation/articles/backup-configure-vault/)
- [Backup Azure IaaS VMs（备份 Azure IaaS VM）](/documentation/articles/backup-azure-vms-prepare/)

[green]: ./media/backup-introduction-to-azure-backup/green.png
[yellow]: ./media/backup-introduction-to-azure-backup/yellow.png
[red]: ./media/backup-introduction-to-azure-backup/red.png

<!---HONumber=Mooncake_0116_2017-->
<!---Update_Description: wording update -->
