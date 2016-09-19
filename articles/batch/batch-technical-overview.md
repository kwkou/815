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
	ms.date="08/15/2016"
	wacn.date="09/19/2016"/>

# Azure Batch 基础知识

Azure Batch 可帮助用户在云中有效运行大规模并行的高性能计算 (HPC) 应用程序。它是一个平台服务，可以计划要在托管的虚拟机集合上运行的计算密集型工作，并且可以缩放计算资源以符合作业的需求。

在使用 Batch 服务时，可以定义用于大规模并行执行应用程序的 Azure 计算资源。你可以根据需要或按计划运行作业，而不需要手动创建、配置和管理 HPC 群集、各个虚拟机、虚拟网络或复杂的作业和任务计划基础结构。

## Batch 的用例

Batch 是一种托管的 Azure 服务，可用于实现批处理或批量计算 -- 运行大量类似任务以获取所需的结果。定期处理、转换和分析大量数据的组织最常使用批量计算。

Batch 很适合处理本质并行（也称为“超简单并行”）的应用程序和工作负荷。本质并行的工作负荷可容易拆分成多个任务，在多台计算机上同时执行。

![并行任务][1]<br/>

常见使用此技术处理的一些工作负荷示例如下：

* 金融风险建模
* 气候和水文数据分析
* 图像渲染、分析和处理
* 媒体编码和转码
* 基因序列分析
* 工程压力分析
* 软件测试

用户还可以使用 Batch 执行并行计算（最后加上归纳步骤），以及其他更复杂的 HPC 工作负荷，例如[消息传递接口 (MPI)](/documentation/articles/batch-mpi/) 应用程序。

有关 Batch 与 Azure 中其他 HPC 解决方案选项的比较，请参阅 [Batch 和 HPC 解决方案](/documentation/articles/batch-hpc-solutions/)。

## 使用 Batch 进行开发

使用 Batch 处理并行工作负荷通常是使用 [Batch API](#batch-development-apis) 之一以编程方式实现的。Batch API 可让你创建和管理计算节点（虚拟机）池，以及计划作业和任务在这些节点上运行。编写的客户端应用程序或服务使用 Batch API 来与 Batch 服务通信。

可以为组织高效处理大量工作负荷，或提供服务前端给客户，让他们可以在一个、数百个甚至数千个节点上，按需要或按计划运行作业和任务。

> [AZURE.TIP] 若要深入了解 Batch API 所提供的功能，请参阅 [Batch feature overview for developers（面向开发人员的 Batch 功能概述）](/documentation/articles/batch-api-basics/)。

### 需要的 Azure 帐户

开发 Batch 解决方案时，用户在 Azure 中使用以下帐户。

- **Azure 帐户和订阅** - 如果还没有 Azure 订阅，可以注册 [Azure 试用帐户][free_account]。创建帐户时，系统为用户创建默认订阅。

- **Batch 帐户** - 应用程序与 Batch 服务交互时，使用帐户名、帐户的 URL 和访问密钥作为凭据。所有 Batch 资源（如池、计算节点、作业和任务）都与 Batch 帐户关联。可以在 Azure 门户预览版中[创建和管理 Batch 帐户](/documentation/articles/batch-account-create-portal/)。

- **存储帐户** - Batch 提供的内置支持允许处理 [Azure 存储][azure_storage]中的文件。几乎每个 Batch 方案都使用 Azure 存储来暂存任务所运行的程序及其处理的数据，以及存储任务生成的输出数据。若要创建存储帐户，请参阅[关于 Azure 存储帐户](/documentation/articles/storage-create-storage-account/)。

### <a name="batch-development-apis"></a>Batch 开发 API

应用程序和服务可以发出直接的 REST API 调用，使用以下一个或多个客户端库或将其结合使用，以使用 Batch 服务管理计算资源和大规模运行并行工作负荷。

| API | API 参考 | 下载 | 代码示例 |
| ----------------- | ------------- | -------- | ------------ |
| **Batch REST** | [MSDN][batch_rest] | 不适用 | [MSDN][batch_rest] |
| **Batch .NET** | [MSDN][api_net] | [NuGet ][api_net_nuget] | [GitHub][api_sample_net] |
| **Batch Python** | [readthedocs.io][api_python] | [PyPI][api_python_pypi] |[GitHub][api_sample_python] |
| **Batch Node.js** | [github.io][api_nodejs] | [npm][api_nodejs_npm] | - |
| **Batch Java**（预览版）| [github.io][api_java] | [Maven][api_java_jar] | [GitHub][api_sample_java] |

### Batch 资源管理

除了客户端 API 以外，还可以使用以下功能来管理 Batch 帐户中的资源。

- [Batch PowerShell cmdlet][batch_ps]：[Azure PowerShell](/documentation/articles/powershell-install-configure/) 模块中的 Azure Batch cmdlet 可让用户使用 PowerShell 管理 Batch 资源。

- [Azure CLI](/documentation/articles/xplat-cli-install/)：Azure 命令行接口 (Azure CLI) 是一个跨平台工具集，提供用来与许多 Azure 服务（包括 Batch）交互的 shell 命令。

- [Batch Management .NET](/documentation/articles/batch-management-dotnet/) 客户端库：也可以通过 [NuGet][api_net_mgmt_nuget] 获取。用户可以使用 Batch Management .NET 客户端库以编程方式管理 Batch 帐户、配额和应用程序包。[MSDN][api_net_mgmt] 上提供了管理库的参考信息。

### Batch 工具

以下是在生成和调试 Batch 应用程序和服务时可以使用的一些有用工具，虽然在使用 Batch 生成解决方案时并不需要这些工具。

 - [Azure 门户预览版][portal]：用户可以在 Azure 门户预览版的 Batch 边栏选项卡中创建、监视和删除 Batch 池、作业和任务。用户运行作业时，可以查看这些资源和其他资源的状态信息，甚至从池中的计算节点下载文件（例如，在进行故障排除时下载失败任务的 `stderr.txt`）。用户还可以下载可用于登录到计算节点的远程桌面 (RDP) 文件。

 - [Azure Batch Explorer][batch_explorer]：Batch Explorer 提供与 Azure 门户预览版类似的 Batch 资源管理功能，但位于独立的 Windows Presentation Foundation (WPF) 客户端应用程序中。[GitHub][github_samples] 上提供的一个 Batch .NET 示例应用程序，用户可以使用 Visual Studio 2015 或更高版本生成它，然后在开发和调试 Batch 解决方案时，使用它浏览和管理 Batch 帐户中的资源。可以查看作业、池和任务详细信息，从计算节点下载文件，以及使用可通过 Batch Explorer 下载的远程桌面 (RDP) 文件以远程方式连接到节点。

 - [Azure 存储资源管理器][storage_explorer]：虽然严格地说，存储资源管理器不算是 Azure Batch 工具，但却是开发和调试 Batch 解决方案时的另一个很有用的工具。

## 方案：扩大并行工作负荷

使用 Batch API 来与 Batch 服务交互的一个常见方案涉及在计算节点池上放大本质并行任务，例如渲染 3D 场景的图像。例如，此计算节点池可能是“渲染场”，为渲染作业提供数十、数百甚至数千个核心。

下图显示一个常见的 Batch 工作流，其中有一个客户端应用程序或托管服务使用 Batch 运行并行工作负荷。

![Batch 解决方案工作流][2]

在此常见方案中，应用程序或服务执行以下步骤，在 Azure Batch 中处理计算工作负荷：

1. 将**输入文件**和处理这些文件的**应用程序**上载到 Azure 存储帐户。输入文件可以是应用程序要处理的任何数据，例如金融建模数据或要转码的视频文件。应用程序文件可以是任何用于处理数据的应用程序，例如 3D 渲染应用程序或媒体转码器。

2. 在 Batch 帐户中创建计算节点的 Batch **池** - 这些节点是将执行任务的虚拟机。需要指定属性，例如[节点大小](/documentation/articles/cloud-services-sizes-specs/)、其操作系统，以及节点加入池时要安装的应用程序在 Azure 存储中的位置（在步骤 1 中上载的应用程序）。用户还可以配置池来随着任务所生成的工作负荷而[自动缩放](/documentation/articles/batch-automatic-scaling/) - 动态调整池中的计算节点数。

3. 创建 Batch **作业**，在计算节点池上运行工作负荷。创建作业时，需要将它与 Batch 池关联。

4. 将**任务**添加到作业。当你将任务添加到作业时，Batch 服务将自动计划任务在池中的计算节点上执行。每项任务使用上载的应用程序来处理输入文件。

    - 4a.任务执行之前，它可以将它要处理的数据（输入文件）下载到它被分配到的计算节点。如果应用程序未安装在节点上（请参阅步骤 2#），可以从此处下载。下载完成后，任务将在它被分配到的节点上执行。

5. 任务执行时，你可以查询 Batch 来监视作业及其任务的进度。客户端应用程序或服务通过 HTTPS 来与 Batch 服务通信，因为可能要监视数千个计算节点上运行的数千个任务，因此请务必[有效地查询 Batch 服务](/documentation/articles/batch-efficient-list-queries/)。

6. 当任务完成时，它们可以将其输出数据上载到 Azure 存储空间。你也可以直接从计算节点检索文件。

7. 当监视检测到作业中的任务已完成时，客户端应用程序或服务可以下载输出数据来进一步处理或评估。

请记住，这只是使用 Batch 的一种方式，此方案只描述它的几项可用功能。例如，可以在每个计算节点上[以并行方式执行多项任务](/documentation/articles/batch-parallel-node-tasks/)，也可以使用[作业准备和完成任务](/documentation/articles/batch-job-prep-release/)来准备作业的节点，然后再清除。

## 后续步骤

在大致了解 Batch 服务后，接下来可以更深入探索该服务，以了解如何使用它来处理计算密集型并行工作负荷。

- 阅读 [Batch feature overview for developers（面向开发人员的 Batch 功能概述）](/documentation/articles/batch-api-basics/)，深入了解 Batch 提供的用于处理工作负荷的 API 功能。想要使用 Batch 的任何人都有必要阅读此文。

- 阅读 [Get started with the Azure Batch library for .NET（适用于 .NET 的 Azure Batch 库入门）](/documentation/articles/batch-dotnet-get-started/)，了解如何使用 C# 和 Batch .NET 库在常见的 Batch 工作流中执行简单的工作负荷。若要了解如何使用 Batch 服务，应先学习此文。此外还提供了该教程的 [Python 版本](/documentation/articles/batch-python-tutorial/)。

- 下载 [GitHub 上的代码示例][github_samples]，了解如何通过综合使用 C# 和 Python 与 Batch 来计划和处理示例工作负荷。


[azure_storage]: https://azure.microsoft.com/services/storage/
[api_java]: http://azure.github.io/azure-sdk-for-java/
[api_java_jar]: http://search.maven.org/#search%7Cga%7C1%7Ca%3A%22azure-batch%22
[api_net]: https://msdn.microsoft.com/zh-cn/library/azure/mt348682.aspx
[api_net_nuget]: https://www.nuget.org/packages/Azure.Batch/
[api_net_mgmt]: https://msdn.microsoft.com/zh-cn/library/azure/mt463120.aspx
[api_net_mgmt_nuget]: https://www.nuget.org/packages/Microsoft.Azure.Management.Batch/
[api_nodejs]: http://azure.github.io/azure-sdk-for-node/azure-batch/latest/
[api_nodejs_npm]: https://www.npmjs.com/package/azure-batch
[api_python]: http://azure-sdk-for-python.readthedocs.io/en/latest/ref/azure.batch.html
[api_python_pypi]: https://pypi.python.org/pypi/azure-batch
[api_sample_net]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp
[api_sample_python]: https://github.com/Azure/azure-batch-samples/tree/master/Python/Batch
[api_sample_java]: https://github.com/Azure/azure-batch-samples/tree/master/Java/
[batch_ps]: https://msdn.microsoft.com/zh-cn/library/azure/mt125957.aspx
[batch_rest]: https://msdn.microsoft.com/zh-cn/library/azure/Dn820158.aspx
[free_account]: /pricing/1rmb-trial/
[github_samples]: https://github.com/Azure/azure-batch-samples
[learning_path]: https://azure.microsoft.com/documentation/learning-paths/batch/
[msdn_benefits]: https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/
[batch_explorer]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/BatchExplorer
[storage_explorer]: http://storageexplorer.com/
[portal]: https://portal.azure.cn

[1]: ./media/batch-technical-overview/tech_overview_01.png
[2]: ./media/batch-technical-overview/tech_overview_02.png

<!---HONumber=Mooncake_0912_2016-->