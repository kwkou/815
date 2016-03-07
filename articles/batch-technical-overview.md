<properties
	pageTitle="Azure 批处理 ( Batch ) 服务基础知识 | Azure"
	description="了解适用于大规模并发工作负荷与 HPC 工作负荷的 Azure 批处理 ( Batch ) 服务的概念、工作流和方案"
	services="batch"
	documentationCenter=""
	authors="dlepow"
	manager="timlt"
	editor=""/>

<tags
	ms.service="batch"
	ms.date="10/26/2015"
	wacn.date="01/15/2016"/>

# Azure 批处理 ( Batch ) 基础知识

Azure 批处理 ( Batch )可帮助你在云中有效运行大规模并行和高性能计算 (HPC) 应用程序。它是一个平台服务，可以计划要在托管的虚拟机（计算节点）集合上运行的计算密集型工作，并且可以缩放计算资源以符合作业的需求。使用批处理 ( Batch ) 服务，你可以编程方式定义 Azure 计算资源以及根据需要或计划运行的大规模批处理 ( Batch ) 作业，而无需手动配置和管理 HPC 群集、单个 VM、虚拟网络或作业计划程序。

## 用例

批处理 ( Batch ) 是一种托管服务，可用于实现*批处理 ( Batch )*或*批量计算* - 运行大量类似任务以获取所需的结果。在按计划或按需处理、转换和分析大量数据的组织中，批量计算是一种常见的模式，应用示例包括金融服务和工程设计等各种领域。

批处理 ( Batch ) 很适合处理本质上并行（有时称为“易并行”）的应用程序或工作负荷，而这些本身就适合在多部计算机上以并行任务运行。请参阅图 1。

![并行任务][parallel]

**图 1.在多台计算机上运行的并行任务**

示例包括：

* 金融风险建模
* 图像呈现和图像处理
* 媒体编码和转码
* 基因序列分析
* 软件测试

你还可以使用批处理 ( Batch ) 来执行并行计算（最后加上归纳步骤），以及其他更复杂的并行工作负荷。

>[AZURE.NOTE]目前批处理 ( Batch ) 仅支持基于 Windows Server 的虚拟机上运行的工作负荷。此外，批处理 ( Batch ) 目前不支持消息传递接口 (MPI) 应用程序。

有关批处理 ( Batch ) 与 Azure 中其他 HPC 解决方案选项的比较，请参阅 [Batch 和 HPC 解决方案](/documentation/articles/batch-hpc-solutions)。

## 使用批处理 ( Batch ) 进行开发

使用批处理 ( Batch ) API 进行开发，以创建和管理计算节点池，以及计划要对池运行的作业和任务。编写客户端应用程序和前端来按需、按计划或者在执行工具管理的大型工作流期间中运行作业和任务。

有关批处理 ( Batch ) 概念的详细信息，请参阅 [Azure 批处理 ( Batch ) 功能概述](/documentation/articles/batch-api-basics)。

### 所需的帐户

+ **Azure 帐户和订阅** - 如果你没有帐户，可以注册获取[免费试用版](/pricing/free-trial/)。

+ **批处理 ( Batch ) 帐户** - 在进行批处理 ( Batch ) API 调用时，应使用批处理 ( Batch ) 帐户的名称和 URL 以及访问密钥作为凭据。所有批处理 ( Batch ) 资源（如计算节点、池、作业和任务）都与批处理 ( Batch ) 帐户关联。

+ **存储帐户** – 大多数批处理 ( Batch ) 方案需要使用一个 Azure 存储帐户来存储数据输入和输出，以及在计算节点上运行的脚本或可执行文件。若要创建存储帐户，请参阅[关于 Azure 存储帐户](/documentation/articles/storage-create-storage-account)。

### Batch 开发库和工具

配合 Visual Studio 使用这些 .NET 库与工具可以开发和管理 Azure 批处理 ( Batch ) 应用程序。

+ [批处理 ( Batch ) 客户端 .NET 库](http://www.nuget.org/packages/Azure.Batch/) (NuGet) – 开发客户端应用程序以使用批处理 ( Batch ) 服务运行作业
+ [批处理 ( Batch ) Management .NET 库](http://www.nuget.org/packages/Microsoft.Azure.Management.Batch/) (NuGet) – 管理批处理 ( Batch ) 帐户
+ [批处理 ( Batch ) 资源管理器](https://github.com/Azure/azure-batch-samples/tree/master/CSharp/BatchExplorer) (GitHub) – 构建此 GUI 应用程序以浏览、访问和更新批处理 ( Batch ) 帐户中的资源，包括作业和任务、计算节点和池以及计算节点上的文件。请参阅[博客文章](http://blogs.technet.com/b/windowshpc/archive/2015/01/20/azure-batch-explorer-sample-walkthrough.aspx)。


## 方案：扩大并行工作负荷

使用批处理 ( Batch ) API 的一个常见方案是扩大本质并行的任务，例如，对包含多达数千个计算核心的计算节点池执行图像绘制。

图 2 显示了使用批处理 ( Batch ) .NET 客户端应用程序运行批处理 ( Batch ) 并行工作负荷的工作流。


![工作项工作流][work_item_workflow]

**图 2.在批处理 ( Batch ) 服务上向外扩展并行工作负荷**

1.	将作业所需的输入文件（例如源数据或图像）上载到 Azure 存储帐户。作业任务运行时，批处理 ( Batch ) 服务会将文件加载到计算节点。

2.	将要在计算节点上运行的程序文件或脚本上载到存储帐户。这些内容可能包括二进制文件及其依赖程序集。运行任务时，批处理 ( Batch ) 服务还会将这些文件载入计算节点。

3.	在批处理 ( Batch )帐户中以编程方式创建批处理 ( Batch ) 计算节点池并指定属性，例如其大小及其运行的操作系统。你还可以定义如何[增加或减少](/documentation/articles/batch-automatic-scaling)池中的节点数目，以响应工作负荷。运行某个任务时，系统会向它分配此池中的某个节点。

4.	以编程方式定义批处理 ( Batch )作业，以便在节点池上运行工作负荷。

5.	将任务添加到作业。每个任务都使用所上载的程序处理已上载的文件中的信息。根据工作负荷，你可能想要在每个计算节点上同时运行[多个任务](/documentation/articles/batch-parallel-node-tasks)，以最大程度地利用资源。批处理 ( Batch )还支持专用的[作业准备和完成任务](/documentation/articles/batch-job-prep-release)，以便在运行计划的任务之前准备计算节点，并在完成这些任务之后清理数据。

6.	运行批处理 ( Batch )工作负荷并监视进度和结果。应用程序将通过 HTTPS 来与批处理 ( Batch ) 服务通信。在监视大量池、作业和任务时，若要改善应用程序的性能，可以利用[有效的方法查询](/documentation/articles/batch-efficient-list-queries) 批处理 ( Batch ) 服务。






## 后续步骤

* 使用 [批处理 ( Batch ) 客户端 .NET 库](/documentation/articles/batch-dotnet-get-started)开始开发你的第一个应用程序
* 浏览 [GitHub](https://github.com/Azure/azure-batch-samples) 上的批处理 ( Batch ) 代码示例

[parallel]: ./media/batch-technical-overview/parallel.png
[work_item_workflow]: ./media/batch-technical-overview/work_item_workflow.png

<!---HONumber=Mooncake_1221_2015-->
