<properties linkid="dev-net-Service-bus" urlDisplayName="Windows Azure 服务总线" pageTitle="Windows Azure 服务管理：服务总线" metaKeywords="服务总线" description="" metaCanonical="" services="服务总线" documentationCenter="Services" title="Learn about flexible messaging in the cloud" authors="" solutions="" manager="" editor="" />

#服务总线

####在云中进行灵活的消息传送
服务总线 是一个云消息平台。它位于您的应用程序组件之间（无论这些组件是在云中还是在本地），提供松散耦合消息交换以扩大规模并提高恢复能力。

####快速链接
-   [服务概述](/home/features/messaging/)
-   [定价详细信息](/pricing/details/service-bus/)
     
##教程和指南

###探究
####[功能概述](http://msdn.microsoft.com/zh-cn/library/ee732537.aspx)         
了解 服务总线 提供哪些功能，以及何时使用这些功能。

####[事件中心](http://msdn.microsoft.com/zh-cn/library/azure/dn789972.aspx)
超规模事件流采集，用于处理任何应用程序或设备工作负荷。

####[队列](/zh-cn/documentation/articles/service-bus-dotnet-how-to-use-queues/)
基于云的有保证的先入先出消息传送策略，支持一系列标准协议（REST、AMQP 和 WS*）以及用于将消息放入/拉出队列的 API。

####[主题](/zh-cn/documentation/articles/service-bus-dotnet-how-to-use-topics-subscriptions/)
支持扇出和扇入通信的发布/订阅消息服务。提供筛选器和高级路由功能。

####[中继](/zh-cn/documentation/articles/cloud-services-dotnet-hybrid-app-using-service-bus-relay/)
用于本地服务且安全、可靠的基于云的通信桥。
        
###构建

####[事件中心入门](/zh-cn/documentation/articles/service-bus-event-hubs-csharp-ephcs-getstarted/)
      
####[使用事件中心的高分辨率遥测摄取应用程序](http://code.msdn.microsoft.com/windowsazure/Service-Bus-Event-Hub-286fd097)
此示例演示事件中心的基本功能，如创建事件中心、将事件发送到事件中心、使用 EventProcessor 处理事件。

####[使用 服务总线 队列的排队消息解决方案](/zh-cn/documentation/articles/cloud-services-dotnet-multi-tier-app-using-service-bus-queues/)
构建一个前端 ASP.NET MVC Web 角色，该角色使用后端辅助角色来处理长时间运行的作业。您将了解如何创建和部署多角色解决方案，以及如何使用 服务总线 队列和主题来实现角色间通信。

####[一款使用 服务总线 主题的发布/订阅应用程序](http://code.msdn.microsoft.com/windowsazure/Simple-Publish-Subscribe-d406eb03)
使用发布/订阅模式创建一个将消息发送到多个目标的应用程序。

####[使用中继的混合云本地应用程序](/zh-cn/documentation/articles/cloud-services-dotnet-hybrid-app-using-service-bus-relay/)
在本教程中，您将使用 Ｗeb 角色创建一个调用防火墙后面的本地服务的网站。

####[使用 AMQP 的跨平台消息应用程序](/zh-cn/documentation/articles/service-bus-dotnet-advanced-message-queuing/)
此操作方法指南说明如何通过 .NET API 从 AMQP 1.0 使用 服务总线 中转消息传递功能（队列和发布/订阅主题）。

