<properties 
	pageTitle="针对服务总线分区队列和主题的 AMQP 1.0 支持 | Windows Azure" 
	description="了解如何将高级消息队列协议 (AMQP) 1.0 用于服务总线分区队列和主题。" 
	services="service-bus" 
	documentationCenter=".net" 
	authors="hillaryc" 
	manager="hillaryc" 
	editor="hillaryc"/>

<tags 
	ms.service="service-bus" 
	ms.date="07/21/2015" 
	wacn.date="10/22/2015"/>



# 针对服务总线分区队列和主题的 AMQP 1.0 支持 

Azure 服务总线现在支持用于 Azure 服务总线**分区队列和主题**的高级消息队列协议 (**AMQP**) 1.0。

**AMQP** 是一种开放标准消息队列协议，它允许使用不同的编程语言开发跨平台应用程序。可在[服务总线中的 AMQP 1.0 支持](/documentation/articles/service-bus-amqp-overview)中找到有关 AMQP 的服务总线常规支持的详细信息。

**分区队列和主题**（也称为分区的实体）提供更高的可用性、可靠性以及通过传统的非分区队列和主题的吞吐量。有关分区的实体的详细信息可在[消息传送实体分区](https://msdn.microsoft.com/zh-cn/library/azure/dn520246.aspx)中找到。

添加 AMQP 1.0 作为与分区的队列和主题通信的协议允许客户构建相互间可操作的应用程序，这些应用程序可利用服务总线分区实体提供的更高的可用性、可靠性和吞吐量。

### 通过分区队列使用 AMQP

对于需要临时分离、负载调配、负载平衡和松散耦合的应用场景，队列十分有用。在队列中，发布服务器将消息发送到队列，使用者从该队列接收消息，其中一条消息仅能被接收一次。一个此类的典型例子是库存系统，其中销售点终端将数据发布到一个队列而不是直接发送到库存管理系统。然后库存管理系统将随时使用数据管理库存补货。这样做有几个好处，包括库存管理系统无需在所有时间都必须处于联机状态。有关服务总线队列的更多详细信息请参阅：[创建使用服务总线队列的应用程序](https://msdn.microsoft.com/zh-cn/library/azure/hh689723.aspx)

分区的队列会进一步提高可用性、可靠性和应用程序的吞吐量，因为这些队列是跨消息中转站和消息传送存储分区的。

#### 创建分区的队列

可以通过 Azure 门户和服务总线 SDK 创建分区的队列。若要创建分区的队列，在 QueueDescription 实例中，EnablePartitioning 属性必须设置为 true。以下代码段演示如何使用服务总线 SDK 创建分区的队列。
 
	// Create partitioned queue
	var nm = NamespaceManager.CreateFromConnectionString(myConnectionString);
	var queueDescription = new QueueDescription("myQueue");
	queueDescription.EnablePartitioning = true;
	nm.CreateQueue(queueDescription);

#### 使用 AMQP 发送和接收消息传送

通过将 TransportType 设置为 TransportType.Amqp 完成使用 AMQP 作为协议发送消息到分区的队列，以及从分区的队列接收消息，如以下代码段所示。

	// Sending and receiving messages to and from a Queue
	var myConnectionStringBuilder = new ServiceBusConnectionStringBuilder(myConnectionString);
	myConnectionStringBuilder.TransportType = TransportType.Amqp;
	string amqpConnectionString = myConnectionStringBuilder.ToString();
	var queueClient = QueueClient.CreateFromConnectionString(amqpConnectionString, "myQueue");

	BrokeredMessage message = new BrokeredMessage("Hello AMQP");
	Console.WriteLine("Sending message {0}...", message.MessageId);
	queueClient.Send(message);

	var receivedMessage = queueClient.Receive();
	Console.WriteLine("Received message: {0}", receivedMessage.GetBody<string>());
	receivedMessage.Complete();


### 将 AMQP 用于分区的队列

与队列相似，对于需要临时分离、负载调配、负载平衡和松散耦合的应用场景，主题十分有用。与队列不同，主题可以路由相同消息的副本到多个订阅服务器。在主题中，发布服务器将消息发送到主题，使用者接收订阅的消息。在此示例中，库存系统销售点终端会将数据发布到主题。然后库存管理系统会从订阅接收消息。此外，监视系统可以从不同的订阅接收同一消息。有关服务总线主题的更多详细信息请参阅：[创建使用服务总线主题和订阅的应用程序](https://msdn.microsoft.com/zh-cn/library/azure/hh699844.aspx)

分区的主题会进一步提高可用性、可靠性和应用程序的吞吐量，因为这些主题及其订阅是跨多个消息中转站和消息传送存储分区的。

#### 创建分区的主题

可以通过 Azure 门户和服务总线 SDK 创建分区的主题。若要创建分区的主题，在 TopicDescription 实例中，EnablePartitioning 属性必须设置为 true。以下代码段演示如何使用服务总线 SDK 创建分区的队列。
	
	// Create partitioned topic
	var nm = NamespaceManager.CreateFromConnectionString(myConnectionString);
	var topicDescription = new TopicDescription("myTopic");
	topicDescription.EnablePartitioning = true;
	nm.CreateTopic(topicDescription);

	var subscriptionDescription = new SubscriptionDescription("myTopic", "mySubscription");
	nm.CreateSubscription(subscriptionDescription);

#### 使用 AMQP 发送和接收消息传送

通过将 TransportType 设置为 TransportType.Amqp 完成使用 AMQP 作为协议发送消息到分区的主题，以及从分区主题的订阅接收消息，如以下代码段所示。

	// Sending and receiving messages to a topic and from a subscription
	var myConnectionStringBuilder = new ServiceBusConnectionStringBuilder(myConnectionString);
	myConnectionStringBuilder.TransportType = TransportType.Amqp;
	string amqpConnectionString = myConnectionStringBuilder.ToString();
	
	var topicClient = TopicClient.CreateFromConnectionString(amqpConnectionString, "myTopic");
	BrokeredMessage message = new BrokeredMessage("Hello AMQP");
	Console.WriteLine("Sending message {0}...", message.MessageId);
	topicClient.Send(message);
	
	var subcriptionClient = SubscriptionClient.CreateFromConnectionString(amqpConnectionString, "myTopic", "mySubscription");
	var receivedMessage = subcriptionClient.Receive();
	Console.WriteLine("Received message: {0}", receivedMessage.GetBody<string>());
	receivedMessage.Complete();


## 参考

*    [分区消息实体](https://msdn.microsoft.com/zh-cn/library/azure/dn520246.aspx)
*    [OASIS 高级消息队列协议 (AMQP) 1.0 版](http://docs.oasis-open.org/amqp/core/v1.0/os/amqp-core-complete-v1.0-os.pdf)
*    [服务总线的 AMQP 1.0 支持](/documentation/articles/service-bus-amqp-overview)
*    [服务总线 AMQP：开发人员指南](https://msdn.microsoft.com/zh-cn/library/azure/jj841071.aspx)
*    [如何将 Java 消息服务 (JMS) API 用于服务总线和 AMQP 1.0](/documentation/articles/service-bus-java-how-to-use-jms-api-amqp)
*    [如何将 AMQP 1.0 与服务总线 .NET API 一起使用](/documentation/articles/service-bus-dotnet-advanced-message-queuing)

<!---HONumber=74-->