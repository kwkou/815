<properties
	pageTitle="服务总线消息传送概述 - Azure"
	description="服务总线消息传送：在云中灵活传送数据"
	services="service-bus"
	documentationCenter=".net"
	authors="djrosanova"
	manager="timlt"
	editor="mattshel"/>

<tags ms.service="service-bus"

	ms.date="07/02/2015"
	wacn.date="10/03/2015"/>


# 服务总线消息传送：在云中灵活传送数据

Windows Azure 服务总线消息传送是一项可靠的信息传送服务。此服务的目的是简化通信。当两个或更多个对象要交换信息时，他们就需要一个通信机制。服务总线消息传送是一种中转或第三方通信机制。这类似于现实世界中的邮递服务。邮递服务可让你十分方便地在世界各地寄送不同种类的信件与包裹，并提供各种递送保证。

Azure 服务总线消息传送类似于寄送信件的邮递服务，可在发送方与接收方之间灵活传送信息。即使双方永远不会同时联机，或双方无法在完全相同的时刻接收消息，消息服务也可以确保能够传送信息。如此看来，消息传送就类似于寄送信件，而非中转通信则类似于拨打电话（在呼叫等待与来电显示功能之前的旧式拨打电话方式比较类似于中转消息传送）。

消息发送方也可能需要各种传送特征，包括事务、重复数据检测、基于时间的过期，以及批处理。这些模式也具有类似于邮递服务的特征：重复传送、需要签名、地址更改或撤回。

服务总线消息传送提供两种不同的功能：队列和主题。这两个消息传送实体都支持上述所有概念，此外还支持其他概念。两者的主要差别在于，主题支持可用于基于内容的复杂路由和传送逻辑的发布/订阅功能，包括发送到多个接收方。

## 后续步骤

若要了解有关服务总线消息传送的详细信息，请参阅以下主题。

- [Azure 服务总线体系结构概述](/documentation/articles/fundamentals-service-bus-hybrid-solutions)

- [如何使用服务总线队列](/documentation/articles/service-bus-dotnet-how-to-use-queues)

- [如何使用服务总线主题](/documentation/articles/service-bus-dotnet-how-to-use-topics-subscriptions)

<!---HONumber=71-->