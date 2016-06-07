<properties 
	pageTitle="如何通过 Java 使用队列存储 | Azure" 
	description="了解如何使用 Azure 队列服务创建和删除队列，以及插入、获取和删除消息。用 Java 编写的示例。" 
	services="storage" 
	documentationCenter="java" 
	authors="rmcmurray" 
	manager="wpickett" 
	editor="jimbe"/>

<tags 
	ms.service="storage" 
	ms.date="05/04/2016"
	wacn.date="06/06/2016"/>

# 如何通过 Java 使用队列存储

[AZURE.INCLUDE [storage-selector-queue-include](../includes/storage-selector-queue-include.md)]

## 概述

本指南将演示如何使用 Azure 队列存储服务执行常见方案。这些示例用 Java 编写并使用 [Azure Storage SDK for Java][]。涉及的方案包括“插入”、“扫视”、“获取”和“删除”队列消息以及“创建”和“删除”队列。有关队列的详细信息，请参阅[后续步骤](#Next-Steps)部分。

注意：为在 Android 设备上使用 Azure 存储的开发人员提供了 SDK。有关详细信息，请参阅 [Azure Storage SDK for Android][]。

[AZURE.INCLUDE [storage-queue-concepts-include](../includes/storage-queue-concepts-include.md)]

[AZURE.INCLUDE [storage-create-account-include](../includes/storage-create-account-include.md)]

## 创建 Java 应用程序

在本指南中，你将使用存储功能，这些功能可在本地 Java 应用程序中运行，或在 Azure 的 Web 角色或辅助角色中通过运行的代码来运行。

为此，你将需要安装 Java 开发工具包 (JDK)，并在你的 Azure 订阅中创建一个 Azure 存储帐户。完成此操作后，你将需要验证开发系统满足最低要求和 GitHub 上的 [Azure Storage SDK for Java][] 存储库中列出的依赖项。如果你的系统满足这些要求，你可以按照说明下载和安装系统中该存储库的 Azure Storage Libraries for Java。完成这些任务后，您将能够创建一个 Java 应用程序，以便使用本文中的示例。

## 配置应用程序以访问队列存储

将下列 import 语句添加到需要在其中使用 Azure 存储 API 来访问队列的 Java 文件的顶部：

    // Include the following imports to use queue APIs.
    import com.microsoft.azure.storage.*;
    import com.microsoft.azure.storage.queue.*;

## 设置 Azure 存储连接字符串

Azure 存储客户端使用存储连接字符串来存储用于访问数据管理服务的终结点和凭据。在客户端应用程序中运行时，必须提供以下格式的存储连接字符串，并对 AccountName 和 AccountKey 值使用[管理门户](https://manage.windowsazure.cn)中列出的存储帐户的名称和存储帐户的主访问密钥。此示例演示如何声明一个静态字段以保存连接字符串：

    // Define the connection-string with your values.
    public static final String storageConnectionString = 
        "DefaultEndpointsProtocol=http;" + 
        "AccountName=your_storage_account;" + 
        "AccountKey=your_storage_account_key;" +
	"EndpointSuffix=core.chinacloudapi.cn";

在 Azure 的角色中运行的应用程序中，此字符串可存储在服务配置文件 *ServiceConfiguration.cscfg* 中，并可通过调用 **RoleEnvironment.getConfigurationSettings** 方法进行访问。下面是从服务配置文件中名为 *StorageConnectionString* 的 **Setting** 元素中获取连接字符串的示例：

    // Retrieve storage account from connection-string.
    String storageConnectionString = 
        RoleEnvironment.getConfigurationSettings().get("StorageConnectionString");

下面的示例假定你使用了这两个方法之一来获取存储连接字符串。

## 如何：创建队列

利用 **CloudQueueClient** 对象，可以获取队列的引用对象。以下代码将创建 **CloudQueueClient** 对象。（注意：还有其他方式来创建 **CloudStorageAccount** 对象；有关详细信息，请参阅 **Azure 存储客户端 SDK 参考**中的 [CloudStorageAccount]。）

使用 **CloudQueueClient** 对象获取对你要使用的队列的引用。如果队列不存在，你可以创建它。

    try
    {
    	// Retrieve storage account from connection-string.
    	CloudStorageAccount storageAccount = 
	       CloudStorageAccount.parse(storageConnectionString);

	   // Create the queue client.
	   CloudQueueClient queueClient = storageAccount.createCloudQueueClient();

	   // Retrieve a reference to a queue.
	   CloudQueue queue = queueClient.getQueueReference("myqueue");

	   // Create the queue if it doesn't already exist.
	   queue.createIfNotExists();
    }
    catch (Exception e)
    {
        // Output the stack trace.
        e.printStackTrace();
    }

## 如何：向队列添加消息

若要将消息插入现有队列，请先创建一个新的 **CloudQueueMessage**。接下来，调用 **addMessage** 方法。可从字符串（UTF-8 格式）或字节数组创建 **CloudQueueMessage**。以下代码将创建队列（如果队列不存在）并插入消息“Hello, World”。

    try
    {
    	// Retrieve storage account from connection-string.
    	CloudStorageAccount storageAccount = 
	       CloudStorageAccount.parse(storageConnectionString);

    	// Create the queue client.
    	CloudQueueClient queueClient = storageAccount.createCloudQueueClient();

    	// Retrieve a reference to a queue.
    	CloudQueue queue = queueClient.getQueueReference("myqueue");

    	// Create the queue if it doesn't already exist.
    	queue.createIfNotExists();

    	// Create a message and add it to the queue.
    	CloudQueueMessage message = new CloudQueueMessage("Hello, World");
    	queue.addMessage(message);
    }
    catch (Exception e)
    {
        // Output the stack trace.
        e.printStackTrace();
    }

## 如何：扫视下一条消息

通过调用 **peekMessage**，你可以扫视队列前面的消息，而不会从队列中删除它。

    try
    {
    	// Retrieve storage account from connection-string.
    	CloudStorageAccount storageAccount = 
	       CloudStorageAccount.parse(storageConnectionString);

    	// Create the queue client.
    	CloudQueueClient queueClient = storageAccount.createCloudQueueClient();

    	// Retrieve a reference to a queue.
    	CloudQueue queue = queueClient.getQueueReference("myqueue");
			
    	// Peek at the next message.
    	CloudQueueMessage peekedMessage = queue.peekMessage();
			
    	// Output the message value.
    	if (peekedMessage != null)
    	{
		  System.out.println(peekedMessage.getMessageContentAsString());
	   }
    }
    catch (Exception e)
    {
        // Output the stack trace.
        e.printStackTrace();
    }

## 如何：更改已排队消息的内容

你可以更改队列中现有消息的内容。如果消息表示工作任务，则你可以使用此功能来更新该工作任务的状态。以下代码使用新内容更新队列消息，并将可见性超时设置为再延长 60 秒。这将保存与消息关联的工作的状态，并额外为客户端提供一分钟的时间来继续处理消息。可使用此方法跟踪队列消息上的多步骤工作流，即使处理步骤因硬件或软件故障而失败，也无需从头开始操作。通常，你还可以保留重试计数，如果某条消息的重试次数超过 *n*，你将删除此消息。这可避免每次处理某条消息时都触发应用程序错误。

下面的代码示例将搜索队列中的消息，查找内容中第一个与“Hello, World”匹配的消息，然后对消息内容进行修改并退出。

    try
    {
        // Retrieve storage account from connection-string.
        CloudStorageAccount storageAccount = 
            CloudStorageAccount.parse(storageConnectionString);

    	// Create the queue client.
    	CloudQueueClient queueClient = storageAccount.createCloudQueueClient();

    	// Retrieve a reference to a queue.
    	CloudQueue queue = queueClient.getQueueReference("myqueue");

        // The maximum number of messages that can be retrieved is 32. 
        final int MAX_NUMBER_OF_MESSAGES_TO_PEEK = 32;

        // Loop through the messages in the queue.
        for (CloudQueueMessage message : queue.retrieveMessages(MAX_NUMBER_OF_MESSAGES_TO_PEEK,1,null,null))
        {
            // Check for a specific string.
            if (message.getMessageContentAsString().equals("Hello, World"))
            {
                // Modify the content of the first matching message.
                message.setMessageContent("Updated contents.");
                // Set it to be visible in 30 seconds.
                EnumSet<MessageUpdateFields> updateFields = 
                    EnumSet.of(MessageUpdateFields.CONTENT,
                    MessageUpdateFields.VISIBILITY);
                // Update the message.
                queue.updateMessage(message, 30, updateFields, null, null);
                break;
            }
        }
    }
    catch (Exception e)
    {
        // Output the stack trace.
        e.printStackTrace();
    }

或者，以下代码示例只更新了队列中第一个可见消息

    try
    {
    	// Retrieve storage account from connection-string.
    	CloudStorageAccount storageAccount = 
	       CloudStorageAccount.parse(storageConnectionString);

    	// Create the queue client.
    	CloudQueueClient queueClient = storageAccount.createCloudQueueClient();

    	// Retrieve a reference to a queue.
    	CloudQueue queue = queueClient.getQueueReference("myqueue");

    	// Retrieve the first visible message in the queue.
    	CloudQueueMessage message = queue.retrieveMessage();
			
    	if (message != null)
    	{
            // Modify the message content.
            message.setMessageContent("Updated contents.");
            // Set it to be visible in 60 seconds.
            EnumSet<MessageUpdateFields> updateFields = 
                EnumSet.of(MessageUpdateFields.CONTENT,
                MessageUpdateFields.VISIBILITY);
            // Update the message.
            queue.updateMessage(message, 60, updateFields, null, null);
    	}
    }
    catch (Exception e)
    {
        // Output the stack trace.
        e.printStackTrace();
    }

## 如何：获取队列长度

你可以获取队列中消息的估计数。**downloadAttributes** 方法会询问队列服务一些当前值，包括队列中消息的计数。此计数仅为近似值，因为只能在队列服务响应您的请求后添加或删除消息。**getApproximateMessageCount** 方法返回通过调用 **downloadAttributes** 检索到的最后一个值，而不会调用队列服务。

    try
    {
    	// Retrieve storage account from connection-string.
    	CloudStorageAccount storageAccount = 
	       CloudStorageAccount.parse(storageConnectionString);

    	// Create the queue client.
    	CloudQueueClient queueClient = storageAccount.createCloudQueueClient();

    	// Retrieve a reference to a queue.
    	CloudQueue queue = queueClient.getQueueReference("myqueue");

	   // Download the approximate message count from the server.
    	queue.downloadAttributes();

    	// Retrieve the newly cached approximate message count.
    	long cachedMessageCount = queue.getApproximateMessageCount();
			
    	// Display the queue length.
    	System.out.println(String.format("Queue length: %d", cachedMessageCount));
    }
    catch (Exception e)
    {
        // Output the stack trace.
        e.printStackTrace();
    }

## 如何：取消对下一条消息的排队

你的代码通过两个步骤来取消对队列中某条消息的排队。在调用 **retrieveMessage** 时，你将获得队列中的下一条消息。从 **retrieveMessage** 返回的消息变得对从此队列读取消息的任何其他代码不可见。默认情况下，此消息将持续 30 秒不可见。若要从队列中删除消息，你还必须调用 **deleteMessage**。此删除消息的两步过程可确保，如果你的代码因硬件或软件故障而无法处理消息，则你的代码的其他实例可以获取相同消息并重试。你的代码在处理消息后会立即调用 **deleteMessage**。

    try
    {
    	// Retrieve storage account from connection-string.
    	CloudStorageAccount storageAccount = 
    	    CloudStorageAccount.parse(storageConnectionString);

    	// Create the queue client.
    	CloudQueueClient queueClient = storageAccount.createCloudQueueClient();

    	// Retrieve a reference to a queue.
    	CloudQueue queue = queueClient.getQueueReference("myqueue");

    	// Retrieve the first visible message in the queue.
    	CloudQueueMessage retrievedMessage = queue.retrieveMessage();
			
    	if (retrievedMessage != null)
    	{
    		// Process the message in less than 30 seconds, and then delete the message.
    		queue.deleteMessage(retrievedMessage);
    	}
    }
    catch (Exception e)
    {
        // Output the stack trace.
        e.printStackTrace();
    }


## 用于取消对消息进行排队的其他选项

你可以通过两种方式自定义队列中的消息检索。首先，你可以获取一批消息（最多 32 个）。其次，你可以设置更长或更短的不可见超时时间，从而允许你的代码使用更多或更少时间来完全处理每个消息。

下面的代码示例使用 **retrieveMessages** 方法以在一次调用中获取 20 条消息。然后，它会使用 **for** 循环处理每条消息。它还将每条消息的不可见超时设置为五分钟（300 秒）。请注意，这五分钟超时对于所有消息都是同时开始的，因此在调用 **retrieveMessages** 五分钟后，尚未删除的任何消息都将再次变得可见。

    try
    {
        // Retrieve storage account from connection-string.
        CloudStorageAccount storageAccount = 
            CloudStorageAccount.parse(storageConnectionString);

        // Create the queue client.
        CloudQueueClient queueClient = storageAccount.createCloudQueueClient();

        // Retrieve a reference to a queue.
        CloudQueue queue = queueClient.getQueueReference("myqueue");

        // Retrieve 20 messages from the queue with a visibility timeout of 300 seconds.
        for (CloudQueueMessage message : queue.retrieveMessages(20, 300, null, null)) {
            // Do processing for all messages in less than 5 minutes, 
            // deleting each message after processing.
            queue.deleteMessage(message);
        }
    }
    catch (Exception e)
    {
        // Output the stack trace.
        e.printStackTrace();
    }

## 如何：列出队列

若要获取当前队列的列表，请调用 **CloudQueueClient.listQueues()** 方法，它将返回 **CloudQueue** 对象的集合。

    try
    {
        // Retrieve storage account from connection-string.
        CloudStorageAccount storageAccount =
            CloudStorageAccount.parse(storageConnectionString);

        // Create the queue client.
        CloudQueueClient queueClient =
            storageAccount.createCloudQueueClient();

        // Loop through the collection of queues.
        for (CloudQueue queue : queueClient.listQueues())
        {
            // Output each queue name.
            System.out.println(queue.getName());
        }
    }
    catch (Exception e)
    {
        // Output the stack trace.
        e.printStackTrace();
    }

## 如何：删除队列

若要删除队列及其包含的所有消息，请对 **CloudQueue** 对象调用 **deleteIfExists** 方法。

    try
    {
        // Retrieve storage account from connection-string.
        CloudStorageAccount storageAccount = 
            CloudStorageAccount.parse(storageConnectionString);

        // Create the queue client.
        CloudQueueClient queueClient = storageAccount.createCloudQueueClient();

        // Retrieve a reference to a queue.
        CloudQueue queue = queueClient.getQueueReference("myqueue");

        // Delete the queue if it exists.
        queue.deleteIfExists();
    }
    catch (Exception e)
    {
        // Output the stack trace.
        e.printStackTrace();
    }

##<a id="Next-Steps"></a> 后续步骤

现在，您已了解有关队列存储的基础知识，可单击下面的链接来了解更复杂的存储任务。

- [Azure Storage SDK for Java][]
- [Azure 存储客户端 SDK 参考][]
- [Azure 存储 REST API][]
- [Azure 存储团队博客][]

[Azure SDK for Java]: /develop/java/
[Azure Storage SDK for Java]: https://github.com/azure/azure-storage-java
[Azure Storage SDK for Android]: https://github.com/azure/azure-storage-android
[Azure 存储客户端 SDK 参考]: http://azure.github.io/azure-storage-java/
[CloudStorageAccount]: http://azure.github.io/azure-storage-java/
[Azure 存储 REST API]: https://msdn.microsoft.com/zh-cn/library/azure/dd179355.aspx
[Azure 存储团队博客]: http://blogs.msdn.com/b/windowsazurestorage/

<!---HONumber=Mooncake_0530_2016-->