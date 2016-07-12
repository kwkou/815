<properties 
   pageTitle="通知用户已收到传感器或其他系统发出的数据 | Azure"
   description="介绍如何使用事件中心来通知用户已收到传感器数据。"
   services="event-hubs"
   documentationCenter="na"
   authors="spyrossak"
   manager="timlt"
   editor="" />
<tags 
   ms.service="event-hubs"
   ms.date="03/08/2016"
   wacn.date="04/11/2016" />

# 通知用户已收到传感器或其他系统发出的数据

假设你的某个应用程序会实时监视数据或按计划生成报告。查看网站上显示的实时图表或报告时，你可能会发现有些情况需要你采取措施。如果有某种机制会提醒你注意这些情况，而不是要记得时常查看该网站，那么会带来哪种优势呢？ 假设你你在温室中安装了一盏灯，你希望灯光熄灭时，立即知道这件事。实现此目的的方法之一是在温室中安装一个灯光传感器，当灯光熄灭时，你会收到电子邮件。

![][1]

另举一个例子：假设你在运营一家宠物寄养店，当用品库存量较低时，你必须看到警报。例如，当狗粮库存量降到临界水平时，ERP 系统应该向你发送短信。

![][2]

问题是如何在达到特定条件时接收关键信息，而不是亲自去查看静态报告。如果你使用 [Azure 事件中心][]或 [Dynamics AX][] 等企业应用程序发出的数据，则你可以使用多种选项来处理这些信息。你可以在网站上查看这些信息，对其进行分析和存储，还可以使用它们来触发命令以执行某种操作。为此，你可以使用功能强大的工具，如 [Azure 网站][]、[SQL Azure][]、[HDInsight][]、[Cortana Analytics Suite][] 或 [Azure 通知中心][]。但有时，你要做的一切就是以极小的开销向某人发送数据。为了向你展示如何以少量的代码实现此目的，我们提供了一个新示例 [AppToNotifyUsers][]。包括的选项为电子邮件 (SMTP)、短信和电话。

## 应用程序结构

该应用程序是以 C# 编写的，示例中的自述文件包含修改、生成和发布应用程序所需的全部信息。以下部分提供了有关该应用程序的功能的全面概述。

首先假设你要将关键事件推送到 Azure 事件中心。事实上可以推送到任何中心，只要你有权访问它并知道连接字符串即可。

如果你没有事件中心，可以遵循 [Connect The Dots](https://github.com/Azure/connectthedots) 中的说明，使用 Arduino 盾板和 Raspberry Pi 轻松设置一个测试平台。Arduino 盾板上的灯光传感器通过 Pi 向 [Azure 事件中心][] (**ehdevices**) 发送光能级，如果收到的光能级低于特定的级别，则 [Azure 流分析](/services/stream-analytics/)作业将向另一个事件中心 (**ehalerts**) 推送警报。

**AppToNotify** 启动时，会读取配置文件 (App.config) 来获取接收警报的事件中心的 URL 和凭据。然后，它会生成一个进程来持续监视是否有任何消息传入该事件中心 - 只要你有权访问事件中心的 URL 并拥有有效的凭据，则此事件中心读取器代码就会持续读取传入的数据。在启动期间，应用程序还会读取你要使用的消息传送服务（电子邮件、短信或电话）的 URL 和凭据，以及发件人的名称/地址和收件人的列表。

一旦事件中心监视器检测到消息，就会触发一个进程，以使用配置文件中指定的方法发送该消息。请注意，该监视器会发送它检测到的每条消息。如果将监视器设置为指向每秒接收 10 条消息的事件中心，则发件人将每秒发送 10 条消息 - 即每秒 10 封电子邮件、每秒 10 条短信或每秒 10 次电话呼叫。因此，请确保监视只会接收需要发出的警报的事件中心，而不要监视从传感器或应用程序接收所有原始数据的事件中心。

## 适用性

本示例中的代码只演示了如何监视事件中心，以及在你想要将此功能添加到应用程序时，如何调用外部消息传送服务。请注意，此解决方案是以开发人员为中心的 DIY 示例。它不能解决企业需求，例如冗余、故障转移、故障时重新启动，等等。有关更全面的解决方案和生产解决方案，请参阅以下主题：

- 使用 [Azure 通知中心](https://msdn.microsoft.com/zh-cn/library/azure/jj927170.aspx)，如博客[使用 Azure 通知中心向数百万台移动设备广播推送通知](http://weblogs.asp.net/scottgu/broadcast-push-notifications-to-millions-of-mobile-devices-using-windows-azure-notification-hubs)中所述。 

## 后续步骤

可以很轻松地创建一个简单的通知服务用于向收件人发送电子邮件或短信或者拨打电话，以便中继事件中心或 IoT 中心收到的数据。若要部署解决方案以便基于这些中心收到的数据来通知用户，请访问 [AppToNotifyUsers][]。

有关这些中心的详细信息，请参阅以下文章：

- [Azure 事件中心]
- 使用[事件中心教程]入门。

若要部署解决方案以便基于这些中心收到的数据来通知用户，请访问：

- [AppToNotifyUsers][]

[事件中心教程]: /documentation/articles/event-hubs-csharp-ephcs-getstarted/
[Azure 事件中心]: /services/event-hubs/
[Azure 事件中心]: /services/event-hubs/
[使用事件中心的完整示例应用程序]: https://code.msdn.microsoft.com/windowsazure/Service-Bus-Event-Hub-286fd097
[队列消息解决方案]: /documentation/articles/service-bus-dotnet-multi-tier-app-using-service-bus-queues/
[AppToNotifyUsers]: https://github.com/Azure-Samples/event-hubs-dotnet-user-notifications
[Dynamics AX]: http://www.microsoft.com/zh-cn/dynamics/erp-ax-overview.aspx
[Azure 网站]: /services/web-sites/
[SQL Azure]: /services/sql-databases/
[HDInsight]: /services/hdinsight/
[Cortana Analytics Suite]: http://www.microsoft.com/server-cloud/cortana-analytics-suite/Overview.aspx?WT.srch=1&WT.mc_ID=SEM_lLFwOJm3&bknode=BlueKai
[Azure 通知中心]: /services/notification-hubs/
[Azure Stream Analytics]: /services/stream-analytics/
 
[1]: ./media/event-hubs-sensors-notify-users/event-hubs-sensor-alert.png
[2]: ./media/event-hubs-sensors-notify-users/event-hubs-erp-alert.png

<!---HONumber=Mooncake_0104_2016-->