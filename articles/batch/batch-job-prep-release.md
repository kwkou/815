<properties
	pageTitle="Batch 中的作业准备和清理 | Azure"
	description="使用作业级准备任务最大程度地减少 Azure Batch 计算节点的数据传输，在完成作业时执行释放任务来清理节点。"
	services="batch"
	documentationCenter=".net"
	authors="mmacy"
	manager="timlt"
	editor="" />  


<tags
	ms.service="batch"
	ms.devlang="multiple"
	ms.topic="article"
	ms.tgt_pltfrm="vm-windows"
	ms.workload="big-compute"
	ms.date="09/16/2016"
	ms.author="marsma" 
   	wacn.date="10/26/2016"/>  


# 在 Azure Batch 计算节点上运行作业准备和完成任务

 Azure Batch 作业在执行其任务之前，通常需要经过某种形式的设置，并且需要在其任务完成时进行作业后维护。可能需要将常见的任务输入数据下载到计算节点，或者在作业完成之后，将任务输出数据上载到 Azure 存储。可以使用**作业准备**和**作业释放**任务来执行这些操作。

## 什么是作业准备和作业释放任务？

在运行作业的任务之前，作业准备任务在计划要运行至少一个任务的所有计算节点上运行。作业完成后，作业释放任务将在池中至少运行了一个任务的每个节点上运行。与普通的 Batch 任务一样，可以指定在运行作业准备或释放任务时要调用的命令行。

作业准备和释放任务可以提供许多熟悉的 Batch 任务功能，例如文件下载（[资源文件][net_job_prep_resourcefiles]）、提升权限的执行、自定义环境变量、最大执行持续时间、重试计数和文件保留时间。

以下部分介绍如何使用 [Batch .NET][api_net] 库中的 [JobPreparationTask][net_job_prep] 和 [JobReleaseTask][net_job_release] 类。

> [AZURE.TIP] 作业准备和释放任务在“共享池”环境中特别有用。在这些环境中，计算节点池在任务运行之间保留，并由许多作业使用。

## 何时使用作业准备和释放任务

作业准备和作业释放任务适用于以下情况：

**下载常见任务数据**

Batch 作业通常需要一组通用的数据作为作业任务的输入。例如，在每日风险分析计算中，市场数据特定于作业，同时也是作业中所有任务通用的数据。这些市场数据（大小通常为若干 GB）应该只下载到每个计算节点一次，以供节点上运行的任意任务使用。在执行作业的其他任务之前使用**作业准备任务**将数据下载到每个节点。

**删除作业和任务输出**

在“共享池”环境中，作业之间的池计算节点不会解除，因此可能需要删除运行之间的作业数据。可能需要保留节点上的磁盘空间，或符合组织的安全策略。使用**作业释放任务**可以删除作业准备任务下载的数据或者在任务执行期间生成的数据。

**日志保留**

你可能想要保留任务生成的日志文件的副本，或失败应用程序可能生成的崩溃转储文件。在这种情况下，使用**作业释放任务**可将这些数据压缩并上载到 [Azure 存储][azure_storage]帐户。

>[AZURE.TIP] 保存日志及其他作业和任务输出数据的另一种方法是使用 [Azure Batch 文件约定](/documentation/articles/batch-task-output/)库。

## 作业准备任务

在执行作业的任务之前，Batch 在计划运行任务的每个计算节点上执行作业准备任务。默认情况下，Batch 服务等待作业准备任务完成，然后才在节点上运行计划执行的任务。但你可以将该服务配置为不要等待。如果节点重新启动，作业准备任务将再次运行，但可以禁用此行为。

作业准备任务只会在计划运行任务的节点上运行。例如，这可以防止未分配任务的节点不必要地执行准备任务，当作业的任务数小于池中的节点数时，可能会出现这种情况。此外，这也适用于在任务计数小于可能的并行任务总数的情况下启用[并行任务执行](/documentation/articles/batch-parallel-node-tasks/)，从而留出一些空闲节点的情况。不在空闲节点上运行作业准备任务可以节省数据传输费用。

> [AZURE.NOTE] [JobPreparationTask][net\_job\_prep\_cloudjob] 与 [CloudPool.StartTask][pool_starttask] 的不同之处在于，JobPreparationTask 在每个作业启动时执行，而 StartTask 只在计算节点首次加入池或重新启动时执行。

## 作业释放任务

将作业标记为完成后，作业释放任务将在池中至少运行了一个任务的每个节点上执行。可以通过发出终止请求将作业标记为已完成。然后，Batch 服务会将作业状态设置为 *正在终止* ，终止与任务关联的任何活动任务或正在运行的任务，并运行作业释放任务。然后，该作业将进入 *已完成* 状态。

> [AZURE.NOTE] 作业删除操作也会执行作业释放任务。但是，如果已经终止了某个作业，则以后删除该作业时，释放任务不会再次运行。

## 使用 Batch .NET 执行作业准备和释放任务

若要使用作业准备任务，可将 [JobPreparationTask][net_job_prep] 对象分配到作业的 [CloudJob.JobPreparationTask][net_job_prep_cloudjob] 属性。同样，初始化 [JobReleaseTask][net_job_release] 并将它分配到作业的 [CloudJob.JobReleaseTask][net_job_prep_cloudjob] 属性可以设置作业的释放任务。

在此代码片段中，`myBatchClient` 是 [BatchClient][net_batch_client] 的实例，`myPool` 是 Batch 帐户中的现有池。

csharp

	// Create the CloudJob for CloudPool "myPool"
	CloudJob myJob =
		myBatchClient.JobOperations.CreateJob(
			"JobPrepReleaseSampleJob",
			new PoolInformation() { PoolId = "myPool" });

	// Specify the command lines for the job preparation and release tasks
	string jobPrepCmdLine =
		"cmd /c echo %AZ_BATCH_NODE_ID% > %AZ_BATCH_NODE_SHARED_DIR%\\shared_file.txt";
	string jobReleaseCmdLine =
		"cmd /c del %AZ_BATCH_NODE_SHARED_DIR%\\shared_file.txt";

	// Assign the job preparation task to the job
	myJob.JobPreparationTask =
		new JobPreparationTask { CommandLine = jobPrepCmdLine };

	// Assign the job release task to the job
	myJob.JobReleaseTask =
		new JobPreparationTask { CommandLine = jobReleaseCmdLine };

	await myJob.CommitAsync();

如前所述，终止或删除作业时会执行释放任务。使用 [JobOperations.TerminateJobAsync][net_job_terminate] 终止作业。使用 [JobOperations.DeleteJobAsync][net_job_delete] 删除作业。通常在作业的任务完成时或者达到定义的超时时终止或删除操作。

csharp

	// Terminate the job to mark it as Completed; this will initiate the
	// Job Release Task on any node that executed job tasks. Note that the
	// Job Release Task is also executed when a job is deleted, thus you
	// need not call Terminate if you typically delete jobs after task completion.
	await myBatchClient.JobOperations.TerminateJobAsy("JobPrepReleaseSampleJob");

## GitHub 上的代码示例

若要了解作业准备和释放的操作实践，请查看 GitHub 上的 [JobPrepRelease][job_prep_release_sample] 示例项目。此控制台应用程序将执行以下操作：

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
	Created pool JobPrepReleaseSamplePool with 2 small nodes
	Checking for existing job JobPrepReleaseSampleJob...
	Job JobPrepReleaseSampleJob not found, creating...
	Submitting tasks and awaiting completion...
	All tasks completed.
	
	Contents of shared\job_prep_and_release.txt on tvm-2434664350_1-20160623t173951z:
	-------------------------------------------
	tvm-2434664350_1-20160623t173951z tasks:
	  task001
	  task004
	  task005
	  task006
	
	Contents of shared\job_prep_and_release.txt on tvm-2434664350_2-20160623t173951z:
	-------------------------------------------
	tvm-2434664350_2-20160623t173951z tasks:
	  task008
	  task002
	  task003
	  task007
	
	Waiting for job JobPrepReleaseSampleJob to reach state Completed
	...
	
	tvm-2434664350_1-20160623t173951z:
	  Prep task exit code:    0
	  Release task exit code: 0
	
	tvm-2434664350_2-20160623t173951z:
	  Prep task exit code:    0
	  Release task exit code: 0
	
	Delete job? [yes] no
	yes
	Delete pool? [yes] no
	yes
	
	Sample complete, hit ENTER to exit...

>[AZURE.NOTE] 由于新池中各个节点的创建和启动时间并不一样（某些节点比其他节点更早做好任务准备），你可能看到不同的输出。具体而言，因为任务快速完成，池的某个节点可能执行作业的所有任务。如果发生这种情况，你会发现未执行任何任务的节点没有作业准备和作业释放任务存在。

### 在 Azure 门户预览中检查作业准备和释放任务

在运行示例应用程序时，可以使用 [Azure 门户预览][portal]查看作业及其任务的属性，甚至可以下载作业任务修改的共享文本文件。

以下屏幕截图显示了在运行示例应用程序之后，Azure 门户预览中出现的**准备任务边栏选项卡**。在任务完成之后（但在删除作业与池之前），导航到 *JobPrepReleaseSampleJob* 属性，然后单击“准备任务”或“释放任务”以查看其属性。

![Azure 门户预览中的作业准备属性][1]  


## 后续步骤

### 应用程序包

除了作业准备任务外，还可以使用 Batch 的[应用程序包](/documentation/articles/batch-application-packages/)功能来为计算节点做好任务执行准备。此功能特别适合用于部署不需要运行安装程序的应用程序、包含许多（100 个以上）文件的应用程序，或需要严格版本控制的应用程序。

### 安装应用程序和暂存数据

以下 MSDN 论坛文章提供了有关准备节点以运行任务的各种方法的概述：

[Installing applications and staging data on Batch compute nodes][forum_post]（在 Batch 计算节点上安装应用程序和暂存数据）

此文章的作者是一位 Azure Batch 团队成员，其中介绍了将应用程序和数据部署到计算节点时可以使用的的多种方法。

[api_net]: http://msdn.microsoft.com/zh-cn/library/azure/mt348682.aspx
[api_net_listjobs]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.joboperations.listjobs.aspx
[api_rest]: http://msdn.microsoft.com/zh-cn/library/azure/dn820158.aspx
[azure_storage]: /home/features/storage/
[portal]: https://portal.azure.cn
[job_prep_release_sample]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/ArticleProjects/JobPrepRelease
[forum_post]: https://social.msdn.microsoft.com/Forums/en-US/87b19671-1bdf-427a-972c-2af7e5ba82d9/installing-applications-and-staging-data-on-batch-compute-nodes?forum=azurebatch
[net_batch_client]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.batchclient.aspx
[net_cloudjob]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.cloudjob.aspx
[net_job_prep]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.jobpreparationtask.aspx
[net_job_prep_cloudjob]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.cloudjob.jobpreparationtask.aspx
[net\_job\_prep\_cloudjob]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.cloudjob.jobpreparationtask.aspx
[net_job_prep_resourcefiles]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.jobpreparationtask.resourcefiles.aspx
[net_job_delete]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.joboperations.deletejobasync.aspx
[net_job_terminate]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.joboperations.terminatejobasync.aspx
[net_job_release]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.jobreleasetask.aspx
[net_job_release_cloudjob]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.cloudjob.jobreleasetask.aspx
[pool_starttask]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.cloudpool.starttask.aspx

[net_list_certs]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.certificateoperations.listcertificates.aspx
[net_list_compute_nodes]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.pooloperations.listcomputenodes.aspx
[net_list_job_schedules]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.jobscheduleoperations.listjobschedules.aspx
[net_list_jobprep_status]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.joboperations.listjobpreparationandreleasetaskstatus.aspx
[net_list_jobs]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.joboperations.listjobs.aspx
[net_list_nodefiles]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.joboperations.listnodefiles.aspx
[net_list_pools]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.pooloperations.listpools.aspx
[net_list_schedule_jobs]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.jobscheduleoperations.listjobs.aspx
[net_list_task_files]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.cloudtask.listnodefiles.aspx
[net_list_tasks]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.joboperations.listtasks.aspx

[1]: ./media/batch-job-prep-release/portal-jobprep-01.png

<!---HONumber=Mooncake_1017_2016-->
