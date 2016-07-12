<properties
	pageTitle="通过 C 和 Apache Storm 使用事件中心入门 | Azure"
	description="遵循本教程开始使用 Azure 事件中心，以通过 C 发送事件，并在 Apache Storm 群集中接收这些事件。"
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

有关详细信息，请参阅[事件中心概述]。

在本教程中，你将学习如何使用用 C 编写的控制台应用程序将消息引入事件中心，以及如何使用 Apache Storm 并行检索这些消息。

为了完成本教程，你将需要以下内容：

+ C 语言开发环境。对于本教程，我们假设 gcc 堆栈位于使用 Ubuntu 14.04 的 [Azure Linux VM](/documentation/articles/virtual-machines-linux-quick-create-portal/) 上。有关其他环境的说明，将在外部链接中提供。

+ 一个 Java 开发环境，配置为运行 [Maven](http://maven.apache.org/)。对于本教程，我们将采用 [Eclipse](https://www.eclipse.org/)。

+ 有效的 Azure 帐户。如果你没有帐户，只需花费几分钟就能创建一个免费试用帐户。有关详细信息，请参阅 [Azure 试用](/zh-cn/pricing/1rmb-trial/)。

[AZURE.INCLUDE [event-hubs-create-event-hub](../includes/event-hubs-create-event-hub.md)]

[AZURE.INCLUDE [service-bus-event-hubs-get-started-send-c](../includes/service-bus-event-hubs-get-started-send-c.md)]

[AZURE.INCLUDE [service-bus-event-hubs-get-started-receive-storm](../includes/service-bus-event-hubs-get-started-receive-storm.md)]

## 运行应用程序

现在，你已准备就绪，可以运行应用程序了。

1.	从 Eclipse 中运行 **LogTopology** 类，然后等待它为所有分区启动接收方。

2.	运行 **sender** 程序，然后就会看到事件出现在接收方窗口中。

	![][23]

> [AZURE.NOTE] 在本教程中，只出于开发目的以本地模式使用 Storm。请参阅 [HDInsight Storm 概述]和官方 [Apache Storm] 文档，以了解 Storm 开发和模式的详细信息。

## 后续步骤

以下资源可用于开发集成事件中心和 Storm 的应用程序。

- [用 Storm 和 HDInsight 分析传感器数据]是一个完整方案教程，它用到了事件中心、Storm 和 HBase，以在 Hadoop 群集中引入传感器数据。
- [使用 SCP.NET 和 C# 在 Storm 和 HDInsight 上开发流式数据处理应用程序]是有关使用 C# 编写 Storm 管道的教程。

<!-- Images. -->
[23]: ./media/event-hubs-c-storm-getstarted/receive-storm3.png

<!-- Links -->
[Azure 管理门户]: https://manage.windowsazure.cn/
[Event Processor Host]: https://www.nuget.org/packages/Microsoft.Azure.ServiceBus.EventProcessorHost
[事件中心概述]: /documentation/articles/event-hubs-overview/

[Apache Storm]: https://storm.incubator.apache.org
[HDInsight Storm 概述]: /documentation/articles/hdinsight-storm-overview/
[用 Storm 和 HDInsight 分析传感器数据]: /documentation/articles/hdinsight-storm-sensor-data-analysis/
[使用 SCP.NET 和 C# 在 Storm 和 HDInsight 上开发流式数据处理应用程序]: /documentation/articles/hdinsight-storm-develop-csharp-visual-studio-topology/
 

<!---HONumber=Mooncake_0613_2016-->