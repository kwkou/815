<properties
    pageTitle="自动转发服务总线消息实体 | Azure"
    description="如何将服务总线队列或订阅链接到另一个队列或主题。"
    services="service-bus"
    documentationCenter="na"
    authors="sethmanheim"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="f7060778-3421-402c-97c7-735dbf6a61e8"
    ms.service="service-bus"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="04/12/2017"
    wacn.date="05/22/2017"
    ms.author="sethm"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="8fd60f0e1095add1bff99de28a0b65a8662ce661"
    ms.openlocfilehash="5d256a6a63e4b209866d75c3d38cd830e2919090"
    ms.lasthandoff="05/12/2017" />

# <a name="chaining-service-bus-entities-with-auto-forwarding"></a>使用自动转发链接服务总线实体

通过服务总线*自动转发*功能可将队列或订阅链接到作为相同命名空间组成部分的另一个队列或主题。 启用自动转发时，服务总线会自动删除放置在第一个队列或订阅（源）中的消息，并将其放入第二个队列或主题（目标）中。 请注意，仍可将消息直接发送到目标实体。 另外，不能将子队列（如死信队列）链接到其他队列或主题。

## <a name="using-auto-forwarding"></a>使用自动转发
可通过在源的 [QueueDescription][QueueDescription] 或 [SubscriptionDescription][SubscriptionDescription] 对象上设置 [QueueDescription.ForwardTo][QueueDescription.ForwardTo] 或 [SubscriptionDescription.ForwardTo][SubscriptionDescription.ForwardTo] 属性来启用自动转发，如以下示例所示。

    SubscriptionDescription srcSubscription = new SubscriptionDescription (srcTopic, srcSubscriptionName);
    srcSubscription.ForwardTo = destTopic;
    namespaceManager.CreateSubscription(srcSubscription));

创建源实体时，目标实体必须存在。 如果目标实体不存在，则当创建源实体时，服务总线将返回异常。

可以使用自动转发来横向扩展单个主题。 服务总线将[给定主题的订阅数](/documentation/articles/service-bus-quotas/)限制为 2,000。 你可以通过创建二级主题来容纳其他订阅。 请注意，即使订阅数并未受到服务总线限制，添加二级主题也可以提高主题的整体吞吐量。

![自动转发方案][0]

自动转发还可用于分离消息发送方与接收方。 例如，考虑一个由以下三个模块组成的 ERP 系统：订单处理、库存管理和客户关系管理。 每个模块都将生成消息，这些消息将被排入相应的主题队。 Alice 和 Bob 是两名销售代表，他们想了解与其客户相关的所有消息。 要接收这些消息，Alice 和 Bob 各自创建一个个人队列和一个针对 ERP 主题的订阅，该订阅将所有消息自动转发给该队列。

![自动转发方案][1]

如果 Alice 去度假，则将填充她的个人队列，而不是 ERP 主题。 在此方案中，由于销售代表没有接收到任何消息，所以所有 ERP 主题都没有达到配额。

## <a name="auto-forwarding-considerations"></a>自动转发注意事项

如果目标实体累积了过多消息并超出配额，或禁用了目标实体，则源实体会将消息添加到其[死信队列](/documentation/articles/service-bus-dead-letter-queues/)，直到目标中存在可用空间（或重新启用了该实体）。 这些消息将继续位于死信队列中，因此你必须从死信队列显式接收和处理它们。

当将单个主题链接到一起以获取包含许多订阅的复合主题时，建议设置适量的一级主题订阅和许多二级主题订阅。例如，一个包含 20 个订阅的一级主题（其中每个订阅都链接到包含 200 个订阅的二级主题）就能够比包含 200 个订阅的一级主题（其中每个订阅链接到包含 20 个订阅的二级主题）具有更高的吞吐量。

服务总线对于每条转发的消息收取一个操作的费用。例如，将一条消息发送到包含 20 个订阅的主题，该主题中的每个订阅配置为将消息自动转发到另一个队列或主题，如果所有一级订阅都接收到消息副本，则按 21 个操作进行计费。

若要创建链接到另一个队列或主题的订阅，则订阅创建者必须具有源和目标实体的**管理**权限。 将消息发送到源主题仅需要源主题的**发送**权限。

## <a name="next-steps"></a>后续步骤

有关自动转发的详细信息，请参阅以下参考主题：

- [SubscriptionDescription.ForwardTo][]
- [QueueDescription][]
- [SubscriptionDescription][]

若要深入了解服务总线性能提升，请参阅：

- [使用服务总线消息传送改进性能的最佳做法](/documentation/articles/service-bus-performance-improvements/)
- [分区消息实体][]。

  [QueueDescription.ForwardTo]: https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.queuedescription#Microsoft_ServiceBus_Messaging_QueueDescription_ForwardTo
  [SubscriptionDescription.ForwardTo]: https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.subscriptiondescription#Microsoft_ServiceBus_Messaging_SubscriptionDescription_ForwardTo
  [QueueDescription]: https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.queuedescription
  [SubscriptionDescription]: https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.queuedescription
  [0]: ./media/service-bus-auto-forwarding/IC628631.gif
  [1]: ./media/service-bus-auto-forwarding/IC628632.gif
  [分区消息实体]: /documentation/articles/service-bus-partitioning/
  
<!---HONumber=Mooncake_Quality_Review_0117_2017-->
<!--Update_Description:update meta properties--> 