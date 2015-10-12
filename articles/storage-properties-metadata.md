
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
  ms.date="08/04/2015" 
  wacn.date="09/18/2015"/>


# 设置和检索属性与元数据 #

## 概述

Azure 存储中的对象支持系统属性和用户定义的元数据，除了该数据以外，它们还包含：

*   **系统属性**。 系统属性存在于每个存储资源上。其中一些属性是可以读取或设置的，而另一些属性是只读的。事实上，有些系统属性与某些标准 HTTP 标头对应。Azure 存储客户端库为您维护这些内容。  

*   **用户定义的元数据**。 用户定义的元数据是在给定资源上以名称/值对的形式指定的元数据。您可以使用元数据来存储存储资源其他值；这些值仅用于您个人目的，不会影响资源的行为方式。

## 设置和检索属性

检索资源的属性和元数据值的过程分为两步。在可以读取这些值之前，您必须通过调用 **FetchAttributes** 或 **FetchAttributesAsync** 方法显式获取它们。

> [AZURE.IMPORTANT] 不会填充存储资源的属性和元数据值，除非调用 **FetchAttributes** 方法之一。

若要在 Blob 上设置属性，请指定属性值，然后调用 **SetProperties** 或 **SetPropertiesAsync** 方法。

以下代码示例创建容器并将属性值写入到控制台窗口：

    //Parse the connection string for the storage account.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        Microsoft.Azure.CloudConfigurationManager.GetSetting("StorageConnectionString"));
	
	//Create the service client object for credentialed access to the Blob service.
    CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();

    // Retrieve a reference to a container. 
    CloudBlobContainer container = blobClient.GetContainerReference("mycontainer");

	// Create the container if it does not already exist.
	container.CreateIfNotExists();

    // Fetch container properties and write out their values.
    container.FetchAttributes();
    Console.WriteLine("Properties for container {0}", container.StorageUri.PrimaryUri.ToString());
    Console.WriteLine("LastModifiedUTC: {0}", container.Properties.LastModified.ToString());
    Console.WriteLine("ETag: {0}", container.Properties.ETag);
    Console.WriteLine();

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
- [适用于 .NET 的 Blob 存储入门](/documentation/articles/storage-dotnet-how-to-use-blobs)  

<!---HONumber=70-->