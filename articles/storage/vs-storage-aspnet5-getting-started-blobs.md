<properties
	pageTitle="Blob 存储和 Visual Studio 连接服务入门 (ASP.NET 5) | Azure"
	description="在使用 Visual Studio 连接服务创建存储帐户后，如何开始在 Visual Studio ASP.NET 5 项目中使用 Azure Blob 存储"
	services="storage"
	documentationCenter=""
	authors="TomArcher"
	manager="douge"
	editor=""/>

<tags 
	ms.service="storage"
	ms.date="05/08/2016"
	wacn.date="06/13/2016"/>

# 开始使用 Azure Blob 存储和 Visual Studio 连接服务 (ASP.NET 5)

##概述

本文介绍通过使用 Visual Studio 中的“添加连接服务”对话框在 ASP.NET 5 项目中已创建或引用 Azure 存储帐户之后，如何开始在 Visual Studio 中使用 Azure Blob 存储。

Azure Blob 存储是一项可存储大量非结构化数据的服务，用户可在世界任何地方通过 HTTP 或 HTTPS 访问这些数据。单个 Blob 可以是任意大小。Blob 可以是图像、音频和视频文件、原始数据以及文档文件等。本文介绍通过使用 Visual Studio 中的“添加连接服务”对话框在 ASP.NET 5 项目中已创建 Azure 存储帐户之后，如何开始使用 blob 存储。

正如文件位于文件夹中一样，存储 Blob 位于容器中。创建存储账号后，可以在存储账号中创建一个或多个容器。例如，在名为“Scrapbook”的存储空间中，可以在名为“images”的存储空间中创建容器，用于存储图片，还可以在名为“audio”的存储空间中创建另一个容器，用于存储音频文件。创建这些容器后，您可以向它们上载单独的 Blob 文件。有关以编程方式操作 blob 的详细信息，请参阅[通过 .NET 开始使用 Azure Blob 存储](/documentation/articles/storage-dotnet-how-to-use-blobs/)。

##使用代码访问 blob 容器

若要以编程方式访问 ASP.NET 5 项目中的 Blob，你需要添加以下项（如果尚未存在）。

1. 在你希望以编程方式访问 Azure 存储服务的任何 C# 文件中，将以下代码命名空间声明添加到文件的顶部。

		using Microsoft.Extensions.Configuration;
		using Microsoft.WindowsAzure.Storage;
		using Microsoft.WindowsAzure.Storage.Blob;
		using System.Threading.Tasks;
		using LogLevel = Microsoft.Extensions.Logging.LogLevel;

2. 获取表示存储帐户信息的 **CloudStorageAccount** 对象。使用下面的代码获取存储连接字符串和 Azure 服务配置中的存储帐户信息。

		 CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
		   CloudConfigurationManager.GetSetting("<storage-account-name>_AzureStorageConnectionString"));

    **注意：**在接下来的部分中，在代码的前面使用上述全部代码


3. 使用 **CloudBlobClient** 对象在存储帐户中获取到的现有容器的 **CloudBlobContainer** 引用。

		// Create a blob client.
		CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();

        // Get a reference to a container named “mycontainer.”
        CloudBlobContainer container = blobClient.GetContainerReference("mycontainer");



##使用代码创建容器

您还可以使用 **CloudBlobClient** 在存储帐户中创建容器。你需要做的就是按照以下代码为 **CreateIfNotExistsAsync** 添加调用：

	// Create a blob client.
    CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();

    // Get a reference to a container named “my-new-container.”
    CloudBlobContainer container = blobClient.GetContainerReference("my-new-container");

    // If “mycontainer” doesn’t exist, create it.
    await container.CreateIfNotExistsAsync();


**注意：**在 ASP.NET 5 中执行调出 Azure 存储的 API 是异步的。有关详细信息，请参阅[使用 Async 和 Await 进行异步编程](http://msdn.microsoft.com/zh-cn/library/hh191443.aspx)。下面的代码假定正在使用异步编程方法。

如果要让容器中的文件可供所有人使用，则可以使用以下代码将容器设置为公共容器。

	await container.SetPermissionsAsync(new BlobContainerPermissions
    {
        PublicAccess = BlobContainerPublicAccessType.Blob
    });

##将 Blob 上载到容器中

若要将 Blob 文件上载到容器中，请获取容器引用，并使用它来获取 Blob 引用。获取 blob 引用后，可以通过调用 **UploadFromStreamAsync** 方法将任何数据流上载到该 blob。此操作将创建 Blob（如果该 Blob 不存在），或者覆盖它（如果该 Blob 存在）。下面的示例演示了如何将 Blob 上载到容器中，并假定已创建容器。

	// Get a reference to a blob named "myblob".
    CloudBlockBlob blockBlob = container.GetBlockBlobReference("myblob");

    // Create or overwrite the "myblob" blob with the contents of a local file
    // named “myfile”.
    using (var fileStream = System.IO.File.OpenRead(@"path\myfile"))
    {
        await blockBlob.UploadFromStreamAsync(fileStream);
    }

##列出容器中的 Blob
若要列出容器中的 Blob，首先需要获取容器引用。然后，可以调用容器的 **ListBlobsSegmentedAsync** 方法检索 blob 和/或其中的目录。若要访问返回的 **IListBlobItem** 的丰富属性和方法，您必须将它转换到 **CloudBlockBlob**、**CloudPageBlob** 或 **CloudBlobDirectory** 对象。如果 Blob 类型未知，您可以使用类型检查来确定要将其转换为哪种类型。以下代码演示了如何检索和输出容器中每项的 URI。

	BlobContinuationToken token = null;
    do
    {
        BlobResultSegment resultSegment = await container.ListBlobsSegmentedAsync(token);
        token = resultSegment.ContinuationToken;

        foreach (IListBlobItem item in resultSegment.Results)
        {
            if (item.GetType() == typeof(CloudBlockBlob))
            {
                CloudBlockBlob blob = (CloudBlockBlob)item;
                Console.WriteLine("Block blob of length {0}: {1}", blob.Properties.Length, blob.Uri);
            }

            else if (item.GetType() == typeof(CloudPageBlob))
            {
                CloudPageBlob pageBlob = (CloudPageBlob)item;

                Console.WriteLine("Page blob of length {0}: {1}", pageBlob.Properties.Length, pageBlob.Uri);
            }

            else if (item.GetType() == typeof(CloudBlobDirectory))
            {
                CloudBlobDirectory directory = (CloudBlobDirectory)item;

                Console.WriteLine("Directory: {0}", directory.Uri);
            }
        }
    } while (token != null);

还有其他方法来列出 Blob 容器内容。有关详细信息，请参阅[通过 .NET 开始使用 Azure Blob 存储](/documentation/articles/storage-dotnet-how-to-use-blobs/#list-the-blobs-in-a-container)。

##下载 Blob
若要下载 blob，请首先获取对该 blob 引用，然后调用 **DownloadToStreamAsync** 方法。下面的示例使用 **DownloadToStreamAsync** 方法将 blob 内容传输到稍后可以另存为本地文件的流对象。

	// Get a reference to a blob named "photo1.jpg".
	CloudBlockBlob blockBlob = container.GetBlockBlobReference("photo1.jpg");

	// Save the blob contents to a file named “myfile”.
	using (var fileStream = System.IO.File.OpenWrite(@"path\myfile"))
	{
    	await blockBlob.DownloadToStreamAsync(fileStream);
	}

还有其他方法将 Blob 另存为文件。有关详细信息，请参阅[通过 .NET 开始使用 Azure Blob 存储](/documentation/articles/storage-dotnet-how-to-use-blobs/#download-blobs)。

##删除 Blob
若要删除 blob，请首先获取对该 blob 的引用，然后调用用于该 blob 的 **DeleteAsync** 方法。

	// Get a reference to a blob named "myblob.txt".
	CloudBlockBlob blockBlob = container.GetBlockBlobReference("myblob.txt");

	// Delete the blob.
	await blockBlob.DeleteAsync();

## 后续步骤

[AZURE.INCLUDE [vs-storage-dotnet-blobs-next-steps](../includes/vs-storage-dotnet-blobs-next-steps.md)]

<!---HONumber=Mooncake_0606_2016-->