#####创建队列
利用 **CloudQueueClient** 对象，可以获取队列的引用对象。以下代码将创建 **CloudQueueClient** 对象。本主题中的所有代码都使用存储在 Azure 应用程序的服务配置中的存储连接字符串。还可采用其他方法创建 **CloudStorageAccount** 对象。请参阅 [CloudStorageAccount](http://msdn.microsoft.com/zh-cn/library/microsoft.windowsazure.cloudstorageaccount_methods.aspx "CloudStorageAccount") documentation for details.

	// Create the queue client.
	CloudQueueClient queueClient = storageAccount.CreateCloudQueueClient();

使用 **queueClient** 对象获取对要使用的队列的引用。该代码将尝试引用名为"myqueue"的队列。如果找不到该名称的队列，它将创建一个。

	// Get a reference to a queue named "myqueue".
	CloudQueue queue = queueClient.GetQueueReference("myqueue");

	// If the queue isn't already there, then create it.
	queue.CreateIfNotExists();

**注意：**在接下来的部分中，将在代码的前面使用此代码块。

#####在队列中插入消息
若要将消息插入现有队列，请先创建一个新的 **CloudQueueMessage** 对象。接下来，调用 AddMessage() 方法。可从字符串（UTF-8 格式）或字节数组创建 **CloudQueueMessage** 对象。以下代码将创建队列（如果队列不存在）并插入消息  'Hello, World'。

	// Create a message and add it to the queue.
	CloudQueueMessage message = new CloudQueueMessage("Hello, World");
	queue.AddMessage(message);

#####扫视下一条消息
通过调用 PeekMessage() 方法，可以查看队列前面的消息，而不必从队列中将其删除。

	// Peek at the next message in the queue.
	CloudQueueMessage peekedMessage = queue.PeekMessage();

	// Display the message.
	Console.WriteLine(peekedMessage.AsString);

#####删除下一条消息
您的代码分两步从队列中删除消息（取消对消息的排队）。 


1. 调用 GetMessage() 以获取队列中的下一条消息。从 GetMessage() 返回的消息变得对从此队列读取消息的任何其他代码不可见。默认情况下，此消息将持续 30 秒不可见。 
2.	若要完成从队列中删除消息，请调用 DeleteMessage()。 

此删除消息的两步过程可确保当您的代码因硬件或软件故障而无法处理消息时，您的其他代码实例可以获取同一消息并重试。以下代码将在处理消息后立即调用 DeleteMessage()。

	// Get the next message in the queue.
	CloudQueueMessage retrievedMessage = queue.GetMessage();

	// Process the message in less than 30 seconds, and then delete the message.
	queue.DeleteMessage(retrievedMessage);

[了解有关 Azure 存储的详细信息](/zh-cn/documentation/services/storage)
另请参阅[在服务器资源管理器中浏览存储资源](http://msdn.microsoft.com/zh-cn/library/azure/ff683677.aspx).<!--HONumber=41-->
