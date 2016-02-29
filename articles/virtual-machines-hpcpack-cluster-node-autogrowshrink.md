<properties
 pageTitle="自动缩放 HPC 群集中的计算资源 | Microsoft Azure"
 description="了解自动增加和减少 Azure 的 HPC Pack 群集中的计算资源的方法"
 services="virtual-machines"
 documentationCenter=""
 authors="dlepow"
 manager="timlt"
 editor=""
 tags="azure-service-management,hpc-pack"/>
<tags
	ms.service="virtual-machines"
	ms.date="01/07/2016"
	wacn.date="02/26/2016"/>

# 在 HPC Pack 群集中根据群集工作负荷自动增加和减少 Azure 计算资源

[AZURE.INCLUDE [了解部署模型](../includes/learn-about-deployment-models-classic-include.md)]


如果在 HPC Pack 群集中部署 Azure“突发”节点，或者在 Azure VM 中创建 HPC Pack 群集，你可能希望有一种方法能够根据群集上作业及任务的当前工作负荷自动增加或减少 Azure 计算资源。这可让你更有效地使用你的 Azure 资源并控制其成本。为此，请使用随 HPC Pack安装的 **AzureAutoGrowShrink.ps1** HPC PowerShell 脚本。

>[AZURE.TIP] 从 HPC Pack 2012 R2 Update 2 开始，HPC Pack 包括可自动增加和减少 Azure 突发节点或 Azure VM 计算节点的一种内置服务。使用 [HPC Pack IaaS 部署脚本](/documentation/articles/virtual-machines-hpcpack-cluster-powershell-script)中的设置配置此服务，或者手动设置 **AutoGrowShrink** 群集属性。请参阅 [Microsoft HPC Pack 2012 R2 Update 2 中的新增功能](https://technet.microsoft.com/zh-cn/library/mt269417.aspx)。

## 先决条件

* **HPC Pack 2012 R2 Update 1 或更高版本群集** - **AzureAutoGrowShrink.ps1** 脚本已安装在 %CCP\_HOME%bin 文件夹中。群集头节点既可以部署在本地，也可以部署在 Azure VM 中。请参阅[使用 HPC Pack 设置一个混合群集](/documentation/articles/cloud-services-setup-hybrid-hpcpack-cluster)，开始使用本地头节点和 Azure“突发”节点。请参阅 [HPC Pack IaaS 部署脚本](/documentation/articles/virtual-machines-hpcpack-cluster-powershell-script)在 Azure VM 中快速部署 HPC Pack 群集。

* **Azure PowerShell 0.8.12** - 该脚本当前依赖于此特定版本的 Azure PowerShell。如果要在头节点上运行更高版本，可能需要将 Azure PowerShell 降级到[版本 0.8.12](http://az412849.vo.msecnd.net/downloads03/azure-powershell.0.8.12.msi) 才能运行该脚本。

* **对于具有 Azure 突发节点的群集** - 在安装了 HPC Pack 的客户端计算机上或在头节点上运行脚本。如果在客户端计算机上运行，请确保正确设置变量 $env:CCP\_SCHEDULER 以指向头节点。Azure“突发”节点必须已添加到群集，但是其可能处于“未部署”状态。


* **对于 Azure VM 中部署的群集** - 请在头节点 VM 上运行脚本，因为相关的 **Start-HpcIaaSNode.ps1** 和 **Stop-HpcIaaSNode.ps1** 脚本安装在这个位置。那些脚本还需要 Azure 管理证书或发布设置文件（请参阅[管理 Azure 的 HPC Pack 群集中的计算节点](/documentation/articles/virtual-machines-hpcpack-cluster-node-manage)）。请确保你需要的所有计算节点 VM 都已添加到群集。它们可能处于停止状态。

## 语法

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

## Parameters

 * **NodeTemplates** - 节点模板名称，可定义节点增加和减少的范围。如果没有指定（默认值是 @()），则在 **NodeType** 的值为 AzureNodes 时，**AzureNodes** 节点组中的所有节点都在范围内，在 **NodeType** 的值为 ComputeNodes 时，**ComputeNodes** 节点组中的所有节点都在范围内。

 * **JobTemplates** - 作业模板的名称，可定义节点增加的范围。

 * **NodeType** - 增加和减少的节点类型。支持的值是：

     * **AzureNodes** - 适用于本地群集或 Azure IaaS 群集的 Azure PaaS（突发）节点。

     * **ComputeNodes** - 仅适用于 Azure IaaS 群集的计算节点 VM。

* **NumOfQueuedJobsPerNodeToGrow** - 要增加一个节点所需的已排队作业数量。

* **NumOfQueuedJobsToGrowThreshold** - 要开始增加流程的已排队作业的阈值数量。

* **NumOfActiveQueuedTasksPerNodeToGrow** - 要增加一个节点所需的活动已排队任务数量。如果 **NumOfQueuedJobsPerNodeToGrow** 指定为大于 0 的值，则忽略此参数。

* **NumOfActiveQueuedTasksToGrowThreshold**- 要开始增加流程的活动已排队任务的阈值数量。

* **NumOfInitialNodesToGrow** - 要增加节点的初始最小数量，如果范围内的所有节点都处于**未部署**或**已停止（已取消分配）**状态。

* **GrowCheckIntervalMins** - 要增加检查之间的间隔（单位为分钟）。

* **ShrinkCheckIntervalMins** - 要减少检查之间的间隔（单位为分钟）。

* **ShrinkCheckIdleTimes**- 连续减少检查的数量（由 **ShrinkCheckIntervalMins** 分隔）表示节点处于闲置状态。

* **UseLastConfigurations*** 以前的配置保存在参数文件中。

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

<!---HONumber=Mooncake_0215_2016-->