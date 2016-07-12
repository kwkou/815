<properties
	pageTitle="Azure Batch 功能概述 | Azure"
	description="从开发的角度了解 Batch 服务的功能及其 API。"
	services="batch"
	documentationCenter=".net"
	authors="yidingzhou"
	manager="timlt"
	editor=""/>

<tags 
	ms.service="batch" 
	ms.date="05/12/2016"
	wacn.date="06/06/2016"/>
	
# Azure Batch 功能概述

本文提供 Azure 批处理 ( Batch ) 服务的核心 API 功能的基本概述。无论是使用 [批处理 ( Batch ) REST][batch_rest_api] 还是 [批处理( Batch ) .NET][batch_net_api] API 来开发分布式计算解决方案，你都要使用下面讨论的许多实体和功能。

> [AZURE.TIP]有关 Batch 的更高级技术概述，请参阅 [Azure 批处理 ( Batch ) 基础知识](/documentation/articles/batch-technical-overview/)。

## <a name="workflow"></a>批处理 ( Batch ) 服务的工作流

在批处理 ( Batch ) 服务中开发的几乎所有分布式计算方案都会使用以下典型高级工作流：

1. 将你要在分布式计算方案中使用的*数据文件*上载到 [Azure 存储][azure_storage]帐户。这些文件必须在存储帐户中，批处理 ( Batch )  服务才能够访问它们。当任务运行时，会将这些文件下载到[计算节点](#computenode)。

2. 将相关*二进制文件*上载到你的存储帐户。这些二进制文件包含任务运行的程序及其任何依赖组件。还必须从存储帐户访问这些文件，使任务可将它们下载到计算节点。

3. 创建计算节点的[池](#pool)。可以在创建池时指定要使用的[计算节点大小][cloud_service_sizes]，运行任务时，将为它分配此池中的节点。

4. 创建[作业](#job)。作业可让你管理一系列的任务。

5. 将[任务](#task)添加到作业。每个任务使用你上载的程序来处理上载到存储帐户的数据文件中的信息。

6. 监视作业进度并检索结果。


在以下部分中，你将了解上述工作流中提到的每个资源，以及其他可实现分布式计算方案的许多批处理 ( Batch ) 功能。

## <a name="resource"></a>批处理 ( Batch ) 服务的资源

当你使用 Batch 时，可以使用以下多种资源：其中有些资源（如帐户、计算节点、池、作业和任务）都可用于所有 Batch 解决方案。其他资源（如作业计划和应用程序包）都很有用，但为可选功能。

- [帐户](#account)

- [计算节点](#computenode)

- [池](#pool)

- [作业](#job)

- [任务](#task)

	- [启动任务](#starttask)
	- [作业管理器任务](#jobmanagertask)
	- [作业准备和释放任务](#jobpreprelease)
	- [多实例任务](#multiinstance)
    - [任务依赖项](#taskdep)
- [作业计划](#jobschedule)
- [应用程序包](#appkg)

### <a name="account"></a>帐户

批处理 ( Batch ) 帐户是批处理 ( Batch ) 服务中唯一标识的实体。所有处理都与一个批处理 ( Batch ) 帐户相关联。当你使用批处理( Batch ) 服务执行操作时，需要同时用到帐户名和帐户密钥。若要创建批处理( Batch ) 帐户，请查看[在 Azure 经典管理门户中创建和管理 Azure 批处理 ( Batch )  帐户](/documentation/articles/batch-account-create-portal/)。

### <a name="computenode"></a>计算节点

计算节点是专门为应用程序处理特定工作负荷的 Azure 虚拟机。节点大小确定了 CPU 核心数目、内存容量，以及分配给节点的本地文件系统大小。节点可以是除 A0 以外的任何云服务节点大小。

节点可以运行可执行文件和脚本，包括可执行文件 (.exe)、命令文件 (.cmd)、批处理 ( Batch ) 文件 (.bat) 和 PowerShell 脚本。节点还具有以下属性：

- 将在每个计算节点上创建标准**文件夹结构**和关联的**环境变量**（用于详细描述其路径）。有关详细信息，请参阅下面的[文件和目录](#files)。
- 可由任务引用的**环境变量**。
- 配置为控制访问的**防火墙**设置。
- 如果需要**远程访问**某个计算节点（例如，出于调试目的），可以先获取 RDP 文件，然后，使用该文件通过*远程桌面*访问该节点。

### <a name="pool"></a>池

池是运行应用程序的节点集合。你可以手动创建池；或者在你指定要完成的工作时，由批处理 ( Batch ) 服务自动创建池。你可以创建和管理符合应用程序需求的池，但池只能由其创建时所在的批处理 ( Batch ) 帐户使用。一个批处理 ( Batch ) 帐户可以有多个池。

Azure 批处理 ( Batch ) 池构建在核心 Azure 计算平台的顶层；批处理 ( Batch ) 池提供大规模的分配、应用程序安装、数据分发和运行状况监视，以及在池内灵活调整计算节点数目（缩放）等功能。

添加到池中的每个节点都分配有唯一的名称和 IP 地址。从池中删除某个节点时，会丢失对操作系统或文件所做的任何更改，并且节点的名称和 IP 地址将被释放供将来使用。当某个节点退出池时，它的生存期即告结束。

你可以配置一个池，以便在该池中的节点之间进行通信。如果为池请求了池内通信，批处理 ( Batch ) 服务允许在池中每个节点上使用大于 1100 的端口。池中的每个节点都配置为只允许这个端口范围和来自池中其他节点的传入连接。如果应用程序不需要节点之间的通信，批处理 ( Batch ) 服务可以将许多不同的群集和数据中心的大量节点分配给池，以发挥更强大的并行处理能力。

在创建池时，可以指定以下属性：

- 池中**节点的大小**
	- 应该根据要在节点上运行的一个或多个应用程序的特征和要求，选择适当的节点大小。通常，选择节点大小时会假设某个任务要在节点上运行一次。考虑应用程序是否是多线程的以及其消耗的内存量等因素有助于确定最合适且经济高效的节点大小。可以分配多个任务和并行运行多个应用程序实例，在此情况下，通常会选择较大的节点 - 有关详细信息，请参阅下面的“任务计划策略”。
	- 池中所有节点的大小必须相同。如果要根据不同的系统要求和/或负载级别运行不同的应用程序，则应该创建独立的池。
	- 可为池配置所有[云服务节点大小][cloud_service_sizes]，但 A0 除外。

- 在节点上运行的**操作系统系列**和**版本**
	- 与云服务中的辅助角色一样，*OS 系列*和 *OS 版本*也是可以指定的（有关辅助角色的详细信息，请参阅 *Azure 提供的计算托管选项*中的[介绍云服务][about_cloud_services]）。
	- 操作系统系列还确定了要与操作系统一起安装哪些版本的 .NET。
	- 与辅助角色一样，对于 OS 版本，建议指定 `*`，使节点可自动升级，而无需采取措施来适应新的版本。选择特定操作系统版本的主要用例是在允许更新版本之前执行向后兼容测试，以确保保持应用程序兼容性。验证后，便可以更新池的操作系统版本并安装新的操作系统映像 – 所有正在运行的任务将会中断并重新排队。

- 应该为池提供的**节点的目标数目**

- 池的**缩放策略**
	- 除了节点数目以外，你还可以为池指定[自动缩放公式](/documentation/articles/batch-automatic-scaling/)。批处理 ( Batch ) 服务将执行该公式，并根据可以指定的各个池、作业、和任务参数，调整池中的节点数目。

- **任务计划**策略
	- [每个节点的最大任务数](/documentation/articles/batch-parallel-node-tasks/)配置选项确定了可以在池中每个节点上并行运行的最大任务数。
	- 默认的配置是每次在一个计算节点上运行一个任务，但在某些情况下，在一个节点上同时执行多个任务可能更有利。一个示例就是当应用程序必须等待 I/O 时增大节点利用率。并行执行多个应用程序会增大 CPU 利用率。另一个示例是减少池中的节点数目。这可以减少大型引用数据集所需的数据传输量 - 如果 A1 节点大小足以供某个应用程序使用，则可以改为选择 A4 节点大小，并针对 8 个并行任务配置池，使每个任务使用一个核心。
	- 还可以指定一个“填充类型”，用于确定批处理 ( Batch ) 是要将任务平均分散到所有节点，还是在将最大数目的任务分配给一个节点后，再将任务分配给池中的另一个节点。

- 池中节点的**通信状态**
	- 可以将池配置为允许池中节点之间的通信，以确定池的底层网络基础结构。请注意，这也会影响节点在群集中的位置。
	- 在大多数情况下，任务将独立运行而无需互相通信，但在某些应用程序中，任务必须相互通信。

- 池中节点的**启动任务**
	- 可以指定每当计算节点加入池和节点重新启动时要执行的“启动任务”。此任务通常用于安装在节点上运行的任务所使用的应用程序。

### <a name="job"></a>作业

作业是任务的集合，指定如何对池中的计算节点执行计算。

- 作业指定要在其上运行工作的**池**。该池可以是以前创建的由多个作业使用的现有池，或者是为作业计划关联的每个作业创建的，或者为作业计划关联的所有作业创建的。
- 可以指定一个可选的**作业优先级**。如果提交的作业的优先级高于当前正在进行的其他作业，则会将高优先级作业任务插入到队列中低优先级作业任务的前面。已经运行的低优先级任务不会预先清空。
- 作业**约束**为作业指定特定的限制。
	- 可为作业设置**最大挂钟时间**。如果作业运行的时间超过了指定的最大挂钟时间，则会结束该作业和所有关联的任务。
	- Azure 批处理 ( Batch ) 可以检测失败的任务并重试任务。可以将**任务重试最大次数**指定为约束，包括指定是要始终重试还是永不重试某个任务。重试某个任务意味着要将任务重新排队以再次运行。
- 可以通过客户端应用程序将任务添加到作业，或是指定[作业管理器任务](#jobmanagertask)。作业管理器任务使用批处理 ( Batch ) API，并包含必要的信息用于为池中某个计算节点上运行的包含任务的作业创建所需的任务。作业管理器任务专门由批处理 ( Batch ) 来处理 – 创建作业和重新启动失败的作业后，会立即将任务排队。作业计划创建的作业需要作业管理器任务，因为它是在实例化作业之前定义任务的唯一方式。下面提供了有关作业管理器任务的详细信息。


### <a name="task"></a>任务

任务是与作业关联的计算单位，在节点上运行。任务将分配到节点以执行，或排入队列直到节点空闲。任务使用以下资源：

- 在任务的**命令行**中指定的应用程序。

- 包含要处理的数据的**资源文件**。这些文件自动从 Azure 存储帐户中的 Blob 存储复制到节点。有关详细信息，请参阅下面的[文件和目录](#files)。

- 应用程序所需的**环境变量**。有关详细信息，请参阅下面的[任务的环境设置](#environment)。

- 运行计算所依据的**约束**。例如，允许运行任务的最长时间、任务运行失败时应该重试的次数上限，以及文件保留在工作目录中的最长时间。

除了可以定义在节点上运行计算的任务以外，批处理 ( Batch ) 服务还提供以下特殊任务：

- [启动任务](#starttask)
- [作业管理器任务](#jobmanagertask)
- [作业准备和释放任务](#jobmanagertask)
- [多实例任务](#multiinstance)
- [任务依赖项](#taskdep)

#### <a name="starttask"></a>启动任务

通过将**启动任务**与池相关联，可以配置池节点的操作环境，以便执行安装软件或启动后台进程等操作。启动任务在节点每次启动时运行，且只要保留在池中就会持续运行（包括首次将节点添加到池时）。启动任务的主要优点是包含全部所需的信息，使你能够配置计算节点，以及安装执行作业任务所需的应用程序。因此，要增加池中的节点数目，只需指定新的目标节点计数即可 - 批处理 ( Batch ) 已包含配置新节点并使其可接受任务所需的全部信息。

与任何批处理 ( Batch ) 任务一样，除了指定要执行的**命令行**以外，还可以指定 [Azure 存储空间][azure_storage]中的**资源文件**列表。Azure 批处理 ( Batch ) 将先从 Azure 存储空间复制文件，然后再运行命令行。对于池启动任务，文件列表通常包含应用程序包或文件，但它还可能包含计算节点上运行的所有任务使用的引用数据。启动任务的命令行可能会执行 PowerShell 脚本或 `robocopy` 操作，例如，将应用程序文件复制到“共享”文件夹中，然后运行 MSI 或 `setup.exe`。

> [AZURE.IMPORTANT] Batch 目前“仅”支持**常规用途**存储帐户类型，如 [About Azure storage accounts（关于 Azure 存储帐户）](/documentation/articles/storage-create-storage-account/)的 [Create a storage account（创建存储帐户）](/documentation/articles/storage-create-storage-account/#create-a-storage-account)中步骤 5 所述。Batch 任务（包括标准任务、启动任务、作业准备和作业释放任务）“只能”指定位于**常规用途**存储帐户中的资源文件。
通常，批处理 ( Batch ) 服务需要等待启动任务完成，然后认为节点已准备好分配任务，但这种行为是可配置的。

如果某个计算节点上的启动任务失败，则节点的状态将会更新以反映失败状态，同时，该节点不可用于要分配的任务。如果从存储中复制启动任务的资源文件时出现问题，或由其命令行执行的进程返回了非零退出代码，则启动任务可能会失败。

#### <a name="jobmanagertask"></a>作业管理器任务

**作业管理器任务**通常用于控制和/或监视作业的执行。例如，创建和提交作业的任务、确定要运行的其他任务，以及确定任务何时完成。但是，作业管理器任务并不限定于这些活动 - 它是功能齐备的任务，可执行作业所需的任何操作。比方说，作业管理器任务可以下载指定为参数的文件、分析该文件的内容，并根据这些内容提交其他任务。

作业管理器任务在所有其他任务之前启动，并提供以下功能：

- 创建作业时由批处理 ( Batch ) 服务自动提交为任务。

- 安排在作业中的其他任务之前执行。

- 缩小池时，关联的节点最后才从池中删除。

- 此终止可能完全取决于作业中的所有任务终止。

- 需要重新启动时，作业管理器任务有最高的优先级。如果找不到空闲的节点，批处理 ( Batch ) 服务可以终止池中正在运行的其他某个任务，以便腾出空间供作业管理器任务运行。

- 一个作业中的作业管理器任务的优先级不高于其他作业的任务。不同作业之间只遵循作业级别的优先级。

#### <a name="jobpreprelease"></a>作业准备和释放任务

批处理 ( Batch ) 提供作业准备任务用于预先设置作业的执行，并提供作业释放任务用于在完成作业后执行维护或清理。

- **作业准备任务** - 在任何其他作业任务执行之前，作业准备任务在计划要运行任务的所有计算节点上运行。例如，使用作业准备任务可以复制所有任务共享的、但对作业唯一的数据。
- **作业释放任务** - 作业完成后，作业释放任务将在池中至少运行了一个任务的每个节点上运行。例如，使用作业释放任务可删除作业准备任务所复制的数据，或压缩并上载诊断日志数据。

作业准备和释放任务允许你指定命令行在任务被调用时运行，并提供许多功能，例如文件下载、提升权限的执行、自定义环境变量、最大执行持续时间、重试计数和文件保留时间。

有关作业准备和释放任务的详细信息，请参阅[在 Azure 批处理 ( Batch ) 计算节点上运行作业准备和完成任务](/documentation/articles/batch-job-prep-release/)。

#### <a name="multiinstance"></a>多实例任务

[多实例任务](/documentation/articles/batch-mpi/)是经过配置后可以在多个计算节点上同时运行的任务。通过多实例任务，你可以启用消息传递接口 (MPI) 等高性能计算方案，此类方案需要将一组计算节点分配到一起来处理单个工作负荷。

有关在 Batch 中使用 Batch .NET 库运行 MPI 作业的详细介绍，请参阅[在 Azure Batch 中使用多实例任务来执行消息传递接口 (MPI) 应用程序](/documentation/articles/batch-mpi/)。

#### <a name="taskdep"></a>任务依赖性

顾名思义，任务依赖性可让你在执行某个任务之前，指定该任务与其他任务的依赖性。此功能提供以下情况的支持：“下游”任务取用“上游”任务的输出，或当上游任务执行下游任务所需的某种初始化时。若要使用此功能，必须先在 Batch 作业中启用任务依赖性。然后，针对每个依赖于另一个任务（或其他许多任务）的任务，指定该任务依赖的任务。

使用任务依赖性，可以配置如下所述的方案：

* taskB 依赖于 taskA（直到 taskA 完成，才开始执行 taskB）
* taskC 同时依赖于 taskA 和 taskB
* taskD 在执行前依赖于某个范围的任务，例如任务 1 到 10

请查看 [azure-batch-samples][github_samples] GitHub 存储库中的 [TaskDependencies][github_sample_taskdeps] 代码示例。你将在其中了解如何使用 [Batch .NET][batch_net_api] 库配置依赖于其他任务的任务。

### <a name="jobschedule"></a>计划的作业

作业计划可让你在批处理 ( Batch ) 服务中创建周期性作业。作业计划指定何时要运行作业，并包含要运行的作业的规范。作业计划允许指定计划的持续时间（计划的持续时间和生效时间），以及在该时间段创建作业的频率。

### <a name="appkg"></a>应用程序包

应用程序包功能可为池中的计算节点提供简单的应用程序管理和部署能力。通过应用程序包，可以轻松上载及管理多个版本的工作执行应用程序（包括二进制文件和支持文件），然后将一或多个这种类型的应用程序自动部署到池中的计算节点。

Batch 能在后台处理使用 Azure 存储空间将应用程序包安全存储及部署到计算节点的详细信息，因此可以简化代码和管理开销。



## <a name="files"></a>文件和目录

每个任务都有一个工作目录，任务将在该目录中创建零个或多个文件和目录用于存储任务运行的程序、任务处理的数据，以及任务执行的处理的输出。在作业运行期间，这些文件和目录可供其他任务使用。节点上的所有任务、文件和目录由单个用户帐户拥有。

批处理 ( Batch ) 服务在节点上公开文件系统的一部分作为“根目录”。 任务可以通过访问 `%AZ_BATCH_NODE_ROOT_DIR%` 环境变量来使用根目录。有关使用环境变量的详细信息，请参阅[任务的环境设置](#environment)。

![计算节点目录结构][1]

根目录包含以下目录结构：

- **共享** - 此位置是在节点上运行的所有任务的共享目录，与作业无关。在节点上，可以通过 `%AZ_BATCH_NODE_SHARED_DIR%` 来访问共享目录。此目录允许对节点上运行的所有任务进行读取/写入访问。任务可以创建、读取、更新和删除此目录中的文件。

- **启动** - 启动任务使用此位置做为它的工作目录。由批处理 ( Batch ) 服务下载用来运行启动任务的所有文件也存储在此目录下。在节点上，可以通过 `%AZ_BATCH_NODE_START_DIR%` 环境变量来访问启动目录。启动任务可以创建、读取、更新和删除此目录下的文件，启动任务可以使用此目录来配置操作系统。

- **任务** -为节点上运行的每个任务创建一个目录，可通过 `%AZ_BATCH_TASK_DIR%` 访问该目录。在每个任务目录中，批处理 ( Batch ) 服务将创建由 `%AZ_BATCH_TASK_WORKING_DIR%` 环境变量指定唯一路径的任务目录 (`wd`)。此目录提供对任务的读/写访问权限。任务可以创建、读取、更新和删除此目录下的文件，此目录根据任务指定的 *RetentionTime* 约束而保留。
  - `stdout.txt` 和 `stderr.txt` - 在任务执行期间，会将这些文件写入任务文件夹。

从池中删除节点时，也会删除节点上存储的所有文件。

## <a name="lifetime"></a>池和计算节点生存期

在设计 Azure 批处理 ( Batch ) 解决方案时，你必须做出有关如何及何时创建池，以及这些池中的计算节点可用性要保持多久的设计决策。

在极端情况下，可以在提交作业为每个作业创建一个池，并在任务完成执行后立即删除节点。这可以最大程度地提高利用率，因为仅当绝对必要时才会分配节点，并且在节点空闲时会立即将其关闭。这意味着作业必须等待分配节点，不过，你必须注意，在任务单独可用、已分配并且启动任务已完成时，会立即将任务安排给节点。批处理 ( Batch )  *不会*在等到池中的所有节点都可用后才分配任务，因此可确保最大程度地利用所有节点。

在另一种极端情况下，如果最高优先级是让作业立即启动，则你可以预先创建池，并使其节点在提交作业之前可用。在此情况下，作业任务可以立即启动，但节点可能会保持空闲状态以等待分配任务。

通常用于处理可变但持续存在的负载的组合方法是创建一个池用于容纳提交的多个作业，但同时根据作业负载向上或向下缩放节点数目（请参阅下面的*缩放应用程序*）。可以根据当前负载被动执行此操作，或者在负载可预测时主动执行此操作。

## <a name="scaling"></a>缩放应用程序

通过[自动缩放](/documentation/articles/batch-automatic-scaling/)功能，你可以让 Batch 服务根据计算方案的当前工作负荷和资源使用状况动态缩放池中的计算节点数目。这样，便可做到只使用所需资源并可释放不需要的资源，因而能够降低运行应用程序的整体成本。可以在创建池时为其指定自动缩放设置或是稍后再启用自动缩放，此外也可以更新已启用自动缩放功能的池上的缩放设置。

为池指定**自动缩放公式**即可执行自动缩放。Batch 服务使用此公式来确定池中下一个缩放间隔（可以指定的间隔）的目标节点数目。

例如，也许某个作业需要提交大量计划执行的任务。你可以将缩放公式分配到池，以根据当前的挂起任务数和这些任务的完成率来调整池中的节点数目。Batch 服务将定期评估公式，并根据工作负荷和公式设置来调整池的大小。

缩放公式可以基于以下度量值：

- **时间度量值** - 根据指定的时数内每隔五分钟收集的统计信息。

- **资源度量值** - 根据 CPU 使用率、带宽使用率、内存使用率和节点的数目。

- **任务度量值** - 根据任务的状态，例如“活动”、“挂起”和“已完成”。

当自动缩放减少池中的计算节点数目时，必须考虑当前正在运行的任务。为了配合这一点，公式中可以包含节点取消分配策略设置，以指定是否立即停止正在运行的任务，或允许先完成再从池中删除节点。

> [AZURE.TIP] 若要获得最大的计算资源使用率，请将节点的目标数目设置成在作业结束时降为零，但允许正在运行的任务完成。

有关自动缩放应用程序的详细信息，请参阅[自动缩放 Azure Batch 池中的计算节点](/documentation/articles/batch-automatic-scaling/)。

## <a name="cert"></a>证书的安全性

在加密或解密任务的敏感信息（例如 [Azure 存储帐户][azure_storage]的密钥）时，通常需要使用证书。为此，可以在节点上安装证书。加密的机密通过命令行参数或内嵌在某个任务资源中来传递给任务，已安装的证书可用于解密机密。

可以使用[添加证书][rest_add_cert]操作 (批处理 ( Batch ) REST API) 或 [CertificateOperations.CreateCertificate][net_create_cert] 方法 (批处理 ( Batch ) .NET API) 将证书添加到批处理 ( Batch ) 帐户。然后，可以将该证书与新的或现有的池相关联。将证书与池关联后，Batch 服务将在池中的每个节点上安装该证书。在启动节点之后、启动任何任务（包括启动任务和作业管理器任务）之前，Batch 服务将安装相应的证书。

## <a name="scheduling"></a>计划优先级

可以向你在批处理 ( Batch ) 中创建的作业分配优先级。批处理 ( Batch ) 服务使用作业的优先级值来确定帐户中的作业计划顺序。优先级值的范围为 -1000 到 1000，-1000 表示最低优先级，1000 表示最高优先级。可以使用[更新作业的属性][rest_update_job]操作 (批处理( Batch ) REST API) 或通过修改 [CloudJob.Priority][net_cloudjob_priority] 属性 (批处理 ( Batch ) .NET API) 来更新作业的优先级。 

在同一个帐户内，高优先级作业的计划优先顺序高于低优先级作业。一个帐户中具有较高优先级值的作业，其计划优先级并不高于不同帐户中较低优先级值的另一个作业。

不同池的作业计划是独立的。在不同的池之间，即使作业的优先级较高，如果其关联的池缺少空闲的节点，则不保证此作业优先计划。在同一个池中，相同优先级的作业有相同的计划机会。

## <a name="environment"></a>任务的环境设置

在批处理 ( Batch ) 作业中执行的每个任务可以访问由批处理 ( Batch ) 服务设置的环境变量（系统定义的环境变量，请参阅下表）以及用户定义的环境变量。任务在计算节点上运行的应用程序和脚本可以在节点上执行期间访问这些环境变量。

可以在使用[将任务添加到作业][rest_add_task]操作 ( 批处理( Batch ) REST API) 时设置用户定义的环境变量，或者在将任务添加到作业时通过修改 [CloudTask.EnvironmentSettings][net_cloudtask_env] 属性(批处理 ( Batch ) .NET API) 进行这项设置。

使用[获取有关任务的信息][rest_get_task_info]操作 (Batch REST API) 或通过访问 [CloudTask.EnvironmentSettings][net_cloudtask_env] 属性 (批处理( Batch ) .NET API)，来获取任务的环境变量（系统定义的和用户定义的环境变量）。如前所述，在计算节点上执行的进程也可以访问所有环境变量，例如，通过使用你所熟悉的 `%VARIABLE_NAME%` 语法来访问。

对于在作业中计划的每项任务，批处理 ( Batch ) 服务将设置以下系统定义的环境变量集：

| 环境变量名称 | 说明 |
|---------------------------------|--------------------------------------------------------------------------|
| `AZ_BATCH_ACCOUNT_NAME` | 任务所属的帐户名。 |
| `AZ_BATCH_JOB_ID` | 任务所属的作业的 ID。 |
| `AZ_BATCH_JOB_PREP_DIR` | 节点上的作业准备任务目录的完整路径。 |
| `AZ_BATCH_JOB_PREP_WORKING_DIR` | 节点上的作业准备任务工作目录的完整路径。 |
| `AZ_BATCH_NODE_ID` | 运行任务的节点的 ID。 |
| `AZ_BATCH_NODE_ROOT_DIR` | 节点上的根目录的完整路径。 |
| `AZ_BATCH_NODE_SHARED_DIR` | 节点上的共享目录的完整路径。 |
| `AZ_BATCH_NODE_STARTUP_DIR` | 节点上的计算节点启动任务目录的完整路径。 |
| `AZ_BATCH_POOL_ID` | 运行任务的池的 ID。 |
| `AZ_BATCH_TASK_DIR` | 节点上的任务目录的完整路径。 |
| `AZ_BATCH_TASK_ID` | 当前任务的 ID。 |
| `AZ_BATCH_TASK_WORKING_DIR` | 节点上的任务工作目录的完整路径。 |

>[AZURE.NOTE]无法覆盖上述任何系统定义的变量 - 它们是只读的。

## <a name="errorhandling"></a>错误处理

有时你可能需要处理批处理( Batch ) 解决方案中的任务和应用程序失败。

### 任务失败处理
任务失败划分为以下类别：

- **计划失败**
	- 如果为任务指定的文件传输出于任何原因失败，将为该任务设置“计划错误”。
	- 计划错误的原因可能是文件已移动、存储帐户不再可用，或者发生其他使文件无法成功复制到节点的问题。
- **应用程序失败**
	- 任务命令行指定的进程也可能会失败。如果任务执行的进程返回非零退出代码，则将该进程视为失败。
	- 对于应用程序失败，可以将批处理 ( Batch ) 配置为自动重试任务，并最多重试指定的次数。
- **约束失败**
	- 可以设置一个约束来指定作业或任务的最大执行持续期间，即 *maxWallClockTime*。此约束可用于终止“挂起的”任务。
	- 如果超出了最长时间，则将任务标记为*已完成*，但退出代码将设置为 `0xC000013A`，*schedulingError* 字段将标记为 `{ category:"ServerError", code="TaskEnded"}`。

### 调试应用程序失败

在执行过程中，应用程序可以生成诊断输出，这些信息可用于排查问题。如前面的[文件和目录](#files)中所述，批处理( Batch ) 服务会将 stdout 和 stderr 输出发送到计算节点上的任务目录中的 `stdout.txt` 和 `stderr.txt` 文件。在批处理( Batch ) .NET API 中使用 [ComputeNode.GetNodeFile][net_getfile_node] 和 [CloudTask.GetNodeFile][net_getfile_task]，可以检索这些文件和其他文件来进行故障排除。

使用*远程桌面*登录到计算节点后，可以执行更广泛的调试。你可以[从节点获取远程桌面协议文件][rest_rdp] (批处理( Batch ) REST API) 或使用 [ComputeNode.GetRDPFile][net_rdp] 方法 (批处理 ( Batch ) .NET API) 来进行远程登录。

>[AZURE.NOTE]若要通过 RDP 连接到某个节点，必须先在该节点上创建一个用户。在 Batch REST API 中[将用户帐户添加到节点][rest_create_user]，或使用批处理( Batch ) .NET 中的 [ComputeNode.CreateComputeNodeUser][net_create_user] 方法。

### 应对任务失败或中断

任务偶尔会失败或中断。任务应用程序本身可能会失败，运行任务的节点可能会重新启动，或者在调整大小操作期间，可能会因为池的取消分配策略设置为在不等待任务完成的情况下立即删除节点，而从池中删除节点。在所有情况下，任务都可以由批处理 ( Batch ) 自动排队，并在另一个节点上执行。

间歇性的问题也有可能会导致任务挂起，或者花费很长时间才能完成执行。可为任务设置最长执行时间，如果超过此时间，批处理 ( Batch ) 会中断任务应用程序。

### 对“不良的”计算节点进行故障排除

在部分任务失败的情况下，Batch 客户端应用程序或服务可以检查失败任务的元数据来找出行为异常的节点。池中的每个节点都有一个唯一 ID，运行任务的节点包含在任务元数据中。一旦找到，你就可以采取以下几种措施：

- **重新启动节点** ([REST][rest_reboot] | [.NET][net_reboot])

	重新启动节点有时可以清除潜在的问题，例如进程停滞或崩溃。请注意，如果池使用启动任务或作业使用作业准备任务，节点重新启动时将执行这些任务。

- **重置映像节点** ([REST][rest_reimage] | [.NET][net_reimage])

	这会在节点上重新安装操作系统。和重新启动节点一样，在重置映像节点后，便重新执行启动任务和作业准备任务。

- **从池中删除节点** ([REST][rest_remove] | [.NET][net_remove])

	有时必须从池中完全删除节点。

- **禁用节点上的任务计划** ([REST][rest_offline] | [.NET][net_offline])

	这实际上是使节点“脱机”，以便不再收到任何分配的任务，但允许节点继续运行并保留在池中。这可让你执行进一步的调查以了解失败原因，却又会不丢失失败任务的数据，并且不让节点造成额外的任务失败。例如，你可以禁用节点上的任务计划，然后从远程登录以检查节点的事件日志，或执行其他故障排除操作。完成调查后，可以启用任务计划（[REST][rest_online]、[.NET][net_online]）使节点重新联机，或者执行上述其他操作。

> [AZURE.IMPORTANT] 可以使用上述各个操作（重新启动、重置映像、删除、禁用任务计划），指定当执行操作时要如何处理节点上当前正在运行的任务。例如，当你禁用具有 Batch .NET 客户端库的节点上的任务计划时，可以指定 [DisableComputeNodeSchedulingOption][net_offline_option] 枚举值，以指定是要“终止”运行中的任务、将任务“重新排队”以在其他节点上计划，还是允许执行中的任务先完成再执行作业 (TaskCompletion)。

## 后续步骤

- 根据[适用于 .NET 的 Azure 批处理 ( Batch ) 库入门](/documentation/articles/batch-dotnet-get-started/)中的步骤创建第一个批处理 ( Batch ) 应用程序
- 下载并构建 [批处理( Batch ) 资源管理器][batch_explorer_project]示例项目，以便在开发批处理 ( Batch ) 解决方案时使用。使用批处理( Batch ) 资源管理器可以执行以下和其他操作：
  - 监视和管理批处理 ( Batch ) 帐户中的池、作业与任务
  - 从节点下载 `stdout.txt`、`stderr.txt` 和其他文件
  - 在节点上创建用户，并下载用于远程登录的 RDP 文件

[1]: ./media/batch-api-basics/node-folder-structure.png

[about_cloud_services]: /documentation/articles/fundamentals-application-models/#tell-me-about-cloud-services
[azure_storage]: /services/storage/
[batch_explorer_project]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/BatchExplorer
[cloud_service_sizes]: /documentation/articles/cloud-services-sizes-specs/
[msmpi]: https://msdn.microsoft.com/library/bb524831.aspx
[github_samples]: https://github.com/Azure/azure-batch-samples
[github_sample_taskdeps]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/ArticleProjects/TaskDependencies

[batch_net_api]: https://msdn.microsoft.com/library/azure/mt348682.aspx
[net_cloudjob_jobmanagertask]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudjob.jobmanagertask.aspx
[net_cloudjob_priority]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudjob.priority.aspx
[net_cloudpool_starttask]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudpool.starttask.aspx
[net_cloudtask_env]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudtask.environmentsettings.aspx
[net_create_cert]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.certificateoperations.createcertificate.aspx
[net_create_user]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.computenode.createcomputenodeuser.aspx
[net_getfile_node]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.computenode.getnodefile.aspx
[net_getfile_task]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudtask.getnodefile.aspx
[net_multiinstancesettings]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.multiinstancesettings.aspx
[net_rdp]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.computenode.getrdpfile.aspx
[net_reboot]: https://msdn.microsoft.com/library/azure/mt631495.aspx
[net_reimage]: https://msdn.microsoft.com/library/azure/mt631496.aspx
[net_remove]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.pooloperations.removefrompoolasync.aspx
[net_offline]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.computenode.disableschedulingasync.aspx
[net_online]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.computenode.enableschedulingasync.aspx
[net_offline_option]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.common.disablecomputenodeschedulingoption.aspx

[batch_rest_api]: https://msdn.microsoft.com/library/azure/Dn820158.aspx
[rest_add_job]: https://msdn.microsoft.com/library/azure/mt282178.aspx
[rest_add_pool]: https://msdn.microsoft.com/library/azure/dn820174.aspx
[rest_add_cert]: https://msdn.microsoft.com/library/azure/dn820169.aspx
[rest_add_task]: https://msdn.microsoft.com/library/azure/dn820105.aspx
[rest_create_user]: https://msdn.microsoft.com/library/azure/dn820137.aspx
[rest_get_task_info]: https://msdn.microsoft.com/library/azure/dn820133.aspx
[rest_multiinstance]: https://msdn.microsoft.com/library/azure/mt637905.aspx
[rest_multiinstancesettings]: https://msdn.microsoft.com/library/azure/dn820105.aspx#multiInstanceSettings
[rest_update_job]: https://msdn.microsoft.com/library/azure/dn820162.aspx
[rest_rdp]: https://msdn.microsoft.com/library/azure/dn820120.aspx
[rest_reboot]: https://msdn.microsoft.com/library/azure/dn820171.aspx
[rest_reimage]: https://msdn.microsoft.com/library/azure/dn820157.aspx
[rest_remove]: https://msdn.microsoft.com/library/azure/dn820194.aspx
[rest_offline]: https://msdn.microsoft.com/library/azure/mt637904.aspx
[rest_online]: https://msdn.microsoft.com/library/azure/mt637907.aspx

<!---HONumber=Mooncake_0530_2016-->