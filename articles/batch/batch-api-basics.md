<properties
    pageTitle="面向开发人员的 Azure 批处理概述 | Microsoft 文档"
    description="从开发的角度了解 Batch 服务的功能及其 API。"
    services="batch"
    documentationcenter=".net"
    author="tamram"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="416b95f8-2d7b-4111-8012-679b0f60d204"
    ms.service="batch"
    ms.devlang="multiple"
    ms.topic="get-started-article"
    ms.tgt_pltfrm="na"
    ms.workload="big-compute"
    ms.date="03/27/2017"
    ms.author="tamram"
    ms.custom="H1Hack27Feb2017"
    wacn.date="05/15/2017"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="3ff18e6f95d8bbc27348658bc5fce50c3320cf0a"
    ms.openlocfilehash="18d56ef2fb64bd1aaa29786d69c0accb8e943c19"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/15/2017" />

# <a name="develop-large-scale-parallel-compute-solutions-with-batch"></a>使用 Batch 开发大规模并行计算解决方案

这篇 Azure Batch 服务核心组件的概述将介绍 Batch 开发人员可用来构建大规模并发计算解决方案的主要服务功能和资源。

不管是在开发可发出直接 [REST API][batch_rest_api] 调用的分布式计算应用程序或服务，还是使用某个 [Batch SDK](/documentation/articles/batch-apis-tools/#batch-development-apis/)，都可以使用本文中介绍的多种资源和功能。

> [AZURE.TIP]
> 有关 Batch 服务的更全面介绍，请参阅 [Basics of Azure Batch](/documentation/articles/batch-technical-overview/)（Azure Batch 基础知识）。
>
>

## <a name="batch-service-workflow"></a>Batch 服务工作流
几乎所有使用 Batch 服务处理并行工作负荷的应用程序和服务都使用以下典型高级工作流：

1. 将要处理的 **数据文件** 上传到 [Azure 存储][azure_storage] 帐户。 Batch 包含访问 Azure Blob 存储的内置支持，在运行任务时，任务可以将这些文件下载到 [计算节点](#compute-node) 。
2. 上传任务所要运行的 **应用程序文件** 。 这些文件可能是二进制文件或脚本及其依赖项，并由作业中的任务执行。 任务可以从存储帐户下载这些文件，也可使用 Batch 的 [应用程序包](#application-packages) 功能来管理和部署应用程序。
3. 创建计算节点的 [池](#pool) 。 创建池时，可以指定池的计算节点数目、其大小和操作系统。 运行作业中的每个任务时，会将任务分配到池中的某个节点以执行。
4. 创建 [作业](#job)。 作业管理任务的集合。 可将每个作业关联到要运行该作业的任务的特定池。
5. 将 [任务](#task) 添加到作业。 每个任务将运行上传的应用程序或脚本，以处理它从存储帐户下载的数据文件。 当每个任务完成时，可将其输出上载到 Azure 存储。
6. 监视作业进度并从 Azure 存储检索任务输出。

以下部分介绍可实现分布式计算方案的上述和其他批处理资源。

> [AZURE.NOTE]
> 需要有[批处理帐户](#account)才能使用批处理服务。 此外，几乎所有解决方案都可以使用 [Azure 存储][azure_storage]帐户存储和检索文件。 批处理目前仅支持**常规用途**存储帐户类型，如 [About Azure storage accounts](/documentation/articles/storage-create-storage-account/)（关于 Azure 存储帐户）的 [Create a storage account](/documentation/articles/storage-create-storage-account/#create-a-storage-account/)（创建存储帐户）中步骤 5 所述。
>
>

## <a name="batch-service-resources"></a>Batch 服务资源
使用 Batch 服务的所有解决方案需要以下某些资源：帐户、计算节点、池、作业、任务。 其他资源（如作业计划和应用程序包）都很有用，但为可选功能。

- [帐户](#account)
- [计算节点](#compute-node)
- [池](#pool)
- [作业](#job)

  - [作业计划](#scheduled-jobs)
- [任务](#task)

  - [启动任务](#start-task)
  - [作业管理器任务](#job-manager-task)
  - [作业准备和释放任务](#job-preparation-and-release-tasks)
  - [多实例任务 (MPI)](#multi-instance-tasks)
  - [任务依赖项](#task-dependencies)
- [应用程序包](#application-packages)

## <a name="account"></a>帐户
批处理帐户是批处理服务中唯一标识的实体。 所有处理都与一个 Batch 帐户相关联。 使用 Batch 服务执行操作时，需要同时用到帐户名及其帐户密钥之一。 可以 [使用 Azure 门户创建 Azure Batch 帐户](/documentation/articles/batch-account-create-portal/)。

## <a name="compute-node"></a>计算节点
计算节点是专门用于处理一部分应用程序工作负荷的 Azure 虚拟机 (VM)。 节点大小确定了 CPU 核心数目、内存容量，以及分配给节点的本地文件系统大小。 可以使用 Azure 云服务或虚拟机应用商店映像创建的 Windows 或 Linux 节点池。 有关这些选项的详细信息，请参阅下面的 [池](#pool) 部分。

节点可以运行节点操作系统环境支持的任何可执行文件或脚本。 这包括适用于 Windows 的 \*.exe、\*.cmd、\*.bat 和 PowerShell 脚本，以及适用于 Linux 的二进制文件、shell 和 Python 脚本。

Batch 中的所有计算节点还包括：

- 任务可引用的标准[文件夹结构](#files-and-directories)和关联的[环境变量](#environment-settings-for-tasks)。
- **防火墙** 设置。
- [远程访问](#connecting-to-compute-nodes) Windows（远程桌面协议 (RDP)）和 Linux（安全外壳 (SSH)）节点。

## <a name="pool"></a>池
池是运行应用程序的节点集合。 你可以手动创建池；或者在你指定要完成的工作时，由 Batch 服务自动创建池。 你可以创建和管理符合应用程序资源要求的池。 池只能由创建它的 Batch 帐户使用。 一个批处理帐户可以有多个池。

Azure Batch 池构建在核心 Azure 计算平台的顶层。 它们提供大规模的分配、应用程序安装、数据分发和运行状况监视，以及在池内灵活调整计算节点数目（[缩放](#scaling-compute-resources)）等功能。

添加到池中的每个节点都分配有唯一的名称和 IP 地址。 从池中删除某个节点时，会丢失对操作系统或文件所做的任何更改，并且节点的名称和 IP 地址将被释放供将来使用。 当某个节点退出池时，它的生存期即告结束。

在创建池时，可以指定以下属性：

- 计算节点的**操作系统**和**版本**

    为池中的节点选择操作系统时，可以使用两个选项：“虚拟机配置”和“云服务配置”。

    **虚拟机配置**可从 [Azure 虚拟机应用商店][vm_marketplace]提供适用于计算节点的 Linux 和 Windows 映像。

    创建包含虚拟机配置节点的池时，不仅需要指定节点的大小，还需要在节点上安装**虚拟机映像引用**和批处理**节点代理 SKU**。 有关指定这些池属性的详细信息，请参阅 [Provision Linux compute nodes in Azure Batch pools](/documentation/articles/batch-linux-nodes/)（在 Azure Batch 池中预配 Linux 计算节点）。

    “云服务配置”*只*提供 Windows 计算节点。 [Azure Guest OS releases and SDK compatibility matrix](/documentation/articles/cloud-services-guestos-update-matrix/)（Azure 来宾 OS 版本和 SDK 兼容性对照表）中列出了适用于云服务配置池的操作系统。 创建包含云服务节点的池时，只需指定节点大小及其 *OS 系列*。 创建 Windows 计算节点池时，最常使用的是云服务。

  - *OS 系列* 还确定了要与操作系统一起安装哪些版本的 .NET。
  - 与云服务中的辅助角色一样，可以指定 *OS 版本*（有关辅助角色的详细信息，请参阅 [Cloud Services overview](/documentation/articles/cloud-services-choose-me/)（云服务概述）中的 [Tell me about cloud services](/documentation/articles/cloud-services-choose-me/#tellmecs/)（介绍云服务）部分）。
  - 与辅助角色一样，对于 *OS 版本*，建议指定 `*`，使节点可自动升级，而无需采取措施来适应新的版本。 选择特定 OS 版本的主要用例是在允许更新版本之前执行向后兼容测试，以确保保持应用程序兼容性。 验证后，便可以更新池的 *OS 版本* 并安装新的操作系统映像 - 所有正在运行的任务会中断并重新排队。
- **节点大小**

    **Sizes for Cloud Services** （云服务的大小）中列出了 [云服务配置](/documentation/articles/cloud-services-sizes-specs/)计算节点大小。 批处理支持 `ExtraSmall`、`STANDARD_A1_V2`、`STANDARD_A2_V2` 以外的所有云服务大小。

    [Sizes for virtual machines in Azure](/documentation/articles/virtual-machines-linux-sizes/)（Azure 中虚拟机的大小）(Linux) 和 [Sizes for virtual machines in Azure](/documentation/articles/virtual-machines-windows-sizes/)（Azure 中虚拟机的大小）(Windows) 中列出了“虚拟机配置”计算节点大小。 Batch 支持除 `STANDARD_A0` 和高级存储大小（`STANDARD_GS`、`STANDARD_DS` 和 `STANDARD_DSV2` 系列）以外所有的 Azure VM 大小。

    选择计算节点大小时，请考虑要在节点上运行的应用程序的特征和要求。 考虑应用程序是否是多线程的以及其消耗的内存量等因素有助于确定最合适且经济高效的节点大小。 通常，选择节点大小时会假设某个任务要在节点上运行一次。 但是，在作业执行期间，可能有多个任务（因而有多个应用程序实例）在计算节点上[并行运行](/documentation/articles/batch-parallel-node-tasks/)。 在此情况下，往往会选择较大的节点大小，以满足更高的并行任务执行需求。 有关详细信息，请参阅 [任务计划策略](#task-scheduling-policy) 。

    池中所有节点的大小相同。 如果打算运行具有不同系统要求和/或负载级别的应用程序，建议使用不同的池。
- **节点目标数目**

    这是要在池中部署的计算节点数目。 之所以称为 *目标* ，是因为在某些情况下，池可能无法达到所需的节点数目。 如果池已达到 Batch 帐户的 [核心配额](/documentation/articles/batch-quota-limit/) ，或应用到池的自动缩放公式限制了最大节点数（请参阅下面的“缩放策略”部分），则池无法达到所需节点数目。
- **缩放策略**

    对于动态工作负荷，可以编写 [自动缩放公式](#scaling-compute-resources) 并将其应用到池中。 Batch 服务将定期计算该公式，并根据可以指定的各个池、作业、和任务参数，调整池中的节点数目。
- **任务计划策略** <a name="task-scheduling-policy"></a>

    [每个节点的最大任务数](/documentation/articles/batch-parallel-node-tasks/) 配置选项确定了可以在池中每个计算节点上并行运行的最大任务数。

    默认配置指定每次在节点上运行一个任务，但在某些情况下，在一个节点上同时执行两个或多个任务可能更有利。 请参阅 [concurrent node tasks](/documentation/articles/batch-parallel-node-tasks/)（并发节点任务）一文中的[示例方案](/documentation/articles/batch-parallel-node-tasks/#example-scenario/)，了解如何通过在每个节点上运行多个任务来受益。

    还可以指定一个 *填充类型* ，用于确定 Batch 是要将任务平均分散到池中的所有节点，还是在将最大数目的任务分配给一个节点后，再将任务分配给另一个节点。
- **通信状态** 

    在大多数情况下，任务将独立运行，并不需要彼此通信。 但是，某些应用程序中的任务必须能够通信，例如 [MPI 方案](/documentation/articles/batch-mpi/)。

    用户可以配置一个池用于 **节点间通信**，以便池中的节点可在运行时进行通信。 启用节点间通信时，云服务配置池中的节点可以在超过 1100 个端口上彼此通信，并且虚拟机配置池不会限制任何端口的流量。

    请注意，启用节点间通信也会影响群集内的节点位置，并且由于部署限制，可能限制池中的最大节点数。 如果应用程序不需要节点之间的通信，Batch 服务可以将许多不同的群集和数据中心的大量节点分配给池，以发挥更强大的并行处理能力。
- **启动任务** 

    可选的 *启动任务* 将在每个节点加入池以及节点每次重新启动或重置映像时在该节点上运行。 启动任务特别适用于准备计算节点，以便执行任务，例如安装可通过任务在计算节点上运行的应用程序。
- **应用程序包**

    可以指定要部署到池中计算节点的 [应用程序包](#application-packages) 。 应用程序包提供任务运行的应用程序的简化部署和版本控制。 为池指定的应用程序包安装在加入该池的每个节点上，每次节点重新启动或重置映像时，将安装这些包。 Linux 计算节点目前不支持应用程序包。
- **网络配置**

    可以指定应在其中创建池计算节点的 Azure [虚拟网络 (VNet)](/documentation/articles/virtual-networks-overview/) 的 ID。 有关详细信息，请参阅[池网络配置](#pool-network-configuration)部分。

> [AZURE.IMPORTANT]
> 所有批处理帐户都有默认**配额**，用于限制批处理帐户中的**核心**（因此也包括计算节点）数目。 可以在 [Quotas and limits for the Azure Batch service](/documentation/articles/batch-quota-limit/)（Azure 批处理服务的配额和限制）中找到默认配额以及如何[提高配额](/documentation/articles/batch-quota-limit/#increase-a-quota/)（例如批处理帐户中的核心数目上限）的说明。 如果你有类似于“为什么我的池不能包含 X 个以上的节点？ ”的疑惑，则原因可能在于此核心配额。
>
>

## <a name="job"></a>作业
作业是任务的集合。 作业控制其任务对池中计算节点执行计算的方式。

- 作业指定要在其中运行工作的 **池** 。 可以为每个作业创建新池，或将池用于多个作业。 可以针对与作业计划关联的每个作业创建池，或者针对与作业计划关联的所有作业创建池。
- 可以指定可选的 **作业优先级**。 如果提交的作业的优先级高于当前正在进行的其他作业，则会将高优先级作业的任务插入到队列中低优先级作业的任务前面。 已经运行的低优先级作业中的任务不会预先清空。
- 可以使用作业 **约束** 来为作业指定特定的限制：

    可以设置 **最大挂钟时间**，以便在作业的运行时间超过指定的最大挂钟时间时，终止该作业及其所有关联的任务。

    Batch 可以检测并重试失败的任务。 可以将**任务重试最大次数**指定为约束，包括指定是要*始终*重试还是*永不*重试某个任务。 重试某个任务意味着要将任务重新排队以再次运行。
- 客户端应用程序可将任务添加到作业，或者也可以指定 [作业管理器任务](#job-manager-task)。 作业管理器任务包含必要的信息用于为池中某个计算节点上运行的包含作业管理器任务的作业创建所需的任务。 作业管理器任务专门由 Batch 来处理 - 创建作业和重新启动失败的作业后，会立即将任务排队。 *作业计划* 创建的作业 [需要](#scheduled-jobs) 作业管理器任务，因为它是在实例化作业之前定义任务的唯一方式。
- 默认情况下，当作业内的所有任务都完成时，作业仍保持活动状态。 可以更改此行为，使作业在其中的所有任务完成时自动终止。 将作业的 **onAllTasksComplete** 属性（在 Batch .NET 中为 [OnAllTasksComplete][net_onalltaskscomplete]）设置为 *terminatejob*，可在作业的所有任务处于已完成状态时自动终止该作业。

    请注意，Batch 服务将 *没有* 任务的作业视为其所有任务都已完成。 因此，此选项往往与 [作业管理器任务](#job-manager-task)配合使用。 如果想要使用自动作业终止而不通过作业管理器终止，首先应该将新作业的 **onAllTasksComplete** 属性设置为 *noaction*，然后只有在完成将任务添加到作业之后才将它设置为 *terminatejob*。

### <a name="job-priority"></a>作业优先级
可以向你在 Batch 中创建的作业分配优先级。 Batch 服务使用作业的优先级值来确定帐户中的作业计划顺序（不要与 [计划的作业](#scheduled-jobs)相混淆）。 优先级值的范围为 -1000 到 1000，-1000 表示最低优先级，1000 表示最高优先级。 若要更新作业的优先级，请调用[更新作业的属性][rest_update_job]操作 (Batch REST) 或修改 [CloudJob.Priority][net_cloudjob_priority] 属性 (Batch .NET)。

在同一个帐户内，高优先级作业的计划优先顺序高于低优先级作业。 一个帐户中具有较高优先级值的作业，其计划优先级并不高于不同帐户中较低优先级值的另一个作业。

不同池的作业计划是独立的。 在不同的池之间，即使作业的优先级较高，如果其关联的池缺少空闲的节点，则不保证此作业优先计划。 在同一个池中，相同优先级的作业有相同的计划机会。

### <a name="scheduled-jobs"></a>计划的作业
[作业计划][rest_job_schedules] 可在 Batch 服务中创建周期性作业。 作业计划指定何时要运行作业，并包含要运行的作业的规范。 可以指定计划的持续时间（计划的持续时间和生效时间），以及在计划的时间段内创建作业的频率。

## <a name="task"></a>任务
任务是与作业关联的计算单位。 它在节点上运行。 任务将分配到节点以执行，或排入队列直到节点空闲。 简而言之，任务将在计算节点上运行一个或多个程序或脚本，以执行你需要完成的工作。

创建任务时，可以指定：

- 任务的 **命令行** 。 这是可在计算节点上运行应用程序或脚本的命令行。

    请务必注意，命令行实际上不是在 shell 下运行。 因此无法以本机方式利用 shell 功能，例如[环境变量](#environment-settings-for-tasks)扩展（包括 `PATH`）。 若要利用此类功能，必须在命令行中调用 shell - 例如，在 Windows 节点上启动 `cmd.exe`，或者在 Linux 上启动 `/bin/sh`：

    `cmd /c MyTaskApplication.exe %MY_ENV_VAR%`

    `/bin/sh -c MyTaskApplication $MY_ENV_VAR`

    如果任务需要运行不在节点的 `PATH` 中的应用程序或脚本，或在引用环境变量，请在任务命令行中显式调用 shell。
- **资源文件** 。 在执行任务的命令行之前，这些文件将自动从常规用途 Azure 存储帐户中的 Blob 存储复制到节点。 有关详细信息，请参阅下面的[启动任务](#start-task)与[文件和目录](#files-and-directories)部分。
- 应用程序所需的 **环境变量** 。 有关详细信息，请参阅下面的 [任务的环境设置](#environment-settings-for-tasks) 部分。
- 执行任务所依据的 **约束** 。 例如，约束包括允许任务运行的最长时间、重试任务失败的次数上限，以及保留任务工作目录中的文件的最长时间。
- **Application packages** 。 [应用程序包](#application-packages) 提供任务运行的应用程序的简化部署和版本控制。 在共享池的环境中，任务级应用程序包特别有用：不同的作业在一个池上运行，完成某个作业时不删除该池。 如果作业中的任务少于池中的节点，任务应用程序包可以减少数据传输，因为应用程序只部署到运行任务的节点。

除了可以定义在节点上运行计算的任务以外，Batch 服务还提供以下特殊任务：

- [启动任务](#start-task)
- [作业管理器任务](#job-manager-task)
- [作业准备和释放任务](#job-preparation-and-release-tasks)
- [多实例任务 (MPI)](#multi-instance-tasks)
- [任务依赖项](#task-dependencies)

### <a name="start-task"></a>启动任务
通过将 **启动任务** 与池相关联，可以准备池节点的操作环境。 例如，可以执行诸如安装任务所要运行的应用程序或启动后台进程等操作。 启动任务在节点每次启动时运行，且只要保留在池中就会持续运行（包括首次将节点添加到池时，以及节点重新启动或重置映像时）。

启动任务的主要优点是可以包含全部所需的信息，使你能够配置计算节点，以及安装执行任务所需的应用程序。 因此，增加池中的节点数和指定新的目标节点计数一样简单。 启动任务向 Batch 服务提供配置新节点并使其准备好接受任务所需的信息。

与任何 Azure Batch 任务一样，除了指定要执行的**命令行**以外，还可以指定 [Azure 存储][azure_storage]中的**资源文件**列表。 Batch 服务先将资源文件从 Azure 存储复制到节点，然后运行命令行。 对于池启动任务，文件列表通常包含任务应用程序及其依赖项。

但是，启动任务还可能包含计算节点上运行的所有任务使用的引用数据。 例如，启动任务的命令行可执行 `robocopy` 操作，将应用程序文件（已指定为资源文件并下载到节点）从启动任务的[工作目录](#files-and-directories)复制到[共享文件夹](#files-and-directories)，然后然后运行 MSI 或 `setup.exe`。

> [AZURE.IMPORTANT]
> 批处理目前*仅*支持**常规用途**存储帐户类型，如 [About Azure storage accounts](/documentation/articles/storage-create-storage-account/)（关于 Azure 存储帐户）的 [Create a storage account](/documentation/articles/storage-create-storage-account/#create-a-storage-account/)（创建存储帐户）中步骤 5 所述。 Batch 任务（包括标准任务、启动任务、作业准备任务和作业释放任务） *只能* 指定位于 **常规用途** 存储帐户中的资源文件。
>
>

通常，Batch 服务需要等待启动任务完成，然后认为节点已准备好分配任务，但你可以配置这种行为。

如果某个计算节点上的启动任务失败，则节点的状态将会更新以反映失败状态，同时，不会为该节点分配任何任务。 如果从存储中复制启动任务的资源文件时出现问题，或由其命令行执行的进程返回了非零退出代码，则启动任务可能会失败。

如果添加或更新 *现有* 池的启动任务，必须重新启动其计算节点，启动任务才应用到节点。

### <a name="job-manager-task"></a>作业管理器任务
通常使用 **作业管理器任务** 来控制和/或监视作业的执行 - 例如，创建和提交作业的任务、确定其他要运行的任务，以及确定任务何时完成。 但是，作业管理器任务并不限定于这些活动。 它是功能齐备的任务，可执行作业所需的任何操作。 例如，作业管理器任务可以下载指定为参数的文件、分析该文件的内容，并根据这些内容提交其他任务。

作业管理员任务在所有其他任务之前启动。 它提供以下功能：

- 创建作业时由 Batch 服务自动提交为任务。
- 安排在作业中的其他任务之前执行。
- 缩小池时，关联的节点最后才从池中删除。
- 此终止可能完全取决于作业中的所有任务终止。
- 需要重新启动时，作业管理器任务有最高的优先级。 如果找不到空闲的节点，Batch 服务可以终止池中正在运行的其他某个任务，以便腾出空间供作业管理器任务运行。
- 一个作业中的作业管理器任务的优先级不高于其他作业的任务。 不同作业之间只遵循作业级别的优先级。

### <a name="job-preparation-and-release-tasks"></a>作业准备和释放任务
Batch 提供作业准备任务来设置作业前的执行。 作业释放任务用于作业后的维护或清理。

- **作业准备任务**：在任何其他作业任务执行之前，作业准备任务在计划要运行任务的所有计算节点上运行。 可使用作业准备任务，复制所有任务共享的、但对作业而言唯一的数据。
- **作业释放任务**：作业完成后，作业释放任务将在池中至少运行了一个任务的每个节点上运行。 可使用作业释放任务，删除作业准备任务所复制的数据，或压缩并上传诊断日志数据。

作业准备和释放任务允许指定调用任务时要运行的命令行。 这些任务提供许多功能，例如文件下载、以提升权限方式执行、自定义环境变量、最大执行持续时间、重试计数和文件保留时间。

有关作业准备和释放任务的详细信息，请参阅 [在 Azure Batch 计算节点上运行作业准备和完成任务](/documentation/articles/batch-job-prep-release/)。

### <a name="multi-instance-task"></a>多实例任务
[多实例任务](/documentation/articles/batch-mpi/) 是经过配置后可以在多个计算节点上同时运行的任务。 通过多实例任务，可以启用等高性能计算方案（例如消息传递接口 (MPI)），此类方案需要将一组计算节点分配到一起来处理单个工作负荷。

有关在 Batch 中使用 Batch .NET 库运行 MPI 作业的详细介绍，请参阅 [Use multi-instance tasks to run Message Passing Interface (MPI) applications in Azure Batch](/documentation/articles/batch-mpi/)（在 Azure Batch 中使用多实例任务来执行消息传递接口 (MPI) 应用程序）。

### <a name="task-dependencies"></a>任务依赖项
顾名思义，使用[任务依赖项](/documentation/articles/batch-task-dependencies/)可以在执行某个任务之前，指定该任务与其他任务的依赖性。 此功能提供以下情况的支持：“下游”任务取用“上游”任务的输出，或当上游任务执行下游任务所需的某种初始化时。 若要使用此功能，必须先在 Batch 作业中启用任务依赖性。 然后，针对每个依赖于另一个任务（或其他许多任务）的任务，指定该任务依赖的任务。

使用任务依赖性，可以配置如下所述的方案：

- *taskB* 依赖于 *taskA*（直到 *taskA* 完成，才开始执行 *taskB*）。
- *taskC* 同时依赖于 *taskA* 和 *taskB*。
- *taskD* 在执行前依赖于某个范围的任务，例如任务 *1* 到 *10*。

有关此功能的更深入信息，请查看 [Azure Batch 中的任务依赖关系](/documentation/articles/batch-task-dependencies/)和 [azure-batch-samples][github_samples] GitHub 存储库中的 [TaskDependencies][github_sample_taskdeps] 代码示例。

## <a name="environment-settings-for-tasks"></a>任务的环境设置
批处理服务执行的每个任务都可以访问在计算节点上设置的环境变量。 这包括 Batch 服务定义的（[服务定义型][msdn_env_vars]）环境变量以及用户可以针对其任务定义的自定义环境变量。 任务执行的应用程序和脚本可以在执行期间访问这些环境变量。

可以通过填充这些实体的 *环境设置* 属性，在任务或作业级别设置自定义环境变量。 有关示例，请参阅[将任务添加到作业][rest_add_task]操作 (Batch REST API)，或 Batch .NET 中的 [CloudTask.EnvironmentSettings][net_cloudtask_env] 和 [CloudJob.CommonEnvironmentSettings][net_job_env] 属性。

客户端应用程序或服务可使用[获取有关任务的信息][rest_get_task_info]操作 (Batch REST) 或通过访问 [CloudTask.EnvironmentSettings][net_cloudtask_env] 属性 (Batch .NET)，来获取任务的环境变量（服务定义型和自定义环境变量）。 在计算节点上执行的进程可以在节点上访问这些和其他环境变量，例如，通过使用熟悉的 `%VARIABLE_NAME%` (Windows) 或 `$VARIABLE_NAME` (Linux) 语法。

可以在[计算节点环境变量][msdn_env_vars]中找到包含所有服务定义型环境变量的完整列表。

## <a name="files-and-directories"></a>文件和目录
每个任务都有一个 *工作目录* ，任务将在该目录中创建零个或多个文件和目录。 此工作目录可用于存储任务运行的程序、任务处理的数据，以及任务执行的处理的输出。 任务的所有文件和目录由任务用户拥有。

Batch 服务在节点上公开文件系统的一部分作为 *根目录*。 任务可通过引用 `AZ_BATCH_NODE_ROOT_DIR` 环境变量来访问根目录。 有关使用环境变量的详细信息，请参阅 [任务的环境设置](#environment-settings-for-tasks)。

根目录包含以下目录结构：

![计算节点目录结构][1]

- **共享**：此目录允许对节点上运行的 *所有* 任务进行读取/写入访问。 在节点上运行的任何任务都可以创建、读取、更新和删除此目录中的文件。 任务可通过引用 `AZ_BATCH_NODE_SHARED_DIR` 环境变量来访问此目录。
- **启动**：启动任务使用此目录作为它的工作目录。 由启动任务下载到的节点所有文件都存储在此处。 启动任务可以创建、读取、更新和删除此目录下的文件。 任务可通过引用 `AZ_BATCH_NODE_STARTUP_DIR` 环境变量来访问此目录。
- **任务**：为节点上运行的每个任务创建一个目录。 可通过引用 `AZ_BATCH_TASK_DIR` 环境变量来访问该目录。

    在每个任务目录中，Batch 服务将创建由 `AZ_BATCH_TASK_WORKING_DIR` 环境变量指定唯一路径的任务目录 (`wd`)。 此目录提供对任务的读/写访问权限。 任务可以创建、读取、更新和删除此目录下的文件。 此目录根据指定给任务的 *RetentionTime* 约束来保留。

    `stdout.txt` 和 `stderr.txt`：在任务执行期间，会将这些文件写入任务文件夹。

> [AZURE.IMPORTANT]
> 从池中删除节点时，也会删除节点上存储的 *所有* 文件。
>
>

## <a name="application-packages"></a>应用程序包
[应用程序包](/documentation/articles/batch-application-packages/) 功能可为池中的计算节点提供简单的应用程序管理和部署能力。 可上传和管理任务所运行应用程序的多个版本，包括二进制文件和支持文件。 然后可以将一个或多个这样的应用程序自动部署到池中的计算节点。

可以在池和任务级别指定应用程序包。 指定池应用程序包时，应用程序将部署到池中的每个节点。 指定任务应用程序包时，应用程序只在运行任务的命令行之前，部署到计划要运行作业的至少一个任务的节点。

Batch 可以处理使用 Azure 存储将应用程序包存储及部署到计算节点的详细信息，因此可以简化代码和管理开销。

若要了解应用程序包功能的详细信息，请参阅 [Application deployment with Azure Batch application packages](/documentation/articles/batch-application-packages/)（使用 Azure Batch 应用程序包部署应用程序）。

> [AZURE.NOTE]
> 如果将池应用程序包添加到 *现有* 池，则必须重新启动其计算节点，应用程序包才会应用到节点。
>
>

## <a name="pool-and-compute-node-lifetime"></a>池和计算节点生存期
在设计 Azure Batch 解决方案时，必须做出有关如何及何时创建池，以及这些池中的计算节点可用性要保持多久的设计决策。

在极端情况下，可以针对提交的每个作业创建一个池，并在其任务执行完成时立即删除该池。 这样，只有在需要时才分配节点，节点空闲时会立即关闭，因此可以最高程度地提高利用率。 这意味着作业必须等待系统分配节点，但需注意，一旦节点单独可用，处于已分配状态且启动任务已完成，系统就会立即安排任务执行。 批处理 *不会* 等到池中的所有节点都可用后才将任务分配给节点。 这可确保最大程度地利用所有可用节点。

在另一种极端情况下，如果最高优先级是让作业立即启动，则你可以预先创建池，并使其节点在提交作业之前可用。 在此情况下，任务可以立即启动，但节点可能会保持空闲状态以等待分配任务。

通常会使用一种组合方法来处理可变但持续存在的负载。 可以创建一个池用于容纳提交的多个作业，但同时根据作业负载扩展或缩减节点数目（请参阅下一部分中的 [缩放计算资源](#scaling-compute-resources) ）。 可以根据当前负载被动执行此操作，或者在负载可预测时主动执行此操作。

## <a name="pool-network-configuration"></a>池网络配置

在 Azure 批处理中创建计算节点池时，可以使用 API 指定应在其中创建池计算节点的 Azure [虚拟网络 (VNet)](/documentation/articles/virtual-networks-overview/) 的 ID。

- VNet 必须满足以下条件：

   - 与 Azure 批处理帐户位于同一 Azure **区域** 。
   - 与 Azure 批处理帐户在同一**订阅**中。

- VNet 应该具有足够的可用 **IP 地址**以适应池的 `targetDedicated` 属性。 如果子网没有足够的可用 IP 地址，批处理服务将分配池中的部分计算节点，并返回调整大小错误。

- 指定的子网必须允许来自批处理服务的通信，才能在计算节点上计划任务。 如果与 VNet 关联的**网络安全组 (NSG)** 拒绝与计算节点通信，则批处理服务会将计算节点的状态设置为“不可用”。 

- 如果指定的 VNet 具有关联的 NSG，则必须启用入站通信。 就 Linux 池来说，端口 29876、29877 和 22 必须启用。 就 Windows 池来说，端口 3389 必须启用。

- *MicrosoftAzureBatch* 服务主体必须为指定的 VNet 提供[经典虚拟机参与者](/documentation/articles/role-based-access-built-in-roles/#classic-virtual-machine-contributor/)基于角色的访问控制 (RBAC) 角色。 在 Azure 门户中：

  - 选择“VNet”，然后单击“访问控制(IAM)” > “角色” > “经典虚拟机参与者” > “添加”
  - 在“搜索”框中输入“MicrosoftAzureBatch”
  - 选中“MicrosoftAzureBatch”复选框 
  - 选择“选择”按钮 


## <a name="scaling-compute-resources"></a>缩放计算资源
通过 [自动缩放](/documentation/articles/batch-automatic-scaling/)功能，可以让 Batch 服务根据计算方案的当前工作负荷和资源使用状况动态缩放池中的计算节点数目。 这样，便可做到只使用所需资源并可释放不需要的资源，因而能够降低运行应用程序的整体成本。

可通过编写 [自动缩放公式](/documentation/articles/batch-automatic-scaling/#automatic-scaling-formulas/) 并将该公式与池相关联，来启用自动缩放。 Batch 服务使用该公式来确定池中下一个缩放间隔（可配置的间隔）的目标节点数目。 可以在创建池时指定池的自动缩放设置，或稍后在池上启用缩放。 还可以更新已启用缩放的池上的缩放设置。

例如，也许某个作业需要提交大量任务以执行。 你可以将缩放公式分配到池，以根据当前的排队任务数和作业中任务的完成率来调整池中的节点数目。 Batch 服务定期评估公式，并根据工作负荷和其他公式设置来调整池的大小。 当有大量的排队任务时该服务会根据需要添加节点，当没有正在排队或运行的任务时则会删除节点。 

缩放公式可以基于以下度量值：

- **时间度量值** 基于指定的时数内每隔五分钟收集的统计信息。
- **资源度量值** 基于 CPU 使用率、带宽使用率、内存使用率和节点的数目。
- **任务指标**基于任务状态，例如“活动”（已排队）、“正在运行”或“已完成”。

如果自动缩放会减少池中的计算节点数，则必须考虑如何处理在执行减少操作时运行的任务。 为了满足这一点，Batch 提供可包含在公式中的 *节点解除分配选项* 。 例如，可以指定运行中的任务立即停止，立即停止然后重新排入队列以便在另一个节点上运行，或允许先完成再从池中删除节点。

有关自动缩放应用程序的详细信息，请参阅 [自动缩放 Azure Batch 池中的计算节点](/documentation/articles/batch-automatic-scaling/)。

> [AZURE.TIP]
> 若要获得最大的计算资源使用率，请将节点的目标数目设置成在作业结束时降为零，但允许正在运行的任务完成。
>
>

## <a name="security-with-certificates"></a>证书的安全性
在加密或解密任务的敏感信息（例如 [Azure 存储帐户][azure_storage]的密钥）时，通常需要使用证书。 为此，可以在节点上安装证书。 加密的机密通过命令行参数或内嵌在某个任务资源中来传递给任务，已安装的证书可用于解密机密。

可以使用[添加证书][rest_add_cert]操作 (Batch REST) 或 [CertificateOperations.CreateCertificate][net_create_cert] 方法 (Batch .NET) 将证书添加到 Batch 帐户。 然后，可以将该证书与新池或现有池相关联。 将证书与池关联后，Batch 服务将在池中的每个节点上安装该证书。 在启动节点之后、启动任何任务（包括启动任务和作业管理器任务）之前，Batch 服务将安装相应的证书。

如果将证书添加到 *现有* 池，必须重新启动其计算节点，证书才会应用到节点。

## <a name="error-handling"></a>错误处理。
有时你可能需要处理 Batch 解决方案中的任务和应用程序失败。

### <a name="task-failure-handling"></a>任务失败处理
任务失败划分为以下类别：

- **计划失败**

    如果为任务指定的文件的传输因任何原因而失败，将为该任务设置 *计划错误* 。

    如果任务的资源文件已移动、存储帐户不再可用，或者发生其他使文件无法成功复制到节点的问题，则可能会出现计划错误。
- **应用程序失败**

    任务命令行指定的进程也可能会失败。 如果任务执行的进程返回非零退出代码，则将该进程视为失败（请参阅下一部分中的 *任务退出代码* ）。

    对于应用程序失败，可以将 Batch 配置为自动重试任务，并最多重试指定的次数。
- **约束失败**

    可以设置一个约束来指定作业或任务的最大执行持续期间，即 *maxWallClockTime*。 此约束可用于终止未能继续进行的任务。

    如果超出了最长时间，则将任务标记为*已完成*，但退出代码将设置为 `0xC000013A`，*schedulingError* 字段将标记为 `{ category:"ServerError", code="TaskEnded"}`。

### <a name="debugging-application-failures"></a>调试应用程序失败
- `stderr` 和 `stdout`

    在执行过程中，应用程序可以生成诊断输出，这些信息可用于排查问题。 如前一部分[文件和目录](#files-and-directories)中所述，批处理服务会将标准输出和标准错误输出发送到计算节点上的任务目录中的 `stdout.txt` 和 `stderr.txt` 文件。 可以使用 Azure 门户或 Batch SDK 之一下载这些文件。 例如，可以使用 Batch .NET 库中的 [ComputeNode.GetNodeFile][net_getfile_node] 和 [CloudTask.GetNodeFile][net_getfile_task] 检索这些文件和其他文件来进行故障排除。
- **任务退出代码**

    如前所述，如果任务执行的程序返回非零退出代码，则 Batch 服务会将此任务标记为失败。 当任务执行某个进程时，Batch 将使用 *进程的返回代码*填充任务的退出代码属性。 请务必注意，任务的退出代码**不是**由 Batch 服务确定， 而是由进程本身或此进程在其上运行的操作系统确定。

### <a name="accounting-for-task-failures-or-interruptions"></a>应对任务失败或中断
任务偶尔会失败或中断。 任务应用程序本身可能会失败，运行任务的节点可能会重新启动，或者在调整大小操作期间，可能会因为池的取消分配策略设置为在不等待任务完成的情况下立即删除节点，而从池中删除节点。 在所有情况下，任务都可以由 Batch 自动排队，并在另一个节点上执行。

间歇性的问题也有可能会导致任务挂起，或者花费很长时间才能完成执行。 可为任务设置最长执行间隔时间。 如果超出最长执行间隔时间，Batch 服务会中断任务应用程序。

### <a name="connecting-to-compute-nodes"></a>连接到计算节点
可通过远程登录到计算节点来进一步执行调试和故障排除。 可以使用 Azure 门户下载 Windows 节点的远程桌面协议 (RDP) 文件，并获取 Linux 节点的安全外壳 (SSH) 连接信息。 也可以使用 Batch API（例如，使用 [Batch .NET][net_rdpfile] 或 [Batch Python](/documentation/articles/batch-linux-nodes/#connect-to-linux-nodes/)）执行此操作。

> [AZURE.IMPORTANT]
> 若要通过 RDP 或 SSH 连接到某个节点，必须先在该节点上创建一个用户。 为此，可以使用 Azure 门户通过 Batch REST API [将用户帐户添加到节点][rest_create_user]、在 Batch .NET 中调用 [ComputeNode.CreateComputeNodeUser][net_create_user] 方法，或在 Batch Python 模块中调用 [add_user][py_add_user] 方法。
>
>

### <a name="troubleshooting-problematic-compute-nodes"></a>对有问题的计算节点进行故障排除
在部分任务失败的情况下，Batch 客户端应用程序或服务可以检查失败任务的元数据来找出行为异常的节点。 池中的每个节点都有一个唯一 ID，运行任务的节点包含在任务元数据中。 识别出“有问题的节点”后，可对其执行多种操作：

- **重新启动节点** ([REST][rest_reboot] | [.NET][net_reboot])

    重新启动节点有时可以清除潜在的问题，例如进程停滞或崩溃。 请注意，如果池使用启动任务或作业使用作业准备任务，节点重新启动时将执行这些任务。
- **重置映像节点** ([REST][rest_reimage] | [.NET][net_reimage])

    这会在节点上重新安装操作系统。 和重新启动节点一样，在重置映像节点后，便重新执行启动任务和作业准备任务。
- **从池中删除节点** ([REST][rest_remove] | [.NET][net_remove])

    有时必须从池中完全删除节点。
- **禁用节点上的任务计划** ([REST][rest_offline] | [.NET][net_offline])

    这实际上是使节点脱机，以便不再收到任何分配的任务，但允许节点继续运行并保留在池中。 这可让你执行进一步的调查以了解失败原因，却又会不丢失失败任务的数据，并且不让节点造成额外的任务失败。 例如，可以禁用节点上的任务计划，然后从 [远程登录](#connecting-to-compute-nodes) 以检查节点的事件日志，或执行其他故障排除操作。 完成调查后，可以启用任务计划 ([REST][rest_online] | [.NET][net_online]) 使节点重新联机，或者执行上述其他操作。

> [AZURE.IMPORTANT]
> 可以使用本部分中所述的每项操作（重新启动、重置映像、删除和禁用任务计划），来指定当执行操作时要如何处理节点上当前正在运行的任务。 例如，禁用具有 Batch .NET 客户端库的节点上的任务计划时，可以指定 [DisableComputeNodeSchedulingOption][net_offline_option] 枚举值，以指定是要**终止**运行中的任务、将任务**重新排队**以在其他节点上计划，还是允许执行中的任务先完成再执行操作 (**TaskCompletion**)。
>
>

## <a name="next-steps"></a>后续步骤
- 了解适用于生成批处理解决方案的[批处理 API 和工具](/documentation/articles/batch-apis-tools/)。
- 在 [Get started with the Azure Batch Library for .NET](/documentation/articles/batch-dotnet-get-started/)（适用于 .NET 的 Azure Batch 库入门）中逐步演练一个示例 Batch 应用程序。 另请参阅该教程的 [Python 版本](/documentation/articles/batch-python-tutorial/) ，其中介绍了如何在 Linux 计算节点上运行工作负荷。
- 下载并构建 [Batch 资源管理器][github_batchexplorer] 示例项目，以便在开发 Batch 解决方案时使用。 使用 Batch 资源管理器可以执行以下和其他操作：

  - 监视和管理 Batch 帐户中的池、作业与任务
  - 从节点下载 `stdout.txt`、`stderr.txt` 和其他文件
  - 在节点上创建用户，并下载用于远程登录的 RDP 文件
- 了解如何 [创建 Linux 计算节点池](/documentation/articles/batch-linux-nodes/)。
- 访问 MSDN 上的 [Azure Batch 论坛][batch_forum] 。 无论你是新手还是 Batch 专家，该论坛都是一个提问的好去处。

[1]: ./media/batch-api-basics/node-folder-structure.png

[azure_storage]: /home/features/storage/
[batch_forum]: https://social.msdn.microsoft.com/Forums/zh-cn/home?forum=azurebatch
[cloud_service_sizes]: /documentation/articles/cloud-services-sizes-specs/
[msmpi]: https://msdn.microsoft.com/zh-cn/library/bb524831.aspx
[github_samples]: https://github.com/Azure/azure-batch-samples
[github_sample_taskdeps]:  https://github.com/Azure/azure-batch-samples/tree/master/CSharp/ArticleProjects/TaskDependencies
[github_batchexplorer]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/BatchExplorer
[batch_net_api]: https://msdn.microsoft.com/zh-cn/library/azure/mt348682.aspx
[msdn_env_vars]: https://msdn.microsoft.com/zh-cn/library/azure/mt743623.aspx
[net_cloudjob_jobmanagertask]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.cloudjob.jobmanagertask.aspx
[net_cloudjob_priority]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.cloudjob.priority.aspx
[net_cloudpool_starttask]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.cloudpool.starttask.aspx
[net_cloudtask_env]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.cloudtask.environmentsettings.aspx
[net_create_cert]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.certificateoperations.createcertificate.aspx
[net_create_user]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.computenode.createcomputenodeuser.aspx
[net_getfile_node]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.computenode.getnodefile.aspx
[net_getfile_task]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.cloudtask.getnodefile.aspx
[net_job_env]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.cloudjob.commonenvironmentsettings.aspx
[net_multiinstancesettings]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.multiinstancesettings.aspx
[net_onalltaskscomplete]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.cloudjob.onalltaskscomplete.aspx
[net_rdp]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.computenode.getrdpfile.aspx
[net_reboot]: https://msdn.microsoft.com/zh-cn/library/azure/mt631495.aspx
[net_reimage]: https://msdn.microsoft.com/zh-cn/library/azure/mt631496.aspx
[net_remove]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.pooloperations.removefrompoolasync.aspx
[net_offline]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.computenode.disableschedulingasync.aspx
[net_online]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.computenode.enableschedulingasync.aspx
[net_offline_option]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.common.disablecomputenodeschedulingoption.aspx
[net_rdpfile]: https://msdn.microsoft.com/zh-cn/library/azure/Mt272127.aspx
[vnet]: https://msdn.microsoft.com/zh-cn/library/azure/dn820174.aspx#bk_netconf

[py_add_user]: http://azure-sdk-for-python.readthedocs.io/en/latest/ref/azure.batch.operations.html#azure.batch.operations.ComputeNodeOperations.add_user

[batch_rest_api]: https://msdn.microsoft.com/zh-cn/library/azure/Dn820158.aspx
[rest_add_job]: https://msdn.microsoft.com/zh-cn/library/azure/mt282178.aspx
[rest_add_pool]: https://msdn.microsoft.com/zh-cn/library/azure/dn820174.aspx
[rest_add_cert]: https://msdn.microsoft.com/zh-cn/library/azure/dn820169.aspx
[rest_add_task]: https://msdn.microsoft.com/zh-cn/library/azure/dn820105.aspx
[rest_create_user]: https://msdn.microsoft.com/zh-cn/library/azure/dn820137.aspx
[rest_get_task_info]: https://msdn.microsoft.com/zh-cn/library/azure/dn820133.aspx
[rest_job_schedules]: https://msdn.microsoft.com/zh-cn/library/azure/mt282179.aspx
[rest_multiinstance]: https://msdn.microsoft.com/zh-cn/library/azure/mt637905.aspx
[rest_multiinstancesettings]: https://msdn.microsoft.com/zh-cn/library/azure/dn820105.aspx#multiInstanceSettings
[rest_update_job]: https://msdn.microsoft.com/zh-cn/library/azure/dn820162.aspx
[rest_rdp]: https://msdn.microsoft.com/zh-cn/library/azure/dn820120.aspx
[rest_reboot]: https://msdn.microsoft.com/zh-cn/library/azure/dn820171.aspx
[rest_reimage]: https://msdn.microsoft.com/zh-cn/library/azure/dn820157.aspx
[rest_remove]: https://msdn.microsoft.com/zh-cn/library/azure/dn820194.aspx
[rest_offline]: https://msdn.microsoft.com/zh-cn/library/azure/mt637904.aspx
[rest_online]: https://msdn.microsoft.com/zh-cn/library/azure/mt637907.aspx

[vm_marketplace]: https://azure.microsoft.com/marketplace/virtual-machines/

<!---Update_Description: wording update -->