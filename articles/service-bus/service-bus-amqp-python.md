<properties 
   pageTitle="服务总线和 Python 与 AMQP 1.0 | Azure"
   description="使用集成了 AMQP 的 Python 中的服务总线。"
   services="service-bus"
   documentationCenter="na"
   authors="sethmanheim"
   manager="timlt"
    editor="" /> 
<tags 
   ms.service="service-bus"
    ms.date="05/10/2016"
   wacn.date="06/27/2016" />

# 使用集成了 AMQP 1.0 的 Python 中的服务总线。

[AZURE.INCLUDE [service-bus-selector-amqp](../includes/service-bus-selector-amqp.md)]

Proton-Python 是绑定到 Proton-C 的 Python 语言；也就是说，Proton-Python 是作为 C 中实现的引擎周围的包装器实现的。

## 下载 Proton 客户端库

你可以从 [http://qpid.apache.org/download.html](http://qpid.apache.org/download.html) 下载 Proton-C 及其关联的绑定（包括 Python）。此下载采用源代码格式。若要生成代码，请按照已下载的程序包中包含的说明操作。

请注意，在撰写本文时，Proton-C 中的 SSL 支持仅适用于 Linux 操作系统。由于 Azure 服务总线需要使用 SSL，目前，Proton-C（及语言绑定）只能用于从 Linux 访问服务总线。对 Windows 上的 SSL 启用 Proton-C 的开发工作正在进行中，因此请经常返回查看是否有更新。

## 通过 Python 使用服务总线队列、主题和订阅

以下代码演示如何从服务总线消息实体发送和接收消息。

### 使用 Proton-Python 发送消息

以下代码演示如何向服务总线消息实体发送消息。

```
messenger = Messenger()
message = Message()
message.address = "amqps://[username]:[password]@[namespace].servicebus.chinacloudapi.cn/[entity]"

message.body = u"This is a text string"
messenger.put(message)
messenger.send()
```

### 使用 Proton-Python 接收消息

以下代码演示如何从服务总线消息实体接收消息。

```
messenger = Messenger()
address = "amqps://[username]:[password]@[namespace].servicebus.chinacloudapi.cn/[entity]"
messenger.subscribe(address)

messenger.start()
messenger.recv(1)
message = Message()
if messenger.incoming:
      messenger.get(message)
messenger.stop()
```

## 在 .NET 和 Proton-Python 之间进行消息传递

### 应用程序属性

#### Proton-Python 到服务总线 .NET API

Proton-Python 消息支持以下类型的应用程序属性：**int**、**long**、**float**、**uuid**、**bool**、**string**。以下 Python 代码显示如何使用上述每种属性类型在消息上设置属性。

```
message.properties[u"TestString"] = u"This is a string"    
message.properties[u"TestInt"] = 1
message.properties[u"TestLong"] = 1000L
message.properties[u"TestFloat"] = 1.5    
message.properties[u"TestGuid"] = uuid.uuid1()    
```

在服务总线 .NET API 中，在 [BrokeredMessage][] 的 **Properties** 集合中携带消息应用程序属性。以下代码演示如何读取从 Python 客户端收到的消息的应用程序属性。

```
if (message.Properties.Keys.Count > 0)
{
  foreach (string name in message.Properties.Keys)
  {
    Object value = message.Properties[name];
    Console.WriteLine(name + ": " + value + " (" + value.GetType() + ")" );
  }
  Console.WriteLine();
}
```

下表将 Python 属性类型映射到 .NET 属性类型。

| Python 属性类型 | .NET 属性类型 |
|----------------------|--------------------|
| int | int |
| float | double |
| long | int64 |
| uuid | guid |
| bool | bool |
| string | string |

#### 服务总线 .NET API 到 Proton-Python

[BrokeredMessage][] 类型支持以下类型的应用程序属性：**byte**、**sbyte**、**char**、**short**、**ushort**、**int**、**uint**、**long**、**ulong**、**float**、**double**、**decimal**、**bool**、**Guid**、**string**、**Uri**、**DateTime**、**DateTimeOffset** 和 **TimeSpan**。以下 .NET 代码显示如何使用上述每种属性类型在 [BrokeredMessage][] 对象上设置属性。

```
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
message.Properties["TestTimeSpan"] = TimeSpan.FromMinutes(60);
```

以下 Python 代码演示如何读取从服务总线 .NET 客户端收到的消息的应用程序属性。

```
if message.properties != None:
   for k,v in message.properties.items():         
         print "--   %s : %s (%s)" % (k, str(v), type(v))         
```

下表将 .NET 属性类型映射到 Python 属性类型。

| .NET 属性类型 | Python 属性类型 | 说明 |
|--------------------|----------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| byte | int | - | 
| sbyte | int | - | 
| char | char | Proton-Python 类 | 
| short | int | - | 
| ushort | int | - | 
| int | int | - | 
| uint | int | - | 
| long | int | - | 
| ulong | long | Proton-Python 类 | 
| float | float | - | 
| double | float | - | 
| decimal | String | Proton 目前不支持 Decimal。| 
| bool | bool | - | 
| Guid | uuid | Proton-Python 类 | 
| string | string | - | 
| DateTime | timestamp | Proton-Python 类 | 
| DateTimeOffset | DescribedType |映射到 AMQP 类型的 DateTimeOffset.UtcTicks：<type name=”datetime-offset” class=restricted source=”long”> <descriptor name=”com.microsoft:datetime-offset” /></type> | 
| TimeSpan | DescribedType | 映射到 AMQP 类型的 Timespan.Ticks：<type name=”timespan” class=restricted source=”long”> <descriptor name=”com.microsoft:timespan” /></type> | 
| Uri | DescribedType | 映射到 AMQP 类型的 Uri.AbsoluteUri：<type name=”uri” class=restricted source=”string”> <descriptor name=”com.microsoft:uri” /></type> |

### 标准属性

下表显示了 Proton-Python 标准消息属性与 [BrokeredMessage][] 标准消息属性之间的映射。

#### Proton-Python 到服务总线 .NET API

| Proton-Python | 服务总线 .NET | 说明 |
|----------------------|--------------------------|-----------------------------------------------------------|
| durable | 不适用 | 服务总线仅支持持久消息。 |
| priority | 不适用 | 服务总线仅支持单一消息优先级。 |
| Ttl | Message.TimeToLive | 转换，Proton-Python TTL 以毫秒为单位定义。 |
| first\_acquirer | 不适用 | - |
| delivery\_count | 不适用 | - |
| Id | Message.MessageID | - |
| user\_id | 不适用 | - |
| address | Message.To | - |
| subject | Message.Label | - |
| reply\_to | Message.ReplyTo | - |
| correlation\_id | Message.CorrelationID | - |
| content\_type | Message.ContentType | - |
| content\_encoding | 不适用 | - |
| expiry\_time | 不适用 | - |
| creation\_time | 不适用 | - |
| group\_id | Message.SessionId | - |
| group\_sequence | 不适用 | - |
| reply\_to\_group\_id | Message.ReplyToSessionId | - |
| format | 不适用 | - |

| 服务总线 .NET | Proton | 说明 |
|-------------------------|------------------------------|-----------------------------------------------------------|
| ContentType | Message.content\_type | - |
| CorrelationId | Message.correlation\_id | - |
| EnqueuedTimeUtc | 不适用 | - |
| Label | Message.subject | - |
| MessageId | Message.id | - |
| ReplyTo | Message.reply\_to | - |
| ReplyToSessionId | Message.reply\_to\_group\_id | - |
| ScheduledEnqueueTimeUtc | 不适用 | - |
| SessionId | Message.group\_id | - |
| TimeToLive | Message.ttl | 转换，Proton-Python TTL 以毫秒为单位定义。|
| To | Message.address | - |

## 后续步骤

准备好了解详细信息？ 请访问以下链接：

- [服务总线 AMQP 概述]
- [适用于 Windows Server 的服务总线中的 AMQP]

[BrokeredMessage]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.brokeredmessage.aspx
[适用于 Windows Server 的服务总线中的 AMQP]: https://msdn.microsoft.com/zh-cn/library/dn574799.aspx

[服务总线 AMQP 概述]: /documentation/articles/service-bus-amqp-overview/

<!---HONumber=82-->