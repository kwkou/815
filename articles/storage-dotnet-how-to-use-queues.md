<properties
	pageTitle="通过 .NET 开始使用 Azure 队列存储 | Azure"
	description="使用 Azure 队列存储在应用程序组件之间异步发送和接收消息。立即开始使用简单的队列存储操作，包括创建和删除队列以及添加、读取和删除队列消息。"
	services="storage"
	documentationCenter=".net"
	authors="tamram"
	manager="adinah"
	editor=""/>

<tags
	ms.service="storage"
	ms.date="04/07/2016"
	wacn.date="05/16/2016"/>

# 通过 .NET 开始使用 Azure 队列存储

[AZURE.INCLUDE [storage-selector-queue-include](../includes/storage-selector-queue-include.md)]

## 概述

Azure 队列存储是一种在云中提供消息传递队列的服务。在设计应用程序以实现可伸缩性时，通常要将各个应用程序组件分离，使其可以独立地进行伸缩。队列存储为在应用程序组件之间进行异步通信提供了一种可靠的消息传送解决方案，无论这些应用程序组件是在云中、在桌面上、在本地服务器上运行还是在移动设备上运行。队列存储还支持管理异步任务以及构建过程工作流。

### 关于本教程

本教程演示如何针对使用 Azure 队列存储一些常见情形编写 .NET 代码。涉及的方案包括创建和删除队列、添加、读取和删除队列消息。

**估计完成时间：**45 分钟

**先决条件：**

- [Microsoft Visual Studio](https://www.visualstudio.com/zh-cn/visual-studio-homepage-vs.aspx)
- [适用于 .NET 的 Azure 存储空间客户端库](https://www.nuget.org/packages/WindowsAzure.Storage/)
- [适用于 .NET 的 Azure Configuration Manager](https://www.nuget.org/packages/Microsoft.WindowsAzure.ConfigurationManager/)
- [Azure 存储帐户](/documentation/articles/storage-create-storage-account/#create-a-storage-account)


[AZURE.INCLUDE [storage-dotnet-client-library-version-include](../includes/storage-dotnet-client-library-version-include.md)]

[AZURE.INCLUDE [storage-queue-concepts-include](../includes/storage-queue-concepts-include.md)]

[AZURE.INCLUDE [storage-create-account-include](../includes/storage-create-account-include.md)]

[AZURE.INCLUDE [storage-development-environment-include](../includes/storage-development-environment-include.md)]

### 添加命名空间声明

将下列 `using` 语句添加到 `program.cs` 文件顶部：

	using Microsoft.Azure; // Namespace for CloudConfigurationManager 
	using Microsoft.WindowsAzure.Storage; // Namespace for CloudStorageAccount
    using Microsoft.WindowsAzure.Storage.Queue; // Namespace for Queue storage types

### 解析连接字符串

[AZURE.INCLUDE [storage-cloud-configuration-manager-include](../includes/storage-cloud-configuration-manager-include.md)]

### 创建队列服务客户端

**CloudQueueClient** 类使你能够检索存储在队列存储中的队列。下面是创建服务客户端的一种方法：

    CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

现在，你已准备好编写从队列存储读取数据并将数据写入队列存储的代码。

## 创建队列

此示例演示如何创建队列（如果队列已经不存在）：

    // Retrieve storage account from connection string.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the queue client.
    CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

    // Retrieve a reference to a container.
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

你可以更改队列中现有消息的内容。如果消息表示工作任务，则你可以使用此功能来更新该工作任务的状态。以下代码使用新内容更新队列消息，并将可见性超时设置为再延长 60 秒。这将保存与消息关联的工作的状态，并额外为客户端提供一分钟的时间来继续处理消息。可使用此方法跟踪队列消息上的多步骤工作流，即使处理步骤因硬件或软件故障而失败，也无需从头开始操作。通常，你还可以保留重试计数，如果某条消息的重试次数超过 n，你将删除此消息。这可避免每次处理某条消息时都触发应用程序错误。

    // Retrieve storage account from connection string.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the queue client.
    CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

    // Retrieve a reference to a queue.
    CloudQueue queue = queueClient.GetQueueReference("myqueue");

	// Get the message from the queue and update the message contents.
    CloudQueueMessage message = queue.GetMessage();
    message.SetMessageContent("Updated contents.");
    queue.UpdateMessage(message,
        TimeSpan.FromSeconds(60.0),  // Make it visible for another 60 seconds.
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

你可以获取队列中消息的估计数。使用 **FetchAttributes** 方法可请求队列服务检索队列属性，包括消息计数。**ApproximateMessageCount** 属性返回 **FetchAttributes** 方法检索到的最后一个值，而不会调用队列服务。

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

若要删除队列及其包含的所有消息，请对队列对象调用 **Delete** 方法。

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
    - [.NET 存储客户端库参考](https://msdn.microsoft.com/zh-cn/library/mt347887.aspx)
    - [REST API 参考](http://msdn.microsoft.com/zh-cn/library/azure/dd179355)
- 了解如何通过使用 [Azure WebJobs SDK](/documentation/articles/websites-dotnet-webjobs-sdk/) 简化为使用 Azure 存储空间而写的代码。
- 查看更多功能指南，以了解在 Azure 中存储数据的其他方式。
    - [通过 .NET 开始使用 Azure 表存储](/documentation/articles/storage-dotnet-how-to-use-tables/)来存储结构化数据。
    - [通过 .NET 开始使用 Azure Blob 存储](/documentation/articles/storage-dotnet-how-to-use-blobs/)来存储非结构化数据。
    - [如何在 .NET 应用程序中使用 Azure SQL 数据库](/documentation/articles/sql-database-dotnet-how-to-use/)来存储关系数据。



  [下载并安装 Azure SDK for.NET]:/develop/net/
  [.NET 客户端库引用]: https://msdn.microsoft.com/zh-cn/library/mt347887.aspx
  [在 Visual Studio 中创建 Azure 项目]: http://msdn.microsoft.com/zh-cn/library/azure/ee405487.aspx
  [Azure 存储团队博客]: http://blogs.msdn.com/b/windowsazurestorage/
  [OData]: http://nuget.org/packages/Microsoft.Data.OData/5.0.2
  [Edm]: http://nuget.org/packages/Microsoft.Data.Edm/5.0.2
  [Spatial]: http://nuget.org/packages/System.Spatial/5.0.2
 

<!---HONumber=Mooncake_0509_2016-->