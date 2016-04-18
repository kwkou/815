
<properties 
  pageTitle="在 Azure 存储空间中设置和检索对象的属性和元数据 | Azure" 
  description="在 Azure 存储空间中存储对象的自定义元数据，并设置和检索系统属性。" 
  services="storage" 
  documentationCenter="" 
  authors="tamram" 
  manager="adinah" 
  editor=""/>

<tags 
  ms.service="storage" 
  ms.date="02/20/2016" 
  wacn.date="04/18/2016"/>


# 设置和检索属性与元数据 #

## 概述

Azure 存储中的对象支持系统属性和用户定义的元数据，除了该数据以外，它们还包含：

*   **系统属性。** 系统属性存在于每个存储资源上。其中一些属性是可以读取或设置的，而另一些属性是只读的。事实上，有些系统属性与某些标准 HTTP 标头对应。Azure 存储客户端库为您维护这些内容。  

*   **用户定义的元数据。** 用户定义的元数据是在给定资源上以名称/值对的形式指定的元数据。您可以使用元数据来存储存储资源其他值；这些值仅用于您个人目的，不会影响资源的行为方式。

检索资源的属性和元数据值的过程分为两步。在可以读取这些值之前，你必须通过调用 **FetchAttributes** 方法显式获取它们。

> [AZURE.IMPORTANT] 不会填充存储资源的属性和元数据值，除非调用 **FetchAttributes** 方法之一。

## 设置和检索属性

若要检索属性值，请对 blob 或容器调用 **FetchAttributes** 方法以填充属性，然后读取值。

若要在对象上设置属性，请指定属性值，然后调用 **SetProperties** 方法。

以下代码示例创建容器并将它的一些属性值写入到控制台窗口：

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
 
以下代码示例在容器上设置元数据。一个值是使用集合的 **Add** 方法设置的。另一个值是使用隐式键/值语法设置的。这两种方法都有效。

    public static void AddContainerMetadata(CloudBlobContainer container)
    {
        //Add some metadata to the container.
        container.Metadata.Add("docType", "textDocuments");
        container.Metadata["category"] = "guidance";

        //Set the container's metadata.
        container.SetMetadata();
    }

若要检索元数据，请对 Blob 或容器调用 **FetchAttributes** 方法以填充 **Metadata** 集合，然后读取值，如下面的示例所示。

    public static void ListContainerMetadata(CloudBlobContainer container)
    {
        //Fetch container attributes in order to populate the container's properties and metadata.
        container.FetchAttributes();

        //Enumerate the container's metadata.
        Console.WriteLine("Container metadata:");
        foreach (var metadataItem in container.Metadata)
        {
            Console.WriteLine("\tKey: {0}", metadataItem.Key);
            Console.WriteLine("\tValue: {0}", metadataItem.Value);
        }
    }

## 另请参阅  

- [适用于 .NET 的 Azure 存储客户端库参考](http://msdn.microsoft.com/zh-cn/library/azure/wa_storage_30_reference_home.aspx)
- [适用于 .NET 的 Azure 存储客户端库包](https://www.nuget.org/packages/WindowsAzure.Storage/)  

<!---HONumber=Mooncake_0411_2016-->