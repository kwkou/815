<properties
	pageTitle="通过 .NET 开始使用 Azure Blob 存储 | Azure"
	description="使用 Azure Blob 存储（对象存储）将非结构化数据存储在云中。"
	services="storage"
	documentationCenter=".net"
	authors="tamram"
	manager="adinah"
	editor=""/>

<tags
	ms.service="storage"
	ms.date="04/25/2016"
	wacn.date="06/06/2016"/>


# 通过 .NET 开始使用 Azure Blob 存储

[AZURE.INCLUDE [storage-selector-blob-include](../includes/storage-selector-blob-include.md)]

## 概述

Azure Blob 存储是一种将非结构化数据作为对象/Blob 存储在云中的服务。Blob 存储可以存储任何类型的文本或二进制数据，例如文档、媒体文件或应用程序安装程序。Blob 存储也称为对象存储。

### 关于本教程

本教程演示如何针对使用 Azure Blob 存储一些常见情形编写 .NET 代码。涉及的任务包括上载、列出、下载和删除 Blob。

**估计完成时间：**45 分钟

**先决条件：**

- [Microsoft Visual Studio](https://www.visualstudio.com/zh-cn/visual-studio-homepage-vs.aspx)
- [适用于 .NET 的 Azure 存储空间客户端库](https://www.nuget.org/packages/WindowsAzure.Storage/)
- [适用于 .NET 的 Azure Configuration Manager](https://www.nuget.org/packages/Microsoft.WindowsAzure.ConfigurationManager/)
- [Azure 存储帐户](/documentation/articles/storage-create-storage-account/#create-a-storage-account)。


[AZURE.INCLUDE [storage-dotnet-client-library-version-include](../includes/storage-dotnet-client-library-version-include.md)]

[AZURE.INCLUDE [storage-blob-concepts-include](../includes/storage-blob-concepts-include.md)]

[AZURE.INCLUDE [storage-create-account-include](../includes/storage-create-account-include.md)]

[AZURE.INCLUDE [storage-development-environment-include](../includes/storage-development-environment-include.md)]

### 添加命名空间声明

将下列 `using` 语句添加到 `program.cs` 文件顶部：

	using Microsoft.Azure; // Namespace for CloudConfigurationManager 
	using Microsoft.WindowsAzure.Storage; // Namespace for CloudStorageAccount
    using Microsoft.WindowsAzure.Storage.Blob; // Namespace for Blob storage types

### 解析连接字符串

[AZURE.INCLUDE [storage-cloud-configuration-manager-include](../includes/storage-cloud-configuration-manager-include.md)]

### 创建 Blob 服务客户端

**CloudBlobClient** 类使你能够在 Blob 存储中检索容器和 blob。下面是创建服务客户端的一种方法：

    CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();

现在，你已准备好编写从 Blob 存储读取数据并将数据写入 Blob 存储的代码。

## 创建容器

[AZURE.INCLUDE [storage-container-naming-rules-include](../includes/storage-container-naming-rules-include.md)]

此示例演示如何创建一个容器（如果该容器不存在）：

    // Retrieve storage account from connection string.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the blob client.
    CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();

    // Retrieve a reference to a container.
    CloudBlobContainer container = blobClient.GetContainerReference("mycontainer");

    // Create the container if it doesn't already exist.
    container.CreateIfNotExists();

默认情况下，新容器是专用容器，意思是必须指定存储访问密钥才能从该容器下载 blob。如果你要让容器中的文件可供所有人使用，则可以使用以下代码将容器设置为公共容器：

    container.SetPermissions(
        new BlobContainerPermissions { PublicAccess = BlobContainerPublicAccessType.Blob });

Internet 中的所有人都可以查看公共容器中的 blob，但是，仅在你具有相应的帐户访问密钥或共享的访问签名时，才能修改或删除它们。

## 将 Blob 上载到容器中

Azure Blob 存储支持块 Blob 和页 Blob。大多数情况下，推荐使用块 Blob。

若要将文件上载到块 Blob，请获取容器引用，并使用它获取块 Blob 引用。获取 Blob 引用后，可以通过调用 **UploadFromStream** 方法，将任何数据流上载到该 Blob。如果之前不存在 Blob，此操作将创建一个；如果存在 Blob，此操作将覆盖它。

下面的示例演示了如何将 Blob 上载到容器中，并假定已创建容器。

    // Retrieve storage account from connection string.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the blob client.
    CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();

    // Retrieve reference to a previously created container.
    CloudBlobContainer container = blobClient.GetContainerReference("mycontainer");

    // Retrieve reference to a blob named "myblob".
    CloudBlockBlob blockBlob = container.GetBlockBlobReference("myblob");

    // Create or overwrite the "myblob" blob with contents from a local file.
    using (var fileStream = System.IO.File.OpenRead(@"path\myfile"))
    {
        blockBlob.UploadFromStream(fileStream);
    }

## 列出容器中的 Blob

若要列出容器中的 Blob，首先需要获取容器引用。然后，您可以使用容器的 **ListBlobs** 方法来检索其中的 Blob 和/或目录。若要访问返回的 **IListBlobItem** 的丰富属性和方法，您必须将它转换到 **CloudBlockBlob**、**CloudPageBlob** 或 **CloudBlobDirectory** 对象。如果类型未知，你可以使用类型检查来确定要将其转换为哪种类型。以下代码演示了如何检索和输出 `photos` 容器中每一项的 URI：

    // Retrieve storage account from connection string.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the blob client.
	CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();

	// Retrieve reference to a previously created container.
	CloudBlobContainer container = blobClient.GetContainerReference("photos");

	// Loop over items within the container and output the length and URI.
	foreach (IListBlobItem item in container.ListBlobs(null, false))
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

如上所示，您可以命名 Blob，在其名称中包含路径信息。这将创建一个虚拟目录结构，你可以像传统文件系统一样组织和遍历。注意，该目录结构仅仅是虚拟的 -  Blob 存储中唯一可用的资源是容器和 Blob。但是，存储空间客户端库提供 **CloudBlobDirectory** 对象来引用虚拟目录，并简化了以这种方式组织的 Blob 的使用过程。

例如，考虑名为 `photos` 的容器中包含的下面一组块 Blob：

	photo1.jpg
	2010/architecture/description.txt
	2010/architecture/photo3.jpg
	2010/architecture/photo4.jpg
	2011/architecture/photo5.jpg
	2011/architecture/photo6.jpg
	2011/architecture/description.txt
	2011/photo7.jpg

当您在“photos”容器上调用 **ListBlobs**（如上面的示例中）时，返回一个层次结构列表。它包含 **CloudBlobDirectory** 和 **CloudBlockBlob** 对象，分别表示容器中的目录和 Blob。生成的输出如下所示：

	Directory: https://<accountname>.blob.core.chinacloudapi.cn/photos/2010/
	Directory: https://<accountname>.blob.core.chinacloudapi.cn/photos/2011/
	Block blob of length 505623: https://<accountname>.blob.core.chinacloudapi.cn/photos/photo1.jpg


另外，也可以将 **ListBlobs** 方法的 **UseFlatBlobListing** 参数设置为 **true**。在这种情况下，作为 **CloudBlockBlob** 对象返回容器中的每一个 Blob。对 **ListBlobs** 的调用返回一个平面列表，如下所示：

    // Loop over items within the container and output the length and URI.
	foreach (IListBlobItem item in container.ListBlobs(null, true))
	{
	   ...
	}

结果如下所示：

	Block blob of length 4: https://<accountname>.blob.core.chinacloudapi.cn/photos/2010/architecture/description.txt
	Block blob of length 314618: https://<accountname>.blob.core.chinacloudapi.cn/photos/2010/architecture/photo3.jpg
	Block blob of length 522713: https://<accountname>.blob.core.chinacloudapi.cn/photos/2010/architecture/photo4.jpg
	Block blob of length 4: https://<accountname>.blob.core.chinacloudapi.cn/photos/2011/architecture/description.txt
	Block blob of length 419048: https://<accountname>.blob.core.chinacloudapi.cn/photos/2011/architecture/photo5.jpg
	Block blob of length 506388: https://<accountname>.blob.core.chinacloudapi.cn/photos/2011/architecture/photo6.jpg
	Block blob of length 399751: https://<accountname>.blob.core.chinacloudapi.cn/photos/2011/photo7.jpg
	Block blob of length 505623: https://<accountname>.blob.core.chinacloudapi.cn/photos/photo1.jpg


## 下载 Blob

若要下载 Blob，请首先检索 Blob 引用，然后调用 **DownloadToStream** 方法。以下示例使用 **DownloadToStream** 方法将 Blob 内容传输到一个流对象，然后您可以将该对象保存到本地文件。

    // Retrieve storage account from connection string.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the blob client.
    CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();

    // Retrieve reference to a previously created container.
    CloudBlobContainer container = blobClient.GetContainerReference("mycontainer");

    // Retrieve reference to a blob named "photo1.jpg".
    CloudBlockBlob blockBlob = container.GetBlockBlobReference("photo1.jpg");

    // Save blob contents to a file.
    using (var fileStream = System.IO.File.OpenWrite(@"path\myfile"))
    {
        blockBlob.DownloadToStream(fileStream);
    }

也可以使用 **DownloadToStream** 方法以文本字符串形式下载 Blob 的内容。

	// Retrieve storage account from connection string.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the blob client.
    CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();

    // Retrieve reference to a previously created container.
    CloudBlobContainer container = blobClient.GetContainerReference("mycontainer");

	// Retrieve reference to a blob named "myblob.txt"
	CloudBlockBlob blockBlob2 = container.GetBlockBlobReference("myblob.txt");

	string text;
	using (var memoryStream = new MemoryStream())
	{
		blockBlob2.DownloadToStream(memoryStream);
		text = System.Text.Encoding.UTF8.GetString(memoryStream.ToArray());
	}

## 删除 Blob

若要删除 Blob，首先要获取 Blob 引用，然后对其调用 **Delete** 方法。

    // Retrieve storage account from connection string.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the blob client.
    CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();

    // Retrieve reference to a previously created container.
    CloudBlobContainer container = blobClient.GetContainerReference("mycontainer");

    // Retrieve reference to a blob named "myblob.txt".
    CloudBlockBlob blockBlob = container.GetBlockBlobReference("myblob.txt");

    // Delete the blob.
    blockBlob.Delete();


## 以异步方式列出页中的 Blob

如果要列出大量 Blob，或需要控制一个列操作中返回的结果数，则可以结果页的方式列出 Blob。此示例显示如何以页面形式异步返回结果，这样就不会在等待返回大型结果集时阻止操作的执行。

此示例演示平面 Blob 列表，但您也可以执行分层列表，只需将 **ListBlobsSegmentedAsync** 方法的 `useFlatBlobListing` 参数设置为 `false` 即可。

由于示例方法调用异步方法，因此必须以 `async` 关键字开头，且必须返回 **Task** 对象。为 **ListBlobsSegmentedAsync** 方法指定的 await 关键字将挂起示例方法的执行，直至列表任务完成。

    async public static Task ListBlobsSegmentedInFlatListing(CloudBlobContainer container)
    {
        //List blobs to the console window, with paging.
        Console.WriteLine("List blobs in pages:");

        int i = 0;
        BlobContinuationToken continuationToken = null;
        BlobResultSegment resultSegment = null;

        //Call ListBlobsSegmentedAsync and enumerate the result segment returned, while the continuation token is non-null.
        //When the continuation token is null, the last page has been returned and execution can exit the loop.
        do
        {
            //This overload allows control of the page size. You can return all remaining results by passing null for the maxResults parameter,
            //or by calling a different overload.
            resultSegment = await container.ListBlobsSegmentedAsync("", true, BlobListingDetails.All, 10, continuationToken, null, null);
            if (resultSegment.Results.Count<IListBlobItem>() > 0) { Console.WriteLine("Page {0}:", ++i); }
            foreach (var blobItem in resultSegment.Results)
            {
                Console.WriteLine("\t{0}", blobItem.StorageUri.PrimaryUri);
            }
            Console.WriteLine();

            //Get the continuation token.
            continuationToken = resultSegment.ContinuationToken;
        }
        while (continuationToken != null);
    }

## 写入追加 Blob

追加 Blob 是一个新类型的 Blob，由用于 .NET 的 Azure 存储服务客户端库 5.x 版引入。追加 Blob 针对追加操作（例如日志记录）进行了优化。类似于块 Blob，追加 Blob 由块组成，但是当您将新的块添加到追加 Blob 时，始终追加到该 Blob 的末尾。您不能更新或删除追加 Blob 中现有的块。追加 Blob 的块 ID 不公开，因为它们是用于一个块存储(block blob)。
 
追加 Blob 中的每个块可以有不同的大小，最大为 4 MB，并且追加 Blob 最多可包含 50000 个块。因此，追加 Blob 的最大容量稍微大于 195 GB（4 MB X 50000 块）。

下面的示例创建一个新的追加 Blob 并向其追加某些数据，模拟一个简单的日志记录操作。

    //Parse the connection string for the storage account.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        Microsoft.Azure.CloudConfigurationManager.GetSetting("StorageConnectionString"));

    //Create service client for credentialed access to the Blob service.
    CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();

    //Get a reference to a container.
    CloudBlobContainer container = blobClient.GetContainerReference("my-append-blobs");

    //Create the container if it does not already exist.
    container.CreateIfNotExists();

    //Get a reference to an append blob.
    CloudAppendBlob appendBlob = container.GetAppendBlobReference("append-blob.log");

    //Create the append blob. Note that if the blob already exists, the CreateOrReplace() method will overwrite it.
    //You can check whether the blob exists to avoid overwriting it by using CloudAppendBlob.Exists().
    appendBlob.CreateOrReplace();

    int numBlocks = 10;

    //Generate an array of random bytes.
    Random rnd = new Random();
    byte[] bytes = new byte[numBlocks];
    rnd.NextBytes(bytes);

    //Simulate a logging operation by writing text data and byte data to the end of the append blob.
    for (int i = 0; i < numBlocks; i++)
    {
        appendBlob.AppendText(String.Format("Timestamp: {0:u} \tLog Entry: {1}{2}",
            DateTime.UtcNow, bytes[i], Environment.NewLine));
    }

    //Read the append blob to the console window.
    Console.WriteLine(appendBlob.DownloadText());

请参阅[了解块 Blob、页 Blob 和追加 Blob](https://msdn.microsoft.com/zh-cn/library/azure/ee691964.aspx)，就有关三种 Blob 之间的差异了解详细信息。

## 管理 Blob 安全性

默认情况下，Azure 存储空间会限制拥有帐户访问密钥的帐户所有者的访问权限来保持数据安全。当你需要共享存储帐户中的 Blob 数据时，请注意不可危及帐户访问密钥的安全性。此外，可以加密 Blob 数据，以确保其在网络中传输时以及在 Azure 存储空间中时的安全性。

[AZURE.INCLUDE [storage-account-key-note-include](../includes/storage-account-key-note-include.md)]

### 控制对 Blob 数据的访问

默认情况下，你的存储帐户中的 Blob 数据仅供存储帐户所有者访问。默认情况下，验证对 Blob 存储的请求需要帐户访问密钥。不过，你可能想要让特定的 Blob 数据可供其他用户使用。可以使用两个选项：

- **匿名访问**：你可让容器或其 Blob 公开供匿名访问。有关详细信息，请参阅[管理对容器和 Blob 的匿名读取访问](/documentation/articles/storage-manage-access-to-resources/)。
- **共享访问签名**：你可以为客户端提供共享访问签名 (SAS)，该共享访问签名可利用所指定的权限在所指定的时间间隔内，针对存储帐户中的资源提供委派访问权限。有关详细信息，请参阅[共享访问签名：了解 SAS 模型](/documentation/articles/storage-dotnet-shared-access-signature-part-1/)。

### 加密 Blob 数据

Azure 存储空间支持在客户端和服务器上加密 Blob 数据：

- **客户端加密**：用于 .NET 的存储客户端库支持在上传到 Azure 存储空间之前加密客户端应用程序中的数据，以及在下载到客户端时解密数据。此库还支持与 Azure 密钥保管库集成，以便管理存储帐户密钥。有关详细信息，请参阅 [Microsoft Azure 存储空间的使用 .NET 客户端加密](/documentation/articles/storage-client-side-encryption/)。另请参阅[教程：在 Microsoft Azure 存储空间中使用 Azure 密钥保管库加密和解密 Blob](/documentation/articles/storage-encrypt-decrypt-blobs-key-vault/)。

## 后续步骤

现在，你已了解 Blob 存储的基础知识，可单击下面的链接了解详细信息。

### Blob 存储参考

- [.NET 存储客户端库参考](https://msdn.microsoft.com/zh-cn/library/mt347887.aspx)
- [REST API 参考](http://msdn.microsoft.com/zh-cn/library/azure/dd179355)

### 概念性指南

- [使用 AzCopy 命令行实用程序传输数据](/documentation/articles/storage-use-azcopy/)
- [开始使用适用于 .NET 的文件存储](/documentation/articles/storage-dotnet-how-to-use-files/)
- [如何通过 WebJobs SDK 使用 Azure Blob 存储](/documentation/articles/app-service-web/websites-dotnet-webjobs-sdk-storage-blobs-how-to)

  [Blob5]: ./media/storage-dotnet-how-to-use-blobs/blob5.png
  [Blob6]: ./media/storage-dotnet-how-to-use-blobs/blob6.png
  [Blob7]: ./media/storage-dotnet-how-to-use-blobs/blob7.png
  [Blob8]: ./media/storage-dotnet-how-to-use-blobs/blob8.png
  [Blob9]: ./media/storage-dotnet-how-to-use-blobs/blob9.png

  [Azure 存储团队博客]: http://blogs.msdn.com/b/windowsazurestorage/
  [配置连接字符串]: http://msdn.microsoft.com/zh-cn/library/azure/ee758697.aspx
  [.NET 客户端库引用]: https://msdn.microsoft.com/zh-cn/library/mt347887.aspx
  [REST API 参考]: http://msdn.microsoft.com/zh-cn/library/azure/dd179355
 
<!---HONumber=Mooncake_0509_2016-->