<properties 
   pageTitle="服务总线队列、主题和订阅 | Microsoft Azure"
   description="服务总线消息传送实例概述。"
   services="service-bus"
   documentationCenter="na"
   authors="sethmanheim"
   manager="timlt"
   editor="tysonn" />
<tags 
   ms.service="service-bus"
   ms.date="03/09/2016"
   wacn.date="03/28/2016" />

# 服务总线队列、主题和订阅

Microsoft Azure 服务总线支持一组基于云的、面向消息的中间件技术，包括可靠的消息队列和持久的发布/订阅消息。这些中转消息传送功能可被视为分离式消息传送功能，支持使用服务总线消息传送结构的发布-订阅、临时分离和负载平衡方案。分离式通信具有很多优点；例如，客户端和服务器可以根据需要进行连接并以异步方式执行其操作。

消息传送实体（队列、主题/订阅、规则/操作和事件中心）是服务总线中转消息传送功能的核心。

## 队列

队列为一个或多个竞争使用方提供“先入先出 (FIFO)”消息传递方式。也就是说，接收方预计会按照消息添加到队列中的顺序来接收并处理消息，并且每条消息仅由一个消息使用方接收并处理。使用队列的主要优点是，实现应用程序组件的“临时分离”。换而言之，由于消息持久存储在队列中，因此创建方（发送方）和使用方（接收方）不必同时发送和接收消息。此外，创建方不必等待使用方的答复即可继续处理并发送更多消息。

相关的优点是“负载分级”，它允许创建方和使用方以不同速率发送和接收消息。在许多应用程序中，系统负载随时间而变化，而每个工作单元所需的处理时间通常为常量。使用队列在消息创建者与使用者之间中继意味着，只需将使用方应用程序预配为适应平均负载而非最大负载。队列深度将随传入负载的变化而加大和减小。这将直接根据为应用程序加载提供服务所需的基础结构的数目来节省成本。随着负载增加，可添加更多的工作进程以从队列中读取。每条消息仅由一个辅助进程处理。另外，可通过此基于拉取的负载平衡来以最合理的方式使用辅助计算机，即使这些辅助计算机具有不同的处理能力（因为它们将以其最大速率拉取消息）也是如此。此模式通常称为“使用者竞争模式”。

使用队列在消息创建方与使用方之间中继可在个组件之间提供固有的松散耦合。由于创建方和使用方互不相识，因此，可升级使用方，不会对创建方产生任何影响。

创建队列是一个多步骤过程。你可以通过 [Microsoft.ServiceBus.NamespaceManager](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.namespacemanager.aspx) 类执行服务总线消息传送实例（队列和主题）的管理操作，该类可通过提供服务总线命名空间的基址和用户凭据进行构建。[NamespaceManager](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.namespacemanager.aspx) 提供了创建、枚举和删除消息传送实体的方法。在使用 SAS 名称和密钥创建 [Microsoft.ServiceBus.TokenProvider](https://msdn.microsoft.com/library/azure/microsoft.servicebus.tokenprovider.aspx) 对象（服务命名空间管理对象）之后，你可以使用 [Microsoft.ServiceBus.NamespaceManager.CreateQueue](https://msdn.microsoft.com/zh-cn/library/azure/hh293157.aspx) 方法以创建队列。例如：

```
// Create management credentials
TokenProvider credentials = TokenProvider. CreateSharedAccessSignatureTokenProvider(sasKeyName,sasKeyValue);
// Create namespace client
NamespaceManager namespaceClient = new NamespaceManager(ServiceBusEnvironment.CreateServiceUri("sb", ServiceNamespace, string.Empty), credentials);
```

你可以随后创建一个队列对象和消息工厂，将服务总线 URI 用作参数。例如：

```
QueueDescription myQueue;
myQueue = namespaceClient.CreateQueue("TestQueue");
MessagingFactory factory = MessagingFactory.Create(ServiceBusEnvironment.CreateServiceUri("sb", ServiceNamespace, string.Empty), credentials); 
QueueClient myQueueClient = factory.CreateQueueClient("TestQueue");
```

你可以随后向队列发送消息。例如，如果具有名为 `MessageList` 的中转消息列表，将出现此代码，类似如下形式：

```
for (int count = 0; count < 6; count++)
{
    var issue = MessageList[count];
    issue.Label = issue.Properties["IssueTitle"].ToString();
    myQueueClient.Send(issue);
}
```

你可以随后接收来自队列的消息，如下所示：

```
while ((message = myQueueClient.Receive(new TimeSpan(hours: 0, minutes: 0, seconds: 5))) != null)
    {
        Console.WriteLine(string.Format("Message received: {0}, {1}, {2}", message.SequenceNumber, message.Label, message.MessageId));
        message.Complete();

        Console.WriteLine("Processing message (sleeping...)");
        Thread.Sleep(1000);
    }
```

当使用 [ReceiveAndDelete](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.receivemode.aspx) 模式时，接收操作是一个单一快照。即，当服务总线收到请求时，它会将该消息标记为“已使用”并将其返回给应用程序。[ReceiveAndDelete](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.receivemode.aspx) 模式是最简单的模式，最适合应用程序允许出现故障时不处理消息的方案。为了理解这一点，可以考虑这样一种情形：使用方发出接收请求，但在处理该请求前发生了崩溃。由于服务总线会将消息标记为“已使用”，因此当应用程序重新启动并重新开始使用消息时，它会漏掉在发生崩溃前使用的消息。

当使用 [PeekLock](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.receivemode.aspx) 模式时，接收操作分成了两步，从而有可能支持无法容忍遗漏消息的应用程序。当服务总线收到请求时，它会找到要使用的下一个消息，将其锁定以防其他使用方接收它，然后将该消息返回给应用程序。应用程序完成消息处理（或可靠地存储消息以供将来处理）后，它将通过对收到的消息调用 [Complete](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.brokeredmessage.complete.aspx) 完成接收过程的第二个阶段。当服务总线发现 [Complete](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.brokeredmessage.complete.aspx) 调用时，它会将消息标记为“已使用”。

如果应用程序出于某种原因无法处理消息，它可以对收到的消息调用 [Abandon](https://msdn.microsoft.com/zh-cn/library/azure/hh181837.aspx) 方法（而不是 [Complete](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.brokeredmessage.complete.aspx) 方法）。这可使服务总线解锁消息并使其能够重新被同一个使用方或其他竞争使用方接收。此外，还存在与锁定关联的超时，并且如果应用程序无法在锁定超时到期之前处理消息（例如，如果应用程序崩溃），服务总线将解锁该消息并使它可再次被接收。

请注意，如果应用程序在处理消息之后，但在发出 [Complete](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.brokeredmessage.complete.aspx) 请求之前发生崩溃，则在应用程序重新启动时会将该消息重新传送给它。此情况通常称作*至少处理一次*，即每条消息将至少被处理一次。但是，在某些情况下，同一消息可能会被重新传送。如果方案不容许重复处理，则应用程序中需要用于检测重复的其他逻辑，此重复可基于消息的 **MessageId** 属性实现，无论传送次数多少，均保持不变。这称为*一次性*处理。

有关如何创建和将消息发送至队列以及从队列发送消息的详细信息和操作示例说明，请参阅[服务总线中转消息传送 .NET 教程](https://msdn.microsoft.com/zh-cn/library/azure/hh367512.aspx)。

## 主题和订阅

与每条消息由单个使用方处理的队列相比，主题和订阅通过*发布/订阅*模式提供“一对多”通信方式。这对于扩展到大量接收方而言十分有用，每个发布的消息对向该主题注册的每个订阅均可用。系统会将消息发送到主题并传递到一个或多个相关联的订阅，具体取决于每个订阅上可以设置的筛选规则。此订阅可以使用其他筛选器来限制其想要接收的消息。可以采用与发送至队列的相同方式将消息发送至主题，但不可直接从主题接收消息。而是从订阅接收消息。主题订阅类似于接收发送至该主题的消息副本的虚拟队列。从订阅接收消息的方式与从队列接收相同。

通过比较，队列的消息发送功能直接映射到主题，而其消息接收功能映射到订阅。此外，这意味着订阅支持本部分前面所述的关于队列的相同模式：竞争使用方、临时分离、负载分级和负载平衡。

创建主题类似于创建队列，如前一节中的示例所示。创建服务 URI，然后使用 [NamespaceManager](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.namespacemanager.aspx) 类来创建命名空间客户端。然后，你可以使用 [CreateTopic](https://msdn.microsoft.com/zh-cn/library/azure/hh293080.aspx) 方法创建主题。例如：

```
TopicDescription dataCollectionTopic = namespaceClient.CreateTopic("DataCollectionTopic");
```

接下来，根据需要添加订阅：

```
SubscriptionDescription myAgentSubscription = namespaceClient.CreateSubscription(myTopic.Path, "Inventory");
SubscriptionDescription myAuditSubscription = namespaceClient.CreateSubscription(myTopic.Path, "Dashboard");
```

然后可以创建主题客户端。例如：

```
MessagingFactory factory = MessagingFactory.Create(serviceUri, tokenProvider);
TopicClient myTopicClient = factory.CreateTopicClient(myTopic.Path)
```

通过消息发送方，你可以将消息发送至主题和从主题接收消息，如上一节所述。例如：

```
foreach (BrokeredMessage message in messageList)
{
    myTopicClient.Send(message);
    Console.WriteLine(
    string.Format("Message sent: Id = {0}, Body = {1}", message.MessageId, message.GetBody<string>()));
}
```

与队列类似，可使用 [SubscriptionClient](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.subscriptionclient.aspx) 对象而不是 [QueueClient](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.queueclient.aspx) 对象接收来自订阅的消息。创建订阅客户端，将主题的名称、订阅的名称和（可选）接收模式作为参数传递。例如，对于“库存”订阅：

```
// Create the subscription client
MessagingFactory factory = MessagingFactory.Create(serviceUri, tokenProvider); 

SubscriptionClient agentSubscriptionClient = factory.CreateSubscriptionClient("IssueTrackingTopic", "Inventory", ReceiveMode.PeekLock);
SubscriptionClient auditSubscriptionClient = factory.CreateSubscriptionClient("IssueTrackingTopic", "Dashboard", ReceiveMode.ReceiveAndDelete); 

while ((message = agentSubscriptionClient.Receive(TimeSpan.FromSeconds(5))) != null)
{
    Console.WriteLine("\nReceiving message from Inventory...");
    Console.WriteLine(string.Format("Message received: Id = {0}, Body = {1}", message.MessageId, message.GetBody<string>()));
    message.Complete();
}          

// Create a receiver using ReceiveAndDelete mode
while ((message = auditSubscriptionClient.Receive(TimeSpan.FromSeconds(5))) != null)
{
    Console.WriteLine("\nReceiving message from Dashboard...");
    Console.WriteLine(string.Format("Message received: Id = {0}, Body = {1}", message.MessageId, message.GetBody<string>()));
}
```

### 规则和操作

在许多情况下，必须以不同方式处理具有特定特征的消息。若要启用此功能，你可以将订阅配置为查找具有所需属性的消息，然后执行这些属性的部分修改操作。虽然服务总线订阅可以看到发送到主题的所有消息，但你也可以仅将这些消息的一个子集复制到虚拟订阅队列。使用订阅筛选器完成此操作。此类修改称为*筛选器操作*。在创建订阅后，你可以提供可对消息的属性（例如，“标签”）和自定义应用程序属性（例如，**StoreName**）进行操作的筛选器表达式。 SQL 筛选器在此示例中为可选；如果没有 SQL 筛选器表达式，将对该订阅的所有消息执行在订阅上定义的任何筛选器操作。

使用上述示例，要仅筛选来自 **Store1** 的消息，如下所示创建“仪表板”订阅：

```
namespaceManager.CreateSubscription("IssueTrackingTopic", "Dashboard", new SqlFilter("StoreName = 'Store1'"));
```

通过此订阅筛选器，只有 `StoreName` 属性设置为 `Store1` 的消息将复制到 `Dashboard` 订阅的虚拟队列。

有关可能的筛选器值的详细信息，请参阅文档 [SqlFilter](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.sqlfilter.aspx) 和 [SqlRuleAction](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.sqlruleaction.aspx) 类。另请参阅[中转消息传送：高级筛选器](http://code.msdn.microsoft.com/Brokered-Messaging-6b0d2749)示例。

## 事件中心

[事件中心](/home/features/event-hubs/)是一种事件处理服务，用于向 Azure 提供大规模的事件与遥测数据入口，并且具有较低的延迟和较高的可靠性。在应用程序检测、用户体验或工作流处理以及物联网 (IoT) 方案中，将此服务与其他下游服务结合使用可以带来极好的效果。

事件中心是消息流式处理构造，尽管它们看上去类似于队列和主题，但其特征却迥然不同。例如，事件中心不提供消息 TTL、死信、事务或确认，因为这些是传统中转消息传送功能，而是流处理功能。事件中心可提供其他与流相关的功能（例如分区、保留顺序和流重播）。

## 后续步骤

有关使用服务总线中转消息传送实例的详细信息，请参阅以下高级主题。

- [服务总线消息传送概述](/documentation/articles/service-bus-messaging-overview/)
- [服务总线中转消息传送 .NET 教程](/documentation/articles/service-bus-brokered-tutorial-dotnet/)
- [服务总线中转消息传送 REST 教程](/documentation/articles/service-bus-brokered-tutorial-rest/)
- [事件中心开发人员指南](/documentation/articles/event-hubs-programming-guide/)
- [中转消息传送：高级筛选器](http://code.msdn.microsoft.com/Brokered-Messaging-6b0d2749)

<!---HONumber=Mooncake_0104_2016-->