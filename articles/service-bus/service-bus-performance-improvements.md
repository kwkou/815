<properties 
   pageTitle="使用服务总线提高性能的最佳实践 | Microsoft Azure"
   description="介绍如何使用 Azure 服务总线在交换中转消息时优化性能。"
   services="service-bus"
   documentationCenter="na"
   authors="sethmanheim"
   manager="timlt"
   editor="" /> 
<tags 
   ms.service="service-bus"
   ms.date="03/16/2016"
   wacn.date="02/26/2016" />

# 使用服务总线中转消息传送改进性能的最佳实践

本主题介绍如何使用 Azure 服务总线在交换中转消息时优化性能。本主题的第一部分介绍有助于提高性能的各种不同机制。第二部分则指导用户如何针对给定方案以能够提供最佳性能的方式使用服务总线。

在本主题中，术语“客户端”是指任何访问服务总线的实体。客户端可以充当发送方或接收方的角色。术语“发送方”用于服务总线队列或向服务总线队列或主题发送消息的主题客户端。术语“接收方”是指服务总线队列或从服务总线队列或订阅接收消息的订阅客户端。

以下几部分介绍服务总线用以帮助提高性能的几个概念。

## 协议

服务总线使客户端能够通过两个协议发送和接收消息：服务总线客户端协议和 HTTP (REST)。服务总线客户端协议更高效，因为只要消息工厂存在，该协议就会始终保持与服务总线服务的连接。它还实现批处理和预提取。服务总线客户端协议可用于使用 .NET ApI 的 .NET 应用程序。

除非明确指出，本主题中的所有内容都假定使用服务总线客户端协议。

## 重用工厂和客户端

[QueueClient][] 或 [MessageSender][] 等服务总线客户端对象是通过 [MessagingFactory][] 对象创建的，该对象还提供连接的内部管理。发送消息后不应关闭消息工厂或队列、主题和订阅客户端，然后，在发送下一条消息时重新创建它们。关闭消息工厂将删除与服务总线服务的连接，并且会在重新创建工厂时建立新的连接。建立连接是一项成本高昂的操作，可通过针对多个操作重复使用相同的工厂和客户端对象来避免这一操作。

## 并发操作

执行某项操作（发送、接收、删除等）需要花费一定时间。这一时间包括服务总线服务处理该操作的时间，外加延迟处理请求和答复的时间。若要增加每次操作的数目，操作必须同时执行。可以通过几种不同的方式实现：

-   **异步操作**：客户端通过执行异步操作来计划操作。前一个请求完成之前便启动下一个请求。以下是异步发送操作的示例：

	```
	BrokeredMessage m1 = new BrokeredMessage(body);
	BrokeredMessage m2 = new BrokeredMessage(body);
	
	Task send1 = queueClient.SendAsync(m1).ContinueWith((t) => 
	  {
	    Console.WriteLine("Sent message #1");
	  });
	Task send2 = queueClient.SendAsync(m2).ContinueWith((t) => 
	  {
	    Console.WriteLine("Sent message #2");
	  });
	Task.WaitAll(send1, send2);
	Console.WriteLine("All messages sent");
	```

	这是异步接收操作的示例：
	
	```
	Task receive1 = queueClient.ReceiveAsync().ContinueWith(ProcessReceivedMessage);
	Task receive2 = queueClient.ReceiveAsync().ContinueWith(ProcessReceivedMessage);
	
	Task.WaitAll(receive1, receive2);
	Console.WriteLine("All messages received");
	
	async void ProcessReceivedMessage(Task<BrokeredMessage> t)
	{
	  BrokeredMessage m = t.Result;
	  Console.WriteLine("{0} received", m.Label);
	  await m.CompleteAsync();
	  Console.WriteLine("{0} complete", m.Label);
	}
	```

-   **多个工厂**：由同一工厂创建的所有客户端（发送方和接收方）共享一个 TCP 连接。最大消息吞吐量受可通过此 TCP 连接的操作的数目限制。单个工厂可获得的吞吐量因 TCP 往返时间和消息大小不同而大有差异。若要获得更高的吞吐速率，应使用多个消息工厂。

## 接收模式

在创建队列或订阅客户端时，可以指定接收模式：*扫视-锁定*或*接收与删除*。默认接收模式是 [PeekLock][]。在此模式下操作时，客户端发送请求以从服务总线接收消息。客户端收到消息后，将发送完成消息的请求。

如果将接收模式设置为 [ReceiveAndDelete][] 时，这两个步骤将合并到单个请求中。这减少了操作的总体数目，并可以提高总消息吞吐量。性能提高的同时也出现丢失消息的风险。

服务总线不支持“接收与删除”操作的事务。此外，在客户端想要延迟消息或将其放入死信队列的情况下，需要使用扫视-锁定语义。

## 客户端批处理

客户端批处理使队列或主题客户端可以将发送消息的操作延迟特定的一段时间。如果客户端在这段时间内发送其他消息，则会将这些消息以单个批次传送。客户端批处理还使队列/订阅客户端将多个**完成**请求批处理为单个请求。批处理仅适用于异步**发送**和**完成**操作。同步操作会立即发送到服务总线服务。不会针对扫视或接收操作执行批处理，也不会跨客户端执行批处理。

如果批大小超出最大消息大小，将从该批中删除最后一个消息，并且客户端会立即发送该批次。最后一条消息将成为下一批的第一个消息。默认情况下，客户端的批处理间隔时间为 20 毫秒。可通过在创建消息工厂之前，设置 [BatchFlushInterval][] 属性，更改批处理的间隔时间。此设置会影响此工厂创建的所有客户端。

要禁用批处理，则将 [BatchFlushInterval][] 属性设置为 **TimeSpan.Zero**。例如：

```
MessagingFactorySettings mfs = new MessagingFactorySettings();
mfs.TokenProvider = tokenProvider;
mfs.NetMessagingTransportSettings.BatchFlushInterval = TimeSpan.FromSeconds(0.05);
MessagingFactory messagingFactory = MessagingFactory.Create(namespaceUri, mfs);
```

批处理不会影响可计费的消息操作的数目，且仅适用于服务总线客户端协议。HTTP 协议不支持批处理。

## 批处理存储访问

为增加队列/主题/订阅的吞吐量，服务总线在写入其内部存储时会将多个消息进行批处理。如果在队列或主题上启用了写入消息到存储，则将会批量实施这一操作。如果在队列或订阅上启用了从存储删除消息，则将会批量实施这一操作。如果对实体启用了批量存储访问，服务总线会将有关此实体的存储写入操作延迟多达 20 毫秒的时间。在此间隔期间发生的其他存储操作将被添加到此批中。批量存储访问仅影响**发送**和**完成**操作；接收操作不会受到影响。批量存储访问是实体上的一个属性。将跨所有启用了批量存储访问的实体实施批处理。

创建新队列、主题或订阅时，将默认启用批量存储访问。若要禁用批量存储访问，则在创建实体之前将 [EnableBatchedOperations][] 属性设置为 **false**。例如：

```
QueueDescription qd = new QueueDescription();
qd.EnableBatchedOperations = false;
Queue q = namespaceManager.CreateQueue(qd);
```

批量存储访问不影响可计费的消息操作的数目，并且是队列、主题或订阅的一个属性。它不依赖于接收模式以及客户端和服务总线服务之间所使用的协议。

## 预提取

预提取使队列或订阅客户端可在执行接收操作时从服务加载其他消息。客户端将这些消息存储在本地缓存中。缓存大小由 [QueueClient.PrefetchCount][] 属性和 [SubscriptionClient.PrefetchCount][] 属性决定。启用预提取的每个客户端维护其自己的缓存。客户端之间不共享缓存。如果客户端启动接收操作，而其缓存是空的，则服务会传输一批消息。批的大小等于缓存的大小或 256 KB，以二者中较小者为准。如果客户端启动接收操作，而其缓存中包含一个消息，则将从缓存中取出消息。

预提取一条消息后，服务将锁定此预提取的消息。通过执行此操作，其他接收方则无法接收到此预提取的消息。如果接收方在锁定过期之前无法完成此消息，则该消息便对其他接收方可用。预提取的消息的副本则保留在缓存中。使用过期的缓存副本的接收方将在尝试完成该消息时接收到一个异常。默认情况下，消息锁定在 60 秒后过期。这一值可延长到 5 分钟。若要阻止过期消息的使用，缓存大小应始终小于客户端可在锁定超时间隔内使用的消息数。

使用 60 秒的默认锁定时限时，[SubscriptionClient.PrefetchCount][] 的合理值是工厂所有接收方最大处理速率的 20 倍。例如，某个工厂创建了 3 个接收方，并且每个接收方每秒可以处理最多 10 个消息。预提取计数不应超过 20*3*10 = 600。默认情况下，[QueueClient.PrefetchCount][] 设置为 0，这表示不会从服务中提取额外消息。

预提取消息会增加队列或订阅的整体吞吐量，因为它会减少消息操作或往返行程的整体数量。但是，提取第一条消息将会耗用更长的时间（因消息大小增加所致）。由于预提取的消息已经被客户端下载，因此预提取消息的接收速度将变快。

服务器会在向客户端发送消息时检查消息的“生存时间 (TTL)”属性。收到邮件时，客户端不检查消息的 TTL 属性。即使消息被客户端缓存时该消息的 TTL 已通过，该消息仍可被接收。

预提取不会影响可计费的消息操作的数目，且仅适用于服务总线客户端协议。HTTP 协议不支持预提取。预提取可用于同步和异步接收操作。

## Express 队列和主题

Express 实体可实现高吞吐量同时减少延迟的情况。使用 express 实体的话，如果向队列或主题发送消息，消息不会立即被存储在消息存储区中。相反，消息会被缓存在内存中。如果消息在队列中留存的时间超过数秒钟，则会自动写入到稳定的存储区内，以避免其因中断而丢失。将消息写入到内存缓存内会增加吞吐量，减少延迟，因为在消息发送时不存在对稳定存储区的访问。将在几秒钟内使用的消息不会写入到消息存储中。以下示例会创建一个 express 主题。

```
TopicDescription td = new TopicDescription(TopicName);
td.EnableExpress = true;
namespaceManager.CreateTopic(td);
```

如果要将包含了必不可失的关键信息的消息发送到 express 实体，则发送方可以通过将 [ForcePersistence][] 属性设置为 **true**，强制服务总线立即将消息保留至稳定的存储区内。

## 使用分区队列或主题

在内部，服务总线使用相同的节点和消息存储来处理和存储消息传递实体（队列或主题）的所有消息。另一方面，分区的队列或主题被分布在多个节点和消息存储上。分区的队列和主题不仅生成比常规队列和主题更高的吞吐量，还表现出极高的可用性。若要生成分区的实体，则如以下示例所示，将 [EnablePartitioning][] 属性设置为 **true**。有关分区实体的详细信息，请参阅[分区消息实体][]。

```
// Create partitioned queue.
QueueDescription qd = new QueueDescription(QueueName);
qd.EnablePartitioning = true;
namespaceManager.CreateQueue(qd);
```

## 使用多个队列

如果不能使用分区的队列或主题，或不能由单个分区的队列或主题处理预期负载，则必须使用多个消息传递实体。在使用多个实体时，为每个实体创建专用客户端，而不是针对所有实体使用同一个客户端。

## 方案

以下各节介绍典型的消息传递方案，并概述首选服务总线设置。吞吐速率分为小（小于 1 条消息/秒）、中等（1 条消息/秒或更大，但不超过 100 条消息/秒）和高（100 条消息/秒或更大）。客户端数分为小（5 个或更少）、中等（5 个以上但小于或等于 20 个）和大（超过 20 个）。

### 高吞吐量队列

目标：将单个队列的吞吐量最大化。发送方和接收方的数目较小。

-   使用分区的队列提高性能和可用性。

-   如要增加面向队列的总发送速率，则使用多个消息工厂来创建发送方。为每个发送方使用异步操作或多个线程。

-   如要增加从队列接收的总体接收速率，则使用多个消息工厂来创建接收方。

-   使用异步操作可利用客户端批处理。

-   将批处理间隔时间设置为 50 毫秒以减少服务总线客户端协议传输的数量。如果使用多个发送方，则将批处理间隔时间增加到 100 毫秒。

-   将批量存储访问保留为启用状态。这会增加可将消息写入队列的总速率。

-   将预提取计数设置为工厂所有接收方最大处理速率的 20 倍。这将减少服务总线客户端协议传输的数量。

### 多个高吞吐量队列

目标：将多个队列的整体吞吐量最大化。单个队列的吞吐量中等或高。

若要在多个队列之间获得最大的吞吐量，则使用所述设置将单个队列的吞吐量最大化。此外，使用不同工厂来创建向不同的队列发送或从其接收的客户端。

### 低延迟队列

目标：将队列或主题的端到端延迟减至最小。发送方和接收方的数目较小。队列的吞吐量较小或为中等。

-   使用分区的队列提高可用性。

-   禁用客户端批处理。客户端会立即发送一条消息。

-   禁用批量存储访问。该服务会立即将消息写入存储。

-   如果使用单个客户端，将预提取计数设置为接收方处理速率的 20 倍。如果多条消息同时到达队列，服务总线客户端协议会将这些消息全部同时传输。当客户端收到下一条消息时，该消息便已存在于本地缓存中。缓存应较小。

-   如果使用多个客户端，则将预提取计数设置为 0。通过执行此操作，在第一个客户端仍在处理第一条消息时，第二个客户端可以接收第二条消息。

### 包含大量发送方的队列

目标：将包含大量发送方的队列或主题的吞吐量最大化。每个发送方均以中等速率发送消息。接收方的数目较小。

服务总线允许最多 1000 个与消息传递实体之间的并发连接（使用 AMQP 则为 5000 个）。该限制在命名空间级别是强制性的，队列/主题/订阅则按命名空间受此并发连接的限制。就队列而言，此数值在发送方和接收方之间共享。如果针对发送方要求全部 1000 个连接，则应该将队列替换为主题或单个订阅。一个主题可从发送方接收最多 1000 个并发连接，而订阅从接收方接受另外 1000 个并发连接。如果需要超过 1000 个并发发送方，则发送方应通过 HTTP 向服务总线协议发送消息。

若要使吞吐量最大化，则执行以下操作：

-   使用分区的队列提高性能和可用性。

-   如果每个发送方驻留在不同进程中，则每个进程仅使用单个工厂。

-   使用异步操作可利用客户端批处理。

-   使用 20 毫秒的默认批处理间隔时间以减少服务总线客户端协议传输的数量。

-   将批量存储访问保留为启用状态。这会增加可将消息写入队列或主题的总速率。

-   将预提取计数设置为工厂所有接收方最大处理速率的 20 倍。这将减少服务总线客户端协议传输的数量。

### 包含大量接收方的队列

目标：将包含大量接收方的队列或订阅的接收速率最大化。每个接收方以中等接收速率接收消息。发送方的数目较小。

服务总线允许最多 1000 个与实体之间的并发连接。如果队列需要超过 1000 个接收方，则应将队列替换为主题和多个订阅。每个订阅可支持最多 1000 个并发连接。或者，接收方可通过 HTTP 协议访问队列。

若要使吞吐量最大化，则执行以下操作：

-   使用分区的队列提高性能和可用性。

-   如果每个接收方驻留在不同进程中，则每个进程仅使用单个工厂。

-   接收方可使用同步或异步操作。如果独立接收方的接收速率给定为中等级别，则客户端对“完成”请求的批处理不会影响接收方的吞吐量。

-   将批量存储访问保留为启用状态。这将减少实体的总负载。还会减少可将消息写入队列或主题的总写入速率。

-   将预提取计数设置为较小值（例如，PrefetchCount = 10）。这可防止接收方在其他接收方已缓存大量消息时处于闲置状态。

### 具有少量订阅的主题

目标：将具有少量订阅的主题的吞吐量最大化。一条消息由多个订阅接收，这意味着所有订阅的合并接收速率比发送速率大。发送方的数目较小。每个订阅的接收方的数目较小。

若要使吞吐量最大化，则执行以下操作：

-   使用分区的主题提高性能和可用性。

-   如要增加面向主题的总发送速率，则使用多个消息工厂来创建发送方。为每个发送方使用异步操作或多个线程。

-   如要增加从订阅接收的总体接收速率，则使用多个消息工厂来创建接收方。为每个接收方使用异步操作或多个线程。

-   使用异步操作可利用客户端批处理。

-   使用 20 毫秒的默认批处理间隔时间以减少服务总线客户端协议传输的数量。

-   将批量存储访问保留为启用状态。这会增加可将消息写入主题的总写入速率。

-   将预提取计数设置为工厂所有接收方最大处理速率的 20 倍。这将减少服务总线客户端协议传输的数量。

### 具有大量订阅的主题

目标：将具有大量订阅的主题的吞吐量最大化。一条消息由多个订阅接收，这意味着所有订阅的合并接收速率比发送速率大得多。发送方的数目较小。每个订阅的接收方的数目较小。

如果所有消息均路由到每个订阅，则具有大量订阅的主题通常会呈现较低的总体吞吐量。这是因为每个消息均被接收了许多次，且主题及其全部订阅包含的所有消息均存储在相同的存储区内。假定每个订阅的发送方和接收方的数目都较小。服务总线支持每个主题最多具有 2000 个订阅。

若要使吞吐量最大化，则执行以下操作：

-   使用分区的主题提高性能和可用性。

-   使用异步操作可利用客户端批处理。

-   使用 20 毫秒的默认批处理间隔时间以减少服务总线客户端协议传输的数量。

-   将批量存储访问保留为启用状态。这会增加可将消息写入主题的总写入速率。

-   将预提取计数设置为预期接收速率的 20 倍（以秒为单位）。这将减少服务总线客户端协议传输的数量。

## 后续步骤

请参阅[分区消息实体][]，了解有关优化服务总线性能的详细信息。

  [QueueClient]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.queueclient.aspx
  [MessageSender]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.messagesender.aspx
  [MessagingFactory]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.messagingfactory.aspx
  [PeekLock]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.receivemode.aspx
  [ReceiveAndDelete]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.receivemode.aspx
  [BatchFlushInterval]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.netmessagingtransportsettings.batchflushinterval.aspx
  [EnableBatchedOperations]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.queuedescription.enablebatchedoperations.aspx
  [QueueClient.PrefetchCount]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.queueclient.prefetchcount.aspx
  [SubscriptionClient.PrefetchCount]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.subscriptionclient.prefetchcount.aspx
  [ForcePersistence]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.brokeredmessage.forcepersistence.aspx
  [EnablePartitioning]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.queuedescription.enablepartitioning.aspx
  [分区消息实体]: /documentation/articles/service-bus-partitioning/
  

<!---HONumber=74-->