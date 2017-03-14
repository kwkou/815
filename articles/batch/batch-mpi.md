<properties
    pageTitle="使用多实例任务运行 MPI 应用程序 - Azure Batch | Azure"
    description="了解如何在 Azure Batch 中使用多实例任务类型执行消息传递接口 (MPI) 应用程序。"
    services="batch"
    documentationcenter=".net"
    author="tamram"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="83e34bd7-a027-4b1b-8314-759384719327"
    ms.service="batch"
    ms.devlang="multiple"
    ms.topic="article"
    ms.tgt_pltfrm="vm-windows"
    ms.workload="big-compute"
    ms.date="01/23/2017"
    wacn.date="03/14/2017"
    ms.author="tamram" />

# 在 Azure Batch 中使用多实例任务来执行消息传递接口 (MPI) 应用程序
使用多实例任务可在多个计算节点上同时运行 Azure Batch 任务。这些任务可在 Batch 中实现高性能计算方案，例如消息传递接口 (MPI) 应用程序。本文介绍如何使用 [Batch .NET][api_net] 库执行多实例任务。

> [AZURE.NOTE]
虽然本文中的示例重点介绍 Batch .NET、MS-MPI 和 Windows 计算节点，但此处讨论的多实例任务概念也适用于其他平台和技术（例如 Linux 节点上的 Python 和 Intel MPI）。
>
>

## 多实例任务概述
在 Batch 中，每个任务通常是在单个计算节点上执行 -- 将多个任务提交给作业，Batch 服务将每个任务安排在节点上执行。但是，可以通过配置任务的“多实例设置”，告知 Batch 改为创建一个主要任务和多个子任务，然后在多个节点上执行任务。

![多实例任务概述][1]  


将具有多实例设置的任务提交给作业时，Batch 执行多实例任务特有的几个步骤：

1. Batch 服务根据多实例设置创建一个**主要任务**和多个**子任务**。任务（主要任务和所有子任务）的总数与用户在多实例设置中指定的**实例**（计算节点）数相符。
2. Batch 将其中一个计算节点指定为**主**节点，将主要任务安排在主节点上执行，将子任务安排在已分配给多实例任务的剩余计算节点上执行，一个节点一个子任务。
3. 主要任务和所有子任务会下载在多实例设置中指定的任何**公共资源文件**。
4. 下载公共资源文件之后，主任务和子任务将执行多实例设置中指定的**协调命令**。通常使用协调命令准备节点，以便执行任务。该操作可能包括启动后台服务（例如 [Microsoft MPI][msmpi_msdn] 的 `smpd.exe`），以及验证节点是否已就绪，能够处理节点间消息。
5. 在主要任务和所有子任务成功完成协调命令*以后*，主要任务会在主节点上执行**应用程序命令**。应用程序命令是多实例任务本身的命令行，只由主要任务执行。在基于 [MS-MPI][msmpi_msdn] 的解决方案中，用户将在此处使用 `mpiexec.exe` 执行已启用 MPI 的应用程序。

> [AZURE.NOTE]
虽然“多实例任务”在功能上不同，但并不是特殊的任务类型，例如 [StartTask][net_starttask] 或 [JobPreparationTask][net_jobprep]。多实例任务只是已设置多实例设置的标准 Batch 任务（Batch .NET 中的 [CloudTask][net_task]）。在本文中，我们将它称为多实例任务。
>
>

## 多实例任务的要求
多实例任务需要有已启用节点间通信和已禁用并发任务执行的池。如果尝试在已禁用节点间通信，或 *maxTasksPerNode* 值大于 1 的池中运行多实例任务，则永远不排定任务 -- 它无限期停留在“活动”状态。本代码片段显示如何使用 Batch .NET 库创建这种池。

csharp

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


此外，多实例任务*只能*在 **2015 年 12 月 14 日之后创建的池**中的节点上执行。

### 使用 StartTask 安装 MPI
若要通过多实例任务运行 MPI 应用程序，首先需在池中的计算节点上安装 MPI 实现（例如 MS-MPI 或 Intel MPI）。这是使用 [StartTask][net_starttask] 的好时机，每当节点加入池或重新启动时，它就会执行。此代码片段创建一个 StartTask，将 MS-MPI 安装程序包指定为[资源文件][net_resourcefile]。资源文件下载到节点之后，将执行启动任务的命令行。在本示例中，命令行执行 MS-MPI 的无人参与安装。

csharp

	// Create a StartTask for the pool which we use for installing MS-MPI on
	// the nodes as they join the pool (or when they are restarted).
	StartTask startTask = new StartTask
	{
	    CommandLine = "cmd /c MSMpiSetup.exe -unattend -force",
	    ResourceFiles = new List<ResourceFile> { new ResourceFile("https://mystorageaccount.blob.core.chinacloudapi.cn/mycontainer/MSMpiSetup.exe", "MSMpiSetup.exe") },
	    RunElevated = true,
	    WaitForSuccess = true
	};
	myCloudPool.StartTask = startTask;
	
	// Commit the fully configured pool to the Batch service to actually create
	// the pool and its compute nodes.
	await myCloudPool.CommitAsync();


### 远程直接内存访问 (RDMA)
在 Batch 池中为计算节点选择[支持 RDMA 的大小](/documentation/articles/virtual-machines-windows-a8-a9-a10-a11-specs?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json/)（例如 A9）时，MPI 应用程序可以使用 Azure 的高性能、低延迟的远程直接内存访问 (RDMA) 网络。

在以下文章中查找指定为“支持 RDMA”的大小：

- **CloudServiceConfiguration** 池

  - [云服务的大小](/documentation/articles/cloud-services-sizes-specs/)（仅 Windows）
- **VirtualMachineConfiguration** 池

  - [Azure 中虚拟机的大小](/documentation/articles/virtual-machines-linux-sizes?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json/) (Linux)
  - [Azure 中虚拟机的大小](/documentation/articles/virtual-machines-windows-sizes?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json/) (Windows)

> [AZURE.NOTE]
若要充分利用 [Linux 计算节点](/documentation/articles/batch-linux-nodes/)上的 RDMA，必须使用节点上的 **Intel MPI**。有关 CloudServiceConfiguration 和 VirtualMachineConfiguration 池的详细信息，请参阅[批处理功能概述](/documentation/articles/batch-api-basics/)的“池”部分。
>
>

## 使用 Batch .NET 创建多实例任务
我们已讨论池的要求和 MPI 包安装，现在让我们创建多实例任务。在此代码段中，我们将创建一个标准 [CloudTask][net_task]，然后配置其 [MultiInstanceSettings][net_multiinstance_prop] 属性。如前所述，多实例任务并不是独特的任务类型，而只是已配置多实例设置的标准 Batch 任务。

csharp

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
	    new ResourceFile("https://mystorageaccount.blob.core.chinacloudapi.cn/mycontainer/MyMPIApplication.exe",
	                     "MyMPIApplication.exe")
	    }
	};

	// Submit the task to the job. Batch will take care of splitting it into subtasks and
	// scheduling them for execution on the nodes.
	await myBatchClient.JobOperations.AddTaskAsync("mybatchjob", myMultiInstanceTask);

## 主要任务和子任务
创建任务的多实例设置时，需要指定用于执行任务的计算节点数目。当你将任务提交给作业时，Batch 服务将创建一个主要任务和足够的子任务，并且合计符合指定的节点数。

系统分配范围介于 0 到 numberOfInstances - 1 的整数 ID 给这些任务。ID 为 0 的任务是主要任务，其他所有 ID 都是子任务。例如，如果为任务创建以下多实例设置，则主要任务的 ID 为 0，而子任务的 ID 为 1 到 9。

csharp

	int numberOfNodes = 10;
	myMultiInstanceTask.MultiInstanceSettings = new MultiInstanceSettings(numberOfNodes);

### 主节点
当用户提交多实例任务时，Batch 服务会将其中一个计算节点指定为“主”节点，并将主要任务安排在主节点上执行。子任务安排在已分配给多实例任务的剩余节点上执行。

## 协调命令
主要任务和子任务都执行协调命令。

阻止调用协调命令 -- 在所有子任务的协调命令成功返回之前，Batch 不执行应用程序命令。因此，协调命令应该启动任何所需的后台服务，确认它们已准备好可供使用，然后退出。例如，在使用 MS-MPI 第 7 版的方案中，此协调命令在节点上启动 SMPD 服务，然后退出：

	cmd /c start cmd /c ""%MSMPI_BIN%\smpd.exe"" -d

请注意此协调命令中使用 `start`。这是必需的，因为 `smpd.exe` 应用程序不会在执行后立即返回。如果不使用 [start][cmd_start] 命令，此协调命令就不返回，因此将阻止执行应用程序命令。

## 应用程序命令
主要任务及所有子任务完成执行协调命令之后，只有主要任务执行多实例任务的命令行。我们将此命令行称为**应用程序命令**，以便与协调命令区分开来。

对于 MS-MPI 应用程序，请使用应用程序命令通过 `mpiexec.exe` 执行已启用 MPI 的应用程序。例如，以下是使用 MS-MPI 第 7 版的方案所执行的应用程序命令：

	cmd /c ""%MSMPI_BIN%\mpiexec.exe"" -c 1 -wdir %AZ_BATCH_TASK_SHARED_DIR% MyMPIApplication.exe


> [AZURE.NOTE]
由于 MS-MPI 的 `mpiexec.exe` 默认使用 `CCP_NODES` 变量（请参阅[环境变量](#environment-variables)），上述示例应用程序命令行已排除该变量。
>
>

## 环境变量
批处理创建的多个[环境变量][msdn_env_var]特定于已分配给某个多实例任务的计算节点上的多实例任务。协调命令行和应用程序命令行可以引用这些环境变量，就像其所执行的脚本和程序一样。

以下环境变量由多实例任务所使用的 Batch 服务创建：

- `CCP_NODES`  

- `AZ_BATCH_NODE_LIST`
- `AZ_BATCH_HOST_LIST`
- `AZ_BATCH_MASTER_NODE`
- `AZ_BATCH_TASK_SHARED_DIR`
- `AZ_BATCH_IS_CURRENT_NODE_MASTER`  


如需这些环境变量以及其他批处理计算节点环境变量的完整详细信息（包括内容和可见性），请参阅 [Compute node environment variables][msdn_env_var]（计算节点环境变量）。

> [AZURE.TIP]
此 Batch Linux MPI 代码示例包含一个示例，介绍了如何使用其中几个环境变量。[coordination-cmd][coord_cmd_example] Bash 脚本可从 Azure 存储下载常用应用程序和输入文件、在主节点上启用网络文件系统 (NFS) 共享，以及将其他分配给多实例任务的节点配置为 NFS 客户端。
>
>

## 资源文件
多实例任务需要考虑两组资源文件：所有任务（主要任务和子任务）下载的一般资源文件，以及为多实例任务本身指定的资源文件（只有主要任务下载）。

可以在任务的多实例设置中指定一个或多个通用资源文件。主要任务及所有子任务从 [Azure 存储](/documentation/articles/storage-introduction/)将这些通用资源文件下载到每个节点的**任务共享目录**。可以使用 `AZ_BATCH_TASK_SHARED_DIR` 环境变量从应用程序命令和协调命令行访问任务共享目录。`AZ_BATCH_TASK_SHARED_DIR` 路径在所有分配给多实例任务的节点上都是相同的，因此可在主要任务和所有子任务之间共享单个协调命令。从远程访问的意义上来说，Batch 并不“共享”目录，但用户可将其用作装入点或共享点，如此前在有关环境变量的提示中所述。

默认情况下，为多实例任务本身指定的资源文件下载到任务的工作目录 `AZ_BATCH_TASK_WORKING_DIR`。如前所述，仅主要任务下载为多实例任务本身指定的资源文件（与公共资源文件相比）。

> [AZURE.IMPORTANT]
在命令行中，请始终使用环境变量 `AZ_BATCH_TASK_SHARED_DIR` 和 `AZ_BATCH_TASK_WORKING_DIR` 来引用这些目录。请勿尝试手动构造路径。
>
>

## 任务生存期
主要任务的生存期控制整个多实例任务的生存期。当主要任务退出时，所有子任务就会终止。主要任务的退出代码就是任务的退出代码，因此在重试用途上用于判断任务成功或失败。

如果任何子任务失败，例如退出时返回代码不是零，则整个多实例任务失败。然后终止并重试多实例任务，直到到达重试限制为止。

当你删除多实例任务时，Batch 服务也会删除主要任务和所有子任务。所有子任务目录及其文件从计算节点中删除，如同在标准任务中一样。

多实例任务的 [TaskConstraints][net_taskconstraints]（例如 [MaxTaskRetryCount][net_taskconstraint_maxretry]、[MaxWallClockTime][net_taskconstraint_maxwallclock] 和 [RetentionTime][net_taskconstraint_retention] 属性）都视为用于标准任务，并应用到主要任务和所有子任务。但是，如果在多实例任务添加到作业之后更改 [RetentionTime][net_taskconstraint_retention] 属性，此更改只应用到主要任务。所有的子任务继续使用原始 [RetentionTime][net_taskconstraint_retention]。

如果最近的任务是多实例任务的一部分，计算节点的最近任务列表反映子任务的 ID。

## 获取有关子任务的信息
若要使用 Batch .NET 库获取子任务的详细信息，请调用 [CloudTask.ListSubtasks][net_task_listsubtasks] 方法。此方法返回所有子任务的相关信息，以及已执行任务的计算节点的相关信息。可以根据此信息判断每项子任务的根目录、池 ID、其当前状态、退出代码等等。可以使用此信息结合 [PoolOperations.GetNodeFile][poolops_getnodefile] 方法，以获取子任务的文件。请注意，此方法不返回主要任务 (ID 0) 的相关信息。

> [AZURE.NOTE]
除非另有指明，否则在多实例 [CloudTask][net_task] 本身执行的 Batch .NET 方法只应用到主要任务。例如，当在多实例任务上调用 [CloudTask.ListNodeFiles][net_task_listnodefiles] 方法时，只返回主要任务的文件。
>
>

以下代码片段演示如何获取子任务信息，以及从它们执行所在的节点请求文件的内容。

csharp

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
				await batchClient.PoolOperations.GetComputeNodeAsync(subtask.ComputeNodeInformation.PoolId,
	                                                                 subtask.ComputeNodeInformation.ComputeNodeId);
	
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

## 代码示例
GitHub 上的 [MultiInstanceTasks][github_mpi] 代码示例演示了如何通过多实例任务在批处理计算节点上运行 [MS-MPI][msmpi_msdn] 应用程序。按[准备](#preparation)和[执行](#execution)中的步骤运行该示例。

### 准备工作 <a name="preparation"></a>
1. 执行 [How to compile and run a simple MS-MPI program][msmpi_howto]（如何编译和运行简单的 MS-MPI 程序）中的头两个步骤。这样即可满足下一步的先决条件。
2. 生成 [MPIHelloWorld][helloworld_proj] 示例 MPI 程序的*发行* 版。该程序是将在计算节点上通过多实例任务运行的程序。
3. 创建包含 `MPIHelloWorld.exe`（在步骤 2 构建）和 `MSMpiSetup.exe`（在步骤 1 下载）的 zip 文件。下一步需要将此 zip 文件作为应用程序包上载。
4. 通过 [Azure 门户预览][portal]创建名为“MPIHelloWorld”的批处理[应用程序](/documentation/articles/batch-application-packages/)，并将在上一步创建的 zip 文件指定为“1.0”版应用程序包。有关详细信息，请参阅[上载和管理应用程序](/documentation/articles/batch-application-packages/#upload-and-manage-applications/)。

> [AZURE.TIP]
生成*发行*版 `MPIHelloWorld.exe`，这样就不需在应用程序包中包括任何其他依赖项（例如 `msvcp140d.dll` 或 `vcruntime140d.dll`）。
>
>

### 执行 <a name="execution"></a>
1. 从 GitHub 下载 [azure-batch-samples][github_samples_zip]。
2. 在 Visual Studio 2015 中打开 MultiInstanceTasks **解决方案**。`MultiInstanceTasks.sln` 解决方案文件位于：

    `azure-batch-samples\CSharp\ArticleProjects\MultiInstanceTasks`  

3. 将批处理和存储帐户凭据输入 **Microsoft.Azure.Batch.Samples.Common** 项目中的 `AccountSettings.settings`。
4. **生成并运行** MultiInstanceTasks 解决方案，在批处理池中的计算节点上执行 MPI 示例应用程序。
5. *可选*：通过 [Azure 门户预览][portal]或[批处理资源管理器][batch_explorer]检查示例池、作业和任务（“MultiInstanceSamplePool”、“MultiInstanceSampleJob”、“MultiInstanceSampleTask”），然后再删除这些资源。

> [AZURE.TIP]
如果没有 Visual Studio，可下载免费版 [Visual Studio Community][visual_studio]。
>
>

`MultiInstanceTasks.exe` 的输出与下面类似：

	Creating pool [MultiInstanceSamplePool]...
	Creating job [MultiInstanceSampleJob]...
	Adding task [MultiInstanceSampleTask] to job [MultiInstanceSampleJob]...
	Awaiting task completion, timeout in 00:30:00...

	Main task [MultiInstanceSampleTask] is in state [Completed] and ran on compute node [tvm-1219235766_1-20161017t162002z]:
	---- stdout.txt ----
	Rank 2 received string "Hello world" from Rank 0
	Rank 1 received string "Hello world" from Rank 0

	---- stderr.txt ----

	Main task completed, waiting 00:00:10 for subtasks to complete...

	---- Subtask information ----
	subtask: 1
	        exit code: 0
	        node: tvm-1219235766_3-20161017t162002z
	        stdout.txt:
	        stderr.txt:
	subtask: 2
	        exit code: 0
	        node: tvm-1219235766_2-20161017t162002z
	        stdout.txt:
	        stderr.txt:

	Delete job? [yes] no: yes
	Delete pool? [yes] no: yes

	Sample complete, hit ENTER to exit...


## 后续步骤
- Microsoft HPC 和 Azure 批处理团队博客讨论 [MPI support for Linux on Azure Batch][blog_mpi_linux]（Azure 批处理上针对 Linux 的 MPI 支持），并介绍如何将 [OpenFOAM][openfoam] 与批处理配合使用。可以查找适用于 [OpenFOAM example on GitHub][github_mpi]（GitHub 上的 OpenFOAM 示例）的 Python 代码示例。
- 了解如何[创建 Linux 计算节点池](/documentation/articles/batch-linux-nodes/)，以便将其用在 Azure 批处理 MPI 解决方案中。

[helloworld_proj]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/ArticleProjects/MultiInstanceTasks/MPIHelloWorld

[api_net]: http://msdn.microsoft.com/zh-cn/library/azure/mt348682.aspx
[api_rest]: http://msdn.microsoft.com/zh-cn/library/azure/dn820158.aspx
[batch_explorer]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/BatchExplorer
[blog_mpi_linux]: https://blogs.technet.microsoft.com/windowshpc/2016/07/20/introducing-mpi-support-for-linux-on-azure-batch/
[cmd_start]: https://technet.microsoft.com/zh-cn/library/cc770297.aspx
[coord_cmd_example]: https://github.com/Azure/azure-batch-samples/blob/master/Python/Batch/article_samples/mpi/data/linux/openfoam/coordination-cmd
[github_mpi]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/ArticleProjects/MultiInstanceTasks
[github_samples]: https://github.com/Azure/azure-batch-samples
[github_samples_zip]: https://github.com/Azure/azure-batch-samples/archive/master.zip
[msdn_env_var]: https://msdn.microsoft.com/zh-cn/library/azure/mt743623.aspx
[msmpi_msdn]: https://msdn.microsoft.com/zh-cn/library/bb524831.aspx
[msmpi_sdk]: http://go.microsoft.com/FWLink/p/?LinkID=389556
[msmpi_howto]: http://blogs.technet.com/b/windowshpc/archive/2015/02/02/how-to-compile-and-run-a-simple-ms-mpi-program.aspx
[openfoam]: http://www.openfoam.com/
[visual_studio]: https://www.visualstudio.com/vs/community/

[net_jobprep]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.jobpreparationtask.aspx
[net_multiinstance_class]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.multiinstancesettings.aspx
[net_multiinstance_prop]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.cloudtask.multiinstancesettings.aspx
[net_multiinsance_commonresfiles]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.multiinstancesettings.commonresourcefiles.aspx
[net_multiinstance_coordcmdline]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.multiinstancesettings.coordinationcommandline.aspx
[net_multiinstance_numinstances]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.multiinstancesettings.numberofinstances.aspx
[net_pool]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.cloudpool.aspx
[net_pool_create]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.pooloperations.createpool.aspx
[net_pool_starttask]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.cloudpool.starttask.aspx
[net_resourcefile]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.resourcefile.aspx
[net_starttask]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.starttask.aspx
[net_starttask_cmdline]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.starttask.commandline.aspx
[net_task]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.cloudtask.aspx
[net_taskconstraints]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.taskconstraints.aspx
[net_taskconstraint_maxretry]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.taskconstraints.maxtaskretrycount.aspx
[net_taskconstraint_maxwallclock]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.taskconstraints.maxwallclocktime.aspx
[net_taskconstraint_retention]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.taskconstraints.retentiontime.aspx
[net_task_listsubtasks]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.cloudtask.listsubtasks.aspx
[net_task_listnodefiles]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.cloudtask.listnodefiles.aspx
[poolops_getnodefile]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.pooloperations.getnodefile.aspx

[portal]: https://portal.azure.cn
[rest_multiinstance]: https://msdn.microsoft.com/zh-cn/library/azure/mt637905.aspx

[1]: ./media/batch-mpi/batch_mpi_01.png "多实例概述"

<!---HONumber=Mooncake_0306_2017-->
<!---Update_Description: wording update -->