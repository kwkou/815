<properties
    pageTitle="通过 .NET 开始使用 Azure 队列存储 | Azure"
    description="Azure 队列用于在应用程序组件之间进行可靠的异步消息传送。 应用程序组件可以利用云消息传送进行独立缩放。"
    services="storage"
    documentationcenter=".net"
    author="robinsh"
    manager="timlt"
    editor="tysonn"
    translationtype="Human Translation" />
<tags
    ms.assetid="c0f82537-a613-4f01-b2ed-fc82e5eea2a7"
    ms.service="storage"
    ms.workload="storage"
    ms.tgt_pltfrm="na"
    ms.devlang="dotnet"
    ms.topic="hero-article"
    ms.date="03/27/2017"
    wacn.date="05/02/2017"
    ms.author="robinsh"
    ms.sourcegitcommit="78da854d58905bc82228bcbff1de0fcfbc12d5ac"
    ms.openlocfilehash="c08e14d54e647d33f4a0609df5da91d10e6b53a4"
    ms.lasthandoff="04/22/2017" />

# <a name="get-started-with-azure-queue-storage-using-net"></a>通过 .NET 开始使用 Azure 队列存储
[AZURE.INCLUDE [storage-selector-queue-include](../../includes/storage-selector-queue-include.md)]

[AZURE.INCLUDE [storage-check-out-samples-dotnet](../../includes/storage-check-out-samples-dotnet.md)]

## <a name="overview"></a>概述
Azure 队列存储用于在应用程序组件之间进行云消息传送。 在设计应用程序以实现可伸缩性时，通常要将各个应用程序组件分离，使其可以独立地进行伸缩。 队列存储提供的异步消息传送适用于在应用程序组件之间进行通信，无论这些应用程序组件是运行在云中、桌面上、本地服务器上还是移动设备上。 队列存储还支持管理异步任务以及构建过程工作流。

### <a name="about-this-tutorial"></a>关于本教程
本教程演示如何针对使用 Azure 队列存储一些常见情形编写 .NET 代码。 涉及的方案包括创建和删除队列、添加、读取和删除队列消息。

**估计完成时间：** 45 分钟

**先决条件：**

* [Microsoft Visual Studio](https://www.visualstudio.com/en-us/visual-studio-homepage-vs.aspx)
* [适用于 .NET 的 Azure 存储空间客户端库](https://www.nuget.org/packages/WindowsAzure.Storage/)
* [适用于 .NET 的 Azure Configuration Manager](https://www.nuget.org/packages/Microsoft.WindowsAzure.ConfigurationManager/)
* 一个 [Azure 存储帐户](/documentation/articles/storage-create-storage-account/#create-a-storage-account)

[AZURE.INCLUDE [storage-dotnet-client-library-version-include](../../includes/storage-dotnet-client-library-version-include.md)]

[AZURE.INCLUDE [storage-queue-concepts-include](../../includes/storage-queue-concepts-include.md)]

[AZURE.INCLUDE [storage-create-account-include](../../includes/storage-create-account-include.md)]

[AZURE.INCLUDE [storage-development-environment-include](../../includes/storage-development-environment-include.md)]

### <a name="add-using-directives"></a>添加 using 指令
将以下 `using` 指令添加到 `Program.cs` 文件顶部：

    using Microsoft.Azure; // Namespace for CloudConfigurationManager
    using Microsoft.WindowsAzure.Storage; // Namespace for CloudStorageAccount
    using Microsoft.WindowsAzure.Storage.Queue; // Namespace for Queue storage types

### <a name="parse-the-connection-string"></a>解析连接字符串
[AZURE.INCLUDE [storage-cloud-configuration-manager-include](../../includes/storage-cloud-configuration-manager-include.md)]

### <a name="create-the-queue-service-client"></a>创建队列服务客户端
**CloudQueueClient** 类使你能够检索存储在队列存储中的队列。 下面是创建服务客户端的一种方法：

    CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

    
现在，你已准备好编写从队列存储读取数据并将数据写入队列存储的代码。

## <a name="create-a-queue"></a>创建队列
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

## <a name="insert-a-message-into-a-queue"></a>在队列中插入消息
若要将消息插入现有队列，请先创建一个新的 **CloudQueueMessage**。 接下来，调用 **AddMessage** 方法。 可从字符串（UTF-8 格式）或**字节**数组创建 **CloudQueueMessage**。 以下代码将创建队列（如果该队列不存在）并插入消息“Hello, World”：

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

## <a name="peek-at-the-next-message"></a>扫视下一条消息
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

## <a name="change-the-contents-of-a-queued-message"></a>更改已排队消息的内容
你可以更改队列中现有消息的内容。 如果消息表示工作任务，可使用此功能来更新该工作任务的状态。 以下代码使用新内容更新队列消息，并将可见性超时设置为再延长 60 秒。 这将保存与消息关联的工作的状态，并额外为客户端提供一分钟的时间来继续处理消息。 可使用此方法跟踪队列消息上的多步骤工作流，即使处理步骤因硬件或软件故障而失败，也无需从头开始操作。 通常同时保留重试计数，当消息重试次数超过 *n* 时再删除该消息。 这可避免每次处理某条消息时都触发应用程序错误。

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
        TimeSpan.FromSeconds(60.0),  // Make it invisible for another 60 seconds.
        MessageUpdateFields.Content | MessageUpdateFields.Visibility);

## <a name="de-queue-the-next-message"></a>取消对下一条消息的排队
你的代码通过两个步骤来取消对队列中某条消息的排队。 调用 **GetMessage**时，你将获取队列中的下一条消息。 从 **GetMessage** 返回的消息变得对从此队列读取消息的任何其他代码不可见。 默认情况下，此消息将持续 30 秒不可见。 若要从队列中删除消息，你还必须调用 **DeleteMessage**。 此删除消息的两步过程可确保，如果你的代码因硬件或软件故障而无法处理消息，则你的代码的其他实例可以获取相同消息并重试。 你的代码在处理消息后会立即调用 **DeleteMessage** 。

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

## <a name="use-async-await-pattern-with-common-queue-storage-apis"></a>将 Async-Await 模式与公用队列存储 API 配合使用
此示例演示如何将 Async-Await 模式和公用队列存储 API 配合使用。 示例调用每个给定方法的异步版本，如每个方法的 *Async* 后缀所示。 使用异步方法时，async-await 模式将暂停本地执行，直到调用完成。 此行为允许当前的线程执行其他工作，这有助于避免性能瓶颈并提高应用程序的整体响应能力。 有关在 .NET 中使用 Async-Await 模式的详细信息，请参阅 [Async 和 Await（C# 和 Visual Basic）](https://msdn.microsoft.com/zh-cn/library/hh191443.aspx)

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

    
## <a name="leverage-additional-options-for-de-queuing-messages"></a>使用其他方法取消对消息的排队
你可以通过两种方式自定义队列中的消息检索。首先，可获取一批消息（最多 32 条）。其次，你可以设置更长或更短的不可见超时时间，从而允许你的代码使用更多或更少时间来完全处理每个消息。以下代码示例使用 **GetMessages** 方法在一次调用中获取 20 条消息。然后，它使用 **foreach** 循环处理每条消息。它还将每条消息的不可见超时时间设置为 5 分钟。请注意，5 分钟超时时间对于所有消息都是同时开始的，因此在调用 **GetMessages** 5 分钟后，尚未删除的任何消息都将再次变得可见。

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

## <a name="get-the-queue-length"></a>获取队列长度
你可以获取队列中消息的估计数。 使用 **FetchAttributes** 方法可请求队列服务检索队列属性，包括消息计数。 **ApproximateMessageCount** 属性返回 **FetchAttributes** 方法检索到的最后一个值，而不会调用队列服务。

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

## <a name="delete-a-queue"></a>删除队列
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

    

## <a name="next-steps"></a>后续步骤
现在，您已了解有关队列存储的基础知识，可单击下面的链接来了解更复杂的存储任务。

* 查看队列服务参考文档，了解有关可用 API 的完整详细信息：
  * [.NET 存储客户端库参考](http://go.microsoft.com/fwlink/?LinkID=390731&clcid=0x409)
  * [REST API 参考](http://msdn.microsoft.com/zh-cn/library/azure/dd179355)
* 了解如何通过使用 [Azure WebJobs SDK](/documentation/articles/websites-dotnet-webjobs-sdk/)简化为使用 Azure 存储空间而写的代码。
* 查看更多功能指南，以了解在 Azure 中存储数据的其他方式。
  * [通过 .NET 开始使用 Azure 表存储](/documentation/articles/storage-dotnet-how-to-use-tables/) 来存储结构化数据。
  * [通过 .NET 开始使用 Azure Blob 存储](/documentation/articles/storage-dotnet-how-to-use-blobs/) 来存储非结构化数据。
  * [使用.NET (C#) 连接到 SQL 数据库](/documentation/articles/sql-database-develop-dotnet-simple/)，存储关系数据。

[Download and install the Azure SDK for .NET]: /develop/net/
[.NET client library reference]: http://go.microsoft.com/fwlink/?LinkID=390731&clcid=0x409
[Creating a Azure Project in Visual Studio]: http://msdn.microsoft.com/zh-cn/library/azure/ee405487.aspx
[Azure Storage Team Blog]: http://blogs.msdn.com/b/windowsazurestorage/
[OData]: http://nuget.org/packages/Microsoft.Data.OData/5.0.2
[Edm]: http://nuget.org/packages/Microsoft.Data.Edm/5.0.2
[Spatial]: http://nuget.org/packages/System.Spatial/5.0.2
<!--Update_Description:wording update;add anchors to sub titles-->