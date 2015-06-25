#####创建容器
正如文件位于文件夹中一样，存储 Blob 位于容器中。您可以使用 **CloudBlobClient** 对象引用现有容器，也可以调用 CreateCloudBlobClient() 方法创建一个新容器。

以下代码将演示如何新建一个 Blob 存储容器。该代码将首先创建一个 **BlobClient** 对象（例如创建一个存储容器），以便您可以访问该对象的功能。然后，该代码将尝试引用名为"mycontainer"的存储容器。如果找不到该名称的容器，它将创建一个。

	// Create a blob client.
	CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();

	// Get a reference to a container named "mycontainer."
	CloudBlobContainer container = blobClient.GetContainerReference("mycontainer");

	// If "mycontainer" doesn't exist, create it.
	container.CreateIfNotExists();

默认情况下，新容器是专用容器，因此您必须指定存储访问密钥才能从该容器下载 Blob。如果您要让容器中的文件可供所有人使用，则可以使用以下代码将容器设置为公共容器。

	container.SetPermissions(
    	new BlobContainerPermissions { PublicAccess = 
	    BlobContainerPublicAccessType.Blob }); 


**注意：**在接下来的部分中，将在代码的前面使用此代码块。

#####将 Blob 上载到容器中
若要将 Blob 文件上载到容器中，请获取容器引用，并使用它来获取 Blob 引用。获取 Blob 引用后，可以通过调用 **UploadFromStream()** 方法，将任意数据流上载到该 Blob。此操作将创建 Blob（如果该 Blob 不存在），或者覆盖它（如果该 Blob 存在）。下面的示例演示了如何将 Blob 上载到容器中，并假定已创建容器。

	// Get a reference to a blob named "myblob".
	CloudBlockBlob blockBlob = container.GetBlockBlobReference("myblob");
	
	// Create or overwrite the "myblob" blob with the contents of a local file
	// named "myfile".
	using (var fileStream = System.IO.File.OpenRead(@"path\myfile"))
	{
    	blockBlob.UploadFromStream(fileStream);
	}

#####列出容器中的 Blob
若要列出容器中的 Blob，首先需要获取容器引用。然后，您可以使用容器的 **ListBlobs()** 方法来检索其中的 Blob 和/或目录。若要访问返回的 **IListBlobItem** 的丰富属性和方法集，您必须将它转换到 **CloudBlockBlob**、**CloudPageBlob** 或 **CloudBlobDirectory** 对象。如果 Blob 类型未知，您可以使用类型检查来确定要将其转换为哪种类型。以下代码演示了如何检索和输出名为"photos"的容器中每项的 URI。

	// Get a reference to a previously created container.
	CloudBlobContainer container = blobClient.GetContainerReference("photos");

	// Loop through items in the container and output the length and URI for each 
	// item.
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

还有其他方法来列出 Blob 容器内容。有关详细信息，请参阅[如何通过 .NET 使用 Blob 存储](/zh-cn/documentation/articles/storage-dotnet-how-to-use-blobs/#list-blob)。

#####下载 Blob
若要下载 Blob，首先请获取对 Blob 的引用，然后再调用 DownloadToStream() 方法。以下示例使用 DownloadToStream() 方法将 Blob 内容传输到稍后可以另存为本地文件的流对象。

	// Get a reference to a blob named "photo1.jpg".
	CloudBlockBlob blockBlob = container.GetBlockBlobReference("photo1.jpg");

	// Save the blob contents to a file named "myfile".
	using (var fileStream = System.IO.File.OpenWrite(@"path\myfile"))
	{
    	blockBlob.DownloadToStream(fileStream);
	}

还有其他方法将 Blob 另存为文件。有关详细信息，请参阅[如何通过 .NET 使用 Blob 存储](/zh-cn/documentation/articles/storage-dotnet-how-to-use-blobs/#download-blobs)。

#####删除 Blob
若要删除 Blob，首先请获取对 Blob 的引用，然后再对其调用 Delete() 方法。

	// Get a reference to a blob named "myblob.txt".
	CloudBlockBlob blockBlob = container.GetBlockBlobReference("myblob.txt");

	// Delete the blob.
	blockBlob.Delete();

[了解有关 Azure 存储的详细信息](/zh-cn/documentation/services/storage)
另请参阅[在服务器资源管理器中浏览存储资源](http://msdn.microsoft.com/zh-cn/library/azure/ff683677.aspx).<!--HONumber=41-->
