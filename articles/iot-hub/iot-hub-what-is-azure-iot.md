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
 ms.devlang="na"
 ms.topic="get-started-article"
 ms.tgt_pltfrm="na"
 ms.workload="na"
 ms.date="10/05/2016"
 ms.author="dobett"
 wacn.date="11/07/2016"/>  


[AZURE.INCLUDE [iot-azure-and-iot](../../includes/iot-azure-and-iot.md)]

## 后续步骤

Azure IoT 中心是一项 Azure 服务，可在一个应用程序后端和数百万个设备之间实现安全可靠的双向通信。它使应用程序后端可以：

- 从你的设备大规模接收遥测。
- 将数据从你的设备路由到流事件处理器。
- 从设备接收文件上传。
- 将云到设备命令发送到特定设备。

你可以使用 IoT 中心来实现自己的解决方案后端。此外，IoT 中心还包含设备标识注册表，可供用来设置设备、其安全凭据和其连接到中心的权限。若要了解有关 IoT 中心的详细信息，请参阅 [IoT 中心是什么？][lnk-iot-hub]。

若要了解 Azure IoT 中心如何实现标准的 IoT 设备管理，以便远程管理、配置和更新设备，请参阅 [Azure IoT 中心设备管理概述][lnk-device-management]。

你可以使用 IoT 设备 SDK 在各种设备硬件平台和操作系统上实现客户端应用程序。IoT 设备 SDK 包含库，可协助将遥测数据发送到 IoT 中心，并接收云到设备的命令。使用 SDK 时，可从数个网络协议中进行选择，以便与 IoT 中心通信。若要了解详细信息，请参阅[设备 SDK 的相关信息][lnk-device-sdks]。

若要开始编写一些代码并运行一些示例，请参阅 [IoT 中心入门][lnk-getstarted]教程。

你也可能会对 [Azure IoT 套件][lnk-iot-suite]有兴趣，这是一套预配置的解决方案。IoT 套件可让你快速入门和扩展 IoT 项目，以应对常见的 IoT 案例，例如远程监视、资产管理和预测性维护。

[lnk-getstarted]: /documentation/articles/iot-hub-csharp-csharp-getstarted/
[lnk-device-sdks]: https://github.com/Azure/azure-iot-sdks/blob/master/readme.md
[lnk-iot-hub]: /documentation/articles/iot-hub-what-is-iot-hub/
[lnk-iot-suite]: /documentation/suites/iot-suite/
[lnk-iotdev]: /develop/iot/
[lnk-device-management]: /documentation/articles/iot-hub-device-management-overview/

<!---HONumber=Mooncake_0321_2016-->