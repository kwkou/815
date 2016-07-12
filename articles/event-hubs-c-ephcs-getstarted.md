<properties
	pageTitle="通过 C 和 C# 使用事件中心入门 | Azure"
	description="遵循本教程开始使用 Azure 事件中心，以通过 C 发送事件，并使用 EventProcessorHost 通过 C# 接收事件。"
	services="event-hubs"
	documentationCenter=""
	authors="fsautomata"
	manager="timlt"
	editor=""/>

<tags
	ms.service="event-hubs"
	ms.date="03/08/2016"
	wacn.date="04/11/2016"/>

# 事件中心入门

[AZURE.INCLUDE [service-bus-selector-get-started](../includes/service-bus-selector-get-started.md)]

## 介绍

事件中心是一个具备高度可伸缩性的引入系统，每秒可收入大量事件，从而使应用程序能够处理和分析你连接的设备和应用程序所产生的海量数据。将数据采集到事件中心后，你可以使用任何实时分析提供程序或存储群集来转换和存储数据。

有关详细信息，请参阅[事件中心概述]。

在本教程中，你将学习如何使用用 C 编写的控制台应用程序将消息引入事件中心，以及如何使用 C# [事件处理程序主机]库并行检索这些消息。

为了完成本教程，你将需要以下内容：

+ C 语言开发环境。对于本教程，我们假设 gcc 堆栈位于使用 Ubuntu 14.04 的 [Azure Linux VM](/documentation/articles/virtual-machines-linux-quick-create-portal/) 上。有关其他环境的说明，将在外部链接中提供。

+ Microsoft Visual Studio Express 2013 for Windows

+ 有效的 Azure 帐户。<br/>如果你没有帐户，只需花费几分钟就能创建一个免费试用帐户。有关详细信息，请参阅 <a href="/pricing/1rmb-trial" target="_blank">Azure 试用</a>。

## 创建事件中心

1. 登录到 [Azure 管理门户]，然后单击屏幕底部的“新建”。

2. 依次单击“App Service”、“服务总线”、“事件中心”、“快速创建”。

   	![][1]

3. 为你的事件中心键入名称，选择所需区域，然后单击“创建新事件中心”。

   	![][2]

4. 单击你刚创建的命名空间（通常为 ***事件中心名称*-ns**）。

   	![][3]

5. 单击页面顶部的“事件中心”选项卡，然后单击你刚创建的事件中心。

   	![][4]

6. 单击页面顶部的“配置”选项卡，添加具有“发送”权限的名为“SendRule”的规则，添加另一具有“管理、发送、侦听”权限的名为“ReceiveRule”的规则，然后单击“保存”。

   	![][5]

7. 在同一页上，记下为 **SendRule** 生成的密钥。

   	![][6b]

8. 单击页面顶部的“仪表板”选项卡，然后单击“连接信息”。记下两个连接字符串。

   	![][6]

现在，你的事件中心就创建好了，你已经有了收发事件所需的连接字符串。

[AZURE.INCLUDE [service-bus-event-hubs-get-started-send-c](../includes/service-bus-event-hubs-get-started-send-c.md)]


[AZURE.INCLUDE [service-bus-event-hubs-get-started-receive-ephcs](../includes/service-bus-event-hubs-get-started-receive-ephcs.md)]

## 运行应用程序

现在，你已准备就绪，可以运行应用程序了。

1.	从 Visual Studio 中运行 **Receiver** 项目，然后等待它为所有分区启动接收方。

   	![][21]

2.	运行 **sender** 程序，然后就会看到事件出现在接收方窗口中。

   	![][24]

## 后续步骤

现在，你已生成了一个可以创建事件中心以及发送和接收数据的有效应用程序，接下来请继续学习以下方案：

- [使用事件中心的完整示例应用程序]。
- [使用事件中心扩大事件处理]示例。
<!--- 使用服务总线队列的[队列消息解决方案]。-->
- [事件中心概述]

<!-- Images. -->
[1]: ./media/event-hubs-c-ephcs-getstarted/create-event-hub1.png
[2]: ./media/event-hubs-c-ephcs-getstarted/create-event-hub2.png
[3]: ./media/event-hubs-c-ephcs-getstarted/create-event-hub3.png
[4]: ./media/event-hubs-c-ephcs-getstarted/create-event-hub4.png
[5]: ./media/event-hubs-c-ephcs-getstarted/create-event-hub5.png
[6]: ./media/event-hubs-c-ephcs-getstarted/create-event-hub6.png
[6b]: ./media/event-hubs-c-ephcs-getstarted/create-event-hub6b.png


[21]: ./media/event-hubs-c-ephcs-getstarted/run-csharp-ephcs1.png
[24]: ./media/event-hubs-c-ephcs-getstarted/receive-eph-c.png

<!-- Links -->
[Azure 管理门户]: https://manage.windowsazure.cn/
[事件处理程序主机]: https://www.nuget.org/packages/Microsoft.Azure.ServiceBus.EventProcessorHost
[事件中心概述]: /documentation/articles/event-hubs-overview/
[使用事件中心的完整示例应用程序]: https://code.msdn.microsoft.com/windowsazure/Service-Bus-Event-Hub-286fd097
[使用事件中心扩大事件处理]: https://code.msdn.microsoft.com/windowsazure/Service-Bus-Event-Hub-45f43fc3
[队列消息解决方案]: /documentation/articles/service-bus-dotnet-multi-tier-app-using-service-bus-queues/

<!---HONumber=Mooncake_1207_2015-->