<properties linkid="dev-net-how-to-service-bus-topics" urlDisplayName="Service Bus Topics" pageTitle="How to use Service Bus topics (.NET) - Azure" metaKeywords="Get started Azure Service Bus topics, Azure publish subscribe messaging, Azure messaging topics and subscriptions C# " description="Learn how to use Service Bus topics and subscriptions in Azure. Code samples are written for .NET applications. " metaCanonical="" services="service-bus" documentationCenter=".NET" title="How to Use Service Bus Topics/Subscriptions" authors="sethm" solutions="" manager="dwrede" editor="mattshel" />

# 如何使用 Service Bus 主题/订阅

<span>本指南演示如何使用 Service Bus 主题和订阅。相关示例用 C# 编写且使用 .NET API。涉及的应用场景包括创建主题和订阅、创建订阅筛选器、将消息发送到主题、从订阅接收消息以及删除主题和订阅。有关主题和订阅的详细信息，请参阅[后续步骤][后续步骤]一节。 </span>

[WACOM.INCLUDE [create-account-note](../includes/create-account-note.md)]

[WACOM.INCLUDE [howto-service-bus-topics](../includes/howto-service-bus-topics.md)]

## <span class="short-header">配置应用程序</span>配置应用程序以使用 Service Bus

在创建使用 Service Bus 的应用程序时，你需要添加对 Service Bus 程序集的引用并包括相应的命名空间。

## <span class="short-header">获取 NuGet 包</span>获取 Service Bus NuGet 包

Service Bus **NuGet** 包是获取 Service Bus API 并为应用程序配置所有 Service Bus 依赖项的最简单的方法。利用 NuGet Visual Studio 扩展，可以轻松地在 Visual Studio 和 Visual Studio Express 2012 for Web 中安装和更新库和工具。Service Bus NuGet 包是获取 Service Bus API 并为应用程序配置所有 Service Bus 依赖项的最简单的方法。

要在你的应用程序中安装 NuGet 包，请执行以下操作：

1.  在“解决方案资源管理器”中，右键单击**“引用”**，然后单击
    **“管理 NuGet 包”**。
2.  搜索“WindowsAzure”，然后选择“AzureService Bus”项。单击“安装”以完成安装，然后关闭此对话框。

    ![][0]

你现在可以为 Service Bus 编写代码。

## <span class="short-header">如何设置连接字符串</span>如何设置 Service Bus 连接字符串

Service Bus 使用连接字符串来存储终结点和凭据。你可以将连接字符串置于配置文件中，而不是在代码中对其进行硬编码：

-   在使用 Azure 云服务时，建议使用 Azure 服务配置系统（`*.csdef` 和 `*.cscfg` 文件）存储连接字符串。
-   在使用 Azure 网站或 Azure 虚拟机时，建议使用 .NET 配置系统（如`web.config` 文件）存储连接字符串。

在两种情况下，你都可以使用`CloudConfigurationManager.GetSetting` 方法检索你的连接字符串，如本指南后面所述。

### <a name="config-connstring"> </a> 使用云服务时配置连接字符串

该服务配置机制是 Azure 云服务项目特有的，它使你能够从 Azure 管理门户动态更改配置设置，而无需部署你的应用程序。例如，将 Setting 添加到你的服务定义 (`*.csdef`) 文件，如下所示：

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

在使用网站或虚拟机时，建议使用 .NET 配置系统（如`web.config`）。你可以使用`<appSettings>` 元素存储连接字符串：

    <configuration>
        <appSettings>
            <add key="Microsoft.ServiceBus.ConnectionString"
                 value="Endpoint=sb://[yourServiceNamespace].servicebus.windows.net/;SharedSecretIssuer=[issuerName];SharedSecretValue=[yourDefaultKey]" />
        </appSettings>
    </configuration>

使用从管理门户检索到的颁发者和密钥值，如上一节中所述。

## <span class="short-header">如何创建主题</span>如何创建主题

你可以通过 **NamespaceManager** 类为 Service Bus 主题和订阅执行管理操作。**NamespaceManager** 类提供了创建、枚举和删除队列的方法。

此示例使用带连接字符串的 Azure **CloudConfigurationManager** 类构造 **NamespaceManager** 对象，此连接字符串包含 Service Bus 服务命名空间的基址和有权管理该命名空间的相应凭据。此连接字符串的形式为

    Endpoint=sb://<yourServiceNamespace>.servicebus.windows.net/;SharedSecretIssuer=<issuerName>;SharedSecretValue=<yourDefaultKey>

例如，考虑上一节中的配置设置：

    // Create the topic if it does not exist already
    string connectionString = 
        CloudConfigurationManager.GetSetting("Microsoft.ServiceBus.ConnectionString");

    var namespaceManager = 
        NamespaceManager.CreateFromConnectionString(connectionString);

    if (!namespaceManager.TopicExists("TestTopic"))
    {
        namespaceManager.CreateTopic("TestTopic");
    }

**CreateTopic** 方法存在一些重载，允许你调整主题的属性，例如，将默认的“生存时间”值设置为应用于发送到主题的消息。使用
**TopicDescription** 类应用这些设置。下面的示例演示如何创建名为“TestTopic”、最大大小为 5 GB、默认消息生存时间为 1 分钟的主题。

    // Configure Topic Settings
    TopicDescription td = new TopicDescription("TestTopic");
    td.MaxSizeInMegabytes = 5120;
    td.DefaultMessageTimeToLive = new TimeSpan(0, 1, 0);

    // Create a new Topic with custom settings
    string connectionString = 
        CloudConfigurationManager.GetSetting("Microsoft.ServiceBus.ConnectionString");

    var namespaceManager = 
        NamespaceManager.CreateFromConnectionString(connectionString);

    if (!namespaceManager.TopicExists("TestTopic"))
    {
        namespaceManager.CreateTopic(td);
    }

**注意：**你可以对 **NamespaceManager** 对象使用 **TopicExists** 方法来检查具有指定名称的主题在某个服务命名空间中是否已存在。

## <span class="short-header">如何创建订阅</span>如何创建订阅

你也可以使用 **NamespaceManager** 类创建主题订阅。订阅已命名，并且具有一个限制传递到订阅的虚拟队列的消息集的可选筛选器。

### 创建具有默认 (MatchAll) 筛选器的订阅

**MatchAll** 筛选器是默认筛选器，在创建新订阅时未指定筛选器的情况下使用。使用 **MatchAll** 筛选器时，发布到主题的所有消息都将置于订阅的虚拟队列中。以下示例创建了名为 “AllMessages”的订阅并使用了默认的 **MatchAll** 筛选器。

    string connectionString = 
        CloudConfigurationManager.GetSetting("Microsoft.ServiceBus.ConnectionString");

    var namespaceManager = 
        NamespaceManager.CreateFromConnectionString(connectionString);

    if (!namespaceManager.SubscriptionExists("TestTopic", "AllMessages"))
    {
        namespaceManager.CreateSubscription("TestTopic", "AllMessages");
    }

### 创建具有筛选器的订阅

还可以设置筛选器，以指定发送到主题的哪些消息应该在特定主题订阅中显示。

订阅支持的最灵活的一种筛选器是**SqlFilter**，它实现了一部分 SQL92 功能。SQL 筛选器将对发布到主题的消息的属性进行操作。有关
可用于 SQL 筛选器的表达式的更多详细信息，请参阅 SqlFilter.SqlExpression 语法。

下面的示例创建了一个名为“HighMessages”的订阅（带有只选择具有大于 3 的 **MessageNumber** 属性的 **SqlFilter**）：

     // Create a "HighMessages" filtered subscription
     SqlFilter highMessagesFilter = 
        new SqlFilter("MessageNumber > 3");

     namespaceManager.CreateSubscription("TestTopic", 
        "HighMessages", 
        highMessagesFilter);

类似地，下面的示例创建一个名为 “LowMessages”的订阅，其 **SqlFilter** 只选择 **MessageNumber** 属性小于或等于 3 的消息：

     // Create a "LowMessages" filtered subscription
     SqlFilter lowMessagesFilter = 
        new SqlFilter("MessageNumber <= 3");

     namespaceManager.CreateSubscription("TestTopic", 
        "LowMessages", 
        lowMessagesFilter);

现在，当消息发送到“TestTopic”时，始终会将它传送到订阅了“AllMessages”主题订阅的接收者，并选择性地传送到订阅了“HighMessages”和“LowMessages”主题订阅的接收者（具体取决于消息内容）。

## <span class="short-header">向主题发送消息</span>如何向主题发送消息

若要向 Service Bus 主题发送消息，你的应用程序需要使用连接字符串创建 **TopicClient** 对象。

下面的代码演示如何使用 **CreateFromConnectionString** API 调用为上面创建的“TestTopic”主题创建 **TopicClient** 对象：

    string connectionString = 
        CloudConfigurationManager.GetSetting("Microsoft.ServiceBus.ConnectionString");

    TopicClient Client = 
        TopicClient.CreateFromConnectionString(connectionString, "TestTopic");

    Client.Send(new BrokeredMessage());

发送到 Service Bus 主题的消息是 **BrokeredMessage** 类的实例。**BrokeredMessage** 对象包含一组标准属性（如 **Label** 和 **TimeToLive**）、一个用来保存自定义应用程序特定属性的词典以及大量随机应用程序数据。应用程序可通过将任何可序列化对象传递到 **BrokeredMessage** 的构造函数来设置消息的正文，然后将使用适当的 **DataContractSerializer** 序列化该对象。或者，也可以提供**System.IO.Stream**。

以下示例演示如何将五条测试消息发送到在上面的代码片段中获取的“TestTopic”**TopicClient**。请注意每条消息的 **MessageNumber** 属性值在循环迭代上有怎样的不同（这一点将确定哪些订阅接收该消息）：

     for (int i=0; i<5; i++)
     {
       // Create message, passing a string message for the body
       BrokeredMessage message = 
        new BrokeredMessage("Test message " + i);

       // Set additional custom app-specific property
       message.Properties["MessageNumber"] = i;

       // Send message to the topic
       Client.Send(message);
     }

Service Bus 主题支持最大为 256 KB 的消息（标头最大为 64 KB，其中包括标准和自定义应用程序属性）。一个主题中包含的消息数量不受限制，但消息的总大小受限制。此队列大小是在创建时定义的，上限为 5 GB。

## <span class="short-header">从订阅接收消息</span>如何从订阅接收消息

从订阅接收消息的最简单方法是使用**SubscriptionClient** 对象。**SubscriptionClient** 对象可在两种不同模式下工作：**ReceiveAndDelete** 和 **PeekLock**。

当使用 **ReceiveAndDelete** 模式时，接收是一个单步操作，即，当 Service Bus 收到对订阅中某消息的读取请求时，它会将消息标记为“已使用”并将其返回给应用程序。**ReceiveAndDelete** 模式是最简单的模式，最适合应用程序允许出现故障时不处理消息的方案。为了理解这一点，可以考虑这样一种情形：使用方发出接收请求，但在处理该请求前发生了崩溃。由于 Service Bus 会将消息标记为“已使用”，因此当应用程序重新启动并重新开始使用消息时，它会漏掉在发生崩溃前使用的消息。

在 **PeekLock** 模式（这是默认模式）下，接收过程变成了一个两阶段操作，这样就可以支持无法容忍遗漏消息的应用程序。当 Service Bus 收到请求时，它会查找要使用的下一条消息，将其锁定以防其他使用方接收它，然后将该消息返回给应用程序。应用程序完成消息处理（或可靠地存储消息以供将来处理）后，它将通过对收到的消息调用 **Complete** 完成接收过程的第二个阶段。当 Service Bus 发现**Complete** 调用时，它会将消息标记为“已使用”并将其从订阅中删除。

下面的示例演示如何使用默认的 **PeekLock** 模式接收和处理消息。若要指定不同的 **ReceiveMode** 值，可以使用 **CreateFromConnectionString** 的另一个重载。此示例创建一个无限循环并在消息到达“HighMessages”订阅时处理消息。请注意，“HighMessages”订阅的路径以“\<*主题路径*\>/subscriptions/\<*订阅名称*\>”的形式提供。

    string connectionString = 
        CloudConfigurationManager.GetSetting("Microsoft.ServiceBus.ConnectionString");

    SubscriptionClient Client = 
        SubscriptionClient.CreateFromConnectionString
                (connectionString, "TestTopic", "HighMessages");

    Client.Receive();
     
    // Continuously process messages received from the HighMessages subscription 
    while (true) 
    {  
       BrokeredMessage message = Client.Receive();

       if (message != null)
       {
          try 
          {
             Console.WriteLine("Body: " + message.GetBody<string>());
             Console.WriteLine("MessageID: " + message.MessageId);
             Console.WriteLine("MessageNumber: " + 
                message.Properties["MessageNumber"]);

             // Remove message from subscription
             message.Complete();
          }
          catch (Exception)
          {
             // Indicate a problem, unlock message in subscription
             message.Abandon();
          }
       }
    } 

## <span class="short-header">应用程序崩溃和不可读消息</span>如何处理应用程序崩溃和不可读消息

Service Bus 提供了相关功能来帮助你轻松地从应用程序错误或消息处理问题中恢复。如果接收方应用程序因某种原因无法处理消息，它可以对收到的消息调用 **Abandon** 方法（而不是 **Complete** 方法）。这会导致 Service Bus 在订阅中将该消息解锁，使之再次可供
同一使用方应用程序或其他使用方应用程序接收。

还存在与订阅中的锁定消息关联的超时，如果应用程序未能在锁定超时过期前处理消息（例如，如果应用程序崩溃），Service Bus 将自动解锁该消息并使它重新可供接收。

如果应用程序在处理消息之后，但在发出 **Complete** 请求之前发生崩溃，则在应用程序重新启动时会将该消息重新传送给它。此情况通常称作**至少处理一次**，即每条消息将至少被处理一次，但在某些情况下，同一消息可能会被重新传送。如果方案无法容忍重复处理，则
应用程序开发人员应向其应用程序添加更多逻辑以处理重复消息传送。这通常可以通过使用消息的 **MessageId** 属性来实现，该属性在多次传送尝试中保持不变。

## <span class="short-header">删除主题和订阅</span>如何删除主题和订阅

下面的示例演示如何从“HowToSample”服务命名空间中删除名为**“TestTopic”**的主题：

     // Delete Topic
     namespaceManager.DeleteTopic("TestTopic");

删除某个主题也会删除向该主题注册的所有订阅。也可以单独删除订阅。下面的代码演示如何从“TestTopic”主题中删除名为 “HighMessages”的订阅：

      namespaceManager.DeleteSubscription("TestTopic", "HighMessages");

## <span class="short-header">后续步骤</span>后续步骤

现在，你已了解有关 Service Bus 主题和订阅的基础知识，单击下面的链接可了解更多信息。

-   查看 MSDN 参考：[队列、主题和订阅][队列、主题和订阅]。
-   [SqlFilter][SqlFilter] 的 API 参考。
-   构建向 Service Bus 队列发送消息以及从中接收
    消息的工作应用程序：[Service Bus 中转消息传送 .NET教程][Service Bus 中转消息传送 .NET教程]。

  [后续步骤]: #nextsteps
  [0]: ./media/service-bus-dotnet-how-to-use-topics-subscriptions/getting-started-multi-tier-13.png
  [队列、主题和订阅]: http://msdn.microsoft.com/zh-cn/library/hh367516.aspx
  [SqlFilter]: http://msdn.microsoft.com/zh-cn/library/microsoft.servicebus.messaging.sqlfilter.aspx
  [Service Bus 中转消息传送 .NET教程]: http://msdn.microsoft.com/zh-cn/library/hh367512.aspx
