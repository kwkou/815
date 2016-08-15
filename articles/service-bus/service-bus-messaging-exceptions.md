<properties 
   pageTitle="服务总线消息传送异常 | Azure"
   description="服务总线消息传送异常和建议的操作列表。"
   services="service-bus"
   documentationCenter="na"
   authors="sethmanheim"
   manager="timlt"
   editor="tysonn" />
<tags 
   ms.service="service-bus"
    ms.date="05/02/2016"
   wacn.date="06/21/2016" />

# 服务总线消息传送异常

本文列出了 Azure 服务总线消息传送 API 生成的一些异常。这些参考信息可随时更改，请不时返回查看更新内容。

## 异常类别

消息传送 API 会生成以下类别的异常，以及你在尝试修复这些异常时可以采取的相关操作。请注意，异常的含义和原因会因消息传送实体的类型（中继、队列/主题、事件中心）而异：

1.  用户代码错误（[System.ArgumentException](https://msdn.microsoft.com/zh-cn/library/system.argumentexception.aspx)、[System.InvalidOperationException](https://msdn.microsoft.com/zh-cn/library/system.invalidoperationexception.aspx)、[System.OperationCanceledException](https://msdn.microsoft.com/zh-cn/library/system.operationcanceledexception.aspx)、[System.Runtime.Serialization.SerializationException](https://msdn.microsoft.com/zh-cn/library/system.runtime.serialization.serializationexception.aspx)）。常规操作：继续之前尝试修复代码。

2.  设置/配置错误（[Microsoft.ServiceBus.Messaging.MessagingEntityNotFoundException](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.messagingentitynotfoundexception.aspx)、[System.UnauthorizedAccessException](https://msdn.microsoft.com/zh-cn/zh-cn/library/system.unauthorizedaccessexception.aspx)）。常规操作：检查你的配置，必要时进行更改。

3.  暂时性异常（[Microsoft.ServiceBus.Messaging.MessagingException](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.messagingexception.aspx)、[Microsoft.ServiceBus.Messaging.ServerBusyException](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.serverbusyexception.aspx)、[Microsoft.ServiceBus.Messaging.MessagingCommunicationException](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.messagingcommunicationexception.aspx)）。常规操作：重试操作或通知用户。

4.  其他异常（[System.Transactions.TransactionException](https://msdn.microsoft.com/zh-cn/library/system.transactions.transactionexception.aspx)、[System.TimeoutException](https://msdn.microsoft.com/zh-cn/library/system.timeoutexception.aspx)、[Microsoft.ServiceBus.Messaging.MessageLockLostException](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.messagelocklostexception.aspx)、[Microsoft.ServiceBus.Messaging.SessionLockLostException](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.sessionlocklostexception.aspx)）。常规操作：特定于异常类型；请参考以下部分中的表。

## 异常类型

下表列出了消息异常的类型及其原因，并说明你可以采取的建议性操作。

| **异常类型** | **说明/原因/示例** | **建议的操作** | **自动/立即重试注意事项** |
|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------|
| [TimeoutException](https://msdn.microsoft.com/zh-cn/library/system.timeoutexception.aspx) | 服务器在 [OperationTimeout](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.messagingfactorysettings.operationtimeout.aspx) 控制的指定时间内未响应请求的操作。服务器可能已完成请求的操作。这可能是由于网络或其他基础结构延迟造成的。 | 检查系统状态的一致性，并根据需要重试。 | 在某些情况下，重试可能会有帮助；在代码中添加重试逻辑。 |
| [InvalidOperationException](https://msdn.microsoft.com/zh-cn/library/system.invalidoperationexception.aspx) | 不允许在服务器或服务中执行请求的用户操作。有关详细信息，请查看异常消息。例如，如果消息是在 **ReceiveAndDelete** 模式下收到的，则 [Complete](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.brokeredmessage.complete.aspx) 将生成此异常。 | 检查代码和文档。确保请求的操作有效。 | 重试不会解决问题。 |
| [OperationCanceledException](https://msdn.microsoft.com/zh-cn/library/system.operationcanceledexception.aspx) | 尝试对已关闭、中止或释放的对象调用某个操作。环境事务已释放，但这种情况很少见。 | 检查代码并确保代码不会对已释放的对象调用操作。 | 重试不会解决问题。 |
| [UnauthorizedAccessException](https://msdn.microsoft.com/zh-cn/library/system.unauthorizedaccessexception.aspx) | [TokenProvider](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.tokenprovider.aspx) 对象无法获取令牌、该令牌无效，或者令牌不包含执行操作所需的声明。 | 确保使用正确的值创建令牌提供程序。检查访问控制服务的配置。 | 在某些情况下，重试可能会有帮助；在代码中添加重试逻辑。 |
| [ArgumentException](https://msdn.microsoft.com/zh-cn/library/system.argumentexception.aspx)<br /> [ArgumentNullException](https://msdn.microsoft.com/zh-cn/library/system.argumentnullexception.aspx)<br />[ArgumentOutOfRangeException](https://msdn.microsoft.com/zh-cn/library/system.argumentoutofrangeexception.aspx) | 为方法提供的一个或多个参数无效。为 [NamespaceManager](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.namespacemanager.aspx) 或 [Create](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.messagingfactory.create.aspx) 提供的 URI 包含路径段。为 [NamespaceManager](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.namespacemanager.aspx) 或 [Create](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.messagingfactory.create.aspx) 提供的 URI 方案无效。属性值大于 32KB。 | 检查调用代码并确保参数正确。 | 重试不会解决问题。 |
| [MessagingEntityNotFoundException](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.messagingentitynotfoundexception.aspx) | 与操作关联的实体不存在或已被删除。 | 确保该实体存在。 | 重试不会解决问题。 |
| [MessageNotFoundException](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.messagenotfoundexception.aspx) | 尝试接收具有特定序列号的消息。找不到此消息。 | 确保该消息尚未接收。检查死信队列，以确定该消息是否被视为死信。 | 重试不会解决问题。 |
| [MessagingCommunicationException](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.messagingcommunicationexception.aspx) | 客户端无法与服务总线建立连接。 | 确保提供的主机名正确并且主机可访问。 | 如果存在间歇性的连接问题，重试可能会有帮助。 |
| [ServerBusyException](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.serverbusyexception.aspx) | 服务目前无法处理请求。 | 客户端可以等待一段时间，然后重试操作。 | 客户端可在特定的时间间隔后重试操作。如果重试导致其他异常，请检查该异常的重试行为。 |
| [MessageLockLostException](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.messagelocklostexception.aspx) | 与消息关联的锁令牌已过期，或者找不到锁令牌。 | 释放消息。 | 重试不会解决问题。 |
| [SessionLockLostException](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.sessionlocklostexception.aspx) | 与此会话关联的锁已丢失。 | 中止 [MessageSession](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.messagesession.aspx) 对象。 | 重试不会解决问题。 |
| [MessagingException](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.messagingexception.aspx) | 在以下情况下，可能会引发一般消息异常：尝试使用属于其他实体类型（例如主题）的名称或路径创建 [QueueClient](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.queueclient.aspx)。尝试发送大于 256KB 的消息。服务器或服务在处理请求期间遇到错误。有关详细信息，请查看异常消息。这通常是暂时性的异常。 | 检查代码，并确保只对消息正文使用可序列化对象（或使用自定义序列化程序）。在文档中查看属性支持的值类型，并只使用支持的类型。检查 [IsTransient](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.messagingexception.istransient.aspx) 属性。如果为 **true**，你可以重试操作。 | 重试行为的效果不确定，可能不会解决问题。 |
| [MessagingEntityAlreadyExistsException](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.messagingentityalreadyexistsexception.aspx) | 尝试使用已被该服务命名空间中另一实体使用的名称创建实体。 | 删除现有的实体，或者选择不同的名称来创建实体。 | 重试不会解决问题。 |
| [QuotaExceededException](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.quotaexceededexception.aspx) | 消息实体已达到其最大允许大小。 | 通过从实体或其子队列接收消息在该实体中创建空间。 | 如果同时已删除消息，则重试可能会有帮助。 |
| [RuleActionException](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.ruleactionexception.aspx) | 如果你尝试创建无效的规则操作，服务总线将返回此异常。如果在处理该消息的规则操作时出错，服务总线会将此异常附加到死信消息。 | 检查规则操作是否正确。 | 重试不会解决问题。 |
| [FilterException](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.filterexception.aspx) | 如果你尝试创建无效的筛选器，服务总线将返回此异常。如果在处理该消息的筛选器时出错，服务总线会将此异常附加到死信消息。 | 检查筛选器是否正确。 | 重试不会解决问题。 |
| [SessionCannotBeLockedException](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.sessioncannotbelockedexception.aspx) | 尝试接受具有特定会话 ID 的会话，但该会话当前已被另一客户端锁定。 | 确保该会话未由其他客户端锁定。 | 如果在此期间会话已释放，则重试可能会有帮助。 |
| [TransactionSizeExceededException](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.transactionsizeexceededexception_methods.aspx) | 事务包含过多的操作。 | 减少此事务中操作的数目。 | 重试不会解决问题。 |
| [MessagingEntityDisabledException](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.messagingentitydisabledexception.aspx) | 对已禁用的实体请求运行时操作。 | 激活实体。 | 如果在此期间该实体已激活，则重试可能会有帮助。 |
| [NoMatchingSubscriptionException](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.nomatchingsubscriptionexception.aspx) | 如果你向已启用预筛选的主题发送消息并且所有筛选器都不匹配，则服务总线将返回此异常。 | 确保至少有一个筛选器匹配。 | 重试不会解决问题。 |
| [MessageSizeExceededException](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.messagesizeexceededexception.aspx) | 消息负载超出 256K 限制。请注意，256K 限制是指总消息大小，可能包括系统属性和任何 .NET 开销。 | 减少消息负载的大小，然后重试操作。 | 重试不会解决问题。 |
| [TransactionException](https://msdn.microsoft.com/zh-cn/library/system.transactions.transactionexception.aspx) | 环境事务 (*Transaction.Current*) 无效。该事务可能已完成或已中止。内部异常可能提供了更多信息。| | 重试不会解决问题。| - |
[TransactionInDoubtException](https://msdn.microsoft.com/zh-cn/library/system.transactions.transactionindoubtexception.aspx) | 尝试对有问题的事务执行操作，或者尝试提交事务时事务出现问题。| 应用程序必须处理此异常（作为特殊情况），因为该事务可能已提交。| - |

## QuotaExceededException

[QuotaExceededException](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.quotaexceededexception.aspx) 指示已超过某个特定实体的配额。

### 队列和主题

对队列和主题而言，这通常指队列的大小。错误消息属性会包含更多详细信息，如以下示例所示：

```
Microsoft.ServiceBus.Messaging.QuotaExceededException
Message: The maximum entity size has been reached or exceeded for Topic: ¡®xxx-xxx-xxx¡¯. 
	Size of entity in bytes:1073742326, Max entity size in bytes:
1073741824..TrackingId:xxxxxxxxxxxxxxxxxxxxxxxxxx, TimeStamp:3/15/2013 7:50:18 AM
```

该消息指出，主题超过其大小限制，本例中为 1 GB（默认大小限制）。

#### 常见原因

此错误有两个常见的原因：死信队列和无法正常运行的消息接收器。

1. **死信队列**：读取器无法完成消息，当锁定过期后，消息会返回队列/主题。如果读取器发生异常，以致无法调用 [BrokeredMessage.Complete](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.brokeredmessage.complete.aspx)，就会出现这种情况。消息读取 10 次后，将默认移至死信队列。此行为由 [QueueDescription.MaxDeliveryCount](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.queuedescription.maxdeliverycount.aspx) 属性控制，默认值为 10。消息堆积在死信队列中会占用空间。

	若要解决此问题，请读取并完成死信队列中的消息，就像处理任何其他队列一样。[QueueClient](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.queueclient.aspx) 类甚至包含 [FormatDeadLetterPath](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.queueclient.formatdeadletterpath.aspx) 方法，可帮助格式化死信队列路径。

2. **接收器已停止**：接收器已停止接收队列或订阅中的消息。识别这种情况的方法是查看 [QueueDescription.MessageCountDetails](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.queuedescription.messagecountdetails.aspx) 属性，它会显示消息的完整细目。如果 [ActiveMessageCount](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.messagecountdetails.activemessagecount.aspx) 属性很高或不断增加，则表示消息写入的速度超过读取的速度。

### 事件中心

每个事件中心最多只能有 20 个使用者组。当你尝试创建更多组时，会收到 [QuotaExceededException](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.quotaexceededexception.aspx)。

### 中继

对于服务总线中继，此异常包含 [System.ServiceModel.QuotaExceededException](https://msdn.microsoft.com/zh-cn/library/system.servicemodel.quotaexceededexception.aspx)，指示已超过此终结点的最大侦听器数目。这会以异常消息的 **MaximumListenersPerEndpoint** 值表示。

## TimeoutException 

[TimeoutException](https://msdn.microsoft.com/zh-cn/library/system.timeoutexception.aspx) 指示用户启动的操作所用的时间超过操作超时值。

### 队列和主题

对于队列和主题，超时在 [MessagingFactorySettings.OperationTimeout](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.messagingfactorysettings.operationtimeout.aspx) 属性中作为连接字符串的一部分指定，或通过 [ServiceBusConnectionStringBuilder](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.servicebusconnectionstringbuilder.aspx) 指定。错误消息本身可能会有所不同，但它始终包含当前操作的指定超时值。

### 事件中心

对于事件中心，超时作为连接字符串的一部分指定，或通过 [ServiceBusConnectionStringBuilder](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.servicebusconnectionstringbuilder.aspx) 指定。错误消息本身可能会有所不同，但它始终包含当前操作的指定超时值。

### 中继

对于服务总线中继，你可能会在第一次打开中继发送器连接时收到超时异常。此异常有两个常见的原因：

1. [OpenTimeout](https://msdn.microsoft.com/zh-cn/library/wcf.opentimeout.aspx) 值可能太小（甚至几分之一秒）。
2. 本地中继侦听器可能无响应（或者可能遇到禁止侦听器接受新客户端连接的防火墙规则问题），而 [OpenTimeout](https://msdn.microsoft.com/zh-cn/library/wcf.opentimeout.aspx) 值约小于 20 秒。

例如：

```
'System.TimeoutException¡¯: The operation did not complete within the allotted timeout of 00:00:10. The time allotted to this operation may have been a portion of a longer timeout.
```

### 常见原因

此错误有两个常见的原因：配置不正确或暂时性服务错误。

1. **配置不正确**：运行条件下的操作超时值可能太小。客户端 SDK 的操作超时默认值为 60 秒。请查看代码是否将该值设置得过小。请注意，网络和 CPU 使用率的状况会影响完成特定操作所用的时间，因此，操作超时不应设置为非常小的值。

2. **暂时性服务错误**：有时，服务总线服务在处理请求时会遇到延迟，例如，高流量时段。在这种情况下，你可以在延迟后重试操作，直到操作成功为止。如果多次尝试同一操作后仍然失败，请访问 [Azure 服务状态站点](https://azure.microsoft.com/status/)，看是否有任何已知的服务中断。

## 后续步骤

有关服务总线和事件中心 .NET API 的完整参考，请参阅 [MSDN 上的 Azure 参考信息](https://msdn.microsoft.com/zh-cn/library/azure/mt419900.aspx)。

若要了解有关[服务总线](/home/features/messaging/)的详细信息，请参阅以下主题。

- [服务总线消息传送概述](/documentation/articles/service-bus-messaging-overview/)
- [服务总线基础知识](/documentation/articles/service-bus-fundamentals-hybrid-solutions/)
- [服务总线体系结构](/documentation/articles/service-bus-architecture/)

<!---HONumber=Mooncake_0307_2016-->