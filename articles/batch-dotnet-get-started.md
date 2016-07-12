<properties
	pageTitle="教程 - Azure  批处理( Batch ) .NET 库入门 | Azure"
	description="了解 Azure  批处理( Batch )的基本概念，以及如何使用一个简单方案开发 批处理( Batch )服务"
	services="batch"
	documentationCenter=".net"
	authors="mmacy"
	manager="timlt"
	editor=""/>

<tags
	ms.service="batch"
	ms.date="05/12/2016"
	wacn.date="06/06/2016"/>

# 适用于 .NET 的 Azure Batch 库入门  

在我们分步讨论 C# 示例应用程序时，了解本文中的 [Azure Batch][azure_batch] 和 [Batch .NET][net_api] 库的基础知识。我们将探讨该示例应用程序如何利用 Batch 服务来处理云中的并行工作负荷，以及如何与 [Azure 存储空间](/documentation/articles/storage-introduction/)交互来暂存和检索文件。你会了解常见的 Batch 应用程序工作流技术。你还会大致了解 Batch 的主要组件，例如作业、任务、池和计算节点。

![Batch 解决方案工作流（基础）][11]

## 先决条件

本文假定你有 C# 和 Visual Studio 的使用知识。本文还假定，你能够满足下面为 Azure 和 Batch 及存储服务指定的帐户创建要求。

### 帐户

- **Azure 帐户** -- 如果你没有 Azure 订阅，可以 [创建一个 试用 Azure 帐户][azure\_free\_account]。
- **Batch 帐户**
- **存储帐户**：请参阅 [About Azure storage accounts](/documentation/articles/storage-create-storage-account/)（关于 Azure 存储帐户）中的 [Create a storage account（创建存储帐户）](/documentation/articles/storage-create-storage-account/#create-a-storage-account)。

> [AZURE.IMPORTANT] Batch 目前*仅*支持**常规用途**存储帐户类型，如 [About Azure storage accounts（关于 Azure 存储帐户）](/documentation/articles/storage-create-storage-account/)的 [Create a storage account（创建存储帐户）](/documentation/articles/storage-create-storage-account/#create-a-storage-account)中步骤 5 所述。

### Visual Studio

必须拥有 **Visual Studio 2013** 或 **Visual Studio 2015** 才能构建示例项目。可以在 [Visual Studio 2015 产品概述][visual_studio]中找到免费试用版的 Visual Studio。

### *DotNetTutorial* 代码示例

[DotNetTutorial][github_dotnettutorial] 示例是 GitHub 上的 [azure-batch-samples][github_samples] 存储库中提供的众多代码示例之一。单击存储库主页上的“下载 ZIP”按钮，或单击 [azure-batch-samples-master.zip][github_samples_zip] 直接下载链接即可下载该示例。将 ZIP 文件的内容解压缩后，可在以下文件夹中找到该解决方案：

`\azure-batch-samples\CSharp\ArticleProjects\DotNetTutorial`

### Azure Batch 资源管理器（可选）

[Azure Batch 资源管理器][github_batchexplorer]是 GitHub 上的 [azure-batch-samples][github_samples] 存储库随附的免费实用工具。尽管完成本教程不要求使用 Batch 资源管理器，但我们强烈建议使用它来调试和管理 Batch 帐户中的实体。你可以在 [Azure Batch Explorer sample walkthrough（Azure Batch 资源管理器示例演练）][batch_explorer_blog]博客文章中了解有关旧版 Batch 资源管理器的信息。

## DotNetTutorial 示例项目概述

*DotNetTutorial* 代码示例是由以下两个项目组成的 Visual Studio 2013 解决方案：**DotNetTutorial** 和 **TaskApplication**。

- **DotNetTutorial** 是与 Batch 和存储服务交互，以在计算节点（虚拟机）上执行并行工作负荷的客户端应用程序。DotNetTutorial 在本地工作站上运行。

- **TaskApplication** 是在 Azure 中的计算节点上运行以执行实际工作的程序。在示例中，`TaskApplication.exe` 分析了从 Azure 存储空间下载的文件（输入文件）中的文本。然后，它会生成一个文本文件（输出文件），其中包含出现在输入文件中的头三个单词的列表。在创建输出文件以后，TaskApplication 会将文件上载到 Azure 存储空间。这样该文件就可供客户端应用程序下载。TaskApplication 在 Batch 服务中的多个计算节点上并行运行。

下图演示了客户端应用程序 *DotNetTutorial* 执行的主要操作，以及任务执行的应用程序 *TaskApplication*。此基本工作流是通过 Batch 创建的许多计算解决方案中常见的工作流。尽管它并未演示 Batch 服务提供的每项功能，但几乎每个 Batch 方案都包含类似的过程。

![Batch 示例工作流][8]

[**步骤 1.**](#step-1-create-storage-containers) 在 Azure Blob 存储中创建**容器**。<br/>
[**步骤 2.**](#step-2-upload-task-application-and-data-files) 将任务应用程序文件和输入文件上载到容器。<br/>
[**步骤 3.**](#step-3-create-batch-pool) 创建 Batch **池**。<br/>
  &nbsp;&nbsp;&nbsp;&nbsp;**3a.** 池 **StartTask** 在节点加入池时将任务二进制文件 (TaskApplication) 下载到节点。<br/>
[**步骤 4.**](#step-4-create-batch-job) 创建 Batch **作业**。<br/>
[**步骤 5.**](#step-5-add-tasks-to-job) 将**任务**添加到作业。
  <br/>&nbsp;&nbsp;&nbsp;&nbsp;**5a.** 任务计划在节点上执行。<br/>
	&nbsp;&nbsp;&nbsp;&nbsp;**5b.** 每项任务从 Azure 存储空间下载其输入数据，然后开始执行。<br/>
[**步骤 6.**](#step-6-monitor-tasks) 监视任务。<br/>
  &nbsp;&nbsp;&nbsp;&nbsp;**6a.** 当任务完成时，会将其输出数据上载到 Azure 存储空间。<br/>
[**步骤 7.**](#step-7-download-task-output) 从存储空间下载任务输出。

如前所述，并非每个 Batch 解决方案都会执行这些具体步骤，此类方案可能包含更多步骤，但 DotNetTutorial 示例应用程序将演示 Batch 方案中的常见过程。

## 构建 DotNetTutorial 示例项目

你必须先在 DotNetTutorial 项目的 `Program.cs` 文件中指定 Batch 和存储帐户凭据才能成功运行该示例。请在 Visual Studio 中双击 `DotNetTutorial.sln` 解决方案文件以打开该解决方案（如果你尚未这样做）。也可以在 Visual Studio 中使用“文件”>“打开”>“项目/解决方案”菜单打开它。

在 DotNetTutorial 项目中打开 `Program.cs`。然后，添加在文件顶部附近指定的凭据：


		// Update the Batch and Storage account credential strings below with the values
		// unique to your accounts. These are used when constructing connection strings
		// for the Batch and Storage client objects.
		// Batch account credentials
		private const string BatchAccountName = "";
		private const string BatchAccountKey  = "";
		private const string BatchAccountUrl  = "";
		// Storage account credentials
		private const string StorageAccountName = "";
		private const string StorageAccountKey  = "";
		

> [AZURE.IMPORTANT] 如上所述，目前必须为 Azure 存储空间中的**常规用途**存储帐户指定凭据。Batch 应用程序将使用**常规用途**存储帐户中的 Blob 存储。请不要为通过选择 “Blob 存储” 帐户类型创建的存储帐户指定凭据。

可以在 [Azure 门户][azure_portal]中每个服务的帐户边栏选项卡中找到 Batch 和存储帐户凭据：

![门户中的 Batch 凭据][9]
![门户中的存储空间凭据][10]<br/>

使用凭据更新项目后，在“解决方案资源管理器”中右键单击该解决方案，然后单击“构建解决方案”。出现提示时，请确认还原任何 NuGet 包。

> [AZURE.TIP] 如果未自动还原 NuGet 包，或者你看到了有关包还原失败的错误，请确保已安装 [NuGet 包管理器][nuget_packagemgr]，然后启用遗失包的下载。若要启用包下载，请参阅 [Enabling Package Restore During Build（在构建期间启用包还原）][nuget_restore]。

在以下部分中，我们将示例应用程序细分为用于处理 Batch 服务中工作负荷的多个步骤，并详细讨论这些步骤。建议你在学习本文的余下部分时参考 Visual Studio 中打开的解决方案，因为我们并不会讨论示例中的每一行代码。

导航到 *DotNetTutorial* 项目的 `Program.cs` 文件中 `MainAsync` 方法的顶部，开始执行步骤 1。以下每个步骤大致遵循 `MainAsync` 中方法调用的进度。

## 步骤 1：创建存储容器

![在 Azure 存储空间中创建容器][1] <br/>

Batch 包含的内置支持支持与 Azure 存储空间交互。存储帐户中的容器将为 Batch 帐户中运行的任务提供所需的文件。这些容器还提供存储任务生成的输出数据所需的位置。DotNetTutorial 客户端应用程序首先在 [Azure Blob 存储](/documentation/articles/storage-introduction/)中创建三个容器：

- **应用程序**：此容器用于存储任务所要运行的应用程序及其依赖项，例如 DLL。
- **输入**：任务将从*输入*容器下载所要处理的数据文件。
- **输出**：当任务完成输入文件的处理时，会将其结果上载到“输出”容器。

为了与存储帐户交互并创建容器，我们使用了[适用于 .NET 的 Azure 存储空间客户端库][net_api_storage]。我们将创建包含 [CloudStorageAccount][net_cloudstorageaccount] 的帐户引用，并从中创建 [CloudBlobClient][net_cloudblobclient]：

		
		// Construct the Storage account connection string
		string storageConnectionString = String.Format(
		    "DefaultEndpointsProtocol=https;AccountName={0};AccountKey={1}",
		    StorageAccountName,
		    StorageAccountKey);
		// Retrieve the storage account
		CloudStorageAccount storageAccount = CloudStorageAccount.Parse(storageConnectionString);
		// Create the blob client, for use in obtaining references to blob storage containers
		CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();


我们将在整个应用程序中使用 `blobClient` 引用，并将它作为参数传递给多个方法。紧接在上述代码的代码块中提供了示例，我们在其中调用 `CreateContainerIfNotExistAsync` 以实际创建容器。


		// Use the blob client to create the containers in Azure Storage if they don't
		// yet exist
		const string appContainerName    = "application";
		const string inputContainerName  = "input";
		const string outputContainerName = "output";
		await CreateContainerIfNotExistAsync(blobClient, appContainerName);
		await CreateContainerIfNotExistAsync(blobClient, inputContainerName);
		await CreateContainerIfNotExistAsync(blobClient, outputContainerName);
		private static async Task CreateContainerIfNotExistAsync(
		    CloudBlobClient blobClient,
		    string containerName)
		{
				CloudBlobContainer container = blobClient.GetContainerReference(containerName);
				if (await container.CreateIfNotExistsAsync())
				{
						Console.WriteLine("Container [{0}] created.", containerName);
				}
				else
				{
						Console.WriteLine("Container [{0}] exists, skipping creation.",
		                    containerName);
				}
		}


创建容器之后，应用程序现在即可上载任务使用的文件。

> [AZURE.TIP] [How to use Blob Storage from .NET](/documentation/articles/storage-dotnet-how-to-use-blobs/) 对如何使用 Azure 存储容器和 blob 进行了很好的概述。当你开始使用 Batch 时，它应该位于阅读列表顶部附近。

## 步骤 2：上载任务应用程序和数据文件

![将任务应用程序和输入（数据）文件上载到容器][2] <br/>

在文件上载操作中，*DotNetTutorial* 先定义**应用程序**和**输入**文件在本地计算机上的路径的集合，然后将这些文件上载到上一步骤创建的容器。

		
		// Paths to the executable and its dependencies that will be executed by the tasks
		List<string> applicationFilePaths = new List<string>
		{
		    // The DotNetTutorial project includes a project reference to TaskApplication,
		    // allowing us to determine the path of the task application binary dynamically
		    typeof(TaskApplication.Program).Assembly.Location,
		    "Microsoft.WindowsAzure.Storage.dll"
		};
		// The collection of data files that are to be processed by the tasks
		List<string> inputFilePaths = new List<string>
		{
		    @"..\..\taskdata1.txt",
		    @"..\..\taskdata2.txt",
		    @"..\..\taskdata3.txt"
		};
		// Upload the application and its dependencies to Azure Storage. This is the
		// application that will process the data files, and will be executed by each
		// of the tasks on the compute nodes.
		List<ResourceFile> applicationFiles = await UploadFilesToContainerAsync(
		    blobClient,
		    appContainerName,
		    applicationFilePaths);
		// Upload the data files. This is the data that will be processed by each of
		// the tasks that are executed on the compute nodes within the pool.
		List<ResourceFile> inputFiles = await UploadFilesToContainerAsync(
		    blobClient,
		    inputContainerName,
		    inputFilePaths);


`Program.cs` 中有两个方法涉及到上载过程：

- `UploadFilesToContainerAsync`：此方法返回 [ResourceFile][net_resourcefile] 对象的集合（下面将会介绍），并在内部调用 `UploadFileToContainerAsync` 以上载在 *filePaths* 参数中传递的每个文件。
- `UploadFileToContainerAsync`：这是实际执行文件上载并创建 [ResourceFile][net_resourcefile] 对象的方法。上载文件后，它将获取该文件的共享访问签名 (SAS) 并返回代表它的 ResourceFile 对象。下面也会介绍共享访问签名。

		
		private static async Task<ResourceFile> UploadFileToContainerAsync(
		    CloudBlobClient blobClient,
		    string containerName,
		    string filePath)
		{
				Console.WriteLine(
		            "Uploading file {0} to container [{1}]...", filePath, containerName);
				string blobName = Path.GetFileName(filePath);
				CloudBlobContainer container = blobClient.GetContainerReference(containerName);
				CloudBlockBlob blobData = container.GetBlockBlobReference(blobName);
				await blobData.UploadFromFileAsync(filePath, FileMode.Open);
				// Set the expiry time and permissions for the blob shared access signature.
		        // In this case, no start time is specified, so the shared access signature
		        // becomes valid immediately
				SharedAccessBlobPolicy sasConstraints = new SharedAccessBlobPolicy
				{
						SharedAccessExpiryTime = DateTime.UtcNow.AddHours(2),
						Permissions = SharedAccessBlobPermissions.Read
				};
				// Construct the SAS URL for blob
				string sasBlobToken = blobData.GetSharedAccessSignature(sasConstraints);
				string blobSasUri = String.Format("{0}{1}", blobData.Uri, sasBlobToken);
				return new ResourceFile(blobSasUri, blobName);
		}


### ResourceFiles

[ResourceFile][net_resourcefile] 提供 Batch 中的任务，以及 Azure 存储空间中将在任务运行之前下载到计算节点的文件的 URL。[ResourceFile.BlobSource][net_resourcefile_blobsource] 属性指定存在于 Azure 存储空间的文件的完整 URL。该 URL 还可以包含用于对文件进行安全访问的共享访问签名 (SAS)。Batch .NET 中的大多数任务类型都包含ResourceFiles属性，这些类型包括：

- [CloudTask][net_task]
- [StartTask][net_pool_starttask]
- [JobPreparationTask][net_jobpreptask]
- [JobReleaseTask][net_jobreltask]

DotNetTutorial 示例应用程序不使用 JobPreparationTask 或 JobReleaseTask 任务类型，但你可以通过[在 Azure Batch 计算节点上运行作业准备和完成任务](/documentation/articles/batch-job-prep-release/)来详细了解这些任务类型。

### 共享访问签名 (SAS)

共享访问签名是一些字符串，包含为 URL 的一部分时，它们可以提供对 Azure 存储空间中容器和 Blob 的安全访问。DotNetTutorial 应用程序使用 Blob 和容器共享访问签名 URL，并演示如何从存储空间服务获取这些共享访问签名字符串。

- **Blob 共享访问签名**：DotNetTutorial 中池的 StartTask 在从存储空间下载应用程序二进制文件和输入数据文件时使用 Blob 共享访问签名（请参阅下面的步骤 3）。DotNetTutorial 的 `Program.cs` 中的 `UploadFileToContainerAsync` 方法包含的代码可用于获取每个 blob 的共享访问签名。它是通过调用 [CloudBlob.GetSharedAccessSignature][net_sas_blob] 来完成此操作的。

- **容器共享访问签名**：每个任务在计算节点上完成其工作后，会将其输出文件上载到 Azure 存储空间中的输出容器。为此，TaskApplication 使用容器共享访问签名，在上载文件时，该共享访问签名提供对路径中包含的容器的写访问。获取容器共享访问签名的操作方式类似于获取 blob 共享访问签名。在 DotNetTutorial 中，你会发现 `GetContainerSasUrl` 帮助器方法调用 [CloudBlobContainer.GetSharedAccessSignature][net_sas_container] 来执行该操作。你可以在下面的“步骤 6：监视任务”中详细了解 TaskApplication 如何使用容器共享访问签名。

> [AZURE.TIP] 请查看有关共享访问签名的两篇系列教程的[第 1 部分：了解共享访问签名 (SAS) 模型](/documentation/articles/storage-dotnet-shared-access-signature-part-1/)和[第 2 部分：创建共享访问签名 (SAS) 并将其用于 Blob 服务](/documentation/articles/storage-dotnet-shared-access-signature-part-2/)，以详细了解如何提供对存储帐户中数据的安全访问。

## 步骤 3：创建 Batch 池

![创建 Batch 池][3] <br/>

将应用程序和数据文件上载到存储帐户之后，DotNetTutorial 将使用 Batch .NET 库开始与 Batch 服务交互。为此，需要先创建一个 [BatchClient][net_batchclient]：

		
		BatchSharedKeyCredentials cred = new BatchSharedKeyCredentials(
		    BatchAccountUrl,
		    BatchAccountName,
		    BatchAccountKey);
		using (BatchClient batchClient = BatchClient.Open(cred))
		{
			...


然后，调用 `CreatePoolAsync` 以在 Batch 帐户中创建计算节点池。`CreatePoolAsync` 使用 [BatchClient.PoolOperations.CreatePool][net_pool_create] 方法在 Batch 服务中实际创建该池。

		
		private static async Task CreatePoolAsync(
		    BatchClient batchClient,
		    string poolId,
		    IList<ResourceFile> resourceFiles)
		{
		    Console.WriteLine("Creating pool [{0}]...", poolId);
		    // Create the unbound pool. Until we call CloudPool.Commit() or CommitAsync(),
		    // no pool is actually created in the Batch service. This CloudPool instance is
		    // therefore considered "unbound," and we can modify its properties.
		    CloudPool pool = batchClient.PoolOperations.CreatePool(
					poolId: poolId,
					targetDedicated: 3,           // 3 compute nodes
					virtualMachineSize: "small",  // single-core, 1.75 GB memory, 224 GB disk
					cloudServiceConfiguration:
					    new CloudServiceConfiguration(osFamily: "4")); // Win Server 2012 R2
		    // Create and assign the StartTask that will be executed when compute nodes join
		    // the pool. In this case, we copy the StartTask's resource files (that will be
		    // automatically downloaded to the node by the StartTask) into the shared
		    // directory that all tasks will have access to.
		    pool.StartTask = new StartTask
		    {
		        // Specify a command line for the StartTask that copies the task application
		        // files to the node's shared directory. Every compute node in a Batch pool
		        // is configured with several pre-defined environment variables that you can
		        // reference by using commands or applications run by tasks.
		        // Since a successful execution of robocopy can return a non-zero exit code
		        // (e.g. 1 when one or more files were successfully copied) we need to
		        // manually exit with a 0 for Batch to recognize StartTask execution success.
		        CommandLine = "cmd /c (robocopy %AZ_BATCH_TASK_WORKING_DIR% %AZ_BATCH_NODE_SHARED_DIR%) ^& IF %ERRORLEVEL% LEQ 1 exit 0",
		        ResourceFiles = resourceFiles,
		        WaitForSuccess = true
		    };
		    await pool.CommitAsync();
		}


使用 [CreatePool][net_pool_create] 创建池时，需指定一些参数，例如计算节点数目、[节点大小](/documentation/articles/cloud-services-sizes-specs/)以及节点的操作系统。在 DotNetTutorial 中，我们使用 [CloudServiceConfiguration][net_cloudserviceconfiguration] 从[云服务](/documentation/articles/cloud-services-guestos-update-matrix/)指定 Windows Server 2012 R2。但是，如果指定 [VirtualMachineConfiguration][net_virtualmachineconfiguration]，则可以从应用商店映像（包括 Windows 和 Linux 映像）创建节点池 -- 有关详细信息，请参阅 [Introducing Linux support on Azure Batch（Azure Batch 上的 Linux 支持简介）][blog_linux]。

> [AZURE.IMPORTANT] 你需要支付 Batch 中计算资源的费用。若要将费用降到最低，可以在运行示例之前，将 `targetDedicated` 降为 1。

你也可以连同这些实体节点属性一起指定池的 [StartTask][net_pool_starttask]。StartTask 将在每个节点加入池以及每次重新启动节点时在该节点上运行。StartTask 特别适合用于在任务执行之前在计算节点上安装应用程序。例如，如果任务使用 Python 脚本处理数据，则你可以使用 StartTask 在计算节点上安装 Python。

在此示例应用程序中，StartTask 将它从存储空间下载的文件（使用 [StartTask][net_starttask].[ResourceFiles][net_starttask_resourcefiles] 属性指定），从 StartTask 工作目录复制到在节点上运行的所有任务可以访问的共享目录。本质上，这会在节点加入池时，将 `TaskApplication.exe` 及其依赖项复制到每个节点上的共享目录，因此该节点上运行的任何任务都可以访问它。

> [AZURE.TIP] Azure Batch 的**应用程序包**功能提供另一种方法用于将应用程序转移到池中的计算节点。

此外，在上述代码段中，值得注意的问题是，StartTask 的 *CommandLine* 属性中使用了两个环境变量：`%AZ_BATCH_TASK_WORKING_DIR%` 和 `%AZ_BATCH_NODE_SHARED_DIR%`。将自动为 Batch 池中的每个计算节点配置多个特定于 Batch 的环境变量。由任务执行的任何进程都可以访问这些环境变量。

> [AZURE.TIP] 若要深入了解 Batch 池中计算节点上可用的环境变量，以及有关任务工作目录的信息，请参阅 [Azure Batch 功能概述](/documentation/articles/batch-api-basics/)中的“任务的环境设置”及“文件和目录”部分。

## 步骤 4：创建 Batch 作业

![创建 Batch 作业][4]

Batch 作业实质上是与计算节点池关联的任务的集合。它不仅可用来组织和跟踪相关工作负荷中的任务，也可以实施特定的约束，例如作业（并扩展到其任务）的最大运行时，以及 Batch 帐户中其他作业的相关作业优先级。不过在本示例中，该作业仅与步骤 3 中创建的池关联。未配置任何其他属性。

所有 Batch 作业都与特定的池关联。此关联指示将要在其上执行作业任务的节点。你可以通过 [CloudJob.PoolInformation][net_job_poolinfo] 属性进行这方面的指定，如下面的代码段所示。


		private static async Task CreateJobAsync(
		    BatchClient batchClient,
		    string jobId,
		    string poolId)
		{
		    Console.WriteLine("Creating job [{0}]...", jobId);
		    CloudJob job = batchClient.JobOperations.CreateJob();
		    job.Id = jobId;
		    job.PoolInformation = new PoolInformation { PoolId = poolId };
		    await job.CommitAsync();
		}


创建作业后，可以添加任务来执行工作。

## 步骤 5：将任务添加到作业

![将任务添加到作业][5]<br/>
(1) 将任务添加到作业；(2) 将任务计划为在节点上运行；(3) 任务下载要处理的数据文件

若要实际执行工作，必须将任务添加到作业。每个 [CloudTask][net_task] 都是使用命令行属性以及任务在其命令行自动执行前下载到节点的 [ResourceFiles][net_task_resourcefiles]（如同池的 StartTask）进行配置的。在 DotNetTutorial 示例项目中，每个任务只处理一个文件。因此，其 ResourceFiles 集合包含单个元素。


		private static async Task<List<CloudTask>> AddTasksAsync(
		    BatchClient batchClient,
		    string jobId,
		    List<ResourceFile> inputFiles,
		    string outputContainerSasUrl)
		{
		    Console.WriteLine("Adding {0} tasks to job [{1}]...", inputFiles.Count, jobId);
		    // Create a collection to hold the tasks that we'll be adding to the job
		    List<CloudTask> tasks = new List<CloudTask>();
		    // Create each of the tasks. Because we copied the task application to the
		    // node's shared directory with the pool's StartTask, we can access it via
		    // the shared directory on the node that the task runs on.
		    foreach (ResourceFile inputFile in inputFiles)
		    {
		        string taskId = "topNtask" + inputFiles.IndexOf(inputFile);
		        string taskCommandLine = String.Format(
		            "cmd /c %AZ_BATCH_NODE_SHARED_DIR%\\TaskApplication.exe {0} 3 "{1}"",
		            inputFile.FilePath,
		            outputContainerSasUrl);
		        CloudTask task = new CloudTask(taskId, taskCommandLine);
		        task.ResourceFiles = new List<ResourceFile> { inputFile };
		        tasks.Add(task);
		    }
		    // Add the tasks as a collection, as opposed to issuing a separate AddTask call
		    // for each. Bulk task submission helps to ensure efficient underlying API calls
		    // to the Batch service.
		    await batchClient.JobOperations.AddTaskAsync(jobId, tasks);
		    return tasks;
		}


> [AZURE.IMPORTANT] 在访问环境变量（例如 `%AZ_BATCH_NODE_SHARED_DIR%`）或执行节点的 `PATH` 中找不到的应用程序时，任务命令行必须带有 `cmd /c` 前缀。这样才可以显式执行命令解释器，并指示其在执行命令后终止操作。如果任务在节点的 `PATH` 中执行应用程序（例如 *robocopy.exe* 或 *powershell.exe*），而且未使用任何环境变量，则就不必要满足此要求。

在上述代码段中的 `foreach` 循环内，可以看到已构造任务的命令行，因此有三个命令行参数已传递到 *TaskApplication.exe*：

1. **第一个参数**是要处理的文件的路径。这是节点上现有文件的本地路径。首次创建上面 `UploadFileToContainerAsync` 中的 ResourceFile 对象时，会将文件名用于此属性（作为 ResourceFile 构造函数的参数）。这意味着可以在 *TaskApplication.exe* 所在的目录中找到此文件。

2. **第二个参数**指定应将前 *N* 个单词写入输出文件。在示例中，此参数已经过硬编码，因此会将前 3 个单词写入输出文件。

3. **第三个参数**是共享访问签名 (SAS)，提供对 Azure 存储空间中**输出**容器的写访问。在将输出文件上载到 Azure 存储空间时，*TaskApplication.exe* 使用此共享访问签名 URL。你可以在 TaskApplication 项目的 `Program.cs` 文件的 `UploadFileToContainer` 方法中找到此方面的代码：

		
		// NOTE: From project TaskApplication Program.cs
		private static void UploadFileToContainer(string filePath, string containerSas)
		{
				string blobName = Path.GetFileName(filePath);
				// Obtain a reference to the container using the SAS URI.
				CloudBlobContainer container = new CloudBlobContainer(new Uri(containerSas));
				// Upload the file (as a new blob) to the container
				try
				{
						CloudBlockBlob blob = container.GetBlockBlobReference(blobName);
						blob.UploadFromFile(filePath, FileMode.Open);
						Console.WriteLine("Write operation succeeded for SAS URL " + containerSas);
						Console.WriteLine();
				}
				catch (StorageException e)
				{
						Console.WriteLine("Write operation failed for SAS URL " + containerSas);
						Console.WriteLine("Additional error information: " + e.Message);
						Console.WriteLine();
						// Indicate that a failure has occurred so that when the Batch service
		                // sets the CloudTask.ExecutionInformation.ExitCode for the task that
		                // executed this application, it properly indicates that there was a
		                // problem with the task.
						Environment.ExitCode = -1;
				}
		}


## 步骤 6：监视任务

![监视任务][6]<br/>客户端应用程序将会：(1) 监视任务的完成和成功状态；(2) 监视将结果数据上载到 Azure 存储空间的任务

任务在添加到作业后，将自动排入队列并计划在与作业关联的池中的计算节点上执行。根据你指定的设置，Batch 将为你处理所有任务排队、计划、重试和其他任务管理工作。监视任务的执行有许多方法。DotNetTutorial 显示了一个简单的示例，该示例只报告完成状态以及任务的失败或成功状态。

DotNetTutorial 的 `Program.cs` 中的 `MonitorTasks` 方法内有三个 Batch .NET 概念值得讨论。下面按出现顺序列出了这些概念：

1. **ODATADetailLevel**：必须在列出操作（例如获取作业的任务列表）中指定 [ODATADetailLevel][net_odatadetaillevel]，以确保 Batch 应用程序的性能。如果你打算在 Batch 应用程序中进行任何类型的状态监视，请将[有效地查询 Azure Batch 服务](/documentation/articles/batch-efficient-list-queries/)加入你的阅读列表。

2. **TaskStateMonitor**：[TaskStateMonitor][net_taskstatemonitor] 为 Batch .NET 应用程提供用于监视任务状态的帮助器实用程序。在 `MonitorTasks` 中，DotNetTutorial 将等待所有任务在时限内达到 [TaskState.Completed][net_taskstate]，然后终止作业。

3. **TerminateJobAsync**：通过 [JobOperations.TerminateJobAsync][net_joboperations_terminatejob] 终止作业（或阻止 JobOperations.TerminateJob）会将该作业标记为已完成。如果你的 Batch 解决方案使用 [JobReleaseTask][net_jobreltask]，则这样做很重要。这是一种特殊类型的任务，在[作业准备与完成任务](/documentation/articles/batch-job-prep-release/)中有说明。

DotNetTutorial 的 `Program.cs` 中的 `MonitorTasks` 方法如下所示：

		
		private static async Task<bool> MonitorTasks(
		    BatchClient batchClient,
		    string jobId,
		    TimeSpan timeout)
		{
		    bool allTasksSuccessful = true;
		    const string successMessage = "All tasks reached state Completed.";
		    const string failureMessage = "One or more tasks failed to reach the Completed state within the timeout period.";
		    // Obtain the collection of tasks currently managed by the job. Note that we use
		    // a detail level to specify that only the "id" property of each task should be
		    // populated. Using a detail level for all list operations helps to lower
		    // response time from the Batch service.
		    ODATADetailLevel detail = new ODATADetailLevel(selectClause: "id");
		    List<CloudTask> tasks =
		        await batchClient.JobOperations.ListTasks(JobId, detail).ToListAsync();
		    Console.WriteLine("Awaiting task completion, timeout in {0}...", timeout.ToString());
		    // We use a TaskStateMonitor to monitor the state of our tasks. In this case, we
		    // will wait for all tasks to reach the Completed state.
		    TaskStateMonitor taskStateMonitor = batchClient.Utilities.CreateTaskStateMonitor();
		    bool timedOut = await taskStateMonitor.WaitAllAsync(
		        tasks,
		        TaskState.Completed,
		        timeout);
		    if (timedOut)
		    {
		        allTasksSuccessful = false;
		        await batchClient.JobOperations.TerminateJobAsync(jobId, failureMessage);
		        Console.WriteLine(failureMessage);
		    }
		    else
		    {
		        await batchClient.JobOperations.TerminateJobAsync(jobId, successMessage);
		        // All tasks have reached the "Completed" state. However, this does not
		        // guarantee that all tasks were completed successfully. Here we further
		        // check each task's ExecutionInfo property to ensure that it did not
		        // encounter a scheduling error or return a non-zero exit code.
		        // Update the detail level to populate only the task id and executionInfo
		        // properties. We refresh the tasks below, and need only this information
		        // for each task.
		        detail.SelectClause = "id, executionInfo";
		        foreach (CloudTask task in tasks)
		        {
		            // Populate the task's properties with the latest info from the Batch service
		            await task.RefreshAsync(detail);
		            if (task.ExecutionInformation.SchedulingError != null)
		            {
		                // A scheduling error indicates a problem starting the task on the
		                // node. It is important to note that the task's state can be
		                // "Completed," yet the task still might have encountered a
		                // scheduling error.
		                allTasksSuccessful = false;
		                Console.WriteLine(
		                    "WARNING: Task [{0}] encountered a scheduling error: {1}",
		                    task.Id,
		                    task.ExecutionInformation.SchedulingError.Message);
		            }
		            else if (task.ExecutionInformation.ExitCode != 0)
		            {
		                // A non-zero exit code may indicate that the application executed by
		                // the task encountered an error during execution. As not every
		                // application returns non-zero on failure by default (e.g. robocopy),
		                // your implementation of error checking may differ from this example.
		                allTasksSuccessful = false;
		                Console.WriteLine("WARNING: Task [{0}] returned a non-zero exit code - this may indicate task execution or completion failure.", task.Id);
		            }
		        }
		    }
		    if (allTasksSuccessful)
		    {
		        Console.WriteLine("Success! All tasks completed successfully within the specified timeout period.");
		    }
		    return allTasksSuccessful;
		}


## 步骤 7：下载任务输出

![从存储空间下载任务输出][7]

完成作业后，可以从 Azure 存储空间下载任务的输出。可以在 *DotNetTutorial* 的 `Program.cs` 中调用 `DownloadBlobsFromContainerAsync` 来实现此目的：


		private static async Task DownloadBlobsFromContainerAsync(
		    CloudBlobClient blobClient,
		    string containerName,
		    string directoryPath)
		{
				Console.WriteLine("Downloading all files from container [{0}]...", containerName);
				// Retrieve a reference to a previously created container
				CloudBlobContainer container = blobClient.GetContainerReference(containerName);
				// Get a flat listing of all the block blobs in the specified container
				foreach (IListBlobItem item in container.ListBlobs(
		                    prefix: null,
		                    useFlatBlobListing: true))
				{
						// Retrieve reference to the current blob
						CloudBlob blob = (CloudBlob)item;
						// Save blob contents to a file in the specified folder
						string localOutputFile = Path.Combine(directoryPath, blob.Name);
						await blob.DownloadToFileAsync(localOutputFile, FileMode.Create);
				}
				Console.WriteLine("All files downloaded to {0}", directoryPath);
		}


> [AZURE.NOTE] 对 *DotNetTutorial* 应用程序中 `DownloadBlobsFromContainerAsync` 的调用可以指定应将文件下载到 `%TEMP%` 文件夹。可以随意修改此输出位置。

## 步骤 8：删除容器

由于你需要对位于 Azure 存储空间中的数据付费，因此我们建议删除 Batch 作业不再需要的所有 Blob。在 DotNetTutorial 的 `Program.cs` 中，调用帮助器方法 `DeleteContainerAsync` 三次即可实现此目的：


		// Clean up Storage resources
		await DeleteContainerAsync(blobClient, appContainerName);
		await DeleteContainerAsync(blobClient, inputContainerName);
		await DeleteContainerAsync(blobClient, outputContainerName);


该方法本身只获取对容器的引用，然后调用 [CloudBlobContainer.DeleteIfExistsAsync][net_container_delete]：

		
		private static async Task DeleteContainerAsync(
		    CloudBlobClient blobClient,
		    string containerName)
		{
		    CloudBlobContainer container = blobClient.GetContainerReference(containerName);
		    if (await container.DeleteIfExistsAsync())
		    {
		        Console.WriteLine("Container [{0}] deleted.", containerName);
		    }
		    else
		    {
		        Console.WriteLine("Container [{0}] does not exist, skipping deletion.",
		            containerName);
		    }
		}


## 步骤 9：删除作业和池

在最后一个步骤，系统将提示用户删除 DotNetTutorial 应用程序创建的作业和池。虽然作业和任务本身不收费，但计算节点收费。因此，建议你只在需要的时候分配节点。在维护过程中，可能需要删除未使用的池。

BatchClient 的 [JobOperations][net_joboperations] 和 [PoolOperations][net_pooloperations] 都有对应的删除方法（在用户确认删除时调用）：


		// Clean up the resources we've created in the Batch account if the user so chooses
		Console.WriteLine();
		Console.WriteLine("Delete job? [yes] no");
		string response = Console.ReadLine().ToLower();
		if (response != "n" && response != "no")
		{
		    await batchClient.JobOperations.DeleteJobAsync(JobId);
		}
		Console.WriteLine("Delete pool? [yes] no");
		response = Console.ReadLine();
		if (response != "n" && response != "no")
		{
		    await batchClient.PoolOperations.DeletePoolAsync(PoolId);
		}
		

> [AZURE.IMPORTANT] 请记住，你需要支付计算资源的费用，而删除未使用的池可将费用降到最低。另请注意，删除池也会删除该池内的所有计算节点，并且删除池后，将无法恢复节点上的任何数据。

## 运行DotNetTutorial示例

当你运行示例应用程序时，控制台输出如下所示。在执行期间启动池的计算节点时，你将会遇到暂停并看到`Awaiting task completion, timeout in 00:30:00...`。在执行期间和之后，可以使用 [Batch 资源管理器][github_batchexplorer]来监视池、计算节点、作业和任务。使用某个可用的 [Azure 存储空间资源管理器][storage_explorers]来查看应用程序创建的存储资源（容器和 blob）。

以默认配置运行应用程序时，典型的执行时间**大约为 5 分钟**。


		Sample start: 1/8/2016 09:42:58 AM
		Container [application] created.
		Container [input] created.
		Container [output] created.
		Uploading file C:\repos\azure-batch-samples\CSharp\ArticleProjects\DotNetTutorial\bin\Debug\TaskApplication.exe to container [application]...
		Uploading file Microsoft.WindowsAzure.Storage.dll to container [application]...
		Uploading file ..\..\taskdata1.txt to container [input]...
		Uploading file ..\..\taskdata2.txt to container [input]...
		Uploading file ..\..\taskdata3.txt to container [input]...
		Creating pool [DotNetTutorialPool]...
		Creating job [DotNetTutorialJob]...
		Adding 3 tasks to job [DotNetTutorialJob]...
		Awaiting task completion, timeout in 00:30:00...
		Success! All tasks completed successfully within the specified timeout period.
		Downloading all files from container [output]...
		All files downloaded to C:\Users\USERNAME\AppData\Local\Temp
		Container [application] deleted.
		Container [input] deleted.
		Container [output] deleted.
		Sample end: 1/8/2016 09:47:47 AM
		Elapsed time: 00:04:48.5358142
		Delete job? [yes] no: yes
		Delete pool? [yes] no: yes
		Sample complete, hit ENTER to exit...


## 后续步骤

你可以随意更改 *DotNetTutorial* 和 *TaskApplication*，以体验不同的计算方案。例如，尝试将执行延迟添加到 *TaskApplication*（例如使用 [Thread.Sleep][net_thread_sleep]），以模拟长时间运行的任务并使用 Batch 资源管理器的“热度地图”功能监视这些任务。尝试添加更多任务，或调整计算节点的数目。添加逻辑来检查并允许使用现有的池加速执行时间（“提示”：请查看 [azure-batch-samples][github_samples] 中 [Microsoft.Azure.Batch.Samples.Common][github_samples_common] 项目的 `ArticleHelpers.cs`）。

熟悉 Batch 解决方案的基本工作流后，接下来可以深入了解 Batch 服务的其他功能。

- 查看 [Azure Batch 功能概述](/documentation/articles/batch-api-basics/)这篇文章。如果你对服务不熟悉，建议你阅读该文章。
- 通过 [TopNWords][github_topnwords] 示例了解有关使用 Batch 处理“前 N 个单词”工作负荷的不同实现方式。

[azure_batch]: /services/batch/
[azure_free_account]: /free/
[azure_portal]: https://portal.azure.cn
[batch_explorer_blog]: http://blogs.technet.com/b/windowshpc/archive/2015/01/20/azure-batch-explorer-sample-walkthrough.aspx

[blog_linux]: http://blogs.technet.com/b/windowshpc/archive/2016/03/30/introducing-linux-support-on-azure-batch.aspx
[github_batchexplorer]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/BatchExplorer
[github_dotnettutorial]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/ArticleProjects/DotNetTutorial
[github_samples]: https://github.com/Azure/azure-batch-samples
[github_samples_common]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/Common
[github_samples_zip]: https://github.com/Azure/azure-batch-samples/archive/master.zip
[github_topnwords]: https://github.com/Azure/azure-batch-samples/tree/master/CSharp/TopNWords
[net_api]: http://msdn.microsoft.com/library/azure/mt348682.aspx
[net_api_storage]: https://msdn.microsoft.com/library/azure/mt347887.aspx
[net_batchclient]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.batchclient.aspx
[net_cloudblobclient]: https://msdn.microsoft.com/library/microsoft.windowsazure.storage.blob.cloudblobclient.aspx
[net_cloudblobcontainer]: https://msdn.microsoft.com/library/microsoft.windowsazure.storage.blob.cloudblobcontainer.aspx
[net_cloudstorageaccount]: https://msdn.microsoft.com/library/azure/microsoft.windowsazure.storage.cloudstorageaccount.aspx
[net_cloudserviceconfiguration]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudserviceconfiguration.aspx
[net_container_delete]: https://msdn.microsoft.com/library/microsoft.windowsazure.storage.blob.cloudblobcontainer.deleteifexistsasync.aspx
[net_job]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudjob.aspx
[net_job_poolinfo]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.protocol.models.cloudjob.poolinformation.aspx
[net_joboperations]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.batchclient.joboperations
[net_joboperations_terminatejob]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.joboperations.terminatejobasync.aspx
[net_jobpreptask]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudjob.jobpreparationtask.aspx
[net_jobreltask]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudjob.jobreleasetask.aspx
[net_node]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.computenode.aspx
[net_odatadetaillevel]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.odatadetaillevel.aspx
[net_pool]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudpool.aspx
[net_pool_create]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.pooloperations.createpool.aspx
[net_pool_starttask]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudpool.starttask.aspx
[net_pooloperations]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.batchclient.pooloperations
[net_resourcefile]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.resourcefile.aspx
[net_resourcefile_blobsource]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.resourcefile.blobsource.aspx
[net_sas_blob]: https://msdn.microsoft.com/library/azure/microsoft.windowsazure.storage.blob.cloudblob.getsharedaccesssignature.aspx
[net_sas_container]: https://msdn.microsoft.com/library/azure/microsoft.windowsazure.storage.blob.cloudblobcontainer.getsharedaccesssignature.aspx
[net_starttask]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.starttask.aspx
[net_starttask_resourcefiles]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.starttask.resourcefiles.aspx
[net_task]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudtask.aspx
[net_task_resourcefiles]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.cloudtask.resourcefiles.aspx
[net_taskstate]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.common.taskstate.aspx
[net_taskstatemonitor]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.taskstatemonitor.aspx
[net_thread_sleep]: https://msdn.microsoft.com/library/274eh01d(v=vs.110).aspx
[net_virtualmachineconfiguration]: https://msdn.microsoft.com/library/azure/microsoft.azure.batch.virtualmachineconfiguration.aspx
[nuget_packagemgr]: https://visualstudiogallery.msdn.microsoft.com/27077b70-9dad-4c64-adcf-c7cf6bc9970c
[nuget_restore]: https://docs.nuget.org/consume/package-restore/msbuild-integrated#enabling-package-restore-during-build
[storage_explorers]: http://blogs.msdn.com/b/windowsazurestorage/archive/2014/03/11/windows-azure-storage-explorers-2014.aspx
[visual_studio]: https://www.visualstudio.com/products/vs-2015-product-editions

[1]: ./media/batch-dotnet-get-started/batch_workflow_01_sm.png "在 Azure 存储空间中创建容器"
[2]: ./media/batch-dotnet-get-started/batch_workflow_02_sm.png "将任务应用程序和输入（数据）文件上载到容器"
[3]: ./media/batch-dotnet-get-started/batch_workflow_03_sm.png "创建 Batch 池"
[4]: ./media/batch-dotnet-get-started/batch_workflow_04_sm.png "创建 Batch 作业"
[5]: ./media/batch-dotnet-get-started/batch_workflow_05_sm.png "将任务添加到作业"
[6]: ./media/batch-dotnet-get-started/batch_workflow_06_sm.png "监视任务"
[7]: ./media/batch-dotnet-get-started/batch_workflow_07_sm.png "从存储空间下载任务输出"
[8]: ./media/batch-dotnet-get-started/batch_workflow_sm.png "Batch 解决方案工作流（完整流程图）"
[9]: ./media/batch-dotnet-get-started/credentials_batch_sm.png "门户中的 Batch 凭据"
[10]: ./media/batch-dotnet-get-started/credentials_storage_sm.png "门户中的存储空间凭据"
[11]: ./media/batch-dotnet-get-started/batch_workflow_minimal_sm.png "Batch 解决方案工作流（精简流程图）"

<!---HONumber=Mooncake_0530_2016-->