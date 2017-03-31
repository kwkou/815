<properties
    pageTitle="Azure 事件中心示例 | Azure"
    description="事件中心示例"
    services="event-hubs"
    documentationcenter="na"
    author="jtaubensee"
    manager="timlt"
    editor="" />
<tags
    ms.assetid=""
    ms.service="event-hubs"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="02/01/2017"
    wacn.date="03/24/2017"
    ms.author="jotaub;sethm" />  


# 事件中心示例 

事件中心示例演示 [Azure 事件中心](/documentation/services/event-hubs/)中的主要功能。本文对可用示例进行了分类和介绍，每个示例均具有链接。

撰写本文时，事件中心示例位于多个不同位置：

- [MSDN 开发人员代码示例](https://code.msdn.microsoft.com/site/search?query=event%20hubs&f%5B0%5D.Value=event%20hubs&f%5B0%5D.Type=SearchText&ac=5)
- [GitHub](https://github.com/Azure/azure-event-hubs/tree/master/samples)

若要深入了解 .NET Framework 的不同版本，请参阅[框架和目标](http://docs.microsoft.com/zh-cn/dotnet/articles/standard/frameworks)。

后续将添加更多示例，因此请时常回访本网站获取更新。

## .NET Standard

以下示例演示如何使用 [.NET Standard 库](http://docs.microsoft.com/zh-cn/dotnet/articles/standard/library)的[事件中心客户端](https://github.com/Azure/azure-event-hubs/blob/master/readme.md)发送和接收事件。

### 发送事件 

[开始发送](https://github.com/Azure/azure-event-hubs/tree/master/samples/SampleSender)示例演示如何编写将事件发送到事件中心的 .NET Core 控制台应用程序。

### 接收事件 

[使用事件处理程序主机开始接收](https://github.com/Azure/azure-event-hubs/tree/master/samples/SampleEphReceiver)示例是 .NET Core 控制台应用程序，它使用[事件处理程序主机](https://www.nuget.org/packages/Microsoft.Azure.ServiceBus.EventProcessorHost/)从事件中心接收消息。

## .NET framework	

这些示例针对 [.NET Framework 库](https://msdn.microsoft.com/zh-cn/library/w0x726c2.aspx)，演示 Azure 事件中心的各种其他功能。
 
### 通知用户已收到事件

[AppToNotifyUsers](https://github.com/Azure-Samples/event-hubs-dotnet-user-notifications) 示例通知用户已收到传感器或其他系统发出的数据。

### 事件中心入门 

[事件中心入门](https://code.msdn.microsoft.com/Service-Bus-Event-Hub-286fd097)示例演示事件中心的基本功能，如创建事件中心、将事件发送到事件中心，以及使用[事件处理程序主机](https://www.nuget.org/packages/Microsoft.Azure.ServiceBus.EventProcessorHost/)处理事件的方法。

### 扩大事件处理 

[扩大事件处理](https://code.msdn.microsoft.com/Service-Bus-Event-Hub-45f43fc3)示例演示如何使用[事件处理程序主机](https://www.nuget.org/packages/Microsoft.Azure.ServiceBus.EventProcessorHost/)分发事件中心流消耗的工作负荷。它介绍如何实现 **EventProcessor** 和 **EventProcessorFactory** 对象来管理事件流。

###  将数据从 SQL 提取到事件中心

[提取 SQL 数据](https://github.com/Azure-Samples/event-hubs-dotnet-import-from-sql)示例演示如何从 SQL 表中提取数据并将其推送到事件中心，以便用作下游分析应用程序中的输入。

### 将 Web 数据推送到事件中心 

[从 Web 导入数据](https://github.com/Azure-Samples/event-hubs-dotnet-importfromweb)示例演示如何从公共源（如运输部的交通信息源）中提取数据并将其推送到事件中心。

## 后续步骤

访问以下链接，了解有关 .NET Framework 版本的详细信息：

- [框架和目标](https://docs.microsoft.com/zh-cn/dotnet/articles/standard/frameworks)
- [.NET Framework 4.6 和 4.5](https://msdn.microsoft.com/zh-cn/library/w0x726c2.aspx)

可在以下文章中了解有关事件中心的详细信息：

- [事件中心概述](/documentation/articles/event-hubs-what-is-event-hubs/)
- [创建事件中心](/documentation/articles/event-hubs-create/)
- [事件中心常见问题](/documentation/articles/event-hubs-faq/)

<!---HONumber=Mooncake_0320_2017-->
<!--Update_Description:new article about samples of event hubs-->