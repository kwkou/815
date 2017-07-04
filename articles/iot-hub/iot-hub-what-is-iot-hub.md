<properties
 pageTitle="Azure IoT 中心概述 | Azure"
 description="Azure IoT 中心服务概述：什么是 IoT 中心、设备连接、物联网通信模式和服务辅助通信模式"
 services="iot-hub"
 documentationCenter=""
 authors="dominicbetts"
 manager="timlt"
 editor=""/>  

<tags
 ms.service="iot-hub"
 ms.devlang="na"
 ms.topic="get-started-article"
 ms.tgt_pltfrm="na"
 ms.workload="na"
 ms.date="01/31/2017"
 wacn.date="03/10/2017"
 ms.author="dobett"/>
  


# Azure IoT 中心是什么？

欢迎使用 Azure IoT 中心。本文对 Azure IoT 中心进行了概述，并介绍了应该使用此服务实现物联网 \(IoT\) 解决方案的原因。Azure IoT 中心是一项完全托管的服务，可在数百万个 IoT 设备和一个解决方案后端之间实现安全可靠的双向通信。Azure IoT 中心：

- 提供多个设备到云和云到设备通信选项，包括单向消息传送、文件传输和请求-答复方法。
- 提供路由到其他 Azure 服务的内置声明性消息。
- 提供设备元数据和同步状态信息的可查询存储。
- 使用每个设备的安全密钥或 X.509 证书来实现安全通信和访问控制。
- 可广泛监视设备连接性和设备标识管理事件。
- 包含最流行语言和平台的设备库。

[IoT 中心与事件中心的比较][lnk-compare]一文描述这两个服务之间的主要差异，并重点说明了在 IoT 解决方案中使用 IoT 中心的优点。

![在物联网解决方案中充当云网关的 Azure IoT 中心][img-architecture]  


> [AZURE.NOTE] 有关 IoT 体系结构的深入讨论，请参阅 [Azure IoT Reference Architecture][lnk-refarch]（Azure IoT 参考体系结构）。

## IoT 设备连接性挑战
IoT 中心和设备库可帮助你应对挑战，即如何以可靠且安全的方式将设备连接到解决方案后端。IoT 设备：

- 通常是无人操作的嵌入式系统。
- 可位于物理访问较昂贵的远程位置。
- 可能只能通过解决方案后端来访问。
- 能力和处理资源可能都有限。
- 网络连接可能不稳定、缓慢或昂贵。
- 可能需要使用专属、自定义或行业特定的应用程序协议。
- 可以使用大量常见的硬件和软件平台来创建。

除了上述需求之外，所有 IoT 解决方案还必须提供可扩展性、安全性和可靠性。使用传统技术（例如 Web 容器和消息传送代理）时，所产生的一系列连接需求不仅难以实现，而且实现起来非常耗时。

## 为何使用 Azure IoT 中心
除丰富的[设备到云][lnk-d2c-guidance]和[云到设备][lnk-c2d-guidance]通信选项（包括消息传送、文件传输和请求-答复方法）外，Azure IoT 中心还通过以下方式解决设备连接问题：

- **设备孪生**。借助[设备孪生][lnk-twins]，可以存储、同步和查询设备元数据及状态信息。设备孪生是存储设备状态信息（元数据、配置和条件）的 JSON 文档。IoT 中心为连接到 IoT 中心的每台设备保留一个设备孪生。
- **每个设备的身份验证和安全连接性**。你可以为每个设备设置独有的[安全密钥][lnk-devguide-security]，让它连接到 IoT 中心。[IoT 中心标识注册表][lnk-devguide-identityregistry]会在解决方案中存储设备标识和密钥。解决方案后端可将个别设备添加到允许或拒绝列表，完全掌控设备访问权限。
- **基于声明性规则，将设备到云消息路由到 Azure 服务**。通过 IoT 中心，可定义基于消息规则的消息路由，控制中心将设备到云消息发送到何处。消息规则不要求编写任何代码，可代替自定义的引入后消息分派程序。
-   **设备连接操作监视**。你可以收到有关设备标识管理操作与设备连接事件的详细操作日志。借助此监视功能，IoT 解决方案可识别连接问题，例如，尝试用错误凭据连接的设备、消息发送过于频繁的设备或者拒绝所有云到设备消息的设备。

-   **一组丰富的设备库**。[Azure IoT 设备 SDK][lnk-device-sdks] 适用于各种语言和平台 - C 面向很多 Linux 分发版、Windows 和实时操作系统。Azure IoT 设备 SDK 也支持 C#、Java 和 JavaScript 等托管语言。

-   **IoT 协议和可扩展性**。如果解决方案无法使用设备库，则 IoT 中心会公开一个公共协议，它使设备可以通过本机方式使用 MQTT v3.1.1、HTTP 1.1 或 AMQP 1.0 协议。还可以通过以下方式扩展 IoT 中心，以便为自定义协议提供支持：

    - 使用 [Azure IoT 网关 SDK][lnk-gateway-sdk]创建现场网关，该 SDK 可将自定义协议转换为 IoT 中心所理解的三个协议之一。 
    - 自定义 [Azure IoT 协议网关][protocol-gateway]（在云中运行的一个开放源代码组件）。

-   **扩展**。Azure IoT 中心可扩展为数百万个同时连接的设备，以及每秒数百万个事件。

## 网关

IoT 解决方案中的网关通常是部署于云中的[协议网关][lnk-gateway]或使用设备在本地部署的[现场网关][lnk-field-gateway]。协议网关执行协议转换，例如 MQTT 到 AMQP。现场网关可在边缘运行分析、制定能减少延迟的时间敏感型决策，提供设备管理服务，强制实施安全和隐私约束，还可执行协议转换。这两种网关都可以充当设备与 IoT 中心之间的媒介。

现场网关与简单的流量路由设备（例如网络地址转换设备或防火墙）不同，它通常在解决方案中主动管理访问和信息流。

解决方案可以同时包含协议网关和现场网关。

## IoT 中心如何运作？
Azure IoT 中心会实现[服务辅助通信][lnk-service-assisted-pattern]模式，以调节设备与解决方案后端之间的交互。服务辅助通信的目标是在控制系统（例如 IoT 中心）与专用设备（部署在不受信任的物理空间中）之间，建立可信任的双向通信路径。该模式会建立下列原则：

- 安全性的优先级高于其他所有功能。
- 设备不接受未经请求的网络信息。设备以仅限出站的方式建立所有连接和路由。若要让设备从解决方案后端接收命令，设备必须定期启动连接，检查是否有任何挂起的命令要处理。
- 设备只能同与它们对等的已知服务（例如 IoT 中心）进行连接或建立路由。
- 设备和服务之间或设备和网关之间的通信路径在应用程序协议层受到保护。
- 系统级别的授权和身份验证以每个设备的标识为基础。它们可让访问凭据和权限近乎实时地撤销。
- 对于因为电源或连接性而导致连接不稳定的设备而言，可通过保留命令和设备通知直到设备连接并接收它们，进而促进其双向通信。IoT 中心会为发送的命令维护设备特定的队列。
- 针对通过网关到特定服务的受保护传输，应用程序有效负载数据会受到单独保护。

移动行业大规模地使用了服务辅助通信模式来实现推送通知服务，例如 [Windows 推送通知服务][lnk-wns]和 Apple Push Notification 服务。

支持通过 ExpressRoute 的公共对等路径访问 IoT 中心。

## 后续步骤
若要了解如何从设备发送并从 IoT 中心接收消息，以及如何配置 IoT 中心的消息路由，请参阅[使用 IoT 中心发送和接收消息][lnk-send-messages]。

若要了解 IoT 中心如何实现标准的设备管理，从而远程管理、配置和更新设备，请参阅 [IoT 中心设备管理概述][lnk-device-management]。

若要在各种设备硬件平台和操作系统上实现客户端应用程序，可使用 Azure IoT 设备 SDK。设备 SDK 包含一些库，有助于将遥测数据发送到 IoT 中心和接收云到设备的消息。使用设备 SDK 时，可从选取各种网络协议与 IoT 中心进行通信。若要了解详细信息，请参阅[设备 SDK 的相关信息][lnk-device-sdks]。

若要开始编写一些代码并运行一些示例，请参阅 [IoT 中心入门][lnk-get-started]教程。

[img-architecture]: ./media/iot-hub-what-is-iot-hub/hubarchitecture.png


[lnk-get-started]: /documentation/articles/iot-hub-csharp-csharp-getstarted/
[protocol-gateway]: https://github.com/Azure/azure-iot-protocol-gateway/blob/master/README.md
[lnk-service-assisted-pattern]: http://blogs.msdn.com/b/clemensv/archive/2014/02/10/service-assisted-communication-for-connected-devices.aspx "服务辅助通信，博客作者 Clemens Vasters"
[lnk-compare]: /documentation/articles/iot-hub-compare-event-hubs/
[lnk-gateway]: /documentation/articles/iot-hub-protocol-gateway/
[lnk-field-gateway]: /documentation/articles/iot-hub-devguide-endpoints/#field-gateways
[lnk-devguide-identityregistry]: /documentation/articles/iot-hub-devguide-identity-registry/
[lnk-devguide-security]: /documentation/articles/iot-hub-devguide-security/
[lnk-wns]: https://msdn.microsoft.com/zh-cn/library/windows/apps/mt187203.aspx
[lnk-google-messaging]: https://developers.google.com/cloud-messaging/

[lnk-device-sdks]: https://github.com/Azure/azure-iot-sdks
[lnk-refarch]: http://download.microsoft.com/download/A/4/D/A4DAD253-BC21-41D3-B9D9-87D2AE6F0719/Microsoft_Azure_IoT_Reference_Architecture.pdf
[lnk-gateway-sdk]: https://github.com/Azure/azure-iot-gateway-sdk
[lnk-send-messages]: /documentation/articles/iot-hub-devguide-messaging/
[lnk-device-management]: /documentation/articles/iot-hub-device-management-overview/

[lnk-twins]: /documentation/articles/iot-hub-devguide-device-twins/
[lnk-c2d-guidance]: /documentation/articles/iot-hub-devguide-c2d-guidance/
[lnk-d2c-guidance]: /documentation/articles/iot-hub-devguide-d2c-guidance/
<!---HONumber=Mooncake_0109_2017-->
<!--Update_Description:update wording and link references-->