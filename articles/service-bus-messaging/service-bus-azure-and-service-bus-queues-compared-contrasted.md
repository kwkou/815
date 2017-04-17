<properties
    pageTitle="Azure 存储队列和服务总线队列 - 比较与对照 | Azure"
    description="分析 Azure 提供的两种队列类型之间的不同点和相似点。"
    services="service-bus"
    documentationCenter="na"
    author="sethmanheim"
    manager="timlt"
    editor="tysonn"
    translationtype="Human Translation" />
<tags
    ms.service="service-bus"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="tbd"
    ms.date="01/19/2017"
    wacn.date="04/17/2017"
    ms.author="sethm"
    ms.sourcegitcommit="7cc8d7b9c616d399509cd9dbdd155b0e9a7987a8"
    ms.openlocfilehash="e149117d53c0429f38cc1670d20fb617ed05e535"
    ms.lasthandoff="04/07/2017" />

# <a name="storage-queues-and-service-bus-queues---compared-and-contrasted"></a>存储队列和服务总线队列 - 比较与对照
本文分析 Azure 目前提供的以下两种队列类型之间的不同点和相似点：存储队列和服务总线队列。 通过使用该信息，你可以比较和对照这两种技术，并可以明智地决定哪种解决方案最符合你的需要。

## <a name="introduction"></a>介绍
Azure 支持两种队列机制：**存储队列**和**服务总线队列**。

**存储队列**是 [Azure 存储](/home/features/storage/)基础结构的一部分，采用基于 REST 的简单 Get/Put/Peek 接口，可在服务内部和服务之间提供可靠的持久性消息传送。

**服务总线队列**是更广的 [Azure 消息传送](/home/features/messaging/)基础结构的一部分，可支持队列以及发布/订阅和更高级的集成模式。 有关服务总线队列/主题/订阅的详细信息，请参阅[服务总线概述](/documentation/articles/service-bus-messaging-overview/)。

当两种队列技术同时存在时，首先引入的是存储队列，这是一种基于 Azure 存储服务构建的专用队列存储机制。 服务总线队列基于更广泛的“消息传送”基础结构而构建，旨在集成跨多个通信协议、数据约定、信任域和/或网络环境的应用程序或应用程序组件。

## <a name="technology-selection-considerations"></a>技术选择注意事项
存储队列和服务总线队列都是 Azure 目前提供的消息队列服务的实现形式。 两种队列的功能集略有不同，这意味着你可以选择其中一种，或者同时使用这两种，具体取决于你的特定解决方案或要解决的业务/技术问题的需要。

在确定哪种队列技术适合给定的解决方案时，解决方案架构师和开发人员应考虑以下建议。有关更多详细信息，请参阅下一部分。

作为解决方案架构师/开发人员，在以下情况下，**应考虑使用存储队列**：

- 你的应用程序需要在队列中存储超过 80 GB 的消息，这些消息的生存期不到 7 天。

- 你的应用程序需要跟踪队列内部消息的处理进度。这在处理消息的工作进程发生崩溃时很有用。然后，后续的工作进程可以使用该信息从之前的工作进程停止处继续。

- 你要求在服务器端记录针对你的队列执行的所有事务。

作为解决方案架构师/开发人员，在以下情况下，**应考虑使用服务总线队列**：

- 你的解决方案必须能够在无需轮询队列的情况下接收消息。有了服务总线，就可以使用服务总线支持的基于 TCP 的协议，通过长轮询接收操作实现这一点。

- 你的解决方案要求队列必须遵循先入先出 (FIFO) 的传递顺序。

- 你希望在 Azure 和 Windows Server（私有云）上获得对称体验。有关详细信息，请参阅[适用于 Windows Server 的服务总线](https://msdn.microsoft.com/zh-cn/library/dn282144.aspx)。

- 你的解决方案必须能够支持自动重复检测。

- 希望应用程序将消息作为长时间运行的并行流进行处理（使用消息的 [SessionId](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.brokeredmessage.sessionid.aspx) 属性，将消息与流相关联）。 在这种模式下，消费应用程序中的每个节点将竞争流而不是消息。 当流被提供给某个消费节点时，该节点可以使用事务检查应用程序流的状态。

- 你的解决方案在发送或接收来自队列的多个消息时，需要事务行为和原子性。

- 特定于应用程序的工作负荷的生存时间 (TTL) 特性可以超过 7 天的期限。

- 你的应用程序处理的消息介于 64 KB 和 256 KB 之间。

- 你需要向队列提供基于角色的访问模型，为发送方和接收方提供不同权限。

- 你的队列大小不会增长到超过 80 GB。

- 你希望使用基于 AMQP 1.0 标准的消息传送协议。 有关 AMQP 的详细信息，请参阅[服务总线 AMQP 概述](/documentation/articles/service-bus-amqp-overview/)。

- 你想要从基于队列的点到点通信最终迁移到消息交换模式，后者允许无缝集成其他接收者（订阅服务器），其中每个接收者都接收发送到该队列的某些消息或所有消息的独立副本。 消息交换模式是服务总线本机提供的发布/订阅功能。

- 你的消息传送解决方案必须能够支持“至多一次”传递保证，而无需构建其他基础结构组件。

- 你希望能够发布并使用消息批处理。

## <a name="comparing-storage-queues-and-service-bus-queues"></a>比较存储队列和服务总线队列
以下各部分中的表格提供队列功能的逻辑分组，使你可以一目了然地比较存储队列和服务总线队列中的功能。

## <a name="foundational-capabilities"></a>基础功能
本部分比较存储队列和服务总线队列提供的一些基础队列功能。

| 比较条件 | 存储队列 | 服务总线队列 |
|---|---|---|
|排序保障|**否** <br/><br>有关详细信息，请参阅“其他信息”部分中的第一个注意事项。</br>|**是 - 先进先出 (FIFO)**<br/><br>（通过使用消息传送会话）|
|传递保障|**至少一次**|**至少一次**<br/><br/>**至多一次**|
|原子操作支持|**否**|**是**<br/><br/>|
|接收行为|**非阻止**<br/><br/>（如果没有发现新消息，则立即完成）|**阻止超时/未超时**<br/><br/>（提供长轮询，或[“Comet 技术”](http://go.microsoft.com/fwlink/?LinkId=613759)）<br/><br/>**非阻止**<br/><br/>（通过仅使用 .NET 托管的 API）|
|推送样式 API|**否**|**是**<br/><br/>[OnMessage](https://msdn.microsoft.com/zh-cn/library/azure/jj908682.aspx) 和 **OnMessage** 会话 .NET API。|
|接收模式|**扫视与租赁**|**扫视与锁定**<br/><br/>**接收并删除**|
|独占访问模式|**基于租赁**|**基于锁定**|
|租赁/锁定持续时间|**30 秒（默认值）**<br/><br/>**7 天（最大值）**（可使用 [UpdateMessage](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.windowsazure.storage.queue.cloudqueue.updatemessage.aspx) API 续订或释放消息租赁。）|**60 秒（默认值）**<br/><br/>可使用 [RenewLock](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.brokeredmessage.renewlock.aspx) API 续订消息锁。|
|租赁/锁定精度|**消息级别**<br/><br/>（每条消息可具有不同的超时值，可在处理消息时，根据需要使用 [UpdateMessage](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.windowsazure.storage.queue.cloudqueue.updatemessage.aspx) API 来更新超时值）|**队列级别**<br/><br/>（每个队列都具有一个适用于其中所有消息的锁定精度，但是可使用 [RenewLock](https://msdn.microsoft.com/zh-cn/library/microsoft.servicebus.messaging.brokeredmessage.renewlock.aspx) API 续订该锁。）|
|成批接收|**是**<br/><br/>（在检索消息时显式指定消息计数，最多可达 32 条消息）|**是**<br/><br/>（隐式启用预提取属性或通过使用事务显式启用）|
|成批发送|**否**|**是**<br/><br/>（通过使用事务或客户端批处理）|

### <a name="additional-information"></a>其他信息
- 存储队列中的消息通常是先进先出的，但有时其顺序可能会颠倒。例如，当消息的可见性超时持续时间到期时（例如，由于客户端应用程序在处理过程中崩溃），就会发生这种情况。 当可见性超时到期时，消息会再次变成在队列上可见，让另一个工作进程能够取消它的排队。 此时，重新变成可见的消息可以放置在队列中（可以再次取消其排队），位于原先排在它后面的消息之后。
- 服务总线队列中有保障的 FIFO 模式要求使用消息传送会话。 处理以“扫视与锁定”模式接收的消息时，如果应用程序发生崩溃，则当下次队列接收者接受消息传送会话时，它将在失败消息的生存时间 (TTL) 期限过期后开始传递此消息。
- 存储队列可支持标准队列方案，例如解除应用程序组件之间的关联，增加可伸缩性和容错能力、进行负载分级，以及生成过程工作流。
- 服务总线队列支持“至少一次”传递保障  。 此外，可以通过使用会话状态来存储应用程序状态，并通过使用事务自动接收消息和更新会话状态，从而支持“最多一次”语义  。
- 存储队列可在多个队列、表和 Blob 上提供统一和一致的编程模型 – 对于开发人员和运营团队都是如此。
- 服务总线队列为单个队列的上下文中的本地事务提供支持。
- 服务总线支持的“接收与删除”模式提供了减少消息传送操作计数（和相关成本）以换取降低安全传递保证的能力。
- 存储队列提供租赁且可延长消息租赁时间。 这使工作进程能够对消息保持短的租赁时间。 因此，如果某个工作进程崩溃，则其他工作进程可以再次快速处理该消息。 此外，如果工作进程处理消息所需的时间比当前租赁时间长，则工作进程可以延长该消息的租赁时间。
- 存储队列提供了你可以针对消息入队或出队设置的可见性超时。 此外，你可以在运行时使用不同租赁值更新消息，并且可以跨同一队列的消息更新不同值。 服务总线锁定超时值在队列元数据中定义，但是你可以通过调用 [RenewLock](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.brokeredmessage.renewlock.aspx) 方法续订该锁。
- 服务总线队列中阻塞接收操作的最大超时值为 24 天。 但是，基于 REST 的最大超时值为 55 秒。
- 服务总线提供的客户端批处理允许队列客户端将多个消息纳入一个发送操作中进行批处理。 批处理仅适用于异步发送操作。
- 有些特性，例如存储队列的 200 TB 上限（当你虚拟化帐户时更多）和无限制队列，使得它成为 SaaS 提供商的理想平台。
- 存储队列提供灵活且性能良好的委托访问控制机制。

## <a name="advanced-capabilities"></a>高级功能
本部分对存储队列和服务总线队列提供的高级功能进行了比较。

| 比较条件 | 存储队列 | 服务总线队列 |
| --- | --- | --- |
| 计划的传递 |**是** |**是** |
| 自动死信 |**否** |**是** |
| 提高队列生存时间值 |**是**<br/><br/>（通过就地更新可见性超时） |**是**<br/><br/>（通过一个专用 API 功能提供） |
| 有害消息支持 |**是** |**是** |
| 就地更新 |**是** |**是** |
| 服务器端事务日志 |**是** |**否** |
| 存储度量值 |**是**<br/><br/>**分钟度量值**：提供可用性、TPS、API 调用计数、错误计数等指标的实时度量值，所有这些值都是实时的（每分钟进行汇总，并在生产过程中发生后几分钟之内报告）。 有关详细信息，请参阅[关于存储分析度量值](https://msdn.microsoft.com/zh-cn/library/azure/hh343258.aspx)。 |**是**<br/><br/>（通过调用 [GetQueues](https://msdn.microsoft.com/zh-cn/library/azure/hh293128.aspx) 进行大容量查询） |
| 状态管理 |**否** |**是**<br/><br/>[Microsoft.ServiceBus.Messaging.EntityStatus.Active](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.entitystatus.aspx)、[Microsoft.ServiceBus.Messaging.EntityStatus.Disabled](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.entitystatus.aspx)、[Microsoft.ServiceBus.Messaging.EntityStatus.SendDisabled](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.entitystatus.aspx)、[Microsoft.ServiceBus.Messaging.EntityStatus.ReceiveDisabled](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.entitystatus.aspx) |
| 消息自动转发 |**否** |**是** |
| 清除队列函数 |**是** |**否** |
| 消息组 |**否** |**是**<br/><br/>（通过使用消息传送会话） |
| 每个消息组的应用程序状态 |**否** |**是** |
| 重复检测 |**否** |**是**<br/><br/>（可在发送端配置） |
| 浏览消息组 |**否** |**是** |
| 按 ID 提取消息会话 |**否** |**是** |

### <a name="additional-information"></a>其他信息
* 两种队列技术都允许将消息安排在以后传送。
* 利用队列自动转发功能，数千个队列可将它们的消息自动转发至单个队列，而接收应用程序将使用来自该队列的消息。 你可以使用此机制来实现安全性和控制流，并且隔离每个消息发布者的存储。
* 存储队列为更新消息内容提供支持。 可以使用此功能将状态信息和递增进度更新持久保留到消息中，以便能够从上一个已知的检查点处理该消息，而不是从头开始。 借助服务总线队列，你可以通过使用消息会话实现相同的方案。 会话允许保存和检索应用程序处理状态（通过使用 [SetState](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.messagesession.setstate.aspx) 和 [GetState](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.messagesession.getstate.aspx)）。
* [死信](/documentation/articles/service-bus-dead-letter-queues/)（仅受服务总线队列支持）可用于隔离接收应用程序无法成功处理的消息，或隔离由于生存时间 (TTL) 属性过期而无法到达其目的地的消息。 TTL 值指定消息在队列中保留的时间。 对于服务总线，在 TTL 期限过期时，该消息将移到一个特殊的队列（称为 $DeadLetterQueue）中。
* 若要查找存储队列中的“有害”消息，则在消息取消排队时，应用程序会检查该消息的 **[DequeueCount](https://msdn.microsoft.com/zh-cn/library/azure/dd179474.aspx)** 属性。 如果 **DequeueCount** 超出给定的阈值，应用程序会将消息移到应用程序定义的“死信”队列中。
* 存储队列使你可以获取针对该队列执行的所有事务的详细日志以及聚合度量值。 这两个选项可用于调试和了解你的应用程序如何使用存储队列。 它们还用于对应用程序进行性能优化并降低使用队列的成本。
* 服务总线支持的“消息会话”概念允许属于特定逻辑组的消息与给定的接收方关联，而这样一来又能在消息与其各自接收方之间创建类似于会话的关联。 可以通过对消息设置 [SessionID](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.brokeredmessage.sessionid.aspx) 属性，在服务总线中启用此高级功能。 然后，接收方可以侦听特定会话 ID，并接收共享特定会话标识符的消息。
* 根据 [MessageId](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.brokeredmessage.messageid.aspx) 属性的值，服务总线队列支持的重复项检测功能会自动删除发送到队列或主题的重复消息。


## <a name="capacity-and-quotas"></a> 容量和配额
本部分从适用的[容量和配额](/documentation/articles/service-bus-quotas/)角度对存储队列和服务总线队列进行了比较。

| 比较条件 | 存储队列 | 服务总线队列 |
| --- | --- | --- |
| 最大队列大小 |**200 TB**<br/><br/>（限制为单个存储帐户容量） |**1 GB 到 80 GB**<br/><br/>（在创建队列和[启用分区](/documentation/articles/service-bus-partitioning/)时定义 – 请参阅“其他信息”部分） |
| 最大消息大小 |**64 KB**<br/><br/>（使用 **Base64** 编码时为 48 KB）<br/><br/>Azure 可以通过合并队列和 Blob 支持大消息 – 单个项目排队的消息最多达到 200GB。 |**256 KB** 或 **1 MB**<br/><br/>（包含标题和正文，最大标题大小：64 KB）。<br/><br/>取决于[服务层](/documentation/articles/service-bus-premium-messaging/)。 |
| 最大消息 TTL |**7 天** |**`TimeSpan.Max`** |
| 最大队列数 |**不受限制** |**10,000**<br/><br/>（每个服务命名空间，可以增加） |
| 并发客户端的最大数目 |**不受限制** |**不受限制**<br/><br/>（100 个并发连接限制仅适用于基于 TCP 协议的通信） |

### <a name="additional-information"></a>其他信息

- 服务总线强制实施队列大小限制。在创建队列时指定最大队列大小，其值可以在 1 至 80 GB 之间。如果达到创建队列时设置的队列大小值，则将拒绝其他传入消息，并且调用代码将收到一个异常。有关服务总线中配额的详细信息，请参阅[服务总线配额](/documentation/articles/service-bus-quotas/)。

- 你可以创建 1、2、3、4 或 5 GB 大小的服务总线队列（默认值为 1 GB）。启用分区（默认设置）时，服务总线将为你指定的每个 GB 创建 16 个分区。因此，如果你创建了一个大小为 5 GB 的队列，由于每 GB 16 个分区，最大队列大小将变为 (5 * 16) = 80 GB。可以通过在 [Azure 经典门户][]上查看分区队列或主题的条目，来了解该队列或主题的最大大小。

- 在存储队列中，如果消息的内容不属于 XML 安全内容，则必须对其进行 **Base64** 编码。如果你使用 **Base64** 编码此消息，则用户负载可高达 48 KB，而不是 64 KB。

- 对于服务总线队列，存储在队列中的每条消息由两个部分组成：标头和正文。消息的总大小不能超过服务层支持的最大消息大小。

- 当客户端通过 TCP 协议与服务总线队列进行通信时，到单个服务总线队列的最大并发连接数不得超过 100。此数值在发送方和接收方之间共享。如果达到此配额，将拒绝后续的其他连接请求，调用代码将收到一个异常。使用基于 REST 的 API 连接到队列的客户端不受此限制。

- 如果在单个服务总线命名空间中需要超过 10,000 个队列，你可以联系 Azure 支持团队并请求增加数目。若要使用服务总线扩展到 10,000 个以上的队列，你还可以使用 [Azure 经典管理门户][]创建其他命名空间。

## <a name="management-and-operations"></a>管理和操作
本部分对存储队列和服务总线队列提供的管理功能进行了比较。

| 比较条件 | 存储队列 | 服务总线队列 |
|---|---|---|
|管理协议|**基于 HTTP/HTTPS 的 REST**|**基于 HTTPS 的 REST**|
|运行时协议|**基于 HTTP/HTTPS 的 REST**|**基于 HTTPS 的 REST**<br/><br/>**AMQP 1.0 标准（具有 TLS 的 TCP）**|
| .NET API |**是**<br/><br/>（.NET 存储客户端 API） |**是**<br/><br/>（.NET 服务总线 API） |
| 本机 C++ |**是** |**是** |
|Java API|**是**|**是**|
|PHP API|**是**|**是**|
|Node.js API|**是**|**是**|
|任意元数据支持|**是**|**否**|
|队列命名规则|**长度最多为 63 个字符**<br/><br/>（队列名称中的字母必须小写。）|**长度最多为 260 个字符**<br/><br/>（队列路径和名称不区分大小写。）|
|获取队列长度函数|**是**<br/><br/>（在消息超出 TTL 但未删除的情况下的近似值。）|**是**<br/><br/>（精确时间点值。）|
|Peek 函数|**是**|**是**|

### <a name="additional-information"></a>其他信息

- 存储队列为可应用于队列说明的任意属性提供支持（以名称/值对形式）。

- 两种队列技术还提供无需锁定消息即可进行消息速览的功能，这在实现队列资源管理器/浏览器工具时可能非常有用。

- 服务总线 .NET 中转消息传送 API 利用全双工 TCP 连接，因此与基于 HTTP 的 REST 相比提高了性能，另外它们还支持 AMQP 1.0 标准协议。

- 存储队列名称长度可以在 3-63 个字符之间，可以包含小写字母、数字和连字符。 有关详细信息，请参阅 [命名队列和元数据](https://msdn.microsoft.com/zh-cn/library/azure/dd179349.aspx)。

- 服务总线队列名称长度最大可达 260 个字符，且命名规则限制较少。 服务总线队列名称可以包含字母、数字、句点、连字符和下划线。

## <a name="authentication-and-authorization"></a>身份验证和授权
本部分讨论存储队列和服务总线队列支持的身份验证和授权功能。

| 比较条件 | 存储队列 | 服务总线队列 |
| --- | --- | --- |
| 身份验证 |**对称密钥** |**对称密钥** |
| 安全模型 |通过 SAS 令牌进行的委托访问。 |SAS |
| 标识提供者联合 |**否** |**是** |

### <a name="additional-information"></a>其他信息

- 针对任一队列技术的每个请求都必须进行身份验证。不支持使用匿名访问的公共队列。使用 [SAS](/documentation/articles/service-bus-sas/)，可以通过发布只写 SAS、只读 SAS 甚至完全访问权限 SAS 来满足这种方案的需求。

- 存储队列提供的身份验证方案涉及对称密钥的运用，该密钥是基于哈希的消息身份验证代码 (HMAC)，使用 SHA-256 算法计算并编码为 **Base64** 字符串。有关相应协议的详细信息，请参阅 [Azure 存储空间服务的身份验证](https://msdn.microsoft.com/zh-cn/library/azure/dd179428.aspx)。服务总线队列支持使用对称密钥的类似模型。有关详细信息，请参阅[使用服务总线进行共享访问签名身份验证](/documentation/articles/service-bus-sas/)。

## 结束语
## <a name="conclusion"></a>结束语
通过更深入地了解这两种技术，你将能够就使用哪种队列技术以及何时使用做出更明智的决策。决定何时使用存储队列或服务总线队列时，显然要考虑很多因素。这些因素很大程度上取决于应用程序及其体系结构的独特需要。如果你的应用程序已使用  Azure 的核心功能，则可能更倾向于选择存储队列，尤其在服务之间需要基本通信和消息传送或需要大小可大于 80 GB 的队列时。

因为服务总线队列提供许多高级功能（如会话、事务、重复检测、自动死信和持久发布/订阅功能），所以，如果你正在构建混合应用程序或你的应用程序需要这些功能，这些队列将是首选。

## <a name="next-steps"></a>后续步骤
以下文章提供了有关使用存储队列或服务总线队列的更多指导和信息。

- [如何使用 Service Bus 队列](/documentation/articles/service-bus-dotnet-get-started-with-queues/)
- [如何使用队列存储服务](/documentation/articles/storage-dotnet-how-to-use-queues/)
- [使用服务总线中转消息传送改进性能的最佳实践](/documentation/articles/service-bus-performance-improvements/)
- [Azure 服务总线中的队列和主题简介](http://www.code-magazine.com/article.aspx?quickid=1112041)
- [服务总线开发人员指南](http://www.cloudcasts.net/devguide/Default.aspx?id=11030)
- [在 Azure 中使用队列服务](http://www.developerfusion.com/article/120197/using-the-queuing-service-in-windows-azure/)

[Azure 经典管理门户]: http://manage.windowsazure.cn
 

<!---HONumber=Mooncake_Quality_Review_1230_2016-->