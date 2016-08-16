<properties
	pageTitle="通过 C# 使用事件中心入门 | Azure"
	description="遵循本教程开始使用 Azure 事件中心，以通过 C# 发送事件，并使用 EventProcessorHost 通过 Java 接收事件。"
	services="event-hubs"
	documentationCenter=""
	authors="fsautomata"
	manager="timlt"
	editor=""/>

<tags
	ms.service="event-hubs"
	ms.date="06/16/2016"
	wacn.date="07/11/2016"/>

# 事件中心入门

[AZURE.INCLUDE [service-bus-selector-get-started](../../includes/service-bus-selector-get-started.md)]

## 介绍

事件中心是一个服务，可用于处理来自连接设备和应用程序的大量事件数据（遥测）。将数据采集到事件中心后，可以使用任何实时分析提供程序或存储群集来转换和存储数据。这种大规模事件收集和处理功能是现代应用程序体系结构（包括物联网 (IoT)）的重要组件。

本教程说明如何使用 Azure 经典管理门户创建事件中心。此外，还将说明如何使用以 C# 编写的控制台应用程序将消息收集到事件中心，以及如何使用 Java 事件处理程序主机库并行检索这些消息。

为了完成本教程，你需要有：

+ [Microsoft Visual Studio](http://visualstudio.com)

+ 有效的 Azure 帐户。<br/>如果你没有帐户，只需几分钟的时间就能创建一个帐户。有关详细信息，请参阅 [Azure 试用版](/pricing/1rmb-trial)。

[AZURE.INCLUDE [event-hubs-create-event-hub](../../includes/event-hubs-create-event-hub.md)]

[AZURE.INCLUDE [service-bus-event-hubs-get-started-send-csharp](../../includes/service-bus-event-hubs-get-started-send-csharp.md)]

[AZURE.INCLUDE [service-bus-event-hubs-get-started-receive-ephjava](../../includes/service-bus-event-hubs-get-started-receive-ephjava.md)]

## 运行应用程序

现在，你已准备就绪，可以运行应用程序了。

1.	运行 **Receiver** 项目。

	![][21]

2.	运行 **Sender** 项目。

	![][22]

## 后续步骤

现在，你已生成了一个可以创建事件中心以及发送和接收数据的有效应用程序，接下来请继续学习以下方案：

- [使用事件中心的完整示例应用程序][]。
- [使用事件中心扩大事件处理][]示例。
- 使用服务总线队列的[队列消息解决方案][]。
- [事件中心概述][]

<!-- Images. -->
[21]: ./media/event-hubs-csharp-ephjava-getstarted/ephjava.png
[22]: ./media/event-hubs-csharp-ephjava-getstarted/cs-send.png

<!-- Links -->
[Azure classic portal]: https://manage.windowsazure.cn/
[事件中心概述]: /documentation/articles/event-hubs-overview
[使用事件中心的完整示例应用程序]: https://code.msdn.microsoft.com/Service-Bus-Event-Hub-286fd097
[使用事件中心扩大事件处理]: https://code.msdn.microsoft.com/Service-Bus-Event-Hub-45f43fc3
[队列消息解决方案]: /documentation/articles/service-bus-dotnet-multi-tier-app-using-service-bus-queues
 

<!---HONumber=Mooncake_0704_2016-->