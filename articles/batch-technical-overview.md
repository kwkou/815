<properties
	pageTitle="Azure Batch 服务基础知识 | Azure"
	description="了解如何使用 Azure Batch 服务执行大规模并发工作负荷与 HPC 工作负荷"
	services="batch"
	documentationCenter=""
	authors="mmacy"
	manager="timlt"
	editor=""/>

<tags
	ms.service="batch"
	ms.date="02/16/2016"
	wacn.date="06/06/2016"/>

# Azure 批处理 ( Batch ) 基础知识

Azure Batch 可帮助你在云中有效运行大规模并行和高性能计算 (HPC) 应用程序。它是一个平台服务，可以计划要在托管的虚拟机集合上运行的计算密集型工作，并且可以缩放计算资源以符合作业的需求。

使用 Batch 服务可让你以编程方式定义 Azure 计算资源来执行大规模批处理作业。你可以根据需要或按计划运行这些作业，而不需要手动配置和管理 HPC 群集、各个虚拟机、虚拟网络或作业计划程序。

## Batch 的用例

Batch 是一种托管的 Azure 服务，可用于实现批处理或批量计算 -- 运行大量类似任务以获取所需的结果。定期处理、转换和分析大量数据的组织最常使用批量计算。

Batch 很适合处理本质并行（也称为“超简单并行”）的应用程序和工作负荷。本质并行的工作负荷可容易拆分成多个任务，在多台计算机上同时执行。

![并行任务][1]

常见使用此技术处理的一些工作负荷示例如下：

* 金融风险建模
* 气候和水文数据分析
* 图像渲染、分析和处理
* 媒体编码和转码
* 基因序列分析
* 工程压力分析
* 软件测试

你还可以使用批处理 ( Batch ) 来执行并行计算（最后加上归纳步骤），以及其他更复杂的 HPC 工作负荷，例如[消息传递接口 (MPI)](/documentation/articles/batch-mpi/) 应用程序。

有关 Batch 与 Azure 中其他 HPC 解决方案选项的比较，请参阅 [Batch 和 HPC 解决方案](/documentation/articles/batch-hpc-solutions/)。

## 使用 Batch 进行开发

当你构建解决方案来使用 Azure Batch 处理并行工作负荷时，可以使用 Batch API 来编程。Batch API 可让你创建和管理计算节点（虚拟机）池，以及计划作业和任务在这些节点上运行。编写的客户端应用程序或服务使用 Batch API 来与 Batch 服务通信。可以为组织有效率地处理大量工作负荷，或提供前端服务给客户，让他们可以在一个、数百个或数千个节点上，按需要或按计划运行作业和任务。

> [AZURE.TIP] 当你准备钻研 Batch API 以深入了解它所提供的功能时，请参阅 [Azure Batch 功能概述](/documentation/articles/batch-api-basics/)。

### 需要的 Azure 帐户

开发 Batch 解决方案时，你将在 Microsoft Azure 中使用以下帐户。

- **Azure 帐户和订阅** - 如果你没有 Azure 订阅，可以激活你的 [MSDN 订户权益][msdn_benefits]，或注册[试用 Azure 帐户][free_account]。当你创建帐户时，系统将为你创建默认订阅。

- **Batch 帐户** - 当应用程序与 Batch 服务交互时，将使用帐户名、帐户的 URL 和访问密钥作为凭据。所有 Batch 资源（如池、计算节点、作业和任务）都与 Batch 帐户关联。

- **存储帐户** - Batch 内置支持处理 [Azure 存储空间][azure_storage]中的文件。几乎每个 Batch 方案都使用 Azure 存储空间来暂存文件（用于任务执行的程序及它们处理的数据），以及存储任务生成的输出数据。若要创建存储帐户，请参阅[关于 Azure 存储帐户](/documentation/articles/storage-create-storage-account/)。

### Batch 开发库和工具

若要使用 Azure Batch 构建解决方案，可以使用 Batch .NET 客户端库、PowerShell，甚至直接发出 REST API 调用。请使用任何或所有这些工具来开发在 Batch 中运行作业的客户端应用程序和服务。

- [Batch .NET][api_net] 客户端库 - 大多数 Batch 解决方案都是使用 Batch .NET 客户端库构建的，可以通过 [NuGet 获取][api_net_nuget]它。

- [Batch Management .NET][api_net_mgmt] 客户端库 - 也可以通过 [NuGet 获取][api_net_mgmt_nuget]，在客户端应用程序或服务中，使用 Batch Management .NET 客户端库以编程方式管理 Batch 帐户。

- [Batch REST][batch_rest] API - Batch REST API 提供与 Batch .NET 客户端库相同的所有功能。事实上，Batch .NET 库本身在幕后使用 Batch REST API 来与 Batch 服务交互。

- [Batch PowerShell cmdlet][batch_ps] - [Azure PowerShell](/documentation/articles/powershell-install-configure/) 模块中的 Batch PowerShell cmdlet 可让你使用 PowerShell 来管理 Batch 资源。

- [Azure Batch 资源管理器][batch_explorer] - Batch 资源管理器是可[通过 GitHub 获取][github_samples]的其中一个 Batch .NET 示例应用程序。使用 Visual Studio 2013 或 2015 构建此 Windows Presentation Foundation (WPF) 应用程序，在开发及调试 Batch 解决方案时，使用它来浏览和管理 Batch 帐户中的资源。在 Batch 资源管理器接口中只需按几下鼠标，就可以查看作业、池和任务详细信息、从计算节点下载文件，甚至使用获取的远程桌面 (RDP) 文件从远程连接到节点。

- [Microsoft Azure 存储空间资源管理器][storage_explorer] - 严格地说，虽然存储空间资源管理器不算是 Azure Batch 工具，但却是开发和调试 Batch 解决方案时的另一个很有用的工具。

## 方案：扩大并行工作负荷

使用 Batch API 来与 Batch 服务交互的一个常见方案涉及在计算节点池上放大本质并行任务，例如渲染 3D 场景的图像。例如，此计算节点池可能是“渲染场”，为渲染作业提供数十、数百甚至数千个核心。

下图显示一个常见的 Batch 工作流，其中有一个客户端应用程序或托管服务使用 Batch 运行并行工作负荷。

![Batch 解决方案工作流][2]

在此常见方案中，应用程序或服务执行以下步骤，在 Azure Batch 中处理计算工作负荷：

1. 将**输入文件**和处理这些文件的**应用程序**上载到 Azure 存储帐户。输入文件可以是应用程序要处理的任何数据，例如金融建模数据或要转码的视频文件。应用程序文件可以是任何用于处理数据的应用程序，例如 3D 渲染应用程序或媒体转码器。

2. 在 Batch 帐户中创建计算节点的 Batch **池** -- 这些是将执行任务的虚拟机。需要指定属性，例如[节点大小](/documentation/articles/cloud-services-sizes-specs/)、其操作系统，以及节点加入池时要安装的应用程序在 Azure 存储空间中的位置（在步骤 #1 中上载的应用程序）。你还可以配置池来随着任务所生成的工作负荷而[自动缩放](/documentation/articles/batch-automatic-scaling/) - 动态缩放池中的计算节点数目。

3. 创建 Batch **作业**以便在计算节点池上运行工作负荷。创建作业时，需要将它与 Batch 池关联。

4. 将**任务**添加到作业。当你将任务添加到作业时，Batch 服务将自动计划任务在池中的计算节点上执行。每项任务使用上载的应用程序来处理输入文件。

    - 4a.任务执行之前，它可以将它要处理的数据（输入文件）下载到它被分配到的计算节点。如果应用程序未安装在节点上（请参阅步骤 2#），可以从此处下载。下载完成后，任务将在它被分配到的节点上执行。

5. 任务执行时，你可以查询 Batch 来监视作业及其任务的进度。客户端应用程序或服务通过 HTTPS 来与 Batch 服务通信，因为可能监视数千个计算节点上执行的数千个任务，因此请务必[有效地查询 Batch 服务](/documentation/articles/batch-efficient-list-queries/)。

6. 当任务完成时，它们可以将其输出数据上载到 Azure 存储空间。你也可以直接从计算节点检索文件。

7. 当监视检测到作业中的任务已完成时，客户端应用程序或服务可以下载输出数据来进一步处理或评估。

请记住，这只是使用 Batch 的一种方式，此方案只描述它可用的几项功能。例如，可以在每个计算节点上[以并行方式执行多项任务](/documentation/articles/batch-parallel-node-tasks/)，也可以使用[作业准备和完成任务](/documentation/articles/batch-job-prep-release/)来准备作业的节点，然后再清除。

## 后续步骤

在学习示例 Batch 方案之后，接下来可以更深入探索该服务，以了解如何使用它处理计算密集型并行工作负荷。

- 参阅[适用于 .NET 的 Azure Batch 库入门](/documentation/articles/batch-dotnet-get-started/)，了解如何使用 C# 和 Batch .NET 库来运用上述技巧。这应该是学习如何使用 Batch 服务的第一站。

- 查看 [Batch 功能概述](/documentation/articles/batch-api-basics/)，深入了解 Batch 提供用于处理计算密集型工作负荷的 API 功能。

- 除了 Batch 资源管理器，其他 [GitHub 上的代码示例][github_samples]也演示了如何使用 Batch .NET 库中的许多 Batch 功能。



[azure_storage]: /services/storage/
[api_net]: https://msdn.microsoft.com/library/azure/mt348682.aspx
[api_net_nuget]: https://www.nuget.org/packages/Azure.Batch/
[api_net_mgmt]: https://msdn.microsoft.com/library/azure/mt463120.aspx
[api_net_mgmt_nuget]: https://www.nuget.org/packages/Microsoft.Azure.Management.Batch/
[batch_explorer]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/BatchExplorer
[batch_ps]: https://msdn.microsoft.com/library/azure/mt125957.aspx
[batch_rest]: https://msdn.microsoft.com/library/azure/Dn820158.aspx

[free_account]: /pricing/1rmb-trial/
[github_samples]: https://github.com/Azure/azure-batch-samples

[storage_explorer]: http://storageexplorer.com/

[1]: ./media/batch-technical-overview/tech_overview_01.png
[2]: ./media/batch-technical-overview/tech_overview_02.png

<!---HONumber=Mooncake_0503_2016-->