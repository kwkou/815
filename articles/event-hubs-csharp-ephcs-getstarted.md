<properties
	pageTitle="通过 C# 使用事件中心入门 | Azure"
	description="遵循本教程开始使用以 C# 编写的 Azure 事件中心和 EventProcessorHost。"
	services="event-hubs"
	documentationCenter=""
	authors="fsautomata"
	manager="timlt"
	editor=""/>

<tags
	ms.service="event-hubs"
	ms.date="05/13/2016"
	wacn.date="06/06/2016"/>

# 事件中心入门

[AZURE.INCLUDE [service-bus-selector-get-started](../includes/service-bus-selector-get-started.md)]

## 介绍

事件中心是一个服务，可用于处理来自连接设备和应用程序的大量事件数据（遥测）。将数据采集到事件中心后，可以使用任何实时分析提供程序或存储群集来转换和存储数据。这种大规模事件收集和处理功能是现代应用程序体系结构（包括物联网 (IoT)）的重要组件。

本教程说明如何使用 Azure 管理门户创建事件中心。此外，还将说明如何使用以 C# 编写的控制台应用程序将消息收集到事件中心，以及如何使用 C# [事件处理程序主机][]库并行检索这些消息。

为了完成本教程，你需要有：

+ Microsoft Visual Studio 2013，或 Microsoft Visual Studio 2015。

+ 有效的 Azure 帐户。<br/>如果你没有帐户，只需花费几分钟就能创建一个试用帐户。有关详细信息，请参阅 <a href="/pricing/1rmb-trial/" target="_blank">Azure 试用</a>。


[AZURE.INCLUDE [event-hubs-create-event-hub](../includes/event-hubs-create-event-hub.md)]

[AZURE.INCLUDE [service-bus-event-hubs-get-started-send-csharp](../includes/service-bus-event-hubs-get-started-send-csharp.md)]


[AZURE.INCLUDE [service-bus-event-hubs-get-started-receive-ephcs](../includes/service-bus-event-hubs-get-started-receive-ephcs.md)]

## 运行应用程序

现在，你已准备就绪，可以运行应用程序了。

1. 从 Visual Studio 中，打开先前创建的 **Receiver** 项目。

2. 右键单击“Receiver”解决方案，单击“添加”，然后单击“现有项目”。
 
3. 找到现有的 Sender.csproj 文件，然后双击该文件以将其添加到解决方案中。
 
4. 再次右键单击“Receiver”解决方案，然后单击“属性”。随即显示“Receiver”属性页面。

5. 单击“启动项目”，然后单击“多个启动项目”按钮。将 **Receiver** 和 **Sender** 项目的“操作”框设置为“启动”。

	![][19]

6. 单击“项目依赖项”。在“项目”框中，单击“Sender”。在“依赖于”框中，确保选中“Receiver”。

	![][20]

7. 单击“确定”以消除“属性”对话框。

1.	从 Visual Studio 中，按 F5 运行 **Receiver** 项目，然后等待它为所有分区启动接收方。

	![][21]

2.	**Sender** 项目将自动运行。在控制台窗口中按 **Enter**，便会看到事件出现在接收方窗口中。

	![][22]

在“Sender”窗口中按 **Ctrl+C** 以结束 Sender 应用程序，然后在“Receiver”窗口中按 **Enter** 以关闭该应用程序。

## 后续步骤

现在，你已生成了一个可以创建事件中心以及发送和接收数据的有效应用程序，接下来请继续学习以下方案：

- [使用事件中心的完整示例应用程序][]。
- [使用事件中心扩大事件处理][]示例。
- 使用服务总线队列的[队列消息解决方案][]。
- [事件中心概述][]

<!-- Images. -->
[19]: ./media/event-hubs-csharp-ephcs-getstarted/create-eh-proj1.png
[20]: ./media/event-hubs-csharp-ephcs-getstarted/create-eh-proj2.png
[21]: ./media/event-hubs-csharp-ephcs-getstarted/run-csharp-ephcs1.png
[22]: ./media/event-hubs-csharp-ephcs-getstarted/run-csharp-ephcs2.png

<!-- Links -->
[Azure 管理门户]: https://manage.windowsazure.cn/
[事件处理程序主机]: https://www.nuget.org/packages/Microsoft.Azure.ServiceBus.EventProcessorHost
[事件中心概述]: /documentation/articles/event-hubs-overview/
[使用事件中心的完整示例应用程序]: https://code.msdn.microsoft.com/windowsazure/Service-Bus-Event-Hub-286fd097
[使用事件中心扩大事件处理]: https://code.msdn.microsoft.com/windowsazure/Service-Bus-Event-Hub-45f43fc3
[队列消息解决方案]: /documentation/articles/service-bus-dotnet-multi-tier-app-using-service-bus-queues/
 

<!---HONumber=Mooncake_0530_2016-->