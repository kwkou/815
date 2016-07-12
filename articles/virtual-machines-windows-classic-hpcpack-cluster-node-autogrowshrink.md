<properties
 pageTitle="自动缩放 HPC 群集中的计算资源 | Azure"
 description="自动扩展和收缩 Azure 中的 HPC Pack 群集计算节点数"
 services="virtual-machines-windows"
 documentationCenter=""
 authors="dlepow"
 manager="timlt"
 editor=""
 tags="azure-service-management,hpc-pack"/>
<tags
	ms.service="virtual-machines-windows"
	ms.date="04/14/2016"
	wacn.date="05/16/2016"/>

# 在 Azure 中根据群集工作负荷自动扩展和收缩 HPC Pack 群集资源




如果在 HPC Pack 群集中部署 Azure“突发”节点，或者在 Azure VM 中创建 HPC Pack 群集，你可能希望有一种方法能够根据群集上的当前工作负荷自动增大或减小 Azure 计算资源（例如核心）的数目。这可让你更有效地使用你的 Azure 资源并控制其成本。为此，可以设置 HPC Pack 群集属性 **AutoGrowShrink**。或者，使用随 HPC Pack 一起安装的 **AzureAutoGrowShrink.ps1** HPC PowerShell 脚本。

> [AZURE.IMPORTANT] Azure 具有用于创建和处理资源的两个不同的部署模型：[资源管理器和经典](/documentation/articles/resource-manager-deployment-model/)。本文介绍使用经典部署模型。Azure 建议大多数新部署使用资源管理器模型。

## 设置 AutoGrowShrink 群集属性

### 先决条件

* **HPC Pack 2012 R2 Update 2 或更高版群集** - 群集头节点既可以部署在本地，也可以部署在 Azure VM 中。请参阅[使用 HPC Pack 设置一个混合群集](/documentation/articles/cloud-services-setup-hybrid-hpcpack-cluster/)，开始使用本地头节点和 Azure“突发”节点。若要在 Azure VM 中快速部署 HPC Pack 群集，请参阅 [HPC Pack IaaS deployment script](/documentation/articles/virtual-machines-windows-classic-hpcpack-cluster-powershell-script/)（HPC Pack IaaS 部署脚本）。


* **对于 Azure 中包含头节点的群集** - 如果你使用 HPC Pack IaaS 部署脚本来创建群集，请通过在群集配置文件中设置 AutoGrowShrink 选项来启用 **AutoGrowShrink** 群集属性。有关详细信息，请[下载的脚本](https://www.microsoft.com/download/details.aspx?id=44949)随附的文档。

    或者，使用下一部分所述的 HPC PowerShell 命令在部署群集后设置 **AutoGrowShrink** 群集属性。若要使用 HPC PowerShell 执行此操作，请先完成以下步骤：
    1. 在头节点和 Azure 订阅上配置 Azure 管理证书。对于测试部署，可以使用 HPC Pack 在头节点上安装的默认 Microsoft HPC Azure 自签名证书，并只将该证书上载到 Azure 订阅。有关选项和步骤，请参阅 [TechNet 库指南](https://technet.microsoft.com/zh-cn/library/gg481759.aspx)。
    2. 在头节点上运行 **regedit**，转到 HKLM\\SOFTWARE\\Micorsoft\\HPC\\IaasInfo，然后添加新字符串值。将“值”名称设置为“ThumbPrint”，将“值”数据设置为步骤 1 中的证书指纹。


### 用于设置 AutoGrowShrink 属性的 HPC PowerShell 命令

下面是 HPC PowerShell 命令示例，它们可以设置 **AutoGrowShrink** 并使用其他参数来优化其行为。请参阅本文稍后的 [AutoGrowShrink 参数](#AutoGrowShrink-parameters)以获取完整的设置列表。

若要运行这些命令，请在群集头节点上以管理员身份启动 HPC PowerShell。

**启用 AutoGrowShrink 属性**

    Set-HpcClusterProperty -EnableGrowShrink 1

**禁用 AutoGrowShrink 属性**

    Set-HpcClusterProperty -EnableGrowShrink 0

**更改扩展间隔（以分钟为单位）**

    Set-HpcClusterProperty -GrowInterval <interval>

**更改收缩间隔（以分钟为单位）**

    Set-HpcClusterProperty -ShrinkInterval <interval>

**查看 AutoGrowShrink 的当前配置**

    Get-HpcClusterProperty -AutoGrowShrink

###<a name="AutoGrowShrink-parameters"></a> AutoGrowShrink 参数

下面是可以使用 **Set-HpcClusterProperty** 命令修改的 AutoGrowShrink 参数。

* **EnableGrowShrink** - 用于启用或禁用 **AutoGrowShrink** 属性的开关。
* **ParamSweepTasksPerCore** - 用于扩展一个核心的参数扫描任务数目。默认为每个任务扩展一个核心。 
 
    >[AZURE.NOTE] HPC Pack QFE KB3134307 将 **ParamSweepTasksPerCore** 更改为 **TasksPerResourceUnit**。它基于作业资源类型，并且可以是节点、套接字或核心。
    
* **GrowThreshold** - 用于触发自动扩展的排队任务的阈值。默认值为 1，这意味着，如果有 1 个或多个任务处于排队状态，将自动扩展节点。
* **GrowInterval** - 触发自动扩展的间隔，以分钟为单位。默认间隔为 5 分钟。
* **ShrinkInterval** - 触发自动收缩的间隔，以分钟为单位。默认间隔为 5 分钟。|
* **ShrinkIdleTimes** - 指示节点为空闲状态之前，持续检查收缩的次数。默认值为 3 次。例如，如果 **ShrinkInterval** 为 5 分钟，HPC Pack 将每隔 5 分钟检查节点是否处于空闲状态。如果节点连续 3 次检查（15 分钟）都处于空闲状态，HPC Pack 将收缩该节点。
* **ExtraNodesGrowRatio** - 要为消息传递接口 (MPI) 作业扩展的节点附加百分比。默认值为 1，表示 HPC Pack 针对 MPI 作业扩展节点 1%。 
* **GrowByMin** - 用于指示自动扩展策略是否基于作业所需的最少资源的开关。默认值为 false，这意味着 HPC Pack 将基于作业所需的资源数上限为作业扩展节点。
* **SoaJobGrowThreshold** - 传入 SOA 请求以触发自动扩展过程的阈值。默认值为 50000。  
    
    >[AZURE.NOTE] 从 HPC Pack 2012 R2 Update 3 开始支持此参数。
    
* **SoaRequestsPerCore** - 用于扩展一个核心的传入 SOA 请求数目。默认值为 20000。

    >[AZURE.NOTE] 从 HPC Pack 2012 R2 Update 3 开始支持此参数。

### MPI 示例

默认情况下，HPC Pack 针对 MPI 作业额外扩展 1% 的节点（**ExtraNodesGrowRatio** 设置为 1）。原因是 MPI 可能需要多个节点，且只有在准备好所有节点时才能运行该作业。当 Azure 启动节点时，一个节点偶尔可能需要比其他节点更多的时间才能启动，使得其他节点在等待该节点就绪前空闲。通过扩展附加节点，HPC Pack 可减少此资源等待时间，并可能会节省成本。若要扩展 MPI 操作的附加节点百分比（例如，10%），请运行类似于下面的命令

    Set-HpcClusterProperty -ExtraNodesGrowRatio 10

### SOA 示例

默认情况下，**SoaJobGrowThreshold** 设置为 50000，**SoaRequestsPerCore** 设置为 200000。如果提交一个包含 70000 个请求的 SOA 作业，则会有一个排队任务，并且传入请求数为 70000。在此情况下，HPC Pack 将会针对排队任务扩展 1 个核心，并针对传入请求扩展 (70000 - 50000)/20000 = 1 个核心，因此总共为此 SOA 作业扩展了 2 个核心。

## 运行 AzureAutoGrowShrink.ps1 脚本

### 先决条件

* **HPC Pack 2012 R2 Update 1 或更高版本群集** - **AzureAutoGrowShrink.ps1** 脚本已安装在 %CCP\_HOME%bin 文件夹中。群集头节点既可以部署在本地，也可以部署在 Azure VM 中。请参阅[使用 HPC Pack 设置一个混合群集](/documentation/articles/cloud-services-setup-hybrid-hpcpack-cluster/)，开始使用本地头节点和 Azure“突发”节点。若要在 Azure VM 中快速部署 HPC Pack 群集，请参阅 [HPC Pack IaaS deployment script](/documentation/articles/virtual-machines-windows-classic-hpcpack-cluster-powershell-script/)（HPC Pack IaaS 部署脚本）。

* **Azure PowerShell 0.8.12** - 该脚本当前依赖于此特定版本的 Azure PowerShell。如果要在头节点上运行更高版本，可能需要将 Azure PowerShell 降级到[版本 0.8.12](http://az412849.vo.msecnd.net/downloads03/azure-powershell.0.8.12.msi) 才能运行该脚本。

* **对于具有 Azure 突发节点的群集** - 在安装了 HPC Pack 的客户端计算机上或在头节点上运行脚本。如果在客户端计算机上运行，请确保正确设置变量 $env:CCP\_SCHEDULER 以指向头节点。Azure“突发”节点必须已添加到群集，但是其可能处于“未部署”状态。


* **对于 Azure VM 中部署的群集** - 请在头节点 VM 上运行脚本，因为相关的 **Start-HpcIaaSNode.ps1** 和 **Stop-HpcIaaSNode.ps1** 脚本安装在这个位置。那些脚本还需要 Azure 管理证书或发布设置文件（请参阅[管理 Azure 的 HPC Pack 群集中的计算节点](/documentation/articles/virtual-machines-windows-classic-hpcpack-cluster-node-manage/)）。请确保你需要的所有计算节点 VM 都已添加到群集。它们可能处于停止状态。

### 语法

	AzureAutoGrowShrink.ps1
	[[-NodeTemplates] <String[]>] [[-JobTemplates] <String[]>] [[-NodeType] <String>]
	[[-NumOfQueuedJobsPerNodeToGrow] <Int32>]
	[[-NumOfQueuedJobsToGrowThreshold] <Int32>]
	[[-NumOfActiveQueuedTasksPerNodeToGrow] <Int32>]
	[[-NumOfActiveQueuedTasksToGrowThreshold] <Int32>]
	[[-NumOfInitialNodesToGrow] <Int32>] [[-GrowCheckIntervalMins] <Int32>]
	[[-ShrinkCheckIntervalMins] <Int32>] [[-ShrinkCheckIdleTimes] <Int32>]
	[-UseLastConfigurations] [[-ArgFile] <String>] [[-LogFilePrefix] <String>]
	[<CommonParameters>]

### Parameters

 * **NodeTemplates** - 节点模板名称，可定义节点增加和减少的范围。如果没有指定（默认值是 @()），则在 **NodeType** 的值为 AzureNodes 时，**AzureNodes** 节点组中的所有节点都在范围内，在 **NodeType** 的值为 ComputeNodes 时，**ComputeNodes** 节点组中的所有节点都在范围内。

 * **JobTemplates** - 作业模板的名称，可定义节点增加的范围。

 * **NodeType** - 增加和减少的节点类型。支持的值是：

     * **AzureNodes** - 适用于本地群集或 Azure IaaS 群集的 Azure PaaS（迸发）节点。

     * **ComputeNodes** - 仅适用于 Azure IaaS 群集的计算节点 VM。

* **NumOfQueuedJobsPerNodeToGrow** - 要增加一个节点所需的已排队作业数量。

* **NumOfQueuedJobsToGrowThreshold** - 要开始增加流程的已排队作业的阈值数量。

* **NumOfActiveQueuedTasksPerNodeToGrow** - 要增加一个节点所需的活动已排队任务数量。如果 **NumOfQueuedJobsPerNodeToGrow** 指定为大于 0 的值，则忽略此参数。

* **NumOfActiveQueuedTasksToGrowThreshold**- 要开始增加流程的活动已排队任务的阈值数量。

* **NumOfInitialNodesToGrow** - 要增加节点的初始最小数量，如果范围内的所有节点都处于**未部署**或**已停止（已取消分配）**状态。

* **GrowCheckIntervalMins** - 要增加检查之间的间隔（单位为分钟）。

* **ShrinkCheckIntervalMins** - 要减少检查之间的间隔（单位为分钟）。

* **ShrinkCheckIdleTimes** - 连续减少检查的数量（由 **ShrinkCheckIntervalMins** 分隔）表示节点处于闲置状态。

* **UseLastConfigurations** - 以前的配置保存在参数文件中。

* **ArgFile**- 用于保存和更新配置以运行脚本的参数文件的名称。

* **LogFilePrefix**- 日志文件的前缀名称。你可以指定一个路径。默认情况下，日志将写入当前工作目录。

### 示例 1

下面的示例将使用“默认 AzureNode 模板”部署的 Azure 突发节点配置为自动增加和减少。如果所有节点最初都处于**未部署**状态，则至少启动了 3 个节点。如果已排队作业的数量超过 8 个，则脚本会启动节点，直至节点数量超过已排队作业与 **NumOfQueuedJobsPerNodeToGrow** 的比。如果发现一个节点连续 3 次闲置，则会停止此节点。

	.\AzureAutoGrowShrink.ps1 -NodeTemplates @('Default AzureNodeTemplate') `
			-NodeType AzureNodes -NumOfQueuedJobsPerNodeToGrow 5 `
			-NumOfQueuedJobsToGrowThreshold 8 -NumOfInitialNodesToGrow 3 `
			-GrowCheckIntervalMins 1 -ShrinkCheckIntervalMins 1 -ShrinkCheckIdleTimes 3

### 示例 2

下面的示例将使用“默认 ComputeNode 模板”部署的 Azure 计算节点 VM 配置为自动增加和减少。由默认作业模板配置的作业可定义群集上工作负荷的范围。如果所有节点最初都处于已停止状态，则至少启动了 5 个节点。如果活动已排队任务的数量超过 15 个，则脚本会启动节点，直至节点数量超过活动已排队任务与 **NumOfActiveQueuedTasksPerNodeToGrow** 的比。如果发现一个节点连续 10 次闲置，则会停止此节点。

	.\AzureAutoGrowShrink.ps1 -NodeTemplates 'Default ComputeNode Template' -JobTemplates 'Default' -NodeType ComputeNodes -NumOfActiveQueuedTasksPerNodeToGrow 10 -NumOfActiveQueuedTasksToGrowThreshold 15 -NumOfInitialNodesToGrow 5 -GrowCheckIntervalMins 1 -ShrinkCheckIntervalMins 1 -ShrinkCheckIdleTimes 10 -ArgFile 'IaaSVMComputeNodes_Arg.xml' -LogFilePrefix 'IaaSVMComputeNodes_log'

<!---HONumber=Mooncake_0509_2016-->