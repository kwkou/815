
<properties 
  pageTitle="设置和检索 Blob 存储的属性与元数据 | Windows Azure" 
  description="了解如何设置和检索 Azure 存储容器及 Blob 的属性与元数据。" 
  services="storage" 
  documentationCenter="" 
  authors="tamram" 
  manager="adinah" 
  editor=""/>

<tags 
  ms.service="storage"  
  ms.date="04/21/2015" 
  wacn.date="06/26/2015"/>


# 设置和检索属性与元数据 #

## 概述

除本身包含的数据外，容器和 Blob 还支持两种形式的关联数据：

*   **系统属性**。 系统属性存在于每个容器或 Blob 资源上。其中一些属性是可以读取或设置的，而另一些属性是只读的。事实上，有些系统属性与 Azure 托管库为你保留的某些标准 HTTP 标头对应。  

*   **用户定义的元数据**。 用户定义的元数据是在给定资源上以名称/值对的形式指定的元数据。你可以使用元数据来存储容器或 Blob 的其他值；这些值仅用于你个人目的，不会影响容器和 Blob 的行为方式。

> [AZURE.IMPORTANT]检索资源的属性和元数据值的过程分为两步。必须先在你的 [CloudBlobContainer](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.windowsazure.storage.blob.cloudblobcontainer.aspx)、[CloudBlockBlob](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.windowsazure.storage.blob.cloudblockblob.aspx) 或 [CloudPageBlob](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.windowsazure.storage.blob.cloudpageblob.aspx) 对象上显式获取这些值，然后才能读取它们。若要以同步方式获取属性和元数据，请对容器或 Blob 调用 **FetchAttributes**；若要以异步方式获取它们，请调用 **FetchAttributesAsync**。

## 设置和检索属性

容器只有只读属性，而 Blob 同时具有只读属性和读写属性。若要在 Blob 上设置属性，请指定属性值，然后调用 [SetProperties](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.windowsazure.storage.blob.cloudblockblob.setproperties.aspx) 方法或 [SetProperties](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.windowsazure.storage.blob.cloudpageblob.setproperties.aspx) 方法。

若要读取容器或 Blob 的属性，请调用 **FetchAttributes** 方法，然后检索属性值。

以下代码示例创建容器和 Blob 并将属性值写入到控制台窗口。此示例使用了存储模拟器，因此必须运行模拟的存储服务，这样代码才能工作：

	// Use the storage emulator account.
	CloudStorageAccount storageAccount = CloudStorageAccount.DevelopmentStorageAccount;

	// As an alternative, you can create the credentials from the account name and key.
	// string accountName = "myaccount";
	// string accountKey = "SzlFqgzqhfkj594cFoveYqCuvo8v9EESAnOLcTBeBIo31p16rJJRZx/5vU/oY3ZsK/VdFNaVpm6G8YSD2K48Nw==";
	// StorageCredentials credentials = new StorageCredentials(accountName, accountKey);
	// CloudStorageAccount storageAccount = new CloudStorageAccount(credentials, true);

	CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();

	// Retrieve a reference to a container.
	CloudBlobContainer container = blobClient.GetContainerReference(<span style="color:#A31515;">"mycontainer");

	// Create the container if it does not already exist.
	container.CreateIfNotExists();

	// Fetch container properties and write out their values.
	container.FetchAttributes();
	Console.WriteLine(<span style="color:#A31515;">"Properties for container " + container.Uri + <span style="color:#A31515;">":");
	Console.WriteLine(<span style="color:#A31515;">"LastModifiedUTC: " + container.Properties.LastModified);
	Console.WriteLine(<span style="color:#A31515;">"ETag: " + container.Properties.ETag);
	Console.WriteLine();

	// Create a blob.
	CloudBlockBlob blob = container.GetBlockBlobReference(<span style="color:#A31515;">"myblob.txt");

	// Create or overwrite the "myblob.txt" blob with contents from a local file.
	<span style="color:Blue;">using (<span style="color:Blue;">var fileStream = System.IO.File.OpenRead(<span style="color:#A31515;">@"c:\test\myblob.txt"))
	{
	   blob.UploadFromStream(fileStream);
	}

	// Fetch container properties and write out their values.
	container.FetchAttributes();
	Console.WriteLine(<span style="color:#A31515;">"Properties for container " + container.Uri + <span style="color:#A31515;">":");
	Console.WriteLine(<span style="color:#A31515;">"LastModifiedUTC: " + container.Properties.LastModified);
	Console.WriteLine(<span style="color:#A31515;">"ETag: " + container.Properties.ETag);
	Console.WriteLine();

	// Create a blob.
	Uri blobUri =<span style="color:Blue;">new UriBuilder(containerUri.AbsoluteUri + <span style="color:#A31515;">"/ablob.txt").Uri;
	CloudPageBlob blob = <span style="color:Blue;">new CloudPageBlob(blobUri, credentials);
	blob.Create(1024);


	// Set the CacheControl property.
	blob.Properties.CacheControl = <span style="color:#A31515;">"public, max-age=31536000";
	blob.SetProperties();

	// Fetch blob attributes.
	blob.FetchAttributes();

	Console.WriteLine(<span style="color:#A31515;">"Read-only properties for blob " + blob.Uri + <span style="color:#A31515;">":");
	Console.WriteLine(<span style="color:#A31515;">"BlobType: " + blob.Properties.BlobType);
	Console.WriteLine(<span style="color:#A31515;">"ETag: " + blob.Properties.ETag);
	Console.WriteLine(<span style="color:#A31515;">"LastModifiedUtc: " + blob.Properties.LastModified);
	Console.WriteLine(<span style="color:#A31515;">"Length: " + blob.Properties.Length);
	Console.WriteLine();

	Console.WriteLine(<span style="color:#A31515;">"Read-write properties for blob " + blob.Uri + <span style="color:#A31515;">":");
	Console.WriteLine(<span style="color:#A31515;">"CacheControl: " +
	   (blob.Properties.CacheControl == <span style="color:Blue;">null ? <span style="color:#A31515;">"Not set" : blob.Properties.CacheControl));
	Console.WriteLine(<span style="color:#A31515;">"ContentEncoding: " +
	   (blob.Properties.ContentEncoding == <span style="color:Blue;">null ? <span style="color:#A31515;">"Not set" : blob.Properties.ContentEncoding));
	Console.WriteLine(<span style="color:#A31515;">"ContentLanguage: " +
	   (blob.Properties.ContentLanguage == <span style="color:Blue;">null ? <span style="color:#A31515;">"Not set" : blob.Properties.ContentLanguage));
	Console.WriteLine(<span style="color:#A31515;">"ContentMD5: " +
	   (blob.Properties.ContentMD5 == <span style="color:Blue;">null ? <span style="color:#A31515;">"Not set" : blob.Properties.ContentMD5));
	Console.WriteLine(<span style="color:#A31515;">"ContentType: " +
	   (blob.Properties.ContentType == <span style="color:Blue;">null ? <span style="color:#A31515;">"Not set" : blob.Properties.ContentType));

	// Clean up.
	blob.DeleteIfExists();
	container.Delete();
## 设置和检索元数据

你可以在 Blob 或容器资源上指定元数据作为一个或多个名称/值对。若要设置元数据，请将名称/值对添加到资源上的 **Metadata** 集合，然后调用 **SetMetadata** 方法以将值保存到服务。

> [AZURE.NOTE]：元数据的名称必须符合 C# 标识符命名约定。

若要检索元数据，请对 Blob 或容器调用 **FetchAttributes** 方法以填充 **Metadata** 集合，然后读取值。

以下代码示例创建新容器并为其设置元数据，然后读回元数据值：


	// Account name and key.  Modify for your account.
	<span style="color:Blue;">string accountName = <span style="color:#A31515;">"myaccount";
	<span style="color:Blue;">string accountKey = <span style="color:#A31515;">"SzlFqgzqhfkj594cFoveYqCuvo8v9EESAnOLcTBeBIo31p16rJJRZx/5vU/oY3ZsK/VdFNaVpm6G8YSD2K48Nw==";

	// Get a reference to the storage account and client with authentication credentials.
	StorageCredentials credentials = <span style="color:Blue;">new StorageCredentials(accountName, accountKey);
	CloudStorageAccount storageAccount = <span style="color:Blue;">new CloudStorageAccount(credentials, <span style="color:Blue;">true);
	CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();

	// Retrieve a reference to a container.
	CloudBlobContainer container = blobClient.GetContainerReference(<span style="color:#A31515;">"mycontainer");

	// Create the container if it does not already exist.
	container.CreateIfNotExists();

	// Set metadata for the container.
	container.Metadata[<span style="color:#A31515;">"category"] = <span style="color:#A31515;">"images";
	container.Metadata[<span style="color:#A31515;">"owner"] = <span style="color:#A31515;">"azure";
	container.SetMetadata();

	// Get container metadata.
	container.FetchAttributes();
	<span style="color:Blue;">foreach (<span style="color:Blue;">string key <span style="color:Blue;">in container.Metadata.Keys)
	{
	   Console.WriteLine(<span style="color:#A31515;">"Metadata key: " + key);
	   Console.WriteLine(<span style="color:#A31515;">"Metadata value: " + container.Metadata[key]);
	}

	//Clean up.
	container.Delete();

## 另请参阅  

- [Azure 存储客户端库参考](http://msdn.microsoft.com/zh-cn/library/azure/wa_storage_30_reference_home.aspx)
- [适用于 .NET 的 Blob 存储入门](storage-dotnet-how-to-use-blobs)  

<!---HONumber=61-->