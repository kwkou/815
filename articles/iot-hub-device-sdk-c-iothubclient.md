<properties
	pageTitle="适用于 C 语言的 Azure IoT 设备 SDK - IoTHubClient | Azure"
	description="了解如何使用适用于 C 语言的 Azure IoT 设备 SDK 中的 IoTHubClient 库"
	services="iot-hub"
	documentationCenter=""
	authors="olivierbloch"
	manager="timlt"
	editor=""/>

<tags
     ms.service="iot-hub"
     ms.date="05/17/2016"
     wacn.date="07/04/2016"/>

# 适用于 C 语言的 Azure IoT 设备 SDK – 有关 IoTHubClient 的详细信息

本系列教程的[第一篇文章](/documentation/articles/iot-hub-device-sdk-c-intro/)介绍了**适用于 C 语言的 Azure IoT 设备 SDK**。该文章已说明 SDK 中有两个体系结构层。底层是 **IoTHubClient** 库，用于直接管理与 IoT 中心的通信。另有一个**序列化程序**库，它构建在 SDK 的顶部，可提供序列化服务。在本文中，我们将提供有关 **IoTHubClient** 库的更多详细信息。

前一篇文章介绍了如何使用 **IoTHubClient** 库将事件发送到 IoT 中心及接收消息。本文将扩大讨论范围，介绍**较低级别 API**，说明如何更精确地管理发送和接收数据的*时机*。此外将说明如何使用 **IoTHubClient** 库中的属性处理功能，将属性附加到事件（以及从消息中检索属性）。最后，进一步说明如何以不同的方式来处理从 IoT 中心收到的消息。

本文的总结部分提供了多个其他主题，包括有关设备凭据的详细信息以及如何通过配置选项更改 **IoTHubClient** 的行为。

我们将使用 **IoTHubClient** SDK 示例来解释这些主题。如果想要继续，请参阅适用于 C 的 Azure IoT 设备 SDK 中随附的 **iothub\_client\_sample\_http** 和 **iothub\_client\_sample\_amqp** 应用程序。以下部分所述的所有内容都将通过这些示例来演示。

## 较低级别 API

前一篇文章介绍了 **iothub\_client\_sample\_amqp** 应用程序上下文中 **IotHubClient** 的基本操作。例如，该文章说明了如何使用此代码来初始化库。

```
IOTHUB_CLIENT_HANDLE iotHubClientHandle;
iotHubClientHandle = IoTHubClient_CreateFromConnectionString(connectionString, AMQP_Protocol);
```

此外，还说明了如何使用此函数调用来发送事件。

```
IoTHubClient_SendEventAsync(iotHubClientHandle, message.messageHandle, SendConfirmationCallback, &message);
```

该文章还说明了如何通过注册回调函数来接收消息。

```
int receiveContext = 0;
IoTHubClient_SetMessageCallback(iotHubClientHandle, ReceiveMessageCallback, &receiveContext);
```

该文章同时演示了如何使用如下代码释放资源。

```
IoTHubClient_Destroy(iotHubClientHandle);
```

但是每个 API 都有伴随函数：

-   IoTHubClient\_LL\_CreateFromConnectionString

-   IoTHubClient\_LL\_SendEventAsync

-   IoTHubClient\_LL\_SetMessageCallback

-   IoTHubClient\_LL\_Destroy


这些函数的 API 名称中包含“LL”。除此之外，其中每个函数的参数都与其非 LL 的对等项相同。但是，这些函数的行为有一个重要的差异。

当你调用 **IoTHubClient\_CreateFromConnectionString** 时，基础库将创建在后台运行的新线程。此线程将事件发送到 IoT 中心以及从 IoT 中心接收消息。使用“LL”API 时不会创建此类线程。创建后台线程是为了给开发人员提供方便。你无需担心如何明确与 IoT 中心相互发送和接收消息 -- 此操作将在后台自动进行。相比之下，“LL”API 可让你根据需要明确控制与 IoT 中心的通信。

为了更好地理解这一点，让我们看一个示例：

当调用 **IoTHubClient\_SendEventAsync** 时，其实是将事件放入缓冲区中。调用 **IoTHubClient\_CreateFromConnectionString** 时创建的后台线程将持续监视此缓冲区，并将它包含的任何数据发送到 IoT 中心。这些操作在主线程执行其他工作的同时在后台进行。

同样，当你使用 **IoTHubClient\_SetMessageCallback** 注册消息的回调函数时，则会指示 SDK 在收到消息时让后台线程调用回调函数（独立于主线程）。

“LL”API 不会创建后台线程。必须调用一个新 API 明确地与 IoT 中心之间相互发送和接收数据。以下示例就将此进行演示。

SDK 中随附的 **Iothub\_client\_sample\_http** 应用程序演示了较低级别 API。在该示例中，使用如下代码将事件发送到 IoT 中心：

```
EVENT_INSTANCE message;
sprintf_s(msgText, sizeof(msgText), "Message_%d_From_IoTHubClient_LL_Over_HTTP", i);
message.messageHandle = IoTHubMessage_CreateFromByteArray((const unsigned char*)msgText, strlen(msgText));

IoTHubClient_LL_SendEventAsync(iotHubClientHandle, message.messageHandle, SendConfirmationCallback, &message)
```

前三行创建消息，最后一行发送事件。但是，如前所述，“发送”事件只是将数据放置于缓冲区。当我们调用 **IoTHubClient\_LL\_SendEventAsync** 时，不会在网络上传输任何内容。若要实际将数据引入 IoT 中心，必须调用 **IoTHubClient\_LL\_DoWork**，如以下示例所示：

```
while (1)
{
    IoTHubClient_LL_DoWork(iotHubClientHandle);
    ThreadAPI_Sleep(1000);
}
```

此代码（来自 **iothub\_client\_sample\_http** 应用程序）反复调用 **IoTHubClient\_LL\_DoWork**。每次 **IoTHubClient\_LL\_DoWork** 被调用时，会将某些事件从缓冲区发送到 IoT 中心，并检索正在发送到设备的排队消息。对于后一种情况，意味着如果已注册消息的回调函数，则调用回调（假设所有消息都已加入队列）。使用如下代码来注册此类回调函数：

```
IoTHubClient_LL_SetMessageCallback(iotHubClientHandle, ReceiveMessageCallback, &receiveContext)
```

经常在循环中调用 **IoTHubClient\_LL\_DoWork** 的原因是每次调用它时，它都会将*一些*缓冲的事件发送到 IoT 中心，并检索设备的*下一个*排队消息。每次调用并不保证发送所有缓冲的事件或检索所有排队的消息。如果你想要发送缓冲区中的所有事件，并继续进行其他处理，可以使用如下代码来替换此循环：

```
IOTHUB_CLIENT_STATUS status;

while ((IoTHubClient_LL_GetSendStatus(iotHubClientHandle, &status) == IOTHUB_CLIENT_OK) && (status == IOTHUB_CLIENT_SEND_STATUS_BUSY))
{
    IoTHubClient_LL_DoWork(iotHubClientHandle);
    ThreadAPI_Sleep(1000);
}
```

此代码将一直调用 **IoTHubClient\_LL\_DoWork**，直到缓冲区中的所有事件都发送到 IoT 中心为止。请注意，这并不意味着已接收所有已排队的消息。部分原因是检查“所有”消息的确定性并不如操作那样强。如果检索了“所有”消息，但随即将另一个消息发送到设备，会发生什么情况？ 更好的处理方法是使用设定的超时。例如，每次调用消息回调函数时，可以重置计时器。例如，如果在过去 *X* 秒内未收到任何消息，可以接着编写逻辑来继续处理。

当你完成引入事件和接收消息时，请务必调用相应的函数来清理资源。

```
IoTHubClient_LL_Destroy(iotHubClientHandle);
```

基本上，只有一组 API 使用后台线程来发送和接收数据，而另一组 API 不会使用后台线程来执行相同的操作。许多开发人员可能偏好非 LL API，但是当他们想要明确控制网络传输时，较低级别 API 就很有用。例如，有些设备会收集各时间段的数据，并且只按指定的时间间隔引入事件（例如，每小时一次或每天一次）。较低级别 API 可以在与 IoT 中心之间发送和接收数据时提供明确控制的能力。还有一些人纯粹偏好较低级别 API 提供的简单性。所有操作都发生在主线程上，而不是有些操作在后台发生。

无论选择哪种模型，都必须与使用的 API 相一致。如果你首先调用 **IoTHubClient\_LL\_CreateFromConnectionString** 开始，请务必只对任何后续工作使用相应的较低级别 API：

-   IoTHubClient\_LL\_SendEventAsync

-   IoTHubClient\_LL\_SetMessageCallback

-   IoTHubClient\_LL\_Destroy

-   IoTHubClient\_LL\_DoWork

相反的情况也成立。如果首先调用 **IoTHubClient\_CreateFromConnectionString**，请使用非 LL API 进行任何其他处理。

在适用于 C 的 Azure IoT 设备 SDK 中，查看 **iothub\_client\_sample\_http** 应用程序是否有较低级别 API 的完整示例。有关非 LL API 的完整示例，请参考 **Iothub\_client\_sample\_amqp** 应用程序。

## 属性处理

在介绍发送数据时，我们多次提到了消息正文。例如，假设有以下代码：

```
EVENT_INSTANCE message;
sprintf_s(msgText, sizeof(msgText), "Hello World");
message.messageHandle = IoTHubMessage_CreateFromByteArray((const unsigned char*)msgText, strlen(msgText));
IoTHubClient_LL_SendEventAsync(iotHubClientHandle, message.messageHandle, SendConfirmationCallback, &message)
```

此示例将包含文本“Hello World”的消息发送到 IoT 中心。 但是，IoT 中心也允许将属性附加到每个消息。这些属性是可附加到消息的名称/值对。例如，我们可以修改上述代码，以将属性附加到消息：

```
MAP_HANDLE propMap = IoTHubMessage_Properties(message.messageHandle);
sprintf_s(propText, sizeof(propText), "%d", i);
Map_AddOrUpdate(propMap, "SequenceNumber", propText);
```

首先先调用 **IoTHubMessage\_Properties**，然后将消息的句柄传递给它。返回的结果是 **MAP\_HANDLE** 引用，它可让我们开始添加属性。后一项操作通过调用 **Map\_AddOrUpdate**（使用对 MAP\_HANDLE、属性名称和属性值的引用）来实现。使用此 API 可需要任意数目的属性。

从**事件中心**读取事件时，接收方可以枚举属性并检索其对应值。例如，在 .NET 中，这可以通过访问 [EventData 对象中的属性集合](https://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.eventdata.properties.aspx)来实现。

在上述示例中，我们已将属性附加到发送给 IoT 中心的事件。属性也可以附加到从 IoT 中心接收的消息。如果想要从消息检索属性，可以在消息回调函数中使用如下所示的代码：

```
static IOTHUBMESSAGE_DISPOSITION_RESULT ReceiveMessageCallback(IOTHUB_MESSAGE_HANDLE message, void* userContextCallback)
{
    // Retrieve properties from the message
    MAP_HANDLE mapProperties = IoTHubMessage_Properties(message);
    if (mapProperties != NULL)
    {
        const char*const* keys;
        const char*const* values;
        size_t propertyCount = 0;
        if (Map_GetInternals(mapProperties, &keys, &values, &propertyCount) == MAP_OK)
        {
            if (propertyCount > 0)
            {
                printf("Message Properties:\r\n");
                for (size_t index = 0; index < propertyCount; index++)
                {
                    printf("\tKey: %s Value: %s\r\n", keys[index], values[index]);
                }
                printf("\r\n");
            }
        }
    }
}
```

调用 **IoTHubMessage\_Properties** 会返回 **MAP\_HANDLE** 引用。然后我们将该引用传递给 **Map\_GetInternals**，以获取对名称/值对数组的引用（以及属性的计数）。此时可以很轻松地枚举属性以获取所需的值。

无需在应用程序中使用属性。但是，如果需要在事件中设置属性或者从消息中检索属性，使用 **IoTHubClient** 库很轻松就能做到。

## 消息处理

如前所述，当 IoT 中心发出的消息抵达时，**IoTHubClient** 库将调用注册的回调函数来做出响应。有必要进一步了解此函数的一个返回参数。以下是 **iothub\_client\_sample\_http** 示例应用程序中回调函数的摘录：

```
static IOTHUBMESSAGE_DISPOSITION_RESULT ReceiveMessageCallback(IOTHUB_MESSAGE_HANDLE message, void* userContextCallback)
{
    return IOTHUBMESSAGE_ACCEPTED;
}
```

请注意，返回类型为 **IOTHUBMESSAGE\_DISPOSITION\_RESULT**，本特定案例将返回 **IOTHUBMESSAGE\_ACCEPTED**。此函数还可能返回其他值，这些值将改变 **IoTHubClient** 库响应消息回调的方式。选项如下。

-   **IOTHUBMESSAGE\_ACCEPTED** - 消息已成功处理。**IoTHubClient** 库将不对同一消息再次调用回调函数。

-   **IOTHUBMESSAGE\_REJECTED** - 未处理消息，且将来也不会处理。**IoTHubClient** 库不应该对同一消息再次调用回调函数。

-   **IOTHUBMESSAGE\_ABANDONED** - 消息未成功处理，但 **IoTHubClient** 库应该对同一消息再次调用回调函数。

对于前两个返回代码，**IoTHubClient** 库会将消息发送到 IoT 中心，指示应该从设备队列中删除消息且不再传送。最终结果一样（从设备队列删除消息），但仍记录是已接受还是已拒绝消息。对于可听取反馈并了解设备是已接受还是拒绝特定消息的消息发送者而言，记录这种区分信息的功能非常有用。

在最后一个案例中，消息也会发送到 IoT 中心，但指示应重新传送消息。如果你遇到了错误但想要再次尝试处理消息，通常会放弃消息。相比之下，当你遇到不可恢复的错误（或者只是决定你不想要处理消息）时，拒绝消息是适当的方式。

在任何情况下，请留意不同的返回代码，以便能够推测 **IoTHubClient** 库的行为。

## 替代设备凭据

如前所述，使用 **IoTHubClient** 库时，首先必须使用如下所示的调用来获取 **IOTHUB\_CLIENT\_HANDLE**：

```
IOTHUB_CLIENT_HANDLE iotHubClientHandle;
iotHubClientHandle = IoTHubClient_CreateFromConnectionString(connectionString, AMQP_Protocol);
```

**IoTHubClient\_CreateFromConnectionString** 的参数是设备的连接字符串，以及用于指明我们在与 IoT 中心通信时要使用的协议的参数。该连接字符串的格式如下：

```
HostName=IOTHUBNAME.IOTHUBSUFFIX;DeviceId=DEVICEID;SharedAccessKey=SHAREDACCESSKEY
```

此字符串包含四个信息片段：IoT 中心名称、IoT 中心后缀、设备 ID 和共享访问密钥。当你在 Azure 门户中创建 IoT 中心实例时，可以获取 IoT 中心的完全限定域名 (FQDN) - 它提供了 IoT 中心名称（FQDN 的第一个部分）和 IoT 中心后缀（FQDN 的其余部分）。使用 IoT 中心注册设备时，可以获取设备 ID 和共享访问密钥（如[前一篇文章](/documentation/articles/iot-hub-device-sdk-c-intro/)中所述）。

**IoTHubClient\_CreateFromConnectionString** 提供了初始化库的方式。如果需要，你可以使用其中的每个参数而不是连接字符串来创建新的 **IOTHUB\_CLIENT\_HANDLE**。使用以下代码即可实现此目的：

```
IOTHUB_CLIENT_CONFIG iotHubClientConfig;
iotHubClientConfig.iotHubName = "";
iotHubClientConfig.deviceId = "";
iotHubClientConfig.deviceKey = "";
iotHubClientConfig.iotHubSuffix = "";
iotHubClientConfig.protocol = HTTP_Protocol;
IOTHUB_CLIENT_HANDLE iotHubClientHandle = IoTHubClient_LL_Create(&iotHubClientConfig);
```

结果将与使用 **IoTHubClient\_CreateFromConnectionString** 相同。

显然，你更想要使用 **IoTHubClient\_CreateFromConnectionString**，而不是这种更繁琐的初始化方法。但请记住，当你在 IoT 中心注册设备时，获得的是设备 ID 和设备密钥（而不是连接字符串）。[前一篇文章](/documentation/articles/iot-hub-device-sdk-c-intro/)中介绍的**设备管理器** SDK 工具使用 **Azure IoT 服务 SDK** 中的库，从设备ID、设备密钥和 IoT 中心主机名创建连接字符串。因此调用 **IoTHubClient\_LL\_Create** 可能更好，因为这样可以免除生成连接字符串的步骤。使用任何一种方法都很方便。

## 配置选项

到目前为止，有关 **IoTHubClient** 库工作方式的所有介绍内容都反映了其默认行为。但是，你可以设置几个选项来更改库的工作方式。此目的可以利用 **IoTHubClient\_LL\_SetOption** API 来实现。请看以下示例：

```
unsigned int timeout = 30000;
IoTHubClient_LL_SetOption(iotHubClientHandle, "timeout", &timeout);
```

有一些常用的选项：

-   **SetBatching**（布尔值）– 如果为 **true**，则将数据分批发送到 IoT 中心。如果为 **false**，则逐个发送消息。默认值为 **false**。

-   **Timeout**（无符号整数）– 此值以毫秒表示。如果发送 HTTP 请求或接收响应所花费的时间超过此时间，即表示连接超时。

batching 选项非常重要。默认情况下，库将逐个引入事件（单个事件是你传递给 **IoTHubClient\_LL\_SendEventAsync** 的任何内容）。如果 batching 选项为 **true**，库将尽可能多地从缓冲区中收集事件（上限为 IoT 中心接受的消息大小上限）。事件批在单个 HTTP 调用中发送到 IoT 中心（单个事件已捆绑到 JSON 数组中）。启用批处理通常可以大幅提升性能，因为网络往返时间会缩减。不过，这也会明显减少带宽，因为你要在事件批中发送一组 HTTP 标头，而不是针对每个事件发送一组标头。除非有使用其他方式的特殊理由，否则建议你启用批处理。

## 后续步骤

本文详细探讨了**适用于 C 语言的 Azure IoT 设备 SDK** 中的 **IoTHubClient** 库的行为。参考这些信息可以充分了解 **IoTHubClient** 库的功能。[下一篇文章](/documentation/articles/iot-hub-device-sdk-c-serializer/)将提供有关**序列化程序**库的类似详细信息。

<!---HONumber=Mooncake_0307_2016-->