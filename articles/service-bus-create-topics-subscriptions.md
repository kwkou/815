<properties 
   pageTitle="创建使用服务总线主题和订阅的应用程序 | Microsoft Azure"
   description="服务总线主题和订阅提供的发布-订阅功能简介。"
   services="service-bus"
   documentationCenter="na"
   authors="sethmanheim"
   manager="timlt"
   editor="tysonn" />
<tags 
   ms.service="service-bus"
    ms.date="03/16/2016"
   wacn.date="06/27/2016" />

# 创建使用服务总线主题和订阅的应用程序

Azure 服务总线支持一组基于云的、面向消息的中间件技术，包括可靠的消息队列和持久发布/订阅消息。本文以[创建使用服务总线队列的应用程序](/documentation/articles/service-bus-create-queues/)中提供的信息为基础，并介绍了服务总线主题提供的发布/订阅功能。

## 演变零售方案

本文将继续[创建使用服务总线队列的应用程序](/documentation/articles/service-bus-create-queues/)中使用的零售方案。回顾来自各销售点 (POS) 终端的销售数据必须路由到库存管理系统，该系统使用此数据来确定何时需要补充库存。每个 POS 终端通过向 **DataCollectionQueue** 队列发送消息来报告其销售数据，这些数据将保留在该队列中，直到库存管理系统接收它们，如下所示：

![Service-Bus1](./media/service-bus-create-topics-subscriptions/IC657161.gif)

为演变此方案，对系统增加了新的要求：零售店所有者需要能够实时监视零售店的绩效。

要解决这一要求，系统必须“分接”出销售数据流。我们仍然希望 POS 终端发送的每条消息像从前一样被发送到库存管理系统，但我们还需要每条消息的另一个副本，用于向商店所有者显示仪表板视图。

在任何类似于此的情况下，当需要每条消息供多方使用时，可以使用服务总线主题。主题提供发布/订阅模式，在该模式中，会为向主题注册的一个或多个订阅提供每个已发布消息。相比之下，使用队列时每条消息由单个使用方接收。

将消息发送到某主题的方式与将它们发送到队列的方式相同。但是，不会直接从该主题中接收消息；而是从订阅中接收消息。你可以将主题订阅视为接收发送至该主题的消息副本的虚拟队列。从订阅接收消息的方式与从队列接收的方式相同。

返回到零售方案时，队列被替换为一个主题，并添加了库存管理系统组件可以使用的订阅。系统现将如下所示：

![Service-Bus2](./media/service-bus-create-topics-subscriptions/IC657165.gif)

这里的配置方式与先前基于队列的设计相同。即，发送到主题的消息被路由到“库存”订阅，以便“库存管理系统”从中使用它们。

为了支持管理仪表板，我们创建了另一个主题订阅，如下所示：

![Service-Bus3](./media/service-bus-create-topics-subscriptions/IC657166.gif)

通过此配置，来自 POS 终端的每条消息对“仪表板”和“库存”订阅都可用。

## 显示相关代码

[创建使用服务总线队列的应用程序](/documentation/articles/service-bus-create-queues/)介绍了如何注册服务总线帐户以及如何创建服务命名空间。若要使用服务总线命名空间，应用程序必须引用服务总线程序集，特别是 Microsoft.ServiceBus.dll。引用服务总线依赖项的最简单方法是安装服务总线 [Nuget 程序包](https://www.nuget.org/packages/WindowsAzure.ServiceBus/)。还可以作为 Azure SDK 的一部分查找该程序集。可从 [Azure SDK 下载页](/downloads/)下载。

### 创建主题和订阅

服务总线消息传送实体（队列和发布/订阅主题）的管理操作通过 [NamespaceManager](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.namespacemanager.aspx) 类执行。需要适当的凭据才能为特定命名空间创建 [NamespaceManager](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.namespacemanager.aspx) 实例。服务总线使用基于安全模型的[共享访问签名 (SAS)](/documentation/articles/service-bus-sas-overview/)。[TokenProvider](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.tokenprovider.aspx) 类代表具有内置工厂方法的安全令牌提供程序，这些方法可返回一些众所周知的令牌提供程序。我们将使用 [CreateSharedAccessSignatureTokenProvider](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.tokenprovider.createsharedaccesssignaturetokenprovider.aspx) 方法来保留 SAS 凭据。然后使用服务总线命名空间和令牌提供程序的基址构建 [NamespaceManager](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.namespacemanager.aspx) 实例。

[NamespaceManager](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.namespacemanager.aspx) 类提供了创建、枚举和删除消息传送实体的方法。此处显示的代码介绍了创建 [NamespaceManager](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.namespacemanager.aspx) 实例并用它创建 **DataCollectionTopic** 主题的方法。

```
Uri uri = ServiceBusEnvironment.CreateServiceUri("sb", "test-blog", string.Empty);
string name = "RootManageSharedAccessKey";
string key = "abcdefghijklmopqrstuvwxyz";
     
TokenProvider tokenProvider = TokenProvider.CreateSharedAccessSignatureTokenProvider(name, key);
NamespaceManager namespaceManager = new NamespaceManager(uri, tokenProvider);
 
namespaceManager.CreateTopic("DataCollectionTopic");
```

请注意，存在 [CreateTopic](https://msdn.microsoft.com/zh-cn/library/azure/hh293080.aspx) 方法的重载，你可通过该方法设置主题的属性。例如，可为发送给主题的消息设置默认生存期 (TTL) 值。接下来，添加“库存”和“仪表板”订阅。

```
namespaceManager.CreateSubscription("DataCollectionTopic", "Inventory");
namespaceManager.CreateSubscription("DataCollectionTopic", "Dashboard");
```

### 将消息发送到主题

为了对服务总线实体进行运行时操作（例如发送和接收消息），应用程序必须首先创建 [MessagingFactory](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.messagingfactory.aspx) 对象。类似于 [NamespaceManager](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.namespacemanager.aspx) 类，[MessagingFactory](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.messagingfactory.aspx) 实例也从服务命名空间和令牌提供程序的基址创建。

```
MessagingFactory factory = MessagingFactory.Create(uri, tokenProvider);
```

在服务总线主题中发送和接收的消息是 [BrokeredMessage](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.brokeredmessage.aspx) 类的实例。此类包含一组标准属性（如 [Label](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.brokeredmessage.label.aspx) 和 [TimeToLive](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.brokeredmessage.timetolive.aspx)）、一个用来保存应用程序属性的词典以及大量随机应用程序数据。应用程序可以通过传入任何可序列化对象来设置正文（下面的示例传入 **SalesData** 对象，表示来自 POS 终端的销售数据），它将使用 [DataContractSerializer](https://msdn.microsoft.com/zh-cn/library/azure/system.runtime.serialization.datacontractserializer.aspx) 来序列化该对象。或者，也可以提供 [Stream](https://msdn.microsoft.com/zh-cn/library/azure/system.io.stream.aspx) 对象。

```
BrokeredMessage bm = new BrokeredMessage(salesData);
bm.Label = "SalesReport";
bm.Properties["StoreName"] = "Redmond";
bm.Properties["MachineID"] = "POS_1";
```

将消息发送到主题的最简单方法是使用 [CreateMessageSender](https://msdn.microsoft.com/zh-cn/library/azure/hh322659.aspx) 从 [MessagingFactory](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.messagingfactory.aspx) 实例直接创建 [MessageSender](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.messagesender.aspx) 对象。

```
MessageSender sender = factory.CreateMessageSender("DataCollectionTopic");
sender.Send(bm);
```

### 从订阅接收消息

类似于使用队列，若要从订阅接收消息，可以使用 [MessageReceiver](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.messagereceiver.aspx) 对象，可使用 [CreateMessageReceiver](https://msdn.microsoft.com/zh-cn/library/azure/hh322642.aspx) 从 [MessagingFactory](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.messagingfactory.aspx) 直接创建它。可以使用两种不同接收模式之一（**ReceiveAndDelete** 和 **PeekLock**），如[创建使用服务总线队列的应用程序](/documentation/articles/service-bus-create-queues/)中所述。

请注意，为订阅创建 [MessageReceiver](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.messagereceiver.aspx) 时，*entityPath* 参数的形式为 `topicPath/subscriptions/subscriptionName`。因此，若要为 **DataCollectionTopic** 主题的“库存”订阅创建 [MessageReceiver](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.messagereceiver.aspx)，必须将 *entityPath* 设置为 `DataCollectionTopic/subscriptions/Inventory`。代码将如下所示：

```
MessageReceiver receiver = factory.CreateMessageReceiver("DataCollectionTopic/subscriptions/Inventory");
BrokeredMessage receivedMessage = receiver.Receive();
try
{
    ProcessMessage(receivedMessage);
    receivedMessage.Complete();
}
catch (Exception e)
{
    receivedMessage.Abandon();
}
```

## 订阅筛选器

到目前为止，此方案中发送到主题的所有消息都可供所有已注册订阅使用。这里的关键词是“可供使用”。 虽然服务总线订阅可以看到发送到主题的所有消息，但你也可以仅将这些消息的一个子集复制到虚拟订阅队列。使用订阅筛选器执行此操作。创建订阅时，可以用 SQL92 样式谓词的形式提供筛选器表达式，对消息的属性进行操作，包括系统属性（例如 [Label](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.brokeredmessage.label.aspx)）和应用程序属性（例如以上示例中的 **StoreName**）。

推演此方案以说明这一点，向我们的零售方案中加入第二家零售店。来自两家零售店所有 POS 终端的销售数据仍必须路由到集中库存管理系统，但使用仪表板工具的零售店经理只对本店绩效感兴趣。可以使用订阅筛选来实现此目的。请注意，当 POS 终端发布消息时，会在消息上设置 **StoreName** 应用程序属性。给定两个零售店，例如 **Redmond** 和 **Seattle**，Redmond 商店中的 POS 终端使其销售数据带上等于 **Redmond** 的 **StoreName** 戳记，而 Seattle 商店 POS 终端使用的 **StoreName** 等于 **Seattle**。Redmond 商店经理仅希望查看来自其 POS 终端的数据。系统将如下所示：

![Service-Bus4](./media/service-bus-create-topics-subscriptions/IC657167.gif)

若要设置此路由，可以创建“仪表板”订阅，如下所示：

```
SqlFilter dashboardFilter = new SqlFilter("StoreName = 'Redmond'");
namespaceManager.CreateSubscription("DataCollectionTopic", "Dashboard", dashboardFilter);
```

通过此订阅筛选器，只有 **StoreName** 属性设置为 **Redmond** 的消息将复制到“仪表板”订阅的虚拟队列。而且，订阅筛选的功能还不止这些。应用程序可为每个订阅设定多个筛选器，还能在将消息传递给订阅的虚拟队列时修改其属性。

## 摘要

[创建使用服务总线队列的应用程序](/documentation/articles/service-bus-create-queues/)中所述所有使用队列的理由也适用于主题，具体如下：

- 暂时分离 – 消息创建方和使用方不必同时处于联机状态。

- 负载分级 – 通过主题允许将消费应用程序配置为平均负载而不是最高负载，均衡了负载中的峰值。

- 负载平衡 – 与队列类似，可使多个竞争使用方侦听单个订阅，将每条消息只移交给一个使用方，从而平衡负载。

- 松散耦合 – 还可以在不影响现有终结点的情况下发展消息传送网络，例如向主题添加订阅或更改其筛选器，以允许新的使用方使用它。

## 后续步骤

请参阅[创建使用服务总线队列的应用程序](/documentation/articles/service-bus-create-queues/)，以了解有关如何在 POS 零售方案中使用队列的信息。

<!---HONumber=Mooncake_0215_2016-->