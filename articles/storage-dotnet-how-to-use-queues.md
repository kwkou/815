<properties linkid="dev-net-how-to-queue-service" urlDisplayName="Queue Service" pageTitle="如何通过 .NET 使用队列存储 | Microsoft Azure" metaKeywords="Get started Azure queue   Azure asynchronous processing   Azure queue   Azure queue storage   Azure queue .NET   Azure queue storage .NET   Azure queue C#   Azure queue storage C#" description="了解如何使用 Microsoft Azure 队列存储创建和删除队列，以及插入、扫视、获取和删除队列消息。" metaCanonical="" disqusComments="1" umbracoNaviHide="1" services="storage" documentationCenter=".NET" title="How to use Windows Azure Queue Storage" authors="tamram" />

# 如何通过 .NET 使用队列存储

本指南将演示如何使用 Azure 队列存储服务执行常见方案。示例是用 C\# 代码编写的并使用了 Azure .NET 存储客户端库。介绍的方案包括插入、查看、获取和删除队列消息以及创建和删除队列。有关队列的详细信息，请参阅[后续步骤][]部分。

> [WACOM.NOTE] 本指南适用于 Azure .NET 存储客户端库 2.x 及更高版本。建议使用的版本是存储客户端库 4.x，可通过 [NuGet](https://www.nuget.org/packages/WindowsAzure.Storage/) 或 [Azure SDK for .NET](/zh-cn/downloads/) 获得。请参阅[如何：以编程方式访问队列存储][]下面有关获得存储客户端库的详细信息。

<h2>目录</h2>

-   [什么是队列存储？][]
-   [概念][]
-   [创建 Azure 存储帐户][]
-   [设置 Azure 存储连接字符串][]
-   [如何：以编程方式访问队列存储][]
-   [如何：创建队列][]
-   [如何：在队列中插入消息][]
-   [如何：扫视下一条消息][]
-   [如何：更改已排队消息的内容][]
-   [如何：取消对下一条消息的排队][]
-   [如何：使用其他方法取消对消息的排队][]
-   [如何：获取队列长度][]
-   [如何：删除队列][]
-   [后续步骤][]

[WACOM.INCLUDE [howto-queue-storage](../includes/howto-queue-storage.md)]

## <h2><a name="create-account"></a>创建 Azure 存储帐户</h2>

[WACOM.INCLUDE [create-storage-account](../includes/create-storage-account.md)]

## <h2><a name="setup-connection-string"></a>设置 Azure 存储连接字符串</h2>

[WACOM.INCLUDE [storage-configure-connection-string](../includes/storage-configure-connection-string.md)]

## <a name="configure-access"> </a>如何：以编程方式访问队列存储

<h3>获得程序集</h3>
你可以使用 NuGet 获取  `Microsoft.WindowsAzure.Storage.dll` 程序集。在"解决方案资源管理器"中，右键单击你的项目并选择"管理 NuGet 包"。在线搜索"MicrosoftAzure.Storage"，然后单击"安装"以安装 Azure 存储包和依赖项。

Azure SDK for .NET 中也包括了 `Microsoft.WindowsAzure.Storage.dll`，可从 <a href="/zh-cn/develop/net/#">.NET 开发人员中心</a>下载该版本。程序集安装在 `%Program Files%\Microsoft SDKs\Windows Azure\.NET SDK\<sdk-version>\ref\` 目录中。

<h3>命名空间声明</h3>
在你希望以编程方式访问 Azure 存储的任何 C# 文件中，将以下代码命名空间声明添加到文件的顶部。

    using Microsoft.WindowsAzure.Storage;
    using Microsoft.WindowsAzure.Storage.Auth;
	using Microsoft.WindowsAzure.Storage.Queue;

确保你引用  `Microsoft.WindowsAzure.Storage.dll` 程序集。

<h3>检索连接字符串</h3>
可以使用 CloudStorageAccount 类型来表示您的存储帐户信息。如果你使用了 Microsoft 
Azure 项目模板并且/或者引用了 
Microsoft.WindowsAzure.CloudConfigurationManager，则可以使用 **CloudConfigurationManager** 类型从Azure 服务配置中检索你的存储连接字符串和存储帐户信息：

    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

如果你要创建应用程序而不引用 Microsoft.WindowsAzure.CloudConfigurationManager，并且你的连接字符串位于前面显示的  `web.config` 或  `app.config` 中，那么你可以使用 **ConfigurationManager** 来检索连接字符串。你需要将对 System.Configuration.dll 的引用添加到你的项目中，并为其添加另一个命名空间声明：

	using System.Configuration;
	...
	CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
		ConfigurationManager.ConnectionStrings["StorageConnectionString"].ConnectionString);

<h3>ODataLib 依赖项</h3>
.NET 存储客户端库中的 ODataLib 依赖项可通过在 NuGet （而非 WCF 数据服务）上获得的 ODataLib（5.0.2 版）包来解析。ODataLib 库可直接下载或者通过 NuGet 由代码项目引用。特定的 ODataLib 包为 [OData]、[Edm] 和 [Spatial]。

<h2><a name="create-queue"></a>如何：创建队列</h2>

利用 CloudQueueClient 对象，可以获取队列的引用对象。
以下代码将创建 CloudQueueClient 对象。本指南中的所有代码都使用存储在 Azure 应用程序的服务配置中的存储连接字符串。还可采用其他方法创建 CloudStorageAccount 对象。有关详细信息，请参阅 [CloudStorageAccount][] 文档。

    // Retrieve storage account from connection string
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the queue client
    CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

使用 queueClient 对象获取对要使用的队列的引用。如果队列不存在，你可以创建它。

    // Retrieve a reference to a queue
    CloudQueue queue = queueClient.GetQueueReference("myqueue");

    // Create the queue if it doesn't already exist
    queue.CreateIfNotExists();

<h2><a name="insert-message"> </a>如何：在队列中插入消息</h2>

若要将消息插入现有队列，请先创建一个新的 CloudQueueMessage。紧接着，调用 AddMessage 方法。可从字符串（UTF-8 格式）或字节数组创建 CloudQueueMessage。以下代码将创建队列（如果队列不存在）并插入消息  'Hello, World'。

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

<h2><a name="peek-message"></a>如何：扫视下一条消息</h2>

通过调用 PeekMessage() 方法，可以查看队列前面的消息，而不必从队列中将其删除。

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

<h2><a name="change-contents"></a>如何：更改已排队消息的内容</h2>

你可以更改队列中现有消息的内容。如果消息表示工作任务，则可以使用此功能来更新该工作任务的状态。以下代码使用新内容更新队列消息，并将可见性超时设置为再延长 60 秒。这将保存与消息关联的工作的状态，并额外为客户端提供一分钟的时间来继续处理消息。可使用此方法跟踪队列消息上的多步骤工作流，即使处理步骤因硬件或软件故障而失败，也无需从头开始操作。通常，你还可以保留重试计数，如果某条消息的重试次数超过  *n* 次，你将删除此消息。这可避免每次处理某条消息时都触发应用程序错误。

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

<h2><a name="get-message"></a>如何：取消对下一条消息的排队</h2>

你的代码通过两个步骤来取消对队列中某条消息的排队。当你调用
**GetMessage** 时，你将获得队列中的下一条消息。从 GetMessage 返回的消息变得对从此队列读取消息的任何其他代码不可见。默认情况下，此消息将持续 30 秒不可见。若要从队列中删除消息，你还必须调用 DeleteMessage。此删除消息的两步过程可确保当你的代码因硬件或软件故障而无法处理消息时，你的其他代码实例可以获取同一消息并重试。你的代码在处理消息后会立即调用 DeleteMessage。

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

<h2><a name="advanced-get"></a>如何：使用其他方法取消对消息的排队</h2>

你可以通过两种方式自定义队列中的消息检索。
首先，你可以获取一批消息（最多 32 个）。其次，您可以设置更长或更短的不可见超时时间，从而允许您的代码使用更多或更少时间来完全处理每个消息。下面的代码示例使用
**GetMessages** 方法在一次调用中获取 20 条消息。然后，它使用 foreach 循环来处理每条消息。它还将每条消息的不可见超时时间设置为 5 分钟。请注意，5 分钟超时时间对于所有消息都是同时开始的，因此在调用 GetMessages 5 分钟后，尚未删除的任何消息都将再次变得可见。

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

<h2><a name="get-queue-length"></a>如何：获取队列长度</h2>

你可以获取队列中消息的估计数。将显示
使用 FetchAttributes 方法可要求队列服务检索队列属性，包括消息计数。ApproximateMethodCount** 属性返回通过
**FetchAttributes** 方法检索的最后一个值，不调用队列服务。

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

<h2><a name="delete-queue"></a>如何：删除队列</h2>

若要删除队列及其中包含的所有消息，请针对队列对象调用
**Delete** 方法。

    // Retrieve storage account from connection string.
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
        CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // Create the queue client.
    CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

    // Retrieve a reference to a queue.
    CloudQueue queue = queueClient.GetQueueReference("myqueue");

    // Delete the queue.
    queue.Delete();

<h2><a name="next-steps"></a>后续步骤</h2>

现在，你已了解有关队列存储的基础知识，可单击下面的链接来了解如何执行更复杂的存储任务。

<ul>
<li>查看队列服务参考文档，了解有关可用 API 的完整详细信息：
  <ul>
    <li><a href="http://msdn.microsoft.com/zh-cn/library/azure/wa_storage_30_reference_home.aspx">.NET 存储客户端库参考</a>
    </li>
    <li><a href="http://msdn.microsoft.com/zh-cn/library/azure/dd179355">REST API 参考</a></li>
  </ul>
</li>
<li>在以下位置了解使用 Azure 存储空间能够执行的更高级任务：<a href="http://msdn.microsoft.com/zh-cn/library/azure/gg433040.aspx">在 Azure 中存储和访问数据</a>。</li>
<li>了解如何使用 <a href="/zh-cn/documentation/articles/ Websites-dotnet-webjobs-sdk/">Azure WebJobs SDK 简化你编写的用于 Azure 存储空间的代码。</li>
<li>查看更多功能指南，以了解在 Azure 中存储数据的其他方式。
  <ul>
    <li>使用<a href="/zh-cn/documentation/articles/storage-dotnet-how-to-use-tables/">表存储</a>来存储结构化数据。</li>
    <li>使用 <a href="/zh-cn/documentation/articles/storage-dotnet-how-to-use-blobs/">Blob 存储</a>来存储非结构化数据。</li>
    <li>使用 <a href="/zh-cn/documentation/articles/sql-database-dotnet-how-to-use/">SQL Database</a> 来存储关系数据。</li>
  </ul>
</li>
</ul>



  [后续步骤]: #next-steps
  [什么是队列存储？]: #what-is
  [概念]: #concepts
  [创建 Azure 存储帐户]: #create-account
  [设置 Azure 存储连接字符串]: #setup-connection-string
  [如何：以编程方式访问队列存储]: #configure-access
  [如何：创建队列]: #create-queue
  [如何：在队列中插入消息]: #insert-message
  [如何：扫视下一条消息]: #peek-message
  [如何：更改已排队消息的内容]: #change-contents
  [如何：取消对下一条消息的排队]: #get-message
  [如何：使用其他方法取消对消息的排队]: #advanced-get
  [如何：获取队列长度]: #get-queue-length
  [如何：删除队列]: #delete-queue
  [下载并安装 Azure SDK for.NET]：/zh-cn/develop/net/
  [.NET 客户端库引用]: http://msdn.microsoft.com/zh-cn/library/azure/wa_storage_30_reference_home.aspx
  [在 Visual Studio 中创建 Azure 项目]: http://msdn.microsoft.com/zh-cn/library/azure/ee405487.aspx
  [CloudStorageAccount]: http://msdn.microsoft.com/zh-cn/library/microsoft.windowsazure.cloudstorageaccount_methods.aspx
  [在 Azure 中存储和访问数据]: http://msdn.microsoft.com/zh-cn/library/azure/gg433040.aspx
  [Azure 存储团队博客]: http://blogs.msdn.com/b/windowsazurestorage/
  [配置连接字符串]: http://msdn.microsoft.com/zh-cn/library/azure/ee758697.aspx
  [OData]: http://nuget.org/packages/Microsoft.Data.OData/5.0.2
  [Edm]: http://nuget.org/packages/Microsoft.Data.Edm/5.0.2
  [Spatial]: http://nuget.org/packages/System.Spatial/5.0.2
<!--HONumber=41-->
