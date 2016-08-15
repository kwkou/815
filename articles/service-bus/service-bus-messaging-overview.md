<properties
	pageTitle="服务总线消息传送概述 | Microsoft Azure"
	description="服务总线消息传送：在云中灵活传送数据"
	services="service-bus"
	documentationCenter=".net"
	authors="sethmanheim"
	manager="timlt"
	editor=""/>

<tags
	ms.service="service-bus"
	ms.date="03/09/2016"
	wacn.date="03/28/2016"/>


# 服务总线消息传送：在云中灵活传送数据

Azure 服务总线消息传送是一项可靠的信息传送服务。此服务的目的是简化通信。当两个或更多个对象要交换信息时，他们就需要一个通信机制。服务总线消息传送是一种中转或第三方通信机制。这类似于现实世界中的邮递服务。邮递服务可让你十分方便地在世界各地寄送不同种类的信件与包裹，并提供各种递送保证。

服务总线消息传送类似于寄送信件的邮递服务，可在发送方与接收方之间灵活传送信息。即使双方永远不会同时联机，或双方无法在完全相同的时间接收消息，消息服务也可以确保能够传送信息。如此看来，消息传送就类似于寄送信件，而非中转通信则类似于拨打电话（在呼叫等待与来电显示功能之前的旧式拨打电话方式比较类似于中转消息传送）。

消息发送方也可能需要各种传送特征，包括事务、重复数据检测、基于时间的过期，以及批处理。这些模式也具有类似于邮递服务的特征：重复传送、需要签名、地址更改或撤回。

服务总线支持两种不同的消息传递模式：*中继*消息传送和*中转*消息传送。

## 中继消息传送

服务总线的[中继](/documentation/articles/service-bus-relay-overview/)组件是一种集中式（但负载高度平衡的）服务，支持各种不同的传输协议和 Web 服务标准。包括 SOAP、WS-* 甚至 REST。[中继服务](/documentation/articles/service-bus-dotnet-how-to-use-relay/)提供了各种不同的中继连接选项，并且可以在有可能时帮助协商直接对等连接。服务总线针对使用 Windows Communication Foundation (WCF) 的 .NET 开发人员在性能与可用性方面进行了优化，并且通过 SOAP 和 REST 接口提供到中继服务的完全访问。这样就可以将任何 SOAP 或 REST 编程环境与服务总线进行集成。

中继服务支持传统的单向消息传送、请求/响应消息传送和对等消息传送。它还支持 Internet 范围的事件分发，以实现发布 - 订阅方案和双向套接字通信，从而提高点到点通信效率。在中继消息模式中，本地服务会通过出站端口连接至中继服务，并创建双向套接字用于绑定至特定集合地址的通信。客户端然后可以通过将消息发送到抵达会合地址的中继服务来与内部服务通信。中继服务接着通过已部署的双向套接字将消息“中继”到本地服务。客户端不需要与本地服务建立直接连接，也不需要了解服务所在的位置，并且本地服务无需在防火墙上打开任何入站端口。

你必须使用一套 WCF“中继”绑定在本地服务与中继服务之间发起连接。在幕后，中继绑定将映射到传输绑定元素，这些元素旨在创建与云中服务总线集成的 WCF 通道组件。

中继消息传送提供许多好处，但是为了发送和接收消息，需要服务器和客户端同一时间同时处于联机状态。这对于 HTTP 方式的通信并非最佳，在这种方式下，请求通常可能无法长期存在，对仅偶尔连接的客户端（例如浏览器和移动应用程序等）也同样并非最佳。中转消息传送支持分离的通信，且具有自身的优势；客户端和服务器可以在有需要时连接并以异步方式执行其操作。

## <a name="Brokered-messaging"></a>中转消息传送

对照中继消息传送方案，可认为[中转消息传送](/documentation/articles/service-bus-fundamentals-hybrid-solutions/)是异步的，或者“暂时分离的”。 消息生产者（发送者）和使用者（接收者）不必同时处于联机状态。消息传送基础结构将消息可靠地存储在“中转站”中（例如一个队列），直到使用方准备好接收它们。这将允许分布式应用程序的组件断开连接，例如，为进行维护而自动断开，或因组件故障断开连接，而不会影响整个系统。此外，接收应用程序可能仅需在一天中某个时间处于联机状态，例如库存管理系统只需在工作日结束时运行。

服务总线中转消息传送基础结构的核心组件是队列、主题和订阅。两者的主要差别在于，主题支持可用于基于内容的复杂路由和传送逻辑的发布/订阅功能，包括发送到多个接收方。这些组件启用新的异步消息传送方案，例如暂时分离、发布/订阅和负载平衡。有关消息传送实体的详细信息，请参阅[服务总线队列、主题和订阅](/documentation/articles/service-bus-queues-topics-subscriptions/)。

与中继消息传送基础结构一样，中转消息传送功能是为 WCF 和 .NET Framework 程序员提供的，并且也是通过 REST 提供。

## 后续步骤

若要了解有关服务总线消息传送的详细信息，请参阅以下主题。

- [服务总线队列、主题和订阅](/documentation/articles/service-bus-queues-topics-subscriptions/)
- [服务总线基础知识](/documentation/articles/service-bus-fundamentals-hybrid-solutions/)
- [服务总线体系结构](/documentation/articles/service-bus-architecture/)
- [如何使用服务总线队列](/documentation/articles/service-bus-dotnet-how-to-use-queues/)

- [如何使用服务总线主题](/documentation/articles/service-bus-dotnet-how-to-use-topics-subscriptions/)

<!---HONumber=Mooncake_0104_2016-->