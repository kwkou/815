<properties
 pageTitle="面向物联网的 Azure 解决方案 | Azure"
 description="Azure 上 IoT 的概述，包括一个解决方案体系结构示例以及它如何与 Azure IoT 中心、设备 SDK 和预配置解决方案关联"
 services="iot-hub"
 documentationCenter=""
 authors="dominicbetts"
 manager="timlt"
 editor=""/>

<tags
 ms.service="iot-hub"
 ms.date="04/29/2016"
 wacn.date="05/30/2016"/>

[AZURE.INCLUDE [iot-azure-and-iot](../includes/iot-azure-and-iot.md)]

## 后续步骤

Azure IoT 中心是一项 Azure 服务，可在一个应用程序后端和数百万个设备之间实现安全可靠的双向通信。它可让应用程序后端接收设备的大规模遥测数据，将该数据路由到流事件处理器，而且还能对特定设备发送云到设备的命令。你可以使用 IoT 中心来实现自己的解决方案后端。此外，IoT 中心还包含设备标识注册表，可供用来设置设备、其安全凭据和其连接到中心的权限。若要了解更多信息，请参阅以下文章：

- [IoT 中心是什么？][lnk-iot-hub]
- [IoT 中心入门][lnk-getstarted]

你可以使用 IoT 设备 SDK 在各种设备硬件平台和操作系统上实现客户端应用程序。IoT 设备 SDK 包含库，可协助将遥测数据发送到 IoT 中心，并接收云到设备的命令。使用 SDK 时，可从数个网络协议中进行选择，以便与 IoT 中心通信。若要了解详细信息，请参阅[设备 SDK 的相关信息][lnk-device-sdks]。



[lnk-getstarted]: /documentation/articles/iot-hub-csharp-csharp-getstarted/
[lnk-device-sdks]: https://github.com/Azure/azure-iot-sdks/blob/master/readme.md
[lnk-iot-hub]: /documentation/articles/iot-hub-what-is-iot-hub/
[lnk-iot-suite]: /documentation/suites/iot-suite/
[lnk-iotdev]: /develop/iot/

<!---HONumber=Mooncake_0321_2016-->