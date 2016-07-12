<properties
	pageTitle="什么是 Azure 备份？| Azure"
	description="使用 Azure 备份和恢复服务，你可以从 Windows 服务器、Windows 客户端计算机、System Center DPM 服务器和 Azure 虚拟机备份和还原数据与应用程序。"
	services="backup"
	documentationCenter=""
	authors="Jim-Parker"
	manager="jwhit"
	editor="tysonn"
	keywords="备份和还原;恢复服务;备份解决方案"/>

<tags
	ms.service="backup"
	ms.date="04/13/2016"
	wacn.date="05/20/2016"/>


# 什么是 Azure 备份？
Azure 备份是用于备份和还原 Microsoft 云数据的服务。它取代了现有的本地或异地备份解决方案，并且是可靠、安全、高性价比的基于云的解决方案。它还有助于保护在云中运行的资产。Azure 备份提供的恢复服务构建在一流的基础结构之上，具有可缩放、持久且高度可用的优点。



## 为何使用 Azure 备份？
传统的备份解决方案已演变成将云视为类似于磁盘或磁带的终结点。这种方法很简单，但也存在限制。它并不能充分利用基础云平台，可谓是效率不佳又昂贵的解决方案。
相比之下，Azure 备份可提供功能强大且价格合理的云备份解决方案的所有优点。Azure 备份提供的部分主要优点如下：

| 功能 | 优势 |
| ------- | ------- |
| 自动存储管理 | 不需要为本地存储设备投资。Azure 备份自动分配和管理备份存储，采用即用即付的付费模式。 |
| 无限缩放 | 充分利用高可用性保障，不会出现额外的维护与监视开销。Azure 备份具有不中断的自动缩放功能，使用 Azure 云的基础能力和缩放性。 |
| 多种存储选项 | 根据需要选择备份存储：<li>本地冗余存储块 blob 非常适合注重价格的客户，同时为数据提供本地硬件故障保护。<li>异地复制存储块 blob 可以在配对的数据中心提供三个额外的副本。这些额外的副本可以确保你的备份数据即使在出现 Azure 站点级灾难的情况下，也具有很高的可用性。 |
| 无限制的数据传输 | 在还原操作过程中，从备份保管库进行任何传出（出站）数据传输无需支付费用。向 Azure 传入数据也是免费的。 |
| 数据加密 | 数据加密允许在公有云中安全地传输和存储客户数据。加密通行短语存储在源中，永远不会传输或存储到 Azure。还原任何数据都要使用加密密钥，只有客户才对服务中的数据拥有完全访问权限。 |  
| 应用程序一致性备份 | Windows 上的应用程序一致备份有助于确保在还原时不需要修复，以减少恢复时间目标并可让你客户更快地回到运行中状态。 |
| 长期保留 | 与其为异地磁带备份解决方案付费，客户可以备份到以低成本提供令人信服且近似磁带解决方案的 Azure。 |

## Azure 备份组件
该备份是混合式备份解决方案，因此是由多个共同工作的组件构成的，支持端到端的备份并可还原工作流。

![Azure 备份组件](./media/backup-introduction-to-azure-backup/azure-backup-overview.png)

## 部署方案

| 组件 | 可以在 Azure 中部署吗？ | 可以在本地部署吗？ | 支持的目标存储|
| --- | --- | --- | --- |
| Azure 备份代理 | <p>**是**</p> <p>Azure 备份代理可以部署在 Azure 中运行的任何 Windows Server VM 上。</p> | <p>**是**</p> <p>该备份代理可以部署在任何 Windows Server VM 或物理机上。</p> | <p>Azure 备份保管库</p> |
| System Center Data Protection Manager (DPM) | <p>**是**</p> <p>详细了解[如何使用 System Center DPM 保护 Azure 中的工作负荷](http://blogs.technet.com/b/dpm/archive/2014/09/02/azure-iaas-workload-protection-using-data-protection-manager.aspx)。</p> | <p>**是**</p> <p>详细了解[如何保护数据中心的工作负荷和 VM](https://technet.microsoft.com/library/hh758173.aspx)。</p> | <p>本地附加的磁盘、</p><p>Azure 备份保管库、</p><p>磁带（仅限本地）</p> |
| Azure 备份服务器 | <p>**是**</p> | <p>**是**</p> </p> | <p>本地附加的磁盘、</p><p>Azure 备份保管库</p> |
| Azure 备份（VM 扩展） | <p>是</p> <p>专门用于[备份 Azure 基础结构即服务 (IaaS) 虚拟机](/documentation/articles/backup-azure-vms-introduction/)。</p> | <p>**否**</p> <p>使用 System Center DPM 备份数据中心的虚拟机。</p> | <p>Azure 备份保管库</p> |

### 组件级别的优势和限制

| 组件 | 优点 | 限制 | 恢复粒度 |
| --- | --- | --- | --- |
| Azure 备份 (MARS) 代理 | <li>可以在 Windows OS 计算机（不管是物理机还是虚拟机）上备份文件和文件夹（VM 可位于本地或 Azure 上的任何位置）<li>不需要单独的备份服务器<li>使用 Azure 备份保管库 | <li>一天备份三次/文件级还原<li>仅限文件/文件夹/卷级还原，非应用程序感知<li>不支持 Linux | 文件/文件夹/卷 |
| System Center Data Protection Manager | <li>应用感知的快照 (VSS)<li>创建备份时提供充分的弹性<li>恢复粒度（全部）<li>可以使用 Azure 备份保管库<li>Linux 支持（如果托管在 Hyper-V 上） | <li>缺少异构支持（VMware VM 备份、Oracle 工作负荷备份）。 | 文件/文件夹/卷<br>/VM/应用程序 |
| Microsoft Azure 备份服务器 | <li>应用感知的快照 (VSS)<li>创建备份时提供充分的弹性<li>恢复粒度（全部）<li>可以使用 Azure 备份保管库<li>Linux 支持（如果托管在 Hyper-V 上）<li>不需要 System Center 许可证 | <li>缺少异构支持（VMware VM 备份、Oracle 工作负荷备份）。<li>始终需要实时 Azure 订阅<li>不支持磁带备份 | 文件/文件夹/卷<br>/VM/应用程序 |
| Azure IaaS VM 备份 | <li>针对 Windows/Linux 的本机备份<li>不需要安装特定的代理<li>无需使用备份基础结构进行结构级备份 | <li>一天备份一次/磁盘级还原<li>无法本地备份 | VM<br>所有磁盘（使用 PowerShell） |

## 可以备份哪些应用程序和工作负荷？

| 工作负载 | 源计算机 | Azure 备份解决方案 |
| --- | --- |---|
| 文件和文件夹 | Windows Server | <p>[Azure 备份代理](/documentation/articles/backup-configure-vault/)、</p><p>[System Center DPM](/documentation/articles/backup-azure-dpm-introduction/)、</p> |
| 文件和文件夹 | Windows 客户端 | <p>[Azure 备份代理](/documentation/articles/backup-configure-vault/)、</p><p>[System Center DPM](/documentation/articles/backup-azure-dpm-introduction/)、</p>|
| Hyper-V 虚拟机 (Windows) | Windows Server | <p>[System Center DPM](/documentation/articles/backup-azure-backup-sql/)、</p> |
| Hyper-V 虚拟机 (Linux) | Windows Server | <p>[System Center DPM](/documentation/articles/backup-azure-backup-sql/)、</p> |
| Microsoft SQL Server | Windows Server | <p>[System Center DPM](/documentation/articles/backup-azure-backup-sql/)、</p> |
| Microsoft SharePoint | Windows Server | <p>[System Center DPM](/documentation/articles/backup-azure-backup-sql/)、</p>|
| Microsoft Exchange | Windows Server | <p>[System Center DPM](/documentation/articles/backup-azure-backup-sql/)、</p> |
| Azure IaaS VM (Windows)| - | [Azure 备份（VM 扩展）](/documentation/articles/backup-azure-vms-introduction/)|
| Azure IaaS VM (Linux) | - | [Azure 备份（VM 扩展）](/documentation/articles/backup-azure-vms-introduction/)|
## ARM 和 Linux 支持

| 组件 | ARM 支持 | Linux（Azure 认可）支持 |
| --- | --- | --- |
| Azure 备份 (MARS) 代理 | 是 | 否（仅限基于 Windows 的代理） |
| System Center Data Protection Manager | 是（来宾中的代理） | 仅限 Hyper-V（非 Azure VM），只能进行文件一致的备份 |
| Azure 备份服务器 (MABS) | 是（来宾中的代理） | 仅限 Hyper-V（非 Azure VM），只能进行文件一致的备份（与 DPM 相同） |
| Azure IaaS VM 备份 | 公共预览版 | 在公开预览中 - Resource Manager 部署模型中的 Linux VM<br>（文件系统级一致性）<br><br>是（对于经典部署模型中的 Linux VM） |

[AZURE.INCLUDE [了解部署模型](../includes/learn-about-deployment-models-include.md)]


## 备份和还原高级存储 VM

Azure 备份服务现在可以保护高级存储 VM。

### 备份高级存储 VM

在备份高级存储 VM 时，备份服务在高级存储帐户中创建临时暂存位置。名为“AzureBackup-”的暂存位置相当于连接到 VM 的高级磁盘的数据大小总计。

>[AZURE.NOTE] 请不要修改或编辑暂存位置。

备份作业完成后，将删除暂存位置。用于暂存位置的存储的价格在所有[高级存储定价](/documentation/articles/storage-premium-storage/#pricing-and-billing)层中保持一致。

### 还原高级存储 VM

将高级存储 VM 的恢复点还原到高级存储是典型的还原过程。但是，将高级存储 VM 的恢复点还原到标准存储更符合成本效益。如果需要 VM 中的一部分文件，可以使用这种还原类型。

将高级存储 VM 的恢复点还原到高级存储的步骤如下：

1. [将 VM 恢复点还原到标准存储。](/documentation/articles/backup-azure-restore-vms/)
2. [将磁盘复制到高级存储。](/documentation/articles/storage-use-azcopy/)

## 功能
下面的五个表汇总了如何在每个组件中处理备份功能。

### 存储

| 功能 | Azure 备份代理 | System Center DPM | Azure 备份服务器 | Azure 备份（VM 扩展） |
| ------- | --- | --- | --- | ---- |
| Azure 备份保管库 | ![是][green] | ![是][green] | ![是][green] | ![是][green] |
| 磁盘存储 | | ![是][green] | ![是][green] | |
| 表存储 | | ![是][green] | | |
| 压缩（在备份保管库中） | ![是][green] | ![是][green]| ![是][green] | |
| 增量备份 | ![是][green] | ![是][green] | ![是][green] | ![是][green] |
| 磁盘重复数据删除 | | ![部分][yellow] | ![部分][yellow]| | |

![表键](./media/backup-introduction-to-azure-backup/table-key.png)

备份保管库是所有组件中首选的存储目标。System Center DPM 和备份服务器还提供生成本地磁盘副本的选项。但是，System Center DPM 提供将数据写入磁带存储设备的选项。

#### 增量备份
无论目标存储（磁盘、磁带、备份保管库）为何，每个组件都支持增量备份。增量备份可帮助确保有效使用备份的存储和时间。做法是只传输上次备份到目标存储后所做的更改。

#### 压缩
备份经过压缩以减少所需的存储空间。唯一不进行压缩的组件为 VM 扩展。使用 VM 扩展时，所有备份数据从客户存储帐户复制到同一区域中的备份保管库，且不进行压缩。尽管不经过压缩会稍微增大使用的存储，但存储未经压缩的数据可以加速还原。

#### 重复数据删除
[部署在 Hyper-V 虚拟机中](http://blogs.technet.com/b/dpm/archive/2015/01/06/deduplication-of-dpm-storage-reduce-dpm-storage-consumption.aspx)的 System Center DPM 和备份服务器支持重复数据删除。重复数据删除是使用 Windows Server 的重复数据删除功能在主机级别执行的，具体说来是在作为备份存储附加到虚拟机的虚拟硬盘 (VHD) 上执行。

>[AZURE.WARNING] 重复数据删除不适用于 Azure 中的任何备份组件。如果 System Center DPM 和备份服务器部署在 Azure 中，则附加到 VM 的存储磁盘无法进行重复数据删除。

### “安全”

| 功能 | Azure 备份代理 | System Center DPM | Azure 备份服务器 | Azure 备份（VM 扩展） |
| ------- | --- | --- | --- | ---- |
| 网络安全性（到 Azure） | ![是][green] |![是][green] | ![是][green] | ![部分][yellow]|
| 数据安全性（Azure 中） | ![是][green] |![是][green] | ![是][green] | ![部分][yellow]|

![表键](./media/backup-introduction-to-azure-backup/table-key.png)

从服务器到备份保管库的所有备份流量均通过高级加密标准 256 进行加密。数据通过安全的 HTTPS 链接发送。备份数据还会以加密格式存储在备份保管库中。只有持有通行短语的客户才能解锁该数据。Microsoft 无法解密任何位置的备份数据。

>[AZURE.WARNING] 只有客户持有用于加密备份数据的密钥。Microsoft 不会在 Azure 中保留副本，并且无权访问密钥。如果客户丢失了密钥，Microsoft 无法恢复备份数据。

备份 Azure VM 时，需要在虚拟机内部设置加密。在 Windows 虚拟机上使用 BitLocker，在 Linux 虚拟机上使用 **dm-crypt**。Azure 备份不会自动加密来自此路径的备份数据。

### 支持的工作负荷

| 功能 | Azure 备份代理 | System Center DPM | Azure 备份服务器 | Azure 备份（VM 扩展） |
| ------- | --- | --- | --- | ---- |
| Windows Server 计算机--文件和文件夹 | ![是][green] | ![是][green] | ![是][green] | |
| Windows 客户端计算机--文件和文件夹 | ![是][green] | ![是][green] | ![是][green] | |
| Hyper-V 虚拟机 (Windows) | | ![是][green] | ![是][green] | |
| Hyper-V 虚拟机 (Linux) | | ![是][green] | ![是][green] | |
| Microsoft SQL Server | | ![是][green] | ![是][green] | |
| Microsoft SharePoint | | ![是][green] | ![是][green] | |
| Microsoft Exchange | | ![是][green] | ![是][green] | |
| Azure 虚拟机 (Windows) | | | | ![是][green] |
| Azure 虚拟机 (Linux) | | | | ![是][green] |

![表键](./media/backup-introduction-to-azure-backup/table-key-2.png)

### 网络

| 功能 | Azure 备份代理 | System Center DPM | Azure 备份服务器 | Azure 备份（VM 扩展） |
| ------- | --- | --- | --- | ---- |
| 网络压缩（到备份服务器） | | ![是][green] | ![是][green] | |
| 网络压缩（到备份保管库） | ![是][green] | ![是][green] | ![是][green] | |
| 网络协议（到备份服务器） | | TCP | TCP | |
| 网络协议（到备份保管库） | HTTPS | HTTPS | HTTPS | HTTPS |

![表键](./media/backup-introduction-to-azure-backup/table-key-2.png)

由于 VM 扩展通过存储网络直接从 Azure 存储帐户读取数据，因此不需要优化此流量。流量通过 Azure 数据中心的本地存储网络传送，因此不太需要出于带宽的考虑而压缩数据。

如果你要将数据备份到备份服务器（DPM 或备份服务器），可以压缩从主服务器传输到备份服务器的流量以节省带宽。

#### 网络限制
Azure 备份代理提供的限制功能可让你控制在数据传输期间使用网络带宽的方式。如果需要在上班时间内备份数据，但不希望备份程序干扰其他 Internet 流量，这很有帮助。数据传输的限制适用于备份和还原活动。

### 备份和保留

| | Azure 备份代理 | System Center DPM | Azure 备份服务器 | Azure 备份（VM 扩展） |
| --- | --- | --- | --- | --- |
| 备份频率（到备份保管库） | 每天三次备份 | 每天两次备份 |每天两次备份 | 每天一次备份 |
| 备份频率（到磁盘） | 不适用 | <p>SQL Server 每 15 分钟一次</p><p>其他工作负荷每小时一次</p> | <p>SQL Server 每 15 分钟一次</p><p>其他工作负荷每小时一次</p> |不适用 |
| 保留期选项 | 每日、每周、每月、每年 | 每日、每周、每月、每年 | 每日、每周、每月、每年 |每日、每周、每月、每年 |
| 保留期 | 最长 99 年 | 最长 99 年 | 最长 99 年 | 最长 99 年 |
| 备份保管库中的恢复点 | 不受限制 | 不受限制 | 不受限制 | 不受限制 |
| 本地磁盘上的恢复点 | 不适用 | 对于文件服务器为 64，<br><br>对于应用程序服务器为 448 | 对于文件服务器为 64，<br><br>对于应用程序服务器为 448 |不适用 |
| 磁带上的恢复点 | 不适用 | 不受限制 | 不适用 | 不适用 |

## 什么是保管库凭据文件？

保管库凭据文件是经典管理门户为每个备份保管库生成的证书。然后，经典管理门户会将公钥上载到访问控制服务 (ACS)。当用户下载凭据，然后在计算机注册期间输入凭据时，系统将向用户提供私钥。当计算机向 Azure 备份服务中标识的保管库发送备份数据时，该私钥将对计算机进行身份验证。

保管库凭据仅在注册工作流的过程中使用。你需负责确保保管库凭据文件不会泄露。如果该文件落入恶意用户之手，他们可以使用保管库凭据文件将其他计算机注册到同一个保管库。但是，但是备份数据已使用属于客户的通行短语加密，因此现有的备份数据不会泄露。为了消除这种忧虑，保管库凭据设置为 48 小时后过期。尽管你可以下载备份保管库的保管库凭据任意次数，但是，在运行注册工作流期间，只有最新的文件才适用。

## Azure 备份与 Azure Site Recovery 有何不同？
许多客户将备份恢复与灾难恢复混为一谈。两者都可以捕获数据和提供还原语义，但两者的核心价值主张不同。

Azure 备份在本地和云中备份数据。Azure Site Recovery 可以协调虚拟机和物理服务器的复制、故障转移和故障回复。这两个服务都很重要，因为灾难恢复解决方案需要让数据保持安全且可恢复（备份），同时在服务中断时使工作负荷可供使用并可访问（站点恢复）。

以下概念可帮助你做出有关备份和灾难恢复的重要决策。

| 概念 | 详细信息 | 备份 | 灾难恢复 (DR) |
| ------- | ------- | ------ | ----------------- |
| 恢复点目标 (RPO) | 在需要执行恢复的情况下可接受的数据丢失量。 | 备份解决方案的可接受 RPO 存在很大差异。虚拟机备份的 RPO 通常为一天，而数据库备份的 RPO 只有 15 分钟。 | 灾难恢复解决方案的 RPO 较低。DR 复制可以落后几秒钟或几分钟时间。 |
| 恢复时间目标 (RTO) | 完成恢复或还原所需的时间量。 | 由于 RPO 较大，备份解决方案需要处理的数据量通常更多，这会导致 RTO 较长。例如，根据从异地转送磁带所需的时间，从磁带还原数据可能需要数天的时间。 | 由于灾难恢复解决方案与源之间的同步程度更高，因此其 RTO 更小，需要处理的更改也更少。 |
| 保留 | 数据需要存储多久 | 对于需要进行操作恢复的情况（数据损坏、意外的文件删除、OS 故障），备份数据通常会保留 30 天或更短。<br>从合规性的角度来看，数据可能需要存储数月甚至数年。在这种情况下，备份数据非常适合存档。 | 灾难恢复只需操作性恢复数据，通常只需几个小时或最多一天的数据。由于 DR 解决方案中使用了精细数据捕获，因此不建议长期保留 DR 数据。 |

## 后续步骤

尝试使用简单易用的 Azure 备份。有关说明，请参阅以下教程之一：

- [尝试 Azure 备份](/documentation/articles/backup-try-azure-backup-in-10-mins/)
- [尝试 Azure VM 备份](/documentation/articles/backup-azure-vms-first-look/)

这些教程可帮助你快速备份，因此只说明了备份数据的最直接途径。有关可执行的备份类型的详细信息，请参阅：

- [Back up Windows machine（备份 Windows 计算机）](/documentation/articles/backup-configure-vault/)

- [Backup Azure IaaS VMs（备份 Azure IaaS VM）](/documentation/articles/backup-azure-vms-prepare/)


[green]: ./media/backup-introduction-to-azure-backup/green.png
[yellow]: ./media/backup-introduction-to-azure-backup/yellow.png
[red]: ./media/backup-introduction-to-azure-backup/red.png

<!---HONumber=Mooncake_0503_2016-->