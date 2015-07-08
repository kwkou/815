<properties 
    pageTitle="使用 .NET 对 Azure 存储帐户的访问进行身份验证 | Windows Azure" 
    description="使用 .NET 客户端库对 Azure 存储帐户的访问进行身份验证。" 
    services="storage" 
    documentationCenter=".net" 
    authors="tamram" 
    manager="adinah" 
    editor=""/>

<tags 
	wacn.date="05/15/2015"
    ms.service="storage" 
    ms.workload="storage" 
    ms.tgt_pltfrm="na" 
    ms.devlang="dotnet" 
    ms.topic="article" 
    ms.date="04/15/2015" 
    ms.author="tamram"/>

# 使用 .NET 对 Azure 存储帐户的访问进行身份验证

## 概述

你对 Azure 存储服务发出的任何请求都必须进行身份验证，除非它是针对公用容器或其 blob 的匿名请求。Azure 客户端库可帮助你简化这一身份验证过程。你可以使用两种方法对存储服务请求进行身份验证：

- 对 Blob、队列、表和文件服务中的资源使用共享密钥或共享密钥 Lite 身份验证方案。这些身份验证方案使用以 SHA-256 算法计算并编码为 Base64 的 HMAC。HMAC 是通过一组与请求相关的字段构造的。有关协议的详细信息，请参阅 [Azure 存储服务的身份验证](https://msdn.microsoft.com/zh-CN/library/azure/dd179428.aspx)。

- 创建共享访问签名。共享访问签名包含以安全方式在 URI 中进行身份验证所需的凭据，以及要访问的资源的地址。由于共享访问签名包括在 URI 上进行身份验证所需的所有数据，因此可用于授予对 Blob、队列或表服务中的资源的受控访问权限，并可以在你的代码之外分发。有关协议的详细信息，请参阅[使用共享访问签名委托访问权限](https://msdn.microsoft.com/zh-CN/library/azure/ee395415.aspx)。

## 身份验证访问选项

Azure .NET 管理的库提供一些重要的类，用于对你的存储帐户的访问进行身份验证：

- [CloudStorageAccount](https://msdn.microsoft.com/zh-CN/library/azure/microsoft.windowsazure.storage.cloudstorageaccount.aspx) 类表示你的 Azure 存储帐户。
- [StorageCredentials](https://msdn.microsoft.com/zh-CN/library/azure/microsoft.windowsazure.storage.auth.storagecredentials.aspx) 类存储两种不同类型的凭据，这些凭据可用于对请求进行身份验证：存储帐户名称和访问密钥，它们可用于通过共享密钥和共享密钥 Lite 身份验证方案或共享访问签名对请求进行身份验证。 
- [CloudBlobClient](https://msdn.microsoft.com/zh-CN/library/azure/microsoft.windowsazure.storage.blob.cloudblobclient.aspx)、[CloudQueueClient](https://msdn.microsoft.com/zh-CN/library/azure/microsoft.windowsazure.storage.queue.cloudqueueclient.aspx) 和 [CloudTableClient](https://msdn.microsoft.com/zh-CN/library/azure/microsoft.windowsazure.storage.table.cloudtableclient.aspx) 类为 Blob 服务、队列服务和表服务分别提供资源层次结构的输入点。换而言之，若要使用容器和 Blob，你将创建 **CloudBlobClient** 对象；若要使用队列和消息，你将创建 **CloudQueueClient**；若要使用表和实体，你将创建 **CloudTableClient**。客户端对象可通过提供服务终结点和一组凭据直接创建，也可通过调用 [CreateCloudBlobClient](https://msdn.microsoft.com/zh-CN/library/azure/microsoft.windowsazure.storage.cloudstorageaccount.createcloudblobclient.aspx)、[CreateCloudQueueClient](https://msdn.microsoft.com/zh-CN/library/azure/microsoft.windowsazure.storage.cloudstorageaccount.createcloudqueueclient.aspx) 或 [CreateCloudTableClient](https://msdn.microsoft.com/zh-CN/library/azure/microsoft.windowsazure.storage.cloudstorageaccount.createcloudtableclient.aspx) 方法之一从 **CloudStorageAccount** 对象创建。这些方法允许你从使用单组凭据定义的 **CloudStorageAccount** 对象为一个或多个服务返回客户端对象。
> [AZURE.NOTE] 若要以匿名方式访问容器或 blob，你不需要创建 **CloudStorageAccount** 或客户端对象。匿名访问不需要身份验证，因此，你可以直接访问资源。只有某些读取操作（使用 HTTP GET）才是通过匿名访问获得支持的。

### 访问存储模拟器

此属性在你编写仅使用存储模拟器的代码时很有用，但是，如果你要在完成测试存储模拟器后将代码指向 Azure 存储帐户，则你可能需要使用连接字符串。你可以快速修改连接字符串以在存储帐户之间切换，而不必修改你的代码；有关详细信息，请参阅[配置连接字符串](https://msdn.microsoft.com/zh-CN/library/azure/ee758697.aspx)。

### 使用默认终结点访问 Azure 存储帐户

若要从你的代码访问 Azure 存储帐户，最简单的方法是创建新的 **CloudStorageAccount** 对象，方法是以 **StorageCredentials** 对象的形式提供存储帐户名称和访问密钥，同时指示是否使用 HTTP 或 HTTPS 访问该帐户。通过此 **CloudStorageAccount** 或其派生对象之一发出的请求将使用这些凭据进行身份验证。

此 **CloudStorageAccount** 对象将默认终结点用于存储服务。默认服务终结点是  `myaccount.blob.core.chinacloudapi.cn`、 `myaccount.queue.core.chinacloudapi.cn` 和  `myaccount.table.core.chinacloudapi.cn`，其中， `myaccount` 是存储帐户的名称。

以下代码示例显示如何使用对帐户名称和访问密钥的引用创建新的 **CloudStorageAccount** 对象。然后，创建新的 **CloudBlobClient** 对象，再创建新的容器：

	//Account name and key. Modify for your account.
	string accountName = "myaccount";
	string accountKey = "nYS0gln9fA7bvY+rwu2iWADyzSNITGkhM88J8HUoyofpK7C8fHcZc2kIZp6cKgYJUM74rHI82L50Iau4+9hPjQ==";
	
	//Get a reference to the storage account, with authentication credentials.
	CloudStorageAccount storageAccount = new CloudStorageAccount(new StorageCredentials(accountName, accountKey), true);
	
	//Create a new client object.
	CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();
	
	// Retrieve a reference to a container. 
	CloudBlobContainer container = blobClient.GetContainerReference("mycontainer");
	
	// Create the container if it does not already exist.
	container.CreateIfNotExists();
	
	// Output container URI to debug window.
	Console.WriteLine(container.Uri);  
### 使用显式终结点访问 Azure 存储帐户

你还可以通过明确指定服务终结点访问存储帐户。这种方法适用于以下两种情况：

- 通过共享访问签名访问存储帐户时。

- 已针对存储帐户配置自定义域，并且希望使用这些自定义终结点来访问该帐户。请注意，除了自定义终结点，默认终结点仍然可用。

#### 创建并使用共享访问签名

通过共享访问签名，可以为其他客户端提供对存储帐户中的资源的受控访问权限。共享访问签名包含以下方面的信息：要访问的资源、要授予的权限、资源可用的间隔和容器访问策略（如果有）。出于身份验证的目的，采用编码为 UTF-8 的字符串到签名形式封装此信息，而后使用 HMAC-SHA256 算法构造签名。签名是客户端可用于访问资源的令牌的一部分。例如，客户端可以使用共享访问签名在存储帐户中创建或删除 blob，无法访问容器的操作仅标记为公共。有关共享访问签名的详细信息，请参阅[共享访问签名：第 1 部分：了解 SAS 模型](/documentation/articles/storage-dotnet-shared-access-signature-part-1)。

## 后续步骤

**其他资源**  
[开始使用适用于 .NET 的 Blob 存储](/documentation/articles/storage-dotnet-how-to-use-blobs)   
[开始使用适用于 .NET 的队列存储](/documentation/articles/storage-dotnet-how-to-use-queues)   
[开始使用适用于 .NET 的表存储](/documentation/articles/storage-dotnet-how-to-use-tables)
[开始使用适用于 .NET 的文件存储](/documentation/articles/storage-dotnet-how-to-use-files)

<!--HONumber=53-->