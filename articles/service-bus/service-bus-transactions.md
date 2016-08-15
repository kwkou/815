<properties 
    pageTitle="服务总线事务处理 | Azure" 
    description="Azure 服务总线原子事务和发送方式概述" 
    services="service-bus" 
    documentationCenter=".net" 
    authors="sethmanheim" 
    manager="timlt" 
    editor=""/>

<tags
    ms.service="service-bus"
    ms.date="05/16/2016"
    wacn.date="06/27/2016"/>

# 服务总线事务处理概述

本文将讨论 Azure 服务总线的事务功能。很多讨论已在 [Atomic Transactions with Service Bus sample（服务总线中的原子事务示例）](https://github.com/Azure-Samples/azure-servicebus-messaging-samples/tree/master/AtomicTransactions)中进行了说明。本文仅限于概述服务总线中的事务处理和发送方式功能，虽然原子事务示例在作用域内更广泛且更复杂。

## 服务总线中的事务

一个[事务](https://github.com/Azure-Samples/azure-servicebus-messaging-samples/tree/master/AtomicTransactions#what-are-transactions)将两个或更多操作组合成执行作用域。就本质而言，此类事务必须确保所有操作属于给定的操作组，无论联合成功还是失败。在这方面，事务作为一个单元进行操作，通常称为原子性。

服务总线是事务性消息代理，并确保针对其消息存储的所有内部操作的事务完整性。服务总线内部的所有消息传输，如将消息移到[死信队列](/documentation/articles/service-bus-dead-letter-queues/)或在实体之间[自动转发](/documentation/articles/service-bus-auto-forwarding/)消息，都是事务性的。因此，如果服务总线接受一条消息，则该消息已存储并标有一个序列号。从那时起，服务总线内的任何消息传输都是实体之间协调的操作，将从不会导致消息丢失（源成功而目标失败）或重复（源失败而目标成功）。

服务总线支持针对事务作用域内的单个消息实体（队列、主题、订阅）进行分组操作。例如，可以从事务作用域内将多条消息发送到一个队列，在事务成功完成时，这些消息将仅提交到该队列的日志。

## 事务作用域内的操作 

可以在事务作用域内执行的操作如下所示：

- **[QueueClient](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.queueclient.aspx), [MessageSender](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.messagesender.aspx), [TopicClient](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.topicclient.aspx)**：Send, SendAsync, SendBatch, SendBatchAsync 

- **[BrokeredMessage](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.brokeredmessage.aspx)**：Complete, CompleteAsync, Abandon, AbandonAsync, Deadletter, DeadletterAsync, Defer, DeferAsync, RenewLock, RenewLockAsync

不包括接收操作，因为假定应用程序在某个接收循环内使用 [ReceiveMode.PeekLock](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.receivemode.aspx) 模式或通过 [OnMessage](https://msdn.microsoft.com/zh-cn/library/azure/dn369601.aspx) 回调获取消息，而且只有那时才打开用于处理消息的事务作用域。

然后，消息的处置（完成、放弃、死信、延迟）将在事务作用域内进行，并依赖于在事务处理的整体结果。

## 传输和“发送方式”

若要启用将数据从队列到处理器，然后到另一个队列的事务性移交，服务总线支持传输。在传输操作中，发送方先将消息发送到“传输队列”，然后传输队列立即使用自动转发功能所依赖的同一强大传输实现将消息移到预期的目标队列。消息永远不会以对传输队列的使用者可见的方式提交到传输队列的日志中。

当传输队列本身是发送方的输入消息的源时，此事务功能的优势越明显。换而言之，服务总线在对输入消息执行完成（或延迟或死信）操作时，可以“通过”传输队列将消息传输到目标队列中，所有这一切都通过一个原子操作完成。

### 在代码中查看它

若要设置此类传输，需创建通过传输队列以目标队列为目标的消息发送方。你还将设置接收方，以便从该同一队列提取消息。例如：

```
var sender = this.messagingFactory.CreateMessageSender(destinationQueue, myQueueName);
var receiver = this.messagingFactory.CreateMessageReceiver(myQueueName);
```

简单事务处理随后使用这些元素，如以下示例所示：

```
var msg = receiver.Receive();

using (scope = new TransactionScope())
{
    // Do whatever work is required 

    var newmsg = ... // package the result 

    msg.Complete(); // mark the message as done
    sender.Send(newmsg); // forward the result

    scope.Complete(); // declare the transaction done
} 
```

## 后续步骤

有关服务总线队列的详细信息，请参阅以下文章：

- [使用自动转发链接服务总线实体](/documentation/articles/service-bus-auto-forwarding/)
- [自动转发示例](https://github.com/Azure-Samples/azure-servicebus-messaging-samples/tree/master/AutoForward)
- [服务总线中的原子事务示例](https://github.com/Azure-Samples/azure-servicebus-messaging-samples/tree/master/AtomicTransactions)
- [比较 Azure 队列和服务总线队列](/documentation/articles/service-bus-azure-and-service-bus-queues-compared-contrasted/)
- [如何使用服务总线队列](/documentation/articles/service-bus-dotnet-how-to-use-queues/)

<!---HONumber=Mooncake_0620_2016-->