<properties
	pageTitle="Batch 中的作业准备和清理 | Azure"
	description="采用作业级准备任务最大程度地减少 Azure 批处理 ( Batch )计算节点的数据传输，并在完成作业时执行释放任务来清理节点。"
	services="batch"
	documentationCenter=".net"
	authors="mmacy"
	manager="timlt"
	editor="" />
	
<tags
	ms.service="batch"
	ms.date="04/21/2016"
	wacn.date="06/06/2016"/>

# 在 Azure 批处理( Batch ) 计算节点上运行作业准备和完成任务

Azure 批处理( Batch ) 任务在执行之前通常需要进行某种形式的设置，并且在作业的任务完成之后需要进行某种形式的作业后维护。批处理( Batch ) 以可选的准备和作业释放任务的形式提供这种准备和维护机制。

在任何作业任务运行之前，作业准备任务在计划要运行任务的所有计算节点上运行。作业完成后，作业释放任务将在池中至少运行了一个任务的每个节点上运行。与普通的 Batch 任务一样，你可以指定在运行作业准备或释放任务要调用的命令行。这些特殊任务可以提供许多熟悉的任务功能，例如文件下载、提升权限的执行、自定义环境变量、最大执行持续时间、重试计数和文件保留时间。

在以下部分中，你将了解如何在 [Batch .NET][api_net] API 中利用 [JobPreparationTask][net_job_prep] 类和 [JobReleaseTask][net_job_release] 类来使用这两种特殊类型的任务。

> [AZURE.TIP] 作业准备和释放任务在“共享池”环境中特别有用；在这种环境中，计算节点的池在任务运行之间保留，并在许多不同的作业之间共享。

## 何时使用作业准备和释放任务

许多情况可受益于作业准备和释放任务。下面只是其中的几种情况：

**通用任务数据的传输**

Batch 作业通常需要一组通用的数据作为作业任务的输入。例如，在每日风险分析计算中，市场数据特定于作业，同时也是作业中所有任务通用的数据。这些市场数据（大小通常为若干 GB）应该只下载到每个计算节点一次，以供节点上运行的任意任务使用。在执行作业的其他任务之前使用作业准备任务将数据下载到每个节点。

**作业数据删除**

在共享池环境中，池的计算节点不在作业之间取消保留，为了要节省节点上的磁盘空间或满足组织的安全政策，可能需要在运行之间删除作业数据。使用作业释放任务可以删除作业准备任务下载的数据或者在任务执行期间生成的数据。

**日志保留**

你可能想要保留任务生成的日志文件的副本，或失败应用程序可能生成的崩溃转储文件。在这种情况下，使用作业释放任务可将这些数据压缩并上载到 [Azure 存储][azure_storage]帐户。

## 作业准备任务

在执行作业的任务之前，将在计划运行任务的每个计算节点上执行作业准备任务。默认情况下，Batch 服务将等待作业准备任务完成，然后才在节点上运行计划执行的任务。但你可以将该服务配置为不要等待。如果计算节点重新启动，作业准备任务将在该节点上再次运行，但你也可以禁用此行为。

作业准备任务只会在计划运行任务的节点上运行。例如，这可以防止未分配任务的节点不必要地执行准备任务，当作业的任务数小于池中的节点数时，可能会出现这种情况。此外，这也适用于在任务计数小于可能的并行任务总数的情况下启用[并行任务执行](/documentation/articles/batch-parallel-node-tasks/)，从而留出一些空闲节点的情况。不在空闲节点上运行作业准备任务可以节省数据传输费用。

> [AZURE.NOTE] [JobPreparationTask][net\_job\_prep\_cloudjob] 与 [CloudPool.StartTask][pool_starttask] 的不同之处在于，JobPreparationTask 在每个作业启动时执行，而 StartTask 只在计算节点首次加入池或重新启动时执行。

## 作业释放任务

将作业标记为完成后，作业释放任务将在池中至少运行了一个任务的每个节点上执行。可以通过发出终止请求将作业标记为已完成。然后，Batch 服务会将作业状态设置为“正在终止”，终止与任务关联的任何活动任务或正在运行的任务，并运行作业释放任务。然后，该作业将进入“已完成”状态。

> [AZURE.NOTE] 作业删除操作也会执行作业释放任务。但是，如果已经终止了某个作业，则以后删除该作业时，释放任务不会再次运行。

## Batch .NET API 中的作业准备和释放任务

若要使用作业准备任务，可以创建并配置 [JobPreparationTask][net_job_prep] 对象，然后将它分配到作业的 [CloudJob.JobPreparationTask][net_job_prep_cloudjob] 属性。同样，初始化 [JobReleaseTask][net_job_release] 并将它分配到作业的 [CloudJob.JobReleaseTask][net_job_prep_cloudjob] 属性可以设置作业的释放任务。

在此代码段中，`myBatchClient` 是完全初始化的 [BatchClient][net_batch_client] 实例，`myPool` 是 Batch 帐户中的现有池。
		// Create the CloudJob for CloudPool "myPool"
		CloudJob myJob = myBatchClient.JobOperations.CreateJob("JobPrepReleaseSampleJob",
															   new PoolInformation() { PoolId = "myPool" });
		// Specify the command lines for the job preparation and release tasks
		string jobPrepCmdLine = "cmd /c echo %AZ_BATCH_NODE_ID% > %AZ_BATCH_NODE_SHARED_DIR%\\shared_file.txt";
		string jobReleaseCmdLine = "cmd /c del %AZ_BATCH_NODE_SHARED_DIR%\\shared_file.txt";
		// Assign the job preparation task to the job
		myJob.JobPreparationTask = new JobPreparationTask { CommandLine = jobPrepCmdLine };
		// Assign the job release task to the job
		myJob.JobReleaseTask = new JobPreparationTask { CommandLine = jobReleaseCmdLine };
		await myJob.CommitAsync();

如上所述，终止或删除作业时会执行释放任务。可以通过调用 [JobOperations.TerminateJobAsync][net_job_terminate] 使用 Batch .NET API 终止作业。可以使用 [JobOperations.DeleteJobAsync][net_job_delete] 删除作业。这两项操作通常都是在作业的任务已完成或者达到了你定义的超时时完成。

		// Terminate the job to mark it as Completed; this will initiate the Job Release Task on any node
		// that executed job tasks. Note that the Job Release Task is also executed when a job is deleted,
		// thus you need not call Terminate if you typically delete your jobs upon task completion.
		await myBatchClient.JobOperations.TerminateJobAsync("JobPrepReleaseSampleJob");

## 后续步骤

### GitHub 上的示例项目

查看 GitHub 上的 [JobPrepRelease][job_prep_release_sample] 示例项目，了解作业准备和释放的操作实践。此控制台应用程序将执行以下操作：

1. 创建包含两个“小”节点的池。
2. 创建具有作业准备、释放和标准任务的作业。
3. 运行作业准备任务，该任务首先会将节点 ID 写入节点的“共享”目录中的文本文件内。
4. 在每个节点上运行一个任务，该任务将其任务 ID 写入同一文本文件。
5. 完成所有任务（或达到超时）后，将每个节点的文本文件内容输出到控制台。
6. 完成作业后，运行作业释放任务以从节点中删除该文件。
6. 输出执行作业准备和释放任务的每个节点上的这些任务的退出代码。
7. 暂停执行，以便确认删除作业和/或池。

示例应用程序的输出类似于：

		
		Attempting to create pool: JobPrepReleaseSamplePool
		The pool already existed when we tried to create it
		Checking for existing job JobPrepReleaseSampleJob...
		Job JobPrepReleaseSampleJob not found, creating...
		Submitting tasks and awaiting completion...
		All tasks completed.
		Contents of shared\job_prep_and_release.txt on tvm-3105992504_1-20151015t150030z:
		-------------------------------------------
		tvm-3105992504_1-20151015t150030z tasks:
		  task001
		  task002
		  task006
		  task007
		Contents of shared\job_prep_and_release.txt on tvm-3105992504_2-20151015t150030z:
		-------------------------------------------
		tvm-3105992504_2-20151015t150030z tasks:
		  task003
		  task005
		  task004
		  task008
		Waiting for job JobPrepReleaseSampleJob to reach state Completed
		....
		tvm-3105992504_1-20151015t150030z:
		  Prep task exit code:    0
		  Release task exit code: 0
		tvm-3105992504_2-20151015t150030z:
		  Prep task exit code:    0
		  Release task exit code: 0
		Delete job? [yes] no
		yes
		Delete pool? [yes] no
		no
		Sample complete, hit ENTER to exit...


### 使用 Batch 资源管理器检查作业准备和释放任务

[批处理( Batch ) 资源管理器][batch_explorer_article]（在 GitHub 上的[示例代码存储库][batch_explorer_project]中也能找到）是在 Azure 批处理( Batch ) 中开发解决方案时可以使用的绝佳工具。例如，在运行上述示例应用程序时，你可以使用批处理( Batch ) 资源管理器查看作业及其任务的属性，甚至可以下载作业任务修改的共享文本文件。

以下屏幕截图突出显示了当你在“作业”选项卡中选择 *JobPrepReleaseSampleJob* 作业时，“作业详细信息”窗格中显示的作业准备和释放任务属性。

![批处理( Batch )资源管理器][1]

显示作业准备和释放任务的 Batch 资源管理器屏幕截图

[api_net]: http://msdn.microsoft.com/library/azure/mt348682.aspx
[api_net_listjobs]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.joboperations.listjobs.aspx
[api_rest]: http://msdn.microsoft.com/library/azure/dn820158.aspx
[azure_storage]: /services/storage/
[batch_explorer_article]: http://blogs.technet.com/b/windowshpc/archive/2015/01/20/azure-batch-explorer-sample-walkthrough.aspx
[batch_explorer_project]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/BatchExplorer
[job_prep_release_sample]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/ArticleProjects/JobPrepRelease
[net_batch_client]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.batchclient.aspx
[net_cloudjob]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudjob.aspx
[net_job_prep]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.jobpreparationtask.aspx
[net_job_prep_cloudjob]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudjob.jobpreparationtask.aspx
[net\_job\_prep\_cloudjob]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudjob.jobpreparationtask.aspx
[net_job_delete]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.joboperations.deletejobasync.aspx
[net_job_terminate]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.joboperations.terminatejobasync.aspx
[net_job_release]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.jobreleasetask.aspx
[net_job_release_cloudjob]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudjob.jobreleasetask.aspx
[pool_starttask]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudpool.starttask.aspx

[net_list_certs]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.certificateoperations.listcertificates.aspx
[net_list_compute_nodes]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.pooloperations.listcomputenodes.aspx
[net_list_job_schedules]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.jobscheduleoperations.listjobschedules.aspx
[net_list_jobprep_status]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.joboperations.listjobpreparationandreleasetaskstatus.aspx
[net_list_jobs]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.joboperations.listjobs.aspx
[net_list_nodefiles]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.joboperations.listnodefiles.aspx
[net_list_pools]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.pooloperations.listpools.aspx
[net_list_schedule_jobs]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.jobscheduleoperations.listjobs.aspx
[net_list_task_files]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudtask.listnodefiles.aspx
[net_list_tasks]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.joboperations.listtasks.aspx

[1]: ./media/batch-job-prep-release/batchexplorer-01.png
[2]: ./media/batch-job-prep-release/batchexplorer-02.png

<!---HONumber=Mooncake_0530_2016-->