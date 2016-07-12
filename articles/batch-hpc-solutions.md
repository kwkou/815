<properties
   pageTitle="云中的 Batch 和 HPC 解决方案 | Azure"
   description="介绍 Azure 中的批处理和高性能计算（HPC 和大型计算）应用方案和解决方案选项"
   services="batch, virtual-machines, cloud-services"
   documentationCenter=""
   authors="dlepow"
   manager="timlt"
   editor=""/>

<tags
   ms.service="batch"
   ms.date="04/21/2016"
   wacn.date="06/06/2016"/>

#  批处理( Batch )和 HPC 解决方案

Azure 提供高效且可伸缩的云解决方案来进行处理和高性能计算（HPC，又称为大型计算）。在此处了解大型计算工作负荷以及 Azure 提供的相应支持服务，或者直接跳至本文后面的[解决方案应用场景](#scenarios)。本文主要面向技术决策人、IT 经理和独立软件供应商，但其他 IT 专业人员和开发人员也可以使用它来熟悉这些解决方案。

组织面临着一些大规模计算问题，包括工程设计和分析、图像渲染、复杂建模、Monte Carlo 仿真和财务风险计算。为了帮助组织解决这些问题并在资源、缩放和计划方面做出决策，Azure 提供了可缩放、按需计算功能和服务。借助 Azure，组织可以：

* 创建混合式解决方案，扩展本地 HPC 群集，将高峰工作负荷卸载到云

* 完全在 Azure 中运行 HPC 群集工具和工作负荷

* 使用托管的可缩放 Azure 服务（例如[批处理](/documentation/services/batch/)）运行计算密集型工作负荷，而无需部署和管理计算基础结构

Azure 还为开发人员和合作伙伴提供一整套功能、体系结构选项和开发工具，用于构建大规模的自定义大型计算工作流，当然这超出了本文的范围。你还可以使用规模不断扩大的合作伙伴生态系统，以提高大型计算工作流在 Azure 云中的工作效率。


## 背景：批处理 ( Batch )和 HPC 应用程序

与 Web 应用程序和许多业务线应用程序不同，批处理 ( Batch ) 和 HPC 应用程序有确定的开始和结束时间，而且它们可以按照计划运行或按需运行。其中的大多数应用程序可分为两大类：“内在并行”（有时称为“高度并行”，因为它们解决的问题有助于使自身在多个计算机或处理器上并行运行）和“紧密耦合”。有关这些应用程序类型的详细信息，请参阅下表。某些 Azure 解决方案方法更适合一种类型或其他类型。

>[AZURE.NOTE]在批处理( Batch )和 HPC 解决方案中，应用程序的运行中实例通常称为*作业*，而每个作业可以划分成*任务*。应用程序的群集计算资源通常称为*计算节点*。

类型 | 特征 | 示例
------------- | ----------- | ---------------
**内在并行**<br/><br/>![内在并行][parallel] |• 各计算机独立运行应用程序逻辑<br/><br/> • 增加计算机使应用程序能够轻松扩展和缩短计算时间<br/><br/>• 应用程序由多个独立的可执行文件组成，或者分为一组由客户端调用的服务（一个面向服务的体系结构 (SOA)、应用程序） |• 金融风险建模<br/><br/>• 图像渲染和图像处理<br/><br/>• 媒体编码和转码<br/><br/>• Monte Carlo 仿真<br/><br/>• 软件测试
**紧密耦合**<br/><br/>![紧密耦合][coupled] |• 应用程序需要计算节点进行交互或者交换中间结果<br/><br/>• 计算节点可以使用消息传递接口 (MPI) 进行通信，MPI 是一种用于并行计算的常见通信协议<br/><br/>• 应用程序容易受到网络延迟和带宽的影响<br/><br/>• 可以使用支持 InfiniBand 等高速联网技术和远程直接内存访问 (RDMA) 等高速联网技术提高应用程序的性能 |• 油气储层建模<br/><br/>• 工程设计和分析，例如计算流体动力学<br/><br/>• 汽车碰撞及核反应等物理模拟<br/><br/>• 天气预报

### 在云中运行批处理( Batch )和 HPC 应用程序的注意事项

设计为运行在本地 HPC 群集中的许多应用程序随时可以迁移到 Azure 或混合（跨界）环境中。但是，可能有一些限制或注意事项，包括：


* **云资源的可用性** - 根据所使用的云计算资源类型，你可能无法在作业执行期间持续地使用计算机。状态处理和进度检查点是处理可能的暂时性故障的常见技巧，且在利用云资源时更有必要。


* **数据访问** - 企业网络群集内经常使用的数据访问技术（例如 NFS）可能需要云中的特殊配置，或者你可能需要为云采用不同的数据访问做法与模式。

* **数据移动** - 对于处理大量数据的应用程序，需要使用策略将数据移至云存储和计算资源，并可能需要高速跨界网络，例如 [Azure ExpressRoute](/services/expressroute/)。另外，还要考虑到法律、法规或政策对存储或访问该数据的限制。


* **许可** - 请向供应商咨询有关任何商业应用程序在云中的运行许可或其他限制。并非所有供应商都提供即付即用许可。你可能需要根据自己的解决方案，在云中规划许可证服务器，或连接到本地许可证服务器。


### 大型计算还是大数据？

大型计算和大数据应用程序之间的分界线并不总是很清晰，而且有些应用程序可能同时具有这两者的特征。两者通常都涉及在计算机群集上运行大规模计算。但解决方案方法和支持工具可能会不同。

• **大型计算**通常涉及依赖于 CPU 性能和内存的应用程序，例如工程仿真、金融风险建模和数字渲染。支持大型计算解决方案的群集可能包括配备有可执行原始计算的专用多核处理器的计算机，以及可连接这些计算机的专用高速联网硬件。

• **大数据**可解决数据分析问题，这些问题涉及无法由单个计算机或数据库管理系统管理的大量数据（例如大量 Web 日志或其他商业智能数据）。大数据往往更多地依赖于磁盘容量和 I/O 性能而不是 CPU 性能，通常使用 Apache Hadoop 等专用工具来管理群集以及对数据分区。（有关 Azure HDInsight 和其他 Azure Hadoop 解决方案的信息，请参阅 [Hadoop](https://azure.microsoft.com/en-us//solutions/hadoop/)。）

## 计算管理和作业计划

运行中的批处理( Batch ) 和 HPC 应用程序通常包括一个群集管理器和一个作业计划程序，可帮助管理群集计算资源并将其分配到运行作业的应用程序。可以通过单独的工具或者集成的工具或服务来实施这些功能。

* **群集管理器** - 设置、发布和管理计算资源（或计算节点）。群集管理器可以自动在计算节点上安装操作系统映像和应用程序，按需缩放计算资源，以及监视节点的性能。

* **作业计划程序** - 指定应用程序需要的资源（例如处理器或内存）及其运行时的条件。作业计划程序维护作业队列，并根据分配的优先级或其他特征向这些作业分配资源。

为基于 Windows 的群集和基于 Linux 的群集提供的群集工具与作业计划工具都可以顺利迁移到 Azure。例如，[Microsoft HPC Pack](https://technet.microsoft.com/library/cc514029)（Microsoft 推出的免费计算群集解决方案，适用于 Windows 和 Linux HPC 工作负荷）提供多种可在 Azure 中运行的选项。你还可以构建 Linux 群集，以便在 Azure 中运行开源工具（例如 Torque 和 SLURM），或者运行商业工具（例如 [TIBCO DataSynapse GridServer](http://www.tibco.com/products/automation/application-development/grid-computing/gridserver) 和 [Univa Grid Engine](http://www.univa.com/products/grid-engine)）。

如以下部分所示，你还可以利用 Azure 服务来管理计算资源和计划作业，不需要使用传统群集管理工具（也可以在使用传统群集管理工具的基础上进行该操作）。


## 方案

以下是利用现有 HPC 群集解决方案和/或 Azure 服务在 Azure 中运行大型计算工作负荷的三种常见方案。已列出选择每种方案时的重要注意事项（尚未详尽）。本文后面会详细介绍可以在解决方案中使用的 Azure 服务。

 | 方案 | 为什么选择它？
------------- | ----------- | ---------------
**让 HPC 群集迸发到 Azure**<br/><br/>[![群集迸发][burst_cluster]](./media/batch-hpc-solutions/burst_cluster.png) <br/><br/> 详细了解：<br/>• [使用 Microsoft HPC Pack 迸发到 Azure](https://technet.microsoft.com/library/gg481749.aspx)<br/><br/>• [使用 Microsoft HPC Pack 设置混合计算群集](/documentation/articles/cloud-services-setup-hybrid-hpcpack-cluster/)<br/><br/>|• 在混合解决方案中组合使用本地 [Microsoft HPC Pack](https://technet.microsoft.com/library/cc514029) 群集与其他 Azure 资源。<br/><br/>• 扩展大型计算工作负荷，以便在平台即服务 (PaaS) 虚拟机实例上运行（目前仅限 Windows Server）。<br/><br/>• 使用可选的 Azure 虚拟网络访问本地许可证服务器或数据存储|• 你已经有一个 HPC Pack 群集，需要更多资源 <br/><br/>• 你不想购买和管理更多的 HPC 群集基础结构<br/><br/>• 你的高峰需求时段很短暂，或者你的项目特殊
**完全在 Azure 中创建 HPC 群集**<br/><br/>[![IaaS 中的群集][iaas_cluster]](./media/batch-hpc-solutions/iaas_cluster.png)<br/><br/>详细了解：<br/>• [Azure 中的 HPC 群集解决方案](/documentation/articles/big-compute-resources/)<br/><br/>|• 在标准的或自定义的 Windows 或 Linux 基础结构即服务 (IaaS) 虚拟机上快速一致地部署应用程序和群集工具。<br/><br/>• 通过所选作业计划解决方案运行各种大型计算工作负荷。<br/><br/>• 使用其他 Azure 服务（包括联网和存储）来创建基于云的完整解决方案。 |• 你不想购买和管理更多的 Linux 或 Windows HPC 群集基础结构<br/><br/>• 你的高峰需求时段很短暂，或者你的项目特殊<br/><br/>• 你在一段时间内需要增加一个群集，但不想投资在部署该群集所需的计算机和空间上<br/><br/>• 你想卸载计算密集型应用程序，以便在云中完全以服务形式运行它
**将并行应用程序横向扩展到 Azure**<br/><br/>[![Azure 批处理][batch_proc]](./media/batch-hpc-solutions/batch_proc.png)<br/><br/>详细了解：<br/>• [Azure Batch 基础](/documentation/articles/batch-technical-overview/)<br/><br/>• [适用于 .NET 的 Azure Batch 库入门](/documentation/articles/batch-dotnet-get-started/)|• 使用 [Azure Batch](/documentation/services/batch/) API 进行开发，以便横向扩展各种能够在平台即服务 (PaaS) 虚拟机池上运行的大型计算工作负荷（目前仅限 Windows Server）。<br/><br/>• 使用 Azure 服务来管理虚拟机的部署和自动缩放、作业计划、灾难恢复、数据移动、依赖项管理以及应用程序部署 - 不需要单独的 HPC 群集或作业计划程序。|• 你不想管理计算资源或作业计划程序，只想专注于应用程序的运行<br/><br/>• 你希望卸载计算密集型应用程序，让其在云中以服务形式运行<br/><br/>• 你希望根据计算工作负荷自动缩放计算资源


## 用于大型计算的 Azure 服务

下面是有关计算、数据、网络和相关服务的详细信息，你可以将它们组合用于大型计算解决方案和工作流。有关 Azure 服务的深入指导，请参阅 Azure 服务[文档](/documentation/)。本文前面的[方案](#scenarios)仅显示了这些服务的部分使用方法。

>[AZURE.NOTE] Azure 会定期推出新的服务，这些服务可能适用于你的方案。如有疑问，请联系 [Azure 合作伙伴](https://pinpoint.microsoft.com/zh-cn/search?keyword=azure)或者向bigcompute@microsoft.com发送电子邮件。

### 计算服务

Azure 计算服务是大型计算解决方案的核心，不同的计算服务适用于不同的方案。在基本级别中，这些服务为使用 Windows Server Hyper-V 技术，由 Azure 提供的基于虚拟机的计算实例上运行的应用程序提供不同模式。这些实例可以运行各种标准的和自定义的 Linux 和 Windows 操作系统与工具。Azure 允许你选择[实例大小](/documentation/articles/virtual-machines-windows-sizes/)，对 CPU 核心、内存、磁盘容量和其他特征进行了不同的配置。你可以根据需要将实例扩展到数千个核心，并在需要较少的资源时相应减少。

服务 | 说明
------------- | -----------
**[云服务](/documentation/services/cloud-services)**<br/><br/> |• 可以在辅助角色实例中运行大型计算应用程序，辅助角色实例是运行某个 Windows Server 版本的虚拟机并且完全由 Azure 托管<br/><br/>• 可以较低的管理开销支持运行在平台即服务 (PaaS) 模型中的可缩放的可靠应用程序<br/><br/>• 可能需要额外的工具或开发来与现有的本地 HPC 群集解决方案进行集成
**[虚拟机](/documentation/services/virtual-machines)**<br/><br/> |• 使用 Microsoft Hyper-V 技术提供计算基础结构即服务 (IaaS)<br/><br/>• 使你能够从标准 Windows Server 或 Linux 映像，或者你提供的映像和数据磁盘灵活地设置和管理永久性虚拟机<br/><br/>• 完全在云中运行本地计算群集工具和应用程序
**[ 批处理( Batch )](/documentation/services/batch)**<br/><br/> |• 在完全托管的服务中运行大规模的并行与 批处理( Batch )工作负荷<br/><br/>• 针对虚拟机的托管池提供作业计划和自动缩放<br/><br/>• 允许开发人员构建自定义大型计算解决方案或支持云的现有应用程序<br/>

### 存储服务

大型计算解决方案通常会操作一组输入数据，然后生成其结果的数据。许多大型计算解决方案中使用的一些 Azure 存储空间服务包括：

* [Blob、表和队列存储](/documentation/services/storage) - 分别管理大量非结构化数据、NoSQL 数据，以及有关工作流和通信的消息。例如，你可以为大型技术数据集或应用程序处理的输入图像或媒体文件使用 Blob 存储。可以在解决方案中使用队列以进行异步通信。有关这些存储解决方案的详细信息，请参阅 [Azure 存储空间简介](/documentation/articles/storage-introduction/)。



### 数据和分析服务

某些大型计算方案涉及到大规模数据流，或者会生成需要进一步处理或分析的数据。为了应对这种情况，Azure 提供了许多数据和分析服务，包括：

* [SQL 数据库](/documentation/services/sql-databases) - 提供托管平台服务中 Microsoft SQL Server 关系数据库管理系统的主要功能。

* [HDInsight](/documentation/services/hdinsight) - 在云中部署和设置基于 Windows Server 或 Linux 的 Apache Hadoop 群集，用于管理、分析和报告具有高可靠性与可用性的大数据。


### 其他服务

大型计算解决方案可能需要包含其他 Azure 基础结构和平台服务，才能连接到本地或其他环境中的资源。示例包括：

* [虚拟网络](/documentation/services/networking) - 在 Azure 中创建逻辑隔离的区段，以使用 IPSec 将 Azure 资源连接到本地数据中心或单个客户端计算机；可让大型计算应用程序访问本地数据、Active Directory 服务和许可证服务器

* [服务总线](/documentation/services/service-bus) - 提供多种机制让应用程序进行通信或交换数据，无论这些应用程序位于 Azure、另一个云平台还是数据中心。

## 后续步骤

* 如需有关你解决方案的技术指导，请参阅[批处理 ( Batch )和 HPC 的技术资源](/documentation/articles/big-compute-resources/)。

* 与 Cycle Computing 和 UberCloud 等合作伙伴讨论 Azure 选项。

* 了解 [Towers Watson](https://customers.microsoft.com/Pages/CustomerStory.aspx?recid=18222)、[Altair](https://azure.microsoft.com/blog/availability-of-altair-radioss-rdma-on-microsoft-azure/) 和 [d3VIEW](https://customers.microsoft.com/Pages/CustomerStory.aspx?recid=22088) 提供的 Azure 大型计算解决方案。

* 有关最新通告，请参阅 [Microsoft HPC 和批处理团队博客](http://blogs.technet.com/b/windowshpc/)与 [Azure 博客](https://azure.microsoft.com/blog/tag/hpc/)。

<!--Image references-->
[parallel]: ./media/batch-hpc-solutions/parallel.png
[coupled]: ./media/batch-hpc-solutions/coupled.png
[iaas_cluster]: ./media/batch-hpc-solutions/iaas_cluster.png
[burst_cluster]: ./media/batch-hpc-solutions/burst_cluster.png
[batch_proc]: ./media/batch-hpc-solutions/batch_proc.png

<!---HONumber=Mooncake_0530_2016-->