<properties
	pageTitle="什么是 Azure 备份？ | Azure"
	description="使用 Azure 备份和恢复服务，你可以从 Windows 服务器、Windows 客户端计算机、SCDPM 服务器或 Azure 虚拟机备份和还原数据与应用程序。"
	services="backup"
	documentationCenter=""
	authors="Jim-Parker"
	manager="jwhit"
	editor="tysonn"
	keywords="备份和还原;恢复服务"/>

<tags
	ms.service="backup"
	ms.date="01/15/2016"
	wacn.date="02/25/2016"/>


# 什么是 Azure 备份？
Azure 备份是用于备份和还原 Microsoft 云数据的服务。它取代了现有的本地或异地备份解决方案，并且是可靠、安全、高性价比的基于云的解决方案。它还可以保护云中运行的资产。Azure 备份提供的恢复服务构建在一流的基础结构之上，具有可缩放、持久且高度可用的优点。



## 为何使用 Azure 备份？
传统的备份解决方案已演变成将云视为类似于磁盘或磁带的终结点。尽管这种方法很简单，但它的用途有限，并且不能充分利用基础云平台。这相当于一个效率低下而昂贵的解决方案。
在另一方面，Azure 备份提供强大且经济实惠的云备份解决方案具备的所有优势。Azure 备份提供的部分主要优点包括：

| 功能 | 优势 |
| ------- | ------- |
| 自动存储管理 | 不需要为本地存储设备投资。Azure 备份自动分配和管理备份存储，采用即用即付的付费模式。 |
| 无限缩放 | 高可用性保证不会出现额外的维护与监视开销。Azure 备份具有不中断的自动缩放功能，使用 Azure 云的基础能力和缩放性。 |
| 多种存储选项 | 根据需要选择备份存储：<li>LRS（本地冗余存储）块 Blob 非常适合注重价格的客户，同时为数据提供本地硬件故障保护。<li>GRS（异地复制存储）块 Blob 在配对的数据中心提供 3 个额外的副本，即使发生 Azure 站点级别的灾难，也能确保备份数据具有高可用性。 |
| 无限制的数据传输 | 在还原操作过程中，从 Azure 备份保管库进行任何传出（出站）数据传输无需支付费用。向 Azure 传入数据也是免费的。 |
| 集中管理 | Azure 门户提供简单熟悉的界面。随着服务的发展，你可以使用中心管理等功能，从单个位置管理你的备份基础结构。 |
| 数据加密 | 在公有云中安全传输和存储客户数据。加密通行短语存储在源中，永远不会传输或存储到 Azure。还原任何数据都要使用加密密钥，只有客户才对服务中的数据拥有完全访问权限。 |  
| VSS 集成 | 在 Windows 上提供应用程序一致的备份可确保还原时不需要修复。这可以减少 RTO 并使客户能够更快地恢复运行状态。 |
| 长期保留 | 备份到 Azure 后，客户无需支付场外磁带备份解决方案的费用，Azure 以极低的价格提供类似于磁带语义的有吸引力解决方案。 |

## Azure 备份组件
Azure 备份是混合式备份解决方案，它由多个共同工作的组件构成，支持端到端的备份和还原工作流。

![Azure 备份组件](./media/backup-introduction-to-azure-backup/azure-backup-overview.png)

## 部署方案
| 组件 | 可以在 Azure 中部署吗？ | 可以在本地部署吗？ | 支持的目标存储|
| --- | --- | --- | --- |
| Azure 备份代理 | <p>**是**</p><p>Azure 备份代理可以部署在 Azure 中运行的任何 Windows Server VM 上。</p> | <p>**是**</p><p>Azure 备份代理可以部署在 Azure 中运行的任何 Windows Server VM 或物理机上。</p> | <p>Azure 备份保管库</p> |
| System Center Data Protection Manager (SCDPM) | <p>是</p><p>请了解有关[使用 SCDPM 保护 Azure 中的工作负荷](http://blogs.technet.com/b/dpm/archive/2014/09/02/azure-iaas-workload-protection-using-data-protection-manager.aspx)的详细信息。</p> | <p>是</p><p>请了解有关[保护数据中心内的工作负荷和 VM](https://technet.microsoft.com/library/hh758173.aspx) 的详细信息。</p> | <p>本地附加的磁盘、</p><p>Azure 备份保管库、</p><p>磁带（仅限本地）</p> |
| Azure 备份（VM 扩展） | <p>是</p><p>专门用于[备份 Azure IaaS 虚拟机](/documentation/articles/backup-azure-vms-introduction)。</p> | <p>否</p><p>请使用 SCDPM 备份数据中心内的虚拟机。</p> | <p>Azure 备份保管库</p> |

## 可以备份哪些应用程序和工作负荷？

| 工作负载 | 源计算机 | Azure 备份解决方案 |
| --- | --- |---|
| 文件和文件夹 | Windows Server | <p>[Azure 备份代理](/documentation/articles/backup-configure-vault)、</p><p>[System Center DPM](/documentation/articles/backup-azure-dpm-introduction)、</p><p>[Azure 备份服务器](/documentation/articles/backup-azure-microsoft-azure-backup)</p> |
| 文件和文件夹 | Windows 客户端 | <p>[Azure 备份代理](/documentation/articles/backup-configure-vault)、</p><p>[System Center DPM](/documentation/articles/backup-azure-dpm-introduction)、</p><p>[Azure 备份服务器](/documentation/articles/backup-azure-microsoft-azure-backup)</p> |
| Hyper-V 虚拟机 (Windows) | Windows Server | <p>System Center DPM、</p> |
| Hyper-V 虚拟机 (Linux) | Windows Server | <p>System Center DPM、</p> |
| Microsoft SQL Server | Windows Server | <p>[System Center DPM](/documentation/articles/backup-azure-backup-sql)、</p> |
| Microsoft SharePoint | Windows Server | <p>[System Center DPM](/documentation/articles/backup-azure-backup-sql)、</p> |
| Microsoft Exchange | Windows Server | <p>[System Center DPM](/documentation/articles/backup-azure-backup-sql)、</p>|
| Azure IaaS VM (Windows)| - | [Azure 备份（VM 扩展）](/documentation/articles/backup-azure-vms-introduction)|
| Azure IaaS VM (Linux) | - | [Azure 备份（VM 扩展）](/documentation/articles/backup-azure-vms-introduction)|

## 功能
下列表格汇总了如何在每个组件中处理 Azure 备份功能：

### 1\.存储

| 功能 | Azure 备份代理 | SCDPM | Azure 备份服务器 | Azure 备份（VM 扩展） |
| ------- | --- | --- | --- | ---- |
| Azure 备份保管库 | ![是][green] | ![是][green] | ![是][green] | ![是][green] |
| 磁盘存储 | | ![是][green] | ![是][green] | |
| 表存储 | | ![是][green] | | |
| 压缩（在备份保管库中） | ![是][green] | ![是][green]| ![是][green] | |
| 增量备份 | ![是][green] | ![是][green] | ![是][green] | ![是][green] |
| 磁盘重复数据删除 | | ![部分][yellow] | ![部分][yellow]| | |

Azure 备份保管库是所有组件中首选的存储目标。SCDPM 和 Azure 备份服务器都提供创建本地磁盘副本的选项，但只有 SCDPM 提供将数据写入磁带存储设备的选项。

#### 增量备份
与目标存储（磁盘、磁带、备份保管库）无关，每个组件都支持增量备份。仅获取上次备份后的增量更改，然后将这些更改传输到目标存储，从而确保备份有效利用存储和时间。此外，备份已经过压缩，以减少存储占用空间。

不进行压缩的组件为 VM 扩展。所有备份数据从客户存储帐户复制到同一区域中的备份保管库，且不进行压缩。尽管这会稍微增大使用的存储，但存储未经压缩的数据可以加速还原。

#### 重复数据删除
[部署在 Hyper-V 虚拟机中](http://blogs.technet.com/b/dpm/archive/2015/01/06/deduplication-of-dpm-storage-reduce-dpm-storage-consumption.aspx)的 SCDPM 和 Azure 备份服务器支持重复数据删除。重复数据删除是利用 Windows Server 功能在主机级别执行的 - 在作为备份存储附加到虚拟机的 VHD 上执行。

>[AZURE.WARNING] 重复数据删除不适用于 Azure 中的任何 Azure 备份组件。 如果 SCDPM 和 Azure 备份服务器部署在 Azure 中，则附加到 VM 的存储磁盘无法进行重复数据删除。

### 2\.“安全”

| 功能 | Azure 备份代理 | SCDPM | Azure 备份服务器 | Azure 备份（VM 扩展） |
| ------- | --- | --- | --- | ---- |
| 网络安全性（到 Azure） | ![是][green] |![是][green] | ![是][green] | ![部分][yellow]|
| 数据安全性（Azure 中） | ![是][green] |![是][green] | ![是][green] | ![部分][yellow]|

从服务器到 Azure 备份保管库的所有备份流量都已使用 AES256 加密，数据通过安全的 HTTPS 链接发送。备份数据还会以加密格式存储在备份保管库中。只有持有通行短语的客户可以解锁此数据 - Microsoft 无论何时都不能解密备份数据。

>[AZURE.WARNING] 只有客户持有用于加密备份数据的密钥。Microsoft 不会在 Azure 中保留副本，并且无权访问密钥。如果客户丢失了密钥，Microsoft 无法恢复备份数据。

对于 Azure VM 备份，必须在虚拟机*内部*显式设置加密。在 Windows 虚拟机上使用 BitLocker，在 Linux 虚拟机上使用 dm-crypt。Azure 备份不会自动加密来自此路径的备份数据。

### 3\.支持的工作负荷

| 功能 | Azure 备份代理 | SCDPM | Azure 备份服务器 | Azure 备份（VM 扩展） |
| ------- | --- | --- | --- | ---- |
| Windows Server 计算机 - 文件/文件夹 | ![是][green] | ![是][green] | ![是][green] | |
| Windows 客户端计算机 - 文件/文件夹 | ![是][green] | ![是][green] | ![是][green] | |
| Hyper-V 虚拟机 (Windows) | | ![是][green] | ![是][green] | |
| Hyper-V 虚拟机 (Linux) | | ![是][green] | ![是][green] | |
| Microsoft SQL Server | | ![是][green] | ![是][green] | |
| Microsoft SharePoint | | ![是][green] | ![是][green] | |
| Microsoft Exchange | | ![是][green] | ![是][green] | |
| Azure 虚拟机 (Windows) | | | | ![是][green] |
| Azure 虚拟机 (Linux) | | | | ![是][green] |

### 4\.网络

| 功能 | Azure 备份代理 | SCDPM | Azure 备份服务器 | Azure 备份（VM 扩展） |
| ------- | --- | --- | --- | ---- |
| 网络压缩（到备份服务器） | | ![是][green] | ![是][green] | |
| 网络压缩（到备份保管库） | ![是][green] | ![是][green] | ![是][green] | |
| 网络协议（到备份服务器） | | TCP | TCP | |
| 网络协议（到备份保管库） | HTTPS | HTTPS | HTTPS | HTTPS |

由于 VM 扩展通过存储网络直接从 Azure 存储帐户读取数据，因此不需要优化此流量。流量通过 Azure 数据中心的本地存储网络传送，因此不太需要出于带宽的考虑而压缩数据。

在备份服务器（SCDPM 或 Azure 备份服务器）上保护数据的客户也可以压缩从主服务器传输到备份服务器的流量以节省带宽。

## Azure 备份与 Azure Site Recovery 有何不同？
许多客户将备份与灾难恢复混为一谈。两者都可以捕获数据和提供还原语义，但两者的核心价值主张不同。

Azure 备份在本地和云中备份数据。Azure Site Recovery 可以协调虚拟机和物理服务器的复制、故障转移和故障回复。需要两者才能构成完整的灾难恢复解决方案，因为灾难恢复策略需要将数据保持安全且可恢复（Azure 备份），同时在服务中断时使工作负荷可供使用并可访问 (Azure Site Recovery)。

若要在备份和灾难恢复方面做出决策，应了解以下一些重要概念：

| 概念 | 详细信息 | 备份 | 灾难恢复 (DR) |
| ------- | ------- | ------ | ----------------- |
| 恢复点目标 (RPO) | 在需要执行恢复的情况下可接受的数据丢失量。 | 备份解决方案的可接受 RPO 具有广泛的差异。虚拟机备份的 RPO 通常为 1 天，而数据库备份的 RPO 只有 15 分钟。 | 灾难恢复解决方案的 RPO 极低。DR 复制可以落后几秒钟或几分钟时间。 |
| 恢复时间目标 (RTO) | 完成恢复/还原所需的时间量。 | 由于 RPO 较大，备份解决方案需要处理的数据量通常更多 - 这会导致 RTO 较长。例如，根据从异地转送磁带所需的时间，从磁带还原数据可能需要数天的时间。 | 由于灾难恢复解决方案与源之间的同步程度更高，因此其 RTO 更小，需要处理的更改也更少。 |
| 保留 | 数据需要存储多久 | <p>对于需要进行操作恢复的情况（数据损坏、意外的文件删除、OS 故障），备份数据通常会保留 30 天或更短。</p> <p>从合规性的立场来看，数据可能需要存储几个月或甚至数年。在这种情况下，备份数据非常适合存档。</p> | 灾难恢复只需操作性恢复数据 - 通常只需几个小时或最多一天的数据。由于 DR 解决方案中使用了精细数据捕获，因此不建议长期保留 DR 数据。 |

## 后续步骤
- [尝试 Azure 备份](/documentation/articles/backup-try-azure-backup-in-10-mins)
- [此处](/documentation/articles/backup-azure-backup-faq)列出了有关 Azure 备份服务的常见问题。
- 访问 [Azure 备份论坛](http://go.microsoft.com/fwlink/p/?LinkId=290933)。


[green]: ./media/backup-introduction-to-azure-backup/green.png
[yellow]: ./media/backup-introduction-to-azure-backup/yellow.png
[red]: ./media/backup-introduction-to-azure-backup/red.png

<!---HONumber=Mooncake_0215_2016-->