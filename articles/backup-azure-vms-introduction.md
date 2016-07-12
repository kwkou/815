<properties
	pageTitle="在 Azure 中规划 VM 备份基础结构 | Azure"
	description="规划在 Azure 中备份虚拟机时的重要注意事项"
	services="backup"
	documentationCenter=""
	authors="markgalioto"
	manager="jwhit"
	editor=""
	keywords="备份 vm, 备份虚拟机"/>

<tags
	ms.service="backup"
	ms.date="02/12/2016"
	wacn.date="06/06/2016"/>

# 在 Azure 中计划 VM 备份基础结构
本文介绍在 Azure 中规划备份虚拟机时需要注意的重要事项。如果你已[准备好环境](/documentation/articles/backup-azure-vms-prepare/)，则在开始[备份 VM](/documentation/articles/backup-azure-vms/) 之前，下一步就是执行此操作。如果你需要有关 Azure 虚拟机的详细信息，请参阅[虚拟机文档](/documentation/services/virtual-machines/)。

## <a name="how-does-azure-back-up-virtual-machines"></a>Azure 虚拟机备份原理
当 Azure 备份服务在计划的时间启动备份作业时，它将触发进行时间点快照拍摄所需的备份扩展。创建此快照时，将借助卷影复制服务 (VSS) 来获取虚拟机中磁盘的一致性快照，不必关闭该虚拟机。

拍摄快照后，数据将由 Azure 备份服务传输到备份保管库中。为了使备份过程更加高效，服务只标识并传输自上次备份后已更改的数据块。

![Azure 虚拟机备份体系结构](./media/backup-azure-vms-introduction/vmbackup-architecture.png)

数据传输完成后，将会删除快照并创建恢复点。

### 数据一致性
备份和还原业务关键型数据十分复杂，因为需要在生成数据的应用程序仍在运行时进行备份。为了解决此问题，Azure 备份对 Microsoft 工作负荷进行应用程序一致性备份，使用 VSS 来确保将数据正确写入到存储中。

>[AZURE.NOTE]Linux 虚拟机只能使用文件一致性备份，因为 Linux 没有与 VSS 相当的平台。

Azure 备份将在 Windows VM 上创建 VSS 完全备份（阅读有关 [VSS 完全备份](http://blogs.technet.com/b/filecab/archive/2008/05/21/what-is-the-difference-between-vss-full-backup-and-vss-copy-backup-in-windows-server-2008.aspx)的详细信息）。若要启用 VSS 复制备份，需要在 VM 上设置以下注册表项。


		[HKEY_LOCAL_MACHINE\SOFTWARE\MICROSOFT\BCDRAGENT]
		"USEVSSCOPYBACKUP"="TRUE"



此表介绍了一致性类型，以及在 Azure VM 备份和还原过程中，这些一致性类型会在何种条件下出现。

| 一致性 | 基于 VSS | 解释和详细信息 |
|-------------|-----------|---------|
| 应用程序一致性 | 是 | 这是适合 Microsoft 工作负荷的理想的一致性类型，因为它可以确保：<ol><li>VM 启动。<li>不发生数据损坏。<li>不发生数据丢失。<li> 对于使用数据的应用程序，数据将保持一致，因为备份时会使用 VSS 将应用程序纳入考虑。</ol> 大多数 Microsoft 工作负荷都有 VSS 写入器，负责执行与数据一致性相关的工作负荷特定操作。例如，Microsoft SQL Server 的 VSS 写入器可确保正确写入事务日志文件和数据库。<br><br> 对于 Azure VM 备份，获取应用程序一致恢复点意味着备份扩展可以调用 VSS 工作流，并在获取 VM 快照之前*正确*完成。当然，这也意味着会调用 Azure VM 中所有应用程序的 VSS 写入器。<br><br>（了解 [VSS 的基础知识](http://blogs.technet.com/b/josebda/archive/2007/10/10/the-basics-of-the-volume-shadow-copy-service-vss.aspx)，并详细了解其[工作原理](https://technet.microsoft.com/library/cc785914%28v=ws.10%29.aspx)）。 |
| 文件系统一致性 | 是 - 对于基于 Windows 的计算机 | 在两种情况下，恢复点可以做到文件系统一致性：<ul><li>在 Azure 中备份 Linux VM，因为 Linux 没有相当于 VSS 的平台。<li>在 Azure 中备份 Windows VM 期间 VSS 失败。</li></ul> 在这两种情况下，最佳做法是确保：<ol><li>VM 启动。<li>不发生数据损坏。<li>不发生数据丢失。</ol> 应用程序需要对还原的数据实施自己的“修复”机制。|
| 崩溃一致性 | 否 | 这种情况相当于虚拟机“崩溃”（通过软重置或硬重置）。这通常发生于 Azure 虚拟机在备份期间关闭时。对于 Azure 虚拟机备份，获取崩溃一致恢复点意味着 Azure 备份不保证存储媒体上的数据一致 - 无论从操作系统还是应用程序的观点来说都一样。备份时已存在磁盘上的数据才被捕获和备份。<br/> <br/>尽管不会保证，但大多数情况下操作系统都会启动。随后通常是运行磁盘检查过程（如 chkdsk）以修复任何损坏错误。内存中任何未完全刷新到磁盘的数据或写入操作都将丢失。如果需要执行数据回滚，应用程序通常会接着执行其自身的验证机制。<br><br>例如，如果事务日志中的条目不在数据库中，则数据库软件将执行回滚，直到数据一致。当数据分散在多个虚拟磁盘上时（例如跨区卷），崩溃一致恢复点不保证数据的正确性。|


## 性能和资源利用率
与本地部署的备份软件一样，Azure 中 VM 的备份也需要在容量和资源利用率方面进行规划。[Azure 存储空间限制](/documentation/articles/azure-subscription-service-limits/#storage-limits)定义如何构建 VM 部署，以获得最大性能，同时对运行中的工作负荷造成最小的影响。

有两个主要 Azure 存储空间限制会影响备份性能：

- 每个存储帐户的最大传出
- 每个存储帐户的总请求速率

### 存储帐户限制
从客户存储帐户复制备份数据将会计入该存储帐户的每秒输入/输出操作数 (IOPS) 和出口（存储吞吐量）度量值。同时，虚拟机会运行并消耗 IOPS 与吞吐量。其目的是确保总流量（备份和虚拟机）不会超过存储帐户限制。

### 磁盘数目
备份过程具有贪婪性，为了尽快完成备份，它会试图尽量消耗掉它所能使用的资源。但是，单个 Blob 的目标吞吐量实施 I/O 操作总次数限制为每秒 60 MB。为了加快备份过程，将会尝试对 VM 的每个磁盘执行并行备份。因此，如果 VM 有四个磁盘，则 Azure 备份将尝试并行备份所有四个磁盘。确定从客户存储帐户传出流量的唯一重要因素是从存储帐户备份的**磁盘数**。

### 备份计划
影响性能的另一因素是**备份计划**。如果你将所有 VM 配置为同时备份，则要并行备份的磁盘数将会增加 - 因为 Azure 备份会尝试备份尽可能多的磁盘。因此，减少存储帐户备份流量的一种方法是确保在一天的不同时间备份不同的 VM，且备份时间不会重叠。

## 容量规划
通盘考虑所有这些因素意味着需要正确规划存储帐户的使用。请下载 [VM 备份容量规划 Excel 电子表格](https://gallery.technet.microsoft.com/Azure-Backup-Storage-a46d7e33)，以了解磁盘和备份计划选择所造成的影响。

### 备份吞吐量
对于每个要备份的磁盘，Azure 备份将读取磁盘上的块，并只存储更改的数据（增量备份）。下表显示了应该能够从 Azure 备份获得的平均吞吐量值。使用此表可以估算备份给定大小的磁盘所需的时间。

| 备份操作 | 最有利情况下的吞吐量 |
| ---------------- | ---------- |
| 初始备份 | 160 Mbps |
| 增量备份 (DR) | 640 Mbps <br><br> 如果需要备份的磁盘上发生大量离散变换，此吞吐量可能会显著下降。 |

### VM 备份总时间
尽管大部分备份时间花在读取和复制数据上，但其他一些操作也会影响备份 VM 所用的总时间：

- 花费在[安装或更新备份扩展](/documentation/articles/backup-azure-vms/#offline-vms)上的时间。
- 快照时间，即触发某个快照所花费的时间。在接近计划的备份时间时，将会触发快照。
- 队列等待时间。由于备份服务要处理来自多个客户的备份，可能不会立即将备份数据从快照复制到 Azure 备份保管库。在负载高峰期，由于要处理的备份数过多，等待时间可能会长达 8 小时。但是，每日备份策略规定的 VM 备份总时间不会超过 24 小时。

## 数据加密

在备份过程中，Azure 备份不会加密数据。但是，你可以在 VM 中加密数据，然后无缝备份保护的数据（阅读有关[备份加密数据](/documentation/articles/backup-azure-vms-encryption/)的详细信息）。


## 如何计算受保护的实例？
通过 Azure 备份进行备份的 Azure 虚拟机的收费依据 [Azure 备份定价](/home/features/back-up/pricing/)。受保护的实例计算基于虚拟机的*实际*大小，即虚拟机中除“资源磁盘”外的所有数据之和。

*不是*按连接到虚拟机的每个受支持数据磁盘的最大大小对你收费，而是按存储在数据磁盘中的实际数据收费。与此类似，备份存储空间的收费是根据通过 Azure 存储空间存储的数据容量，即每个恢复点中实际数据之和。

以 A2 标准大小的虚拟机为例，该虚拟机有两个额外的数据磁盘，总大小为 1TB。下表提供了每个这样的磁盘上存储的实际数据：

|磁盘类型|最大大小|实际存在的数据|
|---------|--------|------|
| 操作系统磁盘 | 1023 GB | 17 GB |
| 本地磁盘/资源磁盘 | 135 GB | 5 GB（不包括在备份中） |
| 数据磁盘 1 |	1023 GB | 30 GB |
| 数据磁盘 2 | 1023 GB | 0 GB |

此示例中，虚拟机的*实际*大小为 17 GB + 30 GB + 0 GB = 47 GB。这就是按月收费所基于的受保护实例大小。随着虚拟机中数据量的增长，用于计费的受保护实例大小也会相应变化。

完成第一次成功备份后才会计费。存储和受保护的实例也会在此同时开始计费。只要虚拟机包含 Azure 备份存储的任何备份数据，就会持续计费。如果保留备份数据，执行“停止保护”操作将不会停止计费。

指定的虚拟机只有在停止保护*和*删除全部备份数据后才会停止计费。没有活动的备份作业（已停止保护）时﹐受保护实例大小将根据上次成功备份时的虚拟机大小进行按月计费。

## 有疑问？
如果你有疑问，或者希望包含某种功能，请[给我们反馈](http://aka.ms/azurebackup_feedback)。

## 后续步骤

- [备份虚拟机](/documentation/articles/backup-azure-vms/)
- [管理虚拟机备份](/documentation/articles/backup-azure-manage-vms/)
- [恢复虚拟机](/documentation/articles/backup-azure-restore-vms/)
- [解决 VM 备份问题](/documentation/articles/backup-azure-vms-troubleshoot/)

<!---HONumber=Mooncake_0405_2016-->