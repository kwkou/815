<properties
    pageTitle="Azure 服务总线消息传送队列、主题和订阅概述 | Azure"
    description="服务总线消息传送实体概述。"
    services="service-bus"
    documentationCenter="na"
    author="sethmanheim"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="a306ced4-74e9-47c6-990a-d9c47efa31d5"
    ms.service="service-bus"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="03/23/2017"
    wacn.date="05/22/2017"
    ms.author="sethm"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="8fd60f0e1095add1bff99de28a0b65a8662ce661"
    ms.openlocfilehash="eb79ca0f6de9a1861f241c112ba5adb8fb206334"
    ms.lasthandoff="05/12/2017" />

# <a name="service-bus-queues-topics-and-subscriptions"></a>服务总线队列、主题和订阅
Azure 服务总线支持一组基于云的、面向消息的中间件技术，包括可靠的消息队列和持久发布/订阅消息。 这些中转消息传送功能可被视为分离式消息传送功能，支持使用服务总线消息传送结构的发布-订阅、临时分离和负载均衡方案。 分离式通信具有很多优点；例如，客户端和服务器可以根据需要进行连接并以异步方式执行其操作。

构成服务总线消息传送功能核心的消息传送实体包括队列、主题/订阅、规则/操作。

## <a name="queues"></a> 队列
队列为一个或多个竞争使用方提供*先入先出* (FIFO) 消息传递方式。 也就是说，接收方预计会按照消息添加到队列中的顺序来接收并处理消息，并且每条消息仅由一个消息使用方接收并处理。 使用队列的主要优点是，实现应用程序组件的“临时分离”。 换而言之，创建方（发送方）和使用方（接收方）无需同时发送和接收消息，因为消息持久存储在队列中。 此外，创建方不必等待使用方的答复即可继续处理并发送更多消息。

相关的优点是“负载分级”，它允许创建方和使用方以不同速率发送和接收消息。在许多应用程序中，系统负载随时间而变化，而每个工作单元所需的处理时间通常为常量。使用队列在消息创建方与使用方之间中继意味着，只需将消费应用程序预配为处理平均负载而非最大负载。队列深度将随传入负载的变化而加大和减小。这将直接根据为应用程序负载提供服务所需的基础结构的数目来节省成本。随着负载增加，可添加更多的工作进程以从队列中读取。每条消息仅由一个工作进程处理。另外，可通过此基于拉取的负载均衡来以最合理的方式使用辅助计算机，即使这些辅助计算机具有不同的处理能力（因为它们将以其最大速率拉取消息）也是如此。此模式通常称为“使用方竞争”模式。

使用队列在消息创建方与使用方之间中继可在各组件之间提供固有的松散耦合。由于创建方和使用方互不相识，因此，可升级使用方，而不会对创建方产生任何影响。

创建队列是一个多步骤过程。 可通过 [Microsoft.ServiceBus.NamespaceManager](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.namespacemanager#microsoft_servicebus_namespacemanager) 类执行服务总线消息传送实例（队列和主题）的管理操作，该类可通过提供服务总线命名空间的基址和用户凭据进行构建。 [NamespaceManager](/dotnet/api/microsoft.servicebus.namespacemanager#microsoft_servicebus_namespacemanager) 提供了创建、枚举和删除消息传送实体的方法。 在使用 SAS 名称和密钥创建 [Microsoft.ServiceBus.TokenProvider](/dotnet/api/microsoft.servicebus.tokenprovider#microsoft_servicebus_tokenprovider) 对象以及一个服务命名空间管理对象之后，可使用 [Microsoft.ServiceBus.NamespaceManager.CreateQueue](/dotnet/api/microsoft.servicebus.namespacemanager#Microsoft_ServiceBus_NamespaceManager_CreateQueue_System_String_) 方法来创建队列。 例如：

    // Create management credentials
    TokenProvider credentials = TokenProvider. CreateSharedAccessSignatureTokenProvider(sasKeyName,sasKeyValue);
    // Create namespace client
    NamespaceManager namespaceClient = new NamespaceManager(ServiceBusEnvironment.CreateServiceUri("sb", ServiceNamespace, string.Empty), credentials);

你可以随后创建一个队列对象和消息工厂，将服务总线 URI 用作参数。 例如：

    QueueDescription myQueue;
    myQueue = namespaceClient.CreateQueue("TestQueue");
    MessagingFactory factory = MessagingFactory.Create(ServiceBusEnvironment.CreateServiceUri("sb", ServiceNamespace, string.Empty), credentials); 
    QueueClient myQueueClient = factory.CreateQueueClient("TestQueue");

你可以随后向队列发送消息。 例如，如果具有名为 `MessageList`的中转消息列表，将出现此代码，类似如下形式：

    for (int count = 0; count < 6; count++)
    {
        var issue = MessageList[count];
        issue.Label = issue.Properties["IssueTitle"].ToString();
        myQueueClient.Send(issue);
    }

你可以随后接收来自队列的消息，如下所示：

    while ((message = myQueueClient.Receive(new TimeSpan(hours: 0, minutes: 0, seconds: 5))) != null)
        {
            Console.WriteLine(string.Format("Message received: {0}, {1}, {2}", message.SequenceNumber, message.Label, message.MessageId));
            message.Complete();

            Console.WriteLine("Processing message (sleeping...)");
            Thread.Sleep(1000);
        }

使用 [ReceiveAndDelete](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.receivemode) 模式时，接收操作是一个单一快照。即，当服务总线收到请求时，会将该消息标记为“已使用”并将其返回给应用程序。 **ReceiveAndDelete** 模式是最简单的模式，最适合应用程序允许出现故障时不处理消息的方案。 为了理解这一点，可以考虑这样一种情形：使用方发出接收请求，但在处理该请求前发生了崩溃。 由于服务总线会将消息标记为“已使用”，因此当应用程序重新启动并重新开始使用消息时，它会漏掉在发生崩溃前使用的消息。

使用 [PeekLock](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.receivemode) 模式时，接收操作分成了两步，从而有可能支持无法容忍遗漏消息的应用程序。 当服务总线收到请求时，它会找到要使用的下一个消息，将其锁定以防其他使用方接收它，然后将该消息返回给应用程序。 应用程序完成消息处理（或可靠地存储消息以供将来处理）后，它将通过对收到的消息调用 [Complete](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.brokeredmessage#Microsoft_ServiceBus_Messaging_BrokeredMessage_Complete) 完成接收过程的第二个阶段。 服务总线发现 **Complete** 调用时会将消息标记为“正在使用”。

如果应用程序出于某种原因无法处理消息，则其可对收到的消息调用 [Abandon](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.brokeredmessage#Microsoft_ServiceBus_Messaging_BrokeredMessage_Abandon) 方法（而不是 [Complete](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.brokeredmessage#Microsoft_ServiceBus_Messaging_BrokeredMessage_Complete) 方法）。 这可使服务总线解锁消息并使其能够重新被同一个使用方或其他竞争使用方接收。 此外，还存在与锁定关联的超时，并且如果应用程序无法在锁定超时到期之前处理消息（例如，如果应用程序崩溃），服务总线将解锁该消息并使其可再次被接收（实质上是默认执行一个 [Abandon](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.brokeredmessage#Microsoft_ServiceBus_Messaging_BrokeredMessage_Abandon) 操作）。

请注意，如果应用程序在处理消息之后，但在发出 **Complete** 请求之前发生崩溃，则在应用程序重新启动时会将该消息重新传送给它。 此情况通常称作 *至少处理一次* ，即每条消息将至少被处理一次。 但是，在某些情况下，同一消息可能会被重新传送。 如果某个场景不允许重复处理，则应用程序中需要用于检测重复的其他逻辑，此要求可基于消息的 **MessageId** 属性来实现，该属性在多次传送尝试中保持不变。 这称为 *仅一次* 处理。

## <a name="topics-and-subscriptions"></a>主题和订阅
与每条消息都由单个使用方处理的队列相比，主题和订阅通过发布/订阅模式提供“一对多”通信方式。 这对于扩展到大量接收方而言十分有用，每个发布的消息对向该主题注册的每个订阅均可用。 系统会将消息发送到主题并传递到一个或多个相关联的订阅，具体取决于每个订阅上可以设置的筛选规则。 此订阅可以使用其他筛选器来限制其想要接收的消息。 可以采用与发送至队列的相同方式将消息发送至主题，但不可直接从主题接收消息。 而是从订阅接收消息。 主题订阅类似于接收发送至该主题的消息副本的虚拟队列。 从订阅接收消息的方式与从队列接收相同。

通过比较，队列的消息发送功能直接映射到主题，而其消息接收功能映射到订阅。 此外，这意味着订阅支持本部分前面所述的关于队列的相同模式：竞争使用方、临时分离、负载分级和负载均衡。

创建主题类似于创建队列，如上一部分中的示例所示。 创建服务 URI，然后使用 [NamespaceManager](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.namespacemanager) 类创建命名空间客户端。 然后，你可以使用 [CreateTopic](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.namespacemanager#Microsoft_ServiceBus_NamespaceManager_CreateTopic_System_String_) 方法创建主题。 例如：

    TopicDescription dataCollectionTopic = namespaceClient.CreateTopic("DataCollectionTopic");

接下来，根据需要添加订阅：

    SubscriptionDescription myAgentSubscription = namespaceClient.CreateSubscription(myTopic.Path, "Inventory");
    SubscriptionDescription myAuditSubscription = namespaceClient.CreateSubscription(myTopic.Path, "Dashboard");

然后可以创建主题客户端。 例如：

    MessagingFactory factory = MessagingFactory.Create(serviceUri, tokenProvider);
    TopicClient myTopicClient = factory.CreateTopicClient(myTopic.Path)

通过消息发送方，你可以将消息发送至主题和从主题接收消息，如上一部分所述。 例如：

    foreach (BrokeredMessage message in messageList)
    {
        myTopicClient.Send(message);
        Console.WriteLine(
        string.Format("Message sent: Id = {0}, Body = {1}", message.MessageId, message.GetBody<string>()));
    }


与队列类似，通过 [SubscriptionClient](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.subscriptionclient) 对象而不是 [QueueClient](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.queueclient) 对象从订阅接收消息。 创建订阅客户端，将主题名称、订阅名称和接收模式（可选）作为参数传递。 例如，对于**清单**订阅：


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

### <a name="rules-and-actions"></a>规则和操作
在许多情况下，必须以不同方式处理具有特定特征的消息。 若要启用此功能，你可以将订阅配置为查找具有所需属性的消息，然后执行这些属性的部分修改操作。 虽然服务总线订阅可以看到发送到主题的所有消息，但你仅可以将这些消息的一个子集复制到虚拟订阅队列。 使用订阅筛选器完成此操作。 此类修改称为筛选器操作。 创建订阅后，可提供一个对消息属性进行操作的筛选器表达式，其中属性包括系统属性（例如 **Label**）和自定义应用程序属性（例如 **StoreName**）这种情况下，SQL 筛选器表达式为可选项；如没有 SQL 筛选器表达式，订阅上定义的任何筛选器会对该订阅的所有消息执行。

要使用先前的示例仅筛选来自 **Store1** 的消息，请创建仪表板订阅，如下所示：

    namespaceManager.CreateSubscription("IssueTrackingTopic", "Dashboard", new SqlFilter("StoreName = 'Store1'"));

使用此订阅筛选器，仅 `StoreName` 属性设置为 `Store1` 的消息将被复制到 `Dashboard` 订阅的虚拟队列。

有关可能的筛选器值的详细信息，请参阅文档 [SqlFilter](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.sqlfilter) 和 [SqlRuleAction](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.sqlruleaction) 类。 另请参阅[中转消息传送：高级筛选器](http://code.msdn.microsoft.com/Brokered-Messaging-6b0d2749)和[主题筛选器](https://github.com/Azure-Samples/azure-servicebus-messaging-samples/tree/master/TopicFilters)示例。

## <a name="next-steps"></a>后续步骤
有关使用服务总线消息传送的详细信息和示例，请参阅以下高级主题。

- [服务总线消息传送概述](/documentation/articles/service-bus-messaging-overview/)
- [服务总线中转消息传送 .NET 教程](/documentation/articles/service-bus-dotnet-get-started-with-queues/)
- [服务总线中转消息传送 REST 教程](/documentation/articles/service-bus-dotnet-get-started-with-queues/)
- [主题筛选器示例](https://github.com/Azure-Samples/azure-servicebus-messaging-samples/tree/master/TopicFilters)
- [中转消息传送：高级筛选器](http://code.msdn.microsoft.com/Brokered-Messaging-6b0d2749)


<!--Update_Description:update wording-->