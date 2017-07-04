<properties
    pageTitle="使 Azure 服务总线应用程序免受服务中断和灾难影响 | Azure"
    description="介绍可用于保护应用程序免受潜在服务总线中断影响的技术。"
    services="service-bus"
    documentationcenter="na"
    author="sethmanheim"
    manager="timlt"
    editor="tysonn" />
<tags
    ms.assetid="fd9fa8ab-f4c4-43f7-974f-c876df1614d4"
    ms.service="service-bus"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="04/12/2017"
    ms.author="sethm"
    wacn.date="05/22/2017"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="8fd60f0e1095add1bff99de28a0b65a8662ce661"
    ms.openlocfilehash="ac80abc477d0a457ca89fc45e519d342bdcbcc15"
    ms.lasthandoff="05/12/2017" />
# <a name="best-practices-for-insulating-applications-against-service-bus-outages-and-disasters"></a>使应用程序免受服务总线中断和灾难影响的最佳实践
任务关键型应用程序必须连续运行，即使是在计划外中断或灾难发生时。本主题介绍可用于保护服务总线应用程序免受潜在的服务中断和灾难影响的技术。

中断定义为 Azure 服务总线暂时不可用。中断会影响服务总线的一些组件，例如消息存储空间，甚至整个数据中心。问题解决后，服务总线将恢复可用。通常情况下，中断不会导致消息或其他数据丢失。组件故障的一个示例是特定的消息存储空间不可用。数据中心范围中断的示例有数据中心电源故障或数据中心网络交换机故障。中断可能会持续几分钟到几天的时间。

灾难定义为服务总线缩放单位或数据中心永久丢失。数据中心可能会也可能不会恢复可用。通常，灾难将导致消息或其他数据的部分或全部丢失。灾难的示例包括火灾、洪灾或地震。

## <a name="current-architecture"></a>当前体系结构
服务总线使用多个消息存储空间来存储发送到队列或主题的消息。 将未分区的队列或主题分配到一个消息存储空间。 如果此消息存储空间不可用，则针对该队列或主题的所有操作将都失败。

所有服务总线消息传送实体（队列、主题、中继）都位于隶属于数据中心的同一服务命名空间中。 服务总线不允许对数据进行自动异地复制，也不允许一个命名空间跨多个数据中心。

## <a name="protecting-against-acs-outages"></a>针对 ACS 中断进行保护
如果使用 ACS 凭据，则当 ACS 凭据不可用时，客户端无法再获取令牌。 ACS 出现故障时具有令牌的客户端可以在令牌到期前继续使用服务总线。 默认令牌生存期为 3 小时。

若要针对 ACS 中断进行保护，请使用共享访问签名 (SAS) 令牌。在这种情况下，客户端使用密钥对自创令牌进行签名，从而与服务总线直接进行身份验证。不再需要调用 ACS。有关 SAS 令牌的详细信息，请参阅[服务总线身份验证][]。

## <a name="protecting-queues-and-topics-against-messaging-store-failures"></a>保护队列和主题免受消息存储空间故障的影响
将未分区的队列或主题分配到一个消息存储空间。 如果此消息存储空间不可用，则针对该队列或主题的所有操作将都失败。另一方面，分区的队列包括多个片段。每个片段存储在不同的消息存储空间中。当向分区的队列或主题发送消息时，服务总线会将该消息分配到其中一个片段。如果相应的消息存储空间不可用，则服务总线会将消息写入另一片段（如有可能）。有关分区实体的详细信息，请参阅[分区消息传送实体][]。

## <a name="protecting-against-datacenter-outages-or-disasters"></a>针对数据中心中断或灾难进行保护
若要允许在两个数据中心之间进行故障转移，你可以在每个数据中心中各创建一个服务总线服务命名空间。例如，服务总线服务命名空间 **contosoPrimary.servicebus.chinacloudapi.cn** 可能位于中国（北部）区域，而 **contosoSecondary.servicebus.chinacloudapi.cn** 可能位于中国（东部）区域。如果必须在数据中心中断的情况下仍可访问服务总线消息传送实体，你可以在两个命名空间中都创建该实体。

有关详细信息，请参阅[异步消息传送模式和高可用性][]中的“Azure 数据中心内的服务总线故障”部分。

## <a name="protecting-relay-endpoints-against-datacenter-outages-or-disasters"></a>保护中继终结点免受数据中心中断或灾难的影响
中继终结点的异地复制使得公开中继终结点的服务在服务总线中断时可用。若要实现异地复制，该服务必须在不同的命名空间中创建两个中继终结点。命名空间必须位于不同的数据中心，且两个终结点必须具有不同的名称。例如，可在 **contosoPrimary.servicebus.chinacloudapi.cn/myPrimaryService** 下访问主要终结点，而在 **contosoSecondary.servicebus.chinacloudapi.cn/mySecondaryService** 下访问其辅助终结点。

该服务随后侦听两个终结点，客户端可通过其中任一终结点调用服务。客户端应用程序随机选取一个中继作为主要终结点，并向活动终结点发送请求。如果操作失败并返回错误代码，此故障指示中继终结点不可用。应用程序将打开通向备份终结点的通道并重新发送请求。此时，活动终结点与备份终结点将互换角色：客户端应用程序会将旧的活动终结点认定为新的备份终结点，而将旧的备份终结点认定为新的活动终结点。如果两次发送操作都失败，则两个实体的角色将保持不变并返回错误。

[使用服务总线中继消息进行异地复制][]示例演示了如何复制中继。

## <a name="protecting-queues-and-topics-against-datacenter-outages-or-disasters"></a>保护队列和主题免受数据中心中断或灾难的影响
为了在使用中转消息传送时实现针对数据中心中断的恢复，服务总线支持两种方法：主动和被动复制。对于每一种方法，如果必须在数据中心中断的情况下仍可访问给定的队列或主题，可以在两个命名空间中创建。两个实体可以具有相同的名称。例如，可在 **contosoPrimary.servicebus.chinacloudapi.cn/myQueue** 下访问主要队列，而在 **contosoSecondary.servicebus.chinacloudapi.cn/myQueue** 下访问其辅助队列。

如果应用程序不需要发送方到接收方的持续通信，则该应用程序可实施一个用于防止消息丢失的持久客户端队列，从而保护发送方免受任何暂时性服务总线故障的影响。

## <a name="active-replication"></a>主动复制

主动复制对于每个操作都使用这两个命名空间中的实体。任何发送消息的客户端都将发送同一条消息的两个副本。第一个副本将发送到主要实体（例如 **contosoPrimary.servicebus.chinacloudapi.cn/sales**），该消息的第二个副本将发送到辅助实体（例如 **contosoSecondary.servicebus.chinacloudapi.cn/sales**）。

客户端从两个队列接收消息。如果接收方处理了消息的第一个副本，则第二个副本将被取消。要取消重复的消息，发送方必须用唯一标识符标记每一条消息。必须用同一标识符标记消息的两个副本。可使用 [BrokeredMessage.MessageId][] 或 [BrokeredMessage.Label][] 属性或自定义属性对消息进行标记。接收方必须保留已接收消息的列表。

[使用服务总线中转消息进行异地复制][]示例演示了消息传送实体的主动复制。

> [AZURE.NOTE]
> 主动复制方法将使操作数加倍，因此这种方法可能导致成本上升。
> 
> 

## <a name="passive-replication"></a>被动复制
在无故障的情况下，被动复制仅使用两个消息传送实体之一。客户端将消息发送给活动实体。如果针对活动实体的操作失败并返回错误代码，表明承载活动实体的数据中心可能不可用，则客户端将该消息的副本发送到备份实体。此时，活动实体与备份实体互换角色：进行发送的客户端将旧的活动实体认定为新的备份实体，而将旧的备份实体认定为新的活动实体。如果两次发送操作都失败，则两个实体的角色将保持不变并返回错误。

客户端从两个队列接收消息。因为接收方可能接收同一条消息的两个副本，所以接收方必须取消重复消息。可通过与主动复制中所述的相同方式取消重复消息。

一般来说，被动复制比主动重复更经济，因为在大多数情况下仅执行一个操作。延迟、吞吐量和货币成本均与非复制场景相同。

使用被动复制时，在以下情况下可能丢失消息或接收两次：

-   **消息延迟或丢失**：假定发送方将消息 m1 成功发送到主要队列，而该队列在接收方接收 m1 之前变为不可用。发送方将后续消息 m2 发送给辅助队列。如果主要队列是暂时不可用，则接收方将在该队列恢复可用后接收 m1。如果发生灾难，则接收方可能永远无法接收 m1。

-   **重复接收**：假定发件人将消息 m 发送给主要队列。服务总线成功处理了 m 但无法发送响应。发送操作超时后，发送方将向辅助队列发送 m 的一份相同副本。如果接收方能够在主要队列变为不可用之前接收 m 的第一个副本，则接收方将在几乎同一时间接收 m 的两个副本。如果接收方不能在主要队列变为不可用之前接收 m 的第一个副本，则接收方首先仅接收 m 的第二个副本，但将在主要队列变为可用后接收 m 的另一个副本。

[使用服务总线中转消息进行异地复制][]示例演示了消息传送实体的被动复制。

## <a name="next-steps"></a>后续步骤
若要了解有关灾难恢复的详细信息，请参阅这些文章：

- [Azure SQL 数据库业务连续性][]


  [服务总线身份验证]: /documentation/articles/service-bus-authentication-and-authorization/
  [分区消息传送实体]: /documentation/articles/service-bus-partitioning/
  [异步消息传送模式和高可用性]: /documentation/articles/service-bus-async-messaging/#failure-of-service-bus-within-an-azure-datacenter
  [使用服务总线中继消息进行异地复制]: http://code.msdn.microsoft.com/Geo-replication-with-16dbfecd
  [BrokeredMessage.MessageId]: https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.brokeredmessage#Microsoft_ServiceBus_Messaging_BrokeredMessage_MessageId
  [BrokeredMessage.Label]: https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.brokeredmessage#Microsoft_ServiceBus_Messaging_BrokeredMessage_Label
  [使用服务总线中转消息进行异地复制]: http://code.msdn.microsoft.com/Geo-replication-with-f5688664
  [Azure SQL 数据库业务连续性]: /documentation/articles/sql-database-business-continuity/

<!---HONumber=Mooncake_Quality_Review_0104_2017-->
<!--Update_Description:update meta properties--> 