<properties
	pageTitle="如何通过 .NET 使用队列存储 | Windows Azure"
	description="了解如何使用 Windows Azure 队列存储创建和删除队列，以及插入、扫视、获取和删除队列消息。"
	services="storage"
	documentationCenter=".net"
	authors="tamram"
	manager="adinah"
	editor=""/>

<tags
	ms.service="storage"
	ms.date="08/04/2015"
	wacn.date="09/18/2015"/>

# 如何通过 .NET 使用队列存储

[AZURE.INCLUDE [storage-selector-queue-include](../includes/storage-selector-queue-include.md)]

## 概述

本指南将演示如何使用 Azure 队列存储服务执行常见方案。示例用 C# 代码编写，并使用了用于 .NET 的 Azure 存储空间客户端库。介绍的方案包括“插入”、“扫视”、“获取”和“删除”队列消息以及“创建”和“删除”队列。

[AZURE.INCLUDE [storage-dotnet-client-library-version-include](../includes/storage-dotnet-client-library-version-include.md)]

[AZURE.INCLUDE [storage-queue-concepts-include](../includes/storage-queue-concepts-include.md)]

[AZURE.INCLUDE [storage-create-account-include](../includes/storage-create-account-include.md)]

[AZURE.INCLUDE [storage-configure-connection-string-include](../includes/storage-configure-connection-string-include.md)]

## 以编程方式访问队列存储

[AZURE.INCLUDE [storage-dotnet-obtain-assembly](../includes/storage-dotnet-obtain-assembly.md)]

### 命名空间声明
在你希望以编程方式访问 Azure 存储的任何 C# 文件中，将以下代码命名空间声明添加到文件的顶部：

    using Microsoft.WindowsAzure.Storage;
    using Microsoft.WindowsAzure.Storage.Auth;
	using Microsoft.WindowsAzure.Storage.Queue;

确保你引用  `Microsoft.WindowsAzure.Storage.dll` 程序集。

[AZURE.INCLUDE [storage-dotnet-retrieve-conn-string](../includes/storage-dotnet-retrieve-conn-string.md)]

## 创建队列

利用 **CloudQueueClient** 对象，可以获取队列的引用对象。以下代码将创建 **CloudQueueClient** 对象。本指南中的所有代码都使用存储在 Azure 应用程序的服务配置中的存储连接字符串。还可采用其他方法创建 **CloudStorageAccount** 对象。有关详细信息，请参阅 [CloudStorageAccount][] 文档。

    // Retrieve storage account from connection string
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the queue client
    CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

使用 **queueClient** 对象获取对要使用的队列的引用。如果队列不存在，你可以创建它。

    // Retrieve a reference to a queue
    CloudQueue queue = queueClient.GetQueueReference("myqueue");

    // Create the queue if it doesn't already exist
    queue.CreateIfNotExists();

## 在队列中插入消息

若要将消息插入现有队列，请先创建一个新的 **CloudQueueMessage**。接下来，调用 **AddMessage** 方法。可从字符串（UTF-8 格式）或**字节**数组创建 **CloudQueueMessage**。以下代码将创建队列（如果该队列不存在）并插入消息“Hello, World”：

    // Retrieve storage account from connection string.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the queue client.
    CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

    // Retrieve a reference to a queue.
    CloudQueue queue = queueClient.GetQueueReference("myqueue");

    // Create the queue if it doesn't already exist.
    queue.CreateIfNotExists();

    // Create a message and add it to the queue.
    CloudQueueMessage message = new CloudQueueMessage("Hello, World");
    queue.AddMessage(message);

## 扫视下一条消息

通过调用 **PeekMessage** 方法，可以查看队列前面的消息，而不必从队列中将其删除。

    // Retrieve storage account from connection string
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the queue client
    CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

    // Retrieve a reference to a queue
    CloudQueue queue = queueClient.GetQueueReference("myqueue");

    // Peek at the next message
    CloudQueueMessage peekedMessage = queue.PeekMessage();

	// Display message.
	Console.WriteLine(peekedMessage.AsString);

## 更改已排队消息的内容

你可以更改队列中现有消息的内容。如果消息表示工作任务，则你可以使用此功能来更新该工作任务的状态。以下代码使用新内容更新队列消息，并将可见性超时设置为再延长 60 秒。这将保存与消息关联的工作的状态，并额外为客户端提供一分钟的时间来继续处理消息。可使用此方法跟踪队列消息上的多步骤工作流，即使处理步骤因硬件或软件故障而失败，也无需从头开始操作。通常，你还可以保留重试计数，如果某条消息的重试次数超过 *n*，你将删除此消息。这可避免每次处理某条消息时都触发应用程序错误。

    // Retrieve storage account from connection string.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the queue client.
    CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

    // Retrieve a reference to a queue.
    CloudQueue queue = queueClient.GetQueueReference("myqueue");

	// Get the message from the queue and update the message contents.
    CloudQueueMessage message = queue.GetMessage();
    message.SetMessageContent("Updated contents.") ;
    queue.UpdateMessage(message, 
        TimeSpan.FromSeconds(0.0),  // Make it visible immediately.
        MessageUpdateFields.Content | MessageUpdateFields.Visibility);

## 取消对下一条消息的排队

你的代码通过两个步骤来取消对队列中某条消息的排队。调用 **GetMessage** 时，你将获取队列中的下一条消息。从 **GetMessage** 返回的消息变得对从此队列读取消息的任何其他代码不可见。默认情况下，此消息将持续 30 秒不可见。若要从队列中删除消息，你还必须调用 **DeleteMessage**。此删除消息的两步过程可确保，如果你的代码因硬件或软件故障而无法处理消息，则你的代码的其他实例可以获取相同消息并重试。你的代码在处理消息后会立即调用 **DeleteMessage**。

    // Retrieve storage account from connection string
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the queue client
    CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

    // Retrieve a reference to a queue
    CloudQueue queue = queueClient.GetQueueReference("myqueue");

    // Get the next message
    CloudQueueMessage retrievedMessage = queue.GetMessage();

    //Process the message in less than 30 seconds, and then delete the message
    queue.DeleteMessage(retrievedMessage);

## 将 Async-Await 模式与公用队列存储 API 配合使用

此示例演示如何将 Async-Await 模式和公用队列存储 API 配合使用。示例调用每个给定方法的异步版本，如每个方法的 *Async* 后缀所示。使用异步方法时，async-await 模式将暂停本地执行，直到调用完成。此行为允许当前的线程执行其他工作，这有助于避免性能瓶颈并提高应用程序的整体响应能力。有关在 .NET 中使用 Async-Await 模式的详细信息，请参阅 [Async 和 Await（C# 和 Visual Basic）](https://msdn.microsoft.com/zh-cn/library/hh191443.aspx)

    // Create the queue if it doesn't already exist
    if(await queue.CreateIfNotExistsAsync())
    {
        Console.WriteLine("Queue '{0}' Created", queue.Name);
    }
    else
    {
        Console.WriteLine("Queue '{0}' Exists", queue.Name);
    }

    // Create a message to put in the queue
    CloudQueueMessage cloudQueueMessage = new CloudQueueMessage("My message");

    // Async enqueue the message
    await queue.AddMessageAsync(cloudQueueMessage);
    Console.WriteLine("Message added");

    // Async dequeue the message
    CloudQueueMessage retrievedMessage = await queue.GetMessageAsync();
    Console.WriteLine("Retrieved message with content '{0}'", retrievedMessage.AsString);

    // Async delete the message
    await queue.DeleteMessageAsync(retrievedMessage);
    Console.WriteLine("Deleted message");

## 使用其他方法取消对消息的排队

你可以通过两种方式自定义队列中的消息检索。首先，你可以获取一批消息（最多 32 个）。其次，你可以设置更长或更短的不可见超时时间，从而允许你的代码使用更多或更少时间来完全处理每个消息。以下代码示例使用 **GetMessages** 方法在一次调用中获取 20 条消息。然后，它使用 **foreach** 循环处理每条消息。它还将每条消息的不可见超时时间设置为 5 分钟。请注意，5 分钟超时时间对于所有消息都是同时开始的，因此在调用 **GetMessages** 5 分钟后，尚未删除的任何消息都将再次变得可见。

    // Retrieve storage account from connection string.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the queue client.
    CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

    // Retrieve a reference to a queue.
    CloudQueue queue = queueClient.GetQueueReference("myqueue");

    foreach (CloudQueueMessage message in queue.GetMessages(20, TimeSpan.FromMinutes(5)))
    {
        // Process all messages in less than 5 minutes, deleting each message after processing.
        queue.DeleteMessage(message);
    }

## 获取队列长度

你可以获取队列中消息的估计数。使用 **FetchAttributes** 方法可请求队列服务检索队列属性，包括消息计数。**ApproximateMethodCount** 属性返回 **FetchAttributes** 方法检索到的最后一个值，而不会调用队列服务。

    // Retrieve storage account from connection string.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the queue client.
    CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

    // Retrieve a reference to a queue.
    CloudQueue queue = queueClient.GetQueueReference("myqueue");

	// Fetch the queue attributes.
	queue.FetchAttributes();

    // Retrieve the cached approximate message count.
    int? cachedMessageCount = queue.ApproximateMessageCount;

	// Display number of messages.
	Console.WriteLine("Number of messages in queue: " + cachedMessageCount);

## 删除队列

若要删除队列及其包含的所有消息，请调用队列对象的 **Delete** 方法。

    // Retrieve storage account from connection string.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the queue client.
    CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

    // Retrieve a reference to a queue.
    CloudQueue queue = queueClient.GetQueueReference("myqueue");

    // Delete the queue.
    queue.Delete();

## 后续步骤

现在，您已了解有关队列存储的基础知识，可单击下面的链接来了解更复杂的存储任务。

- 查看队列服务参考文档，了解有关可用 API 的完整详细信息：
    - [.NET 存储客户端库参考](http://msdn.microsoft.com/zh-cn/library/azure/wa_storage_30_reference_home.aspx)
    - [REST API 参考](http://msdn.microsoft.com/zh-cn/library/azure/dd179355)
- 通过查看[在 Azure 中存储和访问数据](http://msdn.microsoft.com/zh-cn/library/azure/gg433040.aspx)了解可以通过 Azure 存储空间执行的更高级任务
- 了解如何通过使用 [Azure WebJobs SDK](/documentation/articles/websites-dotnet-webjobs-sdk) 简化为使用 Azure 存储空间而写的代码。
- 查看更多功能指南，以了解在 Azure 中存储数据的其他方式。
    - 使用[表存储](/documentation/articles/storage-dotnet-how-to-use-tables)来存储结构化数据。 
    - 使用 [Blob 存储](/documentation/articles/storage-dotnet-how-to-use-blobs)来存储非结构化数据。
    - 使用 [SQL 数据库](/documentation/articles/sql-database-dotnet-how-to-use)来存储关系数据。



  [下载并安装 Azure SDK for.NET]：/develop/net/
  [.NET 客户端库引用]: http://msdn.microsoft.com/zh-cn/library/azure/wa_storage_30_reference_home.aspx
  [在 Visual Studio 中创建 Azure 项目]: http://msdn.microsoft.com/zh-cn/library/azure/ee405487.aspx
  [CloudStorageAccount]: https://msdn.microsoft.com/zh-cn/library/microsoft.windowsazure.storage.cloudstorageaccount_methods.aspx
  [在 Azure 中存储和访问数据]: http://msdn.microsoft.com/zh-cn/library/azure/gg433040.aspx
  [Azure 存储团队博客]: http://blogs.msdn.com/b/windowsazurestorage/
  [配置连接字符串]: http://msdn.microsoft.com/zh-cn/library/azure/ee758697.aspx
  [OData]: http://nuget.org/packages/Microsoft.Data.OData/5.0.2
  [Edm]: http://nuget.org/packages/Microsoft.Data.Edm/5.0.2
  [Spatial]: http://nuget.org/packages/System.Spatial/5.0.2
 

<!---HONumber=70-->