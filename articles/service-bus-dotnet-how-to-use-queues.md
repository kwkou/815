<properties linkid="dev-net-how-to-service-bus-queues" urlDisplayName="Service Bus Queues" pageTitle="How to use Service Bus queues (.NET) - Azure" metaKeywords="Azure Service Bus queues, Azure queues, Azure messaging, Azure queues C#, Azure queues .NET" description="Learn how to use Service Bus queues in Azure. Code samples written in C# using the .NET API." metaCanonical="" services="service-bus" documentationCenter=".NET" title="How to Use Service Bus Queues" authors="sethm" solutions="" manager="dwrede" editor="mattshel" />

# 如何使用 Service Bus 队列

<span>本指南演示如何使用 Service Bus 队列。相关示例用 C# 编写且使用 .NET API。所涉及的任务包括**创建队列、发送和接收消息**和**删除队列**。有关队列的详细信息，请参阅[后续步骤][后续步骤]一节。 </span>

[WACOM.INCLUDE [create-account-note](../includes/create-account-note.md)]

[WACOM.INCLUDE [howto-service-bus-queues](../includes/howto-service-bus-queues.md)]

## <span class="short-header">配置应用程序</span>配置应用程序以使用 Service Bus

在你创建使用 Service Bus 的应用程序时，必须
添加对 Service Bus 程序集的引用并包括
相应的命名空间。

## <span class="short-header">获取 NuGet 包</span>获取 Service Bus NuGet 包

Service Bus **NuGet** 包是获取 Service Bus API 并为应用程序配置所有 Service Bus 依赖项的最简单的方法。利用 NuGet Visual Studio 扩展，可以轻松地在 Visual Studio 和Visual Studio Express 2012 for Web 中安装和更新库和工具。Service Bus NuGet 包是获取 Service Bus API 并为应用程序配置所有 Service Bus 依赖项的最简单的方法。

要在你的应用程序中安装 NuGet 包，请执行以下操作：

1.  在“解决方案资源管理器”中，右键单击**“引用”**，然后单击**“管理 NuGet 包”**。
2.  搜索“WindowsAzure”，然后选择“AzureService Bus”项。单击“安装”以完成安装，然后关闭此对话框。

    ![][0]

你现在可以为 Service Bus 编写代码。

## <span class="short-header">设置连接字符串</span>如何设置 Service Bus 连接字符串

Service Bus 使用连接字符串来存储终结点和凭据。你可以将连接字符串置于配置文件中，而不是在代码中对其进行硬编码：

-   在使用 Azure 云服务时，建议使用 Azure 服务配置系统（`*.csdef` 和 `*.cscfg` 文件）存储连接字符串。
-   在使用 Azure 网站或 Azure 虚拟机时，建议使用 .NET 配置系统（如`web.config` 文件）存储连接字符串。

在两种情况下，你都可以使用`CloudConfigurationManager.GetSetting` 方法检索你的连接字符串，如本指南后面所述。

### <a name="config-connstring"> </a> 使用云服务时配置连接字符串

该服务配置机制是 Azure 云服务项目特有的，它使你能够从 Azure 管理门户动态更改配置设置，而无需部署你的应用程序。
例如，将 Setting 添加到你的服务定义 (`*.csdef`) 文件，如下所示：

    <ServiceDefinition name="WindowsAzure1">
    ...
        <WebRole name="MyRole" vmsize="Small">
            <ConfigurationSettings>
                <Setting name="Microsoft.ServiceBus.ConnectionString" />
            </ConfigurationSettings>
        </WebRole>
    ...
    </ServiceDefinition>

然后在服务配置 (`*.cscfg`) 文件中指定值：

    <ServiceConfiguration serviceName="WindowsAzure1">
    ...
        <Role name="MyRole">
            <ConfigurationSettings>
                <Setting name="Microsoft.ServiceBus.ConnectionString" 
                         value="Endpoint=sb://[yourServiceNamespace].servicebus.windows.net/;SharedSecretIssuer=[issuerName];SharedSecretValue=[yourDefaultKey]" />
            </ConfigurationSettings>
        </Role>
    ...
    </ServiceConfiguration>

使用从管理门户检索到的颁发者和密钥值，如上一节中所述。

### 在使用网站或虚拟机时配置连接字符串

在使用网站或虚拟机时，建议使用 .NET 配置系统（如`web.config`). 你可以使用`<appSettings>` 元素存储连接字符串：

    <configuration>
        <appSettings>
            <add key="Microsoft.ServiceBus.ConnectionString"
                 value="Endpoint=sb://[yourServiceNamespace].servicebus.windows.net/;SharedSecretIssuer=[issuerName];SharedSecretValue=[yourDefaultKey]" />
        </appSettings>
    </configuration>

使用从管理门户检索到的颁发者和密钥值，如上一节中所述。

## <span class="short-header">如何创建队列</span>如何创建队列

你可以通过 **NamespaceManager** 类对 Service Bus 队列执行管理操作。**NamespaceManager** 类提供了创建、枚举和删除队列的方法。

此示例使用带连接字符串的 Azure **CloudConfigurationManager** 类构造 **NamespaceManager** 对象，此连接字符串包含 Service Bus 服务命名空间的基址和有权管理该命名空间的相应凭据。此连接字符串的形式为

    Endpoint=sb://[yourServiceNamespace].servicebus.windows.net/;SharedSecretIssuer=[issuerName];SharedSecretValue=[yourDefaultKey]

例如，考虑上一节中的配置设置：

    // Create the queue if it does not exist already
    string connectionString = 
        CloudConfigurationManager.GetSetting("Microsoft.ServiceBus.ConnectionString");

    var namespaceManager = 
        NamespaceManager.CreateFromConnectionString(connectionString);

    if (!namespaceManager.QueueExists("TestQueue"))
    {
        namespaceManager.CreateQueue("TestQueue");
    }

这里使用了 **CreateQueue** 方法的重载，以允许你调整队列属性例如，为了将默认“生存时间”值设置为应用于发送到队列的消息）。使用**QueueDescription** 类应用这些设置。下面的示例演示如何创建最大大小为 5GB 且默认消息生存时间为 1 分钟的名为“TestQueue”的队列：

    // Configure Queue Settings
    QueueDescription qd = new QueueDescription("TestQueue");
    qd.MaxSizeInMegabytes = 5120;
    qd.DefaultMessageTimeToLive = new TimeSpan(0, 1, 0);

    // Create a new Queue with custom settings
    string connectionString = 
        CloudConfigurationManager.GetSetting("Microsoft.ServiceBus.ConnectionString");

    var namespaceManager = 
        NamespaceManager.CreateFromConnectionString(connectionString);

    if (!namespaceManager.QueueExists("TestQueue"))
    {
        namespaceManager.CreateQueue(qd);
    }

**注意：**你可以对 **NamespaceManager** 对象使用 **QueueExists**方法，以检查具有指定名称的队列是否已位于服务命名空间中。

## <span class="short-header">向队列发送消息</span>如何向队列发送消息

若要向 Service Bus 队列发送消息，你的应用程序需要使用连接字符串创建**QueueClient** 对象。

下面的代码演示如何使用 **CreateFromConnectionString** API 调用为上面创建的“TestQueue”队列创建 **QueueClient** 对象：

    string connectionString = 
        CloudConfigurationManager.GetSetting("Microsoft.ServiceBus.ConnectionString");

    QueueClient Client = 
        QueueClient.CreateFromConnectionString(connectionString, "TestQueue");

    Client.Send(new BrokeredMessage());

发送到（和接收自）Service Bus 队列的消息是**BrokeredMessage** 类的实例。**BrokeredMessage** 对象包含一组标准属性（如 **Label** 和 **TimeToLive**）、一个用来保存自定义应用程序特定属性的词典以及大量随机应用程序数据。应用程序可通过将任何可序列化对象传递到**BrokeredMessage** 的构造函数中来设置消息的正文，然后将使用适当的 **DataContractSerializer** 序列化该对象。或者，也可以提供**System.IO.Stream**。

下面的示例演示如何将五条测试消息发送到在上面的代码段中获得的“TestQueue”**QueueClient**：

     for (int i=0; i<5; i++)
     {
       // Create message, passing a string message for the body
       BrokeredMessage message = new BrokeredMessage("Test message " + i);

       // Set some addtional custom app-specific properties
       message.Properties["TestProperty"] = "TestValue";
       message.Properties["Message number"] = i;   

       // Send message to the queue
       Client.Send(message);
     }

Service Bus 队列支持最大为 256 KB 的消息（标头最大为 64 KB，其中包括标准和自定义应用程序属性）。一个队列可包含的消息数不受限制，但消息的总大小受限。此队列大小是在创建时定义的，上限为 5 GB。

## <span class="short-header">从队列接收消息</span>如何从队列接收消息

从队列接收消息的最简单方法是使用**QueueClient** 对象。这些对象可在两种不同模式下工作：**ReceiveAndDelete** 和 **PeekLock**。

当使用 **ReceiveAndDelete** 模式时，接收是一项单步操作。即，当 Service Bus 收到对队列中某消息的读取请求时，它会将该消息
标记为“已使用”并将其返回给应用程序。**ReceiveAndDelete** 模式是最简单的模式，最适合应用程序允许出现故障时不处理消息的方案。为了理解这一点，可以考虑这样一种情形：使用方发出接收请求，但在处理该请求前发生了崩溃。由于 Service Bus 会将消息标记为“已使用”，因此当应用程序重新启动并重新开始使用消息时，它会漏掉在发生崩溃前使用的消息。

在 **PeekLock** 模式（这是默认模式）下，接收过程变成了一个两阶段操作，这样就可以支持无法容忍遗漏消息的应用程序。当 Service Bus 收到请求时，它会查找要使用的下一条消息，将其锁定以防其他使用方接收它，然后将该消息返回给应用程序。应用程序完成消息处理（或可靠地存储消息以供将来处理）后，它将通过对收到的消息调用 **Complete** 完成接收过程的第二个阶段。当 Service Bus 发现**Complete** 调用时，它会将消息标记为“已使用”并将其从队列中删除。

下面的示例演示如何使用默认的 **PeekLock** 模式接收和处理消息。若要指定不同的 **ReceiveMode** 值，可以使用**CreateFromConnectionString** 的另一个重载。此示例创建无限循环并在消息到达“TestQueue”后对其进行处理：

    Client.Receive();
     
    // Continuously process messages sent to the "TestQueue" 
    while (true) 
    {  
       BrokeredMessage message = Client.Receive();

       if (message != null)
       {
          try 
          {
             Console.WriteLine("Body: " + message.GetBody<string>());
             Console.WriteLine("MessageID: " + message.MessageId);
             Console.WriteLine("Test Property: " + 
                message.Properties["TestProperty"]);

             // Remove message from queue
             message.Complete();
          }
          catch (Exception)
          {
             // Indicate a problem, unlock message in queue
             message.Abandon();
          }
       }
    } 

## <span class="short-header">应用程序崩溃和不可读消息</span>如何处理应用程序崩溃和不可读消息

Service Bus 提供了相关功能来帮助你轻松地从应用程序错误或消息处理问题中恢复。如果接收方应用程序因某种原因无法处理消息，它可以对收到的消息调用 **Abandon** 方法（而不是 **Complete** 方法）。这会导致 Service Bus 在队列中将该消息解锁，使之再次可供同一使用方应用程序或其他使用方应用程序接收。

还存在与队列中的锁定消息关联的超时，如果应用程序未能在锁定超时过期前处理消息（例如，如果应用程序崩溃），Service Bus 将自动解锁该消息并使它重新可供接收。

如果应用程序在处理消息之后，但在发出 **Complete** 请求之前发生崩溃，则在应用程序重新启动时会将该消息重新传送给它。此情况通常称作**至少处理一次**，即每条消息将至少被处理一次，但在某些情况下，同一消息可能会被重新传送。如果方案无法容忍重复处理，则应用程序开发人员应向其应用程序添加更多逻辑以处理重复消息传送。这通常可以通过使用消息的**MessageId** 属性来实现，该属性在多次传送尝试中保持不变。

## <span class="short-header">后续步骤</span>后续步骤

现在，你已了解有关 Service Bus 队列的基础知识，单击下面的链接可了解更多信息。

-   查看 MSDN 参考：[队列、主题和订阅。][队列、主题和订阅。]
-   构建向 Service Bus 队列发送消息以及从中接收消息的工作应用程序：[Service Bus 中转消息传送 .NET教程][Service Bus 中转消息传送 .NET教程]。

  [后续步骤]: #next-steps
  [0]: ./media/service-bus-dotnet-how-to-use-queues/getting-started-multi-tier-13.png
  [队列、主题和订阅。]: http://msdn.microsoft.com/zh-cn/library/windowsazure/hh367516.aspx
  [Service Bus 中转消息传送 .NET教程]: http://msdn.microsoft.com/zh-cn/library/windowsazure/hh367512.aspx
