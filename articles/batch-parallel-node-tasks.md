<properties
	pageTitle="通过并行任务最大限度地使用 Batch 节点 | Azure"
	description="通过减少所用的计算节点数并在 Azure Batch 池的每个节点上运行并发任务，来提高效率并降低成本"
	services="batch"
	documentationCenter=".net"
	authors="mmacy"
	manager="timlt"
	editor="" />

   <tags
   	ms.service="batch"
	ms.date="04/21/2016"
   	wacn.date="06/06/2016"/>

# 通过并发节点任务最大限度提高 Azure 批处理 ( Batch ) 计算资源的使用量

了解如何同时在 Azure 批处理 ( Batch ) 池的每个计算节点上运行多个任务。允许在池的计算节点上并行执行任务可以最大程度地利用池中的资源，减少所需的节点数目。对于某些工作负荷，这可以缩短作业时间并降低成本。

尽管在某些情况下，将一个节点的所有资源专用于单个任务会更有利，但在更多情况下，最好是让多个任务共享这些资源：

 - **尽量减少数据传输**：适用于任务可以共享数据的情况。在此方案中，将共享数据复制到较小数目的节点并在每个节点上并行执行任务可以大大减少数据传输费用，尤其是在复制到每个节点的数据必须跨地理区域传输的情况下。

 - **尽量增加内存使用**：适用于任务需要大量的内存，但这种需要仅在执行过程中短时出现且时间不固定的情况。你可以减少计算节点的数量但增加其大小，同时提供更多的内存，以便有效地应对此类高峰负载。这些节点将在每个节点上并行运行多个任务，而每个任务都会充分利用节点在不同时间的大量内存。

 - **减少节点数目限制**：适用于需要在池中进行节点间通信的情况。目前，经过配置可以进行节点间通信的池仅限 50 个计算节点。因此，如果此类池中的每个节点都可以并行执行任务，则可同时执行更大数目的任务。

 - **复制本地计算群集**：适用于首次将计算环境移至 Azure 等情况。可以通过增大节点任务的最大数目来更紧密地完成现有物理配置的镜像操作，前提是该配置目前允许在单个计算节点上执行多个任务。

## 示例方案

以下示例说明了并行执行任务的好处。假设你的任务应用程序具有特定的 CPU 和内存要求，根据该要求，节点大小为 Standard\_D1 很合适。但若要在所需时间内执行作业，则需使用 1,000 个这样的节点。

如果不使用 Standard\_D1 节点（有 1 个 CPU 核心），则可使用每个节点 16 核的 Standard\_D14 节点，同时允许并行执行任务。因此，在这种情况下，可以使用 1/16 数目的节点，即只需使用 63 个节点，不需使用 1,000 个节点。如果每个节点都需要大型应用程序文件或引用数据，这会大大缩短作业执行时间并提高效率。

## 允许并行执行任务

你可以在Batch 解决方案中进行池级别的计算节点配置，以便并行执行任务。在创建池时，可以通过 Batch .NET 库设置 [CloudPool.MaxTasksPerComputeNode][maxtasks_net] 属性。如果你使用的是 Batch REST API，则可在创建池时在请求正文中设置 [maxTasksPerNode][maxtasks_rest] 元素。

使用 Azure Batch 时，可以通过节点设置来设置多达四倍 (4x) 的节点核心数，从而最大限度地提高任务数。例如，如果将池的节点大小配置为“大型”（四核），则可将 `maxTasksPerNode` 设置为 16。有关每个节点大小的核心数的详细信息，请参阅[云服务的大小](/documentation/articles/cloud-services-sizes-specs/)。有关服务限制的详细信息，请参阅 [Azure Batch 服务的配额和限制](/documentation/articles/batch-quota-limit/)。

> [AZURE.TIP] 为池构造[自动缩放公式][enable_autoscaling]时，请务必考虑 `maxTasksPerNode` 值。例如，如果增加每个节点的任务数，则可能会极大地影响对 `$RunningTasks` 求值的公式。有关详细信息，请参阅[自动缩放 Azure Batch 池中的计算节点](/documentation/articles/batch-automatic-scaling/)。

## 任务分发

当池中的计算节点可以并行执行任务时，必须指定任务在池中各节点之间的分发情况。

你可以通过 [CloudPool.TaskSchedulingPolicy][task_schedule] 属性来指定任务，即让任务在池中所有节点之间平均分配（“散布式”）。或者，先给池中的每个节点分配尽量多的任务，然后再将任务分配给池中的其他节点（“装箱式”）。

此功能十分重要，如需示例，请参阅上面示例中节点数为 Standard\_D14 的池，该池配置后的 [CloudPool.MaxTasksPerComputeNode][maxtasks_net] 值为 16。如果对 [CloudPool.TaskSchedulingPolicy][task_schedule] 进行配置时，将 [ComputeNodeFillType][fill_type] 设置为 *Pack*，则会充分使用每个节点的所有 16 个核心，并可通过[自动缩放池](/documentation/articles/batch-automatic-scaling/)将不使用的节点（没有分配任何任务的节点）从池中删除。这可以最大程度地减少资源使用量并节省资金。

## Batch .NET 示例

此 [Batch .NET][api_net] API 代码段演示了一个请求，该请求要求创建一个包含四个大型节点的池，每个节点最多四个任务。它指定了一个任务计划策略，要求先用任务填充一个节点，然后再将任务分配给池中的其他节点。有关如何使用 Batch .NET API 添加池的详细信息，请参阅 [BatchClient.PoolOperations.CreatePool][poolcreate_net]。

csharp

		CloudPool pool =
		    batchClient.PoolOperations.CreatePool(
		        poolId: "mypool",
				targetDedicated: 4
				virtualMachineSize: "large",
				cloudServiceConfiguration: new CloudServiceConfiguration(osFamily: "4"));
		pool.MaxTasksPerComputeNode = 4;
		pool.TaskSchedulingPolicy = new TaskSchedulingPolicy(ComputeNodeFillType.Pack);
		pool.Commit();


## Batch REST 示例

此 [Batch REST][api_rest] API 代码段演示了一个请求，该请求要求创建一个包含两个大型节点的池，每个节点最多四个任务。有关如何使用 REST API 添加池的详细信息，请参阅[将池添加到帐户][rest_addpool]。

json

		{
		  "odata.metadata":"https://myaccount.myregion.batch.azure.com/$metadata#pools/@Element",
		  "id":"mypool",
		  "vmSize":"large",
		  "cloudServiceConfiguration": {
		    "osFamily":"4",
		    "targetOSVersion":"*",
		  }
		  "targetDedicated":2,
		  "maxTasksPerNode":4,
		  "enableInterNodeCommunication":true,
		}


> [AZURE.NOTE] 只能在创建池时设置 `maxTasksPerNode` 元素和 [MaxTasksPerComputeNode][maxtasks_net] 属性。创建完池以后，不能对上述元素和属性进行修改。

## 探究示例项目

请查看 GitHub 上的 [ParallelNodeTasks][parallel_tasks_sample] 项目。这是一个工作代码示例，说明了如何使用 [CloudPool.MaxTasksPerComputeNode][maxtasks_net]。

此 C# 控制台应用程序使用 [Batch .NET][api_net] 库来创建包含一个或多个计算节点的池，并在这些节点上执行其数量可以配置的任务，以便模拟可变负荷。应用程序的输出指定了哪些节点执行了每个任务。该应用程序还提供了作业参数和持续时间的摘要。下面显示了同一个应用程序运行两次后的输出摘要部分。


		Nodes: 1
		Node size: large
		Max tasks per node: 1
		Tasks: 32
		Duration: 00:30:01.4638023


第一次执行示例应用程序时，结果显示，在池中只有一个节点且使用默认的一个节点一个任务设置的情况下，作业持续时间超过 30 分钟。


		Nodes: 1
		Node size: large
		Max tasks per node: 4
		Tasks: 32
		Duration: 00:08:48.2423500


第二次运行示例应用程序时，显示作业持续时间显著缩短。这是因为该池已被配置为每个节点四个任务，因此可以并行执行任务，使得作业可以在大约四分之一的时间内完成。

> [AZURE.NOTE] 上述摘要中的作业持续时间不包括创建池的时间。上述每个作业都提交到此前已创建的池，这些池的计算节点在提交时处于空闲状态。

## 批处理 ( Batch ) 资源管理器热度地图

[Azure 批处理 ( Batch ) 资源管理器][batch_explorer]是 Azure Batch [示例应用程序][github_samples]之一，包含*热度地图* 功能，提供任务执行可视化。执行 [ParallelTasks][parallel_tasks_sample] 示例应用程序时，可以使用“热图”功能来轻松可视化每个节点上并行任务的执行。

![批处理 ( Batch ) 资源管理器热度地图][1]

*批处理 ( Batch ) 资源管理器热度地图，其中显示了包含四个节点的池，每个节点当前正在执行四个任务*

[api_net]: http://msdn.microsoft.com/library/azure/mt348682.aspx
[api_rest]: http://msdn.microsoft.com/library/azure/dn820158.aspx
[batch_explorer]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/BatchExplorer
[cloudpool]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudpool.aspx
[enable_autoscaling]: https://msdn.microsoft.com/library/azure/dn820173.aspx
[fill_type]: https://msdn.microsoft.com/zh-CN/library/microsoft.azure.batch.common.computenodefilltype.aspx
[github_samples]: https://github.com/Azure/azure-batch-samples
[maxtasks_net]: http://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudpool.maxtaskspercomputenode.aspx
[rest_addpool]: https://msdn.microsoft.com/library/azure/dn820174.aspx
[parallel_tasks_sample]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/ArticleProjects/ParallelTasks
[poolcreate_net]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.pooloperations.createpool.aspx
[task_schedule]: https://msdn.microsoft.com/library/microsoft.azure.batch.cloudpool.taskschedulingpolicy.aspx

[1]: ./media/batch-parallel-node-tasks/heat_map.png

<!---HONumber=Mooncake_0530_2016-->