<properties
    pageTitle="配置依赖于其他任务的任务 - Azure Batch | Azure"
    description="在 Azure Batch 中创建依赖于其他任务的成功完成的任务，以处理 MapReduce 样式工作负荷和类似的大数据工作负荷。"
    services="batch"
    documentationcenter=".net"
    author="tamram"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="b8d12db5-ca30-4c7d-993a-a05af9257210"
    ms.service="batch"
    ms.devlang="multiple"
    ms.topic="article"
    ms.tgt_pltfrm="vm-windows"
    ms.workload="big-compute"
    ms.date="01/23/2017"
    wacn.date="03/16/2017"
    ms.author="tamram" />

# Azure Batch 中的任务依赖关系
Azure Batch 的任务依赖关系功能适用于处理以下项：

- 云中的 MapReduce 样式工作负荷。
- 数据处理任务可以表示为有向无环图 (DAG) 的作业。
- 下游任务依赖于上游任务输出的任何其他作业。

借助 Batch 任务依赖关系，创建的任务仅在一个或多个其他任务成功完成后才会在计算节点上按计划执行。例如，可以创建一个作业，使用单独的并行任务呈现 3D 影片的每个帧。最后一个任务为“合并任务”，仅在成功呈现所有帧后，才会将呈现的帧合并为完整的影片。

用户可以创建依赖于一对一或一对多关系中其他任务的任务。用户甚至可以创建一个范围依赖关系，使其中一项任务依赖于特定任务 ID 范围内一组任务的成功完成。你可以组合这三种基本方案，以创建多对多关系。

## Batch .NET 的任务依赖关系
本文讨论如何使用 [Batch .NET][net_msdn] 库配置任务依赖关系。我们首先说明如何为作业[启用任务依赖关系](#enable-task-dependencies)，然后演示如何[为任务配置依赖关系](#create-dependent-tasks)。最后介绍 Batch 支持的[依赖关系方案](#dependency-scenarios)。

## <a name="enable-task-dependencies"></a>启用任务依赖关系

若要在 Batch 应用程序中使用任务依赖关系，必须先告知 Batch 服务：作业要使用任务依赖关系。在 Batch .NET 中，为 [CloudJob][net_cloudjob] 启用任务依赖关系的方法是将其 [UsesTaskDependencies][net_usestaskdependencies] 属性设置为 `true`：

csharp

	CloudJob unboundJob = batchClient.JobOperations.CreateJob( "job001",
	    new PoolInformation { PoolId = "pool001" });

	// IMPORTANT: This is REQUIRED for using task dependencies.
	unboundJob.UsesTaskDependencies = true;


在上面的代码片段中，“batchClient”是 [BatchClient][net_batchclient] 类的一个实例。

## <a name="create-dependent-tasks"></a>创建依赖任务

若要创建一个依赖于一个或多个其他任务的成功完成的任务，需告知 Batch：该任务“依赖于”其他任务。在 Batch .NET 中，为 [CloudTask][net_cloudtask].[DependsOn][net_dependson] 属性配置 [TaskDependencies][net_taskdependencies] 类的一个实例：

csharp

	// Task 'Flowers' depends on completion of both 'Rain' and 'Sun'
	// before it is run.
	new CloudTask("Flowers", "cmd.exe /c echo Flowers")
	{
	    DependsOn = TaskDependencies.OnIds("Rain", "Sun")
	},

此代码片段创建了一个 ID 为“Flowers”的任务，该任务计划为仅在 ID 为“Rain”和“Sun”的任务成功完成后，才在计算节点上运行。

> [AZURE.NOTE]
当任务处于“已完成”状态并且其**退出代码**为 `0` 时，可认为该任务已完成。在 Batch .NET 中，这意味着 [CloudTask][net_cloudtask].[State][net_taskstate] 属性值为 `Completed`，CloudTask 的 [TaskExecutionInformation][net_taskexecutioninformation].[ExitCode][net_exitcode] 属性值为 `0`。
> 
> 

## 依赖关系方案 <a name="dependency-scenarios"></a>
可以在 Azure Batch 中使用三种基本任务依赖关系方案：一对一、一对多和任务 ID 范围依赖关系。可以组合这些方案以提供第四种方案：多对多。

| 方案&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; | 示例 | |
|:---:| --- | --- |
| [一对一](#one-to-one) |*taskB* 依赖于 *taskA* <p/> 直到 *taskA* 成功完成，*taskB* 才会按计划执行 |![关系图：一对一任务依赖关系][1] |
| [一对多](#one-to-many) |*taskC* 同时依赖于 *taskA* 和 *taskB* <p/> 直到 *taskA* 和 *taskB* 成功完成，*taskC* 才会按计划执行 |![关系图：一对多任务依赖关系][2] |
| [任务 ID 范围](#task-id-range) |*taskD* 依赖于某个范围的任务 <p/> 直到 ID 为 *1* 到 *10* 的任务成功完成，*taskD* 才会按计划执行 |![关系图：任务 ID 范围依赖关系][3] |

> [AZURE.TIP]
可以创建**多对多**关系，例如，在此关系中任务 C、D、E 和 F 都依赖于任务 A 和 B。这很有用，例如，在下游任务依赖于多个上游任务的输出的并行化预处理方案中，即可以这样操作。
> 
> 

### <a name="one-to-one"></a>一对一
若要创建依赖于一个其他任务的成功完成的任务，可在填充 [CloudTask][net_cloudtask] 的 [DependsOn][net_dependson] 属性时，向 [TaskDependencies][net_taskdependencies].[OnId][net_onid] 静态方法提供单个任务 ID。

csharp

	// Task 'taskA' doesn't depend on any other tasks
	new CloudTask("taskA", "cmd.exe /c echo taskA"),

	// Task 'taskB' depends on completion of task 'taskA'
	new CloudTask("taskB", "cmd.exe /c echo taskB")
	{
	    DependsOn = TaskDependencies.OnId("taskA")
	},

### <a name="one-to-many"></a>一对多

若要创建依赖于多个任务的成功完成的任务，可在填充 [CloudTask][net_cloudtask] 的 [DependsOn][net_dependson] 属性时，向 [TaskDependencies][net_taskdependencies].[OnIds][net_onids] 静态方法提供任务 ID 的集合。

csharp

	// 'Rain' and 'Sun' don't depend on any other tasks
	new CloudTask("Rain", "cmd.exe /c echo Rain"),
	new CloudTask("Sun", "cmd.exe /c echo Sun"),

	// Task 'Flowers' depends on completion of both 'Rain' and 'Sun'
	// before it is run.
	new CloudTask("Flowers", "cmd.exe /c echo Flowers")
	{
	    DependsOn = TaskDependencies.OnIds("Rain", "Sun")
	},

### <a name="task-id-range"></a>任务 ID 范围

若要创建依赖于一组任务（其 ID 在某个范围内）的成功完成的任务，可在填充 [CloudTask][net_cloudtask] 的 [DependsOn][net_dependson] 属性时，向 [TaskDependencies][net_taskdependencies].[OnIdRange][net_onidrange] 静态方法提供该范围内的第一个和最后一个任务 ID。

> [AZURE.IMPORTANT]
将任务 ID 范围用于依赖关系时，该范围内的任务 ID *必须*采用整数值的字符串表示形式。此外，范围内的每项任务必须成功完成，依赖任务才能按计划执行。
> 
> 

csharp

	// Tasks 1, 2, and 3 don't depend on any other tasks. Because
	// we will be using them for a task range dependency, we must
	// specify string representations of integers as their ids.
	new CloudTask("1", "cmd.exe /c echo 1"),
	new CloudTask("2", "cmd.exe /c echo 2"),
	new CloudTask("3", "cmd.exe /c echo 3"),

	// Task 4 depends on a range of tasks, 1 through 3
	new CloudTask("4", "cmd.exe /c echo 4")
	{
	    // To use a range of tasks, their ids must be integer values.
	    // Note that we pass integers as parameters to TaskIdRange,
	    // but their ids (above) are string representations of the ids.
	    DependsOn = TaskDependencies.OnIdRange(1, 3)
	},

## 代码示例
[TaskDependencies][github_taskdependencies] 示例项目是 GitHub 上的 [Azure Batch 代码示例][github_samples]之一。此 Visual Studio 2015 解决方案演示如何在作业上启用任务依赖关系、如何创建依赖于其他任务的任务，以及如何在计算节点池中执行这些任务。

## 后续步骤
### 应用程序部署
使用 Batch 的[应用程序包](/documentation/articles/batch-application-packages/)功能，可以轻松地部署任务在计算节点上执行的应用程序并对其进行版本控制。

### 安装应用程序和暂存数据
有关准备节点以运行任务的各种方法的概述，请查看 Azure Batch 论坛中的帖子 [Installing applications and staging data on Batch compute nodes][forum_post]（在 Batch 计算节点上安装应用程序和暂存数据）。此帖子由 Azure Batch 团队的一名成员撰写，是一个很好的入门教程，它介绍了如何在计算节点上以不同方式获取文件（包括应用程序和任务输入数据）。

[forum_post]: https://social.msdn.microsoft.com/Forums/zh-cn/87b19671-1bdf-427a-972c-2af7e5ba82d9/installing-applications-and-staging-data-on-batch-compute-nodes?forum=azurebatch
[github_taskdependencies]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/ArticleProjects/TaskDependencies
[github_samples]: https://github.com/Azure/azure-batch-samples
[net_batchclient]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.batchclient.aspx
[net_cloudjob]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.cloudjob.aspx
[net_cloudtask]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.cloudtask.aspx
[net_dependson]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.cloudtask.dependson.aspx
[net_exitcode]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.taskexecutioninformation.exitcode.aspx
[net_msdn]: https://msdn.microsoft.com/zh-cn/library/azure/mt348682.aspx
[net_onid]: https://msdn.microsoft.com/zh-cn/library/microsoft.azure.batch.taskdependencies.onid.aspx
[net_onids]: https://msdn.microsoft.com/zh-cn/library/microsoft.azure.batch.taskdependencies.onids.aspx
[net_onidrange]: https://msdn.microsoft.com/zh-cn/library/microsoft.azure.batch.taskdependencies.onidrange.aspx
[net_taskexecutioninformation]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.taskexecutioninformation.aspx
[net_taskstate]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.common.taskstate.aspx
[net_usestaskdependencies]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.cloudjob.usestaskdependencies.aspx
[net_taskdependencies]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.taskdependencies.aspx

[1]: ./media/batch-task-dependency/01_one_to_one.png "关系图：一对一依赖关系"
[2]: ./media/batch-task-dependency/02_one_to_many.png "关系图：一对多依赖关系"
[3]: ./media/batch-task-dependency/03_task_id_range.png "关系图：任务 ID 范围依赖关系"

<!---HONumber=Mooncake_0306_2017-->
<!---Update_Description: update meta properties -->