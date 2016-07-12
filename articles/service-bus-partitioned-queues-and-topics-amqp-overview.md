<properties 
	pageTitle="针对服务总线分区队列和主题的 AMQP 1.0 支持 | Microsoft Azure" 
	description="了解如何将高级消息队列协议 (AMQP) 1.0 用于服务总线分区队列和主题。" 
	services="service-bus" 
	documentationCenter=".net" 
	authors="hillaryc" 
	manager="timlt" 
	editor=""/>

<tags 
	ms.service="service-bus" 
	ms.date="03/16/2016" 
	wacn.date="04/05/2016"/>



# 针对服务总线分区队列和主题的 AMQP 1.0 支持 

Azure 服务总线现在支持用于服务总线**分区队列和主题**的高级消息队列协议 (**AMQP**) 1.0。

**AMQP** 是一种开放标准消息队列协议，它允许使用不同的编程语言开发跨平台应用程序。有关服务总线的 AMQP 支持的常规信息，请参阅[服务总线的 AMQP 1.0 支持](/documentation/articles/service-bus-amqp-overview/)。

**分区队列和主题**也称为*分区实体*，相比传统的非分区队列和主题，提供更高可用性、可靠性和吞吐量。有关分区实体的详细信息，请参阅[分区消息实体](/documentation/articles/service-bus-partitioning/)。

添加 AMQP 1.0 作为与分区队列和主题通信的协议使你能够构建相互间可操作的应用程序，这些应用程序可利用服务总线分区实体提供的更高可用性、可靠性和吞吐量。

## 将 AMQP 用于分区队列

对于需要临时分离、负载调配、负载平衡和松散耦合的应用场景，队列十分有用。在队列中，发布服务器将消息发送到队列，使用者从该队列接收消息，在这种情况下一条消息仅能被接收一次。此类的一个很好例子是库存系统，其中销售点终端将数据发布到一个队列而不是直接发送到库存管理系统。然后库存管理系统将随时使用该数据管理库存补货。这样做有几个好处，包括库存管理系统无需在所有时间都必须处于联机状态。有关服务总线队列的更多详细信息，请参阅[创建使用服务总线队列的应用程序](/documentation/articles/service-bus-create-queues/)

分区队列会进一步提高应用程序的可用性、可靠性和吞吐量，因为这些队列是跨多个消息中转站和消息存储分区的。

### 创建分区队列

可以使用 [Azure 经典门户][]和服务总线 SDK 创建分区队列。若要创建分区队列，请在 [QueueDescription](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.queuedescription.aspx) 实例中将 [EnablePartitioning](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.queuedescription.enablepartitioning.aspx) 属性设置为 **true**。以下代码演示如何使用服务总线 SDK 创建分区队列。
 
```
// Create partitioned queue
var nm = NamespaceManager.CreateFromConnectionString(myConnectionString);
var queueDescription = new QueueDescription("myQueue");
queueDescription.EnablePartitioning = true;
nm.CreateQueue(queueDescription);
```

### 使用 AMQP 发送和接收消息

通过将 [TransportType](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.servicebusconnectionstringbuilder.transporttype.aspx) 属性设置为 [TransportType.Amqp](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.transporttype.aspx) 可以使用 AMQP 作为协议发送消息到分区队列，以及从分区队列接收消息，如以下代码所示。

```
// Sending and receiving messages to and from a queue
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
```

## 将 AMQP 用于分区主题

主题在概念上类似于队列，但主题可以将同一消息的副本路由到多个订阅服务器。在主题中，发布服务器将消息发送到主题，使用者从订阅接收消息。在库存系统销售点方案中，终端会将数据发布到主题。然后，库存管理系统会从订阅接收消息。此外，监视系统可以从不同的订阅接收同一消息。有关服务总线主题和订阅的更多详细信息，请参阅[创建使用服务总线主题和订阅的应用程序](/documentation/articles/service-bus-create-topics-subscriptions/)

分区主题会进一步提高应用程序的可用性、可靠性和吞吐量，因为这些主题及其订阅是跨多个消息中转站和消息存储分区的。

### 创建分区主题

可以通过 [Azure 管理门户][]和服务总线 SDK 创建分区主题。若要创建分区主题，请在 [TopicDescription](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.topicdescription.aspx) 实例中将 [EnablePartitioning](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.topicdescription.enablepartitioning.aspx) 属性设置为 **true**。以下代码演示如何使用服务总线 SDK 创建分区主题。
	
```
// Create partitioned topic
var nm = NamespaceManager.CreateFromConnectionString(myConnectionString);
var topicDescription = new TopicDescription("myTopic");
topicDescription.EnablePartitioning = true;
nm.CreateTopic(topicDescription);

var subscriptionDescription = new SubscriptionDescription("myTopic", "mySubscription");
nm.CreateSubscription(subscriptionDescription);
```

### 使用 AMQP 发送和接收消息

通过将 [TransportType](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.servicebusconnectionstringbuilder.transporttype.aspx) 属性设置为 [TransportType.Amqp](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.transporttype.aspx) 可以使用 AMQP 作为协议发送消息到分区主题订阅，以及从分区主题订阅接收消息，如以下代码所示。

```
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
```

## 后续步骤

请参阅以下附加信息来了解有关分区消息实体的更多信息。

*    [分区消息实体](/documentation/articles/service-bus-partitioning/)
*    [OASIS 高级消息队列协议 (AMQP) 1.0 版](http://docs.oasis-open.org/amqp/core/v1.0/os/amqp-core-complete-v1.0-os.pdf)
*    [服务总线的 AMQP 1.0 支持](/documentation/articles/service-bus-amqp-overview/)
*    [如何将 Java 消息服务 (JMS) API 用于服务总线和 AMQP 1.0](/documentation/articles/service-bus-java-how-to-use-jms-api-amqp/)
*    [如何将 AMQP 1.0 与服务总线 .NET API 一起使用](/documentation/articles/service-bus-dotnet-advanced-message-queuing/)

[Azure 管理门户]: http://manage.windowsazure.cn

<!---HONumber=Mooncake_0104_2016-->