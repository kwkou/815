> [AZURE.SELECTOR]
- [Node.js](/documentation/articles/iot-hub-node-node-direct-methods/)
- [C#](/documentation/articles/iot-hub-csharp-node-direct-methods/)

## 介绍
Azure IoT 中心是一项完全托管的服务，可在数百万个 IoT 设备和一个应用程序后端之间实现安全可靠的双向通信。以前的教程（[Get started with IoT Hub]（IoT 中心入门）和 [Send Cloud-to-Device messages with IoT Hub]（使用 IoT 中心发送云到设备的消息））介绍了 IoT 中心的设备到云和云到设备的基本消息传递功能。用户还可以通过 IoT 中心从云调用设备上的非持久方法。这些方法表示与设备进行的请求-答复式交互，类似于 HTTP 调用，因为它们不管是成功还是失败，速度都非常快（在用户指定的超时过后），会让用户知道调用的状态。[Invoke a direct method on a device][lnk-devguide-methods]（调用设备上的直接方法）更详细地介绍了各种方法，指导用户何时使用这些方法，何时使用云到设备的消息。

本教程演示如何：

* 使用 Azure 门户预览创建 IoT 中心，以及如何在 IoT 中心创建设备标识。
* 创建一个模拟设备，以便通过云调用其中的直接方法。
* 创建一个控制台应用程序，以便通过 IoT 中心调用模拟设备上的直接方法。

> [AZURE.NOTE]
目前只能使用 MQTT 协议从连接到 IoT 中心的设备访问直接方法。有关如何转换现有设备应用以使用 MQTT 的说明，请参阅 [MQTT 支持][lnk-devguide-mqtt]一文。
> 
> 




[lnk-devguide-methods]: /documentation/articles/iot-hub-devguide-direct-methods/
[lnk-devguide-mqtt]: /documentation/articles/iot-hub-mqtt-support/

[Send Cloud-to-Device messages with IoT Hub]: /documentation/articles/iot-hub-csharp-csharp-c2d/
[Get started with IoT Hub]: /documentation/articles/iot-hub-node-node-getstarted/

<!---HONumber=Mooncake_1212_2016-->