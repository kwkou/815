<properties
    pageTitle="保存 Azure Batch 中的作业和任务输出 | Azure"
    description="了解如何使用 Azure 存储作为 Batch 任务和作业输出的持久性存储，在 Azure 门户预览中查看这些保存的输出。"
    services="batch"
    documentationcenter=".net"
    author="tamram"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="16e12d0e-958c-46c2-a6b8-7843835d830e"
    ms.service="batch"
    ms.devlang="multiple"
    ms.topic="article"
    ms.tgt_pltfrm="vm-windows"
    ms.workload="big-compute"
    ms.date="01/05/2017"
    wacn.date="02/22/2017"
    ms.author="tamram" />  


# 保存 Azure Batch 作业和任务输出
在 Batch 中运行的任务通常会生成输出，这些输出必须得到存储，以便今后由作业中的其他任务和/或执行作业的客户端应用程序检索。此输出可能是处理输入数据后创建的文件，也可能是与任务执行关联的日志文件。本文将介绍一个 .NET 类库，它使用基于约定的技术在 Azure Blob 存储中保存此类任务输出，这样，即使在删除池、作业和计算节点后，你也可以使用这些输出。

使用本文所述的方法，还可以在 [Azure 门户预览][portal]上的“保存的输出文件”和“保存的日志”中查看任务输出。

![门户中“保存的输出文件”和“保存的日志”选择器][1]  


> [AZURE.NOTE]
本文中所述的 [Azure Batch 文件约定][nuget_package] .NET 类库目前以预览版提供。在正式版推出之前，本文所述的某些功能可能会更改。
> 
> 

## 任务输出注意事项
设计 Batch 解决方案时，必须考虑几个与作业和任务输出相关的因素。

- **计算节点生存期**：计算节点通常是瞬态的，尤其是在启用了自动缩放的池中。在某个节点上运行的任务的输出仅在该节点存在时才可用，并且仅在你为任务设置的文件保留时间范围内才可用。为了确保保留任务输出，你的任务必须将其输出文件上载到持久性存储，例如 Azure 存储空间。
- **输出存储**：若要将任务输出数据保存到持久性存储，可以在任务代码中使用 [Azure 存储空间 SDK](/documentation/articles/storage-dotnet-how-to-use-blobs/)，将任务输出上载到 Blob 存储容器中。如果你实现了容器和文件命名约定，则客户端应用程序或作业中的其他任务可以根据该约定查找并下载此输出。
- **输出检索**：你可以直接从池中的计算节点检索任务输出，如果任务保存了其输出，则你可以从 Azure 存储空间检索任务输出。若要直接从计算节点检索任务输出，需要获取文件名及其在节点上的输出位置。如果将输出保存到 Azure 存储，则下游任务或客户端应用程序必须获得 Azure 存储中文件的完整路径才能使用 Azure 存储 SDK 来下载输出。
- **查看输出**：导航到 Azure 门户预览中的某个 Batch 任务并选择“节点上的文件”时，将看到与该任务关联的所有文件，而不仅仅是想要查看的输出文件。同样，计算节点上的文件仅在该节点存在时才可用，并且仅在你为任务设置的文件保留时间范围内才可用。若要在门户或某个应用程序（例如 [Azure 存储资源管理器][storage_explorer]）中查看你已保存到 Azure 存储空间的任务输出，必须知道该文件的位置并直接导航到该文件。

## 有关保存的输出的帮助
为了帮助你更轻松地保存作业和任务输出，Batch 团队定义并实现了一组命名约定以及一个 .NET 类库（[Azure Batch 文件约定][nuget_package]库），供你在 Batch 应用程序中使用。此外，Azure 门户预览可识别这些命名约定，因此可以轻松找到使用该库存储的文件。

## 使用文件约定库
[Azure Batch 文件约定][nuget_package]是一个 .NET 类库，Batch .NET 应用程序可以使用它来轻松向 Azure 存储空间存储任务输出，以及从中检索任务输出。该库可在任务代码和客户端代码中使用 -- 在任务代码中用于保存文件，在客户端代码用于列出和检索文件。任务还可以使用该库来检索上游任务的输出（例如，在[任务依赖性](/documentation/articles/batch-task-dependencies/)方案中）。

该约定库负责确保存储容器和任务输出文件根据约定命名，并在保存到 Azure 存储空间时上载到正确的位置。当你检索输出时，可以按 ID 和用途列出或者检索给定作业或任务的输出，从而轻松找到所需的输出，而无需知道文件名或者文件在存储空间中的位置。

例如，你可以使用该库“列出任务 7 的所有中间文件”或“获取作业 *mymovie* 的缩略图预览”，而无需知道文件名或者文件在存储帐户中的位置。

### 获取该库
可以从 [NuGet][nuget_package] 获取该库，其中包含新类并使用新方法扩展了 [CloudJob][net_cloudjob] 和 [CloudTask][net_cloudtask] 类。你可以使用 [NuGet 库包管理器][nuget_manager]将它添加到 Visual Studio 项目。

> [AZURE.TIP]
可以在 GitHub 上的用于 .NET 的 Azure SDK 存储库中找到 Azure Batch 文件约定库的[源代码][github_file_conventions]。
> 
> 

## 要求：链接的存储帐户  <a name="requirement-linked-storage-account"></a>
若要使用文件约定库将输出存储到持久性存储并在 Azure 门户预览中查看这些输出，必须[将 Azure 存储帐户链接](/documentation/articles/batch-application-packages/#link-a-storage-account/)到 Batch 帐户。如果尚未这样做，请使用 Azure 门户预览将存储帐户链接到 Batch 帐户：

“Batch 帐户”边栏选项卡 >“设置”>“存储帐户”>“存储帐户”（无）> 在你的订阅中选择一个存储帐户

有关链接存储帐户的更详细演练，请参阅 [Application deployment with Azure Batch application packages](/documentation/articles/batch-application-packages/)（使用 Azure Batch 应用程序包部署应用程序）。

## 保存输出
使用文件约定库保存作业和任务输出时需要执行两个主要操作：创建存储容器，将输出保存到容器。

> [AZURE.WARNING]
由于所有作业和任务输出将存储在同一个容器中，因此，如果有大量的任务同时尝试保存文件，则可能会强制实施[存储节流限制](/documentation/articles/storage-performance-checklist/#blobs/)。
> 
> 

### 创建存储容器
在任务开始将输出保存到存储空间之前，必须创建一个 Blob 存储容器，以便任务将其输出上载到其中。可通过调用 [CloudJob][net_cloudjob].[PrepareOutputStorageAsync][net_prepareoutputasync] 来执行此操作。此扩展方法使用 [CloudStorageAccount][net_cloudstorageaccount] 对象作为参数，并且会创建一个容器，该容器的命名能使 Azure 门户预览以及本文稍后介绍的检索方法发现其内容。

通常会将此代码放入客户端应用程序 -- 即创建池、作业和任务的应用程序。

csharp

	CloudJob job = batchClient.JobOperations.CreateJob(
		"myJob",
		new PoolInformation { PoolId = "myPool" });
	
	// Create reference to the linked Azure Storage account
	CloudStorageAccount linkedStorageAccount =
		new CloudStorageAccount(myCredentials, true);
	
	// Create the blob storage container for the outputs
	await job.PrepareOutputStorageAsync(linkedStorageAccount);

### 存储任务输出
在 Blob 存储中准备一个容器后，任务可以使用文件约定库中的 [TaskOutputStorage][net_taskoutputstorage] 类将输出保存到该容器。

在任务代码中，请先创建一个 [TaskOutputStorage][net_taskoutputstorage] 对象，然后，当任务完成其工作时，会调用 [TaskOutputStorage][net_taskoutputstorage].[SaveAsync][net_saveasync] 方法将其输出保存到 Azure 存储空间。

csharp

	CloudStorageAccount linkedStorageAccount = new CloudStorageAccount(myCredentials);
	string jobId = Environment.GetEnvironmentVariable("AZ_BATCH_JOB_ID");
	string taskId = Environment.GetEnvironmentVariable("AZ_BATCH_TASK_ID");
	
	TaskOutputStorage taskOutputStorage = new TaskOutputStorage(
		linkedStorageAccount, jobId, taskId);
	
	/* Code to process data and produce output file(s) */
	
	await taskOutputStorage.SaveAsync(TaskOutputKind.TaskOutput, "frame_full_res.jpg");
	await taskOutputStorage.SaveAsync(TaskOutputKind.TaskPreview, "frame_low_res.jpg");

“output kind”参数可保存的文件分类。有四个预定义的 [TaskOutputKind][net_taskoutputkind] 类型：“TaskOutput”、“TaskPreview”、“TaskLog”和“TaskIntermediate”。 你还可以指定自定义类型（如果在工作流中有用的话）。

以后在 Batch 中查询给定任务的已保存输出时，可以使用这些输出类型来指定要列出哪种类型的输出。换而言之，当你列出某个任务的输出时，可以根据某种输出类型来筛选列表。例如，“列出任务 *109* 的*预览*输出。” 本文稍后的[检索输出](#retrieve-output)部分中详细介绍了如何列出和检索输出。

> [AZURE.TIP]
输出类型还会指定特定的文件将显示在 Azure 门户预览中的哪个位置：*TaskOutput* 分类的文件将显示在“任务输出文件”中，*TaskLog* 文件将显示在“任务日志”中。
> 
> 

### 存储作业输出
除了存储任务输出以外，你还可以存储与整个作业关联的输出。例如，在电影渲染作业的合并任务中，可以将完全渲染的电影保存为作业输出。作业完成后，客户端应用程序只需列出并检索该作业的输出，而不需要查询各个任务。

通过调用 [JobOutputStorage][net_joboutputstorage].[SaveAsync][net_joboutputstorage_saveasync] 方法存储作业输出，并指定 [JobOutputKind][net_joboutputkind] 和文件名：

	CloudJob job = await batchClient.JobOperations.GetJobAsync(jobId);
	JobOutputStorage jobOutputStorage = job.OutputStorage(linkedStorageAccount);
	
	await jobOutputStorage.SaveAsync(JobOutputKind.JobOutput, "mymovie.mp4");
	await jobOutputStorage.SaveAsync(JobOutputKind.JobPreview, "mymovie_preview.mp4");

与用于任务输出的 TaskOutputKind 一样，你可以使用 [JobOutputKind][net_joboutputkind] 参数来分类作业的保存文件。以后可以使用此参数查询（列出）特定的输出类型。JobOutputKind 包括输出和预览类型，并支持创建自定义类型。

### 存储任务日志
除了在任务或作业完成时将文件保持到持久性存储以外，你还可能觉得有必要保存执行某个任务期间更新的文件 -- 例如，日志文件或 `stdout.txt` 和 `stderr.txt`。为此，Azure Batch 文件约定库提供了 [TaskOutputStorage][net_taskoutputstorage].[SaveTrackedAsync][net_savetrackedasync] 方法。使用 [SaveTrackedAsync][net_savetrackedasync]，可以跟踪对节点上的文件所做的更新（按照指定的间隔），并将这些更新保持到 Azure 存储空间。

在以下代码片段中，我们将在执行任务期间，每隔 15 秒使用 [SaveTrackedAsync][net_savetrackedasync] 更新 Azure 存储空间中的 `stdout.txt`：

csharp

	TimeSpan stdoutFlushDelay = TimeSpan.FromSeconds(3);
	string logFilePath = Path.Combine(
		Environment.GetEnvironmentVariable("AZ_BATCH_TASK_DIR"), "stdout.txt");

	// The primary task logic is wrapped in a using statement that sends updates to
	// the stdout.txt blob in Storage every 15 seconds while the task code runs.
	using (ITrackedSaveOperation stdout =
			await taskStorage.SaveTrackedAsync(
			TaskOutputKind.TaskLog,
			logFilePath,
			"stdout.txt",
			TimeSpan.FromSeconds(15)))
	{
		/* Code to process data and produce output file(s) */

		// We are tracking the disk file to save our standard output, but the
		// node agent may take up to 3 seconds to flush the stdout stream to
		// disk. So give the file a moment to catch up.
	 	await Task.Delay(stdoutFlushDelay);
	}

`Code to process data and produce output file(s)` 只是任务通常会执行的代码的占位符。例如，代码可能会从 Azure 存储下载数据，并对其执行转换或计算。此代码片段的重要部分演示了如何在 `using` 中包装此类代码，以定期使用 [SaveTrackedAsync][net_savetrackedasync] 更新文件。

`Task.Delay` 必须位于此 `using` 块的末尾，确保节点代理有时间将标准输出的内容刷新到节点上的 stdout.txt 文件（节点代理是在池中的每个节点上运行，在节点与 Batch 服务之间提供命令和控制接口的程序）。若没有此延迟，可能会遗漏最后几秒的输出。并非所有文件都需要此延迟。

> [AZURE.NOTE]
启用 SaveTrackedAsync 文件跟踪时，只会在 Azure 存储空间中保存被跟踪文件的*追加*内容。此方法只应该用于跟踪非轮转的日志文件或追加到的其他文件，也就是说，数据在更新时只会添加到文件末尾。
> 
> 

## 检索输出  <a name="retrieve-output"></a>
使用 Azure Batch 文件约定库检索保存的输出时，将以任务和作业为中心的方式执行此操作。你可以请求给定任务或作业的输出，而无需知道输出在 Blob 存储中的路径，甚至不需要知道其文件名。你只需提出，“给我返回任务 *109* 的输出文件。”

以下代码片段将循环访问某个作业的所有任务，列显有关该任务的输出文件的一些信息，然后从存储空间下载该任务的文件。

csharp

	foreach (CloudTask task in myJob.ListTasks())
	{
	    foreach (TaskOutputStorage output in
			task.OutputStorage(storageAccount).ListOutputs(
				TaskOutputKind.TaskOutput))
	    {
	        Console.WriteLine($"output file: {output.FilePath}");
	
			output.DownloadToFileAsync(
				$"{jobId}-{output.FilePath}",
				System.IO.FileMode.Create).Wait();
	    }
	}

## 任务输出和 Azure 门户预览
Azure 门户预览将显示使用 Azure Batch 文件约定自述文件中提到的命名约定保存到链接的 Azure 存储帐户中的任务输出和日志。你可以使用所选语言自行实现这些约定，也可以在 .NET 应用程序中使用该文件约定库。

### 启用门户显示
若要在门户中显示输出，必须满足以下要求：

1. [将 Azure 存储帐户链接](#requirement-linked-storage-account)到你的 Batch 帐户。
2. 保存输出时遵循存储容器和文件的预定义命名约定。可在文件约定库的自述文件中找到这些约定的定义。如果你使用 [Azure Batch 文件约定][nuget_package]库来保存输出，则可以满足此要求。

### 在门户中查看输出
若要在 Azure 门户预览中查看任务输出和日志，请导航到要查看其输出的任务，然后单击“保存的输出文件”或“保存的日志”。下图显示了 ID 为“007”的任务的“保存的输出文件”：

![Azure 门户预览中的“任务输出”边栏选项卡][2]

## 代码示例
[PersistOutputs][github_persistoutputs] 示例项目是 GitHub 上的 [Azure Batch 代码示例][github_samples]之一。此 Visual Studio 2015 解决方案演示如何使用 Azure Batch 文件约定库将任务输出保存到持久性存储。若要运行该示例，请遵循以下步骤：

1. 在 **Visual Studio 2015** 中打开该项目。
2. 将你的 Batch 和存储**帐户凭据**添加到 Microsoft.Azure.Batch.Samples.Common 项目中的 **AccountSettings.settings**。
3. **生成**（但不要运行）该解决方案。根据提示还原所有 NuGet 包。
4. 使用 Azure 门户预览上载 **PersistOutputsTask** 的[应用程序包](/documentation/articles/batch-application-packages/)。在 .zip 包中包含 `PersistOutputsTask.exe` 及其依赖程序集，将应用程序 ID 设置为“PersistOutputsTask”，将应用程序包版本设置为“1.0”。
5. **启动**（运行）**PersistOutputs** 项目。

## 后续步骤
### 应用程序部署
使用 Batch 的[应用程序包](/documentation/articles/batch-application-packages/)功能，可以轻松地部署任务在计算节点上执行的应用程序并对其进行版本控制。

### 安装应用程序和暂存数据
有关准备节点以运行任务的各种方法的概述，请查看 Azure Batch 论坛中的帖子 [Installing applications and staging data on Batch compute nodes][forum_post]（在 Batch 计算节点上安装应用程序和暂存数据）。此帖子由 Azure Batch 团队的一名成员编写，是一个很好的入门教程，介绍如何在计算节点上以不同方式获取文件（包括应用程序和任务输入数据），以及每种方法的一些特殊注意事项。

[forum_post]: https://social.msdn.microsoft.com/Forums/zh-cn/87b19671-1bdf-427a-972c-2af7e5ba82d9/installing-applications-and-staging-data-on-batch-compute-nodes?forum=azurebatch
[github_file_conventions]: https://github.com/Azure/azure-sdk-for-net/tree/AutoRest/src/Batch/FileConventions
[github_persistoutputs]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/ArticleProjects/PersistOutputs
[github_samples]: https://github.com/Azure/azure-batch-samples
[net_batchclient]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.batchclient.aspx
[net_cloudjob]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.cloudjob.aspx
[net_cloudstorageaccount]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.windowsazure.storage.cloudstorageaccount.aspx
[net_cloudtask]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.cloudtask.aspx
[net_joboutputkind]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.conventions.files.joboutputkind.aspx
[net_joboutputstorage]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.conventions.files.joboutputstorage.aspx
[net_joboutputstorage_saveasync]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.conventions.files.joboutputstorage.saveasync.aspx
[net_msdn]: https://msdn.microsoft.com/zh-cn/library/azure/mt348682.aspx
[net_prepareoutputasync]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.conventions.files.cloudjobextensions.prepareoutputstorageasync.aspx
[net_saveasync]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.conventions.files.taskoutputstorage.saveasync.aspx
[net_savetrackedasync]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.conventions.files.taskoutputstorage.savetrackedasync.aspx
[net_taskoutputkind]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.conventions.files.taskoutputkind.aspx
[net_taskoutputstorage]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.conventions.files.taskoutputstorage.aspx
[nuget_manager]: https://docs.nuget.org/consume/installing-nuget
[nuget_package]: https://www.nuget.org/packages/Microsoft.Azure.Batch.Conventions.Files
[portal]: https://portal.azure.cn
[storage_explorer]: http://storageexplorer.com/

[1]: ./media/batch-task-output/task-output-01.png "门户中“保存的输出文件”和“保存的日志”选择器"
[2]: ./media/batch-task-output/task-output-02.png "Azure 门户预览中的“任务输出”边栏选项卡"

<!---HONumber=Mooncake_0213_2017-->
<!---Update_Description: wording update -->