<properties
   pageTitle="云中的 批处理( Batch )和 HPC 解决方案 | Windows Azure"
   description="介绍 Azure 中的 批处理( Batch )和高性能计算（大型计算）应用方案和解决方案选项"
   services="batch, virtual-machines, cloud-services"
   documentationCenter=""
   authors="dlepow"
   manager="timlt"
   editor=""/>

<tags
   ms.service="batch"
   ms.date="07/01/2015"
   wacn.date="10/30/2015"/>

#  批处理( Batch )和 HPC 解决方案

本文介绍 批处理( Batch )和高性能计算（HPC，又称为*大型计算*）的 Azure 解决方案。尽管本文主要面向技术决策人、IT 经理和独立软件供应商，但其他 IT 专业人员和开发人员也可以使用它来熟悉这些解决方案。

组织面临着一些大规模计算问题，包括工程设计和分析、图像渲染、复杂建模、Monte Carlo 仿真和财务风险计算。为了帮助组织解决这些问题并在资源、缩放和计划方面做出决策，Azure 提供了可缩放、按需计算功能和服务。借助 Azure，组织可以：

* 创建混合式解决方案，扩展本地 HPC 群集，将高峰工作负荷卸载到云

* 使用现有工具集和应用程序，完全在 Azure 中运行 HPC 群集工作负荷

* 使用托管的可缩放 Azure 服务（例如[批处理( Batch )](/documentation/services/batch/)）运行计算密集型工作负荷，而无需部署和管理计算基础结构

Azure 还为组织和软件供应商提供开发工具与服务，用于构建定制的端到端大数据解决方案。


## 背景： 批处理( Batch )和 HPC 应用程序

与 Web 应用程序和许多业务线应用程序不同， 批处理( Batch )和 HPC 应用程序有确定的开始和结束时间，而且它们可以按照计划运行或按需运行。其中的大多数应用程序可分为两大类：*内在并行*（有时称为“高度并行”，因为它们解决的问题有助于使自身在多个计算机或处理器上并行运行）和*紧密耦合*。有关这些应用程序类型的详细信息，请参阅下表。某些 Azure 解决方案方法更适合一种类型或其他类型。

>[AZURE.NOTE]在 批处理( Batch )和 HPC 解决方案中，应用程序的运行中实例通常称为*作业*，而每个作业可以划分成*任务*。应用程序的群集计算资源通常称为*计算节点*。

类型 | 特征 | 示例
------------- | ----------- | ---------------
**内在并行**<br/><br/>![内在并行][parallel] |• 各计算机独立运行应用程序逻辑<br/><br/> • 增加计算机使应用程序能够轻松扩展和缩短计算时间<br/><br/>• 应用程序由多个独立的可执行文件组成，或者分为一组由客户端调用的服务（一个面向服务的体系结构 (SOA)、应用程序） |• 金融风险建模<br/><br/>• 图像渲染和图像处理<br/><br/>• 媒体编码和转码<br/><br/>• Monte Carlo 仿真<br/><br/>• 软件测试
**紧密耦合**<br/><br/>![紧密耦合][coupled] |• 应用程序需要计算节点进行交互或者交换中间结果<br/><br/>• 计算节点可以使用消息传递接口 (MPI) 进行通信，MPI 是一种用于并行计算的常见通信协议<br/><br/>• 应用程序容易受到网络延迟和带宽的影响<br/><br/>• 可以使用支持 InfiniBand 等高速联网技术和远程直接内存访问 (RDMA) 等高速联网技术提高应用程序的性能 |• 油气储层建模<br/><br/>• 工程设计和分析，例如计算流体动力学<br/><br/>• 汽车碰撞及核反应等物理模拟<br/><br/>• 天气预报

### 在云中运行批处理( Batch )和 HPC 应用程序的注意事项

设计为运行在本地 HPC 群集中的许多应用程序随时可以迁移到 Azure 或混合（跨界）环境中。但是，可能有一些限制或注意事项，包括：


* **云资源的无中断可用性** - 根据针对解决方案选择的云计算资源类型，你可能无法在作业执行持续时间内保证计算机连续工作。状态处理和进度检查点是处理可能的暂时性故障的常见技巧，且在利用云资源时更有必要。


* **数据访问** - 企业网络群集内经常使用的数据访问技术（例如 NFS）可能需要云中的特殊配置，或者你可能需要为云采用不同的数据访问做法与模式。

* **数据移动** - 对于处理大量数据的应用程序，需要使用策略将数据移至云存储和计算资源，并可能需要考虑部署高速跨界网络。另外，还要考虑到法律、法规或政策对存储或访问该数据的限制。


* **许可** - 请向供应商咨询有关任何商业应用程序在云中的运行许可或其他限制。并非所有供应商都提供即付即用许可。你可能需要根据自己的解决方案，在云中规划许可服务器，或连接到本地许可服务器。


### 大型计算还是大数据？

大型计算和大数据应用程序之间的分界线并不总是很清晰，而且有些应用程序可能同时具有这两者的特征。两者通常都涉及在运行在本地和/或云中的计算机群集上运行大规模计算。但解决方案方法和支持工具可能会不同。

• **大型计算**通常涉及依赖于 CPU 性能和内存的应用程序，例如工程仿真、金融风险建模和数字渲染。支持大型计算解决方案的群集可能包括配备有可执行原始计算的专用多核处理器的计算机，以及可连接这些计算机的专用高速联网硬件。

• **大数据**可解决数据分析问题，这些问题涉及无法由单个计算机或数据库管理系统管理的大量数据（例如大量 Web 日志或其他商业智能数据）。大数据往往更多地依赖于磁盘容量和 I/O 性能而不是 CPU 性能，大数据解决方案通常包括 Apache Hadoop 等专用工具，可管理群集以及对数据进行分区。<!-- (For information about Azure HDInsight and other Azure Hadoop solutions, see [Hadoop](http://azure.microsoft.com/solutions/hadoop/).)-->

## 资源管理和作业计划

运行中的批处理( Batch )和 HPC 应用程序通常包括一个*群集管理器*和一个*作业计划程序*，可帮助管理群集计算资源并将其分配到运行作业的应用程序。可以通过单独的工具或者一个集成工具来实施这些功能。

* **群集管理器** - 设置、发布和管理计算资源（或计算节点）。根据不同的工具，群集管理器可以自动在计算节点上安装操作系统映像和应用程序，按需缩放计算资源，以及监视节点的性能。

* **作业计划程序** - 指定应用程序需要的资源（例如处理器或内存）及其运行时的条件。作业计划程序维护作业队列，并根据分配的优先级或其他特征向这些作业分配资源。

为基于 Windows 和 Linux 的群集提供的现成群集组建与作业计划工具，或者独立开发的那些工具都可以顺利迁移到 Azure。例如，[Microsoft HPC Pack](https://technet.microsoft.com/zh-cn/library/cc514029) 是 Microsoft 发布的面向 Windows Server 和 Windows 计算机的免费计算群集解决方案。若要减少对专用本地计算资源的需求，你可以根据需要扩展 HPC Pack 群集以使用 Azure 头节点，或者将整个群集部署在 Azure 虚拟机中。

### 大型计算工作流

群集管理和作业计划工具可帮助你使用可预测的步骤来完成大型计算工作流。根据具体的解决方案，你可以省略以下列表中的某些步骤，或引入其他定制工具。数据密集型工作流可能需要额外的数据前期和后期处理步骤（未列出）。

1. **设置** - 准备包含所需基础结构、计算资源和存储的计算环境，以运行应用程序

2. **过渡** - 将输入数据和应用程序转移到计算环境

3. **计划** - 配置作业和任务，并向其分配可用的计算资源

4. **监视** - 提供有关作业和任务进度的状态信息；处理错误或异常

5. **完成** - 返回结果，并视需要对输出数据进行后处理

## 用于大型计算的 Azure 服务

Azure 提供一系列的计算、数据、网络和相关服务，你可将其用于大型计算解决方案和工作流。有关其中每项服务的深入指导，请参阅 Azure 服务文档。有关涉及 批处理( Batch )和 HPC 应用程序的一些常见方法，请参阅本文中的[解决方案应用方案](#solution-scenarios)。

>[AZURE.NOTE]新服务会定期导入 Azure 平台，可能适用于你的方案。建议仅将预览版服务用于测试或概念认证部署，而不要用于非生产任务负荷。如有疑问，请联系 [Azure 合作伙伴](https://pinpoint.microsoft.com/zh-CN/search?keyword=azure)或者向 *bigcompute@microsoft.com* 发送电子邮件。

### 计算服务

Azure 中的计算服务是大计算解决方案的核心。下表是经常使用的计算服务，它们能够为各种方案带来优势。在基本级别中，这些服务为使用 Windows Server Hyper-V 技术，由 Azure 提供的基于虚拟机的计算实例上运行的应用程序提供不同模式。根据具体的服务，这些实例可以运行各种标准和自定义的 Linux 和 Windows 操作系统与工具。Azure 提供[各种实例大小](/documentation/articles/virtual-machines-sizes-specs)，对 CPU 核心、内存、磁盘容量和其他特征进行了不同的配置。你可以根据需要将实例扩展到数千个核心，并在需要较少的资源时相应减少。

>[AZURE.NOTE]你可以利用 A8-A11 实例改善某些大型计算任务负荷的性能，包括需要低延迟、高吞吐量应用程序网络的并行 MPI 应用程序。请参阅[关于 A8、A9、A10 和 A11 计算密集型实例](/documentation/articles/virtual-machines-a8-a9-a10-a11-specs)。

服务 | 说明
------------- | -----------
**[云服务](http://www.windowsazure.cn/documentation/services/cloud-services)**<br/><br/> |• 可以在辅助角色实例中运行大型计算应用程序，辅助角色实例是运行某个 Windows Server 版本的虚拟机并且完全由 Azure 托管<br/><br/>• 可以较低的管理开销支持运行在平台即服务 (PaaS) 模型中的可缩放的可靠应用程序<br/><br/>• 可能需要额外的工具或开发来与现有的本地 HPC 群集解决方案进行集成
**[虚拟机](http://www.windowsazure.cn/documentation/services/virtual-machines)**<br/><br/> |• 使用 Microsoft Hyper-V 技术提供计算基础结构即服务 (IaaS)<br/><br/>• 使你能够从标准 Windows Server 或 Linux 映像，或者你提供的映像和数据磁盘灵活地设置和管理永久性虚拟机<br/><br/>• 完全在云中运行本地计算群集工具和应用程序
**[ 批处理( Batch )](http://www.windowsazure.cn/documentation/services/batch)**<br/><br/> |• 在完全托管的服务中运行大规模的并行与 批处理( Batch )工作负荷，例如图像渲染及媒体编码和转码<br/><br/>• 针对虚拟机的托管池提供作业计划和自动缩放<br/><br/>• 允许开发人员构建自定义大型计算解决方案或支持云的现有应用程序<br/>

### 存储服务

大型计算解决方案通常会操作一组输入数据，然后生成其结果的数据。许多大型计算解决方案中使用的一些 Azure 存储空间服务包括：

* [Blob、表和队列存储](/documentation/services/storage) - 分别管理大量非结构化数据、NoSQL 数据，以及有关工作流和通信的消息。例如，你可以为大型技术数据集或应用程序处理的输入图像或媒体文件使用 Blob 存储。可以在解决方案中使用队列以进行异步通信。有关这些存储解决方案的详细信息，请参阅 [Microsoft Azure 存储空间简介](/documentation/articles/storage-introduction)。

* Azure 文件存储 - 在 Azure 中使用标准 SMB 通信协议（某些 HPC 群集解决方案需要这种协议）来共享公用文件和数据。

### 数据和分析服务

某些大型计算方案涉及到大规模数据流，或者会生成需要进一步处理或分析的数据。为了应对这种情况，Azure 提供了许多数据和分析服务，包括：

<!--* [Data Factory](/documentation/services/data-factory) - Builds data-driven workflows (pipelines) that join, aggregate, and transform data sourced from on-premises, cloud-based, and Internet data stores.-->

* [SQL Database](/documentation/services/sql-databases) - 提供托管平台服务中 Microsoft SQL Server 关系数据库管理系统的主要功能。

* [HDInsight](/documentation/services/hdinsight) - 在云中部署和设置基于 Windows Server 或 Linux 的 Apache Hadoop 群集，用于管理、分析和报告具有高可靠性与可用性的大数据。

<!--* [Machine Learning](/documentation/services/machine-learning) - Helps you create, test, operate, and manage predictive analytic solutions in a fully managed platform service.-->

### 其他服务

大型计算解决方案可能需要包含其他 Azure 基础结构和平台服务，才能连接到本地或其他环境中的资源。示例包括：

* [虚拟网络](/documentation/services/virtual-network) - 在 Azure 中创建逻辑隔离的区段，以使用 IPSec 将 Azure 资源连接到本地数据中心或单个客户端计算机；可让大型计算应用程序访问本地数据、Active Directory 服务和许可证服务器

<!--* [ExpressRoute](/documentation/services/expressroute) - Creates a private connection between Microsoft data centers and infrastructure that’s on-premises or in a co-location environment, with higher security, more reliability, faster speeds, and lower latencies than typical connections over the Internet.-->

* [服务总线](/documentation/services/service-bus) - 提供多种机制让应用程序进行通信或交换数据，无论这些应用程序位于 Azure、另一个云平台还是数据中心。

## 解决方案应用方案

以下是利用现有 HPC 群集解决方案和/或 Azure 服务在 Azure 中运行大型计算工作负荷的常见方案。

>[AZURE.NOTE]对于使用不适合标准 HPC 群集工具的计算密集型工作负荷，或不直接迁移到 Azure 服务的组织，Azure 为开发人员和合作伙伴提供了整套计算功能、服务、体系结构选项和开发工具。Azure 支持可扩展到数千个计算核心的自定义大型计算工作流。

### 方案 1.迸发到 Azure 的本地 HPC 群集

**何时选择此方案？**- 你可能已有运行计算密集型工作负荷的本地 HPC 群集，但它在高峰期（例如月末报告或开发特殊项目时）需要额外的计算资源。你不需要购买、部署和管理在大部分情况下可能处于闲置状态的额外软硬件，就可以使用 Azure 将按需计算功能添加到现有群集。

例如，如果现有本地 HPC 群集是使用 [Microsoft HPC Pack](https://technet.microsoft.com/zh-cn/library/cc514029) 构建的，则你可以使用在云服务中运行的 Azure 辅助角色实例形式添加额外的计算资源。请参阅下图。有关详细信息和分步说明，请参阅[使用 Microsoft HPC Pack 迸发到 Azure](https://technet.microsoft.com/zh-cn/library/gg481749.aspx)。

![群集迸发][burst_cluster]

>[AZURE.NOTE]如果你想要将 HPC Pack 群集占用率降到最低，可以将本地群集减少到只剩 HPC Pack 头节点。然后，在 Azure 中按需添加所有计算资源。有关逐步讲解此方案的教程，请参阅[使用 Microsoft HPC Pack 设置混合计算群集](/documentation/articles/cloud-services-setup-hybrid-hpcpack-cluster)。

此混合解决方案将利用本地群集的现有投资，但可让你为典型（非高峰）工作负荷缩放固定的本地基础结构。如果需要访问本地许可证服务器或数据存储，你可以设置 Azure 虚拟网络，将本地群集连接到 Azure。

### 方案 2.完全在 Azure 中创建 HPC 群集

**何时选择此方案？**- 你可能对本地 Windows 或 Linux HPC 群集做了大笔投资，包括管理和计划工具及定制的应用程序。你可能需要额外的群集容量来运行现有工具集和应用程序，却又不希望额外投资本地基础结构或修改应用程序。或者，你可能需要创建短期或长期使用的新群集，但不想要支付采购、维护及运营成本，或分配空间来安装及部署该群集。

你可以使用 Azure 自动化工具在 Azure 虚拟机中创建 HPC 群集，以提供所需的容量。根据部署的 VM，你可以运行各种 HPC 和并行工作负荷，包括紧密耦合的 MPI 应用程序。

>[AZURE.NOTE]请与你的本地群集解决方案和应用程序的供应商核实在提供基础结构即服务 (IaaS) 的公有云中运行的其他要求和最佳实践。

例如，你可以在 Azure 基础结构服务虚拟机 (IaaS) 中，使用 [Microsoft HPC Pack](https://technet.microsoft.com/zh-cn/library/cc514029) 创建基于 Windows Server 的 HPC 群集来运行工作负荷，如以下简图中所示。群集用户可通过客户端计算机上运行的标准 HPC Pack 作业提交工具，将作业安全地提交到云群集。有关详细信息和部署选项，请参阅 [Azure VM 中的 Microsoft HPC Pack](https://msdn.microsoft.com/zh-cn/library/azure/dn518135.aspx)。

![IaaS 中的群集][iaas_cluster]

**自动化部署** - 若要部署大量 Windows Server 或 Linux VM，可以使用标准或自定义的 VM 映像与 Azure 自动化工具，例如 [Azure 命令行界面](/documentation/articles/xplat-cli)或 [Azure PowerShell](/documentation/articles/powershell-install-configure)。示例包括：

* 若要在 Azure 基础结构服务中部署 HPC Pack 群集，可以从客户端计算机运行灵活的 [Azure PowerShell 脚本](https://msdn.microsoft.com/zh-cn/library/azure/dn864734.aspx)；该脚本使用预装了 HPC Pack 的 Windows Server VM 映像。<1你也可以使用 Azure [快速入门模板](https://azure.microsoft.com/zh-cn/documentation/templates/create-hpc-cluster/)与 Azure PowerShell 或 Azure CLI 来部署 HPC Pack 群集。

* 可以使用 Azure [快速入门模板](https://azure.microsoft.com/zh-cn/documentation/templates/slurm/)与 Azure PowerShell 或 Azure CLI，来部署运行 [SLURM](https://computing.llnl.gov/linux/slurm/) 开源工作负荷管理器的 Linux 群集。

将整个 HPC 群集置于云中会带来明显的好处。

* 组织无需购买和管理其他本地硬件即可运行 HPC 作业，并可以控制群集的大小，以便有效使用计算资源。

* 许多本地群集应用程序无需经过更改或者只需少量的修改就能在基于云的群集中运行。

* 相比于可将本地群集扩展到云的混合解决方案，完全在云中运行应用程序可以简化数据访问。你可以将所有应用程序数据存储在云中，而不必在云与本地安装之间分割数据，或者从应用程序远程访问数据。

* 某些软件供应商优化了其应用程序，使其可以在基于云的群集中运行。例如，通过将 MathWorks 提供的 [MATLAB 分布式计算服务器](http://www.mathworks.com/products/distriben/)部署到 Azure 虚拟机中的 HPC Pack 群集上，可以完全使用基于云的计算资源运行并行 MATLAB 作业。

### 方案 3.将并行应用程序横向扩展到 Azure

**何时选择此方案？**- 你可能会在本地工作站或小型群集中运行计算密集型应用程序，例如 Monte Carlo 仿真、动画渲染或媒体转码。你不想要管理计算资源或作业计划程序，而是想要专注于有效运行应用程序来解决业务问题。或者，你可能想要卸载计算密集型应用程序或第三方应用程序，使其在云中完全作为服务运行。

根据工作负荷，你可以利用 Azure 中由 Microsoft 或其他服务供应商托管的现有大型计算服务，以简化解决方案的基础结构和应用程序管理。某些服务为特定行业的客户托管特定的应用程序。某些服务会插入到本地应用程序，从而实现混合解决方案。[Azure 媒体服务](/documentation/services/media-services)等其他一些服务属于专用平台服务。

为了轻松地从云启用并相应地横向扩展目前运行的应用程序，[ 批处理( Batch )](/documentation/services/batch)提供了用于计划作业和管理服务中计算资源的 API。 批处理( Batch )可管理虚拟机的部署和自动缩放、作业计划、灾难恢复、数据移动、依赖性管理、应用程序部署，以及在云中运行作业所需的所有其他管道。目前， 批处理( Batch )非常适合用于内在并行应用程序，例如，在 Windows Server 计算资源中运行的图像渲染、金融风险建模和 Monte Carlo 仿真。

请参阅下图，了解开发人员可使用 批处理( Batch )创建的典型工作流。

![Azure  批处理( Batch )][batch_proc]

1. 将输入文件（例如源数据或图像、所需的应用程序、.exe 或脚本文件及其依赖项）上载到 Azure 存储空间。

2. 创建一个 批处理( Batch )服务，以便：

    a.部署包含相同 Azure VM 的池以运行应用程序，这类似于 HPC 群集中的计算节点。开发人员指定其大小、其运行的操作系统及其他属性，例如池是否可以自动缩放。在运行某个任务时，会自动向它分配此池中的某个 VM。

    b.创建用于运行该应用程序的作业。开发人员可以指定作业的计划和优先级，或者，作业可以按需运行。

    c.将作业分割成任务。每个任务实质上是在池中某个 VM 上运行的命令行，可以处理上载到存储空间的某个数据文件中的信息。

3. 运行服务并监视结果。


## 后续步骤

* 如需有关你解决方案的技术指导，请参阅[ 批处理( Batch )和 HPC 的技术资源](/documentation/articles/big-compute-resources)。

* 有关最新通告，请参阅 [Microsoft HPC 和 批处理( Batch )团队博客](http://blogs.technet.com/b/windowshpc/)与 [Azure 博客](http://azure.microsoft.com/blog/tag/hpc/)。

<!--Image references-->
[parallel]: ./media/batch-hpc-solutions/parallel.png
[coupled]: ./media/batch-hpc-solutions/coupled.png
[iaas_cluster]: ./media/batch-hpc-solutions/iaas_cluster.png
[burst_cluster]: ./media/batch-hpc-solutions/burst_cluster.png
[batch_proc]: ./media/batch-hpc-solutions/batch_proc.png

<!---HONumber=66-->