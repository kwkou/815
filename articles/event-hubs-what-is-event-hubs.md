<properties
	pageTitle="什么是 Azure 事件中心？| Azure"
	description="Azure 事件中心概述"
	services="event-hubs"
	documentationCenter=".net"
	authors="nberdy"
	manager="timlt"
	editor=""/>

<tags
	ms.service="event-hubs"
	ms.date="04/12/2016"
	wacn.date="05/23/2016"/>

# 什么是 Azure 事件中心？

事件中心是一个高度可缩放的引入服务，每秒可以引入数百万的事件，从而使你能够处理和分析连接设备和应用程序生成的海量数据。事件中心充当事件管道“前门”，将数据收集到事件中心后，可以使用任何实时分析提供程序或批处理/存储适配器来转换和存储这些数据。事件中心可将事件流的生成与这些事件的使用分离开来，因此，事件使用者可以根据自己的计划访问事件。

## 事件中心的功能

事件中心是一种事件处理服务，用于向云提供大规模的事件与遥测数据入口，并且具有较低的延迟和较高的可靠性。此服务特别适合用于：

* 应用程序检测
* 用户体验或工作流处理
* 物联网 (IoT) 方案

事件中心的其他一些主要功能包括移动应用中的行为跟踪、从 Web 场中采集流量信息、控制台游戏中的游戏内事件捕获，或者从工业机器或互联汽车中收集遥测数据。

与[服务总线队列和主题](/documentation/articles/service-bus-messaging-overview/)不同，事件中心重点提供大规模的消息流处理。事件中心功能与主题的不同之处在于，它明显偏向于高吞吐量和事件处理方案。因此，事件中心未实现[主题](/documentation/articles/service-bus-fundamentals-hybrid-solutions/#topics)提供的某些消息传递功能。如果你需要这些功能，主题仍是最佳的选择。

## 后续步骤

有关事件中心的详细信息，请参阅以下主题。

- [事件中心概述](/documentation/articles/event-hubs-overview/)
- [事件中心编程指南](/documentation/articles/event-hubs-programming-guide/)
- [事件中心可用性和支持常见问题](/documentation/articles/event-hubs-availability-and-support-faq/)
- 使用[事件中心教程]入门
- [使用事件中心的完整示例应用程序]

[事件中心教程]: /documentation/articles/hdinsight-apache-storm-tutorial-get-started/
[使用事件中心的完整示例应用程序]: https://github.com/Azure-Samples/
<!---HONumber=Mooncake_0321_2016-->