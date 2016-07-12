<properties 
	pageTitle="如何将 AMQP 1.0 用于 .NET 服务总线 API | Azure" 
	description="了解如何将高级消息队列协议 (AMQP) 1.0 用于 Azure .NET 服务总线 API。" 
	services="service-bus" 
	documentationCenter=".net" 
	authors="sethmanheim" 
	manager="timlt" 
	editor=""/>

<tags 
	ms.service="service-bus" 
	ms.date="05/10/2016" 
	wacn.date="06/27/2016"/>

# 如何将 AMQP 1.0 与服务总线 .NET API 一起使用

高级消息队列协议 (AMQP) 1.0 是一个高效、可靠的线级消息传送协议，可用于构建可靠的跨平台消息传送应用程序。

在服务总线中支持 AMQP 1.0 意味着可以通过一系列使用有效的二进制协议的平台利用队列和发布/订阅中转消息传送功能。此外，你还可以生成由结合使用多个语言、框架和操作系统构建的组件组成的应用程序。

本文说明如何使用服务总线 .NET API 通过 .NET 应用程序来使用服务总线中转消息传送功能（队列和发布/订阅主题）。有一个[配套文章](/documentation/articles/service-bus-java-how-to-use-jms-api-amqp/)，该文说明如何使用标准 Java 消息服务 (JMS) API 执行相同的操作。使用 AMQP 1.0，可以同时使用以下两个指南来了解跨平台消息。

## 服务总线入门

本文假定你已拥有包含名为“queue1”的队列的服务总线命名空间。 如果没有，则可以使用 [Azure 管理门户](http://manage.windowsazure.cn)创建命名空间和队列。有关如何创建服务总线命名空间和队列的详细信息，请参阅[如何使用服务总线队列](/documentation/articles/service-bus-dotnet-how-to-use-queues/)。

## 下载服务总线 SDK

AMQP 1.0 支持在服务总线 SDK 2.1 版或更高版本中提供。可从以下位置的 NuGet 中下载最新的 SDK：[http://nuget.org/packages/WindowsAzure.ServiceBus/](http://nuget.org/packages/WindowsAzure.ServiceBus/)。

## 编写 .NET 应用程序

默认情况下，服务总线 .NET 客户端库使用基于 SOAP 的专用协议与服务总线服务通信。若要使用 AMQP 1.0 而非默认协议，需要服务总线连接字符串上的显式配置，如下一节所述。除了此更改之外，在使用 AMQP 1.0 时应用程序代码基本保持不变。

在当前版本中，有一些在使用 AMQP 时不受支持的 API 功能。这些不受支持的功能将在后面的[不支持的功能和限制](#unsupported-features-and-restrictions)部分中列出。在使用 AMQP 时，一些高级配置设置还具有不同的含义。在本文中没有使用这些设置，但在[服务总线 AMQP 概述](/documentation/articles/service-bus-amqp-dotnet/#unsupported-features-restrictions-and-behavioral-differences)中提供了更多详细信息。

### 通过 App.config 进行配置

应用程序使用 App.config 配置文件存储设置是一个建议的做法。对于服务总线应用程序，你可以使用 App.config 来存储服务总线 **ConnectionString**。此示例应用程序还使用 App.config 来存储它使用的服务总线消息传送实体的名称。

下面显示了 App.config 文件示例：

```
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
  	<appSettings>
	    <add key="Microsoft.ServiceBus.ConnectionString"
       	     value="Endpoint=sb://[namespace].servicebus.chinacloudapi.cn;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=[SAS key];TransportType=Amqp" />
	    	<add key="EntityName" value="queue1" />
	</appSettings>
</configuration>
```

### 配置服务总线连接字符串

**Microsoft.ServiceBus.ConnectionString** 设置的值是用于配置服务总线连接的服务总线连接字符串。其格式如下所示：

```
Endpoint=sb://[namespace].servicebus.chinacloudapi.cn;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=[SAS key];TransportType=Amqp
```

其中 `[namespace]` 和 `[SAS key]` 是从 [Azure 管理门户][]获取的。有关详细信息，请参阅 [如何使用服务总线队列][]。

在使用 AMQP 时，在连接字符串后面追加了 `;TransportType=Amqp`，以通知客户端库使用 AMQP 1.0 连接到服务总线。

### 配置实体名称

此应用程序示例使用 App.config 文件的 **appSettings** 节中的 `EntityName` 设置来配置应用程序与其交换消息的队列的名称。

### 使用服务总线队列的简单 .NET 应用程序

以下示例向服务总线队列发送消息以及从中接收消息。

```
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
            BrokeredMessage message = new BrokeredMessage("Test AMQP message from .NET");
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
```

### 运行应用程序

运行应用程序将产生以下形式的输出：

```
> SimpleSenderReceiver.exe
Press [enter] to send a message. Type 'exit' + [enter] to quit.
	
Sent message with MessageID = fb7f5d3733704e4ba4bd55f759d9d7cf
Received message with MessageID = fb7f5d3733704e4ba4bd55f759d9d7cf
	
Sent message with MessageID = d00e2e088f06416da7956b58310f7a06
Received message with MessageID = d00e2e088f06416da7956b58310f7a06
	
Received message with MessageID = f27f79ec124548c196fd0db8544bca49
Sent message with MessageID = f27f79ec124548c196fd0db8544bca49
exit
```

## JMS 和 .NET 之间的跨平台消息传送

本主题说明了如何使用 .NET 将消息发送到服务总线以及如何使用 .NET 接收这些消息。但是，AMQP 1.0 的关键优势之一是它支持通过以不同语言编写的组件生成应用程序，从而能够可靠和完全无损地交换消息。

使用上面的 .NET 应用程序示例，以及从随附的[如何将 Java 消息服务 (JMS) API 用于服务总线和 AMQP 1.0](/documentation/articles/service-bus-java-how-to-use-jms-api-amqp/) 指南中选取的类似 Java 应用程序，可以在 .NET 和 Java 之间交换消息。

有关使用服务总线和 AMQP 1.0 的跨平台消息传送的详细信息，请参阅[服务总线 AMQP 1.0 概述](/documentation/articles/service-bus-amqp-overview/)。

### JMS 到 .NET

演示 JMS 到 .NET 消息传送：

* 启动 .NET 示例应用程序而不使用任何命令行参数。
* 使用“sendonly”命令行参数启动 Java 示例应用程序。在此模式下，应用程序不会从队列接收消息，而只会发送消息。
* 在 Java 应用程序控制台中按 **Enter** 多次，这将导致消息发送。
* 这些消息由 .NET 应用程序接收。

### 从 JMS 应用程序输出

```
> java SimpleSenderReceiver sendonly
Press [enter] to send a message. Type 'exit' + [enter] to quit.
Sent message with JMSMessageID = ID:4364096528752411591
Sent message with JMSMessageID = ID:459252991689389983
Sent message with JMSMessageID = ID:1565011046230456854
exit
```

### 从 .NET 应用程序输出

```
> SimpleSenderReceiver.exe	
Press [enter] to send a message. Type 'exit' + [enter] to quit.
Received message with MessageID = 4364096528752411591
Received message with MessageID = 459252991689389983
Received message with MessageID = 1565011046230456854
exit
```

## .NET 到 JMS

演示 NET 到 JMS 消息传送：

* 使用“sendonly”命令行参数启动 .NET 示例应用程序。在此模式下，应用程序不会从队列接收消息，而只会发送消息。
* 启动 Java 应用程序示例而不使用任何命令行参数。
* 在 .NET 应用程序控制台中按 **Enter** 多次，这将导致消息发送。
* 这些消息由 Java 应用程序接收。

#### 从 .NET 应用程序输出

```
> SimpleSenderReceiver.exe sendonly
Press [enter] to send a message. Type 'exit' + [enter] to quit.
Sent message with MessageID = d64e681a310a48a1ae0ce7b017bf1cf3	
Sent message with MessageID = 98a39664995b4f74b32e2a0ecccc46bb
Sent message with MessageID = acbca67f03c346de9b7893026f97ddeb
exit
```

#### 从 JMS 应用程序输出

```
> java SimpleSenderReceiver	
Press [enter] to send a message. Type 'exit' + [enter] to quit.
Received message with JMSMessageID = ID:d64e681a310a48a1ae0ce7b017bf1cf3
Received message with JMSMessageID = ID:98a39664995b4f74b32e2a0ecccc46bb
Received message with JMSMessageID = ID:acbca67f03c346de9b7893026f97ddeb
exit
```

## <a name="unsupported-features-and-restrictions"></a>不支持的功能和限制

在使用 AMQP 时，.NET 服务总线 API 的以下功能目前不受支持：

* 事务
* 通过传输目标发送
* 按消息序列号接收
* 消息和会话浏览
* 会话状态
* 基于批处理的 API
* 向外扩展接收
* 订阅规则的运行时操作
* 会话锁定续订
* 一些微小的行为差异

有关详细信息，请参阅[服务总线 AMQP 概述](/documentation/articles/service-bus-amqp-dotnet/)。本文包括不受支持的 API 的详细列表。

## 摘要

本文介绍了如何使用 AMQP 1.0 和服务总线 .NET API 通过 .NET 访问服务总线中转消息传送功能（队列和发布/订阅主题）。

也可以通过其他语言（包括 Java、C、Python 和 PHP）使用服务总线 AMQP 1.0。使用这些语言构建的组件可以使用服务总线中的 AMQP 1.0 可靠且完全无损地交换消息。有关详细信息，请参阅[服务总线 AMQP 概述](/documentation/articles/service-bus-amqp-dotnet/)。

## 后续步骤

现在，你已阅读服务总线和 AMQP 与 .NET 的概述，请参阅以下链接了解详细信息。

* [Azure 服务总线中的 AMQP 1.0 支持](/documentation/articles/service-bus-amqp-overview/)
* [如何将 Java 消息服务 (JMS) API 用于服务总线 和 AMQP 1.0](/documentation/articles/service-bus-java-how-to-use-jms-api-amqp/)
* [如何使用服务总线队列](/documentation/articles/service-bus-dotnet-how-to-use-queues/)
 
[Azure 管理门户]: http://manage.windowsazure.cn
[如何使用服务总线队列]: /documentation/articles/service-bus-dotnet-how-to-use-queues/

<!---HONumber=Mooncake_0104_2016-->