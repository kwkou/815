<properties
	pageTitle="在 Azure Batch 中使用多实例任务运行 MPI 应用程序 | Azure"
	description="了解如何在 Azure Batch 中使用多实例任务类型执行消息传递接口 (MPI) 应用程序。"
	services="batch"
	documentationCenter=".net"
	authors="mmacy"
	manager="timlt"
	editor="" />

<tags
	ms.service="batch"
	ms.date="02/19/2016"
	wacn.date="05/18/2016" />

# 在 Azure Batch 中使用多实例任务来执行消息传递接口 (MPI) 应用程序

多实例任务可让你在多个计算节点上同时执行 Azure Batch 任务，以实现高性能计算方案，例如消息传递接口 (MPI) 应用程序。在本文中，你将了解如何使用 [Batch .NET][api_net] 库来执行多实例任务。

## 多实例任务概述

在 Batch 中，每个任务通常是在单个计算节点上执行 -- 你将多个任务提交给作业，Batch 服务将每个任务安排在节点上执行。不过，通过配置任务的**多实例设置**，可以指示 Batch 将该任务拆分成子任务，以便在多个节点上执行。

![多实例任务概述][1]

将具有多实例设置的任务提交给作业时，Batch 执行多实例任务特有的几个步骤：

1. Batch 服务自动将任务拆分成一个**主要任务**和多个**子任务**。Batch 然后将主要任务和子任务排定在池的计算节点上执行。
2. 这些任务（主要任务和子任务）下载在多实例设置中指定的任何**通用资源文件**。
3. 下载一般资源文件之后，主要任务和子任务执行在多实例设计中指定的**协调命令**。此协调命令通常用于启动后台服务（例如 [Microsoft MPI][msmpi_msdn] 的 `smpd.exe`），也可能确认节点已准备好处理节点间的消息。
4. 主要任务及所有子任务成功完成协调命令之后，只有**主要任务**执行多实例任务的**命令行**（“应用程序命令”）。例如，在基于 [MS-MPI][msmpi_msdn] 的方案中，你将在此处使用 `mpiexec.exe` 执行已启用 MPI 的应用程序。

> [AZURE.NOTE] 虽然“多实例任务”在功能上不同，但不是特殊的任务类型，例如 [StartTask][net_starttask] 或 [JobPreparationTask][net_jobprep]。多实例任务只是已设置多实例设置的标准 Batch 任务（Batch .NET 中的 [CloudTask][net_task]）。在本文中，我们将它称为**多实例任务**。

## 多实例任务的要求

多实例任务需要有已启用**节点间通信**和**已禁用并发任务执行**的池。如果尝试在已禁用节点间通信，或 maxTasksPerNode 值大于 1 的池中执行多实例任务，则永远不排定任务 -- 它无限期停留在“活动”状态。本代码段显示如何使用 Batch .NET 库创建这种池。


		CloudPool myCloudPool =
			myBatchClient.PoolOperations.CreatePool(
				poolId: "MultiInstanceSamplePool",
				targetDedicated: 3
				virtualMachineSize: "small",
				cloudServiceConfiguration: new CloudServiceConfiguration(osFamily: "4"));
		
		// Multi-instance tasks require inter-node communication, and those nodes
		// must run only one task at a time.
		myCloudPool.InterComputeNodeCommunicationEnabled = true;
		myCloudPool.MaxTasksPerComputeNode = 1;


此外，多实例任务只在 **2015 年 12 月 14 日之后创建的池**中的节点上执行。

> [AZURE.TIP] 当你在 Batch 池中使用大小时，MPI 应用程序可以使用 Azure 的高性能、低延迟的远程直接内存访问 (RDMA) 网络。[云服务的大小](/documentation/articles/cloud-services-sizes-specs/)提供了 Batch 池中可用的计算节点大小的完整列表。

### 使用 StartTask 安装 MPI 应用程序

若要使用多实例任务来执行 MPI 应用程序，首先需要将 MPI 软件安装到池中的计算节点。这是使用 [StartTask][net_starttask] 的好时机，每当节点加入池或重新启动时，它就会执行。此代码段创建 StartTask 将 MS-MPI 安装包指定为[资源文件][net_resourcefile]，并指定将于资源文件下载到节点之后执行的命令行。

		
		// Create a StartTask for the pool which we use for installing MS-MPI on
		// the nodes as they join the pool (or when they are restarted).
		StartTask startTask = new StartTask
		{
		    CommandLine = "cmd /c MSMpiSetup.exe -unattend -force",
		    ResourceFiles = new List<ResourceFile> { new ResourceFile("https://mystorageaccount.blob.core.windows.net/mycontainer/MSMpiSetup.exe", "MSMpiSetup.exe") },
		    RunElevated = true,
		    WaitForSuccess = true
		};
		myCloudPool.StartTask = startTask;
		
		// Commit the fully configured pool to the Batch service to actually create
		// the pool and its compute nodes.
		await myCloudPool.CommitAsync();


> [AZURE.NOTE] 当在 Batch 中使用多实例任务实现 MPI 方案时，不限于只能使用 MS-MPI。任何 MPI 标准实现只要与为池中的计算节点指定的操作系统兼容都可使用。

## 使用 Batch .NET 创建多实例任务

我们已讨论池的要求和 MPI 包安装，现在让我们创建多实例任务。在此代码段中，我们将创建一个标准 [CloudTask][net_task]，然后配置其 [MultiInstanceSettings][net_multiinstance_prop] 属性。如上所述，多实例任务不是独特的任务类型，而只是已配置多实例设置的标准 Batch 任务。

		
		// Create the multi-instance task. Its command line is the "application command"
		// and will be executed *only* by the primary, and only after the primary and
		// subtasks execute the CoordinationCommandLine.
		CloudTask myMultiInstanceTask = new CloudTask(id: "mymultiinstancetask",
		    commandline: "cmd /c mpiexec.exe -wdir %AZ_BATCH_TASK_SHARED_DIR% MyMPIApplication.exe");
		
		// Configure the task's MultiInstanceSettings. The CoordinationCommandLine will be executed by
		// the primary and all subtasks.
		myMultiInstanceTask.MultiInstanceSettings =
		    new MultiInstanceSettings(numberOfNodes) {
		    CoordinationCommandLine = @"cmd /c start cmd /c ""%MSMPI_BIN%\smpd.exe"" -d",
		    CommonResourceFiles = new List<ResourceFile> {
		    new ResourceFile("https://mystorageaccount.blob.core.windows.net/mycontainer/MyMPIApplication.exe",
		                     "MyMPIApplication.exe")
		    }
		};
		
		// Submit the task to the job. Batch will take care of splitting it into subtasks and
		// scheduling them for execution on the nodes.
		await myBatchClient.JobOperations.AddTaskAsync("mybatchjob", myMultiInstanceTask);


## 主要任务和子任务

当你创建任务的多实例设置时，需要指定用于执行任务的计算节点数目。当你将任务提交给作业时，Batch 服务将创建一个**主要任务**和足够的**子任务**，并且合计符合指定的节点数。

系统分配范围介于 0 到 numberOfInstances-1 的整数 ID 给这些任务。ID 为 0 的任务是主要任务，其他所有 ID 都是子任务。例如，如果为任务创建以下多实例设置，则主要任务的 ID 为 0，而子任务的 ID 为 1 到 9。

		
		int numberOfNodes = 10;
		myMultiInstanceTask.MultiInstanceSettings = new MultiInstanceSettings(numberOfNodes);


## 协调命令和应用程序命令

主要任务和子任务都执行**协调命令**。主要任务及所有子任务完成执行协调命令之后，只有主要任务执行多实例任务的命令行。我们将此命令行称为**应用程序命令**，以便与协调命令有所区别。

阻止调用协调命令 -- 在所有子任务的协调命令成功返回之前，Batch 不执行应用程序命令。因此，协调命令应该启动任何所需的后台服务，确认它们已准备好可供使用，然后退出。例如，在使用 MS-MPI 第 7 版的方案中，此协调命令在节点上启动 SMPD 服务，然后退出：


		cmd /c start cmd /c ""%MSMPI_BIN%\smpd.exe"" -d


请注意此协调命令中使用 `start`。这是必需的，因为 `smpd.exe` 应用程序不会在执行后立即返回。如果不使用 [start][cmd_start] 命令，此协调命令就不返回，因此将阻止执行应用程序命令。

只有主要任务执行**应用程序命令**（为多实例任务执行的命令行）。就 MS MPI 应用程序而言，这是使用 `mpiexec.exe` 来执行已启用 MPI 的应用程序。例如，以下是使用 MS-MPI 第 7 版的方案所执行的应用程序命令：


		cmd /c ""%MSMPI_BIN%\mpiexec.exe"" -c 1 -wdir %AZ_BATCH_TASK_SHARED_DIR% MyMPIApplication.exe


## 资源文件

多实例任务需要考虑两组资源文件：所有任务（主要任务和子任务）下载的**一般资源文件**，以及为多实例任务本身指定的**资源文件**（只有主要任务下载）。

可以在任务的多实例设置中指定一个或多个**通用资源文件**。主要任务及所有子任务从 [Azure 存储空间](/documentation/articles/storage-introduction/)将这些通用资源文件下载到每个节点的任务共享目录。可以使用 `AZ_BATCH_TASK_SHARED_DIR` 环境变量从应用程序命令和协调命令行访问任务共享目录。

只有主要任务将为多实例任务本身指定的资源文件，下载到任务的任务目录 `AZ_BATCH_TASK_WORKING_DIR` -- 子任务不下载为多实例任务指定的资源文件。

节点上执行的主要任务及所有子任务可访问 `AZ_BATCH_TASK_SHARED_DIR` 的内容。`tasks/mybatchjob/job-1/mymultiinstancetask/` 是任务共享目录的一个例子。主要任务和每项子任务也有一个只有该任务才能访问的任务目录，并且可使用环境变量 `AZ_BATCH_TASK_WORKING_DIR` 来访问。

请注意，在本文的代码示例中，我们未指定多实例任务本身的资源文件，只有为池的 StartTask 和多实例设置的 [CommonResourceFiles][net_multiinsance_commonresfiles] 指定资源文件。

> [AZURE.IMPORTANT] 在命令行中，请始终使用环境变量 `AZ_BATCH_TASK_SHARED_DIR` 和 `AZ_BATCH_TASK_WORKING_DIR` 来引用这些目录。请勿尝试手动构造路径。

## 任务生存期

主要任务的生存期控制整个多实例任务的生存期。当主要任务退出时，所有子任务就会终止。主要任务的退出代码就是任务的退出代码，因此在重试用途上用于判断任务成功或失败。

如果任何子任务失败，例如退出时返回代码不是零，则整个多实例任务失败。然后终止并重试多实例任务，直到到达重试限制为止。

当你删除多实例任务时，Batch 服务也会删除主要任务和所有子任务。所有子任务目录及其文件从计算节点中删除，如同在标准任务中一样。

多实例任务的 [TaskConstraints][net_taskconstraints]（例如 [MaxTaskRetryCount][net_taskconstraint_maxretry]、[MaxWallClockTime][net_taskconstraint_maxwallclock] 和 [RetentionTime][net_taskconstraint_retention] 属性）都视为用于标准任务，并应用到主要任务和所有子任务。但是，如果在多实例任务添加到作业之后更改 [RetentionTime][net_taskconstraint_retention] 属性，此更改只应用到主要任务。所有的子任务将继续使用原始 [RetentionTime][net_taskconstraint_retention]。

如果最近的任务是多实例任务的一部分，计算节点的最近任务列表反映子任务的 ID。

## 获取有关子任务的信息

若要使用 Batch .NET 库获取子任务的详细信息，请调用 [CloudTask.ListSubtasks][net_task_listsubtasks] 方法。此方法返回所有子任务的相关信息，以及已执行任务的计算节点的相关信息。可以根据此信息判断每项子任务的根目录、池 ID、其当前状态、退出代码等等。可以使用此信息结合 [PoolOperations.GetNodeFile][poolops_getnodefile] 方法，以获取子任务的文件。请注意，此方法不返回主要任务 (ID 0) 的相关信息。

> [AZURE.NOTE] 除非另有指明，否则在多实例 [CloudTask][net_task] 本身执行的 Batch .NET 方法只应用到主要任务。例如，当在多实例任务上调用 [CloudTask.ListNodeFiles][net_task_listnodefiles] 方法时，只返回主要任务的文件。

以下代码段演示如何获取子任务信息，以及从它们执行所在的节点请求文件的内容。

		
		// Obtain the job and the multi-instance task from the Batch service
		CloudJob boundJob = batchClient.JobOperations.GetJob("mybatchjob");
		CloudTask myMultiInstanceTask = boundJob.GetTask("mymultiinstancetask");
		
		// Now obtain the list of subtasks for the task
		IPagedEnumerable<SubtaskInformation> subtasks = myMultiInstanceTask.ListSubtasks();
		
		// Asynchronously iterate over the subtasks and print their stdout and stderr
		// output if the subtask has completed
		await subtasks.ForEachAsync(async (subtask) =>
		{
		    Console.WriteLine("subtask: {0}", subtask.Id);
		    Console.WriteLine("exit code: {0}", subtask.ExitCode);
		
		    if (subtask.State == TaskState.Completed)
		    {
		        ComputeNode node =
					await batchClient.PoolOperations.GetComputeNodeAsync(subtask.ComputeNodeInformation.PoolId,		                                                                 subtask.ComputeNodeInformation.ComputeNodeId);
		
		        NodeFile stdOutFile = await node.GetNodeFileAsync(subtask.ComputeNodeInformation.TaskRootDirectory + "\" + Constants.StandardOutFileName);
		        NodeFile stdErrFile = await node.GetNodeFileAsync(subtask.ComputeNodeInformation.TaskRootDirectory + "\" + Constants.StandardErrorFileName);
		        stdOut = await stdOutFile.ReadAsStringAsync();
		        stdErr = await stdErrFile.ReadAsStringAsync();
		
		        Console.WriteLine("node: {0}:", node.Id);
		        Console.WriteLine("stdout.txt: {0}", stdOut);
		        Console.WriteLine("stderr.txt: {0}", stdErr);
		    }
		    else
		    {
		        Console.WriteLine("\tSubtask {0} is in state {1}", subtask.Id, subtask.State);
		    }
		});


## 后续步骤

- 你可以构建一个简单的 MS-MPI 应用程序，以便在 Batch 中测试多实例任务时使用。Microsoft HPC 和 Azure Batch 团队博客文章[如何编译和运行简单的 MS-MPI 程序][msmpi_howto]中包含有关使用 MS-MPI 创建简单 MPI 应用程序的演练。

- 有关 MS-MPI 的最新信息，请参阅 MSDN 上的 [Microsoft MPI][msmpi_msdn] 页面。

[api_net]: http://msdn.microsoft.com/library/azure/mt348682.aspx
[api_rest]: http://msdn.microsoft.com/library/azure/dn820158.aspx
[batch_explorer]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/BatchExplorer
[cmd_start]: https://technet.microsoft.com/library/cc770297.aspx
[github_samples]: https://github.com/Azure/azure-batch-samples
[msmpi_msdn]: https://msdn.microsoft.com/library/bb524831.aspx
[msmpi_sdk]: http://go.microsoft.com/FWLink/p/?LinkID=389556
[msmpi_howto]: http://blogs.technet.com/b/windowshpc/archive/2015/02/02/how-to-compile-and-run-a-simple-ms-mpi-program.aspx

[net_jobprep]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.jobpreparationtask.aspx
[net_multiinstance_class]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.multiinstancesettings.aspx
[net_multiinstance_prop]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudtask.multiinstancesettings.aspx
[net_multiinsance_commonresfiles]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.multiinstancesettings.commonresourcefiles.aspx
[net_multiinstance_coordcmdline]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.multiinstancesettings.coordinationcommandline.aspx
[net_multiinstance_numinstances]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.multiinstancesettings.numberofinstances.aspx
[net_pool]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudpool.aspx
[net_pool_create]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.pooloperations.createpool.aspx
[net_pool_starttask]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudpool.starttask.aspx
[net_resourcefile]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.resourcefile.aspx
[net_starttask]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.starttask.aspx
[net_starttask_cmdline]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.starttask.commandline.aspx
[net_task]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudtask.aspx
[net_taskconstraints]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.taskconstraints.aspx
[net_taskconstraint_maxretry]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.taskconstraints.maxtaskretrycount.aspx
[net_taskconstraint_maxwallclock]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.taskconstraints.maxwallclocktime.aspx
[net_taskconstraint_retention]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.taskconstraints.retentiontime.aspx
[net_task_listsubtasks]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudtask.listsubtasks.aspx
[net_task_listnodefiles]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudtask.listnodefiles.aspx
[poolops_getnodefile]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.pooloperations.getnodefile.aspx

[rest_multiinstance]: https://msdn.microsoft.com/library/azure/mt637905.aspx

[1]: ./media/batch-mpi/batch_mpi_01.png "多实例概述"

<!---HONumber=Mooncake_0503_2016-->