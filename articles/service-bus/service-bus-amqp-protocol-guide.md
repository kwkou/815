<properties 
    pageTitle="Azure 服务总线和事件中心内的 AMQP 1.0 协议指南 | Azure" 
    description="Azure 服务总线和事件中心内 AMQP 1.0 协议的表达与描述指南" 
    services="service-bus,event-hubs" 
    documentationCenter=".net" 
    authors="clemensv" 
    manager="timlt" 
    editor=""/>

<tags
    ms.service="service-bus"
    ms.date="07/01/2016"
    wacn.date="08/15/2016"/>

# Azure 服务总线和事件中心内的 AMQP 1.0 协议指南

高级消息队列协议 1.0 是一种标准化组帧和传输协议，能够以异步、安全且可靠的方式在两方之间传输消息。它是 Azure 服务总线消息传送和 Azure 事件中心的主要协议。这两种服务还支持 HTTPS。同时支持的专用 SBMP 协议即将淘汰并由 AMQP 取代。

AMQP 1.0 是中间件供应商（例如 Microsoft 和 Red Hat）与许多消息传送中间件用户（例如代表金融服务行业的 JP Morgan Chase）广泛合作的成果。OASIS 是 AMQP 协议和扩展规范的技术标准化论坛，它已获 ISO/IEC 19494 国际标准的官方批准。

## 目标

本文简要说明 AMQP 1.0 消息传送规范的核心概念以及当前正由 OASIS AMQP 技术委员定案的一小部分草稿扩展规范，并说明 Azure 服务总线如何根据这些规范进行实现和构建。

目的是要让在任何平台上使用任何现有 AMQP 1.0 客户端堆栈的开发人员通过 AMQP 1.0 与 Azure 服务总线交互。

常见的通用型 AMQP 1.0 堆栈（例如 Apache Proton 或 AMQP.NET Lite）已实现所有核心 AMQP 1.0 手势。这些基本手势有时以更高级别的 API 包装；Apache Proton 甚至提供两个，命令式 Messenger API 和反应式 Reactor API。

在以下介绍中，我们假设 AMQP 连接、会话和链接的管理以及帧传输和流量控制的处理都是由单个堆栈（例如 Apache Proton-C）处理，而不需要应用程序开发人员特别注意。我们以抽象方式假设存在一些 API 基元，例如连接能力，以及创建某种形式的 *sender* 和 *receiver* 抽象对象的能力，然后分别有某种形式的 `send()` 和 `receive()` 操作。

介绍 Azure 服务总线的高级功能（例如消息浏览或会话管理）时，将以 AMQP 术语说明这些功能，但也可作为以此假设 API 抽象概念基于的分层虚拟实现。

## AMQP 是什么？

AMQP 是一种组帧和传输协议。组帧表示它为以网络连接的任一方向流入的二进制数据流提供结构。该结构针对要在已连接方之间交换的不同数据块（帧）提供略图。传输功能确保通信双方都可以对于何时应发送帧以及传输何时应视为完成建立共识。

与 AMQP 工作组生成且前面过期的草稿版本（仍有一些消息代理在使用）不同，任务组的最终标准化 AMQP 1.0 协议并未规定要存在消息代理或消息中转站内实体的任何特定拓扑。

该协议可用于对称的对等通信，以便与支持队列和发布/订阅实体的消息代理交互，如 Azure 服务总线所为。它还可以用于与消息传送基础结构交互，其交互模式不同于一般队列（和 Azure 事件中心的情况一样）。当事件发送到事件中心时，事件中心的作用如同队列，但从中读取事件时，其作用更类似于串行存储服务；它有点类似于磁带机。客户端选择可用数据流的偏移，然后获取从该偏移位置转到最新可用的所有事件。

AMQP 1.0 协议被设计为可扩展，允许进一步规范以增强其功能。我们将在本文档中介绍的三个扩展规范说明这点。在通过可能难以设置本机 AMQP TCP 端口的现有 HTTPS/WebSockets 基础结构进行的通信中，绑定规范定义如何通过 WebSockets 将 AMQP 分层。以请求/响应方式与消息传送基础结构交互，以便进行管理或提供高级功能时，AMQP 管理规范定义所需的基本交互基元。在联合授权模型集成中，AMQP 基于声明的安全规范定义如何关联和续订与链接关联的授权令牌。

## 基本 AMQP 方案

本部分说明 AMQP 1.0 与 Azure 服务总线的基本使用方式，其中包括创建连接、会话和链接，以及与服务总线实体（例如队列、主题和订阅）相互传输消息。

了解 AMQP 工作原理的最权威来源是 AMQP 1.0 规范，但此规范是为了精确引导实现而编写，而非用于传授协议知识。本部分着重于尽可能介绍描述服务总线如何使用 AMQP 1.0 的术语。有关 AMQP 的更完整介绍，以及 AMQP 1.0 的更广泛介绍，可以查看[此视频课程][]。

### 连接和会话

![][1]

AMQP 将通信程序称为容器；其中包含节点，即这些容器内的通信实体。队列可以是此类节点。AMQP 允许多路复用，因此单个连接可用于节点之间的许多通信路径；例如，应用程序客户端可以同时从一个队列接收，并通过相同的网络连接发送到另一个队列。

网络连接因此固定在容器上。它由采用客户端角色的容器启动，对采用接收者角色的容器进行出站 TCP 套接字连接，以侦听和接受入站 TCP 连接。连接握手包括协商协议版本，声明或协商传输级别安全性 (TLS/SSL) 的使用，以及基于 SASL 的连接范围的身份验证/授权握手。

Azure 服务总线随时都需要使用 TLS。它支持通过 TCP 端口 5671 的连接，因此 TCP 连接在进入 AMQP 协议握手前先与 TLS 重叠；它还支持通过 TCP 端口 5672 的连接，因此服务器使用 AMQP 规定的模型，立即提供强制的 TLS 连接升级。AMQP WebSockets 绑定创建基于 TCP 端口 443 的隧道，其相当于 AMQP 5671 连接。

在设置连接和 TLS 之后，服务总线提供两个 SASL 机制选项：

-   SASL PLAIN 常用于将用户名和密码凭据发送到服务器。服务总线没有帐户，但是有命名的[共享访问安全规则](service-bus-shared-access-signature-authentication.md)，这些规则可授予权限并与某个密钥关联。规则名称可作为用户名，而密钥（如 base64 编码文本）可作为密码。与所选规则关联的权限控管允许在连接上进行的操作。

-   当客户端想要使用稍后所述的基于声明的安全 (CBS) 模型时，SASL ANONYMOUS 可用于绕过 SASL 授权。使用此选项，即可以匿名方式创建短期的客户端连接，在此连接期间，客户端只能与 CBS 终结点交互，并且 CBS 握手必须完成。

创建传输连接之后，容器各自声明它们愿意处理的帧大小上限；在空闲超时后，如果连接上没有任何活动，它们将单方面断开连接。

它们也声明支持的并发通道数量。通道是基于连接的单向出站虚拟传输路径。会话从每个互连的容器获取通道，以形成双向通信路径。

会话具有基于时段的流量控制模型；创建会话时，每一方声明它愿意在接收时段内接受的帧数。当各方交换帧时，已传输的帧将填满该时段，传输在时段已满时停止，直到该时段使用流程行为原语进行重置或扩展为止（行为原语是 AMQP 术语，表示在双方之间交换的协议级别手势）。

这种基于时段的模型大致类似于 TCP 基于时段的流量控制概念，但属于套接字内的会话级别。协议具有允许多个并发会话的概念，因此高优先级的流量可能冲过限制的正常流量，就像高速公路上的快速车道一样。

Azure 服务总线目前只对每个连接使用一个会话。服务总线标准版和事件中心的服务总线帧大小上限为 262,144 字节 (256 KB)。服务总线高级版则为 1,048,576 (1 MB)。服务总线不强加任何特定会话级别限制时段，但是在链接级别流量控制中定期重置时段（请参阅[下一部分](#links)）。

连接、通道和会话是暂时性的。如果基础连接失效，则必须重新创建连接、TLS 隧道、SASL 授权上下文和会话。

### 链接

![][2]

AMQP 通过链接传输消息。链接是在能以单个方向传输消息的会话中创建的通信路径；传输状态协商通过链接在已连接方之间双向进行。

任一容器可以在现有的会话中随时创建链接，这使 AMQP 不同于其他许多协议（包括 HTTP 和 MQTT），其中启动传输和传输路径是创建套接字连接之一方的独占权限。

启动链接的容器请求相对的容器接受链接，并且选择发送者或接收者的角色。因此，任一容器均可启动创建单向或双向通信路径，而后者建模为成对链接。

链接具有名称并与节点关联。如一开始所述，节点是容器内的通信实体。

在 Azure 服务总线中，节点直接等同于队列、主题、订阅或队列或订阅的死信子队列。AMQP 中使用的节点名称因此是服务总线命名空间内实体的相对名称。如果队列名为 **myqueue**，这也是它的 AMQP 节点名称。主题订阅遵循 HTTP API 约定归类为“订阅”资源集合，因此订阅 **sub** 或主题 **mytopic** 具有 AMQP 节点名称 **mytopic/subscriptions/sub**。

正在连接的客户端也必须使用本地节点名称来创建链接；服务总线不规范这些节点名称，并且不进行解释。AMQP 1.0 客户端堆栈通常使用方案，以确保这些暂时节点名称是客户端范围内的唯一名称。

### 传输

![][3]

建立链接后，即可通过该链接传输消息。在 AMQP 中，使用明确的协议手势运行传输（传输行为原语），以通过链接将消息从发送者转到接收者。传输在“安置好”时完成，这意味着双方已建立该传输结果的共识。

在最简单的情况下，发送者可以选择发送“预先安置”的消息，这意味着客户端对结果不感兴趣，并且接收者不提供任何有关操作结果的反馈。此模式由 Azure 服务总线在 AMQP 协议级别支持，但不显示在任何客户端 API 中。

一般情况是发送未安置好的消息，然后接收者使用处置行为原语表示接受或拒绝。拒绝发生于接收者因为任何原因而无法接受消息时，而拒绝消息包含原因相关信息，这是 AMQP 所定义的错误结构。如果消息因为 Azure 服务总线内部的错误而被拒绝，则服务返回该结构内的额外信息，而如果发出支持请求，该信息即可用于提供诊断提示给支持人员。稍后你将了解有关错误的详细信息。

已解除状态是一种特殊形式的拒绝，表示接收者对传输没有任何技术异议，但是对于安置传输也不感兴趣。该情况的确存在，例如，当消息传递到服务总线客户端，而客户端因为无法运行处理消息所生成的任务（尽管消息传递本身并未出错）而选择“放弃”消息时。该状态的变化是已修改状态，该状态允许在消息解除后进行更改。服务总线目前不使用该状态。

AMQP 1.0 规范定义进一步的处置状态已接收，其对于处理链接恢复特别有用。当以前的连接和会话丢失时，链接恢复允许重组链接的状态，以及新连接和会话的任何搁置传递。

Azure 服务总线不支持链接恢复；如果客户端失去对服务总线的连接，并且未安置的消息传输已搁置，该消息传输便丢失，而客户端必须重新连接、重建链接，以及重试传输。

同样地，Azure 服务总线和事件中心都支持“至少一次”传输，其中的发送者可确保消息已存储并接受，但是不支持在 AMQP 级别的“刚好一次”传输，其中的系统尝试恢复链接并继续协商传递状态，以避免重复传输消息。

为了弥补可能的重复发送，Azure 服务总线支持重复检测队列和主题（可选功能）。重复检测在用户定义的时段内记录所有传入消息的消息 ID，并以无消息方式放弃在该相同时段内使用相同消息 ID 发送的所有消息。

### 流量控制

![][4]

除了以前介绍过的会话级别流量控制模型以外，每个链接都有自己的流量控制模型。会话级别流量控制可防止容器必须一次处理太多帧，链接级别流量控制让应用程序负责控制它想要从链接处理的消息数目以及时机。

在链接上，传输只发生于发送者有足够的“链接信用额度”时。链接信用额度是接收者使用流程行为原语所设置的计数器，其范围是链接。将链接信用额度分配给发送者时，将通过传递消息来尝试用完该信用额度。每个消息传递使剩余的链接信用额度减一。当链接信用额度用完时，便会停止传递。

当服务总线采用接收者角色时，则立即提供给发送者充足的链接信用额度，以便立即发送消息。使用链接信用额度时，服务总线偶尔发送流程行为原语给发送者，以更新链接信用额度余额。

在发送者角色中，服务总线立即发送消息，以用完任何未偿付的链接信用额度。

在 API 级别的“接收”调用转译成由客户端发送到服务总线的流程行为原语，而服务总线使用队列中第一个可用、已解除锁定的消息，进行锁定并传输，以使用该信用额度。如果没有可立即传递的消息，由任何链接使用该特定实体创建的任何未偿付信用额度仍以抵达顺序记录，而消息遭到锁定并且在能够使用任何未偿付信用额度时传输。

当传输进入其中一种终端状态已接受、已拒绝或已解除时，消息锁定就会解除。终端的状态为已接受时，将从服务总线中删除消息。它保留在服务总线中，并将在传输达到任何其他状态时传递给下一个接收者。服务总线在因为重复拒绝或解除而达到实体所允许的最大传递计数时，自动将消息转到实体死信队列中。

即使是正式的服务总线 API 现今也不直接公开这种选项，较低级别的 AMQP 协议客户端可以使用链接信用额度模型，通过核发非常大量的链接信用额度，将针对每个接收请求核发一单位信用额度的“提取式”模型变成“推送式”模型，然后接收可用的消息，而不需要任何进一步的交互。推送通过 [MessagingFactory.PrefetchCount](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.messagingfactory.prefetchcount.aspx) 或 [MessageReceiver.PrefetchCount](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.messagereceiver.prefetchcount.aspx) 属性设置来支持。如果两者均不为零，则 AMQP 客户端使用它作为链接信用额度。

在此内容中，务必了解实体内消息锁定的过期时钟在从实体获取消息时启动，而不是在消息放在网络上时启动。每当客户端通过颁发链接信用额度来表示接收消息的整备性，因此预期主动提取网络上的消息并准备好处理它们。否则消息锁定可能在消息传递之前过期。使用链接信用流量控制应直接反映出可立即准备处理分派给接收者的可用消息。

以下部分汇总了在不同 API 交互期间行为原语流程的图解概述。每部分说明不同的逻辑操作。其中一些交互可能很“缓慢”，这意味着它们可能只在有需要时运行。直到发送或请求第一个消息，创建消息发送者才造成网络交互。

箭头显示行为原语流程方向。

#### 创建消息接收者

| 客户端 | 服务总线 |
|---------------------------------------------------------------------------------------------------------------------------------------------------	|--------------------------------------------------------------------------------------------------------------------------------------------	|
| --> attach(<br/>name={link name},<br/>handle={numeric handle},<br/>role=**receiver**,<br/>source={entity name},<br/>target={client link id}<br/>) | 客户端作为接收者附加到实体 |
| 附加到链接末尾的服务总线回复 | <-- attach(<br/>name={link name},<br/>handle={numeric handle},<br/>role=**sender**,<br/>source={entity name},<br/>target={client link id}<br/>) |

#### 创建消息发送者

| 客户端 | 服务总线 |
|------------------------------------------------------------------------------------------------------------------	|--------------------------------------------------------------------------------------------------------------------	|
| --> attach(<br/>name={link name},<br/>handle={numeric handle},<br/>role=**sender**,<br/>source={client link id},<br/>target={entity name}<br/>) | 无操作 |
| 无操作 | <-- attach(<br/>name={link name},<br/>handle={numeric handle},<br/>role=**receiver**,<br/>source={client link id},<br/>target={entity name}<br/>) |

#### 创建消息发送者（错误）

| 客户端 | 服务总线 |
|------------------------------------------------------------------------------------------------------------------	|---------------------------------------------------------------------	|
| --> attach(<br/>name={link name},<br/>handle={numeric handle},<br/>role=**sender**,<br/>source={client link id},<br/>target={entity name}<br/>) | 无操作 |
| 无操作 | <-- attach(<br/>name={link name},<br/>handle={numeric handle},<br/>role=**receiver**,<br/>source=null,<br/>target=null<br/>)<br/><br/><-- detach(<br/>handle={numeric handle},<br/>closed=**true**,<br/>error={error info}<br/>) |

#### 关闭消息接收者/发送者

| 客户端 | 服务总线 |
|-------------------------------------------------	|-------------------------------------------------	|
| --> detach(<br/>handle={numeric handle},<br/>closed=**true**<br/>) | 无操作 |
| 无操作 | <-- detach(<br/>handle={numeric handle},<br/>closed=**true**<br/>) |

#### 发送（成功）

| 客户端 | 服务总线 |
|------------------------------------------------------------------------------------------------------------------------------	|------------------------------------------------------------------------------------------------------	|
| --> transfer(<br/>delivery-id={numeric handle},<br/>delivery-tag={binary handle},<br/>settled=**false**,,more=**false**,<br/>state=**null**,<br/>resume=**false**<br/>) | 无操作 |
| 无操作 | <-- disposition(<br/>role=receiver,<br/>first={delivery id},<br/>last={delivery id},<br/>settled=**true**,<br/>state=**accepted**<br/>) |

#### 发送（错误）

| 客户端 | 服务总线 |
|------------------------------------------------------------------------------------------------------------------------------	|-----------------------------------------------------------------------------------------------------------------------------	|
| --> transfer(<br/>delivery-id={numeric handle},<br/>delivery-tag={binary handle},<br/>settled=**false**,,more=**false**,<br/>state=**null**,<br/>resume=**false**<br/>) | 无操作 |
| 无操作 | <-- disposition(<br/>role=receiver,<br/>first={delivery id},<br/>last={delivery id},<br/>settled=**true**,<br/>state=**rejected**(<br/>error={error info}<br/>)<br/>) |

#### 接收

| 客户端 | 服务总线 |
|------------------------------------------------------------------------------------------------------	|------------------------------------------------------------------------------------------------------------------------------	|
| --> flow(<br/>link-credit=1<br/>) | 无操作 |
| 无操作 | < transfer(<br/>delivery-id={numeric handle},<br/>delivery-tag={binary handle},<br/>settled=**false**,<br/>more=**false**,<br/>state=**null**,<br/>resume=**false**<br/>) |
| --> disposition(<br/>role=**receiver**,<br/>first={delivery id},<br/>last={delivery id},<br/>settled=**true**,<br/>state=**accepted**<br/>) | 无操作 |

#### 多消息接收

| 客户端 | 服务总线 |
|--------------------------------------------------------------------------------------------------------	|--------------------------------------------------------------------------------------------------------------------------------	|
| --> flow(<br/>link-credit=3<br/>) | 无操作 |
| 无操作 | < transfer(<br/>delivery-id={numeric handle},<br/>delivery-tag={binary handle},<br/>settled=**false**,<br/>more=**false**,<br/>state=**null**,<br/>resume=**false**<br/>) |
| 无操作 | < transfer(<br/>delivery-id={numeric handle+1},<br/>delivery-tag={binary handle},<br/>settled=**false**,<br/>more=**false**,<br/>state=**null**,<br/>resume=**false**<br/>) |
| 无操作 | < transfer(<br/>delivery-id={numeric handle+2},<br/>delivery-tag={binary handle},<br/>settled=**false**,<br/>more=**false**,<br/>state=**null**,<br/>resume=**false**<br/>) |
| --> disposition(<br/>role=receiver,<br/>first={delivery id},<br/>last={delivery id+2},<br/>settled=**true**,<br/>state=**accepted**<br/>) | 无操作 |

### 消息

以下部分说明服务总线使用标准 AMQP 消息部分中的哪些属性，以及它们如何映射到正式的服务总线 API。

#### 标头的值开始缓存响应

| 字段名称 | 使用情况 | API 名称 |
|----------------	|-------------------------------	|---------------	|
| durable        	| -                             	| -             	|
| priority       	| -                             	| -             	|
| ttl            	| 此消息的生存时间 			| [TimeToLive](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.brokeredmessage.timetolive.aspx)    	|
| first-acquirer 	| -                             	| -             	|
| delivery-count 	| -                             	| [DeliveryCount](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.brokeredmessage.deliverycount.aspx) 	|

#### 属性

| 字段名称 | 使用情况 | API 名称 |
|----------------------	|---------------------------------------------------------------------------------------------------------------------------------	|--------------------------------------------	|
| message-id | 应用程序为此消息定义的自由格式标识符。用于重复检测。 | [MessageId](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.brokeredmessage.messageid.aspx) |
| user-id | 应用程序定义的用户标识符，服务总线无法进行解释。 | 无法通过服务总线 API 访问。 |
| to | 应用程序定义的目标标识符，服务总线无法进行解释。 | [如果](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.brokeredmessage.to.aspx) |
| subject | 应用程序定义的消息用途标识符，服务总线无法进行解释。 | [标签](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.brokeredmessage.label.aspx) |
| reply-to | 应用程序定义的回复路径指示符，服务总线无法进行解释。 | [ReplyTo](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.brokeredmessage.replyto.aspx) |
| correlation-id | 应用程序定义的相关性标识符，服务总线无法进行解释。 | [CorrelationId](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.brokeredmessage.correlationid.aspx) |
| content-type | 应用程序定义的内容类型指示符，服务总线无法进行解释。 | [ContentType](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.brokeredmessage.contenttype.aspx) |
| content-encoding | 应用程序定义的内容编码指示符，服务总线无法进行解释。 | 无法通过服务总线 API 访问。 |
| absolute-expiry-time | 声明消息过期的绝对时刻。在输入时忽略（观察到标头 ttl），在输出时授权具权威性。 | [ExpiresAtUtc](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.brokeredmessage.expiresatutc.aspx) |
| creation-time | 声明消息的创建时间。不由服务总线使用 | 无法通过服务总线 API 访问。 |
| group-id | 应用程序为相关的消息集定义的标识符。用于服务总线会话。 | [SessionId](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.brokeredmessage.sessionid.aspx) |
| group-sequence | 用于标识消息在会话内的相对序列号的计数器。服务总线会将其忽略。 | 无法通过服务总线 API 访问。 |
| reply-to-group-id | - | [ReplyToSessionId](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.brokeredmessage.replytosessionid.aspx) |

## 高级服务总线功能

本部分介绍 Azure 服务总线的高级功能，这些功能基于 AMQP 的 OASIS 技术委员会目前正在开发的 AMQP 草稿扩展。Azure 服务总线实现这些草稿的最新状态，并且采用这些草稿达到标准状态时所引进的更改。

> [AZURE.NOTE] 服务总线消息高级操作通过请求/响应模式受到支持。文档 [AMQP 1.0 in Service Bus: request/response-based operations](https://msdn.microsoft.com/zh-cn/library/azure/mt727956.aspx)（服务总线中的 AMQP 1.0：基于请求/响应的操作）中介绍了这些操作的详细信息。

### AMQP 管理

AMQP 管理规范是我们将在此介绍的第一个草稿扩展。此规范定义一组基于 AMQP 协议的协议手势，以便通过 AMQP 进行消息基础结构的管理交互。此规范定义泛型操作（例如创建、读取、更新和删除），以便管理消息传送基础结构内的实体和一组查询操作。

上述所有手势都需要客户端与消息传送基础结构之间的请求/响应交互，因此此规范定义如何制作 AMQP 上交互模式的模型：客户端连接到消息传送基础结构、启动会话，然后创建一组链接。在某一个链接上，客户端扮演发送者，而在其他链接上扮演接收者，因此创建一组可作为双向通道的链接。

| 逻辑运算 | 客户端 | 服务总线 |
|------------------------------|-----------------------------|-----------------------------|
| 创建请求响应路径 | --> attach(<br/>name={*link name*},<br/>handle={*numeric handle*},<br/>role=**sender**,<br/>source=**null**,<br/>target=”myentity/$management”<br/>) |无操作 |
|创建请求响应路径 |无操作 | <-- attach(<br/>name={*link name*},<br/>handle={*numeric handle*},<br/>role=**receiver**,<br/>source=null,<br/>target=”myentity”<br/>) |
|创建请求响应路径 | --> attach(<br/>name={*link name*},<br/>handle={*numeric handle*},<br/>role=**receiver**,<br/>source=”myentity/$management”,<br/>target=”myclient$id”<br/>) | |无操作
|创建请求响应路径 |无操作 | <-- attach(<br/>name={*link name*},<br/>handle={*numeric handle*},<br/>role=**sender**,<br/>source=”myentity”,<br/>target=”myclient$id”<br/>) |

准备好该组链接，请求/响应实现就相当简单：请求是发送到消息传送基础结构内了解此模式之实体的消息。在该请求消息中，*properties* 节中的 *reply-to* 字段设置为响应传递到之链接的 *target* 标识符。处理实体将处理此请求，然后通过 *target* 标识符匹配所示的 *reply-to* 标识符的链接传递回复。

显然，该模式要求回复目标的客户端容器和客户端生成的标识符在所有客户端中是唯一的，并且出于安全原因，还要难以预测。

用于管理协议和所有其他使用相同模式的协议的消息交换发生于应用程序级别；它们不定义新的 AMQP 协议级别手势。这是刻意设计的，以便应用程序立即使用与 AMQP 1.0 堆栈兼容的这些扩展。

Azure 服务总线目前不实现管理规范的任何核心功能，但是对声明型安全功能以及我们将在以下部分中介绍的几乎所有高级功能而言，管理规范所定义的请求/响应模式是基本的。

### 基于声明的授权

AMQP 基于声明的授权 (CBS) 规范草案基于管理规范的请求/响应模式，主要说明如何配合使用联合安全令牌与 AMQP 的广义模型。

简介中所述的 AMQP 默认安全模型基于 SASL，并与 AMQP 连接握手集成。使用 SASL 的好处是它提供已定义一组机制的可扩展模型，任何正式依赖 SASL 的协议均可受益。在这些机制之中，“PLAIN”用于传输用户名和密码，“EXTERNAL”用于绑定到 TLS 级别安全，“ANONYMOUS”用于表示缺少显式身份验证/授权，还有其他各种机制能够用于发送身份验证和/或授权凭据或令牌。

AMQP 的 SASL 集成有两个缺点：

-   所有凭据与令牌的范围都只限于连接。消息传送基础结构可能想根据每个实体提供不同的访问控制。例如，允许令牌的持有者发送到队列 A，而不是到队列 B。使用固定在连接上的授权上下文，就不可能使用单个连接并且对队列 A 和 B 使用不同的访问令牌。

-   访问令牌的有效时间通常有限。这可强制用户定期重新获取令牌，而如果用户的访问权限已更改，令牌的颁发者便有机会拒绝颁发刷新令牌。AMQP 连接可能持续很长时间。SASL 模型只提供一个机在连接时设置令牌，这意味着消息传送基础结构必须在令牌过期时断开客户端的连接，或者必须接受允许与访问权限可能已在其间吊销的客户端持续通信的风险。

Azure 服务总线实现的 AMQP CBS 规范可让这两个问题获得圆满的解决：它可让客户端创建访问令牌与每个节点的关联，以及在这些令牌过期前进行更新，而无需中断消息流。

CBS 定义由消息传送基础结构所提供的虚拟管理节点（名为* $cbs*）。管理节点可代表消息传送基础结构中的任何其他节点接受令牌。

协议手势是管理规范定义的请求/回复交换。这意味着客户端使用 *$cbs* 节点创建一组链接，在输出链接上传递请求，然后在输入链接上等待响应。

请求消息具有以下应用程序属性：

| 键 | 可选 | 值类型 | 值内容 |
|------------|----------|------------|--------------------------------------------|
| operation | 否 | 字符串 | **put-token** |
| type | 否 | 字符串 | 正在放置的令牌类型。 |
| 名称 | 否 | 字符串 | 令牌应用到的“受众”。 |
| expiration | 是 | timestamp | 令牌过期时间。 |

*name* 属性标识应与令牌关联的实体。在服务总线中，这是队列或主题/订阅的路径。*type* 属性标识令牌类型：

| 令牌类型 | 令牌说明 | 正文类型 | 说明 |
|---------------------------------|------------------------|---------------------|----------------------------------------------------------|
| amqp:jwt | JSON Web 令牌 (JWT) | AMQP 值（字符串） | 尚不可用。 |
| amqp:swt | 简单 Web 令牌 (SWT) | AMQP 值（字符串） | 仅支持 AAD/ACS 颁发的 SWT 令牌 |
| servicebus.windows.net:sastoken | 服务总线 SAS 令牌 | AMQP 值（字符串）| - |

令牌赋予权限。服务总线识别三个基本权限：“发送”允许发送、“侦听”允许接收，“管理”允许操作实体。AAD/ACS 颁发的 SWT 令牌明确将这些权限包含为声明。服务总线 SAS 令牌引用在命名空间或实体上配置的规则，这些规则是使用权限配置的。使用与该规则关联的密钥来签名令牌，以此方式让令牌表达各自的权限。与使用 *put-token* 的实体关联的令牌将允许已连接的客户端根据每个令牌权限来与实体交互。客户端承担 *sender* 角色的链接需要“发送”权限，而承担 *receiver* 角色的链接则需要“侦听”权限。

回复消息具有以下 *application-properties* 值

| 键 | 可选 | 值类型 | 值内容 |
|--------------------|----------|------------|-----------------------------------|
| status-code | 否 | int | HTTP 响应代码 **[RFC2616]**。 |
| status-description | 是 | 字符串 | 状态的说明。 |

客户端可以针对消息传送基础结构中的任何实体重复调用 *put-token*。令牌的范围是当前客户端且定位点为当前连接，这意味着服务器在删除连接时将删除所有保留的令牌。

目前的服务总线实现只允许 CBS 配合SASL 方法“ANONYMOUS”。在 SASL 握手之前始终必须存在 SSL/TLS 连接。

因此所选的 AMQP 1.0 客户端必须支持 ANONYMOUS 机制。匿名访问表示发生初始连接握手（包括创建初始会话），而服务总线不知道谁正在创建此连接。

创建连接和会话后，将链接附加到 *$cbs* 节点和发送 *put-token* 请求是唯一允许的操作。必须在创建连接后的 20 秒内使用对某个实体节点的 *put-token* 请求成功创建有效的令牌，否则服务总线将单方面断开连接。

客户端后续负责跟踪令牌过期时间。令牌过期时，服务总线将立即删除相应实体连接上的所有链接。若要避免这种情况，客户端随时可以通过具有相同 *put-token* 手势的虚拟 *$cbs* 管理节点，使用新的令牌来替换节点的令牌，且不干扰在不同链接上流动的有效负载流量。

## 后续步骤

若要了解有关 AMQP 的详细信息，请参阅以下链接：

- [服务总线 AMQP 概述]
- [针对服务总线分区队列和主题的 AMQP 1.0 支持]
- [适用于 Windows Server 的服务总线中的 AMQP]

[此视频课程]: https://www.youtube.com/playlist?list=PLmE4bZU0qx-wAP02i0I7PJWvDWoCytEjD
[1]: ./media/service-bus-amqp/amqp1.png
[2]: ./media/service-bus-amqp/amqp2.png
[3]: ./media/service-bus-amqp/amqp3.png
[4]: ./media/service-bus-amqp/amqp4.png

[服务总线 AMQP 概述]: /documentation/articles/service-bus-amqp-overview/
[针对服务总线分区队列和主题的 AMQP 1.0 支持]: /documentation/articles/service-bus-partitioned-queues-and-topics-amqp-overview/
[适用于 Windows Server 的服务总线中的 AMQP]: https://msdn.microsoft.com/zh-cn/library/dn574799.aspx

<!---HONumber=Mooncake_0808_2016-->