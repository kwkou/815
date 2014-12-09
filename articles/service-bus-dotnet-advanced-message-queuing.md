<properties linkid="develop-net-how-to-guides-service-bus-amqp" urlDisplayName="Service Bus AMQP" pageTitle="How to use AMQP 1.0 with the .NET Service Bus API - Azure" metaKeywords="" description="Learn how to use Advanced Message Queuing Protodol (AMQP) 1.0 with the Azure .NET Service Bus API." metaCanonical="" services="service-bus" documentationCenter=".NET" title="How to use AMQP 1.0 with the Service Bus .NET API" authors="sethm" solutions="" manager="dwrede" editor="mattshel" />

# 如何将 AMQP 1.0 与 Service Bus .NET API 一起使用

## <span class="short-header">简介</span>简介

</h2>
高级消息队列协议 (AMQP) 1.0 是一个高效、可靠的线级消息传送协议，可用于构建可靠的跨平台消息传送应用程序。

在 Service Bus 中支持 AMQP 1.0 意味着可以通过一系列使用有效的二进制协议的平台利用队列和发布/订阅中转消息传送功能。此外，你还可以生成由结合使用多个语言、框架和操作系统构建的组件组成的应用程序。

本操作方法指南说明如何使用 Service Bus .NET API 通过 .NET 应用程序来使用 Service Bus 中转消息传送功能（队列和发布/订阅主题）。有一个随附的操作方法指南，该指南说明如何使用标准 Java 消息服务 (JMS) API 执行相同的操作。使用 AMQP 1.0，可以同时使用以下两个指南来了解跨平台消息。

## <span class="short-header">入门</span>Service Bus 入门

此指南假定你已具有包含名为“queue1”的队列的 Service Bus 命名空间。如果没有，则可以使用 [Azure 管理门户][Azure 管理门户]创建命名空间和队列。有关如何创建 Service Bus 命名空间和队列的详细信息，请参阅名为“[如何使用 Service Bus 队列（可能为英文页面）][如何使用 Service Bus 队列（可能为英文页面）]”的操作方法指南。

## <span class="short-header">下载 SDK</span>下载 Service Bus SDK

AMQP 1.0 支持在 Service Bus SDK 2.1 版或更高版本中提供。可从以下位置的 NuGet 中下载最新的 SDK：[][]<http://nuget.org/packages/WindowsAzure.ServiceBus/></a>。

## <span class="short-header">为 .NET 应用程序编码</span>为 .NET 应用程序编码

默认情况下，Service Bus .NET 客户端库使用基于 SOAP 的专用协议与 Service Bus 服务通信。若要使用 AMQP 1.0 而非默认协议，需要 Service Bus 连接字符串上的显式配置，如下一节所述。除了此更改之外，在使用 AMQP 1.0 时应用程序代码基本保持不变。

在当前版本中，有一些在使用 AMQP 时不受支持的 API 功能。这些不受支持的功能将在后面的“不受支持的功能和限制”一节中列出。在使用 AMQP 时，一些高级配置设置还具有不同的含义。在本简短的操作方法指南中，没有使用这些设置，但在 [Service Bus AMQP 1.0 开发人员指南][Service Bus AMQP 1.0 开发人员指南]中提供了更多详细信息。

### 通过 App.config 进行配置

应用程序使用 App.config 配置文件存储设置是一个很好的做法。对于 Service Bus 应用程序，你可以使用 App.config 来存储 Service Bus **ConnectionString** 的设置。另外，下面的应用程序示例存储它所使用的 Service Bus 消息传送实体的名称。

下面显示了 App.config 文件示例：

    <?xml version="1.0" encoding="utf-8" ?>
    <configuration>
        <appSettings>
            <add key="Microsoft.ServiceBus.ConnectionString"
                 value="Endpoint=sb://[namespace].servicebus.windows.net;SharedSecretIssuer=[issuer name];SharedSecretValue=[issuer key];TransportType=Amqp" />
            <add key="EntityName" value="queue1" />
        </appSettings>
    </configuration>

### 配置 Service Bus 连接字符串

**Microsoft.ServiceBus.ConnectionString** 设置的值是用于配置与 Service Bus 的连接的 Service Bus 连接字符串，其格式如下所示：

    Endpoint=sb://[namespace].servicebus.windows.net;SharedSecretIssuer=[issuer name];SharedSecretValue=[issuer key];TransportType=Amqp

其中，[namespace]、[issuer name] 和 [issuer key] 可从 Azure 管理门户获得。有关详细信息，请参阅[如何使用 Service Bus 队列][如何使用 Service Bus 队列]。

在使用 AMQP 时，连接字符串附加了“;TransportType=Amqp”，以通知客户端库使用 AMQP 1.0 连接到 Service Bus。

### 配置实体名称

此应用程序示例使用 App.config 文件的 **appSettings** 节中的“EntityName”设置来配置应用程序与其交换消息的队列的名称。

### 使用 Service Bus 队列的简单 .NET 应用程序

以下示例向 Service Bus 队列发送消息以及从中接收消息。

    // SimpleSenderReceiver.cs

    using System;
    using System.Configuration;
    using System.Threading;
    using Microsoft.ServiceBus.Messaging;

    namespace SimpleSenderReceiver
    {
        class SimpleSenderReceiver
        {
            private static string connectionString;
            private static string entityName;
            private static Boolean runReceiver = true;
            private MessagingFactory factory;
            private MessageSender sender;
            private MessageReceiver receiver;
            private MessageListener messageListener;
            private Thread listenerThread;

            static void Main(string[] args)
            {
                try
                {
                    if ((args.Length > 0) && args[0].ToLower().Equals("sendonly"))
                        runReceiver = false;
                    
                    string ConnectionStringKey = "Microsoft.ServiceBus.ConnectionString";
                    string entityNameKey = "EntityName";
                    entityName = ConfigurationManager.AppSettings[entityNameKey];
                    connectionString = ConfigurationManager.AppSettings[ConnectionStringKey];
                    SimpleSenderReceiver simpleSenderReceiver = new SimpleSenderReceiver();

                    Console.WriteLine("Press [enter] to send a message. " +
                        "Type 'exit' + [enter] to quit.");

                    while (true)
                    {
                        string s = Console.ReadLine();
                        if (s.ToLower().Equals("exit"))
                        {
                            simpleSenderReceiver.Close();
                            System.Environment.Exit(0);
                        }
                        else
                            simpleSenderReceiver.SendMessage();
                    }
                }
                catch (Exception e)
                {
                    Console.WriteLine("Caught exception: " + e.Message);
                }
            }

            public SimpleSenderReceiver()
            {
                factory = MessagingFactory.CreateFromConnectionString(connectionString);
                sender = factory.CreateMessageSender(entityName);

                if (runReceiver)
                {
                    receiver = factory.CreateMessageReceiver(entityName);
                    messageListener = new MessageListener(receiver);
                    listenerThread = new Thread(messageListener.Listen);
                    listenerThread.Start();
                }
            }

            public void Close()
            {
                messageListener.RequestStop();
                listenerThread.Join();
                factory.Close();
            }

            private void SendMessage()
            {
                BrokeredMessage message = 
                    new BrokeredMessage("Test AMQP message from .NET");
                sender.Send(message);
                Console.WriteLine("Sent message with MessageID = " 
                    + message.MessageId);
            }

        }

        public class MessageListener
        {
            private MessageReceiver messageReceiver;
            public MessageListener(MessageReceiver receiver)
            {
                messageReceiver = receiver;
            }

            public void Listen()
            {
                while (!_shouldStop)
                {
                    try
                    {
                        BrokeredMessage message = 
                            messageReceiver.Receive(new TimeSpan(0, 0, 10));
                        if (message != null)
                        {
                            Console.WriteLine("Received message with MessageID = " + 
                                message.MessageId);
                            message.Complete();
                        }
                    }
                    catch (Exception e)
                    {
                        Console.WriteLine("Caught exception: " + e.Message);
                        break;
                    }
                }
            }

            public void RequestStop()
            {
                _shouldStop = true;
            }

            private volatile bool _shouldStop;
        }
    }

### 运行应用程序

运行应用程序将产生以下形式的输出：

    > SimpleSenderReceiver.exe
    Press [enter] to send a message. Type 'exit' + [enter] to quit.

    Sent message with MessageID = fb7f5d3733704e4ba4bd55f759d9d7cf
    Received message with MessageID = fb7f5d3733704e4ba4bd55f759d9d7cf

    Sent message with MessageID = d00e2e088f06416da7956b58310f7a06
    Received message with MessageID = d00e2e088f06416da7956b58310f7a06

    Received message with MessageID = f27f79ec124548c196fd0db8544bca49
    Sent message with MessageID = f27f79ec124548c196fd0db8544bca49
    exit

## <span class="short-header">跨平台消息传送</span>在 JMS 和 .NET 之间实现跨平台消息传送

本指南说明了如何使用 .NET 将消息发送到 Service Bus 以及如何使用 .NET 接收这些消息。但是，AMQP 1.0 的关键优势之一是它支持通过以不同语言编写的组件生成应用程序，从而能够可靠和完全无损地交换消息。

使用上面的 .NET 应用程序示例，以及从随附的[如何将 Java 消息服务 (JMS) API 用于 Service Bus 和 AMQP 1.0][如何将 Java 消息服务 (JMS) API 用于 Service Bus 和 AMQP 1.0] 指南中选取的类似 Java 应用程序，可以在 .NET 和 Java 之间交换消息。

有关使用 Service Bus 和 AMQP 1.0 跨平台消息传送的详情的详细信息，请参阅 [Service Bus AMQP 1.0 开发人员指南][Service Bus AMQP 1.0 开发人员指南]。

### JMS 到 .NET

演示 JMS 到 .NET 消息传送：

-   启动 .NET 示例应用程序而不使用任何命令行参数。
-   使用“sendonly”命令行参数启动 Java 示例应用程序。在此模式下，应用程序不会从队列接收消息，而只会发送消息。
-   在 Java 应用程序控制台中按 **Enter** 几次，这将导致消息发送。
-   这些消息由 .NET 应用程序接收。

#### 从 JMS 应用程序输出

    > java SimpleSenderReceiver sendonly
    Press [enter] to send a message. Type 'exit' + [enter] to quit.
    Sent message with JMSMessageID = ID:4364096528752411591
    Sent message with JMSMessageID = ID:459252991689389983
    Sent message with JMSMessageID = ID:1565011046230456854
    exit

#### 从 .NET 应用程序输出

    > SimpleSenderReceiver.exe  
    Press [enter] to send a message. Type 'exit' + [enter] to quit.
    Received message with MessageID = 4364096528752411591
    Received message with MessageID = 459252991689389983
    Received message with MessageID = 1565011046230456854
    exit

### .NET 到 JMS

演示 NET 到 JMS 消息传送：

-   使用“sendonly”命令行参数启动 .NET 示例应用程序。在此模式下，应用程序不会从队列接收消息，而只会发送消息。
-   启动 Java 应用程序示例而不使用任何命令行参数。
-   在 .NET 应用程序控制台中按 **Enter** 几次，这将导致消息发送。
-   这些消息由 Java 应用程序接收。

#### 从 .NET 应用程序输出

    > SimpleSenderReceiver.exe sendonly
    Press [enter] to send a message. Type 'exit' + [enter] to quit.
    Sent message with MessageID = d64e681a310a48a1ae0ce7b017bf1cf3  
    Sent message with MessageID = 98a39664995b4f74b32e2a0ecccc46bb
    Sent message with MessageID = acbca67f03c346de9b7893026f97ddeb
    exit

#### 从 JMS 应用程序输出

    > java SimpleSenderReceiver 
    Press [enter] to send a message. Type 'exit' + [enter] to quit.
    Received message with JMSMessageID = ID:d64e681a310a48a1ae0ce7b017bf1cf3
    Received message with JMSMessageID = ID:98a39664995b4f74b32e2a0ecccc46bb
    Received message with JMSMessageID = ID:acbca67f03c346de9b7893026f97ddeb
    exit

## <span class="short-header">不受支持的功能</span>不受支持的功能和限制

在使用 AMQP 时，.NET Service Bus API 的以下功能目前不受支持：

-   事务
-   通过传输目标发送
-   按消息序列号接收
-   消息和会话浏览
-   会话状态
-   基于批处理的 API
-   向外扩展接收
-   订阅规则的运行时操作
-   会话锁定续订
-   一些微小的行为差异

有关详细信息，请参阅 [Service Bus AMQP 1.0 开发人员指南][Service Bus AMQP 1.0 开发人员指南]。此主题包括不受支持的 API 的详细列表。

## <span class="short-header">摘要</span>摘要

本操作方法指南介绍了如何使用 AMQP 1.0 和 Service Bus .NET API 通过 .NET 访问 Service Bus 中转消息传送功能（队列和发布/订阅主题）。

也可以通过其他语言（包括 Java、C、Python 和 PHP）使用 Service Bus AMQP 1.0。使用这些语言构建的组件可以使用 Service Bus 中的 AMQP 1.0 可靠且完全无损地交换消息。有关详细信息，请参阅 [Service Bus AMQP 1.0 开发人员指南][Service Bus AMQP 1.0 开发人员指南]。

## <span class="short-header">更多信息</span>更多信息

-   [Azure Service Bus 中的 AMQP 1.0 支持][Azure Service Bus 中的 AMQP 1.0 支持]
-   [如何将 Java 消息服务 (JMS) API 用于 Service Bus 和 AMQP 1.0][如何将 Java 消息服务 (JMS) API 用于 Service Bus 和 AMQP 1.0]
-   [Service Bus AMQP 1.0 开发人员指南][Service Bus AMQP 1.0 开发人员指南]
-   [如何使用 Service Bus 队列][如何使用 Service Bus 队列]

  [Azure 管理门户]: http://manage.windowsazure.cn
  [如何使用 Service Bus 队列（可能为英文页面）]: https://www.windowsazure.com/zh-cn/develop/net/how-to-guides/service-bus-queues/
  []: http://nuget.org/packages/WindowsAzure.ServiceBus/
  [Service Bus AMQP 1.0 开发人员指南]: http://msdn.microsoft.com/zh-cn/library/windowsazure/jj841071.aspx
  [如何使用 Service Bus 队列]: http://www.windowsazure.com/zh-cn/develop/net/how-to-guides/service-bus-queues/
  [如何将 Java 消息服务 (JMS) API 用于 Service Bus 和 AMQP 1.0]: http://aka.ms/ll1fm3
  [Azure Service Bus 中的 AMQP 1.0 支持]: http://aka.ms/pgr3dp
