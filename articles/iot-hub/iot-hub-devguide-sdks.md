<properties
 pageTitle="开发人员指南 - IoT 中心 SDK | Azure"
 description="Azure IoT 中心开发人员指南 - 有关各种 Azure IoT 中心设备和服务 SDK 的信息以及指向这些 SDK 的链接。"
 services="iot-hub"
 documentationCenter=""
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


# IoT 中心 SDK

## IoT 中心设备 SDK

Microsoft Azure IoT 设备 SDK 包含的代码可帮助构建连接到 Azure IoT 中心服务并由这些服务管理的设备和应用程序。

以下 IoT 设备 SDK 可以从 GitHub 进行下载：

- 适用于 C 的 Azure IoT 设备 SDK，采用 ANSI C (C99) 编写，具有可移植性和广泛的平台兼容性。
- 适用于 .NET 的 Azure IoT 设备 SDK
- 适用于 Java 的 Azure IoT 设备 SDK
- 适用于 Node.js 的 Azure IoT 设备 SDK
- 适用于 Python 2.7 的 Microsoft Azure IoT 设备 SDK

> [AZURE.NOTE] 有关使用语言和平台特定的程序包管理器在开发计算机上安装二进制文件和依赖项的信息，请参阅 GitHub 存储库中的自述文件。

## 操作系统平台和硬件兼容性

有关与特定硬件设备的 SDK 兼容性的详细信息，请参阅以下文章：

- [OS 平台和硬件与设备 SDK 的兼容性][lnk-compatibility]

## IoT 中心服务 SDK

Microsoft Azure IoT 服务 SDK 包含的代码可帮助构建直接与 IoT 中心进行交互以管理设备和安全性的应用程序。

可从 GitHub 下载以下 IoT 服务 SDK：

- 适用于 .NET 的 Azure IoT 服务 SDK
- 适用于 Node.js 的 Azure IoT 服务 SDK
- 适用于 Java 的 Azure IoT 服务 SDK

> [AZURE.NOTE] 有关使用语言和平台特定的程序包管理器在开发计算机上安装二进制文件和依赖项的信息，请参阅 GitHub 存储库中的自述文件。

## Azure IoT 网关 SDK

此 Azure IoT 网关 SDK 包含创建 IoT 网关解决方案的基础结构和模块。你可以扩展此 SDK 来创建适用于任何端到端场景的网关。

可以从 GitHub 下载 [Azure IoT 网关 SDK][lnk-gateway-sdk]。

## 联机 API 参考文档

下面是 Azure IoT 设备、服务和网关库的联机 API 参考文档的链接列表：

- [物联网 (IoT) .NET][lnk-dotnet-ref]
- [IoT 中心 REST][lnk-rest-ref]
- [适用于 C 的 Microsoft Azure IoT 设备 SDK][lnk-c-ref]
- [适用于 Java 的 Microsoft Azure IoT 设备 SDK][lnk-java-ref]
- [适用于 Java 的 Microsoft Azure IoT 服务 SDK][lnk-java-service-ref]
- [适用于 Node.js 的 Microsoft Azure IoT 设备 SDK][lnk-node-ref]
- [适用于 Node.js 的 Microsoft Azure IoT 服务 SDK][lnk-node-service-ref]
- [Microsoft Azure IoT 网关 SDK][lnk-gateway-ref]

## 后续步骤

此 IoT 中心开发人员指南中的其他参考主题包括：

- [IoT 中心终结点][lnk-devguide-endpoints]
- [克隆、方法和作业的查询语言][lnk-devguide-query]
- [配额和限制][lnk-devguide-quotas]
- [IoT 中心 MQTT 支持][lnk-devguide-mqtt]

<!-- Links and images -->


[lnk-c-device-sdk]: https://github.com/Azure/azure-iot-sdks/blob/master/c/readme.md
[lnk-dotnet-device-sdk]: https://github.com/Azure/azure-iot-sdks/blob/master/csharp/device/readme.md
[lnk-java-device-sdk]: https://github.com/Azure/azure-iot-sdks/blob/master/java/device/readme.md
[lnk-dotnet-service-sdk]: https://github.com/Azure/azure-iot-sdks/blob/master/csharp/service/README.md
[lnk-java-service-sdk]: https://github.com/Azure/azure-iot-sdks/blob/master/java/service/readme.md
[lnk-node-device-sdk]: https://github.com/Azure/azure-iot-sdks/blob/master/node/device/readme.md
[lnk-node-service-sdk]: https://github.com/Azure/azure-iot-sdks/blob/master/node/service/README.md
[lnk-python-device-sdk]: https://github.com/Azure/azure-iot-sdks/blob/master/python/device/readme.md
[lnk-compatibility]: /documentation/articles/iot-hub-tested-configurations/
[lnk-gateway-sdk]: https://github.com/Azure/azure-iot-gateway-sdk/blob/master/README.md

[lnk-dotnet-ref]: https://msdn.microsoft.com/zh-cn/library/mt488521.aspx
[lnk-c-ref]: http://azure.github.io/azure-iot-sdks/c/api_reference/index.html
[lnk-java-ref]: http://azure.github.io/azure-iot-sdks/java/device/api_reference/index.html
[lnk-node-ref]: http://azure.github.io/azure-iot-sdks/node/api_reference/azure-iot-device/1.0.15/index.html
[lnk-rest-ref]: https://msdn.microsoft.com/zh-cn/library/mt548492.aspx
[lnk-java-service-ref]: http://azure.github.io/azure-iot-sdks/java/service/api_reference/index.html
[lnk-node-service-ref]: http://azure.github.io/azure-iot-sdks/node/api_reference/azure-iothub/1.0.17/index.html
[lnk-gateway-ref]: http://azure.github.io/azure-iot-gateway-sdk/api_reference/c/html/

[lnk-devguide-endpoints]: /documentation/articles/iot-hub-devguide-endpoints/
[lnk-devguide-quotas]: /documentation/articles/iot-hub-devguide-quotas-throttling/
[lnk-devguide-query]: /documentation/articles/iot-hub-devguide-query-language/
[lnk-devguide-mqtt]: /documentation/articles/iot-hub-mqtt-support/

<!---HONumber=Mooncake_1031_2016-->