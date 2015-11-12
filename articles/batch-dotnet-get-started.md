<properties
	pageTitle="教程 - Azure  批处理( Batch ) .NET 库入门"
	description="了解 Azure  批处理( Batch )的基本概念，以及如何使用一个简单方案开发 批处理( Batch )服务"
	services="batch"
	documentationCenter=".net"
	authors="yidingzhou"
	manager="timlt"
	editor=""/>

<tags
	ms.service="batch"
	ms.date="07/21/2015"
	wacn.date="10/30/2015"/>

# 适用于 .NET 的 Azure  批处理( Batch )库入门  

本教程说明如何创建可设置程序的控制台应用程序，以及可在 Azure  批处理( Batch )池的多个计算节点上运行的支持文件。在本教程中创建的任务先评估 Azure 存储中的文件文本，然后返回最常用的词语。示例是用 C# 代码编写的，并使用了 Azure  批处理( Batch ) .NET 库。

## 先决条件

- 帐户：

	- **Azure 帐户** - 只需几分钟即可创建一个免费试用帐户。有关详细信息，请参阅 [Azure 试用](/pricing/1rmb-trial/)。

	- **批处理( Batch )帐户** - 请参阅 [Azure 批处理( Batch )概述](/documentation/articles/batch-technical-overview)中的**批处理( Batch )帐户**部分。

	- **存储帐户** - 请参阅[关于 Azure 存储帐户](/documentation/articles/storage-create-storage-account)的**创建存储帐户**部分。在本教程中，你将在此帐户中创建名为 **testcon1** 的容器。

- Visual Studio 控制台应用程序项目：

	1.  打开 Visual Studio，在“文件”菜单中，单击“新建”，然后单击“项目”。

	2.	在“Windows”的“Visual C#”下，单击“控制台应用程序”，将项目命名为 **GettingStarted**，将解决方案命名为 **AzureBatch**，然后单击“确定”。

- NuGet 程序集：

	1. 在 Visual Studio 中创建项目后，可在“Solution Explorer”中右键单击该项目，然后选择“Manage NuGet Packages”。在线搜索 **Azure.Batch**，然后单击“安装”以安装 Microsoft Azure Batch包和依赖项。

	2. 在线搜索 **WindowsAzure.Storage**，然后单击“安装”以安装 Azure 存储包和依赖项。

## 步骤 1：创建并上载支持文件

为了支持应用程序，请在 Azure 存储空间中创建一个容器，再创建文本文件，然后将文本文件和支持文件上载到该容器。

### 设置存储连接字符串

1. 打开 GettingStarted 项目的 App.config 文件，然后将 &lt;appSettings&gt; 元素添加到 &lt;configuration&gt;。

		<?xml version="1.0" encoding="utf-8" ?>
		<configuration>
			<appSettings>
				<add key="StorageConnectionString" value="DefaultEndpointsProtocol=https;AccountName=[account-name];AccountKey=[account-key];EndpointSuffix=core.chinacloudapi.cn"/>
			</appSettings>
		</configuration>

	替换以下值：

	- **[account-name]** - 前面创建的存储帐户的名称。

	- **[account-key]** - 存储帐户的主密钥。你可以在管理门户的“存储”页中找到主密钥

2. 保存 App.config 文件。

若要了解详细信息，请参阅[配置连接字符串](/documentation/articles/storage-configure-connection-string/)。

### 创建存储容器

1. 将以下命名空间声明添加到 GettingStarted 项目中 Program.cs 的顶部：

		using System.Configuration;
		using Microsoft.WindowsAzure.Storage;
		using Microsoft.WindowsAzure.Storage.Blob;

2. 将此方法添加到 Program 类，该方法将获取存储连接字符串、创建容器，并设置权限：

		static void CreateStorage()
		{
			// Get the storage connection string
			CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
				ConfigurationManager.AppSettings["StorageConnectionString"]);

			// Create the container
			CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();
			CloudBlobContainer container = blobClient.GetContainerReference("testcon1");
			container.CreateIfNotExists();

			// Set permissions on the container
			BlobContainerPermissions containerPermissions = new BlobContainerPermissions();
			containerPermissions.PublicAccess = BlobContainerPublicAccessType.Blob;
			container.SetPermissions(containerPermissions);
			Console.WriteLine("Created the container. Press Enter to continue.");
			Console.ReadLine();
		}

3. 将以下用于调用刚刚添加的方法的代码添加到到 Main 中：

		CreateStorage();

4. 保存 Program.cs 文件。

	> [AZURE.NOTE]在生产环境中，建议你使用共享访问签名。

若要了解详细信息，请参阅[如何通过 .NET 使用 Blob 存储](/documentation/articles/storage-dotnet-how-to-use-blobs)

### 创建处理程序

1. 在解决方案资源管理器中，创建名为 **ProcessTaskData** 的新控制台应用程序项目。

2. 将以下用于处理文件中文本的代码添加到 Main 中：

		string blobName = args[0];
		Uri blobUri = new Uri(blobName);
		int numTopN = int.Parse(args[1]);

		CloudBlockBlob blob = new CloudBlockBlob(blobUri);
		string content = blob.DownloadText();
		string[] words = content.Split(' ');
		var topNWords =
		  words.
			Where(word => word.Length > 0).
			GroupBy(word => word, (key, group) => new KeyValuePair<String, long>(key, group.LongCount())).
			OrderByDescending(x => x.Value).
			Take(numTopN).
			ToList();

		foreach (var pair in topNWords)
		{
			Console.WriteLine("{0} {1}", pair.Key, pair.Value);
		}

3. 保存并生成 ProcessTaskData 项目。

### 上载数据文件

1. 在 GettingStarted 项目中，创建一个名为 **taskdata1** 的新文本文件，将以下文本复制到其中，然后保存该文件。

	当你需要弹性资源来满足业务需求时，可使用 Azure 虚拟机来设置可按需缩放的计算基础结构。可以从库中创建运行 Windows、Linux 和企业应用程序（如 SharePoint 和 SQL Server）的虚拟机。也可以捕获和使用你自己的映像来创建自定义的虚拟机。

2. 创建一个名为 **taskdata2** 的新文本文件，将以下文本复制到其中，然后保存该文件。

	使用 Azure 云服务快速部署和管理功能强大的应用程序和服务。只需上载应用程序，Azure 将处理部署细节（从设置和负载平衡到运行状况监视）以实现持续可用性。您的应用程序由行业领先的每月 99.95% 的 SLA 提供支持。你只需专注于应用程序而非基础结构。

3. 创建一个名为 **taskdata3** 的新文本文件，将以下文本复制到其中，然后保存该文件。

	Azure 网站为托管 Web 应用程序提供了可缩放的、可靠的且易于使用的环境。从一系列框架和模板中进行选择，几秒钟就可以创建一个网站。使用任何工具或 OS 以利用 .NET、PHP、Node.js 或 Python 开发网站。从各种源代码管理选项（包括 TFS、GitHub 和 BitBucket）中进行选择，可设置持续集成并像一个团队一样进行开发。利用其他 Azure 托管服务（如存储、CDN 和 SQL Database）随时间扩展网站功能。

### 将文件上载到容器

1. 打开 GettingStarted 项目的 Program.cs 文件，然后添加以下用于上载文件的方法：

		static void CreateFiles()
		{
		  CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
			ConfigurationManager.AppSettings["StorageConnectionString"]);
		  CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();
		  CloudBlobContainer container = blobClient.GetContainerReference("testcon1");
		  CloudBlockBlob taskData1 = container.GetBlockBlobReference("taskdata1");
		  CloudBlockBlob taskData2 = container.GetBlockBlobReference("taskdata2");
		  CloudBlockBlob taskData3 = container.GetBlockBlobReference("taskdata3");
	  	CloudBlockBlob dataprocessor = container.GetBlockBlobReference("ProcessTaskData.exe");
	  	CloudBlockBlob storageassembly =
			container.GetBlockBlobReference("Microsoft.WindowsAzure.Storage.dll");
		  taskData1.UploadFromFile("..\\..\\taskdata1.txt", FileMode.Open);
		  taskData2.UploadFromFile("..\\..\\taskdata2.txt", FileMode.Open);
	  	taskData3.UploadFromFile("..\\..\\taskdata3.txt", FileMode.Open);
		  dataprocessor.UploadFromFile("..\\..\\..\\ProcessTaskData\\bin\\debug\\ProcessTaskData.exe", FileMode.Open);
		  storageassembly.UploadFromFile("Microsoft.WindowsAzure.Storage.dll", FileMode.Open);
		  Console.WriteLine("Uploaded the files. Press Enter to continue.");
		  Console.ReadLine();
		}
2. 将以下用于调用刚刚添加的方法的代码添加到到 Main 中：

		CreateFiles();
3. 保存 Program.cs 文件。

## 步骤 2.将池添加到帐户

计算节点池是要运行任务时必须创建的第一组资源。

1.	将以下命名空间声明添加到 GettingStarted 项目中 Program.cs 的顶部：

			using Microsoft.Azure.Batch;
			using Microsoft.Azure.Batch.Auth;
			using Microsoft.Azure.Batch.Common;

2. 将以下用于设置凭据以调用 Azure Batch服务的代码添加到 Main 中：

			BatchSharedKeyCredentials cred = new BatchSharedKeyCredentials("https://[account-name].[region].batch.chinacloudapi.cn/", "[account-name]", "[account-key]");
			BatchClient client = BatchClient.Open(cred);

	替换以下值：

	- 将 **[account-name]** 替换为前面创建的批处理( Batch )帐户的名称。
	- 将 **[region]** 替换为帐户所在的区域。<!--See [Azure Regions](http://azure.microsoft.com/regions/) 以了解可用的区域。-->
	- 将 **[account-key]** 替换为 批处理( Batch )帐户的主密钥。

3.	将以下用于创建池的方法添加到 Program 类：

			static void CreatePool(BatchClient client)
			{
			  CloudPool newPool = client.PoolOperations.CreatePool(
			    "testpool1",
			    "3",
			    "small",
			    3);
			  newPool.Commit();
			  Console.WriteLine("Created the pool. Press Enter to continue.");
			  Console.ReadLine();
		  }

4. 将以下用于调用刚刚添加的方法的代码添加到 Main 中：

		CreatePool(client);

5. 将以下用于列出帐户中的池的方法添加到 Program 类。这有助于验证是否已创建池：

			static void ListPools(BatchClient client)
			{
				IPagedEnumerable<CloudPool> pools = client.PoolOperations.ListPools();
				foreach (CloudPool pool in pools)
				{
					Console.WriteLine("Pool name: " + pool.Id);
					Console.WriteLine("   Pool status: " + pool.State);
				}
				Console.WriteLine("Press enter to continue.");
				Console.ReadLine();
			}

6. 将以下用于调用刚刚添加的方法的代码添加到 Main 中：

		ListPools(client);

7. 保存 Program.cs 文件。

## 步骤 2：将作业添加到帐户

创建一个作业用于管理池中运行的任务。所有任务都必须与作业关联。

1. 将以下用于创建作业的方法添加到 Program 类：

		static CloudJob CreateJob (BatchClient client)
		{
			CloudJob newJob = client.JobOperations.CreateJob();
			newJob.Id = "testjob1";
			newJob.PoolInformation = new PoolInformation() { PoolId = "testpool1" };
			newJob.Commit();
			Console.WriteLine("Created the job. Press Enter to continue.");
			Console.ReadLine();

			return newJob;
		}

2. 将以下用于调用刚刚添加的方法的代码添加到 Main 中：

		CreateJob(client);

3. 将以下用于列出帐户中的作业的方法添加到 Program 类。这有助于验证是否已创建作业：

		static void ListJobs (BatchClient client)
		{
			IPagedEnumerable<CloudJob> jobs = client.JobOperations.ListJobs();
			foreach (CloudJob job in jobs)
			{
				Console.WriteLine("Job id: " + job.Id);
				Console.WriteLine("   Job status: " + job.State);
			}
			Console.WriteLine("Press Enter to continue.")
			Console.ReadLine();
		}

4. 将以下用于调用刚刚添加的方法的代码添加到到 Main 中：

		ListJobs(client);

5. 保存 Program.cs 文件。

## 步骤 3：将任务添加到作业

创建作业之后，可以在其中添加任务。每个任务都在计算节点上运行，并处理文本文件。在本教程中，你要将三个任务添加到作业。

1. 将以下方法添加到 Program 类，该方法会将三个任务添加到作业：

		static void AddTasks(BatchClient client)
		{
			CloudJob job = client.JobOperations.GetJob("testjob1");
			ResourceFile programFile = new ResourceFile(
				"https://[account-name].blob.core.chinacloudapi.cn/testcon1/ProcessTaskData.exe",
				"ProcessTaskData.exe");
      	  ResourceFile assemblyFile = new ResourceFile(
				"https://[account-name].blob.core.chinacloudapi.cn/testcon1/Microsoft.WindowsAzure.Storage.dll",
				"Microsoft.WindowsAzure.Storage.dll");
			for (int i = 1; i < 4; ++i)
			{
				string blobName = "taskdata" + i;
				string taskName = "mytask" + i;
				ResourceFile taskData = new ResourceFile("https://[account-name].blob.core.chinacloudapi.cn/testcon1/" +
				  blobName, blobName);
				CloudTask task = new CloudTask(taskName, "ProcessTaskData.exe https://[account-name].blob.core.windows.net/testcon1/" +
				  blobName + " 3");
				List<ResourceFile> taskFiles = new List<ResourceFile>();
				taskFiles.Add(taskData);
				taskFiles.Add(programFile);
				taskFiles.Add(assemblyFile);
				task.ResourceFiles = taskFiles;
				job.AddTask(task);
				job.Commit();
				job.Refresh();
			}

			client.Utilities.CreateTaskStateMonitor().WaitAll(job.ListTasks(),
        TaskState.Completed, new TimeSpan(0, 30, 0));
			Console.WriteLine("The tasks completed successfully.");
			foreach (CloudTask task in job.ListTasks())
			{
				Console.WriteLine("Task " + task.Id + " says:\n" + task.GetNodeFile(Constants.StandardOutFileName).ReadAsString());
			}
			Console.WriteLine("Press Enter to continue.")
			Console.ReadLine();
		}

	需要将 **[account-name]** 替换为前面创建的存储帐户的名称。请确保在所有四处进行替换。

2. 将以下用于调用刚刚添加的方法的代码添加到 Main 中：

			AddTasks(client);

3. 将以下方法添加到 Program 类，该方法将列出与作业关联的任务：

		static void ListTasks(BatchClient client)
		{
			IPagedEnumerable<CloudTask> tasks = client.JobOperations.ListTasks("testjob1");
			foreach (CloudTask task in tasks)
			{
				Console.WriteLine("Task id: " + task.Id);
				Console.WriteLine("   Task status: " + task.State);
			  Console.WriteLine("   Task start: " + task.ExecutionInformation.StartTime);
			}
			Console.ReadLine();
		}

4. 将以下用于调用刚刚添加的方法的代码添加到 Main 中：

		ListTasks(client);

5. 保存 Program.cs 文件。

## 步骤 4：删除资源

由于你需要为 Azure 中的资源付费，因此，删除不再需要的资源总是一种良好的做法。

### 删除任务

1.	将以下用于删除任务的方法添加到 Program 类：

			static void DeleteTasks(BatchClient client)
			{
				CloudJob job = client.JobOperations.GetJob("testjob1");
				foreach (CloudTask task in job.ListTasks())
				{
					task.Delete();
				}
				Console.WriteLine("All tasks deleted.");
				Console.ReadLine();
			}

2. 将以下用于调用刚刚添加的方法的代码添加到到 Main 中：

		DeleteTasks(client);

3. 保存 Program.cs 文件。

### 删除作业

1.	将以下用于删除作业的方法添加到 Program 类：

			static void DeleteJob(BatchClient client)
			{
				client.JobOperations.DeleteJob("davidmujob1");
				Console.WriteLine("Job was deleted.");
				Console.ReadLine();
			}

2. 将以下用于运行刚刚添加的方法的代码添加到到 Main 中：

		DeleteJob(client);

3. 保存 Program.cs 文件。

### 删除池

1. 将以下用于删除池的方法添加到 Program 类：

		static void DeletePool (BatchClient client)
		{
			client.PoolOperations.DeletePool("davidmupl1");
			Console.WriteLine("Pool was deleted.");
			Console.ReadLine();
		}

2. 将以下用于运行刚刚添加的方法的代码添加到到 Main 中：

		DeletePool(client);

3. 保存 Program.cs 文件。

## 步骤 5：运行应用程序

1. 启动 GettingStarted 项目，你应会在容器创建后看到此项目出现在控制台窗口中：

		Created the container. Press Enter to continue.

2. 按 Enter，随后将会创建并上载文件，你现在应会在窗口中看到新的一行：

		Uploaded the files. Press Enter to continue.

3. 按 Enter，随后将会创建池：

		Created the pool. Press Enter to continue.

4. 按 Enter，你应会看到以下新池列表：

		Pool name: testpool1
			Pool status: Active
		Press Enter to continue.

5. 按 Enter，随后将会创建作业：

		Created the job. Press Enter to continue.

6. 按 Enter，你应会看到以下新作业列表：

		Job id: testjob1
			Job status: Active
		Press Enter to continue.

7. 按 Enter，任务将会添加到作业中。添加任务后，它们会自动运行：

		The tasks completed successfully.
		Task mytask1 says:
		can 3
		you 3
		and 3

		Task mytask2 says:
		and 5
		application 3
		the 3

		Task mytask3 says:
		a 5
		and 5
		to 3

		Press Enter to continue.

7. 按 Enter，你应会看到任务及其状态的列表：

		Task id: mytask1
			Task status: Completed
			Task start: 7/17/2015 8:31:58 PM
		Task id: mytask2
			Task status: Completed
			Task start: 7/17/2015 8:31:57 PM
		Task id: mytask3
			Task status: Completed
			Task start: 7/17/2015 8:31:57 PM

8. 此时，你可以进入 Azure 门户，查看所创建的资源。若要删除资源，请按 Enter，直到程序完成。

## 后续步骤

1. 现在你已学习了运行任务的基本知识，接下来可以学习如何在应用程序的需求发生变化时自动缩放计算节点。为此，请参阅[自动缩放 Azure 批处理( Batch )池中的计算节点](/documentation/articles/batch-automatic-scaling)

2. 某些应用程序会生成大量难以处理的数据。解决此问题的方法之一是进行[有效的列表查询](/documentation/articles/batch-efficient-list-queries)。

<!---HONumber=66-->