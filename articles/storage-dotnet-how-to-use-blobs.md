<properties urlDisplayName="Blob Service" pageTitle="如何通过 .NET 使用 Blob 存储 | Azure" metaKeywords="Get started Azure blob   Azure unstructured data   Azure unstructured storage   Azure blob   Azure blob storage   Azure blob .NET   Azure blob C#   Azure blob C#" description="了解如何使用 Microsoft Azure Blob 存储上载、下载、列出和删除 Blob 内容。相关示例用 C# 编写。" metaCanonical="" disqusComments="1" umbracoNaviHide="1" services="storage" documentationCenter=".NET" title="How to use Windows Azure Blob storage in .NET" authors="tamram" manager="adinah" />

# 如何通过 .NET 使用 Blob 存储

本指南将演示如何使用 Azure Blob 存储服务
执行常见方案。示例是用 C\# 编写的并使用了 Azure .NET 存储客户端库。涉及的方案包括
上载、列出、下载和删除 Blob。有关 Blob 的详细信息，请参阅[后续步骤][]部分。

> [WACOM.NOTE] 本指南适用于 Azure .NET 存储客户端库 2.x 及更高版本。建议使用的版本是 Storage Client Library 4.x，可通过 [NuGet] 获取(https://www.nuget.org/packages/WindowsAzure.Storage/) or as part of the [Azure SDK for .NET](/zh-cn/downloads/). See [How to: Programmatically access Blob storage][] below for more details on obtaining the Storage Client Library.

##目录

-   [什么是 Blob 存储][]
-   [概念][]
-   [创建 Azure 存储帐户][]
-   [设置存储连接字符串][]
-   [如何：以编程方式访问 Blob 存储][]
-   [如何：创建容器][]
-   [如何：将 Blob 上载到容器中][]
-   [如何：列出容器中的 Blob][]
-   [如何：下载 Blob][]
-   [如何：删除 Blob][]
-   [如何：以异步方式列出页中的 Blob][]
-   [后续步骤][]

[WACOM.INCLUDE [howto-blob-storage](../includes/howto-blob-storage.md)]

##<a name="create-account"></a>创建 Azure 存储帐户
[WACOM.INCLUDE [create-storage-account](../includes/create-storage-account.md)]

##<a name="setup-connection-string"></a>设置存储连接字符串

[WACOM.INCLUDE [storage-configure-connection-string](../includes/storage-configure-connection-string.md)]

## <a name="configure-access"> </a>如何：以编程方式访问 Blob 存储

###获得程序集
我们建议你使用 NuGet 获取  `Microsoft.WindowsAzure.Storage.dll` 程序集。在"解决方案资源管理器"中，右键单击你的项目并选择"管理 NuGet 包"。在线搜索"MicrosoftAzure.Storage"，然后单击"安装"以安装 Azure 存储包和依赖项。

Azure SDK for .NET 中也包括了 `Microsoft.WindowsAzure.Storage.dll`，可从 <a href="/zh-cn/downloads/?sdk=net">.NET 开发人员中心</a>下载该版本。程序集安装在 `%Program Files%\Microsoft SDKs\Windows Azure\.NET SDK\<sdk-version>\ref\` 目录中。

###命名空间声明
在你希望在其中以编程方式访问 Azure 存储空间的任何 C# 文件中，将以下命名空间声明添加到文件的顶部：

    using Microsoft.WindowsAzure.Storage;
    using Microsoft.WindowsAzure.Storage.Auth;
	using Microsoft.WindowsAzure.Storage.Blob;

确保你引用  `Microsoft.WindowsAzure.Storage.dll` 程序集。

###检索连接字符串
可以使用 CloudStorageAccount 类型来表示您的存储帐户信息。如果你使用了 
Azure 项目模板并且/或者引用了 
Microsoft.WindowsAzure.CloudConfigurationManager，则可以使用 **CloudConfigurationManager** 类型从Azure 服务配置中检索你的存储连接字符串和存储帐户信息：

    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

如果你要创建应用程序而不引用 Microsoft.WindowsAzure.CloudConfigurationManager，并且你的连接字符串位于前面显示的  `web.config` 或  `app.config` 中，那么你可以使用 **ConfigurationManager** 来检索连接字符串。你需要将对 System.Configuration.dll 的引用添加到你的项目中，并为其添加另一个命名空间声明：

	using System.Configuration;
	...
	CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
		ConfigurationManager.ConnectionStrings["StorageConnectionString"].ConnectionString);

你可以使用 CloudBlobClient 类型来检索表示存储在 Blob 存储服务中的容器和 Blob 的对象。以下代码使用我们在上面检索到的存储帐户对象创建 CloudBlobClient 对象：

    CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();

###ODataLib 依赖项
.NET 存储客户端库中的 ODataLib 依赖项可通过在 NuGet （而非 WCF 数据服务）上获得的 ODataLib（5.0.2 版）包来解析。ODataLib 库可直接下载或者通过 NuGet 由代码项目引用。特定的 ODataLib 包为 [OData]、[Edm] 和 [Spatial]。

## <a name="create-container"> </a>如何：创建容器

Azure 存储空间中的每个 Blob 必须驻留在一个容器中。此示例演示如何创建一个容器（如果该容器不存在：

    // Retrieve storage account from connection string.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the blob client.
    CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();

    // Retrieve a reference to a container. 
    CloudBlobContainer container = blobClient.GetContainerReference("mycontainer");

    // Create the container if it doesn't already exist.
    container.CreateIfNotExists();

默认情况下，新容器是专用容器，因此您必须指定存储访问密钥才能从该容器下载 Blob。如果您要让容器中的文件可供所有人使用，则可以使用以下代码将容器设置为公共容器：

    container.SetPermissions(
        new BlobContainerPermissions { PublicAccess = 
 	    BlobContainerPublicAccessType.Blob }); 

Internet 中的所有人都可以查看公共容器中的 Blob，但是，仅在您具有相应的访问密钥时，才能修改或删除它们。

## <a name="upload-blob"> </a>如何：将 Blob 上载到容器中

Azure Blob 存储支持块 Blob 和页 Blob。大多数情况下，推荐使用块 Blob。

若要将文件上载到块 Blob，请获取容器引用，并使用它获取块 Blob 引用。获取 Blob 引用后，可以通过调用 UploadFromStream() 方法，将任意数据流上载到该 Blob。如果之前不存在 Blob，此操作将创建一个；如果存在 Blob，此操作将覆盖它。下面的示例演示了如何将 Blob 上载到容器中，并假定已创建容器。

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

##<a name="list-blob"> </a>如何：列出容器中的 Blob

若要列出容器中的 Blob，首先需要获取容器引用。然后，您可以使用容器的 ListBlobs 方法来检索其中的 Blob 和/或目录。若要访问返回的 **IListBlobItem** 的丰富属性和方法，你必须将其转换为 **CloudBlockBlob**、 
**CloudPageBlob** 或 **CloudBlobDirectory** 对象。如果类型未知，您可以使用类型检查来确定要将其转换为哪种类型。以下代码演示了如何检索和输出  `photos` 容器中每项的 URI：

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

如上所示，还可将 Blob 服务视为容器中的目录。这是为了让您能够以更类似于文件夹的结构来组织 Blob。例如，考虑名为  `photos` 的容器中包含的下面一组块 Blob：

	photo1.jpg
	2010/architecture/description.txt
	2010/architecture/photo3.jpg
	2010/architecture/photo4.jpg
	2011/architecture/photo5.jpg
	2011/architecture/photo6.jpg
	2011/architecture/description.txt
	2011/photo7.jpg

当你对  'photos' 容器调用 ListBlobs 时（如上面的示例所示），返回的集合将包含 CloudBlobDirectory 和 CloudBlockBlob 对象，分别表示最高层中所含的目录和 Blob。下面是生成的输出：

	Directory: https://<accountname>.blob.core.chinacloudapi.cn/photos/2010/
	Directory: https://<accountname>.blob.core.chinacloudapi.cn/photos/2011/
	Block blob of length 505623: https://<accountname>.blob.core.chinacloudapi.cn/photos/photo1.jpg


另外，也可以将 ListBlobs 方法的 UseFlatBlobListing 参数设置为 true。这将导致每个 Blob 将作为 **CloudBlockBlob** 返回，而无论目录如何。下面是对 ListBlobs 的调用：

    // Loop over items within the container and output the length and URI.
	foreach (IListBlobItem item in container.ListBlobs(null, true))
	{
	   ...
	}

下面是结果：

	Block blob of length 4: https://<accountname>.blob.core.chinacloudapi.cn/photos/2010/architecture/description.txt
	Block blob of length 314618: https://<accountname>.blob.core.chinacloudapi.cn/photos/2010/architecture/photo3.jpg
	Block blob of length 522713: https://<accountname>.blob.core.chinacloudapi.cn/photos/2010/architecture/photo4.jpg
	Block blob of length 4: https://<accountname>.blob.core.chinacloudapi.cn/photos/2011/architecture/description.txt
	Block blob of length 419048: https://<accountname>.blob.core.chinacloudapi.cn/photos/2011/architecture/photo5.jpg
	Block blob of length 506388: https://<accountname>.blob.core.chinacloudapi.cn/photos/2011/architecture/photo6.jpg
	Block blob of length 399751: https://<accountname>.blob.core.chinacloudapi.cn/photos/2011/photo7.jpg
	Block blob of length 505623: https://<accountname>.blob.core.chinacloudapi.cn/photos/photo1.jpg

有关更多信息，请参见 [CloudBlobContainer.ListBlobs][]。

## <a name="download-blobs"> </a>如何：下载 Blob

若要下载 Blob，请首先检索 Blob 引用，然后调用 DownloadToStream 方法。以下示例使用 DownloadToStream 方法将 Blob 内容传输到一个流对象，然后你可以将该对象保存到本地文件。

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

也可以使用 DownloadToStream 方法以文本字符串形式下载 Blob 的内容。

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

##<a name="delete-blobs"> </a>如何：删除 Blob

若要删除 Blob，首先要获取 Blob 引用，然后对其调用
**Delete** 方法。

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


##<a name="list-blobs-async"> </a>如何：以异步方式列出页中的 Blob

如果要列出大量 Blob，或需要控制一个列表操作中返回的结果数，则可以结果页的方式列出 Blob。此示例显示如何以页面形式异步返回结果，这样就不会在等待返回大型结果集时阻止操作的执行。

此示例演示平面 Blob 列表，但你也可以执行分层列表，只需将 **ListBlobsSegmentedAsync** 方法的  `useFlatBlobListing` 参数设置为  `false` 即可。

由于示例方法调用异步方法，因此必须以  `async` 关键字开头，且必须返回 **Task** 对象。为 **ListBlobsSegmentedAsync** 方法指定的 await 关键字将挂起示例方法的执行，直至列表任务完成。

    async public static Task ListBlobsSegmentedInFlatListing()
    {
        // Retrieve storage account from connection string.
        CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
            CloudConfigurationManager.GetSetting("StorageConnectionString"));

        // Create the blob client.
        CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();

        // Retrieve reference to a previously created container.
        CloudBlobContainer container = blobClient.GetContainerReference("myblobs");

        //List blobs in pages.
        Console.WriteLine("List blobs in pages:");

        //List blobs with a paging size of 10, for the purposes of the example. 
		//The first call does not include the continuation token.
        BlobResultSegment resultSegment = await container.ListBlobsSegmentedAsync(
                "", true, BlobListingDetails.All, 10, null, null, null);

        //Enumerate the result segment returned.
        int i = 0;
        if (resultSegment.Results.Count<IListBlobItem>() > 0) { Console.WriteLine("Page {0}:", ++i); }
        foreach (var blobItem in resultSegment.Results)
        {
            Console.WriteLine("\t{0}", blobItem.StorageUri.PrimaryUri);
        }
        Console.WriteLine();

        //Get the continuation token, if there are additional pages of results.
        BlobContinuationToken continuationToken = resultSegment.ContinuationToken;

        //Check whether there are more results and list them in pages of 10 while a continuation token is returned.
        while (continuationToken != null)
        {
            //This overload allows control of the page size. 
			//You can return all remaining results by passing null for the maxResults parameter, 
            //or by calling a different overload.
            resultSegment = await container.ListBlobsSegmentedAsync(
					"", true, BlobListingDetails.All, 10, continuationToken, null, null);
            if (resultSegment.Results.Count<IListBlobItem>() > 0) { Console.WriteLine("Page {0}:", ++i); }
            foreach (var blobItem in resultSegment.Results)
            {
                Console.WriteLine("\t{0}", blobItem.StorageUri.PrimaryUri);
            }
            Console.WriteLine();

            //Get the next continuation token.
            continuationToken = resultSegment.ContinuationToken;
        }
    }

## <a name="next-steps"></a>后续步骤

现在，你已了解有关 Blob 存储的基础知识，可单击下面的链接来了解如何执行更复杂的存储任务。
<ul>
<li>查看 Blob 服务参考文档，了解有关可用 API 的完整详情：
  <ul>
    <li><a href="http://msdn.microsoft.com/zh-cn/library/azure/dn261237.aspx">.NET 存储客户端库参考</a>
    </li>
    <li><a href="http://msdn.microsoft.com/zh-cn/library/azure/dd179355">REST API 参考</a></li>
  </ul>
</li>
<li>在以下位置了解使用 Azure 存储空间能够执行的更高级任务：<a href="http://msdn.microsoft.com/zh-cn/library/azure/gg433040.aspx">在 Azure 中存储和访问数据</a>。</li>
<li>了解如何使用 <a href="../ Websites-dotnet-webjobs-sdk/">Azure WebJobs SDK 简化你编写的用于 Azure 存储空间的代码。</li>
<li>查看更多功能指南，以了解在 Azure 中存储数据的其他方式。
  <ul>
    <li>使用<a href="/zh-cn/documentation/articles/storage-dotnet-how-to-use-tables/">表存储</a>来存储结构化数据。</li>
    <li>使用<a href="/zh-cn/documentation/articles/storage-dotnet-how-to-use-queues/">队列存储</a>来存储非结构化数据。</li>
    <li>使用 <a href="/zh-cn/documentation/articles/sql-database-dotnet-how-to-use/">SQL Database</a> 来存储关系数据。</li>
  </ul>
</li>
</ul>

  [后续步骤]: #next-steps
  [什么是 Blob 存储]: #what-is
  [概念]: #concepts
  [创建 Azure 存储帐户]: #create-account
  [设置存储连接字符串]: #setup-connection-string
  [如何：以编程方式访问 Blob 存储]: #configure-access
  [如何：创建容器]: #create-container
  [如何：将 Blob 上载到容器]: #upload-blob
  [如何：列出容器中的 Blob]: #list-blob
  [如何：下载 Blob]: #download-blobs
  [如何：删除 Blob]: #delete-blobs
  [如何：以异步方式列出页中的 Blob]: #list-blobs-async
  [Blob5]: ./media/storage-dotnet-how-to-use-blobs/blob5.png
  [Blob6]: ./media/storage-dotnet-how-to-use-blobs/blob6.png
  [Blob7]: ./media/storage-dotnet-how-to-use-blobs/blob7.png
  [Blob8]: ./media/storage-dotnet-how-to-use-blobs/blob8.png
  [Blob9]: ./media/storage-dotnet-how-to-use-blobs/blob9.png
  
  [在 Azure 中存储和访问数据]: http://msdn.microsoft.com/zh-cn/library/azure/gg433040.aspx
  [Azure 存储团队博客]: http://blogs.msdn.com/b/windowsazurestorage/
  [配置连接字符串]: http://msdn.microsoft.com/zh-cn/library/azure/ee758697.aspx
  [.NET 客户端库引用]: http://msdn.microsoft.com/zh-cn/library/azure/dn261237.aspx
  [REST API 参考]: http://msdn.microsoft.com/zh-cn/library/azure/dd179355
  [OData]: http://nuget.org/packages/Microsoft.Data.OData/5.0.2
  [Edm]: http://nuget.org/packages/Microsoft.Data.Edm/5.0.2
  [Spatial]: http://nuget.org/packages/System.Spatial/5.0.2
<!--HONumber=41-->
