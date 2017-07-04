<properties
    pageTitle="Azure 事件中心消息传送异常 | Azure"
    description="Azure 事件中心消息传送异常和建议操作列表。"
    services="event-hubs"
    documentationcenter="na"
    author="sethmanheim"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="2c6273de-0106-47e5-b45d-59040e51f2c5"
    ms.service="event-hubs"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="02/07/2017"
    wacn.date="03/24/2017"
    ms.author="sethm" />  


# 事件中心消息传送异常

本文列出 Azure 服务总线消息传送 API 生成的一些异常，其中包括事件中心。此参考信息随时更改，请不时返回查看更新内容。

## 异常类别

事件中心 API 会生成以下类别的异常，以及在尝试修复这些异常时可以采取的相关操作。

1. 用户代码错误：[System.ArgumentException](https://msdn.microsoft.com/zh-cn/library/system.argumentexception.aspx)、[System.InvalidOperationException](https://msdn.microsoft.com/zh-cn/library/system.invalidoperationexception.aspx)、[System.OperationCanceledException](https://msdn.microsoft.com/zh-cn/library/system.operationcanceledexception.aspx)、[System.Runtime.Serialization.SerializationException](https://msdn.microsoft.com/zh-cn/library/system.runtime.serialization.serializationexception.aspx)。常规操作：继续之前尝试修复代码。

2. 设置/配置错误：[Microsoft.ServiceBus.Messaging.MessagingEntityNotFoundException](https://docs.microsoft.com/zh-cn/dotnet/api/microsoft.servicebus.messaging.messagingentitynotfoundexception)、[Microsoft.Azure.EventHubs.MessagingEntityNotFoundException](https://docs.microsoft.com/zh-cn/dotnet/api/microsoft.azure.eventhubs.messagingentitynotfoundexception)、[System.UnauthorizedAccessException](https://msdn.microsoft.com/zh-cn/library/system.unauthorizedaccessexception.aspx)。常规操作：检查配置，必要时进行更改。

3. 暂时性异常：[Microsoft.ServiceBus.Messaging.MessagingException](https://docs.microsoft.com/zh-cn/dotnet/api/microsoft.servicebus.messaging.messagingexception)、[Microsoft.ServiceBus.Messaging.ServerBusyException](#serverbusyexception)、[Microsoft.Azure.EventHubs.ServerBusyException](#serverbusyexception)、[Microsoft.ServiceBus.Messaging.MessagingCommunicationException](https://docs.microsoft.com/zh-cn/dotnet/api/microsoft.servicebus.messaging.messagingcommunicationexception)。常规操作：重试操作或通知用户。

4. 其他异常：[System.Transactions.TransactionException](https://msdn.microsoft.com/zh-cn/library/system.transactions.transactionexception.aspx)、[System.TimeoutException](#timeoutexception)、[Microsoft.ServiceBus.Messaging.MessageLockLostException](https://docs.microsoft.com/zh-cn/dotnet/api/microsoft.servicebus.messaging.messagelocklostexception)、[Microsoft.ServiceBus.Messaging.SessionLockLostException](https://docs.microsoft.com/zh-cn/dotnet/api/microsoft.servicebus.messaging.sessionlocklostexception)。常规操作：特定于异常类型；请参考以下部分中的表。

## 异常类型

下表列出了消息传送异常类型及其原因，并说明可以采取的建议操作。

| **异常类型** | **说明/原因/示例** | **建议的操作** | **自动/立即重试注意事项** |
| --- | --- | --- | --- |
| [TimeoutException](https://msdn.microsoft.com/zh-cn/library/system.timeoutexception.aspx) |服务器在 [OperationTimeout](https://docs.microsoft.com/zh-cn/dotnet/api/microsoft.servicebus.messaging.messagingfactorysettings#Microsoft_ServiceBus_Messaging_MessagingFactorySettings_OperationTimeout) 控制的指定时间内未响应请求的操作。服务器可能已完成请求的操作。这可能是由于网络或其他基础结构延迟造成的。 |检查系统状态的一致性，并根据需要重试。<br /> 请参阅 [TimeoutException](#timeoutexception)。 |在某些情况下，重试可能会有帮助；在代码中添加重试逻辑。 |
| [InvalidOperationException](https://msdn.microsoft.com/zh-cn/library/system.invalidoperationexception.aspx) |不允许在服务器或服务中执行请求的用户操作。有关详细信息，请查看异常消息。例如，如果消息是在 [ReceiveAndDelete](https://docs.microsoft.com/zh-cn/dotnet/api/microsoft.servicebus.messaging.receivemode) 模式下收到的，则 [Complete](https://docs.microsoft.com/zh-cn/dotnet/api/microsoft.servicebus.messaging.brokeredmessage#Microsoft_ServiceBus_Messaging_BrokeredMessage_Complete) 会生成此异常。 |检查代码和文档。确保请求的操作有效。 |重试不会解决问题。 |
| [OperationCanceledException](https://msdn.microsoft.com/zh-cn/library/system.operationcanceledexception.aspx) |尝试对已关闭、中止或释放的对象调用某个操作。在极少数情况下，环境事务已释放。 |检查代码并确保代码不会对已释放的对象调用操作。 |重试不会解决问题。 |
| [UnauthorizedAccessException](https://msdn.microsoft.com/zh-cn/library/system.unauthorizedaccessexception.aspx) |[TokenProvider](https://docs.microsoft.com/zh-cn/dotnet/api/microsoft.servicebus.tokenprovider) 对象无法获取令牌、该令牌无效，或者令牌不包含执行操作所需的声明。 |确保使用正确的值创建令牌提供程序。检查访问控制服务的配置。 |在某些情况下，重试可能会有帮助；在代码中添加重试逻辑。 |
| [ArgumentException](https://msdn.microsoft.com/zh-cn/library/system.argumentexception.aspx)<br /> [ArgumentNullException](https://msdn.microsoft.com/zh-cn/library/system.argumentnullexception.aspx)<br />[ArgumentOutOfRangeException](https://msdn.microsoft.com/zh-cn/library/system.argumentoutofrangeexception.aspx) |提供给方法的一个或多个参数无效。提供给 [NamespaceManager](https://docs.microsoft.com/zh-cn/dotnet/api/microsoft.servicebus.namespacemanager) 或 [Create](https://docs.microsoft.com/zh-cn/dotnet/api/microsoft.servicebus.messaging.messagingfactory#Microsoft_ServiceBus_Messaging_MessagingFactory_Create_System_Collections_Generic_IEnumerable_System_Uri__) 的 URI 中包含路径段。提供给 [NamespaceManager](https://docs.microsoft.com/zh-cn/dotnet/api/microsoft.servicebus.namespacemanager) 或 [Create](https://docs.microsoft.com/zh-cn/dotnet/api/microsoft.servicebus.messaging.messagingfactory#Microsoft_ServiceBus_Messaging_MessagingFactory_Create_System_Collections_Generic_IEnumerable_System_Uri__) 的 URI 方案无效。属性值大于 32KB。 |检查调用代码并确保参数正确。 |重试不会解决问题。 |
| [Microsoft.ServiceBus.Messaging.MessagingEntityNotFoundException](https://docs.microsoft.com/zh-cn/dotnet/api/microsoft.servicebus.messaging.messagingentitynotfoundexception)<br />[Microsoft.Azure.EventHubs.MessagingEntityNotFoundException](https://docs.microsoft.com/zh-cn/dotnet/api/microsoft.azure.eventhubs.messagingentitynotfoundexception) |与操作关联的实体不存在或已删除。 |确保该实体存在。 |重试不会解决问题。 |
| [MessageNotFoundException](https://docs.microsoft.com/zh-cn/dotnet/api/microsoft.servicebus.messaging.messagenotfoundexception) |尝试接收具有特定序列号的消息。找不到此消息。 |确保该消息尚未接收。检查死信队列，以查看是否将该消息视为死信。 |重试不会解决问题。 |
| [MessagingCommunicationException](https://docs.microsoft.com/zh-cn/dotnet/api/microsoft.servicebus.messaging.messagingcommunicationexception) |客户端无法与事件中心建立连接。 |确保提供的主机名正确并且主机可访问。 |如果存在间歇性的连接问题，重试可能会有帮助。 |
| [Microsoft.ServiceBus.Messaging.ServerBusyException](https://docs.microsoft.com/zh-cn/dotnet/api/microsoft.servicebus.messaging.serverbusyexception)<br />[Microsoft.Azure.EventHubs.ServerBusyException](https://docs.microsoft.com/zh-cn/dotnet/api/microsoft.azure.eventhubs.serverbusyexception) |服务目前无法处理请求。 |客户端可以等待一段时间，然后重试操作。<br />请参阅 [ServerBusyException](#serverbusyexception)。 |客户端可在特定的时间间隔后重试操作。如果重试导致其他异常，请检查该异常的重试行为。 |
| [MessageLockLostException](https://docs.microsoft.com/zh-cn/dotnet/api/microsoft.servicebus.messaging.messagelocklostexception) |与消息关联的锁令牌已过期，或者找不到锁令牌。 |释放消息。 |重试不会解决问题。 |
| [SessionLockLostException](https://docs.microsoft.com/zh-cn/dotnet/api/microsoft.servicebus.messaging.sessionlocklostexception) |与此会话关联的锁已丢失。 | 中止 [MessageSession](https://docs.microsoft.com/zh-cn/dotnet/api/microsoft.servicebus.messaging.messagesession) 对象。 |重试不会解决问题。 |
| [MessagingException](https://docs.microsoft.com/zh-cn/dotnet/api/microsoft.servicebus.messaging.messagingexception) |在以下情况下，可能会引发一般消息传送异常：尝试使用属于其他实体类型（例如，主题）的名称或路径创建 [QueueClient](https://docs.microsoft.com/zh-cn/dotnet/api/microsoft.servicebus.messaging.queueclient)。尝试发送大于 256KB 的消息。服务器或服务在处理请求期间遇到错误。有关详细信息，请查看异常消息。这通常是暂时性异常。 |检查代码，并确保只对消息正文使用可序列化对象（或使用自定义序列化程序）。在文档中查看属性支持的值类型，并只使用支持的类型。检查 [IsTransient](https://docs.microsoft.com/zh-cn/dotnet/api/microsoft.servicebus.messaging.messagingexception#Microsoft_ServiceBus_Messaging_MessagingException_IsTransient) 属性。如果为 **true**，你可以重试操作。 |重试行为的效果不确定，可能不会解决问题。 |
| [MessagingEntityAlreadyExistsException](https://docs.microsoft.com/zh-cn/dotnet/api/microsoft.servicebus.messaging.messagingentityalreadyexistsexception) |尝试使用已由该服务命名空间中另一实体使用的名称创建实体。 |删除现有的实体，或者选择不同的名称创建实体。 |重试不会解决问题。 |
| [QuotaExceededException](https://docs.microsoft.com/zh-cn/dotnet/api/microsoft.servicebus.messaging.quotaexceededexception) |消息实体已达到其最大允许大小。如果每个使用者组级别上已打开的接收方数量达到最大值（即 5），就会出现此异常。 |通过从实体或其子队列接收消息在该实体中创建空间。<br />请参阅 [QuotaExceededException](#quotaexceededexception) |如果同时已删除消息，重试可能会有帮助。 |
| | | | |
| [SessionCannotBeLockedException](https://docs.microsoft.com/zh-cn/dotnet/api/microsoft.servicebus.messaging.sessioncannotbelockedexception) |尝试接受具有特定会话 ID 的会话，但该会话当前已由另一客户端锁定。 |确保该会话未由其他客户端锁定。 |如果在此期间会话已释放，重试可能会有帮助。 |
| [TransactionSizeExceededException](https://docs.microsoft.com/zh-cn/dotnet/api/microsoft.servicebus.messaging.transactionsizeexceededexception) |事务包含过多的操作。 |减少此事务中操作的数目。 |重试不会解决问题。 |
| [MessagingEntityDisabledException](https://docs.microsoft.com/zh-cn/dotnet/api/microsoft.servicebus.messaging.messagingentitydisabledexception) |对已禁用的实体请求运行时操作。 |激活实体。 |如果在此期间该实体已激活，重试可能会有帮助。 |
| [Microsoft.ServiceBus.Messaging.MessageSizeExceededException](https://docs.microsoft.com/zh-cn/dotnet/api/microsoft.servicebus.messaging.messagesizeexceededexception)<br />[Microsoft.Azure.EventHubs.MessageSizeExceededException](https://docs.microsoft.com/zh-cn/dotnet/api/microsoft.azure.eventhubs.messagesizeexceededexception) | 消息负载超出 256K 限制。请注意，256K 限制是指总消息大小，可能包括系统属性和任何 .NET 开销。 |减少消息负载的大小，然后重试操作。 |重试不会解决问题。 |
| [TransactionException](https://msdn.microsoft.com/zh-cn/library/system.transactions.transactionexception.aspx) |环境事务 (*Transaction.Current*) 无效。该事务可能已完成或已中止。内部异常可能会提供其他信息。 | |重试不会解决问题。 |
| [TransactionInDoubtException](https://msdn.microsoft.com/zh-cn/library/system.transactions.transactionindoubtexception.aspx) |尝试对有问题的事务执行操作，或者尝试提交事务时事务出现问题。|应用程序必须处理此异常（作为特殊情况），因为该事务可能已提交。|- |

## <a id="quotaexceededexception"></a> QuotaExceededException

[QuotaExceededException](https://docs.microsoft.com/zh-cn/dotnet/api/microsoft.servicebus.messaging.quotaexceededexception) 指示已超过某个特定实体的配额。

如果每个使用者组级别上已打开的接收方数量达到最大值 (5)，就会出现此异常。

### 事件中心

每个事件中心最多只能有 20 个使用者组。当你尝试创建更多组时，会收到 [QuotaExceededException](https://docs.microsoft.com/zh-cn/dotnet/api/microsoft.servicebus.messaging.quotaexceededexception)。

## <a id="timeoutexception"></a> TimeoutException 

[TimeoutException](https://msdn.microsoft.com/zh-cn/library/system.timeoutexception.aspx) 指示用户启动的操作所用的时间超过操作超时值。

对于事件中心，超时作为连接字符串的一部分指定，或通过 [ServiceBusConnectionStringBuilder](https://docs.microsoft.com/zh-cn/dotnet/api/microsoft.servicebus.servicebusconnectionstringbuilder) 指定。错误消息本身可能会有所不同，但它始终包含为当前操作指定的超时值。

### 常见原因

此错误有两个常见的原因：配置不正确或暂时性服务错误。

1. **配置不正确**：
	运行条件下的操作超时值可能太小。客户端 SDK 的操作超时默认值为 60 秒。请查看代码是否将该值设置得过小。请注意，网络和 CPU 使用率的状况会影响完成特定操作所用的时间，因此，操作超时不应设置为非常小的值。

2. **暂时性服务错误**：
	有时，事件中心服务在处理请求时会遇到延迟，例如，高流量时段。在这种情况下，你可以在延迟后重试操作，直到操作成功为止。如果多次尝试同一操作后仍然失败，请访问 [Azure 服务状态站点](/support/service-dashboard/)，看是否有任何已知的服务中断。

## <a id="serverbusyexception"></a> ServerBusyException

[Microsoft.ServiceBus.Messaging.ServerBusyException](https://docs.microsoft.com/zh-cn/dotnet/api/microsoft.servicebus.messaging.serverbusyexception) 或 [Microsoft.Azure.EventHubs.ServerBusyException](https://docs.microsoft.com/zh-cn/dotnet/api/microsoft.azure.eventhubs.serverbusyexception) 指示服务器重载。此异常有两个相关的错误代码。

### 错误代码 50002

导致此错误发生的原因可能是以下之一：

1. 未在事件中心的所有分区上均匀分发负载，并且一个分区达到了本地吞吐量单位限制。
	
    解决方法：修改分区分发策略，或尝试 [EventHubClient.Send(eventDataWithOutPartitionKey)](https://docs.microsoft.com/zh-cn/dotnet/api/microsoft.servicebus.messaging.eventhubclient#Microsoft_ServiceBus_Messaging_EventHubClient_Send_Microsoft_ServiceBus_Messaging_EventData_) 可能会有所帮助。

2. 事件中心命名空间没有足够的吞吐量单位（可在 [Azure 门户](https://portal.azure.cn)中检查“事件中心命名空间”边栏选项卡上的“指标”边栏选项卡来确认）。请注意，门户显示聚合（1 分钟）的信息，但吞吐量是实时测量的，因此吞吐量只是一个估计值。

     解决方法：增加命名空间上的吞吐量单位可能会有所帮助。可在门户上“事件中心命名空间”边栏选项卡的“缩放”边栏选项卡中执行此操作。

### 错误代码 50001

此错误很少发生。但如果为命名空间运行代码的容器的 CPU 比较低时（在事件中心负载均衡器开始运行前几秒钟内），则可能发生此错误。

## 后续步骤
访问以下链接可以了解有关事件中心的详细信息：

* [事件中心概述](/documentation/articles/event-hubs-what-is-event-hubs/)
* [创建事件中心](/documentation/articles/event-hubs-create/)
* [事件中心常见问题](/documentation/articles/event-hubs-faq/)

<!---HONumber=Mooncake_0320_2017-->
<!--Update_Description:update meta properties;wording update;update link reference-->