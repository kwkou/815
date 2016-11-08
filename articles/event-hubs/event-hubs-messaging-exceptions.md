<properties 
    pageTitle="事件中心消息传送异常 | Azure"
    description="Azure 事件中心消息传送异常和建议操作列表。"
    services="event-hubs"
    documentationCenter="na"
    authors="sethmanheim"
    manager="timlt"
    editor="" />  

<tags 
    ms.service="event-hubs"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="10/14/2016"
    wacn.date="11/08/2016" />  


# 事件中心消息传送异常

本文列出 Azure 服务总线消息传送 API 生成的一些异常。此 API 集包括事件中心。这些参考信息可随时更改，请不时返回查看更新内容。

## 异常类别

消息传送 API 会生成以下类别的异常，以及你在尝试修复这些异常时可以采取的相关操作。请注意，异常的含义和原因会因消息传送实体的类型（中继、队列/主题、事件中心）而异：

1.  用户代码错误（[System.ArgumentException](https://msdn.microsoft.com/zh-cn/library/system.argumentexception.aspx)、[System.InvalidOperationException](https://msdn.microsoft.com/zh-cn/library/system.invalidoperationexception.aspx)、[System.OperationCanceledException](https://msdn.microsoft.com/zh-cn/library/system.operationcanceledexception.aspx)、[System.Runtime.Serialization.SerializationException](https://msdn.microsoft.com/zh-cn/library/system.runtime.serialization.serializationexception.aspx)）。常规操作：继续之前尝试修复代码。

2.  设置/配置错误（[Microsoft.ServiceBus.Messaging.MessagingEntityNotFoundException](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.messagingentitynotfoundexception.aspx)、[System.UnauthorizedAccessException](https://msdn.microsoft.com/zh-cn/library/system.unauthorizedaccessexception.aspx)）。常规操作：检查你的配置，必要时进行更改。

3.  暂时性异常（[Microsoft.ServiceBus.Messaging.MessagingException](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.messagingexception.aspx)、[Microsoft.ServiceBus.Messaging.ServerBusyException](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.serverbusyexception.aspx)、[Microsoft.ServiceBus.Messaging.MessagingCommunicationException](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.messagingcommunicationexception.aspx)）。常规操作：重试操作或通知用户。

4.  其他异常（[System.Transactions.TransactionException](https://msdn.microsoft.com/zh-cn/library/system.transactions.transactionexception.aspx)、[System.TimeoutException](https://msdn.microsoft.com/zh-cn/library/system.timeoutexception.aspx)、[Microsoft.ServiceBus.Messaging.MessageLockLostException](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.messagelocklostexception.aspx)、[Microsoft.ServiceBus.Messaging.SessionLockLostException](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.sessionlocklostexception.aspx)）。常规操作：特定于异常类型；请参考以下部分中的表。

## 异常类型

下表列出了消息异常的类型及其原因，并说明你可以采取的建议性操作。

| **异常类型** | **说明/原因/示例** | **建议的操作** | **自动/立即重试注意事项** |
|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------|
| [TimeoutException](https://msdn.microsoft.com/zh-cn/library/system.timeoutexception.aspx) | 服务器在 [OperationTimeout](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.messagingfactorysettings.operationtimeout.aspx) 控制的指定时间内未响应请求的操作。服务器可能已完成请求的操作。这可能是由于网络或其他基础结构延迟造成的。 | 检查系统状态的一致性，并根据需要重试。 | 在某些情况下，重试可能会有帮助；在代码中添加重试逻辑。 |
| [InvalidOperationException](https://msdn.microsoft.com/zh-cn/library/system.invalidoperationexception.aspx) | 不允许在服务器或服务中执行请求的用户操作。有关详细信息，请查看异常消息。例如，如果消息是在 **ReceiveAndDelete** 模式下收到的，则 [Complete](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.brokeredmessage.complete.aspx) 会生成此异常。 | 检查代码和文档。确保请求的操作有效。 | 重试不会解决问题。 |
| [OperationCanceledException](https://msdn.microsoft.com/zh-cn/library/system.operationcanceledexception.aspx) | 尝试对已关闭、中止或释放的对象调用某个操作。在极少数情况下，环境事务已释放。 | 检查代码并确保代码不会对已释放的对象调用操作。 | 重试不会解决问题。 |
| [UnauthorizedAccessException](https://msdn.microsoft.com/zh-cn/library/system.unauthorizedaccessexception.aspx) | [TokenProvider](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.tokenprovider.aspx) 对象无法获取令牌、该令牌无效，或者令牌不包含执行操作所需的声明。 | 确保使用正确的值创建令牌提供程序。检查访问控制服务的配置。 | 在某些情况下，重试可能会有帮助；在代码中添加重试逻辑。 |
| [ArgumentException](https://msdn.microsoft.com/zh-cn/library/system.argumentexception.aspx)<br /> [ArgumentNullException](https://msdn.microsoft.com/zh-cn/library/system.argumentnullexception.aspx)<br />[ArgumentOutOfRangeException](https://msdn.microsoft.com/zh-cn/library/system.argumentoutofrangeexception.aspx) | 提供给方法的一个或多个参数无效。提供给 [NamespaceManager](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.namespacemanager.aspx) 或 [Create](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.messagingfactory.create.aspx) 的 URI 中包含路径段。提供给 [NamespaceManager](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.namespacemanager.aspx) 或 [Create](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.messagingfactory.create.aspx) 的 URI 方案无效。属性值大于 32KB。 | 检查调用代码并确保参数正确。 | 重试不会解决问题。 |
| [MessagingEntityNotFoundException](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.messagingentitynotfoundexception.aspx) | 与操作关联的实体不存在或已被删除。 | 确保该实体存在。 | 重试不会解决问题。 |
| [MessageNotFoundException](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.messagenotfoundexception.aspx) | 尝试接收具有特定序列号的消息。找不到此消息。 | 确保该消息尚未接收。检查死信队列，以确定该消息是否被视为死信。 | 重试不会解决问题。 |
| [MessagingCommunicationException](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.messagingcommunicationexception.aspx) | 客户端无法与事件中心建立连接。 | 确保提供的主机名正确并且主机可访问。 | 如果存在间歇性的连接问题，重试可能会有帮助。 |
| [ServerBusyException](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.serverbusyexception.aspx) | 服务目前无法处理请求。 | 客户端可以等待一段时间，然后重试操作。 | 客户端可在特定的时间间隔后重试操作。如果重试导致其他异常，请检查该异常的重试行为。 |
| [MessageLockLostException](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.messagelocklostexception.aspx) | 与消息关联的锁令牌已过期，或者找不到锁令牌。 | 释放消息。 | 重试不会解决问题。 |
| [SessionLockLostException](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.sessionlocklostexception.aspx) | 与此会话关联的锁已丢失。 | 中止 [MessageSession](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.messagesession.aspx) 对象。 | 重试不会解决问题。 |
| [MessagingException](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.messagingexception.aspx) | 在以下情况下，可能会引发一般消息传送异常：尝试使用属于其他实体类型（例如，主题）的名称或路径创建 [QueueClient](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.queueclient.aspx)。尝试发送大于 256KB 的消息。服务器或服务在处理请求期间遇到错误。有关详细信息，请查看异常消息。这通常是暂时性的异常。 | 检查代码，并确保只对消息正文使用可序列化对象（或使用自定义序列化程序）。在文档中查看属性支持的值类型，并只使用支持的类型。检查 [IsTransient](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.messagingexception.istransient.aspx) 属性。如果为 **true**，你可以重试操作。 | 重试行为的效果不确定，可能不会解决问题。 |
| [MessagingEntityAlreadyExistsException](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.messagingentityalreadyexistsexception.aspx) | 尝试使用已被该服务命名空间中另一实体使用的名称创建实体。 | 删除现有的实体，或者选择不同的名称来创建实体。 | 重试不会解决问题。 |
| [QuotaExceededException](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.quotaexceededexception.aspx) | 消息实体已达到其最大允许大小。如果每个使用者组级别上已打开的接收方数量达到最大值（即 5），就会出现此异常。 | 通过从实体或其子队列接收消息在该实体中创建空间。 | 如果同时已删除消息，则重试可能会有帮助。                                                           
|
| [SessionCannotBeLockedException](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.sessioncannotbelockedexception.aspx) | 尝试接受具有特定会话 ID 的会话，但该会话当前已被另一客户端锁定。 | 确保该会话未由其他客户端锁定。 | 如果在此期间会话已释放，则重试可能会有帮助。 |
| [TransactionSizeExceededException](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.transactionsizeexceededexception_methods.aspx) | 事务包含过多的操作。 | 减少此事务中操作的数目。 | 重试不会解决问题。 |
| [MessagingEntityDisabledException](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.messagingentitydisabledexception.aspx) | 对已禁用的实体请求运行时操作。 | 激活实体。 | 如果在此期间该实体已激活，则重试可能会有帮助。 |
| [MessageSizeExceededException](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.messagesizeexceededexception.aspx) | 消息负载超出 256K 限制。请注意，256K 限制是指总消息大小，可能包括系统属性和任何 .NET 开销。 | 减少消息负载的大小，然后重试操作。 | 重试不会解决问题。 |
| [TransactionException](https://msdn.microsoft.com/zh-cn/library/system.transactions.transactionexception.aspx) | 环境事务 (*Transaction.Current*) 无效。该事务可能已完成或已中止。内部异常可能提供了更多信息。| | 重试不会解决问题。| - 
| [TransactionInDoubtException](https://msdn.microsoft.com/zh-cn/library/system.transactions.transactionindoubtexception.aspx) | 尝试对有问题的事务执行操作，或者尝试提交事务时事务出现问题。| 应用程序必须处理此异常（作为特殊情况），因为该事务可能已提交。| - |

## QuotaExceededException

[QuotaExceededException](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.quotaexceededexception.aspx) 指示已超过某个特定实体的配额。

如果每个使用者组级别上已打开的接收方数量达到最大值 (5)，就会出现此异常。

### 事件中心

每个事件中心最多只能有 20 个使用者组。当你尝试创建更多组时，会收到 [QuotaExceededException](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.quotaexceededexception.aspx)。

## TimeoutException 

[TimeoutException](https://msdn.microsoft.com/zh-cn/library/system.timeoutexception.aspx) 指示用户启动的操作所用的时间超过操作超时值。

### 事件中心

对于事件中心，超时作为连接字符串的一部分指定，或通过 [ServiceBusConnectionStringBuilder](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.servicebusconnectionstringbuilder.aspx) 指定。错误消息本身可能会有所不同，但它始终包含当前操作的指定超时值。

### 常见原因

此错误有两个常见的原因：配置不正确或暂时性服务错误。

1. **配置不正确**：
	运行条件下的操作超时值可能太小。客户端 SDK 的操作超时默认值为 60 秒。请查看代码是否将该值设置得过小。请注意，网络和 CPU 使用率的状况会影响完成特定操作所用的时间，因此，操作超时不应设置为非常小的值。

2. **暂时性服务错误**：
	有时，事件中心服务在处理请求时会遇到延迟，例如，高流量时段。在这种情况下，你可以在延迟后重试操作，直到操作成功为止。如果多次尝试同一操作后仍然失败，请访问 [Azure 服务状态站点](/support/service-dashboard/)，看是否有任何已知的服务中断。

## 后续步骤

有关服务总线和事件中心 .NET API 的完整参考，请参阅 [MSDN 上的 Azure 参考](https://msdn.microsoft.com/zh-cn/library/azure/mt419900.aspx)。

<!---HONumber=Mooncake_1031_2016-->