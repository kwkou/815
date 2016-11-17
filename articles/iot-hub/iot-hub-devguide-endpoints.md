<properties
 pageTitle="开发人员指南 - IoT 中心终结点 | Azure"
 description="Azure IoT 中心开发人员指南 - 有关 IoT 中心终结点的参考信息"
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
 ms.date="09/30/2016"
 wacn.date="11/07/2016" 
 ms.author="dobett"/>  


# 参考 - IoT 中心终结点

## IoT 中心终结点列表

Azure IoT 中心属于多租户服务，向各种执行组件公开功能。下图显示了 IoT 中心公开的各种终结点。

![IoT 中心终结点][img-endpoints]

以下是终结点的说明：

* **资源提供程序**。IoT 中心资源提供程序公开 [Azure Resource Manager][lnk-arm] 界面，可让 Azure 订阅所有者创建和删除 IoT 中心以及更新 IoT 中心属性。IoT 中心属性可管理[中心级别的安全策略][lnk-accesscontrol]，相对于设备级别的访问控制和云到设备及设备到云消息传送的功能选项。资源提供程序还可让你[导出设备标识][lnk-importexport]。
* **设备标识管理**。每个 IoT 中心公开一组用于管理设备标识的 HTTP REST 终结点（创建、检索、更新和删除）。[设备标识][lnk-device-identities]用于设备身份验证和访问控制。
* **设备克隆管理**。每个 IoT 中心都会公开一组面向服务的 HTTP REST 终结点，用于查询和更新[设备克隆][lnk-twins]（更新标记和属性）。
* **作业管理**。每个 IoT 中心都会公开一组面向服务的 HTTP REST 终结点，用于查询和管理[作业][lnk-jobs]。
* **设备终结点**。对于设备标识注册表中预配的每个设备，IoT 中心公开设备可用来发送和接收消息的一组终结点：
    - *发送设备到云的消息* 。使用此终结点[发送设备到云的消息][lnk-d2c]。
    - *接收云到设备的消息* 设备使用此终结点接收目标[云到设备的消息][lnk-c2d]。
    - *启动文件上载* 。设备使用此终结点接收来自 IoT 中心的 Azure 存储 SAS URI，以便[上载文件][lnk-upload]。
    - *检索和更新克隆的属性* 。设备使用此终结点访问其[设备克隆][lnk-twins]的属性。
    - *接收直接方法请求* 。设备使用此终结点侦听[直接方法][lnk-methods]的请求。

    这些终结点使用 [MQTT v3.1.1][lnk-mqtt]、HTTP 1.1 和 [AMQP 1.0][lnk-amqp] 协议进行公开。请注意，也可以通过端口 443 上的 [WebSockets][lnk-websockets] 来实现 AMQP。
    
    克隆的终结点和方法的终结点只能通过 [MQTT v3.1.1][lnk-mqtt] 使用。

* **服务终结点**。每个 IoT 中心公开应用程序后端可用来与设备通信的一组终结点。这些终结点目前只能通过 [AMQP][lnk-amqp] 协议公开，方法调用终结点除外，后者通过 HTTP 1.1 公开。
    - *接收设备到云的消息* 。此终结点与 [Azure 事件中心][lnk-event-hubs]兼容。后端服务可用它来读取由设备发送的所有[设备到云的消息][lnk-d2c]。
    - *发送云到设备的消息并接收传递确认* 。这些终结点可让应用程序后端发送可靠的[云到设备的消息][lnk-c2d]，以及接收对应的传送或过期确认。
    - *接收文件通知* 。此消息传递终结点允许你在设备成功上传文件时接收通知。
    - *直接方法调用* 。此终结点允许后端服务调用设备上的[直接方法][lnk-methods]。

[IoT Hub APIs and SDKs][lnk-sdks]（IoT 中心 API 和 SDK）一文介绍了可用于访问这些终结点的各种方式。

最后请务必注意，所有的 IoT 中心终结点都使用 [TLS][lnk-tls] 协议，且绝不会在未加密/不安全的通道上公开任何终结点。

## <a name="field-gateways"></a> 现场网关

在 IoT 解决方案中，*现场网关* 位于设备和 IoT 中心终结点之间。它通常位于靠近设备的位置。设备使用设备支持的协议，直接与现场网关通信。现场网关使用 IoT 中心支持的协议连接到 IoT 中心终结点。现场网关可以是高度专业化的硬件，也可以是运行软件的低功率计算机，只需能够完成网关所适用的端到端方案即可。

可以使用 [Azure IoT 网关 SDK][lnk-gateway-sdk] 实现现场网关。此 SDK 提供特定功能，例如能够在与 IoT 中心的同一个连接上多路复用来自多个设备的通信。

## 后续步骤

此 IoT 中心开发人员指南中的其他参考主题包括：

- [克隆、方法和作业的查询语言][lnk-devguide-query]
- [配额和限制][lnk-devguide-quotas]
- [IoT 中心 MQTT 支持][lnk-devguide-mqtt]

[lnk-gateway-sdk]: https://github.com/Azure/azure-iot-gateway-sdk

[img-endpoints]: ./media/iot-hub-devguide-endpoints/endpoints.png
[lnk-amqp]: https://www.amqp.org/
[lnk-mqtt]: http://mqtt.org/
[lnk-websockets]: https://tools.ietf.org/html/rfc6455
[lnk-arm]: /documentation/articles/resource-group-overview/
[lnk-event-hubs]: /documentation/services/event-hubs/

[lnk-tls]: https://tools.ietf.org/html/rfc5246


[lnk-sdks]: /documentation/articles/iot-hub-devguide-sdks/
[lnk-accesscontrol]: /documentation/articles/iot-hub-devguide-security/#access-control-and-permissions
[lnk-importexport]: /documentation/articles/iot-hub-devguide-identity-registry/#import-and-export-device-identities
[lnk-d2c]: /documentation/articles/iot-hub-devguide-messaging/#device-to-cloud-messages
[lnk-device-identities]: /documentation/articles/iot-hub-devguide-identity-registry/
[lnk-upload]: /documentation/articles/iot-hub-devguide-file-upload/
[lnk-c2d]: /documentation/articles/iot-hub-devguide-messaging/#cloud-to-device-messages
[lnk-methods]: /documentation/articles/iot-hub-devguide-direct-methods/
[lnk-twins]: /documentation/articles/iot-hub-devguide-device-twins/
[lnk-query]: /documentation/articles/iot-hub-devguide-query-language/
[lnk-jobs]: /documentation/articles/iot-hub-devguide-jobs/

[lnk-devguide-quotas]: /documentation/articles/iot-hub-devguide-quotas-throttling/
[lnk-devguide-query]: /documentation/articles/iot-hub-devguide-query-language/
[lnk-devguide-mqtt]: /documentation/articles/iot-hub-mqtt-support/

<!---HONumber=Mooncake_1031_2016-->