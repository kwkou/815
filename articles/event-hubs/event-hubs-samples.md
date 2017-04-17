<properties
    pageTitle="Azure 事件中心示例 | Azure"
    description="事件中心示例"
    services="event-hubs"
    documentationcenter="na"
    author="jtaubensee"
    manager="timlt"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid=""
    ms.service="event-hubs"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="03/07/2017"
    wacn.date="04/17/2017"
    ms.author="jotaub;sethm"
    ms.sourcegitcommit="7cc8d7b9c616d399509cd9dbdd155b0e9a7987a8"
    ms.openlocfilehash="28b600cc68f3bc3fe5bc7545d0a9427e4b2770ce"
    ms.lasthandoff="04/07/2017" />

# <a name="event-hubs-samples"></a>事件中心示例 

事件中心示例演示了 [Azure 事件中心](/documentation/services/event-hubs/)中的主要功能。 本文对可用示例进行了分类和介绍，每个示例均具有链接。

撰写本文时，事件中心示例位于多个不同位置：

- [MSDN 开发人员代码示例](https://code.msdn.microsoft.com/site/search?query=event%20hubs&f%5B0%5D.Value=event%20hubs&f%5B0%5D.Type=SearchText&ac=5)
- [GitHub](https://github.com/Azure/azure-event-hubs/tree/master/samples)
<!-- [GitHub](https://github.com/Azure/azure-event-hubs/tree/master/samples) is correct-->

若要深入了解 .NET Framework 的不同版本，请参阅[框架和目标](http://docs.microsoft.com/zh-cn/dotnet/articles/standard/frameworks)。

后续将添加更多示例，因此请时常回访本网站获取更新。

## <a name="net-standard"></a>.NET Standard

下面的示例演示如何使用 [.NET Standard 库](http://docs.microsoft.com/zh-cn/dotnet/articles/standard/library)的[事件中心客户端](https://github.com/Azure/azure-event-hubs/blob/master/readme.md)发送和接收事件。
<!--[事件中心客户端](https://github.com/Azure/azure-event-hubs/blob/master/readme.md) is correct-->

### <a name="send-events"></a>发送事件 

[发送入门](https://github.com/Azure/azure-event-hubs/tree/master/samples/SampleSender)示例演示如何编写可将事件发送到事件中心的 .NET Core 控制台应用程序。

### <a name="receive-events"></a>接收事件 

[使用事件处理程序主机接收入门](https://github.com/Azure/azure-event-hubs/tree/master/samples/SampleEphReceiver)示例是一个使用 `Event Processor Host` 从事件中心接收消息的 .NET Core 控制台应用程序。

## <a name="net-framework"></a>.NET framework    

这些示例针对 [.NET Framework 库](https://msdn.microsoft.com/zh-cn/library/w0x726c2.aspx)，演示 Azure 事件中心的各种其他功能。

### <a name="notify-users-of-events-received"></a>通知用户已收到事件

[AppToNotifyUsers](https://github.com/Azure-Samples/event-hubs-dotnet-user-notifications) 示例通知用户已收到传感器或其他系统发出的数据。

### <a name="get-started-with-event-hubs"></a>事件中心入门 

[事件中心入门](https://code.msdn.microsoft.com/Service-Bus-Event-Hub-286fd097)示例演示事件中心的基本功能，如如何创建事件中心、将事件发送到事件中心，以及使用[事件处理器主机](https://www.nuget.org/packages/Microsoft.Azure.ServiceBus.EventProcessorHost/)处理事件。

### <a name="scale-out-event-processing"></a>扩大事件处理 

[扩大事件处理](https://code.msdn.microsoft.com/Service-Bus-Event-Hub-45f43fc3)示例演示如何使用[事件处理器主机](https://www.nuget.org/packages/Microsoft.Azure.ServiceBus.EventProcessorHost/)分配事件中心流消耗工作负荷。 它演示如何实现用于管理事件流的 **EventProcessor** 和 **EventProcessorFactory** 对象。 

###  <a name="pull-data-from-sql-into-an-event-hub"></a>将数据从 SQL 提取到事件中心

[提取 SQL 数据](https://github.com/Azure-Samples/event-hubs-dotnet-import-from-sql) 示例演示如何从 SQL 表中提取数据并将其推送到事件中心，以便用作下游分析应用程序中的输入。

### <a name="pull-web-data-into-an-event-hub"></a>将 Web 数据提取到事件中心 

[从 Web 导入数据](https://github.com/Azure-Samples/event-hubs-dotnet-importfromweb)示例演示如何从公共源（例如运输部的流量信息源）提取数据，并将其推送到事件中心。

## <a name="next-steps"></a>后续步骤

访问以下链接，了解有关 .NET Framework 版本的详细信息：

- [框架和目标](https://docs.microsoft.com/zh-cn/dotnet/articles/standard/frameworks)
- [.NET Framework 4.6 和 4.5](https://msdn.microsoft.com/zh-cn/library/w0x726c2.aspx)

可在以下文章中了解有关事件中心的详细信息：

- [事件中心概述](/documentation/articles/event-hubs-what-is-event-hubs/)
- [创建事件中心](/documentation/articles/event-hubs-create/)
- [事件中心常见问题](/documentation/articles/event-hubs-faq/)


<!--Update_Description:update meta prperties;wording update-->