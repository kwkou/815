<properties
 pageTitle="IoT 中心 MQTT 支持 | Azure"
 description="介绍 IoT 中心级别的 MQTT 支持"
 services="iot-hub"
 documentationCenter=".net"
 authors="dominicbetts"
 manager="timlt"
 editor=""/>  


<tags
 ms.service="iot-hub"
 ms.devlang="multiple"
 ms.topic="article"
 ms.tgt_pltfrm="na"
 ms.workload="na"
 ms.date="10/24/2016"
 wacn.date="01/04/2017"
 ms.author="kdotchko"/>  


# IoT 中心 MQTT 支持

IoT 中心允许设备在端口 8883 上使用 [MQTT v3.1.1][lnk-mqtt-org] 协议来与 IoT 中心设备终结点通信。IoT 中心要求使用 TLS/SSL 保护所有的设备通信。

## 连接到 IoT 中心

设备既可以通过 [Microsoft Azure IoT SDK][lnk-device-sdks] 中的库使用 MQTT 协议连接到 IoT 中心，也可以直接使用 MQTT 协议连接到 IoT 中心。

## 使用设备客户端 SDK

支持 MQTT 协议的[设备客户端 SDK][lnk-device-sdks] 提供 Java、Node.js、C 和 C# 版本。设备客户端 SDK 使用标准 IoT 中心连接字符串来连接到 IoT 中心。若要使用 MQTT 协议，必须将客户端协议参数设置为 **MQTT**。默认情况下，设备客户端 SDK 连接到 **CleanSession** 标志设置为 **0** 的 IoT 中心，并使用 **QoS 1** 来与 IoT 中心交换消息。

当设备连接到 IoT 中心时，设备客户端 SDK 将提供方法让设备在 IoT 中心发送和接收消息。

下表包含了每种受支持语言的代码示例链接，并指定了以 MQTT 协议连接到 IoT 中心时要使用的参数。

| 语言 | 协议参数 |
| --- | --- |
| [Node.js][lnk-sample-node] |azure-iot-device-mqtt |
| [Java][lnk-sample-java] |IotHubClientProtocol.MQTT |
| C |MQTT\_Protocol |
| C# |TransportType.Mqtt |
| Python |IoTHubTransportProvider.MQTT |

### 将设备应用从 AMQP 迁移到 MQTT
如果使用[设备客户端 SDK][lnk-device-sdks]，则从使用 AMQP 切换到 MQTT 需要在客户端初始化中更改协议参数，如上所述。

执行此操作时，请确保检查下列各项：

* AMQP 针对许多条件返回错误，而 MQTT 会终止连接。因此异常处理逻辑可能需要进行一些更改。
* MQTT 在接收 [C2D 消息][lnk-messaging]时不支持*拒绝*操作。如果后端需要从设备应用接收响应，请考虑使用[直接方法][lnk-methods]。
* 当前 MQTT 在 WebSocket 上不可用。

## 直接使用 MQTT 协议
如果设备无法使用设备客户端 SDK，仍可使用 MQTT 协议连接到公共设备终结点。在 **CONNECT** 数据包中，设备应使用以下值：

- **ClientId** 字段使用 **deviceId**。
- “用户名”字段使用 `{iothubhostname}/{device_id}`，其中 {iothubhostname} 是 IoT 中心的完整 CName。

    例如，如果 IoT 中心的名称为 **contoso.azure-devices.cn**，设备的名称为 **MyDevice01**，则完整“用户名”字段应包含 `contoso.azure-devices.cn/MyDevice01`。
- “密码”字段使用 SAS 令牌。对于 HTTP 和 AMQP 协议，SAS 令牌的格式是相同的：<br/>`SharedAccessSignature sig={signature-string}&se={expiry}&sr={URL-encoded-resourceURI}`。

    有关如何生成 SAS 令牌的详细信息，请参阅[使用 IoT 中心安全令牌][lnk-sas-tokens]的设备部分。
    
    测试时也可以使用设备资源管理器工具来快速生成可以复制并粘贴到自己的代码中的 SAS 令牌。
    
    1. 转到设备资源管理器中的“管理”选项卡。
    2. 单击“SAS 令牌”（右上角）。
    3. 在 **SASTokenForm** 上，从“DeviceID”下拉列表中选择你的设备。设置 **TTL**。
    4. 单击“生成”创建令牌。
    
    所生成的 SAS 令牌具有以下结构：
    `HostName={your hub name}.azure-devices.cn;DeviceId=javadevice;SharedAccessSignature=SharedAccessSignature sr={your hub name}.azure-devices.cn%2fdevices%2fMyDevice01&sig=vSgHBMUG.....Ntg%3d&se=1456481802`。

    此令牌中要用作“密码”字段以便使用 MQTT 进行连接的部分是：
    `SharedAccessSignature sr={your hub name}.azure-devices.cn%2fdevices%2fyDevice01&sig=vSgHBMUG.....Ntg%3d&se=1456481802g%3d&se=1456481802`。

对于 MQTT 连接和断开连接数据包，IoT 中心将在**操作监视**通道上发出事件。

### 将消息发送到 IoT 中心

成功建立连接后，设备可以使用 `devices/{device_id}/messages/events/` 或 `devices/{device_id}/messages/events/{property_bag}` 作为**主题名称**将消息发送到 IoT 中心。`{property_bag}` 元素可让设备使用 URL 编码格式发送包含其他属性的消息。例如：

		RFC 2396-encoded(<PropertyName1>)=RFC 2396-encoded(<PropertyValue1>)&RFC 2396-encoded(<PropertyName2>)=RFC 2396-encoded(<PropertyValue2>)…

> [AZURE.NOTE] 此 `{property_bag}` 元素使用的编码与 HTTP 协议中用于查询字符串的编码相同。

设备客户端应用程序还可以使用 `devices/{device_id}/messages/events/{property_bag}` 作为 **Will 主题名称**，来定义要以遥测消息形式转发的 *Will 消息*。

### 接收消息

若要从 IoT 中心接收消息，设备应使用 `devices/{device_id}/messages/devicebound/#”` 作为**主题筛选器**来进行订阅。如有任何消息属性，IoT 中心将传送包含**主题名称**、`devices/{device_id}/messages/devicebound/` 或 `devices/{device_id}/messages/devicebound/{property_bag}` 的消息。`{property_bag}` 包含 URL 编码的消息属性键/值对。属性包中只包含应用程序属性和用户可设置的系统属性（例如 **messageId** 或 **correlationId**）。系统属性名称具有前缀 **$**，但应用程序属性使用没有前缀的原始属性名称。

## 后续步骤

有关详细信息，请参阅“Azure IoT 中心开发人员指南”中的[有关 MQTT 支持的说明][lnk-mqtt-devguide]。

若要了解有关 MQTT 协议的详细信息，请参阅 [MQTT 文档][lnk-mqtt-docs]。

若要深入了解如何规划 IoT 中心部署，请参阅：

- [支持的设备][lnk-devices]
- [支持其他协议][lnk-protocols]
- [与事件中心比较][lnk-compare]
- [缩放、HA 和 DR][lnk-scaling]

若要进一步探索 IoT 中心的功能，请参阅：

- [开发人员指南][lnk-devguide]
- [使用网关 SDK 模拟设备][lnk-gateway]

[lnk-device-sdks]: https://github.com/Azure/azure-iot-sdks/blob/master/readme.md
[lnk-mqtt-org]: http://mqtt.org/
[lnk-mqtt-docs]: http://mqtt.org/documentation
[lnk-sample-node]: https://github.com/Azure/azure-iot-sdks/blob/develop/node/device/samples/simple_sample_device.js
[lnk-sample-java]: https://github.com/Azure/azure-iot-sdks/blob/develop/java/device/samples/send-receive-sample/src/main/java/samples/com/microsoft/azure/iothub/SendReceive.java
[lnk-sample-c]: https://github.com/Azure/azure-iot-sdks/tree/master/c/iothub_client/samples/iothub_client_sample_mqtt
[lnk-sample-csharp]: https://github.com/Azure/azure-iot-sdks/tree/master/csharp/device/samples
[lnk-sample-python]: https://github.com/Azure/azure-iot-sdks/tree/master/python/device/samples
[lnk-device-explorer]: https://github.com/Azure/azure-iot-sdks/blob/master/tools/DeviceExplorer/readme.md
[lnk-sas-tokens]: /documentation/articles/iot-hub-devguide-security/#using-sas-tokens-as-a-device
[lnk-mqtt-devguide]: /documentation/articles/iot-hub-devguide-messaging/#notes-on-mqtt-support

[lnk-devices]: /documentation/articles/iot-hub-tested-configurations/
[lnk-protocols]: /documentation/articles/iot-hub-protocol-gateway/
[lnk-compare]: /documentation/articles/iot-hub-compare-event-hubs/
[lnk-scaling]: /documentation/articles/iot-hub-scaling/
[lnk-devguide]: /documentation/articles/iot-hub-devguide/
[lnk-gateway]: /documentation/articles/iot-hub-linux-gateway-sdk-simulated-device/

[lnk-methods]: /documentation/articles/iot-hub-devguide-direct-methods/
[lnk-messaging]: /documentation/articles/iot-hub-devguide-messaging/

<!---HONumber=Mooncake_Quality_Review_1230_2016-->