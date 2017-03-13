<properties
    pageTitle="了解 Azure IoT 中心 MQTT 支持 | Azure"
    description="开发人员指南 - 支持设备使用 MQTT 协议连接到面向设备的 IoT 中心终结点。介绍了 Azure IoT 设备 SDK 中的内置 MQTT 支持。"
    services="iot-hub"
    documentationcenter=".net"
    author="kdotchkoff"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="1d71c27c-b466-4a40-b95b-d6550cf85144"
    ms.service="iot-hub"
    ms.devlang="multiple"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="10/24/2016"
    wacn.date="03/10/2017"
    ms.author="kdotchko" />  


# IoT 中心 MQTT 支持
IoT 中心让设备能够在端口 8883 上使用 [MQTT v3.1.1][lnk-mqtt-org] 协议，或在端口 443 上使用基于 WebSocket 的 MQTT v3.1.1 协议来与 IoT 中心设备终结点通信。IoT 中心要求使用 TLS/SSL 保护所有设备通信（因此，IoT 中心不支持端口 1883 上的非安全连接）。

## 连接到 IoT 中心

设备既可以通过 [Microsoft Azure IoT SDK][lnk-device-sdks] 中的库使用 MQTT 协议连接到 IoT 中心，也可以直接使用 MQTT 协议连接到 IoT 中心。

## 使用设备 SDK
支持 MQTT 协议的[设备 SDK][lnk-device-sdks] 可用于 Java、Node.js、C、C# 和 Python。设备 SDK 使用标准 IoT 中心连接字符串连接到 IoT 中心。若要使用 MQTT 协议，必须将客户端协议参数设置为 **MQTT**。默认情况下，设备 SDK 连接到 **CleanSession** 标志设为 **0** 的 IoT 中心，并使用 **QoS 1** 与 IoT 中心交换消息。

当设备连接到 IoT 中心时，设备 SDK 将提供相关方法，使设备可接收来自 IoT 中心的消息并向其发送消息。

下表包含了每种受支持语言的代码示例链接，并指定了以 MQTT 协议连接到 IoT 中心时要使用的参数。

| 语言 | 协议参数 |
| --- | --- |
| Node.js |azure-iot-device-mqtt |
| Java |IotHubClientProtocol.MQTT |
| C |MQTT\_Protocol |
| C# |TransportType.Mqtt |
| Python |IoTHubTransportProvider.MQTT |

### 将设备应用从 AMQP 迁移到 MQTT
如果使用[设备 SDK][lnk-device-sdks]，则需要在客户端初始化中更改协议参数才可将 AMQP 换用为 MQTT，如上所述。

执行此操作时，请确保检查下列各项：

* AMQP 针对许多条件返回错误，而 MQTT 会终止连接。因此异常处理逻辑可能需要进行一些更改。
* MQTT 在接收[云到设备消息][lnk-messaging]时不支持*拒绝*操作。如果后端应用需要接收来自设备应用的响应，请考虑使用[直接方法][lnk-methods]。

## 直接使用 MQTT 协议
如果设备无法使用设备 SDK，仍可使用 MQTT 协议连接到公共设备终结点。在 **CONNECT** 数据包中，设备应使用以下值：

- **ClientId** 字段使用 **deviceId**。
- “用户名”字段使用 `{iothubhostname}/{device_id}/api-version=2016-11-14`，其中 {iothubhostname} 是 IoT 中心的完整 CName。

    例如，如果 IoT 中心的名称为 **contoso.azure-devices.cn**，设备的名称为 **MyDevice01**，则完整“用户名”字段应包含 `contoso.azure-devices.cn/MyDevice01/api-version=2016-11-14`。
- “密码”字段使用 SAS 令牌。对于 HTTP 和 AMQP 协议，SAS 令牌的格式是相同的：<br/>`SharedAccessSignature sig={signature-string}&se={expiry}&sr={URL-encoded-resourceURI}`。

    有关如何生成 SAS 令牌的详细信息，请参阅[使用 IoT 中心安全令牌][lnk-sas-tokens]的设备部分。

    测试时也可以使用[设备资源管理器][lnk-device-explorer]工具来快速生成可以复制并粘贴到你自己的代码中的 SAS 令牌。

  1. 转到“设备资源管理器”中的“管理”选项卡。
  2. 单击“SAS 令牌”（右上角）。
  3. 在 **SASTokenForm** 上，从“DeviceID”下拉列表中选择你的设备。设置 **TTL**。
  4. 单击“生成”创建令牌。

     所生成的 SAS 令牌具有以下结构：`HostName={your hub name}.azure-devices.cn;DeviceId=javadevice;SharedAccessSignature=SharedAccessSignature sr={your hub name}.azure-devices.net%2Fdevices%2FMyDevice01%2Fapi-version%3D2016-11-14&sig=vSgHBMUG.....Ntg%3d&se=1456481802`。

     此令牌中要用作“密码”字段以便使用 MQTT 进行连接的部分是：`SharedAccessSignature sr={your hub name}.azure-devices.cn%2Fdevices%2FMyDevice01%2Fapi-version%3D2016-11-14&sig=vSgHBMUG.....Ntg%3d&se=1456481802`。

对于 MQTT 连接和断开连接数据包，IoT 中心会在**操作监视**频道发布事件，并提供可帮助对连接问题进行故障排除的其他信息。

### 发送设备到云的消息
成功建立连接后，设备可以使用 `devices/{device_id}/messages/events/` 或 `devices/{device_id}/messages/events/{property_bag}` 作为**主题名称**将消息发送到 IoT 中心。`{property_bag}` 元素可让设备使用 URL 编码格式发送包含其他属性的消息。例如：

		RFC 2396-encoded(<PropertyName1>)=RFC 2396-encoded(<PropertyValue1>)&RFC 2396-encoded(<PropertyName2>)=RFC 2396-encoded(<PropertyValue2>)…

> [AZURE.NOTE] 此 `{property_bag}` 元素使用的编码与 HTTP 协议中用于查询字符串的编码相同。

设备应用还可使用`devices/{device_id}/messages/events/{property_bag}` 作为**遗嘱主题名称**，用于定义要作为遥测消息转发的*遗嘱消息*。

- IoT 中心不支持 QoS 2 消息。如果设备应用使用 **QoS 2** 发布消息，IoT 中心将断开网络连接。
- IoT 中心不会存储保留消息。如果设备发送 **RETAIN** 标志设置为 1 的消息，IoT 中心会在消息中添加 **x-opt-retain** 应用程序属性。在此情况下，IoT 中心不会存储保留消息，而将其传递到后端应用。
- IoT 中心只允许一个设备有一个活动的 MQTT 连接。如果新的 MQTT 连接代表的是同一设备 ID，则会导致 IoT 中心断开现有连接。

有关详细信息，请参阅[消息传送开发人员指南][lnk-messaging]。

### 接收云到设备消息
若要从 IoT 中心接收消息，设备应使用 `devices/{device_id}/messages/devicebound/#` 作为**主题筛选器**来进行订阅。主题筛选器中的多级通配符 **\#** 仅用于允许设备接收主题名称中的其他属性。IoT 中心不允许使用 **\#** 或 **?** 通配符筛选子主题。由于 IoT 中心不是一般用途的发布-订阅消息传送中转站，它仅支持存档的主题名称和主题筛选器。

请注意，在设备成功订阅 `devices/{device_id}/messages/devicebound/#` 主题筛选器表示的设备特定终结点前，设备不会从 IoT 中心收到任何消息。成功建立订阅后，设备仅会开始收到建立订阅后发送给它的云到设备消息。如果设备使用设置为 **0** 的 **CleanSession** 标志连接，会在不同会话中保存订阅。在此情况下，下次使用 **CleanSession 0** 连接时，设备会收到断开连接时发送给它的未处理消息。如果设备使用设置为 **1** 的 **CleanSession** 标志，在订阅其设备终结点前，它不会从 IoT 中心收到任何消息。

如有任何消息属性，IoT 中心将传送包含**主题名称**、`devices/{device_id}/messages/devicebound/` 或 `devices/{device_id}/messages/devicebound/{property_bag}` 的消息。`{property_bag}` 包含 URL 编码的消息属性键/值对。属性包中只包含应用程序属性和用户可设置的系统属性（例如 **messageId** 或 **correlationId**）。系统属性名称具有前缀 **$**，但应用程序属性使用没有前缀的原始属性名称。

当设备应用使用 **QoS 2** 订阅主题时，IoT 中心将在 **SUBACK** 包中授予最高 QoS 级别 1。之后，IoT 中心会使用 QoS 1 将消息传送到设备。

### 检索设备孪生的属性

首先，设备订阅 `$iothub/twin/res/#`，接收操作的响应。然后，它向主题 `$iothub/twin/GET/?$rid={request id}` 发送一条空消息，其中包含 **request id** 的填充值。随后，服务将使用与请求相同的**请求 ID**，发送包含主题 `$iothub/twin/res/{status}/?$rid={request id}` 的设备孪生数据的响应消息。

request id 可以是消息属性值的任何有效值（如 [IoT 中心消息传送开发人员指南][lnk-messaging]中所述），且需要验证确保状态是整数。响应正文将包含设备孪生的属性部分：

标识注册表项的正文限制为“属性”成员，例如，

        {
            "properties": {
                "desired": {
                    "telemetrySendFrequency": "5m",
                    "$version": 12
                },
                "reported": {
                    "telemetrySendFrequency": "5m",
                    "batteryLevel": 55,
                    "$version": 123
                }
            }
        }

可能的状态代码为：

|状态 | 说明 |
| ----- | ----------- |
| 200 | 成功 |
| 429 | 请求过多（限制），如 [IoT 中心限制][lnk-quotas]中所述 |
| 5** | 服务器错误 |

有关详细信息，请参阅[设备孪生开发人员指南][lnk-devguide-twin]。

### 更新设备孪生的报告属性

以下顺序说明了设备如何在 IoT 中心更新设备孪生中报告的属性：

1. 设备必须先订阅 `$iothub/twin/res/#` 主题，以便接收 IoT 中心的操作响应。

1. 设备发送一条消息，其中包含 `$iothub/twin/PATCH/properties/reported/?$rid={request id}` 主题的设备孪生更新。该消息包含**请求 ID** 值。

1. 然后，服务发送一条响应消息，其中包含主题 `$iothub/twin/res/{status}/?$rid={request id}` 的已报告属性集合的全新 ETag 值。该响应消息使用与该请求相同的**请求 ID**。

请求消息正文包含 JSON 文档，该文档提供报告属性的新值（不可修改任何其他属性或元数据）。JSON 文档中的每个成员均会更新或添加设备孪生文档中的相应成员。设置为 `null` 的成员会从包含的对象中删除成员。例如：

        {
            "telemetrySendFrequency": "35m",
            "batteryLevel": 60
        }

可能的状态代码为：

|状态 | 说明 |
| ----- | ----------- |
| 200 | 成功 |
| 400 | 错误的请求。格式不正确的 JSON |
| 429 | 请求过多（限制），如 [IoT 中心限制][lnk-quotas]中所述 |
| 5** | 服务器错误 |

有关详细信息，请参阅[设备孪生开发人员指南][lnk-devguide-twin]。

### 接收所需属性更新通知

设备连接时，IoT 中心会向主题 `$iothub/twin/PATCH/properties/desired/?$version={new version}` 发送通知，内附解决方案后端执行的更新内容。例如：

        {
            "telemetrySendFrequency": "5m",
            "route": null
        }

对于属性更新，`null` 值表示正在删除 JSON 对象成员。

> [AZURE.IMPORTANT] IoT 中心在仅在连接设备时才会生成更改通知，请确保实现[设备重新连接流][lnk-devguide-twin-reconnection]，让 IoT 中心和设备应用之间的所需属性保持同步。

有关详细信息，请参阅[设备孪生开发人员指南][lnk-devguide-twin]。

### 响应直接方法

首先，设备需要订阅 `$iothub/methods/POST/#`。IoT 中心向主题 `$iothub/methods/POST/{method name}/?$rid={request id}` 发送方法请求，其中包含有效的 JSON 或空正文。

设备会向主题 `$iothub/methods/res/{status}/?$rid={request id}` 发送具有有效 JSON 或空正文的消息进行响应，其中 **request id** 必须与请求消息中的对应项匹配，**status** 必须是整数。

有关详细信息，请参阅[直接方法开发人员指南][lnk-methods]。

### 其他注意事项
最后，如果需要自定义云端的 MQTT 协议行为，则应检查 [Azure IoT 协议网关][lnk-azure-protocol-gateway]，可以通过它部署直接与 IoT 中心连接的高性能自定义协议网关。Azure IoT 协议网关可让你自定义设备协议，以适应要重建的 MQTT 部署或其他自定义协议。但是，这种方法要求运行并使用自定义协议网关。

## 后续步骤
有关详细信息，请参阅 IoT 中心开发人员指南中的 [MQTT 支持相关说明][lnk-mqtt-devguide]。

若要了解有关 MQTT 协议的详细信息，请参阅 [MQTT 文档][lnk-mqtt-docs]。

若要深入了解如何规划 IoT 中心部署，请参阅：

- [Azure IoT 认证设备目录][lnk-devices]
- [支持其他协议][lnk-protocols]
- [与事件中心比较][lnk-compare]
- [缩放、HA 和 DR][lnk-scaling]

若要进一步探索 IoT 中心的功能，请参阅：

- [IoT 中心开发人员指南][lnk-devguide]
- [使用 IoT 网关 SDK 模拟设备][lnk-gateway]

[lnk-device-sdks]: https://github.com/Azure/azure-iot-sdks
[lnk-mqtt-org]: http://mqtt.org/
[lnk-mqtt-docs]: http://mqtt.org/documentation
[lnk-sample-node]: https://github.com/Azure/azure-iot-sdk-node/blob/master/device/samples/simple_sample_device.js
[lnk-sample-c]: https://github.com/Azure/azure-iot-sdk-c/tree/master/iothub_client/samples/iothub_client_sample_mqtt
[lnk-sample-csharp]: https://github.com/Azure/azure-iot-sdk-csharp/tree/master/device/samples
[lnk-sample-python]: https://github.com/Azure/azure-iot-sdk-python/tree/master/device/samples
[lnk-device-explorer]: https://github.com/Azure/azure-iot-sdk-csharp/blob/master/tools/DeviceExplorer
[lnk-sas-tokens]: /documentation/articles/iot-hub-devguide-security/#use-sas-tokens-in-a-device-client
[lnk-mqtt-devguide]: /documentation/articles/iot-hub-devguide-messaging/#notes-on-mqtt-support
[lnk-azure-protocol-gateway]: /documentation/articles/iot-hub-protocol-gateway/

[lnk-devices]: /documentation/articles/iot-hub-tested-configurations/
[lnk-protocols]: /documentation/articles/iot-hub-protocol-gateway/
[lnk-compare]: /documentation/articles/iot-hub-compare-event-hubs/
[lnk-scaling]: /documentation/articles/iot-hub-scaling/
[lnk-devguide]: /documentation/articles/iot-hub-devguide/
[lnk-gateway]: /documentation/articles/iot-hub-linux-gateway-sdk-simulated-device/

[lnk-methods]: /documentation/articles/iot-hub-devguide-direct-methods/
[lnk-messaging]: /documentation/articles/iot-hub-devguide-messaging/
[lnk-quotas]: /documentation/articles/iot-hub-devguide-quotas-throttling/
[lnk-devguide-twin-reconnection]: /documentation/articles/iot-hub-devguide-device-twins/#device-reconnection-flow
[lnk-devguide-twin]: /documentation/articles/iot-hub-devguide-device-twins/

<!---HONumber=Mooncake_0306_2017-->
<!--Update_Description:add sections of Receiving c2d messages, Retrieving a device twin's properties, 
update device twin's reported properties, Receiving desired properties update notifications, Respond to a direct method-->