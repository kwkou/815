<properties 
   pageTitle="服务总线 和 PHP 与 AMQP 1.0 | Azure"
   description="使用 AMQP 通过 PHP 使用服务总线。"
   services="service-bus"
   documentationCenter="na"
   authors="sethmanheim"
   manager="timlt"
   editor="" /> 
<tags 
   ms.service="service-bus"
    ms.date="05/06/2016"
   wacn.date="06/27/2016" />

# 使用 AMQP 1.0 通过 PHP 使用服务总线

[AZURE.INCLUDE [service-bus-selector-amqp](../includes/service-bus-selector-amqp.md)]

Proton-PHP 是绑定到 Proton-C 的 PHP 语言；也就是说，Proton-PHP 是作为 C 中实现的引擎周围的包装器实现的。

## 下载 Proton 客户端库

你可以从 [http://qpid.apache.org/download.html](http://qpid.apache.org/download.html) 下载 Proton-C 及其关联的绑定（包括 PHP）。此下载采用源代码格式。若要生成代码，请按照已下载的程序包中包含的说明操作。

> [AZURE.IMPORTANT]在撰写本文时，Proton-C 中的 SSL 支持仅适用于 Linux 操作系统。由于 Azure 服务总线需要使用 SSL，目前，Proton-C（及语言绑定）只能用于从 Linux 访问服务总线。对 Windows 上的 SSL 启用 Proton-C 的开发工作正在进行中，因此请经常返回查看是否有更新。

## 通过 PHP 使用服务总线队列、主题和订阅

以下代码演示如何从服务总线消息实体发送和接收消息。

### 使用 Proton-PHP 发送消息

以下代码演示如何向服务总线消息实体发送消息。

```
$messenger = new Messenger();
$message = new Message();
$message->address = "amqps://[username]:[password]@[namespace].servicebus.chinacloudapi.cn/[entity]";

$message->body = "This is a text string";
$messenger->put($message);
$messenger->send();
```

### 使用 Proton-PHP 接收消息

以下代码演示如何从服务总线消息实体接收消息。

```
$messenger = new Messenger();
$address = "amqps://[username]:[password]@[namespace].servicebus.chinacloudapi.cn/[entity]";
$messenger->subscribe($address);

$messenger->start();
$messenger->recv(1);

if($messenger->incoming())
{
   $message = new Message();
   $messenger->get($message);      
}

$messenger->stop();
```

## 在 .NET 和 Proton-PHP 之间进行消息传送

### 应用程序属性

#### ProtonPHP 到服务总线 .NET API

Proton-PHP 消息支持以下类型的应用程序属性：**integer**、**double**、**Boolean**、**string** 和 **object**。以下 PHP 代码演示如何使用上述每种属性类型在消息上设置属性。

```
$message->properties["TestInt"] = 1;    
$message->properties["TestDouble"] = 1.5;      
$message->properties["TestBoolean"] = False;
$message->properties["TestString"] = "Service Bus";    
$message->properties["TestObject"] = new UUID("1234123412341234");   
```

在服务总线 .NET API 中，在 [BrokeredMessage][] 的 **Properties** 集合中携带消息应用程序属性。以下代码演示如何读取从 PHP 客户端收到的消息的应用程序属性。

```
if (message.Properties.Keys.Count > 0)
{
  foreach (string name in message.Properties.Keys)
  {
    Object value = message.Properties[name];
    Console.WriteLine(name + ": " + value + " (" + value.GetType() + ")" );
  }
  Console.WriteLine();
}if (message.Properties.Keys.Count > 0)
{
foreach (string name in message.Properties.Keys)
{
  Object value = message.Properties[name];
  Console.WriteLine(name + ": " + value + " (" + value.GetType() + ")" );
}
Console.WriteLine();
}
```

下表将 PHP 属性类型映射到 .NET 属性类型。

| PHP 属性类型 | .NET 属性类型 |
|-------------------|--------------------|
| integer | int |
| double | double |
| boolean | bool |
| string | string |
| object | object |

#### 服务总线 .NET API 到 PHP

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
```

以下 PHP 代码演示如何读取从服务总线 .NET 客户端收到的消息的应用程序属性。

```
if ($message->properties != null)
{
  foreach($message->properties as $key => $value)
  {
    printf("-- %s : %s (%s) \n", $key, $value, gettype($value));                       
  }         
}
```

下表将 .NET 属性类型映射到 PHP 属性类型。

| .NET 属性类型 | PHP 属性类型 | 说明 |
|--------------------|-------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| byte | integer | - |
| sbyte | integer | - |
| char | Char | Proton-PHP class |
| short | integer | - |
| ushort | integer | - |
| int | integer | - |
| uint | Integer | - |
| long | integer | - |
| ulong | integer | - |
| float | double | - |
| double | double | - |
| decimal | string | Proton 目前不支持 Decimal。|
| bool | boolean | - |
| Guid | UUID | Proton-PHP 类 |
| string | string | - |
| DateTime | integer | - |
| DateTimeOffset | DescribedType | 映射到 AMQP 类型的 DateTimeOffset.UtcTicks：<type name="datetime-offset" class=restricted source="long"> <descriptor name="com.microsoft:datetime-offset" /></type> |
| TimeSpan | DescribedType | 映射到 AMQP 类型的 Timespan.Ticks：<type name="timespan" class=restricted source="long"> <descriptor name="com.microsoft:timespan" /></type> |
| Uri | DescribedType | 映射到 AMQP 类型的 Uri.AbsoluteUri：<type name="uri" class=restricted source="string"> <descriptor name="com.microsoft:uri" /></type> |

### 标准属性

下表显示了 Proton-PHP 标准消息属性与 [BrokeredMessage][] 标准消息属性之间的映射。

| Proton-PHP | 服务总线 .NET | 说明 |
|----------------------|--------------------------|----------------------------------------------------------|
| Durable | 不适用 | 服务总线仅支持持久消息。 |
| Priority | 不适用 | 服务总线仅支持单一消息优先级。 |
| Ttl | Message.TimeToLive | 转换，Proton-PHP TTL 以毫秒为单位定义。 |
| first\_acquirer | - | - |
| delivery\_count | - | - |
| Id | Message.Id | - |
| user\_id | - | - |
| Address | Message.To | - |
| Subject | Message.Label | - |
| reply\_to | Message.ReplyTo | - |
| correlation\_id | Message.CorrelationId | - |
| content\_type | Message.ContentType | - |
| content\_encoding | 不适用 | - |
| expiry\_time | Message.ExpiresAtUTC | - |
| creation\_time | 不适用 | - |
| group\_id | Message.SessionId | - |
| group\_sequence | - | - |
| reply\_to\_group\_id | Message.ReplyToSessionId | - |
| Format | 不适用 | -

#### 服务总线 .NET API 到 Proton-PHP

| 服务总线 .NET | Proton-PHP | 说明 |
|-------------------------|--------------------------------------------------------|--------------------------------------------------------|
| ContentType | Message->content\_type | - |
| CorrelationId | Message->correlation\_id | - |
| EnqueuedTimeUtc | Message->annotations[x-opt-enqueued-time] | - |
| Label | Message->subject | - |
| MessageId | Message->id | - |
| ReplyTo | Message->reply\_to | - |
| ReplyToSessionId | Message->reply\_to\_group\_id | - |
| ScheduledEnqueueTimeUtc | Message->annotations ["x-opt-scheduled-enqueue-time"] | - |
| SessionId | Message->group\_id | - |
| TimeToLive | Message->ttl | 转换，Proton-PHP TTL 以毫秒为单位定义。|
| To | Message->address | - |

## 后续步骤

准备好了解详细信息？ 请访问以下链接：

- [服务总线 AMQP 概述]
- [适用于 Windows Server 的服务总线中的 AMQP]


[BrokeredMessage]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.brokeredmessage.aspx

[适用于 Windows Server 的服务总线中的 AMQP]: https://msdn.microsoft.com/zh-cn/library/dn574799.aspx

[服务总线 AMQP 概述]: /documentation/articles/service-bus-amqp-overview/

<!---HONumber=82-->