<properties
    pageTitle="了解 Azure IoT 中心消息传送 | Azure"
    description="开发人员指南 - 使用 IoT 中心进行设备到云和云到设备的消息传送。包括消息格式和支持的通信协议的相关信息。"
    services="iot-hub"
    documentationcenter=".net"
    author="dominicbetts"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="3fc5f1a3-3711-4611-9897-d4db079b4250"
    ms.service="iot-hub"
    ms.devlang="multiple"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="12/13/2016"
    wacn.date="01/13/2017"
    ms.author="dobett" />  


# 使用 IoT 中心发送和接收消息
## 概述
IoT 中心提供了用于与设备通信的以下消息传送基元：

* [设备到云][lnk-d2c]：从设备到后端应用。
* [云到设备][lnk-c2d]：从后端应用（*服务*或*云*）。

IoT 中心消息传送功能的核心属性是消息的可靠性和持久性。这些属性可在设备端上恢复间歇性连接，以及在云恢复事件处理的负载高峰。IoT 中心对设备到云和云到设备的消息传送实施*至少一次*传送保证。

IoT 中心支持多个[面向设备的协议][lnk-protocols]（例如 MQTT、AMQP 和 HTTP）。为了支持无缝的跨协议互操作性，IoT 中心定义了所有面向设备的协议均可支持的[通用消息格式][lnk-message-format]。

IoT 中心公开[与事件中心兼容的终结点][lnk-compatible-endpoint]，让后端应用能读取中心收到的设备到云消息。通过将订阅中的其他服务链接到中心，还可以将自定义路由终结点添加到 IoT 中心。

### 何时使用
使用设备到云消息从设备应用发送时序遥测数据和警报，使用云到设备消息向设备应用发送单向通知。

如果在使用报告属性、设备到云消息或文件上载方面有任何疑问，请参阅[设备到云通信指南][lnk-d2c-guidance]。
如果在使用所需属性、直接方法或云到设备消息方面有任何疑问，请参阅[云到设备通信指南][lnk-c2d-guidance]。

有关 IoT 中心与事件中心服务的比较，请参阅 [IoT 中心与事件中心的比较][lnk-compare]。

## <a name="device-to-cloud-messages"></a> 设备到云的消息
通过面向设备的终结点 \(**/devices/{deviceId}/messages/events**\) 发送设备到云消息。消息路由随后将消息路由到 IoT 中心内面向服务的终结点之一。消息路由使用流经中心的设备到云消息的属性来确定将消息路由到的位置。默认情况下，消息将路由到与[事件中心][lnk-event-hubs]兼容的内置面向服务的终结点 \(messages/events\) 中。因此，可以使用标准[事件中心集成和 SDK][lnk-compatible-endpoint] 来接收设备到云的消息。

IoT 中心使用流式消息传递模式实现设备到云的消息传递。与[事件中心][lnk-event-hubs]*事件*和[服务总线][lnk-servicebus]*消息*相比，IoT 中心的设备到云消息更类似前者，类似之处在于有大量事件通过可供多个读取器读取的服务。

这种实现具有以下含义：

* 与事件中心事件类似，设备到云的消息可持久保留在 IoT 中心的默认 **messages/events** 终结点多达 7 天（请参阅[设备到云的配置选项][lnk-d2c-configuration]）。
* 如同事件中心的事件，设备到云的消息最大可为 256 KB，而且可分成多个批以优化发送。Batch 最大可为 256 KB。

不过，IoT 中心的设备到云的消息传递与事件中心之间还有一些重要差异：

* 如[控制 IoT 中心的访问权限][lnk-devguide-security]部分中所述，IoT 中心允许基于设备的身份验证和访问控制。
* IoT 中心允许添加至多 10 个附加终结点。基于 IoT 中心上配置的路由，将消息传递到终结点。
* IoT 中心允许同时连接数百万个设备（请参阅[配额和限制][lnk-quotas]），而事件中心则限制为每个命名空间 5000 个 AMQP 连接。
* IoT 中心不允许使用 **PartitionKey** 任意分区。设备到云的消息根据其源于的 **deviceId** 进行分区。
* IoT 中心的缩放方式与事件中心稍有不同。有关详细信息，请参阅 [Scaling IoT Hub][lnk-guidance-scale]（缩放 IoT 中心）。

有关如何使用设备到云的消息传送的详细信息，请参阅 [Azure IoT SDK][lnk-sdks]。

有关如何设置消息路由的详细信息，请参阅以下设备到云配置选项。

> [AZURE.NOTE]
使用 HTTP 发送设备到云的消息时，属性名称和值只能包含 ASCII 字母数字字符加上 ``{'!', '#', '$', '%, '&', "'", '*', '*', '+', '-', '.', '^', '_', '`', '|', '~'}``。
> 
> 

### 非遥测流量
通常，除了遥测数据点之外，设备还会发送消息以及要求单独执行和处理应用程序业务逻辑层的请求。例如，关键警报必须在后端触发特定操作。可轻松编写路由规则，将这些类型的消息发送到专用于其处理的终结点。

有关此类消息的最佳处理方式的详细信息，请参阅[教程：如何处理 IoT 中心设备到云的消息][lnk-d2c-tutorial]教程。

### <a name="device-to-cloud-configuration-options"></a> 设备到云的配置选项
IoT 中心允许基于消息属性将消息路由到 IoT 中心终结点。使用路由规则可将消息灵活发送到所需目标位置，无需借助其他服务来处理消息，也无需编写其他代码。配置的每个规则都具有以下属性

* **Name**。这是标识规则的唯一名称。
* **Source**。表示要处理的数据流的来源。例如，设备遥测。
* **Condition**。这是路由规则的查询表达式，针对消息属性运行，用于确定它是否与终结点匹配。有关构造路由条件的详细信息，请参阅[参考 - 设备孪生和作业的查询语言][lnk-devguide-query-language]。
* **Endpoint**。IoT 中心将匹配条件的消息发送到的终结点的名称。终结点应与 IoT 中心位于同一区域，因为跨区域写入将产生费用。

一条消息可能与多个消息路由上的条件匹配，在这种情况下，IoT 中心会将该消息传递到与每个匹配规则关联的终结点。IoT 中心还会自动删除重复的消息传递，因此如果消息与具有相同目标位置的多个规则匹配，则仅会将其写入该目标位置一次。

有关将附加终结点添加到 IoT 中心的详细信息，请参阅[IoT 中心终结点][lnk-devguide-endpoints]。

#### 内置终结点：messages/events

IoT 中心公开以下属性以允许用户控制内置消息传送终结点 **messages/events**。

* **分区计数**。在创建时设置此属性，以便为设备到云事件引入定义分区数。
* **保留时间**。此属性指定设备到云消息的保留时间。默认值为一天，但可以增加到七天。

IoT 中心还支持用户管理内置设备到云接收终结点上的使用者组。

默认情况下，不显式匹配消息路由规则的所有消息都将写入到内置终结点。如果禁用此“回退路由”，将删除不显式匹配任何消息路由规则的消息。

可以通过 [IoT 中心资源提供程序 REST API][lnk-resource-provider-apis] 以编程方式修改上述所有属性，或使用 [Azure 门户][lnk-management-portal]进行修改。

### <a name="anti-spoofing-properties"></a> 反欺骗属性
为了避免设备到云的消息中出现设备欺骗，IoT 中心使用以下属性在所有消息上加上戳记：

* **ConnectionDeviceId**
* **ConnectionDeviceGenerationId**
* **ConnectionAuthMethod**

根据[设备标识属性][lnk-device-properties]，前两个属性包含源设备的 **deviceId** 和 **generationId**。

**ConnectionAuthMethod** 属性包含具有以下属性的 JSON 序列化对象：

	{
	  "scope": "{ hub | device}",
	  "type": "{ symkey | sas}",
	  "issuer": "iothub"
	}

## <a name="cloud-to-device-messages"></a> 云到设备的消息
可以通过面向服务的终结点 \(**/messages/devicebound**\) 发送云到设备的消息。设备可以通过特定于设备的终结点 \(**/devices/{deviceId}/messages/devicebound**\) 接收这些消息。

每个云到设备的消息都以单个设备为目标，方法是将 **to** 属性设置为 **/devices/{deviceId}/messages/devicebound**。

> [AZURE.IMPORTANT]
每个设备队列最多可以保留 50 条云到设备的消息。尝试将更多消息传送到同一设备将导致错误。
> 
> [AZURE.NOTE]
发送云到设备的消息时，属性名称和值只能包含 ASCII 字母数字字符加上 ``{'!', '#', '$', '%, '&', "'", '*', '*', '+', '-', '.', '^', '_', '`', '|', '~'}``。
> 
> 

### <a name="message-lifecycle"></a> 消息生命周期
为了保证至少一次消息传递，IoT 中心将云到设备的消息保留在每个设备队列中。设备必须显式确认*完成*，IoT 中心才会从队列中将其删除。这是为了保证连接失败和设备故障时能够恢复。

下图显示了云到设备消息的生命周期状态图。

![云到设备的消息生命周期][img-lifecycle]  


当服务发送消息时，该消息被视为*已排队*。当设备想要*接收*消息时，IoT 中心将*锁定*该消息（将状态设置为**不可见**），以便让同一设备上的其他线程开始接收其他消息。当设备线程完成消息的处理后，将通过*完成*消息来通知 IoT 中心。

设备还可以：

* *拒绝*消息，这会使 IoT 中心将此消息设置为**死信**状态。注意：使用 MQTT 连接的设备不能拒绝云到设备消息。
* *放弃*消息，这会使 IoT 中心将消息放回队列，并将状态设置为**已排队**。

线程可能无法处理消息，且不通知 IoT 中心。在此情况下，在*可见性（或锁定）超时*时间之后，消息将从**不可见**状态自动转换回**已排队**状态。此超时的默认值为一分钟。

消息可以在**已排队**与**不可见**状态之间转换的次数，以 IoT 中心上**最大传递计数**属性中指定的次数为上限。在该转换次数之后，IoT 中心会将消息的状态设置为**死信**。同样，IoT 中心也会在消息的到期时间之后（请参阅[生存时间][lnk-ttl]），将消息的状态设置为**死信**。

有关云到设备的消息的教程，请参阅[教程：如何使用 IoT 中心发送云到设备的消息][lnk-c2d-tutorial]。有关不同 Azure IoT SDK 如何公开云到设备功能的参考主题，请参阅 [Azure IoT SDK][lnk-sdks]。

> [AZURE.NOTE]
通常只要丢失消息不影响应用程序逻辑，就会完成云到设备的消息。例如，消息内容已成功保留在本地存储空间中，或已成功执行某操作。该消息还可能携带暂时性信息，该信息的丢失不会影响应用程序的功能。有时，对于长时间运行的任务，你可以在将任务说明保留到本地存储空间后完成该云到设备的消息。然后，在作业进度的不同阶段，可以使用一条或多条设备到云的消息通知解决方案后端。
> 
> 

### <a name="message-expiration-time-to-live"></a> 消息到期时间（生存时间）
每条云到设备的消息都有过期时间。此时间可以由服务（在 **ExpiryTimeUtc** 属性中）设置，或者由 IoT 中心使用指定为 IoT 中心属性的默认 *生存时间* 来设置。请参阅[云到设备的配置选项][lnk-c2d-configuration]。

> [AZURE.NOTE]
利用消息到期时间并避免将消息发送到已断开连接的设备的常见方法是设置较短的生存时间值。此方法可达到与维护设备连接状态一样的效果，而且更加有效。请求消息确认时，IoT 中心可以通知你哪些设备可以接收消息、哪些设备脱机或已出现故障。
> 
> 

### <a name="message-feedback"></a> 消息反馈
当你发送云到设备的消息时，服务可以请求传送每条消息的反馈（关于该消息的最终状态）。

* 如果将 **Ack** 属性设置为 **positive**，则当且仅当云到设备的消息达到**已完成**状态时，IoT 中心才生成反馈消息。
* 如果将 **Ack** 属性设置为 **negative**，则当且仅当云到设备的消息达到**死信**状态时，IoT 中心才生成反馈消息。
* 如果将 **Ack** 属性设置为 **full**，则 IoT 中心在上述任一情况下都会生成反馈消息。

> [AZURE.NOTE]
如果 **Ack** 为 **full**，且未收到反馈消息，则意味着反馈消息已过期。该服务无法了解原始消息的经历。实际上，服务应该确保它可以在反馈过期之前对其进行处理。最长过期时间是两天，因此当发生失败时，有相当充裕的时间让服务再次运行。
> 
> 

如[终结点][lnk-endpoints]中所述，IoT 中心通过面向服务的终结点 (**/messages/servicebound/feedback**) 以消息方式传送反馈。接收反馈的语义与云到设备的消息的语义相同，并且具有相同的[消息生命周期][lnk-lifecycle]。可能的话，消息反馈将放入单个消息中，其格式如下：

| 属性 | 说明 |
| --- | --- |
| EnqueuedTime |指示创建消息时的时间戳。 |
| UserId |`{iot hub name}` |
| ContentType |`application/vnd.microsoft.iothub.feedback.json` |

正文是记录的 JSON 序列化数组，每条记录具有以下属性：

| 属性 | 说明 |
| --- | --- |
| EnqueuedTimeUtc |指示消息结果出现时的时间戳。例如，设备已完成或消息已过期。 |
| OriginalMessageId |此反馈信息所属的云到设备的消息的 **MessageId**。 |
| StatusCode |必须是整数。在 IoT 中心生成的反馈消息中使用。<br/> 0 = 成功 <br/> 1 = 消息已过期 <br/> 2 = 超出最大传送计数 <br/> 3 = 消息被拒绝 |
| 说明 |**StatusCode** 的字符串值。 |
| DeviceId |此反馈信息所属的云到设备的消息的目标设备的 **DeviceId**。 |
| DeviceGenerationId |此反馈信息所属的云到设备的消息的目标设备的 **DeviceGenerationId**。 |

> [AZURE.IMPORTANT]
服务必须指定云到设备的消息的 **MessageId**，才能将其反馈与原始消息相关联。
> 
> 

以下示例演示了反馈消息的正文。

	[
	  {
	    "OriginalMessageId": "0987654321",
	    "EnqueuedTimeUtc": "2015-07-28T16:24:48.789Z",
	    "StatusCode": 0
	    "Description": "Success",
	    "DeviceId": "123",
	    "DeviceGenerationId": "abcdefghijklmnopqrstuvwxyz"
	  },
	  {
	    ...
	  },
	  ...
	]


### <a name="cloud-to-device-configuration-options"></a> 云到设备的配置选项
每个 IoT 中心都针对云到设备的消息传送公开以下配置选项。

| 属性 | 说明 | 范围和默认值 |
| --- | --- | --- |
| defaultTtlAsIso8601 | 云到设备消息的默认 TTL。 | ISO\_8601 间隔高达 2D（最小为 1 分钟）。默认值：1 小时。 |
| maxDeliveryCount | 每个设备队列的云到设备最大传送计数。 | 1 到 100。默认值：10。 |
| feedback.ttlAsIso8601 | 服务绑定反馈消息的保留时间。 | ISO\_8601 间隔高达 2D（最小为 1 分钟）。默认值：1 小时。 |
| feedback.maxDeliveryCount | 反馈队列的最大传送计数。 | 1 到 100。默认值：100。 |

有关详细信息，请参阅[创建 IoT 中心][lnk-portal]。


## <a name="read-device-to-cloud-messages"></a> 读取设备到云的消息
IoT 中心向后端服务公开内置终结点，以便让后端服务读取中心 - 即 **messages/events** 终结点收到的设备到云的消息。此终结点与事件中心兼容，这样就可以使用事件中心服务支持的任何机制读取消息。

也可向 IoT 中心添加自定义路由终结点。IoT 中心目前支持将事件中心、服务总线队列和服务总线主题添加为自定义路由终结点。有关从这些服务中读取信息的详细信息，请参阅：从[事件中心][lnk-getstarted-eh]读取信息、从[服务总线队列][lnk-getstarted-queue]读取信息，以及从[服务总线主题][lnk-getstarted-topic]读取信息。

### 从内置终结点读取信息

当你使用[适用于 .NET 的 Azure 服务总线 SDK][lnk-servicebus-sdk] 或[事件中心 - 事件处理器主机][lnk-eventprocessorhost]时，可以将任何 IoT 中心连接字符串与正确的权限配合使用。然后使用**消息/事件**作为事件中心名称。

使用无法识别 IoT 中心的 SDK（或产品集成）时，必须从 [Azure 门户预览][lnk-management-portal]中的 IoT 中心设置检索与事件中心兼容的终结点和与事件中心兼容的名称：

1. 单击“IoT 中心边栏选项卡”中的“终结点”。
2. 在“内置终结点”部分，单击“事件”。边栏选项卡包含以下值：“事件中心兼容的终结点”、“事件中心兼容的名称”、“分区”、“保留时间”和“使用者组”。
   
    ![设备到云的设置][img-eventhubcompatible]  

> [AZURE.NOTE]
IoT 中心 SDK 需要 IoT 中心终结点名称，即“终结点”边栏选项卡中所示的 **messages/events**。
>
>

> [AZURE.NOTE]
如果当前使用的 SDK 需要“主机名”或“命名空间”值，请从“事件中心兼容的终结点”中删除方案。例如，如果与事件中心兼容的终结点为 **sb://iothub-ns-myiothub-1234.servicebus.chinacloudapi.cn/**，则**主机名**为 **iothub-ns-myiothub-1234.servicebus.chinacloudapi.cn**，**命名空间**为 **iothub-ns-myiothub-1234**。
> 
>

然后，可以使用具有 **ServiceConnect** 权限的任何共享访问策略连接到指定的事件中心。

如果需要通过使用以前的信息构建事件中心连接字符串，请使用以下模式：

```
Endpoint={Event Hub-compatible endpoint};SharedAccessKeyName={iot hub policy name};SharedAccessKey={iot hub policy key}
```

以下是可以配合 IoT 中心公开的事件中心兼容终结点使用的 SDK 和集成项目列表：

- [Java 事件中心客户端](https://github.com/hdinsight/eventhubs-client)
- [Apache Storm Spout](/documentation/articles/hdinsight-storm-develop-csharp-event-hub-topology/)。可以在 GitHub 上查看 [Spout 源代码](https://github.com/apache/storm/tree/master/external/storm-eventhubs)。

## 参考主题：

以下参考主题提供有关与 IoT 中心交换消息的详细信息。


## <a name="message-format"></a> 消息格式
IoT 中心消息包含：

* 一组*系统属性*。IoT 中心解释或设置的属性。此集合是预先确定的。
* 一组*应用程序属性*。应用程序可以定义的字符串属性字典，而不需将消息正文反序列化即可进行访问。IoT 中心永不修改这些属性。
* 不透明的二进制正文。

有关在不同的协议中如何对消息进行编码的详细信息，请参阅 [Azure IoT SDK][lnk-sdks]。

下表列出 IoT 中心消息中的系统属性集。

| 属性 | 说明 |
| --- | --- |
| MessageId |用户可设置的消息标识符，用于请求-答复模式。格式：ASCII 7 位字母数字字符 + `{'-', ':',’.', '+', '%', '_', '#', '*', '?', '!', '(', ')', ',', '=', '@', ';', '$', '''}` 的区分大小写字符串（最长为 128 个字符）。 |
| 序列号 |IoT 中心分配给每条云到设备消息的编号（对每个设备队列是唯一的）。 |
| 如果 |[云到设备][lnk-c2d]消息中指定的目标。 |
| ExpiryTimeUtc |消息过期的日期和时间。 |
| EnqueuedTime |IoT 中心收到消息的日期和时间。 |
| CorrelationId |响应消息中的字符串属性，通常包含采用“请求-答复”模式的请求的 MessageId。 |
| UserId |用于指定消息的源的 ID。如果消息是由 IoT 中心生成的，则设置为 `{iot hub name}`。 |
| Ack |反馈消息生成器。此属性在云到设备的消息中用于请求 IoT 中心因为设备使用消息而生成反馈消息。可能的值：**none**（默认值）：不生成任何反馈消息；**positive**：如果消息已完成，则接收反馈消息；**negative**：如果消息未由设备完成就过期（或已达到最大传送计数），则收到反馈消息；**full**：positive 和 negative。有关详细信息，请参阅[消息反馈][lnk-feedback]。 |
| ConnectionDeviceId |IoT 中心对设备到云的消息设置的 ID。它包含发送消息的设备的 **deviceId**。 |
| ConnectionDeviceGenerationId |IoT 中心对设备到云的消息设置的 ID。它包含发送消息的设备的 **generationId**（根据[设备标识属性][lnk-device-properties]）。 |
| ConnectionAuthMethod |由 IoT 中心对设备到云的消息设置的身份验证方法。此属性包含用于验证发送消息的设备的身份验证方法的相关信息。有关详细信息，请参阅[设备到云的反欺骗技术][lnk-antispoofing]。 |

## <a name="message-size"></a> 消息大小

IoT 中心用于衡量消息大小的方法与协议无关，仅考虑实际有效负载。以字节为单位的大小计算以下各项之和：

* 以字节为单位的正文大小，加上
* 以字节为单位的消息系统属性的所有值的大小，加上
* 以字节为单位的所有用户属性名称和值的大小。

请注意，属性名称和值限制为 ASCII 字符，因此，字符串的长度等于以字节为单位的大小。

## <a name="communication-protocols"></a> 通信协议
IoT 中心允许设备使用 [MQTT][lnk-mqtt]、基于 WebSocket 的 MQQT、[AMQP][lnk-amqp]、基于 WebSocket 的 AMQP 和 HTTP 协议进行设备端通信。下表提供了针对协议选取的高水平建议：

| 协议 | 何时应选择此协议 |
| --- | --- |
| MQTT <br> 基于 WebSocket 的 MQTT |用于无需使用相同的 TLS 连接来连接多台设备（各有自己的设备凭据）的所有设备。 |
| AMQP <br> 基于 WebSocket 的 AMQP |用于利用跨设备连接复用的字段和云网关。 |
| HTTP |用于不可支持其他协议的设备。 |

在选择设备端通信协议时，请考虑以下几点：

* **云到设备模式**。HTTP 没有用于实现服务器推送的有效方法。因此，使用 HTTP 时，设备会在 IoT 中心轮询云到设备消息。此方法对于设备和 IoT 中心而言是低效的。根据当前 HTTP 准则，每台设备应每 25 分钟或更长时间轮询一次消息。而 MQTT 和 AMQP 支持服务器在收到云到设备的消息时进行推送。它们将启用从 IoT 中心到设备的直接消息推送。如果传送延迟是考虑因素，最好使用 MQTT 或 AMQP 协议。对于很少连接的设备，HTTP 也适用。
* **现场网关**。使用 MQTT 和 HTTP 时，无法使用相同的 TLS 连接来连接多台设备（各有自己的设备凭据）。因此，对于[现场网关方案][lnk-azure-gateway-guidance]，这些并不是最理想的协议，因为对于每个连接到现场网关的设备，这些协议需要在现场网关和 IoT 中心之间建立一个 TLS 连接。
* **低资源设备**。MQTT 和 HTTP 库的占用空间比 AMQP 库的占用空间小。因此，如果设备的资源很少（如低于 1 MB RAM），可能只可实现这些协议。
* **网络遍历**。标准 AMQP 协议使用端口 5671，而 MQTT 在端口 8883 上侦听，这可能会导致对非 HTTP 协议关闭的网络发生问题。基于 WebSocket 的 MQTT、基于 WebSocket 的 AMQP 和 HTTP 均可用于此方案。
* **有效负载大小**。MQTT 和 AMQP 是二进制协议，因此其有效负载比 HTTP 的有效负载更精简。

> [AZURE.NOTE] 使用 HTTP 时，每台设备应每 25 分钟或更长时间轮询一次云到设备消息。但在开发期间，可按低于 25 分钟的更高频率进行轮询。

## 端口号

设备可在 Azure 中使用各种协议来与 IoT 中心通信。通常，选择的协议根据解决方案的具体要求而定。下表列出了必须开放，从而使设备能够使用特定协议的出站端口：

| 协议 | 端口 |
| --- | --- |
| MQTT |8883 |
| 基于 WebSocket 的 MQTT |443 |
| AMQP |5671 |
| 基于 WebSockets 的 AMQP |443 |
| HTTP |443 |
| LWM2M（设备管理） |5684 |

在 Azure 区域创建 IoT 中心后，该 IoT 中心在其生存期内会保留同一 IP 地址。但为了保证服务质量，如果 Microsoft 调整 IoT 中心的大小，则向其分配新 IP 地址。


## <a name="notes-on-mqtt-support"></a> 有关 MQTT 支持的说明
IoT 中心实现 MQTT v3.1.1 协议，但具有以下限制和特定行为：

* **不支持 QoS 2**。如果设备应用使用 **QoS 2** 发布消息，IoT 中心将关闭网络连接。当设备应用使用 **QoS 2** 订阅主题时，IoT 中心将在 **SUBACK** 数据包中授予最大 QoS 级别 1。
* **不会保留保留消息**。如果设备应用发布 RETAIN 标志设置为 1 的消息，IoT 中心将在消息中添加 **x-opt-retain** 应用程序属性。在此情况下，IoT 中心不会持续保留消息，而是将它传递到后端应用。

有关详细信息，请参阅 [IoT Hub MQTT support][lnk-devguide-mqtt]（IoT 中心 MQTT 支持）。

最后请检查 [Azure IoT 协议网关][lnk-azure-protocol-gateway]，可以通过它部署直接与 IoT 中心连接的高性能自定义协议网关。Azure IoT 协议网关可让你自定义设备协议，以适应要重建的 MQTT 部署或其他自定义协议。但是，这种方法要求运行并使用自定义协议网关。

## 其他参考资料
IoT 中心开发人员指南中的其他参考主题包括：

* [IoT 中心终结点][lnk-endpoints]，介绍了每个 IoT 中心针对运行时和管理操作公开的各种终结点。
* [限制和配额][lnk-quotas]，说明了适用于 IoT 中心服务的配额，以及使用服务时预期会碰到的限制行为。
* [Azure IoT 设备和服务 SDK][lnk-sdks]，列出了在开发与 IoT 中心交互的设备和服务应用时可使用的各种语言 SDK。
* [设备孪生和作业的 IoT 中心查询语言][lnk-query]，介绍了在 IoT 中心检索设备孪生和作业相关信息时可使用的 IoT 中心查询语言。
* [IoT 中心 MQTT 支持][lnk-devguide-mqtt]提供有关 IoT 中心对 MQTT 协议的支持的详细信息。

## 后续步骤
了解如何使用 IoT 中心发送和接收消息后，可以根据兴趣参阅以下开发人员指南主题：

- [从设备上载文件][lnk-devguide-upload]
- [管理 IoT 中心中的设备标识][lnk-devguide-identities]
- [控制 IoT 中心的访问权限][lnk-devguide-security]
- [使用设备孪生同步状态和配置][lnk-devguide-device-twins]
- [在设备上调用直接方法][lnk-devguide-directmethods]
- [在多台设备上计划作业][lnk-devguide-jobs]

如果要尝试本文中介绍的一些概念，你可能对以下 IoT 中心教程感兴趣：

- [Azure IoT 中心入门][lnk-getstarted-tutorial]
- [如何使用 IoT 中心发送云到设备的消息][lnk-c2d-tutorial]
- [如何处理 IoT 中心设备到云的消息][lnk-d2c-tutorial]


[img-lifecycle]: ./media/iot-hub-devguide-messaging/lifecycle.png
[img-eventhubcompatible]: ./media/iot-hub-devguide-messaging/eventhubcompatible.png

[lnk-resource-provider-apis]: https://msdn.microsoft.com/zh-cn/library/mt548492.aspx
[lnk-azure-gateway-guidance]: /documentation/articles/iot-hub-devguide-endpoints/#field-gateways
[lnk-guidance-scale]: /documentation/articles/iot-hub-scaling/
[lnk-azure-protocol-gateway]: /documentation/articles/iot-hub-protocol-gateway/
[lnk-amqp]: http://docs.oasis-open.org/amqp/core/v1.0/os/amqp-core-complete-v1.0-os.pdf
[lnk-mqtt]: http://docs.oasis-open.org/mqtt/mqtt/v3.1.1/mqtt-v3.1.1.pdf
[lnk-event-hubs]: /documentation/services/event-hubs/
[lnk-event-hubs-consuming-events]: /documentation/articles/event-hubs-programming-guide/#event-consumers
[lnk-management-portal]: https://portal.azure.cn
[lnk-servicebus]: /documentation/services/service-bus/
[lnk-eventhub-partitions]: /documentation/articles/event-hubs-overview/#partitions
[lnk-portal]: /documentation/articles/iot-hub-create-through-portal/
[lnk-getstarted-eh]: /documentation/articles/event-hubs-csharp-ephcs-getstarted/
[lnk-getstarted-queue]: /documentation/articles/service-bus-dotnet-get-started-with-queues/
[lnk-getstarted-topic]: /documentation/articles/service-bus-dotnet-how-to-use-topics-subscriptions/

[lnk-c2d-guidance]: /documentation/articles/iot-hub-devguide-c2d-guidance/
[lnk-d2c-guidance]: /documentation/articles/iot-hub-devguide-d2c-guidance/
[lnk-endpoints]: /documentation/articles/iot-hub-devguide-endpoints/
[lnk-quotas]: /documentation/articles/iot-hub-devguide-quotas-throttling/
[lnk-sdks]: /documentation/articles/iot-hub-devguide-sdks/
[lnk-query]: /documentation/articles/iot-hub-devguide-query-language/
[lnk-devguide-mqtt]: /documentation/articles/iot-hub-mqtt-support/
[lnk-d2c]: /documentation/articles/iot-hub-devguide-messaging/#device-to-cloud-messages
[lnk-c2d]: /documentation/articles/iot-hub-devguide-messaging/#cloud-to-device-messages
[lnk-compatible-endpoint]: /documentation/articles/iot-hub-devguide-messaging/#read-device-to-cloud-messages
[lnk-protocols]: /documentation/articles/iot-hub-devguide-messaging/#communication-protocols
[lnk-message-format]: /documentation/articles/iot-hub-devguide-messaging/#message-format
[lnk-d2c-configuration]: /documentation/articles/iot-hub-devguide-messaging/#device-to-cloud-configuration-options
[lnk-device-properties]: /documentation/articles/iot-hub-devguide-identity-registry/#device-identity-properties
[lnk-ttl]: /documentation/articles/iot-hub-devguide-messaging/#message-expiration-time-to-live
[lnk-c2d-configuration]: /documentation/articles/iot-hub-devguide-messaging/#cloud-to-device-configuration-options
[lnk-lifecycle]: /documentation/articles/iot-hub-devguide-messaging/#message-lifecycle
[lnk-feedback]: /documentation/articles/iot-hub-devguide-messaging/#message-feedback
[lnk-antispoofing]: /documentation/articles/iot-hub-devguide-messaging/#anti-spoofing-properties
[lnk-compare]: /documentation/articles/iot-hub-compare-event-hubs/

[lnk-devguide-upload]: /documentation/articles/iot-hub-devguide-file-upload/
[lnk-devguide-identities]: /documentation/articles/iot-hub-devguide-identity-registry/
[lnk-devguide-security]: /documentation/articles/iot-hub-devguide-security/
[lnk-devguide-device-twins]: /documentation/articles/iot-hub-devguide-device-twins/
[lnk-devguide-directmethods]: /documentation/articles/iot-hub-devguide-direct-methods/
[lnk-devguide-jobs]: /documentation/articles/iot-hub-devguide-jobs/
[lnk-servicebus-sdk]: https://www.nuget.org/packages/WindowsAzure.ServiceBus
[lnk-eventprocessorhost]: http://blogs.msdn.com/b/servicebus/archive/2015/01/16/event-processor-host-best-practices-part-1.aspx
[lnk-devguide-query-language]: /documentation/articles/iot-hub-devguide-query-language/
[lnk-devguide-endpoints]: /documentation/articles/iot-hub-devguide-endpoints/

[lnk-getstarted-tutorial]: /documentation/articles/iot-hub-csharp-csharp-getstarted/
[lnk-c2d-tutorial]: /documentation/articles/iot-hub-csharp-csharp-c2d/
[lnk-d2c-tutorial]: /documentation/articles/iot-hub-csharp-csharp-process-d2c/

<!---HONumber=Mooncake_0109_2017-->
<!--Update_Description:update wording and add build-in endpoints-->
