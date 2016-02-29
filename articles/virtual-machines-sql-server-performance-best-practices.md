<properties 
	pageTitle="SQL Server 的性能最佳实践 | Microsoft Azure"
	description="提供有关优化 Microsoft Azure VM 中的 SQL Server 性能的最佳实践。"
	services="virtual-machines"
	documentationCenter="na"
	authors="rothja"
	manager="jeffreyg"
	editor="monicar" 
	tags="azure-service-management" />
	
<tags
	ms.service="virtual-machines"
	ms.date="12/22/2015"
	wacn.date=""/>

# SQL Server 在 Azure 虚拟机中的性能最佳实践

## 概述

本主题提供有关优化 SQL Server 在 Microsoft Azure 虚拟机中性能的最佳实践。在 Azure 虚拟机中运行 SQL Server 时，我们建议你继续使用适用于 SQL Server 在本地服务器环境中的相同数据库性能优化选项。但是，关系数据库在公有云中的性能取决于许多因素，如虚拟机的大小和数据磁盘的配置。

创建 SQL Server 映像时，请考虑使用 Azure 管理门户来利用某些功能，如默认使用高级存储和其他选项（如自动修补、自动备份和 AlwaysOn 配置）。

本白皮书重点介绍获得 SQL Server 在 Azure VM 上的最佳性能。如果你的工作负荷要求较低，可能不需要下面列出的每项优化。评估这些建议时应考虑性能需求和工作负荷模式。

[AZURE.INCLUDE [了解部署模型](../includes/learn-about-deployment-models-both-include.md)]

## 快速检查列表

下面是 SQL Server 在 Azure 虚拟机中的优化性能快速检查列表：

|区域|优化|
|---|---|
|**VM 大小**|SQL Enterprise 版本为 [DS3](/documentation/articles/virtual-machines-size-specs#standard-tier-ds-series) 或更高。SQL 标准版和 Web 版本为 <br/><br/>[DS2](/documentation/articles/virtual-machines-size-specs#standard-tier-ds-series) 或更高。|
|**存储**|使用[高级存储](/documentation/articles/storage-premium-storage-preview-portal)。<br/><br/>使[存储帐户](/documentation/articles/storage-create-storage-account)和 SQL Server VM 保存在同一个区域中。<br/><br/>禁用存储帐户上的 Azure [异地冗余存储](/documentation/articles/storage-redundancy)（异地复制）。|
|**磁盘**|至少使用 2 个 [P30 磁盘](/documentation/articles/storage-premium-storage-preview-portal#scalability-and-performance-targets-when-using-premium-storage)（1 个用于日志文件；1 个用于数据文件和 TempDB）。<br/><br/>避免将操作系统磁盘或临时磁盘用于数据库存储或日志记录。<br/><br/>在托管数据文件和 TempDB 的磁盘上启用读缓存。<br/><br/>请勿在托管日志文件的磁盘上启用缓存。<br/><br/>条带化多个 Azure 数据磁盘以获得更高的 IO 吞吐量。<br/><br/>使用文档中记录的分配大小格式化。|
|**I/O**|启用数据库页压缩。<br/><br/>为数据文件启用即时文件初始化。<br/><br/>限制或禁用数据库的自动增长。<br/><br/>禁用数据库的自动收缩。<br/><br/>将所有数据库都移到数据磁盘，包括系统数据库。<br/><br/>将 SQL Server 错误日志和跟踪文件目录移至数据磁盘。<br/><br/>设置默认的备份和数据库文件的位置。<br/><br/>启用锁定的页。<br/><br/>应用 SQL Server 性能修复程序。|
|**特定于功能**|直接备份到 blob 存储。|

有关详细信息，请遵循以下小节中提供的指导原则。

## 虚拟机大小和存储帐户注意事项

对于性能敏感型应用程序，建议你使用以下虚拟机大小：

- **SQL Server Enterprise 版本**：DS3 或更高

- **SQL Server Standard 和 Web Edition**：DS2 或更高

有关支持的虚拟机大小的最新信息，请参阅[虚拟机大小](/documentation/articles/virtual-machines-size-specs)。

此外，我们建议你创建 Azure 存储帐户与 SQL Server 虚拟机在同一数据中心中，以减小传输延迟。创建存储帐户时应禁用异地复制，因为无法保证在多个磁盘上的写入顺序一致。相反，请考虑在两个 Azure 数据中心之间配置一个 SQL Server 灾难恢复技术。有关详细信息，请参阅 [Azure 虚拟机中 SQL Server 的高可用性和灾难恢复](/documentation/articles/virtual-machines-sql-server-high-availability-and-disaster-recovery-solutions)。

## 磁盘和性能注意事项

当创建 Azure 虚拟机时，该平台至少将一个磁盘附加到 VM 作为操作系统磁盘。此磁盘是一个 VHD，在存储空间中存储为一个页 blob。你还可以将其他磁盘作为数据磁盘附加到虚拟机，这些磁盘将在存储空间中存储为页 blob。在 Azure 虚拟机中还存在另一个称为临时磁盘的磁盘。这是可用于暂存空间的节点上的一个磁盘。

### 操作系统磁盘

操作系统磁盘是你可以作为操作系统的运行版本来启动和装载的 VHD，并且标记为 **C** 驱动器。

操作系统磁盘上的默认缓存策略是**读/写**。对于性能敏感型应用程序，我们建议使用数据磁盘而不是操作系统磁盘。请参阅下面有关数据磁盘的部分。

### 临时磁盘

临时存储驱动器，标记为 **D**: 驱动器，不会保留到 Azure blob 存储区。在 **D**: 驱动器上不会存储任何数据或日志文件。

使用 D 系列或 G 系列虚拟机 (VM) 时，仅将 TempDB 和/或缓冲池扩展存储在 **D** 驱动器上。与其他 VM 序列不同，D 系列和 G 系 VM 中的 **D** 驱动器是基于 SSD 的。如果工作负载大量使用临时对象或具有不适合存储在内存中的工作集，这可以提高其性能。有关详细信息，请参阅[在 Azure VM 中使用 SSD 来存储 SQL Server TempDB 和缓冲池扩展](http://blogs.technet.com/b/dataplatforminsider/archive/2014/09/25/using-ssds-in-azure-vms-to-store-sql-server-tempdb-and-buffer-pool-extensions.aspx)。

对于支持高级存储的 VM，我们建议将 TempDB 存储在支持启用了读缓存的高级存储的磁盘上。

### 数据磁盘

- **用于数据和日志文件的数据磁盘数目**：至少使用 2 个 [P30 磁盘](/documentation/articles/storage-premium-storage-preview-portal#scalability-and-performance-targets-when-using-premium-storage)，其中一个磁盘包含日志文件，另一个包含数据文件和 TempDB。要实现更大的吞吐量，你可能需要更多数据磁盘。若要确定数据磁盘数，你需要分析可用于数据和日志磁盘的 IOPS 数。要获取该信息，请参阅以下文章中有关每个 [VM 大小](/documentation/articles/virtual-machines-size-specs)和磁盘大小的 IOPS 的表：[使用磁盘的高级存储](/documentation/articles/storage-premium-storage-preview-portal)。如果需要更多带宽，则可以使用磁盘条带化附加更多的磁盘。如果使用的不是高级存储，建议添加 [VM 大小](/documentation/articles/virtual-machines-size-specs)支持的最大数量的数据磁盘并使用磁盘条带化。有关磁盘条带化的详细信息，请参阅下面的相关部分。

- **缓存策略**：仅在托管你的数据文件和 TempDB 的数据磁盘上启用读缓存。如果使用的不是高级存储，不要在任何数据磁盘上启用任何缓存。有关配置磁盘缓存的说明，请参阅以下主题：[Set-AzureOSDisk](https://msdn.microsoft.com/zh-cn/library/azure/jj152847) 和 [Set-AzureDataDisk](https://msdn.microsoft.com/zh-cn/library/azure/jj152851.aspx)。

- **NTFS 分配单元大小**：当格式化数据磁盘时，建议你为数据和日志文件以及 TempDB 使用 64-KB 分配单元大小。

- **磁盘条带化**：我们建议你遵循以下准则：

	- 对于 Windows 8/Windows Server 2012 或更高版本，使用[存储空间](https://technet.microsoft.com/zh-cn/library/hh831739.aspx)。对于 OLTP 工作负荷，将条带大小设置为 64 KB，对于数据仓库工作负荷，将条带大小设置为 256 KB，以避免分区定位错误导致的性能影响。此外，设置卷计数 = 物理磁盘的数量。若要配置具有 8 个以上磁盘的存储空间，必须使用 PowerShell（而不是服务器管理器 UI）来显式设置卷数以匹配磁盘数。有关如何配置[存储空间](https://technet.microsoft.com/zh-cn/library/hh831739.aspx)的详细信息，请参阅 [Windows PowerShell 中的存储空间 Cmdlet](https://technet.microsoft.com/zh-cn/library/jj851254.aspx)
	
	- 对于 Windows 2008 R2 或更早版本，你可以使用动态磁盘（操作系统条带化卷），条带大小始终为 64 KB。请注意，从 Windows 8/Windows Server 2012 开始不推荐使用此选项。有关信息，请参阅[虚拟磁盘服务正在过渡到 Windows 存储管理 API](https://msdn.microsoft.com/zh-cn/library/windows/desktop/hh848071.aspx) 中的支持声明。
	
	- 如果你的工作负荷不是日志密集型的并且不需要专用的 IOPS，你可以只配置一个存储池。否则，请创建两个存储池，一个用于日志文件，另一个用于数据文件和 TempDB。根据负载预期确定与每个存储池相关联的磁盘数。请记住，不同的 VM 大小允许不同数量的附加数据磁盘。有关详细信息，请参阅[虚拟机的大小](/documentation/articles/virtual-machines-size-specs)。

## I/O 性能注意事项

- 当你并行化你的应用程序和请求时可实现使用高级存储的最佳结果。高级存储专为 IO 队列深度大于 1 的方案设计，因此对于单线程串行请求（即使它们是存储密集型的），你将不会看到明显的性能提升。例如，这会影响性能分析工具（如 SQLIO）的单线程测试结果。

- 请考虑使用[数据库页压缩](https://msdn.microsoft.com/zh-cn/library/cc280449.aspx)，因为这有助于提高 I/O 密集型工作负荷的性能。但是，数据压缩可能会增加数据库服务器上的 CPU 消耗。

- 请考虑在传入/传出 Azure 时压缩所有数据文件。

- 请考虑启用即时文件初始化以减少初始文件分配所需的时间。若要利用即时文件初始化，请将 SE\_MANAGE\_VOLUME\_NAME 授予 SQL Server (MSSQLSERVER) 服务帐户并将其添加到**执行卷维护任务**安全策略。如果使用的是用于 Azure 的 SQL Server 平台映像，默认服务帐户 (NT Service\\MSSQLSERVER) 不会添加到**执行卷维护任务**安全策略。换而言之，在 SQL Server Azure 平台映像中不启用即时文件初始化。将 SQL Server 服务帐户添加到**执行卷维护任务**安全策略后，请重新启动 SQL Server 服务。使用此功能可能有一些安全注意事项。有关详细信息，请参阅[数据库文件初始化](https://msdn.microsoft.com/zh-cn/library/ms175935.aspx)。

- **自动增长**被视为只是非预期增长的偶发情况。自动增长不管理你的数据和记录每天的增长。如果使用自动增长，请使用大小开关预先增长文件。

- 请确保禁用**自动收缩**以避免可能对性能产生负面影响的不必要开销。

- 如果你运行的是 SQL Server 2012，安装 Service Pack 1 Cumulative Update 10。此更新包含在 SQL Server 2012 中执行“select into temporary table”语句时出现的 I/O 性能不良的修补程序。有关信息，请参阅此[知识库文章](http://support.microsoft.com/kb/2958012)。

- 建立锁定的页以减少 IO 和任何分页活动。

## 功能特定的性能注意事项

某些部署可以使用更高级的配置技术，获得更多的性能好处。下面的列表主要介绍可帮助你实现更佳性能的一些 SQL Server 功能：

- **备份到 Azure 存储空间**：为在 Azure 虚拟机中运行的 SQL Server 执行备份时，可以使用 [SQL Server 备份到 URL](https://msdn.microsoft.com/zh-cn/library/dn435916.aspx)。此功能是从 SQL Server 2012 SP1 CU2 开始提供的，在备份到附加数据磁盘时建议使用。当你备份至 Azure 存储空间或从中还原时，请按照 [SQL Server 备份到 URL 最佳实践和故障排除以及从 Azure 存储空间中存储的备份还原](https://msdn.microsoft.com/zh-cn/library/jj919149.aspx)中提供的建议操作。此外还可以使用 [Azure 虚拟机中 SQL Server 的自动备份](/documentation/articles/virtual-machines-sql-server-automated-backup)自动执行这些备份。

	对于 SQL Server 2012 以前版本，可以使用 [SQL Server 备份到 Azure 工具](https://www.microsoft.com/download/details.aspx?id=40740)。此工具可以通过使用多个备份条带目标帮助提高备份吞吐量。

- **Azure 中的 SQL Server 数据文件**：[Azure 中的 SQL Server 数据文件](https://msdn.microsoft.com/zh-cn/library/dn385720.aspx)这一新功能从 SQL Server 2014 开始提供。在 SQL Server 上运行 Azure 中的数据文件，与使用 Azure 数据磁盘时的性能特征相当。

## 后续步骤

如果你有兴趣更深入地了解 SQL Server 和高级存储，请参阅文章[将 Azure 高级存储用于虚拟机上的 SQL Server](/documentation/articles/virtual-machines-sql-server-use-premium-storage)。

有关安全最佳实践，请参阅 [Azure 虚拟机中 SQL Server 的安全注意事项](/documentation/articles/virtual-machines-sql-server-security-considerations)。

查看 [Azure 虚拟机上的 SQL Server 概述](/documentation/articles/virtual-machines-sql-server-infrastructure-services)中的其他 SQL Server 虚拟机主题。

<!---HONumber=Mooncake_0215_2016-->