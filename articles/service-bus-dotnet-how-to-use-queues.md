<properties
    pageTitle="如何使用服务总线队列 (.NET) | Azure"
    description="了解如何在 Azure 中使用 Service Bus 队列。代码示例是使用 .NET API 通过 C# 编写的。"
    services="service-bus"
    documentationCenter=".net"
    authors="sethmanheim"
    manager="timlt"
    editor=""/>

<tags
    ms.service="service-bus"
    ms.date="05/09/2016"
    wacn.date="06/21/2016"/>

# 如何使用 Service Bus 队列

[AZURE.INCLUDE [service-bus-selector-queues](../includes/service-bus-selector-queues.md)]

本文介绍了如何使用服务总线队列。相关示例采用 C# 编写且使用 .NET API。涉及的方案包括创建队列以及发送和接收消息。有关队列的详细信息，请参阅[后续步骤](#next-steps)部分。

[AZURE.INCLUDE [create-account-note](../includes/create-account-note.md)]

[AZURE.INCLUDE [howto-service-bus-queues](../includes/howto-service-bus-queues.md)]

## 添加服务总线 NuGet 包

[服务总线 NuGet 包](https://www.nuget.org/packages/WindowsAzure.ServiceBus)是获取服务总线 API 并为应用程序配置所有服务总线依赖项的最简单的方法。要在你的应用程序中安装 NuGet 包，请执行以下操作：

要在你的应用程序中安装 NuGet 包，请执行以下操作：

1.  在解决方案资源管理器中，右键单击“引用”，然后单击“管理 NuGet 包”。
2.  搜索“服务总线”并选择“Azure 服务总线”项。单击“安装”以完成安装，然后关闭此对话框。

    ![][7]

你现在可以为服务总线编写代码。

## 设置服务总线连接字符串

服务总线使用连接字符串来存储终结点和凭据。你可以将连接字符串置于配置文件中，而不是对其进行硬编码：

- 当使用 Azure 云服务时，建议你使用 Azure 服务配置系统（.csdef 和 .cscfg 文件）来存储连接字符串。
- 在使用 Azure 网站或 Azure 虚拟机时，建议使用 .NET 配置系统（如 Web.config 文件）来存储连接字符串。

在上述两种情况下，你都可以使用 [CloudConfigurationManager.GetSetting][GetSetting] 方法检索连接字符串，本文稍后部分将对此进行介绍。

### 配置连接字符串

利用该服务配置机制，你可以从 [Azure 经典门户][]动态更改配置设置，而无需重新部署应用程序。例如，向服务定义 (.csdef) 文件中添加 `Setting` 标签，如以下示例所示。

```
<ServiceDefinition name="Azure1">
...
    <WebRole name="MyRole" vmsize="Small">
        <ConfigurationSettings>
            <Setting name="Microsoft.ServiceBus.ConnectionString" />
        </ConfigurationSettings>
    </WebRole>
...
</ServiceDefinition>
```
然后在服务配置 (.cscfg) 文件中指定值，如以下示例所示。

```
<ServiceConfiguration serviceName="Azure1">
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

### 在使用网站或 Azure 虚拟机时配置连接字符串

在使用网站或虚拟机时，建议你使用 .NET 配置系统（如 **Web.config**）。你可以使用 `<appSettings>` 元素存储连接字符串：

```
<configuration>
    <appSettings>
        <add key="Microsoft.ServiceBus.ConnectionString"
             value="Endpoint=sb://yourServiceNamespace.servicebus.chinacloudapi.cn/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=yourKey" />
    </appSettings>
</configuration>
```

使用从 Azure 经典门户检索到的 SAS 名称和密钥值，如上一部分中所述。

## 创建队列

你可以使用 [NamespaceManager][] 类对服务总线队列执行管理操作。此类提供了创建、枚举和删除队列的方法。

此示例使用带连接字符串的 Azure [CloudConfigurationManager][] 类构造 [NamespaceManager][] 对象，此连接字符串包含服务总线服务命名空间的基址和有权管理该命名空间的相应 SAS 凭据。此连接字符串的格式如以下示例所示。

````
Endpoint=sb://yourServiceNamespace.servicebus.chinacloudapi.cn/;SharedAccessKeyName=RootManageSharedAccessKey;SharedSecretValue=yourKey
````

使用以下示例，考虑上一节中的配置设置。

```
// Create the queue if it does not exist already.
string connectionString =
    CloudConfigurationManager.GetSetting("Microsoft.ServiceBus.ConnectionString");

var namespaceManager =
    NamespaceManager.CreateFromConnectionString(connectionString);

if (!namespaceManager.QueueExists("TestQueue"))
{
    namespaceManager.CreateQueue("TestQueue");
}
```

这里使用了 [CreateQueue](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.namespacemanager.createqueue.aspx) 方法的重载，以允许你调整队列属性（例如，为了将默认的生存时间 (TTL) 值设置为应用于发送到队列的消息）。使用 [QueueDescription](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.queuedescription.aspx) 类应用这些设置。以下示例演示如何创建名为 `TestQueue`、最大大小为 5 GB、默认消息 TTL 为 1 分钟的队列。

```
// Configure queue settings.
QueueDescription qd = new QueueDescription("TestQueue");
qd.MaxSizeInMegabytes = 5120;
qd.DefaultMessageTimeToLive = new TimeSpan(0, 1, 0);

// Create a new queue with custom settings.
string connectionString =
    CloudConfigurationManager.GetSetting("Microsoft.ServiceBus.ConnectionString");

var namespaceManager =
    NamespaceManager.CreateFromConnectionString(connectionString);

if (!namespaceManager.QueueExists("TestQueue"))
{
    namespaceManager.CreateQueue(qd);
}
```

> [AZURE.NOTE]你可以对 [NamespaceManager][] 对象使用 [QueueExists](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.namespacemanager.queueexists.aspx) 方法，以检查具有指定名称的队列是否已存在于某个服务命名空间中。

## 向队列发送消息

若要向服务总线队列发送消息，你的应用程序需使用连接字符串创建 [QueueClient][] 对象。

以下代码演示如何使用 [CreateFromConnectionString](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.queueclient.createfromconnectionstring.aspx) API 调用为刚创建的 `TestQueue` 队列创建 [QueueClient][] 对象。

```
string connectionString =
    CloudConfigurationManager.GetSetting("Microsoft.ServiceBus.ConnectionString");

QueueClient Client =
    QueueClient.CreateFromConnectionString(connectionString, "TestQueue");

Client.Send(new BrokeredMessage());
```

在服务总线队列中发送和接收的消息是 [BrokeredMessage][] 类的实例。[BrokeredMessage][] 对象包含一组标准属性（如 [Label](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.brokeredmessage.label.aspx) 和 [TimeToLive](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.brokeredmessage.timetolive.aspx)）、一个用来保存自定义应用程序特定属性的词典以及大量随机应用程序数据。应用程序可通过将任何可序列化对象传入到 [BrokeredMessage][] 对象的构造函数中来设置消息的正文，然后将使用适当的 **DataContractSerializer** 序列化对象。或者，你可以提供 **System.IO.Stream** 对象。

以下示例演示了如何将五条测试消息发送到在前面的代码示例中获取的 `TestQueue` [QueueClient][] 对象。

```
for (int i=0; i<5; i++)
{
  // Create message, passing a string message for the body.
  BrokeredMessage message = new BrokeredMessage("Test message " + i);

  // Set some addtional custom app-specific properties.
  message.Properties["TestProperty"] = "TestValue";
  message.Properties["Message number"] = i;

  // Send message to the queue.
  Client.Send(message);
}
```

服务总线队列支持[最大为 256 Kb 的消息](/documentation/articles/service-bus-azure-and-service-bus-queues-compared-contrasted/#capacity-and-quotas)（标头最大为 64 KB，其中包括标准和自定义应用程序属性）。一个队列可包含的消息数不受限制，但消息的总大小受限。此队列大小是在创建时定义的，上限为 5 GB。如果启用了分区，则上限更高。有关详细信息，请参阅[分区消息传送实体](/documentation/articles/service-bus-partitioning/)。

## 如何从队列接收消息

从队列接收消息的建议方法是使用 [QueueClient][] 对象。[QueueClient][] 对象可在两种不同模式下工作：[ReceiveAndDelete 和 PeekLock](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.receivemode.aspx)。

当使用 **ReceiveAndDelete** 模式时，接收是一项单步操作。即，当服务总线收到对队列中某消息的读取请求时，它会将该消息标记为“已使用”并将其返回给应用程序。**ReceiveAndDelete** 是最简单的模式，最适合应用程序允许出现故障时不处理消息的方案。为了理解这一点，可以考虑这样一种情形：使用方发出接收请求，但在处理该请求前发生了崩溃。由于服务总线会将消息标记为“已使用”，因此当应用程序重新启动并重新开始使用消息时，它会漏掉在发生崩溃前使用的消息。

在 **PeekLock** 模式（这是默认模式）下，接收变成了一个两阶段操作，从而有可能支持不允许遗漏消息的应用程序。当 Service Bus 收到请求时，它会查找下一条要使用的消息，锁定该消息以防其他使用者接收，然后将该消息返回到应用程序。应用程序完成消息处理（或可靠地存储消息以供将来处理）后，它将通过对收到的消息调用 [Complete][] 完成接收过程的第二个阶段。当服务总线发现 [Complete][] 调用时，它会将消息标记为“已使用”并将其从队列中删除。

以下示例演示如何使用默认的 **PeekLock** 模式接收和处理消息。若要指定不同的 [ReceiveMode](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.receivemode.aspx) 值，可以使用 [CreateFromConnectionString](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.queueclient.createfromconnectionstring.aspx) 的另一个重载。此示例使用 [OnMessage](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.queueclient.onmessage.aspx) 回调来处理传入 `TestQueue` 的消息。

```
string connectionString =
  CloudConfigurationManager.GetSetting("Microsoft.ServiceBus.ConnectionString");
QueueClient Client =
  QueueClient.CreateFromConnectionString(connectionString, "TestQueue");

// Configure the callback options.
OnMessageOptions options = new OnMessageOptions();
options.AutoComplete = false;
options.AutoRenewTimeout = TimeSpan.FromMinutes(1);

// Callback to handle received messages.
Client.OnMessage((message) =>
{
    try
    {
        // Process message from queue.
        Console.WriteLine("Body: " + message.GetBody<string>());
        Console.WriteLine("MessageID: " + message.MessageId);
        Console.WriteLine("Test Property: " +
        message.Properties["TestProperty"]);

        // Remove message from queue.
        message.Complete();
    }
    catch (Exception)
    {
        // Indicates a problem, unlock message in queue.
        message.Abandon();
    }
}, options);
```

此示例使用 [OnMessageOptions](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.onmessageoptions.aspx) 对象配置 [OnMessage](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.queueclient.onmessage.aspx) 回调。将 [AutoComplete](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.onmessageoptions.autocomplete.aspx) 设置为 **false** 以允许手动控制何时对收到的消息调用 [Complete][]。将 [AutoRenewTimeout](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.onmessageoptions.autorenewtimeout.aspx) 设置为 1 分钟，这会导致客户端最多等待消息一分钟，然后调用会超时并且客户端将发出新的调用以检查是否有消息。此属性值会减少客户端无法检索消息时产生的应计费调用次数。

## 如何处理应用程序崩溃和不可读消息

服务总线提供了相关功能来帮助你轻松地从应用程序错误或消息处理问题中恢复。如果接收方应用程序出于某种原因无法处理消息，它可以对收到的消息调用 [Abandon](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.brokeredmessage.abandon.aspx) 方法（而不是 [Complete][] 方法）。这将导致服务总线解锁队列中的消息并使其能够重新被同一个正在使用的应用程序或其他正在使用的应用程序接收。

还存在与队列中已锁定消息关联的超时，并且如果应用程序无法在锁定超时到期之前处理消息（例如，如果应用程序崩溃），服务总线将自动解锁该消息并使它可再次被接收。

如果应用程序在处理消息之后，但在发出 [Complete][] 请求之前发生崩溃，则在应用程序重新启动时会将该消息重新传送给它。此情况通常称作**至少处理一次**，即每条消息将至少被处理一次，但在某些情况下，同一消息可能会被重新传送。如果方案无法容忍重复处理，则应用程序开发人员应向其应用程序添加更多逻辑以处理重复消息传送。这通常可以通过使用消息的 [MessageId](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.brokeredmessage.messageid.aspx) 属性来实现，该属性在多次传送尝试中保持不变。
 
## 后续步骤

现在，你已了解有关 Service Bus 队列的基础知识，单击下面的链接可了解更多信息。

-   阅读有关[队列、主题和订阅][]中的服务总线消息实体的信息。
-   使用[服务总线中转消息传送 .NET 教程][]构建向服务总线队列发送消息以及从中接收消息的工作应用程序。
-   从 [Azure 示例][]下载服务总线示例，或参阅[服务总线示例概述][]。

  [Azure 经典门户]: http://manage.windowsazure.cn
  [7]: ./media/service-bus-dotnet-how-to-use-queues/getting-started-multi-tier-13.png
  [队列、主题和订阅]: /documentation/articles/service-bus-queues-topics-subscriptions/
  [服务总线中转消息传送 .NET 教程]: /documentation/articles/service-bus-brokered-tutorial-dotnet/
  [Azure 示例]: https://code.msdn.microsoft.com/site/search?query=service%20bus&f%5B0%5D.Value=service%20bus&f%5B0%5D.Type=SearchText&ac=2
  [服务总线示例概述]: /documentation/articles/service-bus-samples/
  [GetSetting]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.windowsazure.cloudconfigurationmanager.getsetting.aspx
  [CloudConfigurationManager]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.windowsazure.cloudconfigurationmanager
  [NamespaceManager]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.namespacemanager.aspx
  [BrokeredMessage]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.brokeredmessage.aspx
  [QueueClient]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.queueclient.aspx
  [Complete]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.brokeredmessage.complete.aspx

<!---HONumber=Mooncake_0104_2016-->