<properties linkid="dev-net-how-to-use-queue-storage-service-java" urlDisplayName="Queue Service" pageTitle="How to use the queue service (Java) | Microsoft Azure" metaKeywords="Azure Queue Service, Azure Queue storage service, queues peeking, queues insert messages, queues get messages, queues delete messages, create queues, delete queues, Queue service Java" description="Learn how to use the Azure Queue service to create and delete queues, and insert, get, and delete messages. Samples written in Java." metaCanonical="" services="storage" documentationCenter="Java" title="How to use the Queue storage service from Java" authors="" solutions="" manager="" editor="" />

# 如何通过 Java 使用队列存储服务

本指南将演示如何使用 Azure 队列存储服务执行常见方案。
示例是用 Java 编写的并且使用了
[Azure SDK for Java][]。涉及的方案包括插入、扫视、
获取和删除队列消息以及创建和删除队列。
有关队列的详细信息，请参阅
[后续步骤][]部分。

## 目录

-   [什么是队列存储][]
-   [概念][]
-   [创建 Azure 存储帐户][]
-   [创建 Java 应用程序][]
-   [配置你的应用程序以访问队列存储][]
-   [设置 Azure 存储连接字符串][]
-   [如何：创建队列][]
-   [如何：向队列添加消息][]
-   [如何：查看下一条消息][]
-   [如何：取消对下一条消息的排队][]
-   [如何：更改已排队消息的内容][]
-   [用于对消息取消排队的其他方法][]
-   [如何：获取队列长度][]
-   [如何：删除队列][]
-   [后续步骤][]

[WACOM.INCLUDE [howto-queue-storage][]]

## 创建 Azure 存储帐户

[WACOM.INCLUDE [create-storage-account][]]

## 创建 Java 应用程序

在本指南中，你将使用存储功能，这些功能可在本地 Java 应用
程序中运行，或在 Azure 的 Web 角色或辅助角色中通过运行的代码
来运行。我们假定你已下载并安装了
Java 开发工具包 (JDK)，已按照
[下载 Azure SDK for Java][Azure SDK for Java] 中的说明安装了
Azure Libraries for Java 和 Azure SDK，
并已在你的 Azure 订阅中创建了一个
Azure 存储帐户。你可以使用任何开发工具（包括“记事本”）
创建应用程序。你只要能够编译 Java 项目并引用
Azure Libraries for Java 即可。

## 配置应用程序以访问队列存储

将下列 import 语句添加到需要在其中使用 Azure 存储 API 来
访问队列的 Java 文件的顶部：

    // 包括下列 import 语句以使用队列 API
    import com.microsoft.windowsazure.services.core.storage.*;
    import com.microsoft.windowsazure.services.queue.client.*;

## 设置 Azure 存储连接字符串

Azure 存储客户端使用存储连接字符串来存储用于访问数据管理服务
的终结点和凭据。在客户端应用程序中运行时，
必须提供以下格式的存储连接字符串，
并对**“AccountName”和**“AccountKey”值使用
管理门户中列出的存储帐户的名称
和存储帐户的主访问密钥。此示例演示了如何
声明一个静态字段来保存连接字符串：

    // 使用你的值定义连接字符串
    public static final String storageConnectionString = 
    "DefaultEndpointsProtocol=http;" + 
    "AccountName=your_storage_account;" + 
    "AccountKey=your_storage_account_key";

在 Azure 中的某个角色内运行的应用程序中，此字符串可
存储在服务配置文件 ServiceConfiguration.cscfg 中，
并可通过调用 RoleEnvironment.getConfigurationSettings
方法进行访问。下面是从服务配置文件中
名为 StorageConnectionString 的**设置**
元素中获取连接字符串的示例**：

    // 通过连接字符串检索存储帐户
    String storageConnectionString = 
    RoleEnvironment.getConfigurationSettings().get("StorageConnectionString");

## 如何：创建队列

利用 CloudQueueClient 对象，可以获取队列的引用对象。
以下代码将创建一个 CloudQueueClient 对象。

本指南中的所有代码都使用声明了上述两种方式之一的存储连接字符串。
还有其他方法可用来创建 CloudStorageAccount 对象。
有关详细信息，请参阅 Javadocs 文档
中的 CloudStorageAccount。

    // 通过连接字符串检索存储帐户
    CloudStorageAccount storageAccount = 
    CloudStorageAccount.parse(storageConnectionString);

    // 创建队列客户端
    CloudQueueClient queueClient = storageAccount.createCloudQueueClient();

使用 CloudQueueClient 对象获取对你要使用的队列的引用。
如果该队列不存在，你可以创建它。

    // 检索对队列的引用
    CloudQueue queue = queueClient.getQueueReference("myqueue");

    // 如果该队列不存在，则创建它
    queue.createIfNotExist();

## 如何：向队列添加消息

若要将消息插入现有队列，请先创建一个新的
CloudQueueMessage。紧接着，调用 addMessage 方法。可基于字符串（UTF-8 格式）
或字节数组创建 CloudQueueMessage。以下代码将
创建队列（如果队列不存在）
并插入消息“Hello, World”。

    // 通过连接字符串检索存储帐户
    CloudStorageAccount storageAccount = 
    CloudStorageAccount.parse(storageConnectionString);

    // 创建队列客户端
    CloudQueueClient queueClient = storageAccount.createCloudQueueClient();

    // 检索对队列的引用
    CloudQueue queue = queueClient.getQueueReference("myqueue");

    // 如果该队列不存在，则创建它
    queue.createIfNotExist();

    // 创建一条消息并将其添加到队列中
    CloudQueueMessage message = new CloudQueueMessage("Hello, World");
    queue.addMessage(message);

## 如何：扫视下一条消息

通过调用 peekMessage，你可以扫视队列前面的消息，而不会从队列中
删除它。

    // 通过连接字符串检索存储帐户
    CloudStorageAccount storageAccount = 
    CloudStorageAccount.parse(storageConnectionString);

    // 创建队列客户端
    CloudQueueClient queueClient = storageAccount.createCloudQueueClient();

    // 检索对队列的引用
    CloudQueue queue = queueClient.getQueueReference("myqueue");

    // 扫视下一条消息
    CloudQueueMessage peekedMessage = queue.peekMessage();

## 如何：取消对下一条消息的排队

你的代码通过两个步骤来取消对队列中某条消息的排队。在调用
retrieveMessage 时，你将获得队列中的下一条消息。对于
从该队列读取消息的任何其他代码，从 retrieveMessage 返回的
消息将变得不可见。默认情况下，此消息将持续 30 秒不可见。
若要从队列中删除消息，你还必须调用 deleteMessage。
此删除消息的两步过程可确保当你的代码因硬件
或软件故障而无法处理消息时，你的代码的
另一实例可以获取同一消息并重试。
你的代码在处理消息后会立即调用 deleteMessage。

    // 通过连接字符串检索存储帐户
    CloudStorageAccount storageAccount = 
    CloudStorageAccount.parse(storageConnectionString);

    // 创建队列客户端
    CloudQueueClient queueClient = storageAccount.createCloudQueueClient();

    // 检索对队列的引用
    CloudQueue queue = queueClient.getQueueReference("myqueue");

    // 检索队列中第一条可见的消息
    CloudQueueMessage retrievedMessage = queue.retrieveMessage();

    // 在不到 30 秒的时间内处理消息，然后删除消息。
    queue.deleteMessage(retrievedMessage);

## 如何：更改已排队消息的内容

你可以更改队列中现有消息的内容。如果消息
表示工作任务，则可以使用此功能来更新该工作任务的状态。
以下代码使用新内容更新队列消息，
并将可见性超时设置为再延长 60 秒。
这将保存与消息关联的工作的状态，并额外为客户端提供
一分钟的时间来继续处理消息。可使用
此方法跟踪队列消息上的多步骤工作流，
即使处理步骤因硬件或软件故障而失败，
也无需从头开始操作。通常，
你还可以保留重试计数，如果某条消息的重试次数超过 n 次，
你将删除该消息。这可避免每次处理某条消息时都触发
应用程序错误。

    // 通过连接字符串检索存储帐户
    CloudStorageAccount storageAccount = 
    CloudStorageAccount.parse(storageConnectionString);

    // 创建队列客户端
    CloudQueueClient queueClient = storageAccount.createCloudQueueClient();

    // 检索对队列的引用
    CloudQueue queue = queueClient.getQueueReference("myqueue");

    // 检索队列中第一条可见的消息
    CloudQueueMessage message = queue.retrieveMessage();

    // 修改消息内容并将其设置为在 60 秒内可见
    message.setMessageContent("Updated contents.");
    EnumSet<MessageUpdateFields> updateFields = 
    EnumSet.of(MessageUpdateFields.CONTENT, MessageUpdateFields.VISIBILITY);
    queue.updateMessage(message, 60, updateFields, null, null);

## 用于取消对消息进行排队的其他选项

你可以通过两种方式自定义队列的消息检索。
首先，你可以获取一批消息（最多 32 条）。其次，你可以
设置更长或更短的不可见超时，从而允许你的代码使用更多或
更少的时间来彻底处理每条消息。

以下代码示例使用 retrieveMessages 方法通过一次调用获取 20 条
消息。然后，它使用 **for**
 循环来处理每条消息。它还将每条消息的不可见超时时间设置为五分钟（300 秒）。
请注意，这五分钟超时对于所有消息都是同时开始的，
因此在调用 retrieveMessages 五分钟后，
尚未删除的任何消息都将再次变得
可见。

    // 通过连接字符串检索存储帐户
    CloudStorageAccount storageAccount = 
    CloudStorageAccount.parse(storageConnectionString);

    // 创建队列客户端
    CloudQueueClient queueClient = storageAccount.createCloudQueueClient();

    // 检索对队列的引用
    CloudQueue queue = queueClient.getQueueReference("myqueue");

    // 以 300 秒的可见性超时从队列中检索 20 条消息
    for (CloudQueueMessage message :queue.retrieveMessages(20, 300, null, null)) {
    // 在不到 5 分钟的时间内处理所有消息， 
    // 在处理后删除每条消息。
    queue.deleteMessage(message);
    }

## 如何：获取队列长度

你可以获取队列中消息的估计数。
downloadAttributes 方法会询问队列服务
一些当前值，包括队列中消息的计数。此计数只是
一个近似值，因为只能在队列服务响应你的请求后
添加或删除消息。getApproximateMethodCount 方法
返回通过调用 downloadAttributes 检索到的最后
一个值，而不会调用队列服务。

    // 通过连接字符串检索存储帐户
    CloudStorageAccount storageAccount = 
    CloudStorageAccount.parse(storageConnectionString);

    // 创建队列客户端
    CloudQueueClient queueClient = storageAccount.createCloudQueueClient();

    // 检索对队列的引用
    CloudQueue queue = queueClient.getQueueReference("myqueue");

    // 从服务器下载近似消息计数
    queue.downloadAttributes();

    // 检索新近缓存的近似消息计数
    long cachedMessageCount = queue.getApproximateMessageCount();

## 如何：删除队列

若要删除队列及其包含的所有消息，请对队列对象
调用 delete 方法。

    // 通过连接字符串检索存储帐户
    CloudStorageAccount storageAccount = 
    CloudStorageAccount.parse(storageConnectionString);

    // 创建队列客户端
    CloudQueueClient queueClient = storageAccount.createCloudQueueClient();

    // 检索对队列的引用
    CloudQueue queue = queueClient.getQueueReference("myqueue");

    // 删除队列
    queue.delete();

## 后续步骤

现在，你已了解有关队列存储的基础知识，可单击下面的链接来了解如何
执行更复杂的存储任务。

-   查看 MSDN 参考：[在 Windows Azure 中存储和访问
    数据]
-   访问 Azure 存储空间团队博客：<http://blogs.msdn.com/b/windowsazurestorage/>

  [Azure SDK for Java]: http://azure.microsoft.com/zh-cn/develop/java/
  [后续步骤]: #NextSteps
  [什么是队列存储]: #what-is
  [概念]: #Concepts
  [创建 Azure 存储帐户]: #CreateAccount
  [创建 Java 应用程序]: #CreateApplication
  [配置你的应用程序以访问队列存储]: #ConfigureStorage
  [设置 Azure 存储连接字符串]: #ConnectionString
  [如何：创建队列]: #create-queue
  [如何：向队列添加消息]: #add-message
  [如何：查看下一条消息]: #peek-message
  [如何：取消对下一条消息的排队]: #dequeue-message
  [如何：更改已排队消息的内容]: #change-message
  [用于对消息取消排队的其他方法]: #additional-options
  [如何：获取队列长度]: #get-queue-length
  [如何：删除队列]: #delete-queue
  [howto-queue-storage]: ../includes/howto-queue-storage.md
  [create-storage-account]: ../includes/create-storage-account.md
