<properties 
   pageTitle="服务总线 和 Java 与 AMQP 1.0 | Azure"
   description="使用 AMQP 通过 Java 使用服务总线。"
   services="service-bus"
   documentationCenter="na"
   authors="sethmanheim"
   manager="timlt"
   editor="tysonn" /> 
<tags 
   ms.service="service-bus"
   ms.date="05/06/2016"
   wacn.date="06/27/2016" />

# 使用 AMQP 1.0 通过 Java 使用服务总线

[AZURE.INCLUDE [service-bus-selector-amqp](../includes/service-bus-selector-amqp.md)]

Java 消息服务 (JMS) 是一种标准 API，用于处理 Java 平台上面向消息的中间件。Azure 服务总线已使用 Apache Qpid 项目开发的基于 AMQP 1.0 的 JMS 客户端库进行测试。此库支持完整的 JMS 1.1 API，并可用于任何 AMQP 1.0 兼容的消息服务。[Windows Server 服务总线](https://msdn.microsoft.com/zh-cn/library/dn282144.aspx)（本地服务总线）中也支持此方案。有关详细信息，请参阅[适用于 Windows Server 的服务总线中的 AMQP][]。

## 下载 Apache Qpid AMQP 1.0 JMS 客户端库

有关下载 Apache Qpid JMS AMQP 1.0 客户端库的最新版本的信息，请访问 [http://people.apache.org/~rgodfrey/qpid-java-amqp-1-0-client-jms.html](http://people.apache.org/~rgodfrey/qpid-java-amqp-1-0-client-jms.html)。

使用服务总线构建和运行 JMS 应用程序时必须将以下 4 个 JAR 文件从 Apache Qpid JMS AMQP 1.0 分发存档添加到 Java CLASSPATH：

-   geronimo-jms\_1.1\_spec-[version].jar

-   qpid-amqp-1-0-client-[version].jar

-   qpid-amqp-1-0-client-jms-[version].jar

-   qpid-amqp-1-0-common-[version].jar

## 通过 JMS 使用服务总线队列、主题和订阅

### Java 命名和目录接口 (JNDI)

JMS 使用 Java 命名和目录接口 (JNDI) 创建逻辑名称和物理名称之间的分隔。将使用 JNDI 解析以下两种类型的 JMS 对象：**ConnectionFactory** 和 **Destination**。JNDI 使用一个提供程序模型，你可以在其中插入不同目录服务来处理名称解析任务。Apache Qpid JMS AMQP 1.0 库附带一个使用文本文件配置的、基于属性文件的简单 JNDI 提供程序。

Qpid 属性文件 JNDI 提供程序是使用以下格式的属性文件配置的：

```
# servicebus.properties – sample JNDI configuration

# Register a ConnectionFactory in JNDI using the form:
# connectionfactory.[jndi_name] = [ConnectionURL]
connectionfactory.SBCONNECTIONFACTORY = amqps://[username]:[password]@[namespace].servicebus.chinacloudapi.cn

# Register some queues in JNDI using the form
# queue.[jndi_name] = [physical_name]
# topic.[jndi_name] = [physical_name]
topic.TOPIC = topic1
queue.QUEUE = queue1
```

#### 配置连接工厂

用于在 Qpid 属性文件 JNDI 提供程序中定义 **ConnectionFactory** 的条目的格式如下：

```
connectionfactory.[jndi_name] = [ConnectionURL]
```

其中 `[jndi\_name]` 和 `[ConnectionURL]` 具有以下含义：

| 名称 | 含义 | | | | |
|-----------------|--------------------------------------------------------------------------------------------------------------------------------------------|---|---|---|---|
| `[jndi\_name]` | 连接工厂的逻辑名称。通过使用 JNDI `IntialContext.lookup()` 方法在 Java 应用程序中解析此名称。 | | | | |
| `[ConnectionURL]` | 用于向 AMQP 代理提供包含所需信息的 JMS 库的 URL。 | | | | |

连接 URL 的格式如下：

```
amqps://[username]:[password]@[namespace].servicebus.chinacloudapi.cn
```

其中 `[namespace]`、`[username]` 和 `[password]` 具有以下含义：

| 名称 | 含义 | | | | |
|---------------|--------------------------------------------------------------------------------|---|---|---|---|
| `[namespace]` | 从 [Azure 经典门户][]获取的服务总线命名空间。 | | | | |
| `[username]` | 从 [Azure 经典门户][]获取的服务总线颁发者名称。 | | | | |
| `[password]` | 从 [Azure 经典门户][]获取的 URL 编码形式的服务总线颁发者密钥。 | | | | |

> [AZURE.NOTE]必须手动为密码进行 URL 编码。在 [http://www.w3schools.com/tags/ref\_urlencode.asp](http://www.w3schools.com/tags/ref_urlencode.asp) 上提供了一个有用的 URL 编码实用工具。

例如，如果从门户获得的信息如下所示：

| Namespace： | test.servicebus.chinacloudapi.cn |
|--------------|----------------------------------------------|
| 颁发者名称： | owner |
| 颁发者密钥： | abcdefg |

那么，为了定义名为 `SBCONNECTIONFACTORY` 的 **ConnectionFactory** 对象，配置字符串将如下所示：

```
connectionfactory.SBCONNECTIONFACTORY = amqps://owner:abcdefg@test.servicebus.chinacloudapi.cn
```

#### 配置目标

用于在 Qpid 属性文件 JNDI 提供程序中定义目标的条目的格式如下：

```
queue.[jndi_name] = [physical_name]
topic.[jndi_name] = [physical_name]
```

其中 `[jndi\_name]` 和 `[physical\_name]` 具有以下含义：

| 名称 | 含义 |
|-------------------|------------------------------------------------------------------------------------------------------------------------------------------|
| `[jndi\_name]` | 目标的逻辑名称。通过使用 JNDI `IntialContext.lookup()` 方法在 Java 应用程序中解析此名称。 |
| `[physical\name]` | 应用程序在其中发送或接收消息的服务总线实体的名称。 |

注意以下事项：

- `[physical\name]` 值可以是服务总线队列或主题。
- 在从 Service Bus 主题订阅中接收时，在 JNDI 中指定的物理名称应该是该主题的名称。在 JMS 应用程序代码中创建可持久订阅时提供该订阅名称。
- 还可以将服务总线主题订阅视为一个 JMS 队列。此方法具有以下几个优点：可以针对队列和主题订阅使用同一接收者代码，并且所有地址信息（主题和订阅名称）都在属性文件中外部化。
- 若要将服务总线主题订阅视为一个 JMS 队列，属性文件中的条目应采用以下形式：`queue.[jndi_name] = [topic_name]/Subscriptions/[subscription_name]`。

若要定义映射到名为“topic1”的服务总线主题的名为“TOPIC”的逻辑 JMS 目标，属性文件中的条目应如下所示：


    topic.TOPIC = topic1


### 使用 JMS 发送消息

以下代码演示如何向服务总线主题发送消息。假设在上一部分中所述的 **servicebus.properties** 配置文件中定义了 `SBCONNECTIONFACTORY` 和 `TOPIC`。


    Hashtable<String, String> env = new Hashtable<String, String>(); 
    env.put(Context.INITIAL_CONTEXT_FACTORY, 
            "org.apache.qpid.amqp_1_0.jms.jndi.PropertiesFileInitialContextFactory"); 
    env.put(Context.PROVIDER_URL, "servicebus.properties"); 
     
    InitialContext context = new InitialContext(env); 
     
    ConnectionFactory cf = (ConnectionFactory) context.lookup("SBCONNECTIONFACTORY");
    Topic topic = (Topic) context.lookup("TOPIC");
    Connection connection = cf.createConnection();
    Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
    MessageProducer producer = session.createProducer(topic);
    TextMessage message = session.createTextMessage("This is a text string"); 
    producer.send(message);


### 使用 JMS 接收消息

以下代码演示如何从服务总线主题订阅接收消息。`how`假设在上一部分中所述的 **servicebus.properties** 配置文件中定义了 `SBCONNECTIONFACTORY` 和 TOPIC。它还假定订阅名称是 `subscription1`。


    Hashtable<String, String> env = new Hashtable<String, String>(); 
    env.put(Context.INITIAL_CONTEXT_FACTORY, 
            "org.apache.qpid.amqp_1_0.jms.jndi.PropertiesFileInitialContextFactory"); 
    env.put(Context.PROVIDER_URL, "servicebus.properties"); 
     
    InitialContext context = new InitialContext(env);
    
    ConnectionFactory cf = (ConnectionFactory) context.lookup("SBCONNECTIONFACTORY");
    Topic topic = (Topic) context.lookup("TOPIC");
    Connection connection = cf.createConnection();
    Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
    TopicSubscriber subscriber = session.createDurableSubscriber(topic, "subscription1");
    connection.start();
    Message message = messageConsumer.receive();


### 用于构建可靠的应用程序的准则

JMS 规范定义了应如何编写 API 方法和应用程序代码的异常约定来处理此类异常。下面是有关异常处理需要注意的其他一些要点：

-   使用 **connection.setExceptionListener** 向 JMS 连接注册 **ExceptionListener**。这允许以异步方式向客户端通知问题。此通知对于仅使用消息的连接特别重要，因为客户端没有其他方法可以获知其连接已失败。如果底层 AMQP 连接、会话或链接有问题，将调用 **ExceptionListener**。在此情况下，应用程序应从零开始重新创建 **JMS Connection**、**Session**、**MessageProducer** 和 **MessageConsumer** 对象。

-   若要验证是否已从 **MessageProducer** 将一条消息成功发送到服务总线实体，请确保已为应用程序配置 **qpid.sync\_publish** 系统属性集。可以通过在启动应用程序时在命令行上设置 **-Dqpid.sync\_publish=true** Java VM 选项启动程序来完成此操作。设置此选项可将库配置为不从发送调用返回，直到收到该消息已被服务总线接受的确认为止。如果在发送操作期间出现问题，则将引发 **JMSException**。有两个可能的原因：
	1. 如果问题是由于服务总线拒绝所发送的特定消息所致，则将引发 **MessageRejectedException** 异常。此错误是暂时的，或者由于消息出现某些问题所致。建议的操作过程是进行多次尝试，以便使用一些后退逻辑重试该操作。如果问题仍然存在，则应使用本地记录的错误放弃该消息。在这种情况下，无需重新创建 **JMS Connection**、**Session** 或 **MessageProducer** 对象。 
	2. 如果问题是由于服务总线关闭 AMQP 链接所致，则将引发 **InvalidDestinationException** 异常。这可能是由于暂时性问题或由于消息实体被删除所致。在这两种情况中的任一情况下，均应重新创建 **JMS Connection**、**Session** 和 **MessageProducer** 对象。如果错误条件是暂时的，则此操作最终将会成功。如果实体已被删除，则失败将是永久的。

## 在 .NET 和 JMS 之间进行消息传递

### 消息正文

JMS 定义了五种不同的消息类型：**BytesMessage**、**MapMessage**、**ObjectMessage**、**StreamMessage** 和 **TextMessage**。服务总线 .NET API 具有单一消息类型 [BrokeredMessage][]。

#### JMS 到服务总线 .NET API

以下各节演示如何通过 .NET 使用每种 JMS 消息类型的消息。尚未包括 **ObjectMessage** 示例，因为 **ObjectMessage** 的正文包含使用 Java 编程语言的可序列化对象，而这是 .NET 应用程序所无法解释的。

##### BytesMessage

以下代码演示如何通过服务总线 .NET API 使用 **BytesMessage** 对象的正文。


    Stream stream = message.GetBody<Stream>();
    int streamLength = (int)stream.Length;
    
    byte[] byteArray = new byte[streamLength];
    stream.Read(byteArray, 0, streamLength);
    
    Console.WriteLine("Length = " + streamLength);
    for (int i = 0; i < stream.Length; i++)
    {
      Console.Write("[" + (sbyte) byteArray[i] + "]");
    }


##### MapMessage

以下代码演示如何通过服务总线 .NET API 使用 **MapMessage** 对象的正文。此代码循环访问映射的元素，并显示每个元素的名称和值。


    Dictionary<String, Object> dictionary = message.GetBody<Dictionary<String, Object>>();
    
    foreach (String mapItemName in dictionary.Keys)
    {
      Object mapItemValue = null;
      if (dictionary.TryGetValue(mapItemName, out mapItemValue))
      {
        Console.WriteLine(mapItemName + ":" + mapItemValue);
      }
    }


##### StreamMessage

以下代码演示如何通过服务总线 .NET API 使用 **StreamMessage** 对象的正文。此代码将列出流中的每一项及其类型。


    List<Object> list = message.GetBody<List<Object>>();
    
    foreach (Object item in list)
    {
      Console.WriteLine(item + " (" + item.GetType() + ")");
    }


##### TextMessage

以下代码演示如何通过服务总线 .NET API 使用 **TextMessage** 对象的正文。此代码将显示消息的正文中包含的文本字符串。


    Console.WriteLine("Text: " + message.GetBody<String>());


#### 服务总线 .NET API 到 JMS

以下各节说明 .NET 应用程序如何创建在 JMS 中接收具有每种不同的 JMS 消息类型的消息。尚未包括 **ObjectMessage** 示例，因为 **ObjectMessage** 的正文包含使用 Java 编程语言的可序列化对象，而这是 .NET 应用程序所无法解释的。

##### BytesMessage

以下代码演示如何在 .NET 中创建由 JMS 客户端接收作为 **BytesMessage** 的 [BrokeredMessage][] 对象。


    byte[] bytes = { 33, 12, 45, 33, 12, 45, 33, 12, 45, 33, 12, 45 };
    message = new BrokeredMessage(bytes);


##### StreamMessage

以下代码演示如何在 .NET 中创建由 JMS 客户端接收作为 **StreamMessage** 的 [BrokeredMessage][] 对象。


    List<Object> list = new List<Object>();
    list.Add("String 1");
    list.Add("String 2");
    list.Add("String 3");
    list.Add((double)3.14159);
    message = new BrokeredMessage(list);


##### TextMessage

以下代码演示如何通过服务总线 .NET API 使用 **TextMessage** 的正文。此代码将显示消息的正文中包含的文本字符串。


    message = new BrokeredMessage("this is a text string");


### 应用程序属性

####JMS 到服务总线 .NET API

JMS 消息支持以下类型的应用程序属性：**boolean**、**byte**、**short**、**int**、**long**、**float**、**double** 和 **String**。以下 Java 代码显示如何使用上述每种属性类型在消息上设置属性。


    message.setBooleanProperty("TestBoolean", true); 
    message.setByteProperty("TestByte", (byte) 33); 
    message.setDoubleProperty("TestDouble", 3.14159D); 
    message.setFloatProperty("TestFloat", 3.13159F); 
    message.setIntProperty("TestInt", 100); 
    message.setStringProperty("TestString", "Service Bus");


在服务总线 .NET API 中，在 [BrokeredMessage][] 的 **Properties** 集合中携带消息应用程序属性。以下代码演示如何读取从 JMS 客户端收到的消息的应用程序属性。


    if (message.Properties.Keys.Count > 0)
    {
      foreach (string name in message.Properties.Keys)
      {
        Object value = message.Properties[name];
        Console.WriteLine(name + ": " + value + " (" + value.GetType() + ")" );
      }
      Console.WriteLine();
    }


下表显示如何将 JMS 属性类型映射到 .NET 属性类型。

| JMS 属性类型 | .NET 属性类型 |
|-------------------|--------------------|
| Byte | sbyte |
| Integer | int |
| Float | float |
| Double | double |
| Boolean | bool |
| String | string |

[BrokeredMessage][] 类型支持以下类型的应用程序属性：**byte**、**sbyte**、**char**、**short**、**ushort**、**int**、**uint**、**long**、**ulong**、**float**、**double**、**decimal**、**bool**、**Guid**、**string**、**Uri**、**DateTime**、**DateTimeOffset** 和 **TimeSpan**。以下 .NET 代码显示如何使用上述每种属性类型在 [BrokeredMessage][] 对象上设置属性。


    message.Properties["TestByte"] = (byte)128;
    message.Properties["TestSbyte"] = (sbyte)-22;
    message.Properties["TestChar"] = (char) 'X';
    message.Properties["TestShort"] = (short)-12345;
    message.Properties["TestUshort"] = (ushort)12345;
    message.Properties["TestInt"] = (int)-100;
    message.Properties["TestUint"] = (uint)100;
    message.Properties["TestLong"] = (long)-12345;
    message.Properties["TestUlong"] = (ulong)12345;
    message.Properties["TestFloat"] = (float)3.14159;
    message.Properties["TestDouble"] = (double)3.14159;
    message.Properties["TestDecimal"] = (decimal)3.14159;
    message.Properties["TestBoolean"] = true;
    message.Properties["TestGuid"] = Guid.NewGuid();
    message.Properties["TestString"] = "Service Bus";
    message.Properties["TestUri"] = new Uri("http://www.bing.com");
    message.Properties["TestDateTime"] = DateTime.Now;
    message.Properties["TestDateTimeOffSet"] = DateTimeOffset.Now;
    message.Properties["TestTimeSpan"] = TimeSpan.FromMinutes(60);


以下 Java 代码演示如何读取从服务总线 .NET 客户端收到的消息的应用程序属性。


    Enumeration propertyNames = message.getPropertyNames(); 
    while (propertyNames.hasMoreElements()) 
    { 
      String name = (String) propertyNames.nextElement(); 
      Object value = message.getObjectProperty(name); 
      System.out.println(name + ": " + value + " (" + value.getClass() + ")"); 
    }

下表显示如何将 .NET 属性类型映射到 JMS 属性类型。

| .NET 属性类型 | JMS 属性类型 | 说明 |
|--------------------|-------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| byte | UnsignedByte | - |
| sbyte | Byte | - |
| char | Character | - |
| short | Short | - |
| ushort | UnsignedShort | - |
| int | Integer | - |
| uint | UnsignedInteger | - |
| long | Long | - |
| ulong | UnsignedLong | - |
| float | Float | - |
| double | Double | - |
| decimal | BigDecimal | - |
| bool | Boolean | - |
| Guid | UUID | - |
| string | String | - |
| DateTime | Date | - |
| DateTimeOffset | DescribedType | 映射到 AMQP 类型的 DateTimeOffset.UtcTicks：<type name=”datetime-offset” class=restricted source=”long”> <descriptor name=”com.microsoft:datetime-offset” /></type> |
| TimeSpan | DescribedType | 映射到 AMQP 类型的 Timespan.Ticks：<type name=”timespan” class=restricted source=”long”> <descriptor name=”com.microsoft:timespan” /></type> |
| Uri | DescribedType | 映射到 AMQP 类型的 Uri.AbsoluteUri：<type name=”uri” class=restricted source=”string”> <descriptor name=”com.microsoft:uri” /></type> |

### 标准标头

下表显示了如何使用 AMQP 1.0 映射 JMS 标准标头和 [BrokeredMessage][] 标准属性。

#### JMS 到服务总线 .NET API

| JMS | 服务总线 .NET | 说明                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
|------------------|--------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| JMSCorrelationID | Message.CorrelationID | -                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| JMSDeliveryMode  | 当前不可用             | 服务总线仅支持持久消息；例如，DeliveryMode.PERSISTENT，而不考虑指定的内容。                                                                                                                                                                                                                                                                                                            |
| JMSDestination   | Message.To            | -                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| JMSExpiration    | Message.TimeToLive    | 转换                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| JMSMessageID     | Message.MessageID     | 默认情况下，JMSMessageID 在 AMQP 消息中以二进制格式编码。收到二进制消息 ID 后，.NET 客户端库将根据字节的 unicode 值将其转换为字符串表示形式。若要将 JMS 库切换为使用字符串消息 ID，请在 JNDI ConnectionURL 的查询参数后面追加“binary-messageid=false”字符串。例如：“amqps://[username]:[password]@[namespace].servicebus.chinacloudapi.cn? binary-messageid=false”。 |
| JMSPriority      | 当前不可用             | 服务总线不支持消息优先级。                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| JMSRedelivered   | 当前不可用             | -                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| JMSReplyTo       | 消息。ReplyTo          | -                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| JMSTimestamp | Message.EnqueuedTimeUtc | 转换 |
| JMSType          | Message.Properties[“jms-type”] | -                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |

#### 服务总线 .NET API 到 JMS

| 服务总线 .NET            | JMS              | 说明                     |
|-------------------------|------------------|-------------------------|
| ContentType             | -                  | 当前不可用              |
| CorrelationId           | JMSCorrelationID | -                        |
| EnqueuedTimeUtc         | JMSTimestamp     | 转换                    |
| Label                   | 不适用            | 当前不可用                |
| MessageId               | JMSMessageID     | -                        |
| ReplyTo                 | JMSReplyTo       | -                        |
| ReplyToSessionId        | 不适用            | 当前不可用                |
| ScheduledEnqueueTimeUtc | 不适用            | 当前不可用                |
| SessionId               | 不适用            | 当前不可用                |
| TimeToLive              | JMSExpiration    | 转换                      |
| To                      | JMSDestination    | -                       |

## 不受支持的功能和限制

通过 AMQP 1.0 将 JMS 用于服务总线时存在以下限制：

-   每个会话只允许一个 **MessageProducer** 或 **MessageConsumer**。如果你需要在应用程序中创建多个 **MessageProducer** 或 **MessageConsumer** 对象，请分别为它们创建专用会话。

-   当前不支持易失性主题订阅。

-   不支持 **MessageSelector** 对象。

-   不支持临时目标，例如 **TemporaryQueue** 或 **TemporaryTopic**，以及使用这些目标的 **QueueRequestor** 和 **TopicRequestor** API。

-   不支持事务处理会话。

-   不支持分布式事务。

## 后续步骤

准备好了解详细信息？ 请访问以下链接：

- [服务总线 AMQP 概述]
- [适用于 Windows Server 的服务总线中的 AMQP]

[适用于 Windows Server 的服务总线中的 AMQP]: https://msdn.microsoft.com/zh-cn/library/dn574799.aspx
[BrokeredMessage]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.brokeredmessage.aspx

[服务总线 AMQP 概述]: /documentation/articles/service-bus-amqp-overview/
[Azure 经典门户]: http://manage.windowsazure.cn

<!---HONumber=Mooncake_0104_2016-->