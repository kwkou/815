<properties
    pageTitle="通过 .NET 使用服务总线主题 | Azure"
    description="了解如何在 Azure 中通过 .NET 使用服务总线主题和订阅。代码示例是针对 .NET 应用程序编写的。"
    services="service-bus"
    documentationCenter=".net"
    authors="sethmanheim"
    manager="timlt"
    editor=""/>

<tags
    ms.service="service-bus"
    ms.date="10/15/2015"
    wacn.date="02/26/2016"/>

# 如何使用服务总线主题和订阅

[AZURE.INCLUDE [service-bus-selector-topics](../includes/service-bus-selector-topics.md)]

本文介绍了如何使用服务总线主题和订阅。相关示例用 C# 编写且使用 .NET API。涉及的应用场景包括创建主题和订阅、创建订阅筛选器、将消息发送到主题、从订阅接收消息以及删除主题和订阅。有关主题和订阅的详细信息，请参阅[后续步骤](#Next-steps)部分。

[AZURE.INCLUDE [create-account-note](../includes/create-account-note.md)]

[AZURE.INCLUDE [howto-service-bus-topics](../includes/howto-service-bus-topics.md)]

## 配置应用程序以使用 Service Bus

在您创建使用 Service Bus 的应用程序时，必须添加对 Service Bus 程序集的引用并包括相应的命名空间。

## 获取服务总线 NuGet 包

服务总线 NuGet 包是获取服务总线 API 并为应用程序配置所有服务总线依赖项的最简单的方法。利用 NuGet Visual Studio 扩展，可以轻松地在 Visual Studio 和 Visual Studio Express 中安装和更新库和工具。服务总线 NuGet 包是获取服务总线 API 并为应用程序配置所有服务总线依赖项的最简单的方法。

要在你的应用程序中安装 NuGet 包，请执行以下操作：

1.  在解决方案资源管理器中，右键单击“引用”，然后单击“管理 NuGet 包”。
2.  搜索“服务总线”并选择“Azure 服务总线”项。单击“安装”以完成安装，然后关闭以下对话框。

    ![][7]

你现在可以为服务总线编写代码。

## 设置服务总线连接字符串

服务总线使用连接字符串来存储终结点和凭据。你可以将连接字符串置于配置文件中，而不是对其进行硬编码：

- 当使用 Azure 云服务时，建议你使用 Azure 服务配置系统（.csdef 和 .cscfg 文件）来存储连接字符串。
- 在使用 Azure 网站或 Azure 虚拟机时，建议使用 .NET 配置系统（如 Web.config 文件）来存储连接字符串。

在上述两种情况下，你都可以使用 `CloudConfigurationManager.GetSetting` 方法检索连接字符串，本文稍后部分将对此进行介绍。

### 使用云服务时配置连接字符串

该服务配置机制是 Azure 云服务项目特有的，它使你能够从 [Azure 经典门户][]动态更改配置设置，而无需重新部署你的应用程序。例如，向服务定义 (****.csdef**) 文件中添加 `Setting` 标签，如以下示例所示。

```
<ServiceDefinition name="WindowsAzure1">
...
    <WebRole name="MyRole" vmsize="Small">
        <ConfigurationSettings>
            <Setting name="Microsoft.ServiceBus.ConnectionString" />
        </ConfigurationSettings>
    </WebRole>
...
</ServiceDefinition>
```

然后在服务配置 (.cscfg) 文件中指定值。

```
<ServiceConfiguration serviceName="WindowsAzure1">
...
    <Role name="MyRole">
        <ConfigurationSettings>
            <Setting name="Microsoft.ServiceBus.ConnectionString"
                     value="Endpoint=sb://yourServiceNamespace.servicebus.chinacloudapi.cn/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=yourKey" />
        </ConfigurationSettings>
    </Role>
...
</ServiceConfiguration>
```

使用从 Azure 经典门户检索到的共享访问签名 (SAS) 密钥名称和密钥值，如上一部分中所述。

### 在使用 Azure 网站或 Azure 虚拟机时配置连接字符串

在使用网站或虚拟机时，建议你使用 .NET 配置系统（如 Web.config）。你可以使用 `<appSettings>` 元素存储连接字符串。

```
<configuration>
    <appSettings>
        <add key="Microsoft.ServiceBus.ConnectionString"
             value="Endpoint=sb://yourServiceNamespace.servicebus.chinacloudapi.cn/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=yourKey" />
    </appSettings>
</configuration>
```

使用从 Azure 经典门户检索到的 SAS 名称和密钥值，如上一部分中所述。

## 创建主题

你可以通过 [NamespaceManager](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.namespacemanager.aspx) 类为服务总线主题和订阅执行管理操作。此类提供了创建、枚举和删除主题的方法。

以下示例使用带连接字符串的 Azure `CloudConfigurationManager` 类构造 `NamespaceManager` 对象，此连接字符串包含服务总线服务命名空间的基址和有权管理该命名空间的相应 SAS 凭据。此连接字符串的形式如下。

```
Endpoint=sb://<yourServiceNamespace>.servicebus.chinacloudapi.cn/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=yourKey
```

使用以下示例，考虑上一节中的配置设置。

```
// Create the topic if it does not exist already.
string connectionString =
    CloudConfigurationManager.GetSetting("Microsoft.ServiceBus.ConnectionString");

var namespaceManager =
    NamespaceManager.CreateFromConnectionString(connectionString);

if (!namespaceManager.TopicExists("TestTopic"))
{
    namespaceManager.CreateTopic("TestTopic");
}
```

[CreateTopic](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.namespacemanager.createtopic.aspx) 方法存在一些重载，允许你调整主题的属性，例如，将默认的生存时间 (TTL) 值设置为应用于发送到主题的消息。使用 [TopicDescription](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.topicdescription.aspx) 类应用这些设置。以下示例演示如何创建名为 **TestTopic**、最大大小为 5 GB、默认消息 TTL 为 1 分钟的主题。

```
// Configure Topic Settings.
TopicDescription td = new TopicDescription("TestTopic");
td.MaxSizeInMegabytes = 5120;
td.DefaultMessageTimeToLive = new TimeSpan(0, 1, 0);

// Create a new Topic with custom settings.
string connectionString =
    CloudConfigurationManager.GetSetting("Microsoft.ServiceBus.ConnectionString");

var namespaceManager =
    NamespaceManager.CreateFromConnectionString(connectionString);

if (!namespaceManager.TopicExists("TestTopic"))
{
    namespaceManager.CreateTopic(td);
}
```

> [AZURE.NOTE]你可以对 [NamespaceManager](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.namespacemanager.aspx) 对象使用 [TopicExists](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.namespacemanager.topicexists.aspx) 方法来检查具有指定名称的主题在某个命名空间中是否已存在。

## 创建订阅

你还可以使用 [`NamespaceManager`](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.namespacemanager.aspx) 类创建主题订阅。订阅已命名，并且具有一个限制传递到订阅的虚拟队列的消息集的可选筛选器。

### 创建具有默认 (MatchAll) 筛选器的订阅

**MatchAll** 筛选器是默认筛选器，在创建新订阅时未指定筛选器的情况下使用。当你使用 **MatchAll** 筛选器时，发布到主题的所有消息都将置于订阅的虚拟队列中。以下示例创建名为“AllMessages”的订阅，并使用默认的 **MatchAll** 筛选器。

```
string connectionString =
    CloudConfigurationManager.GetSetting("Microsoft.ServiceBus.ConnectionString");

var namespaceManager =
    NamespaceManager.CreateFromConnectionString(connectionString);

if (!namespaceManager.SubscriptionExists("TestTopic", "AllMessages"))
{
    namespaceManager.CreateSubscription("TestTopic", "AllMessages");
}
```

### 创建具有筛选器的订阅

还可以设置筛选器，以指定发送到主题的哪些消息应该在特定主题订阅中显示。

订阅支持的最灵活的一种筛选器是 [SqlFilter][] 类，它实现了一部分 SQL92 功能。SQL 筛选器将对发布到主题的消息的属性进行操作。有关可用于 SQL 筛选器的表达式的详细信息，请参阅 [SqlFilter.SqlExpression][] 语法。

以下示例创建了一个名为 **HighMessages** 的订阅，该订阅仅选择具有大于 3 的自定义 **MessageNumber** 属性的 [SqlFilter][] 对象。

```
// Create a "HighMessages" filtered subscription.
SqlFilter highMessagesFilter =
   new SqlFilter("MessageNumber > 3");

namespaceManager.CreateSubscription("TestTopic",
   "HighMessages",
   highMessagesFilter);
```

类似地，以下示例创建一个名为 **LowMessages** 的订阅，其 [SqlFilter][] 只选择 **MessageNumber** 属性小于或等于 3 的消息：

```
// Create a "LowMessages" filtered subscription.
SqlFilter lowMessagesFilter =
   new SqlFilter("MessageNumber <= 3");

namespaceManager.CreateSubscription("TestTopic",
   "LowMessages",
   lowMessagesFilter);
```

现在，当消息发送到 `TestTopic` 时，始终会将它传送到订阅了 **AllMessages** 主题订阅的接收方，并选择性地传送到订阅了 **HighMessages** 和 **LowMessages** 主题订阅的接收方（具体取决于消息内容）。

## 将消息发送到主题

若要向服务总线主题发送消息，你的应用程序需要使用连接字符串创建 [TopicClient](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.topicclient.aspx) 对象。

以下代码演示了如何使用 [`CreateFromConnectionString`](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.topicclient.createfromconnectionstring.aspx) API 调用为以前创建的 **TestTopic** 主题创建 [TopicClient](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.topicclient.aspx) 对象。

```
string connectionString =
    CloudConfigurationManager.GetSetting("Microsoft.ServiceBus.ConnectionString");

TopicClient Client =
    TopicClient.CreateFromConnectionString(connectionString, "TestTopic");

Client.Send(new BrokeredMessage());
```

发送到服务总线主题的消息是 [BrokeredMessage](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.brokeredmessage.aspx) 类的实例。[BrokeredMessage](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.brokeredmessage.aspx) 对象包含一组标准属性（如 [Label](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.brokeredmessage.label.aspx) 和 [TimeToLive](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.brokeredmessage.timetolive.aspx)）、一个用来保存自定义应用程序特定属性的词典以及大量随机应用程序数据。应用程序可通过将任何可序列化对象传入到 [BrokeredMessage](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.brokeredmessage.aspx) 对象的构造函数中来设置消息的正文，然后将使用适当的 **DataContractSerializer** 序列化对象。或者，也可以提供 **System.IO.Stream**。

以下示例演示了如何将五条测试消息发送到在前面的代码示例中获取的 **TestTopic** [TopicClient](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.topicclient.aspx) 对象。请注意，每条消息的 [MessageNumber](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.brokeredmessage.properties.aspx) 属性值因循环迭代而异（这将确定哪些订阅接收它）。

```
for (int i=0; i<5; i++)
{
  // Create message, passing a string message for the body.
  BrokeredMessage message = new BrokeredMessage("Test message " + i);

  // Set additional custom app-specific property.
  message.Properties["MessageNumber"] = i;

  // Send message to the topic.
  Client.Send(message);
}
```

服务总线主题支持[最大为 256 Kb 的消息](/documentation/articles/service-bus-quotas)（标头最大为 64 Kb，其中包括标准和自定义应用程序属性）。一个主题中包含的消息数量不受限制，但消息的总大小受限制。此主题大小是在创建时定义的，上限为 5 GB。如果启用了分区，则上限更高。有关详细信息，请参阅[分区消息实体](https://msdn.microsoft.com/zh-cn/library/azure/dn520246.aspx)。

## 如何从订阅接收消息

从订阅接收消息的建议方法是使用 [SubscriptionClient](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.subscriptionclient.aspx) 对象。[SubscriptionClient](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.subscriptionclient.aspx) 对象可在两种不同模式下工作：[ReceiveAndDelete 和 PeekLock](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.receivemode.aspx)。

当使用 **ReceiveAndDelete** 模式时，接收是一个单步操作 - 即，当服务总线接收订阅中的消息读取请求时，它会将消息标记为“正在使用”并将其返回给应用程序。**ReceiveAndDelete** 模式是最简单的模式，最适合应用程序允许出现故障时不处理消息的方案。为了理解这一点，可以考虑这样一种情形：使用方发出接收请求，但在处理该请求前发生了崩溃。由于服务总线会将消息标记为“已使用”，因此当应用程序重新启动并重新开始使用消息时，它会漏掉在发生崩溃前使用的消息。

在 **PeekLock** 模式（这是默认模式）下，接收过程变成了一个两阶段操作，这样就可以支持无法容忍遗漏消息的应用程序。当 Service Bus 收到请求时，它会查找下一条要使用的消息，锁定该消息以防其他使用者接收，然后将该消息返回到应用程序。应用程序完成消息处理（或可靠地存储消息以供将来处理）后，它将通过对收到的消息调用 [Complete](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.brokeredmessage.complete.aspx) 完成接收过程的第二个阶段。当服务总线发现 **Complete** 调用时，它会将消息标记为“正在使用”并将其从订阅中删除。

以下示例演示如何使用默认的 **PeekLock** 模式接收和处理消息。若要指定不同的 [ReceiveMode](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.receivemode.aspx) 值，可以使用 [CreateFromConnectionString](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.subscriptionclient.createfromconnectionstring.aspx) 的另一个重载。此示例使用 [OnMessage](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.subscriptionclient.onmessage.aspx) 回调来处理传入 **HighMessages** 订阅的消息。

```
string connectionString =
    CloudConfigurationManager.GetSetting("Microsoft.ServiceBus.ConnectionString");

SubscriptionClient Client =
    SubscriptionClient.CreateFromConnectionString
            (connectionString, "TestTopic", "HighMessages");

// Configure the callback options.
OnMessageOptions options = new OnMessageOptions();
options.AutoComplete = false;
options.AutoRenewTimeout = TimeSpan.FromMinutes(1);

Client.OnMessage((message) =>
{
    try
    {
        // Process message from subscription.
        Console.WriteLine("\n**High Messages**");
        Console.WriteLine("Body: " + message.GetBody<string>());
        Console.WriteLine("MessageID: " + message.MessageId);
        Console.WriteLine("Message Number: " +
            message.Properties["MessageNumber"]);

        // Remove message from subscription.
        message.Complete();
    }
    catch (Exception)
    {
        // Indicates a problem, unlock message in subscription.
        message.Abandon();
    }
}, options);
```

此示例使用 [OnMessageOptions](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.onmessageoptions.aspx) 对象配置 [OnMessage](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.subscriptionclient.onmessage.aspx) 回调。将 [AutoComplete](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.onmessageoptions.autocomplete.aspx) 设置为 **false** 以允许手动控制何时对收到的消息调用 [Complete](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.brokeredmessage.complete.aspx)。将 [AutoRenewTimeout](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.onmessageoptions.autorenewtimeout.aspx) 设置为 1 分钟，这会导致客户端最多等待消息一分钟，然后调用会超时并且客户端将发出新的调用以检查是否有消息。此属性值会减少客户端无法检索消息时产生的应计费调用次数。

## 如何处理应用程序崩溃和不可读消息

Service Bus 提供了相关功能来帮助你轻松地从应用程序错误或消息处理问题中恢复。如果接收方应用程序因某种原因无法处理消息，则可以对收到的消息调用 [Abandon](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.brokeredmessage.abandon.aspx) 方法（而不是 [Complete](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.brokeredmessage.complete.aspx) 方法）。这会导致服务总线解锁订阅中的消息并使其能够重新被同一个正在使用的应用程序或其他正在使用的应用程序接收。

还存在与订阅中的锁定消息关联的超时，如果应用程序未能在锁定超时过期前处理消息（例如，如果应用程序崩溃），服务总线将自动解锁该消息并使它重新可供接收。

如果应用程序在处理消息之后，但在发出 [Complete](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.brokeredmessage.complete.aspx) 请求之前发生崩溃，则在应用程序重新启动时会将该消息重新传送给它。此情况通常称作*至少处理一次*，即每条消息将至少被处理一次，但在某些情况下，同一消息可能会被重新传送。如果方案无法容忍重复处理，则应用程序开发人员应向其应用程序添加更多逻辑以处理重复消息传送。这通常可以通过使用消息的 [MessageId](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.brokeredmessage.messageid.aspx) 属性来实现，该属性在多次传送尝试中保持不变。

## 删除主题和订阅

以下示例演示了如何从 **HowToSample** 服务命名空间中删除名为 **TestTopic** 的主题。

```
// Delete Topic.
namespaceManager.DeleteTopic("TestTopic");
```

删除某个主题也会删除向该主题注册的所有订阅。也可以单独删除订阅。以下代码演示如何从 **TestTopic** 主题中删除名为 **HighMessages** 的订阅。

```
namespaceManager.DeleteSubscription("TestTopic", "HighMessages");
```

## 后续步骤

现在，您已了解有关 Service Bus 主题和订阅的基础知识，单击下面的链接可了解更多信息。

-   请参阅[队列、主题和订阅][]。
-   [SqlFilter][] 的 API 参考。
-   构建向服务总线队列发送消息以及从中接收消息的工作应用程序：[服务总线中转消息传送 .NET 教程][]。
-   服务总线示例：从 [Azure 示例][]下载，或参阅[概述](/documentation/articles/service-bus-samples)。

  [Azure 经典门户]: http://manage.windowsazure.cn

  [7]: ./media/service-bus-dotnet-how-to-use-topics-subscriptions/getting-started-multi-tier-13.png
  
  [队列、主题和订阅]: /documentation/articles/service-bus-queues-topics-subscriptions
  [SqlFilter]: http://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.sqlfilter.aspx
  [SqlFilter.SqlExpression]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.sqlfilter.sqlexpression.aspx
  [服务总线中转消息传送 .NET 教程]: /documentation/articles/service-bus-brokered-tutorial-dotnet
  [Azure 示例]: https://code.msdn.microsoft.com/site/search?query=service%20bus&f%5B0%5D.Value=service%20bus&f%5B0%5D.Type=SearchText&ac=2

<!---HONumber=Mooncake_0215_2016-->