<properties pageTitle="事件中心入门 metaKeywords="Azure Service Bus, Event Hub, getting started Event Hubs" description="Follow this tutorial to get started using Azure Event Hubs sending events with C# and receiving them in an Apache Storm cluster" metaCanonical="" services="" documentationCenter="" title="Get Started with Event Hubs" authors="elioda" solutions="" manager="timlt" editor="" />

<tags ms.service="service-bus" ms.workload="core" ms.tgt_pltfrm="csharp" ms.devlang="csharp" ms.topic="hero-article" ms.date="10/27/2014" ms.author="elioda" />

# <a name="getting-started"> </a>事件中心入门

[WACOM.INCLUDE [service-bus-selector-get-started](../includes/service-bus-selector-get-started.md)]

事件中心是一个具备高度可伸缩性的引入系统，每秒可收入大量事件，从而使应用程序能够处理和分析你连接的设备和应用程序所产生的海量数据。将数据采集到事件中心后，你可以使用任何实时分析提供程序或存储群集来转换和存储数据。

有关详细信息，请参阅[事件中心概述]。

在本教程中，你将学习如何使用用 C# 编写的控制台应用程序将消息引入事件中心，以及如何使用 Apache Storm 并行检索这些消息。

为了完成本教程，你将需要以下内容：

+ Microsoft Visual Studio Express 2013 for Windows

+ 一个 Java 开发环境，配置为运行 [Maven](http://maven.apache.org/)。对于本教程，我们将假定 [Eclipse](https://www.eclipse.org/)

+ 有效的 Azure 订阅。<br/>如果你没有帐户，则可以创建一个免费的试用帐户，只需几分钟即可完成。有关详细信息，请参阅 <a href="http://www.windowsazure.cn/pricing/1rmb-trial/" target="_blank">Azure 免费试用</a>。

## 创建事件中心

1. 登录到 [Azure 管理门户]，然后单击屏幕底部的**"新建"**。

2. 依次单击**"应用服务"**、**Service Bus**、**"事件中心"**、**"快速创建"**。

   	![][1]

3. 为你的事件中心键入名称，选择所需区域，然后单击**"创建新事件中心"**。

   	![][2]

4. 单击你刚创建的命名空间（通常为 ***事件中心名称*-ns**）。

   	![][3]

5. 单击页面顶部的**"事件中心"**选项卡，然后单击你刚创建的事件中心。

   	![][4]

6. 单击顶部的**"配置"**选项卡，添加具有"发送"权限的名为**"SendRule"**的规则，添加另一具有"侦听"权限的名为**"ReceiveRule"**的规则，然后单击**"保存"**。

   	![][5]

7. 在同一页上，记下为 **ReceiveRule** 生成的密钥。

   	![][6c]

8. 单击顶部的**"仪表板"**选项卡，然后单击**"连接信息"**。记下两个连接字符串。

   	![][6]

现在，你的事件中心就创建好了，你已经有了收发事件所需的连接字符串。

[WACOM.INCLUDE [service-bus-event-hubs-get-started-send-csharp](../includes/service-bus-event-hubs-get-started-send-csharp.md)]


[WACOM.INCLUDE [service-bus-event-hubs-get-started-receive-storm](../includes/service-bus-event-hubs-get-started-receive-storm.md)]

## 运行应用程序

现在，你已准备就绪，可以运行应用程序了。

1.	从 Eclipse 中运行 **LogTopology** 类，然后等待它为所有分区启动接收方。

2.	运行 **Sender** 项目，在控制台窗口中按 **Enter** 键，然后会看到事件出现在接收方窗口中。

   	![][22]

<!-- Images. -->
[1]: ./media/service-bus-event-hubs-getstarted/create-event-hub1.png
[2]: ./media/service-bus-event-hubs-getstarted/create-event-hub2.png
[3]: ./media/service-bus-event-hubs-getstarted/create-event-hub3.png
[4]: ./media/service-bus-event-hubs-getstarted/create-event-hub4.png
[5]: ./media/service-bus-event-hubs-getstarted/create-event-hub5.png
[6]: ./media/service-bus-event-hubs-getstarted/create-event-hub6.png
[6c]: ./media/service-bus-event-hubs-getstarted/create-event-hub6c.png

[22]: ./media/service-bus-event-hubs-getstarted/receive-storm1.png

<!-- Links -->
[Azure 管理门户]: https://manage.windowsazure.cn/
[事件处理程序主机]: https://www.nuget.org/packages/Microsoft.Azure.ServiceBus.EventProcessorHost
[事件中心概述]: http://msdn.microsoft.com/zh-cn/library/azure/dn836025.aspx

