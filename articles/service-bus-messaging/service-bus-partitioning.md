<properties
    pageTitle="创建分区的 Azure 服务总线队列和主题 | Azure"
    description="介绍如何使用多个消息中转站对服务总线队列和主题进行分区。"
    services="service-bus"
    documentationCenter="na"
    authors="sethmanheim"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="a0c7d5a2-4876-42cb-8344-a1fc988746e7"
    ms.service="service-bus"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="03/28/2017"
    wacn.date="05/22/2017"
    ms.author="sethm;hillaryc"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="8fd60f0e1095add1bff99de28a0b65a8662ce661"
    ms.openlocfilehash="c82833d3b780669faf300be1814ee44a56c0b9a0"
    ms.lasthandoff="05/12/2017" />

# <a name="partitioned-queues-and-topics"></a>分区队列和主题
Azure 服务总线使用多个消息中转站处理消息，并使用多个消息传送存储存储消息。 传统的队列或主题由单个消息中转站进行处理并存储在一个消息存储中。 服务总线*分区*还允许跨多个消息中转站和消息存储对队列或主题进行分区。 这意味着分区的队列或主题的总吞吐量不再受到单个消息中转站或消息存储的性能所限制。 此外，消息传送存储的临时中断不会导致分区队列或主题不可用。 分区队列和主题可以包含所有先进的服务总线功能，如事务和会话支持。

有关服务总线内部的详细信息，请参阅[服务总线体系结构][Service Bus architecture]一文。

## <a name="how-it-works"></a>工作原理
每个分区队列或主题由多个片段构成。 每个片段存储在不同的消息传送存储中并由不同的消息中转站进行处理。 当向分区队列或主题发送消息时，服务总线会将该消息分配到其中一个片段。 通过服务总线或使用发送方可指定的分区键随机进行选择。

客户端要从分区队列或从分区主题的订阅接收消息时，服务总线将查询所有片段以获取消息，然后将从任何消息存储获取的第一条消息返回到接收方。 服务总线缓存其他消息并在收到其他接收请求时将它们返回。 接收客户端无法识别分区；分区队列或主题的面向客户端的行为（例如，读取、完成、延迟、死信、预提取）与常规实体行为相同。

向分区队列或主题发送一条消息，或从分区队列或主题接收消息时无需额外付费。

## <a name="enable-partitioning"></a>启用分区
若要将分区队列和主题用于 Azure 服务总线，请使用 Azure SDK 2.2 版或更高版本，或在 HTTP 请求中指定 `api-version=2013-10`。

可以创建 1、2、3、4 或 5 GB 大小的服务总线队列和主题（默认值为 1 GB）。 启用分区时，服务总线将为指定的每个大小创建 16 个分区。 因此，如果创建了一个大小为 5 GB 的队列，共有 16 个分区，则最大队列大小为 (5 \* 16) = 80 GB。 可以通过在 [Azure 门户预览][Azure portal]中查看分区队列或主题的条目来了解该队列或主题的最大大小。

有多种方法可以创建分区队列或主题。 从应用程序创建队列或主题时，可以通过分别将 [QueueDescription.EnablePartitioning][QueueDescription.EnablePartitioning] 或 [TopicDescription.EnablePartitioning][TopicDescription.EnablePartitioning] 属性设置为 **true** 来启用队列或主题的分区。 这些属性必须在创建队列或主题时设置。 无法更改现有队列或主题上的这些属性。 例如：

    // Create partitioned topic
    NamespaceManager ns = NamespaceManager.CreateFromConnectionString(myConnectionString);
    TopicDescription td = new TopicDescription(TopicName);
    td.EnablePartitioning = true;
    ns.CreateTopic(td);

或者，可以在 Visual Studio 中或在 [Azure 门户预览][Azure portal]中创建分区队列或主题。 在门户中创建队列或主题时，请将队列或主题“设置”窗口的“常规设置”边栏选项卡中的“启用分区”选项设置为“true”。 在 Visual Studio 中，单击“新建队列”或“新建主题”对话框中的“启用分区”复选框。

## <a name="use-of-partition-keys"></a>使用分区键
当一条消息在分区队列或主题中排队时，服务总线检查是否存在分区键。如果找到，它将选择基于该键的片段。如果找不到分区键，它将选择基于内部算法的片段。

### <a name="using-a-partition-key"></a>使用分区键
某些应用场景（例如会话或事务）要求将消息存储在特定的片段中。所有这些应用场景都需要使用分区键。使用相同的分区键的所有消息都分配到同一片段中。如果该片段暂时不可用，服务总线将返回一个错误。

根据应用场景，将不同的消息属性用作分区键：

**SessionId**：如果消息设置了 [BrokeredMessage.SessionId][BrokeredMessage.SessionId] 属性，则服务总线将此属性用作分区键。这样一来，属于同一会话的所有消息都由同一消息中转站处理。这使服务总线得以保证消息顺序以及会话状态的一致性。

**PartitionKey**：如果消息已设置 [BrokeredMessage.PartitionKey][BrokeredMessage.PartitionKey] 属性但未设置 [BrokeredMessage.SessionId][BrokeredMessage.SessionId] 属性，则服务总线将 [PartitionKey][PartitionKey] 属性用作分区键。 如果消息同时设置了 [SessionId][SessionId] 且未设置 [PartitionKey][PartitionKey] 属性，这两个属性必须相同。 如果 [PartitionKey][PartitionKey] 属性设置为与 [SessionId][SessionId] 属性不同的值，则服务总线返回无效操作异常。 如果发送方发送非会话感知事务消息，应使用 [PartitionKey][PartitionKey] 属性。 分区键可确保事务中所发送的所有消息都由同一个消息传送中转站处理。

**MessageId**：如果队列或主题将 [QueueDescription.RequiresDuplicateDetection][QueueDescription.RequiresDuplicateDetection] 属性设置为 **true** 且未设置 [BrokeredMessage.SessionId][BrokeredMessage.SessionId] 或 [BrokeredMessage.PartitionKey][BrokeredMessage.PartitionKey] 属性，则 [BrokeredMessage.MessageId][BrokeredMessage.MessageId] 属性将充当分区键。 （请注意，如果发送方应用程序未分配消息 ID，Microsoft .NET 和 AMQP 库会自动进行分配。）） 在这种情况下，同一消息的所有副本都由同一消息中转站处理。 这使服务总线能够检测并消除重复的消息。 如果 [QueueDescription.RequiresDuplicateDetection][QueueDescription.RequiresDuplicateDetection] 属性未设置为 **true**，服务总线不考虑将 [MessageId][MessageId] 属性用作分区键。

### <a name="not-using-a-partition-key"></a>不使用分区键
如果没有分区键，服务总线以轮循机制形式将消息分发到分区队列或主题的所有片段。 如果所选的片段不可用，服务总线会将消息分配给不同的片段。 这样一来，尽管消息传送存储暂时不可用，发送操作仍可成功。 但是，将无法实现分区键提供的保证排序。

有关可用性（没有分区键）和一致性（使用分区键）之间的权衡的更深入讨论，请参阅[此文](/documentation/articles/event-hubs-availability-and-consistency/)。 此信息同样适用于已分区服务总线实体和事件中心分区。
若要给服务总线足够的时间将消息排入不同片段的队列中，客户端指定的发送消息的 [MessagingFactorySettings.OperationTimeout][MessagingFactorySettings.OperationTimeout] 值必须大于 15 秒。建议你将 [OperationTimeout][OperationTimeout] 属性设置为 60 秒的默认值。

请注意，分区键会将消息“固定”到特定片段。如果保存此片段的消息传送存储不可用，服务总线将返回一个错误。如果没有分区键，服务总线可以选择其他片段且操作将成功。因此，建议你除非必需，否则不要提供分区键。

## <a name="advanced-topics-use-transactions-with-partitioned-entities"></a>高级主题：将事务用于分区实体
作为事务一部分发送的消息必须指定分区键。这可以是以下属性之一：[BrokeredMessage.SessionId][BrokeredMessage.SessionId]、[BrokeredMessage.PartitionKey][BrokeredMessage.PartitionKey] 或 [BrokeredMessage.MessageId][BrokeredMessage.MessageId]。所有作为同一事务一部分发送的消息必须指定相同的分区键。如果尝试在事务中发送一条没有分区键的消息，服务总线会返回无效操作异常。如果尝试在同一事务中发送多条具有不同分区键的消息，服务总线会返回无效操作异常。例如：


		CommittableTransaction committableTransaction = new CommittableTransaction();
		using (TransactionScope ts = new TransactionScope(committableTransaction))
		{
		    BrokeredMessage msg = new BrokeredMessage("This is a message");
		    msg.PartitionKey = "myPartitionKey";
		    messageSender.Send(msg); 
		    ts.Complete();
		}
		committableTransaction.Commit();


如果设置了任何作为分区键的属性，服务总线会将消息固定到特定片段。无论是否使用事务，该行为都会发生。建议你如非必要，不要指定分区键。

## <a name="using-sessions-with-partitioned-entities"></a>将会话用于分区实体
若要将事务消息发送到会话感知的主题或队列，消息必须设置 [BrokeredMessage.SessionId][BrokeredMessage.SessionId] 属性。 如果也指定了 [BrokeredMessage.PartitionKey][BrokeredMessage.PartitionKey] 属性，该属性必须与 [SessionId][SessionId] 属性相同。 如果它们不同，服务总线会返回无效操作异常。

与常规（非分区）队列或主题不同，不能使用单一事务将多条消息发送到不同会话。如果进行尝试，服务总线返回无效操作异常。例如：


		CommittableTransaction committableTransaction = new CommittableTransaction();
		using (TransactionScope ts = new TransactionScope(committableTransaction))
		{
		    BrokeredMessage msg = new BrokeredMessage("This is a message");
		    msg.SessionId = "mySession";
		    messageSender.Send(msg); 
		    ts.Complete();
		}
		committableTransaction.Commit();

## <a name="automatic-message-forwarding-with-partitioned-entities"></a>使用分区实体自动进行消息转发
服务总线支持从分区实体、向分区实体或在分区实体之间进行消息自动转发。 若要启用消息自动转发，请在源队列或订阅上设置 [QueueDescription.ForwardTo][QueueDescription.ForwardTo] 属性。 如果消息指定分区键（[SessionId][SessionId]、[PartitionKey][PartitionKey] 或 [MessageId][MessageId]），则该分区键用于目标实体。

## <a name="considerations-and-guidelines"></a>注意事项和指南
* **高度一致性功能**：如果实体使用会话、重复检测或显式控制分区键等功能，则消息传送操作一定会路由至特定的片段。 如果任何片段遇到过高的流量，或基础存储处于不正常状态，这些操作将失败，可用性会降低。 整体来说，一致性仍然远高于非分区实体；只有一部分流量会遇到问题，而不是所有流量。 有关详细信息，请参阅此处[对可用性和一致性的讨论](/documentation/articles/event-hubs-availability-and-consistency/)。
- **管理**：必须对实体的所有片段执行创建、更新及删除等操作。如果任何片段处于不正常状态，可能会导致这些操作失败。以“获取”操作来说，必须汇总来自所有片段的信息，例如消息计数。如果任何片段处于不正常状态，则实体可用性状态会报告为受限制。
- **少量消息的情况**：对于这类情况，尤其是使用 HTTP 协议时，可能必须执行多次接收操作，才能获取所有消息。对于接收请求，前端会在所有片段上执行接收，并缓存所有收到的响应。相同连接上的后续接收请求将受益于此缓存，而且接收延迟将会缩短。不过，如果你有多个连接或使用 HTTP，则会针对每个请求建立新的连接。因此，不保证抵达相同的节点。如果现有的所有消息均被锁定，而且在另一个前端中缓存，则接收操作返回 **null**。消息最后会到期，你可以再次接收它们。建议使用 HTTP 保持连接。
* **浏览/速览消息**：[PeekBatch](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.queueclient#Microsoft_ServiceBus_Messaging_QueueClient_PeekBatch_System_Int32_) 不一定返回 [MessageCount](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.queuedescription#Microsoft_ServiceBus_Messaging_QueueDescription_MessageCount) 属性中指定的消息数目。 这有两个常见的原因。 其中一个原因是消息集合的汇总大小超过大小上限 256KB。 另一个原因是，如果队列或主题的 [EnablePartitioning 属性](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.queuedescription#Microsoft_ServiceBus_Messaging_QueueDescription_EnablePartitioning) 设为 **true**，则分区可能没有足够的消息来完成所请求的消息数目。 一般情况下，如果应用程序想要接收特定数目的消息，它应该重复调用 [PeekBatch](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.queueclient#Microsoft_ServiceBus_Messaging_QueueClient_PeekBatch_System_Int32_) ，直到获得该数目的消息，或者已没有更多消息可速览为止。 有关详细信息（包括代码示例），请参阅 [QueueClient.PeekBatch](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.queueclient#Microsoft_ServiceBus_Messaging_QueueClient_PeekBatch_System_Int32_) 或 [SubscriptionClient.PeekBatch](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.subscriptionclient#Microsoft_ServiceBus_Messaging_SubscriptionClient_PeekBatch_System_Int32_)。

## <a name="latest-added-features"></a>最新添加的功能

- 分区实体现在支持添加或删除规则。与非分区实体不同的是，不支持在事务下执行这些操作。
- AMQP 现在支持通过分区实体发送和接收消息。
- AMQP 现在支持以下操作：[成批发送](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.queueclient#Microsoft_ServiceBus_Messaging_QueueClient_SendBatch_System_Collections_Generic_IEnumerable_Microsoft_ServiceBus_Messaging_BrokeredMessage__)、[成批接收](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.queueclient#Microsoft_ServiceBus_Messaging_QueueClient_ReceiveBatch_System_Int32_)、[按序列号接收](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.queueclient#Microsoft_ServiceBus_Messaging_QueueClient_Receive_System_Int64_)、[速览](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.queueclient#Microsoft_ServiceBus_Messaging_QueueClient_Peek)、[续订锁定](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.queueclient#Microsoft_ServiceBus_Messaging_QueueClient_RenewMessageLock_System_Guid_)、[计划消息](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.queueclient#Microsoft_ServiceBus_Messaging_QueueClient_ScheduleMessageAsync_Microsoft_ServiceBus_Messaging_BrokeredMessage_System_DateTimeOffset_)、[取消计划的消息](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.queueclient#Microsoft_ServiceBus_Messaging_QueueClient_CancelScheduledMessageAsync_System_Int64_)、[添加规则](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.ruledescription)、[删除规则](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.ruledescription)、[会话续订锁定](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.messagesession#Microsoft_ServiceBus_Messaging_MessageSession_RenewLock)、[设置会话状态](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.messagesession#Microsoft_ServiceBus_Messaging_MessageSession_SetState_System_IO_Stream_)、[获取会话状态](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.messagesession#Microsoft_ServiceBus_Messaging_MessageSession_GetState)和[枚举会话](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.queueclient#Microsoft_ServiceBus_Messaging_QueueClient_GetMessageSessionsAsync)。

## <a name="partitioned-entities-limitations"></a>分区实体限制
当前，服务总线对分区队列和主题施加以下限制：

-   分区队列和主题不支持在单个事务中发送属于不同会话的消息。

-   服务总线当前允许为每个命名空间最多创建 100 个分区队列或主题。每个分区队列或主题都将计入每个命名空间的 10,000 个实体的配额。

## <a name="next-steps"></a>后续步骤
请参阅[针对服务总线分区队列和主题的 AMQP 1.0 支持][]的讨论，了解有关分区消息传送实体的详细信息。

  [Service Bus architecture]: /documentation/articles/service-bus-architecture/
  [Azure portal]: https://portal.azure.cn
  [QueueDescription.EnablePartitioning]: https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.queuedescription#Microsoft_ServiceBus_Messaging_QueueDescription_EnablePartitioning
  [TopicDescription.EnablePartitioning]: https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.topicdescription#Microsoft_ServiceBus_Messaging_TopicDescription_EnablePartitioning
  [BrokeredMessage.SessionId]: https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.brokeredmessage#Microsoft_ServiceBus_Messaging_BrokeredMessage_SessionId
  [BrokeredMessage.PartitionKey]: https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.brokeredmessage#Microsoft_ServiceBus_Messaging_BrokeredMessage_PartitionKey
  [SessionId]: https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.brokeredmessage#Microsoft_ServiceBus_Messaging_BrokeredMessage_SessionId
  [PartitionKey]: https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.brokeredmessage#Microsoft_ServiceBus_Messaging_BrokeredMessage_PartitionKey
  [QueueDescription.RequiresDuplicateDetection]: https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.queuedescription#Microsoft_ServiceBus_Messaging_QueueDescription_RequiresDuplicateDetection
  [BrokeredMessage.MessageId]: https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.brokeredmessage#Microsoft_ServiceBus_Messaging_BrokeredMessage_MessageId
  [MessageId]: https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.brokeredmessage#Microsoft_ServiceBus_Messaging_BrokeredMessage_MessageId
  [MessagingFactorySettings.OperationTimeout]: https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.messagingfactorysettings#Microsoft_ServiceBus_Messaging_MessagingFactorySettings_OperationTimeout
  [OperationTimeout]: https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.messagingfactorysettings#Microsoft_ServiceBus_Messaging_MessagingFactorySettings_OperationTimeout
  [QueueDescription.ForwardTo]: https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.queuedescription#Microsoft_ServiceBus_Messaging_QueueDescription_ForwardTo
  [针对服务总线分区队列和主题的 AMQP 1.0 支持]: /documentation/articles/service-bus-amqp-protocol-guide/

<!---HONumber=Mooncake_0213_2017-->
<!--Update_Description:update wording and link references-->