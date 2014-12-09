<properties linkid="dev-net-how-to-queue-service" urlDisplayName="Queue Service" pageTitle="How to use queue storage from .NET | Microsoft Azure" metaKeywords="Get started Azure queue   Azure asynchronous processing   Azure queue   Azure queue storage   Azure queue .NET   Azure queue storage .NET   Azure queue C#   Azure queue storage C#" description="Learn how to use Microsoft Azure Queue storage to create and delete queues and insert, peek, get, and delete queue messages." metaCanonical="" disqusComments="1" umbracoNaviHide="1" services="storage" documentationCenter=".NET" title="How to use Microsoft Azure Queue Storage" authors="tamram" />

# 如何通过 .NET 使用队列存储

本指南将演示如何使用 Azure 队列存储服务执行常见方案。
示例是用 C\# 代码编写的并
使用了 Azure .NET 存储客户端库。涉及的方案包括**插入**、
**查看**、
**获取**和
**删除**队列消息以及
**创建和删除队列**。有关队列的详细信息，请参阅
[后续步骤][]部分。

> [WACOM.NOTE] 本指南适用于 Azure .NET 存储客户端库 2.x 及更高版本。建议使用的版本是存储客户端库 3.x，可通过 NuGet 或 Azure SDK for .NET 2.3 获得。请参阅下文中的 [如何：以编程方式访问队列存储][]，了解有关获得存储客户端库的详细信息。

## 目录

-   [什么是队列存储][]
-   [概念][]
-   [创建 Azure 存储帐户][]
-   [设置 Azure 存储连接字符串][]
-   [如何：以编程方式访问队列存储][]
-   [如何：创建队列][]
-   [如何：在队列中插入消息][]
-   [如何：查看下一条消息][]
-   [如何：更改已排队消息的内容][]
-   [如何：取消对下一条消息的排队][]
-   [如何：使用其他方法取消对消息的排队][]
-   [如何：获取队列长度][]
-   [如何：删除队列][]
-   [后续步骤][]

[WACOM.INCLUDE [howto-queue-storage][]]

## 创建帐户创建 Azure 存储帐户

[WACOM.INCLUDE [create-storage-account][]]

## 设置连接字符串设置 Azure 存储连接字符串

Azure .NET 存储客户端库支持使用存储连接字符
串来配置终结点和用于访问存
储服务的凭据。你可以将存储连接字符串放置在一个配置文件中，而不是
在代码中对其进行硬编码：

-   当使用 Azure 云服务时，建议你使用 Azure 服务配置系统（`*.csdef` 和 `*.cscfg` 文件）来存储连接字符串。
-   在使用 Azure 网站、Azure 虚拟机时或者构建准备在 Azure 外部运行的 .NET 应用程序时，建议你使用 .NET 配置系统（如 `web.config` 或 `app.config` 文件）来存储连接字符串。

在上述两种情况下，你都可以使用 `CloudConfigurationManager.GetSetting` 方法检索连接字符串，本指南稍后将对此进行介绍。

### 在使用云服务时配置连接字符串

该服务配置机制是 Azure 云服务
项目特有的，它使你能够从 Azure 管理
门户动态更改配置设置，而无需部署你的应用程序。

在 Azure 服务配置中配置连接字符串：

1.  在 Visual Studio 解决方案资源管理器内 Azure 部署项目的**“角色”**文件夹中，右键单击你的 Web 角色或辅助角色，然后单击**“属性”**。

    ![Blob5][]

2.  单击**“设置”**选项卡并按**“添加设置”**按钮。

    ![Blob6][]

    新的 **Setting1** 条目稍后将显示在设置网格中。

3.  在新的 **Setting1** 条目的**“类型”**下拉列表中，选择**“连接字符串”**。

    ![Blob7][]

4.  单击 **Setting1** 条目最右侧的 **...** 按钮。此时将打开**“存储帐户连接字符串”**对话框。

5.  选择是要定位到存储模拟器（在本地计算机上模拟的 Azure 存储空间），还是要定位到云中的实际存储帐户。本指南中的代码使用其中任一方式。如果你希望使用我们之前在 Azure 中创建的存储帐户来存储队列数据，请输入从本教程前面的步骤中复制的**“主访问密钥”**值。

    ![Blob8][]

6.  将条目**“名称”**从 **Setting1** 更改为更友好的名称，例如 **StorageConnectionString**。稍后将在本指南的代码中引用此连接字符串。

    ![Blob9][]

### 使用 .NET 配置来配置连接字符串

如果你正在编写不是 Azure 云服务的应用程序（参见上一部分），则建议你使用 .NET 配置系统（如 `web.config` 或 `app.config`）。这包括 Azure 网站或 Azure 虚拟机，以及设计为在 Azure 外部运行的应用程序。你可以使用 `<appSettings>` 元素存储连接字符串，如下所示：

    <configuration>
    <appSettings>
    <add key="StorageConnectionString" value="DefaultEndpointsProtocol=https;AccountName=[AccountName];AccountKey=[AccountKey" />
    </appSettings>
    </configuration>

阅读[配置连接字符串][]，了解有关存储连接字符串的详细信息。

你现在即可准备执行本指南中的操作任务。

## 以编程方式访问如何：以编程方式访问队列存储

### 获得程序集

你可以使用 NuGet 来获得 `Microsoft.WindowsAzure.Storage.dll` 程序集。在**“解决方案资源管理器”**中，右键单击你的项目并选择**“管理 NuGet 包”**。在线搜索“WindowsAzure.Storage”，然后单击**“安装”**以安装 Azure 存储包和依赖项。

Azure SDK for .NET 中也包括了 `Microsoft.WindowsAzure.Storage.dll`，可从 [.NET 开发人员中心][]下载该版本。该程序集将安装到 `%Program Files%\Microsoft SDKs\Windows Azure\.NET SDK\<sdk-version>\ref\` 目录中。

### 命名空间声明

在你希望在其中以编程方式访问 Azure 存储空间的任何 C\# 文件中，
将以下代码命名空间声明添加到文件的顶部：

    using Microsoft.WindowsAzure.Storage;
    using Microsoft.WindowsAzure.Storage.Auth;
    using Microsoft.WindowsAzure.Storage.Queue;

确保你引用了 `Microsoft.WindowsAzure.Storage.dll` 程序集。

### 检索连接字符串

可以使用 **CloudStorageAccount** 类型来表示你的存储
帐户信息。如果你使用的
是 Windows Azure 项目模板并且/或者引用了
 Microsoft.WindowsAzure.CloudConfigurationManager，
则可以使用 **CloudConfigurationManager** 类型
从 Azure 服务配置中检索你的存储连接字符串和
存储帐户信息：

    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

如果你要创建应用程序而不引用 Microsoft.WindowsAzure.CloudConfigurationManager，并且你的连接字符串位于 `web.config` 或 `app.config` 中（如上所示），则你可以使用 **ConfigurationManager** 来检索连接字符串。你需要将对 System.Configuration.dll 的引用添加到你的项目中，并为其添加另一个命名空间声明：

    using System.Configuration;
    ...
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    ConfigurationManager.ConnectionStrings["StorageConnectionString"].ConnectionString);

### ODataLib 依赖项

.NET 存储客户端库中的 ODataLib 依赖项可通过在 NuGet （而非 WCF 数据服务）上获得的 ODataLib（5.0.2 版）包来解析。ODataLib 库可直接下载或者通过 NuGet 由代码项目引用。特定的 ODataLib 包为 [OData][]、[Edm][] 和 [Spatial][]。

## 创建队列如何：创建队列

利用 **CloudQueueClient** 对象，可以获取队列的引用对象。
下面的代码将创建一个 **CloudQueueClient** 对象。本指南中
的所有代码都使用存储在 Azure 应用程序的服务配置中的
存储连接字符串。还有其他方法可用来创建
**CloudStorageAccount** 对象。有关详细信息，
请参阅[CloudStorageAccount][] 文档。

    // 通过连接字符串检索存储帐户
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // 创建队列客户端
    CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

使用 **queueClient** 对象获取对要使用的队列的引用。
如果该队列不存在，你可以创建它。

    // 检索对队列的引用
    CloudQueue queue = queueClient.GetQueueReference("myqueue");

    // 如果该队列不存在，则创建它
    queue.CreateIfNotExists();

## 插入消息如何：在队列中插入消息

若要将消息插入现有队列，请先创建一个新的
**CloudQueueMessage**。然后调用 **AddMessage** 方法。
可基于字符串（UTF-8 格式）或**“字节”**数组创建
**CloudQueueMessage**。以下代码将创建一个队列（如果该队列不存在）
并插入消息“Hello, World”：

    // 通过连接字符串检索存储帐户。
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // 创建队列客户端。
    CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

    // 检索对队列的引用。
    CloudQueue queue = queueClient.GetQueueReference("myqueue");

    // 如果该队列不存在，则创建它。
    queue.CreateIfNotExists();

    // 创建一条消息并将其添加到队列中。
    CloudQueueMessage message = new CloudQueueMessage("Hello, World");
    queue.AddMessage(message);

## 扫视下一条消息如何：扫视下一条消息

通过调用 **PeekMessage** 方法，你可以扫视队列前面的
消息，而不会从队列中删除它。

    // 通过连接字符串检索存储帐户
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // 创建队列客户端
    CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

    // 检索对队列的引用
    CloudQueue queue = queueClient.GetQueueReference("myqueue");

    // 扫视下一条消息
    CloudQueueMessage peekedMessage = queue.PeekMessage();

    // 显示消息。
    Console.WriteLine(peekedMessage.AsString);

## 更改消息内容如何：更改已排队消息的内容

你可以更改队列中现有消息的内容。如果消息
表示工作任务，则可以使用此功能来更新该工作任务的状态。
以下代码使用新内容更新队列消息，
并将可见性超时设置为再延长 60 秒。
这将保存与消息关联的工作的状态，并额外为客户端提供
一分钟的时间来继续处理消息。可使用
此方法跟踪队列消息上的多步骤工作流，
即使处理步骤因硬件或软件故障而失败，
也无需从头开始操作。通常，
你还可以保留重试计数，如果某条消息的重试次数
超过 *n* 次，你将删除该消息。这可避免每次处理某条消息时
都触发应用程序错误。

    // 通过连接字符串检索存储帐户。
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // 创建队列客户端。
    CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

    // 检索对队列的引用。
    CloudQueue queue = queueClient.GetQueueReference("myqueue");

    // 从队列中获取消息并更新消息内容。
    CloudQueueMessage message = queue.GetMessage();
    message.SetMessageContent("Updated contents.") ;
    queue.UpdateMessage(message, 
    TimeSpan.FromSeconds(0.0),  // 使其立即可见。
    MessageUpdateFields.Content | MessageUpdateFields.Visibility);

## 取消对下一条消息的排队如何：取消对下一条消息的排队

你的代码通过两个步骤来取消对队列中某条消息的排队。在调用
**GetMessage** 时，你将获得队列中的下一条消息。对于从该队列读取消息的
任何其他代码，从 **GetMessage** 返回的
消息将变得不可见。默认情况下，此消息将持续 30 秒不可见。
若要从队列中删除消息，你还必须调用 **DeleteMessage**。
此删除消息的两步过程可确保当你的代码因硬件
或软件故障而无法处理消息时，你的代码的
另一实例可以获取同一消息并重试。
你的代码在处理消息后会立即调用 **DeleteMessage**。

    // 通过连接字符串检索存储帐户
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // 创建队列客户端
    CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

    // 检索对队列的引用
    CloudQueue queue = queueClient.GetQueueReference("myqueue");

    // 获取下一条消息
    CloudQueueMessage retrievedMessage = queue.GetMessage();

    //在不到 30 秒的时间内处理消息，然后删除消息
    queue.DeleteMessage(retrievedMessage);

## 更多取消排队方法如何：使用其他方法取消对消息的排队

你可以通过两种方式自定义队列的消息检索。
首先，你可以获取一批消息（最多 32 条）。其次，你可以
设置更长或更短的不可见超时，从而允许你的代码使用更多或
更少的时间来彻底处理每条消息。以下代码示例使用
**GetMessages** 方法通过一次调用获取 20 条消息。然后，
它使用 **foreach** 循环来处理每条消息。它还将每条消息
的不可见超时时间设置为 5 分钟。请注意，5 分钟超时时间
对于所有消息都是同时开始的，因此在调用
**GetMessages** 5 分钟后，尚未删除的任何消息
都将再次变得可见。

    // 通过连接字符串检索存储帐户。
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // 创建队列客户端。
    CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

    // 检索对队列的引用。
    CloudQueue queue = queueClient.GetQueueReference("myqueue");

    foreach (CloudQueueMessage message in queue.GetMessages(20, TimeSpan.FromMinutes(5)))
    {
    // 在不到 5 分钟的时间内处理所有消息，并在处理后删除每条消息。
    queue.DeleteMessage(message);
    }

## 获取队列长度如何：获取队列长度

你可以获取队列中消息的估计数。使用 **FetchAttributes**
方法可要求队列服务检索队列属性，
包括消息计数。**ApproximateMethodCount** 属性
返回 **FetchAttributes** 方法检索到的
最后一个值，而不会调用队列服务。

    // 通过连接字符串检索存储帐户。
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // 创建队列客户端。
    CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

    // 检索对队列的引用。
    CloudQueue queue = queueClient.GetQueueReference("myqueue");

    // 提取队列属性。
    queue.FetchAttributes();

    // 检索所缓存的近似消息计数。
    int? cachedMessageCount = queue.ApproximateMessageCount;

    // 显示消息的数目。
    Console.WriteLine("Number of messages in queue:" + cachedMessageCount);

## 删除队列如何：删除队列

若要删除队列及其包含的所有消息，请对队列对象
调用 **Delete** 方法。

    // 通过连接字符串检索存储帐户。
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(
    CloudConfigurationManager.GetSetting("StorageConnectionString"));

    // 创建队列客户端。
    CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

    // 检索对队列的引用。
    CloudQueue queue = queueClient.GetQueueReference("myqueue");

    // 删除队列。
    queue.Delete();

## 后续步骤

现在，你已了解有关队列存储的基础知识，可单击下面的链接来了解如何
执行更复杂的存储任务。

-   查看队列服务参考文档，了解有关可用 API 的完整详细信息：
    -   [.NET 存储客户端库参考][]
    -   [REST API 参考][]
-   在以下位置了解使用 Azure 存储空间能够执行的更高级任务：[在 Azure 中存储和访问数据][]。
-   查看更多功能指南，以了解在 Azure 中存储数据的其他方式。
    -   使用[表存储][]来存储结构化数据。
    -   使用 [Blob 存储][]来存储非结构化数据。
    -   使用 [SQL Database][] 来存储关系数据。

  [后续步骤]: #next-steps
  [什么是队列存储]: #what-is
  [概念]: #concepts
  [创建 Azure 存储帐户]: #create-account
  [设置 Azure 存储连接字符串]: #setup-connection-string
  [如何：创建队列]: #create-queue
  [如何：在队列中插入消息]: #insert-message
  [如何：查看下一条消息]: #peek-message
  [如何：更改已排队消息的内容]: #change-contents
  [如何：取消对下一条消息的排队]: #get-message
  [如何：使用其他方法取消对消息的排队]: #advanced-get
  [如何：获取队列长度]: #get-queue-length
  [如何：删除队列]: #delete-queue
  [howto-queue-storage]: ../includes/howto-queue-storage.md
  [create-storage-account]: ../includes/create-storage-account.md
  [Blob5]: ./media/storage-dotnet-how-to-use-queues/blob5.png
  [Blob6]: ./media/storage-dotnet-how-to-use-queues/blob6.png
  [Blob7]: ./media/storage-dotnet-how-to-use-queues/blob7.png
  [Blob8]: ./media/storage-dotnet-how-to-use-queues/blob8.png
  [Blob9]: ./media/storage-dotnet-how-to-use-queues/blob9.png
  [配置连接字符串]: http://msdn.microsoft.com/zh-cn/library/azure/ee758697.aspx
  [.NET 开发人员中心]: http://azure.microsoft.com/zh-cn/develop/net/
  [OData]: http://nuget.org/packages/Microsoft.Data.OData/5.0.2
  [Edm]: http://nuget.org/packages/Microsoft.Data.Edm/5.0.2
  [Spatial]: http://nuget.org/packages/System.Spatial/5.0.2
  [CloudStorageAccount]: http://msdn.microsoft.com/zh-cn/library/microsoft.windowsazure.cloudstorageaccount_methods.aspx
  [.NET 存储客户端库参考]: http://msdn.microsoft.com/zh-cn/library/azure/wa_storage_30_reference_home.aspx
  [REST API 参考]: http://msdn.microsoft.com/zh-cn/library/azure/dd179355
  [在 Azure 中存储和访问数据]: http://msdn.microsoft.com/zh-cn/library/azure/gg433040.aspx
  [表存储]: /en-us/documentation/articles/storage-dotnet-how-to-use-tables/
  [Blob 存储]: /en-us/documentation/articles/storage-dotnet-how-to-use-blobs/
  [SQL Database]: /en-us/documentation/articles/sql-database-dotnet-how-to-use/
