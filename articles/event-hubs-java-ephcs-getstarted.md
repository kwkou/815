<properties
	pageTitle="通过 Java 使用事件中心入门 | Azure"
	description="遵循本教程开始使用 Azure 事件中心，以通过 Java 发送事件，并使用 EventProcessorHost 通过 C# 接收事件。"
	services="event-hubs"
	documentationCenter=""
	authors="fsautomata"
	manager="timlt"
	editor=""/>

<tags
	ms.service="event-hubs"
	ms.date="05/13/2016"
	wacn.date="06/20/2016"/>

# 事件中心入门

[AZURE.INCLUDE [service-bus-selector-get-started](../includes/service-bus-selector-get-started.md)]

## 介绍

事件中心是一个具备高度可伸缩性的引入系统，每秒可收入大量事件，从而使应用程序能够处理和分析你连接的设备和应用程序所产生的海量数据。将数据采集到事件中心后，你可以使用任何实时分析提供程序或存储群集来转换和存储数据。

有关详细信息，请参阅[事件中心概述][]。

本教程介绍如何使用以 Java 编写的控制台应用程序将消息引入事件中心，以及如何使用 C# [事件处理程序主机][]库并行检索这些消息。

若要完成本教程，你需要以下各项：

+ Java 开发环境。对于本教程，我们将采用 Eclipse。

+ Microsoft Visual Studio Express for Windows

+ 有效的 Azure 帐户。<br/>如果你没有帐户，只需花费几分钟就能创建一个免费试用帐户。有关详细信息，请参阅 <a href="/zh-cn/pricing/1rmb-trial/" target="_blank">Azure 试用</a>。

[AZURE.INCLUDE [event-hubs-create-event-hub](../includes/event-hubs-create-event-hub.md)]


[AZURE.INCLUDE [service-bus-event-hubs-get-started-send-java](../includes/service-bus-event-hubs-get-started-send-java.md)]


[AZURE.INCLUDE [service-bus-event-hubs-get-started-receive-ephcs](../includes/service-bus-event-hubs-get-started-receive-ephcs.md)]

## 运行应用程序

现在，你已准备就绪，可以运行应用程序了。

1.	从 Visual Studio 中运行 **Receiver** 项目，然后等待它为所有分区启动接收方。

	![][21]

2.	运行 **Sender** 项目，在控制台窗口中按 **Enter**，然后会看到事件出现在接收方窗口中。

	![][22]

## 后续步骤

现在，你已生成了一个可以创建事件中心以及发送和接收数据的有效应用程序，接下来请继续学习以下方案：

- [使用事件中心的完整示例应用程序][]。
- [使用事件中心扩大事件处理][]示例。
- 使用服务总线队列的[队列消息解决方案][]。

有关详细信息，请参阅 [Java 开发人员中心](/develop/java/)。

<!-- Images. -->
[21]: ./media/event-hubs-java-ephcs-getstarted/run-csharp-ephcs1.png
[22]: ./media/event-hubs-java-ephcs-getstarted/run-csharp-ephcs2.png

<!-- Links -->
[Azure 管理门户]: https://manage.windowsazure.cn/
[事件处理程序主机]: https://www.nuget.org/packages/Microsoft.Azure.ServiceBus.EventProcessorHost
[事件中心概述]: /documentation/articles/event-hubs-overview/
[使用事件中心的完整示例应用程序]: https://code.msdn.microsoft.com/Service-Bus-Event-Hub-286fd097
[使用事件中心扩大事件处理]: https://code.msdn.microsoft.com/Service-Bus-Event-Hub-45f43fc3
[队列消息解决方案]: /documentation/articles/service-bus-dotnet-multi-tier-app-using-service-bus-queues/
 

<!---HONumber=Mooncake_0613_2016-->