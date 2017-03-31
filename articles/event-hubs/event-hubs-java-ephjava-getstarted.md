<properties
    pageTitle="通过 Java 使用事件中心入门 | Azure"
    description="遵循本教程开始使用 Azure 事件中心，以通过 Java 发送事件，并使用 EventProcessorHost 接收事件。"
    services="event-hubs"
    documentationcenter=""
    author="jtaubensee"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="38e3be53-251c-488f-a856-9a500f41b6ca"
    ms.service="event-hubs"
    ms.workload="core"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="01/04/2017"
    wacn.date="02/24/2017"
    ms.author="jotaub;sethm" />  


# 事件中心入门

[AZURE.INCLUDE [service-bus-selector-get-started](../../includes/service-bus-selector-get-started.md)]

## 介绍

事件中心是一个具备高度可伸缩性的引入系统，每秒可收入大量事件，从而使应用程序能够处理和分析连接的设备和应用程序所产生的海量数据。数据采集到事件中心后，可以使用任何实时分析提供程序或存储群集来转换和存储数据。

有关详细信息，请参阅[事件中心概述][Event Hubs overview]。

本教程介绍如何使用以 Java 编写的控制台应用程序将消息引入事件中心，以及如何使用 Java 事件处理程序主机库并行检索这些消息。

若要完成本教程，你需要以下各项：

* Java 开发环境。对于本教程，我们将采用 Eclipse。

* 有效的 Azure 帐户。<br/>如果你没有帐户，只需花费几分钟就能创建一个免费帐户。有关详细信息，请参阅 <a href="/pricing/1rmb-trial/" target="_blank">Azure 试用</a>。

[AZURE.INCLUDE [event-hubs-create-event-hub](../../includes/event-hubs-create-event-hub.md)]

[AZURE.INCLUDE [service-bus-event-hubs-get-started-send-java](../../includes/service-bus-event-hubs-get-started-send-java.md)]

[AZURE.INCLUDE [service-bus-event-hubs-get-started-receive-ephjava](../../includes/service-bus-event-hubs-get-started-receive-ephjava.md)]

## 运行应用程序

现在，你已准备就绪，可以运行应用程序了。

1. 运行 **Receiver** 项目。
   
    ![][21]
2. 运行 **Sender** 项目。
   
    ![][22]

## 后续步骤

现在，你已生成了一个可以创建事件中心以及发送和接收数据的有效应用程序，接下来请继续学习以下方案：

* [使用事件中心的完整示例应用程序][sample application that uses Event Hubs]。
* [使用事件中心扩大事件处理][Scale out Event Processing with Event Hubs]示例。

有关详细信息，请参阅 [Java 开发人员中心](/develop/java/)。

<!-- Images. -->

[21]: ./media/event-hubs-java-ephjava-getstarted/ephjava.png
[22]: ./media/event-hubs-java-ephjava-getstarted/java-send.png

<!-- Links -->

[Event Hubs overview]: /documentation/articles/event-hubs-what-is-event-hubs/
[sample application that uses Event Hubs]: https://code.msdn.microsoft.com/Service-Bus-Event-Hub-286fd097
[Scale out Event Processing with Event Hubs]: https://code.msdn.microsoft.com/Service-Bus-Event-Hub-45f43fc3
 

<!---HONumber=Mooncake_0220_2017-->
<!-- Update_Description: update meta properties; wording update; update link reference -->