<properties 
    pageTitle="服务总线死信队列 | Microsoft Azure" 
    description="Azure 服务总线死信队列概述" 
    services="service-bus" 
    documentationCenter=".net" 
    authors="sethmanheim" 
    manager="timlt" 
    editor=""/>

<tags
    ms.service="service-bus"
    ms.date="03/16/2016"
    wacn.date="04/05/2016"/>

# 服务总线死信队列概述

服务总线队列和主题订阅提供了一个辅助子队列，称为死信队列 (DLQ)。死信队列不需要显式创建，并且不能删除或以其他方式独立于主实体进行管理。

死信队列的用途是存放无法传递给任何接收方的消息或只是存放无法处理的消息。然后，可从 DLQ 中删除和检查这些消息。应用程序可能会在操作员的帮助下，更正问题并重新提交消息，记录出错的实际情况和/或执行更正操作。

从 API 和协议的角度看，DLQ 非常类似于任何其他队列，不同的是，消息只能通过父实体的死信笔势提交。此外，无法查看生存时间，而且不能将 DLQ 中的消息设为死信。死信队列完全支持扫视锁定传递和事务性操作。

请注意，DLQ 不自动执行清理。消息将保留在 DLQ 中，直到你显式从 DLQ 中检索它们以及对死信消息调用 [Complete()](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.brokeredmessage.completeasync.aspx) 为止。

## 将消息移到 DLQ

服务总线中有几个活动会导致从消息引擎本身将消息推送到 DLQ。应用程序也可以显式将消息推送到 DLQ。

如果消息是由代理移动的，在代理对消息调用它的内部版本的 [DeadLetter](https://msdn.microsoft.com/zh-cn/library/azure/hh291941.aspx) 方法时，会将两个属性添加到消息：`DeadLetterReason` 和 `DeadLetterErrorDescription`。

应用程序可以为 `DeadLetterReason` 属性定义自己的代码，但系统设置以下值。

| 条件 | DeadLetterReason | DeadLetterErrorDescription |
|---------------------------------------------------------------------------------------------------------------------------------------|-----------------------------|----------------------------------------------------------------------------------|
| 始终 | HeaderSizeExceeded | 已超过此流的大小配额。 |
| !TopicDescription。<br />EnableFilteringMessagesBeforePublishing 和 SubscriptionDescription。<br />EnableDeadLetteringOnFilterEvaluationExceptions | exception.GetType().Name | exception.Message |
| EnableDeadLetteringOnMessageExpiration | TTLExpiredException | 消息过期并已设为死信。 |
| SubscriptionDescription.RequiresSession | 会话ID 为 null。 | 启用会话的实体不允许使用会话标识符为 null 的消息。 |
| !死信队列 | MaxTransferHopCountExceeded | Null |
| 应用程序显式设为死信 | 由应用程序指定 | 由应用程序指定 |

## 超过 MaxDeliveryCount

每个队列和订阅都具有 [QueueDescription.MaxDeliveryCount](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.queuedescription.maxdeliverycount.aspx) 和 [SubscriptionDescription.MaxDeliveryCount](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.subscriptiondescription.maxdeliverycount.aspx) 属性；默认值为 10。只要消息在锁定 ([ReceiveMode.PeekLock](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.receivemode.aspx)) 的情况下传递，但已显式放弃或锁已过期，该消息的 [BrokeredMessage.DeliveryCount](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.brokeredmessage.deliverycount.aspx) 就会递增。当 [DeliveryCount](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.brokeredmessage.deliverycount.aspx) 超过 [MaxDeliveryCount](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.queuedescription.maxdeliverycount.aspx) 时，该消息将移到 DLQ，并指定 `MaxDeliveryCountExceeded` 原因代码。

无法禁止此行为，但你可以将 [MaxDeliveryCount](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.queuedescription.maxdeliverycount.aspx) 设置为非常大的数。

## 超过 TimeToLive

当 [QueueDescription.EnableDeadLetteringOnMessageExpiration](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.queuedescription.enabledeadletteringonmessageexpiration.aspx) 或 [SubscriptionDescription.EnableDeadLetteringOnMessageExpiration](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.subscriptiondescription.enabledeadletteringonmessageexpiration.aspx) 属性设置为 **true**（默认值是 **false**）时，所有到期的消息将移到 DLQ，并指定 `TTLExpiredException` 原因代码。

请注意，仅当至少有一个活动接收器在主队列或订阅上拉取时，过期的消息才会清除并因此移到 DLQ；该行为是设计使然。

## 处理订阅规则时出错

当为订阅启用了 [SubscriptionDescription.EnableDeadLetteringOnFilterEvaluationExceptions](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.subscriptiondescription.enabledeadletteringonfilterevaluationexceptions.aspx) 属性时，将在 DLQ 中捕获执行订阅的 SQL 筛选器规则时出现的任何错误以及有问题的消息。

## 应用程序级死信

除了系统提供的死信功能外，应用程序也可以使用 DLQ 显式拒绝无法接受的消息。这可能包括由于任何类型的系统问题而导致无法正确处理的消息、存储格式错误的负载的消息或在使用某种消息级安全方案时未通过身份验证的消息。

## 示例

下面的代码片段将创建一个消息接收器。在主队列的接收循环中，此代码使用 [Receive(TimeSpan.Zero)](https://msdn.microsoft.com/zh-cn/library/azure/dn130350.aspx) 检索消息，该方法请求代理立即返回随时可用的任何消息或返回空结果。如果此代码收到一条消息，它会立即放弃该消息并递增 `DeliveryCount`。系统将该消息移到 DLQ 后，主队列将为空，并且循环将退出，因为 [ReceiveAsync](https://msdn.microsoft.com/zh-cn/library/azure/dn130350.aspx) 返回 **null**。

```
var receiver = await receiverFactory.CreateMessageReceiverAsync(queueName, ReceiveMode.PeekLock);
while(true)
{
    var msg = await receiver.ReceiveAsync(TimeSpan.Zero);
    if (msg != null)
    {
        Console.WriteLine("Picked up message; DeliveryCount {0}", msg.DeliveryCount);
        await msg.AbandonAsync();
    }
    else
    {
        break;
    }
}
```

## 后续步骤

有关服务总线队列的详细信息，请参阅以下文章：

- [比较 Azure 队列和服务总线队列](/documentation/articles/service-bus-azure-and-service-bus-queues-compared-contrasted/)
- 如何使用[服务总线队列](/documentation/articles/service-bus-dotnet-how-to-use-queues/)

<!---HONumber=Mooncake_0328_2016-->