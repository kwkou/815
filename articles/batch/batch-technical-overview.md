<properties
    pageTitle="Azure Batch 在云中运行大规模并行计算解决方案 | Azure"
    description="了解如何使用 Azure Batch 服务执行大规模并发工作负荷与 HPC 工作负荷"
    services="batch"
    documentationcenter=""
    author="tamram"
    manager="timlt"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid="93e37d44-7585-495e-8491-312ed584ab79"
    ms.service="batch"
    ms.workload="big-compute"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="get-started-article"
    ms.date="03/08/2017"
    wacn.date="04/24/2017"
    ms.author="tamram"
    ms.custom="H1Hack27Feb2017"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="5832f0159c24fcadc045735ed89a86464aa5172a"
    ms.lasthandoff="04/14/2017" />

# <a name="run-intrinsically-parallel-workloads-with-batch"></a>使用 Batch 运行固有并行的工作负荷

Azure 批处理是一项平台服务，适用于在云中有效运行大规模并行和高性能计算 (HPC) 应用程序。 Azure Batch 可计划要在托管的虚拟机集合上运行的计算密集型工作，并且可以缩放计算资源以符合作业的需求。

使用 Azure Batch 时，可轻松定义用于大规模并行执行应用程序的 Azure 计算资源。 无需手动创建、配置和管理 HPC 群集、各个虚拟机、虚拟网络或复杂的作业和任务计划基础结构。 Azure Batch 将自动执行或简化这些任务。

## <a name="use-cases-for-batch"></a>Batch 的用例
批处理是一种托管的 Azure 服务，可用于实现*批处理*或*批量计算* -- 运行大量类似任务以获取所需的结果。 定期处理、转换和分析大量数据的组织最常使用批量计算。

Batch 很适合处理本质并行（也称为“超简单并行”）的应用程序和工作负荷。 本质并行的工作负荷是指容易拆分成多个任务，在多台计算机上同时执行的工作负荷。

![并行任务][1]<br/>

通常使用此技术处理的工作负荷有以下示例：

- 金融风险建模
- 气候和水文数据分析
- 图像渲染、分析和处理
- 媒体编码和转码
- 基因序列分析
- 工程压力分析
- 软件测试

用户还可以使用 Batch 执行并行计算（最后加上归纳步骤），以及其他更复杂的 HPC 工作负荷，例如 [消息传递接口 (MPI)](/documentation/articles/batch-mpi/) 应用程序。

有关 Batch 与 Azure 中其他 HPC 解决方案选项的比较，请参阅 [Batch 和 HPC 解决方案](/documentation/articles/batch-hpc-solutions/)。

[AZURE.INCLUDE [batch-pricing-include](../../includes/batch-pricing-include.md)]

## <a name="scenario-scale-out-a-parallel-workload"></a>方案：扩大并行工作负荷
使用 Batch API 来与 Batch 服务交互的一个常见方案涉及在计算节点池上放大本质并行任务，例如渲染 3D 场景的图像。 例如，此计算节点池可能是“渲染场”，为渲染作业提供数十、数百甚至数千个核心。

下图显示一个常见的 Batch 工作流，其中有一个客户端应用程序或托管服务使用 Batch 运行并行工作负荷。

![Batch 解决方案工作流][2]

在此常见方案中，应用程序或服务执行以下步骤，在 Azure Batch 中处理计算工作负荷：

1. 将**输入文件**和处理这些文件的**应用程序**上载到 Azure 存储帐户。 输入文件可以是应用程序要处理的任何数据，例如金融建模数据或要转码的视频文件。 应用程序文件可以是任何用于处理数据的应用程序，例如 3D 渲染应用程序或媒体转码器。
2. 在 Batch 帐户中创建计算节点的 Batch **池** - 这些节点是将执行任务的虚拟机。 需要指定属性，例如[节点大小](/documentation/articles/cloud-services-sizes-specs/)、其操作系统，以及节点加入池时要安装的应用程序在 Azure 存储中的位置（在步骤 1 中上载的应用程序）。 用户还可以配置池，使之随任务生成的工作负荷[自动缩放](/documentation/articles/batch-automatic-scaling/)。 自动缩放功能可动态调整池中计算节点的数目。
3. 创建 Batch **作业** ，在计算节点池上运行工作负荷。 创建作业时，需要将它与 Batch 池关联。
4. 将 **任务** 添加到作业。 当你将任务添加到作业时，Batch 服务将自动计划任务在池中的计算节点上执行。 每项任务使用上载的应用程序来处理输入文件。
   
   - 4a. 任务执行之前，它可以将它要处理的数据（输入文件）下载到它被分配到的计算节点。 如果应用程序未安装在节点上（请参阅步骤 2#），可以从此处下载。 下载完成后，任务将在它被分配到的节点上执行。
5. 任务执行时，你可以查询 Batch 来监视作业及其任务的进度。 客户端应用程序或服务通过 HTTPS 与 Batch 服务通信。 由于监视的任务可能成千上万，而这些任务又运行在成千上万的计算节点上，因此请确保[高效查询批处理服务](/documentation/articles/batch-efficient-list-queries/)。
6. 当任务完成时，它们可以将其输出数据上载到 Azure 存储空间。 也可直接从计算节点上的文件系统检索文件。
7. 当监视检测到作业中的任务已完成时，客户端应用程序或服务可以下载输出数据来进一步处理或评估。

请记住，这只是使用 Batch 的一种方式，此方案只描述它的几项可用功能。 例如，可以在每个计算节点上[以并行方式执行多项任务](/documentation/articles/batch-parallel-node-tasks/)，也可以使用[作业准备和完成任务](/documentation/articles/batch-job-prep-release/)来准备作业的节点，然后再清除。

## <a name="next-steps"></a>后续步骤
在大致了解 Batch 服务后，接下来可以更深入探索该服务，以了解如何使用它来处理计算密集型并行工作负荷。

- 对于准备使用 Batch 的任何人，有必要阅读 [面向开发人员的 Batch 功能概述](/documentation/articles/batch-api-basics/)了解基本信息。 本文中包含有关 Batch 服务资源（如池、节点、作业和任务）以及生成你的 Batch 应用程序时可以使用的许多 API 功能的更多详细信息。
- 了解适用于生成批处理解决方案的[批处理 API 和工具](/documentation/articles/batch-apis-tools/)。
- [Get started with the Azure Batch library for .NET](/documentation/articles/batch-dotnet-get-started/) （适用于 .NET 的 Azure Batch 库入门），了解如何使用 C# 和 Batch .NET 库在常见的 Batch 工作流中执行简单的工作负荷。 若要了解如何使用 Batch 服务，应先学习此文。 此外还提供了该教程的 [Python 版本](/documentation/articles/batch-python-tutorial/) 。
- 下载 [GitHub 上的代码示例][github_samples]，了解如何通过综合使用 C# 和 Python 与批处理来计划和处理示例工作负荷。

[github_samples]: https://github.com/Azure/azure-batch-samples

[1]: ./media/batch-technical-overview/tech_overview_01.png
[2]: ./media/batch-technical-overview/tech_overview_02.png

<!---Update_Description: wording update -->