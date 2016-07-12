<properties 
	pageTitle="将 AMQP 1.0 用于 Java 服务总线 API | Microsoft Azure" 
	description="了解如何将 Java 消息服务 (JMS) 用于 Azure 服务总线和高级消息队列"
	services="service-bus"
	documentationCenter="java"
	authors="sethmanheim"  
	manager="timlt" 
	editor=""/>

<tags 
	ms.service="service-bus" 
	ms.date="03/09/2016" 
	wacn.date="01/14/2016"/>


# 如何将 Java 消息服务 (JMS) API 用于服务总线和 AMQP 1.0

高级消息队列协议 (AMQP) 1.0 是一个高效、可靠的线级消息传送协议，可用于构建可靠的跨平台消息传送应用程序。AMQP 1.0 支持已在 2012 年 10 月添加到 Azure 服务总线中，并已在 2013 年 5 月转为正式版 (GA)。

添加 AMQP 1.0 意味着现在可以通过一系列使用高效二进制协议的平台综合利用 Service Bus 的队列及发布/订阅中转消息传送功能。此外，您还可以生成由结合使用多个语言、框架和操作系统构建的组件组成的应用程序。

本操作方法指南说明了如何从使用常用的 Java 消息服务 (JMS) API 标准的 Java 应用程序中使用 Service Bus 中转消息传送功能（队列和发布/订阅主题）。

## 服务总线入门

此指南假定你已具有包含名为“queue1”的队列的 Service Bus 命名空间。 如果没有，则可以使用 [Azure 经典门户](http://manage.windowsazure.cn)创建命名空间和队列。有关如何创建服务总线命名空间和队列的详细信息，请参阅[如何使用服务总线队列](/documentation/articles/service-bus-dotnet-how-to-use-queues/)。

### 下载 AMQP 1.0 JMS 客户端库

有关从哪里下载 Apache Qpid JMS AMQP 1.0 客户端库的最新版本的信息，请访问 [http://people.apache.org/~rgodfrey/qpid-java-amqp-1-0-client-jms.html](http://people.apache.org/~rgodfrey/qpid-java-amqp-1-0-client-jms.html)。

使用 Service Bus 构建和运行 JMS 应用程序时必须将以下 4 个 JAR 文件从 Apache Qpid JMS AMQP 1.0 分发存档添加到 Java CLASSPATH：

*    geronimo-jms\_1.1\_spec-1.0.jar
*    qpid-amqp-1-0-client-[version].jar
*    qpid-amqp-1-0-client-jms-[version].jar
*    qpid-amqp-1-0-common-[version].jar

## 为 Java 应用程序编码

### Java 命名和目录接口 (JNDI)

JMS 使用 Java 命名和目录接口 (JNDI) 创建逻辑名称和物理名称之间的分隔。将使用 JNDI 解析以下两种类型的 JMS 对象：ConnectionFactory 和 Destination。JNDI 使用一个提供程序模型，你可以在其中插入不同目录服务来处理名称解析任务。Apache Qpid JMS AMQP 1.0 库附带一个使用以下格式的属性文件配置的、基于属性文件的简单 JNDI 提供程序。

```

	# servicebus.properties – sample JNDI configuration
		
	# Register a ConnectionFactory in JNDI using the form:
	# connectionfactory.[jndi_name] = [ConnectionURL]
	connectionfactory.SBCF = amqps://[username]:[password]@[namespace].servicebus.chinacloudapi.cn
		
	# Register some queues in JNDI using the form
	# queue.[jndi_name] = [physical_name]
	# topic.[jndi_name] = [physical_name]
	queue.QUEUE = queue1
```

#### 配置 ConnectionFactory

用于在 Qpid 属性文件 JNDI 提供程序中定义 **ConnectionFactory** 的条目的格式如下：

```
connectionfactory.[jndi_name] = [ConnectionURL]
```

其中，**[jndi\_name]** 和 **[ConnectionURL]** 具有以下含义：

- **[jndi\_name]**：ConnectionFactory 的逻辑名称。这是将使用 JNDI IntialContext.lookup() 方法在 Java 应用程序中解析的名称。
- **[ConnectionURL]**：向 AMQP 中转站提供包含所需信息的 JMS 库的 URL。

**ConnectionURL** 的格式如下：

```
amqps://[username]:[password]@[namespace].servicebus.chinacloudapi.cn
```

其中，**[namespace]**、**[username]** 和 **[password]** 的含义如下：

- **[namespace]**：服务总线命名空间。
- **[username]**：服务总线颁发者名称。
- **[password]**：服务总线颁发者密钥的 URL 编码格式。

> [AZURE.NOTE]必须手动为密码进行 URL 编码。在 [http://www.w3schools.com/tags/ref\_urlencode.asp](http://www.w3schools.com/tags/ref_urlencode.asp) 上提供了一个有用的 URL 编码实用工具。

#### 配置目标

用于在 Qpid 属性文件 JNDI 提供程序中定义目标的项的格式如下：

```
queue.[jndi_name] = [physical_name]
```
或

```
topic.[jndi_name] = [physical_name]
```

其中，**[jndi\_name]** 和 **[physical\_name]** 的含义如下：

- **[jndi\_name]**：目标的逻辑名称。这是将使用 JNDI IntialContext.lookup() 方法在 Java 应用程序中解析的名称。
- **[physical\_name]**：应用程序向其发送或从该处接收消息的服务总线实体的名称。


> [AZURE.NOTE]在从 Service Bus 主题订阅中接收时，在 JNDI 中指定的物理名称应该是该主题的名称。在 JMS 应用程序代码中创建可持久订阅时提供该订阅名称。[服务总线 AMQP 1.0 开发人员指南](/documentation/articles/service-bus-amqp-dotnet/)提供了有关从 JMS 使用服务总线主题订阅的更多详细信息。

### 编写 JMS 应用程序

将 JMS 用于服务总线时不需要特殊的 API 或选项。但是，有一些限制，我们将在后面说明。与使用任何 JMS 应用程序一样，若要解析 **ConnectionFactory** 和目标，首先要做的事情是配置 JNDI 环境。

#### 配置 JNDI InitialContext

JNDI 环境是通过将配置信息的哈希表传入到 javax.naming.InitialContext 类的构造函数中来配置的。哈希表中的两个必需元素是初始上下文工厂的类名称和提供程序 URL。以下代码演示了如何配置 JNDI 环境以将基于 Qpid 属性文件的 JNDI 提供程序用于名为 **servicebus.properties** 的属性文件。

```
Hashtable<String, String> env = new Hashtable<String, String>(); 
env.put(Context.INITIAL_CONTEXT_FACTORY, "org.apache.qpid.amqp_1_0.jms.jndi.PropertiesFileInitialContextFactory"); 
env.put(Context.PROVIDER_URL, "servicebus.properties"); 
InitialContext context = new InitialContext(env);
``` 

### 使用服务总线队列的简单 JMS 应用程序

以下示例程序将 JMS TextMessages 发送到 JNDI 逻辑名称为 QUEUE 的 Service Bus 队列，然后接收返回的消息。

	// SimpleSenderReceiver.java
	
	import javax.jms.*;
	import javax.naming.Context;
	import javax.naming.InitialContext;
	import java.io.BufferedReader;
	import java.io.InputStreamReader;
	import java.util.Hashtable;
	import java.util.Random;
	
	public class SimpleSenderReceiver implements MessageListener {
	    private static boolean runReceiver = true;
	    private Connection connection;
	    private Session sendSession;
	    private Session receiveSession;
	    private MessageProducer sender;
	    private MessageConsumer receiver;
	    private static Random randomGenerator = new Random();
	
	    public SimpleSenderReceiver() throws Exception {
	        // Configure JNDI environment
	        Hashtable<String, String> env = new Hashtable<String, String>();
	        env.put(Context.INITIAL_CONTEXT_FACTORY, 
                    "org.apache.qpid.amqp_1_0.jms.jndi.PropertiesFileInitialContextFactory");
	        env.put(Context.PROVIDER_URL, "servicebus.properties");
	        Context context = new InitialContext(env);
	
	        // Lookup ConnectionFactory and Queue
	        ConnectionFactory cf = (ConnectionFactory) context.lookup("SBCF");
	        Destination queue = (Destination) context.lookup("QUEUE");
	
	        // Create Connection
	        connection = cf.createConnection();
	
	        // Create sender-side Session and MessageProducer
	        sendSession = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
	        sender = sendSession.createProducer(queue);
	
	        if (runReceiver) {
	            // Create receiver-side Session, MessageConsumer,and MessageListener
	            receiveSession = connection.createSession(false, Session.CLIENT_ACKNOWLEDGE);
	            receiver = receiveSession.createConsumer(queue);
	            receiver.setMessageListener(this);
	            connection.start();
	        }
	    }
	
	    public static void main(String[] args) {
	        try {
	
	            if ((args.length > 0) && args[0].equalsIgnoreCase("sendonly")) {
	                runReceiver = false;
	            }
	
	            SimpleSenderReceiver simpleSenderReceiver = new SimpleSenderReceiver();
	            System.out.println("Press [enter] to send a message. Type 'exit' + [enter] to quit.");
	            BufferedReader commandLine = new java.io.BufferedReader(new InputStreamReader(System.in));
	
	            while (true) {
	                String s = commandLine.readLine();
	                if (s.equalsIgnoreCase("exit")) {
	                    simpleSenderReceiver.close();
	                    System.exit(0);
	                } else {
	                    simpleSenderReceiver.sendMessage();
	                }
	            }
	        } catch (Exception e) {
	            e.printStackTrace();
	        }
	    }
	
	    private void sendMessage() throws JMSException {
	        TextMessage message = sendSession.createTextMessage();
	        message.setText("Test AMQP message from JMS");
	        long randomMessageID = randomGenerator.nextLong() >>>1;
	        message.setJMSMessageID("ID:" + randomMessageID);
	        sender.send(message);
	        System.out.println("Sent message with JMSMessageID = " + message.getJMSMessageID());
	    }
	
	    public void close() throws JMSException {
	        connection.close();
	    }
	
	    public void onMessage(Message message) {
	        try {
	            System.out.println("Received message with JMSMessageID = " + message.getJMSMessageID());
	            message.acknowledge();
	        } catch (Exception e) {
	            e.printStackTrace();
	        }
	    }
	}	

### 运行应用程序

运行应用程序将生成以下输出：

```
> java SimpleSenderReceiver
Press [enter] to send a message. Type 'exit' + [enter] to quit.
	
Sent message with JMSMessageID = ID:2867600614942270318
Received message with JMSMessageID = ID:2867600614942270318
	
Sent message with JMSMessageID = ID:7578408152750301483
Received message with JMSMessageID = ID:7578408152750301483
	
Sent message with JMSMessageID = ID:956102171969368961
Received message with JMSMessageID = ID:956102171969368961
exit
```

## JMS 和 .NET 之间的跨平台消息传送

本指南说明了如何使用 JMS 向 Service Bus 发送消息以及从 Service Bus 接收消息。但是，AMQP 1.0 的关键优势之一是它支持通过以不同语言编写的组件生成应用程序，从而能够可靠和完全无损地交换消息。

通过使用前面所述的示例 JMS 应用程序和从随附的指南[如何将 AMQP 1.0 用于 .NET 服务总线 .NET API](/documentation/articles/service-bus-dotnet-advanced-message-queuing/) 中选取的类似 .NET 应用程序，你可以在 .NET 和 Java 之间交换消息。

有关使用服务总线和 AMQP 1.0 的跨平台消息传送的详细信息，请参阅[服务总线 AMQP 1.0 开发人员指南](/documentation/articles/service-bus-amqp-dotnet/)。

### JMS 到 .NET

演示 JMS 到 .NET 消息传送：

* 启动 .NET 示例应用程序而不使用任何命令行参数。
* 使用“sendonly”命令行参数启动 Java 示例应用程序。在此模式下，应用程序不会从队列接收消息，而只会发送消息。
* 在 Java 应用程序控制台中按 **Enter** 多次，这将导致消息发送。
* 这些消息由 .NET 应用程序接收。

#### 从 JMS 应用程序输出

```
> java SimpleSenderReceiver sendonly
Press [enter] to send a message. Type 'exit' + [enter] to quit.
Sent message with JMSMessageID = ID:4364096528752411591
Sent message with JMSMessageID = ID:459252991689389983
Sent message with JMSMessageID = ID:1565011046230456854
exit
```

#### 从 .NET 应用程序输出

```

	> SimpleSenderReceiver.exe	
	Press [enter] to send a message. Type 'exit' + [enter] to quit.
	Received message with MessageID = 4364096528752411591
	Received message with MessageID = 459252991689389983
	Received message with MessageID = 1565011046230456854
	exit
```

### .NET 到 JMS

演示 NET 到 JMS 消息传送：

* 使用“sendonly”命令行参数启动 .NET 示例应用程序。在此模式下，应用程序不会从队列接收消息，而只会发送消息。
* 启动 Java 示例应用程序而不使用任何命令行参数。
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

## 不受支持的功能和限制

在将 JMS over AMQP 1.0 用于 Service Bus 时存在以下限制，即：

* 每个**会话**只允许一个 **MessageProducer** 或 **MessageConsumer**。如果你需要在应用程序中创建多个 **MessageProducers** 或 **MessageConsumers**，请分别为它们创建专用**会话**。
* 当前不支持易失性主题订阅。
* 当前不支持 **MessageSelectors**。
* 当前不支持临时目标（即 **TemporaryQueue** 和 **TemporaryTopic**），以及使用这些目标的 **QueueRequestor** 和 **TopicRequestor** API。
* 不支持事务处理会话和分布式事务。

## 摘要

本操作方法指南演示了如何通过使用常用 JMS API 和 AMQP 1.0 通过 Java 使用 Service Bus 中转消息传送功能（队列和发布/订阅主题）。

也可以通过其他语言（包括 .NET、C、Python 和 PHP）使用 Service Bus AMQP 1.0。使用这些不同语言构建的组件可以使用服务总线中的 AMQP 1.0 支持可靠且完全无损地交换消息。有关详细信息，请参阅[服务总线 AMQP 1.0 开发人员指南](/documentation/articles/service-bus-amqp-dotnet/)。

## 后续步骤

* [Azure Service Bus 中的 AMQP 1.0 支持](/documentation/articles/service-bus-amqp-overview/)
* [如何将 AMQP 1.0 与服务总线 .NET API 一起使用](/documentation/articles/service-bus-dotnet-advanced-message-queuing/)
* [服务总线 AMQP 1.0 开发人员指南](/documentation/articles/service-bus-amqp-dotnet/)
* [如何使用 Service Bus 队列](/documentation/articles/service-bus-dotnet-how-to-use-queues/)

<!---HONumber=Mooncake_0104_2016-->