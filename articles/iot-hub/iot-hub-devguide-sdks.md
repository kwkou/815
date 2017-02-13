<properties
    pageTitle="了解 Azure IoT SDK | Azure"
    description="开发人员指南 - 介绍了相关链接，其指向可用于构建设备应用和后端应用的各种 Azure IoT 设备和服务 SDK。"
    services="iot-hub"
    documentationcenter=""
    author="dominicbetts"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="c5c9a497-bb03-4301-be2d-00edfb7d308f"
    ms.service="iot-hub"
    ms.devlang="multiple"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="01/04/2017"
    wacn.date="02/10/2017"
    ms.author="dobett" />  


# Azure IoT SDK
## Azure IoT 设备 SDK
Microsoft Azure IoT 设备 SDK 包含的代码可帮助构建连接到 Azure IoT 中心服务并由这些服务管理的设备和应用程序。

可从 GitHub 下载下述 Azure IoT 设备 SDK：

- [适用于 C 的 Azure IoT 设备 SDK][lnk-c-device-sdk]，采用 ANSI C (C99) 编写，具有可移植性和广泛的平台兼容性。
- [适用于 .NET 的 Azure IoT 设备 SDK][lnk-dotnet-device-sdk]
- [适用于 Java 的 Azure IoT 设备 SDK][lnk-java-device-sdk]
- [适用于 Node.js 的 Azure IoT 设备 SDK][lnk-node-device-sdk]
- [适用于 Python 的 Azure IoT 设备 SDK][lnk-python-device-sdk]

> [AZURE.NOTE]
有关使用语言和平台特定的程序包管理器在开发计算机上安装二进制文件和依赖项的信息，请参阅 GitHub 存储库中的自述文件。
> 
> 

## 操作系统平台和硬件兼容性

有关与特定硬件设备的 SDK 兼容性的详细信息，请参阅以下文章：

- [OS 平台和硬件与设备 SDK 的兼容性][lnk-compatibility]

## Azure IoT 服务 SDK
Azure IoT 服务 SDK 包含的代码可帮助构建直接与 IoT 中心进行交互以管理设备和安全性的应用程序。

可从 GitHub 下载下述 Azure IoT 服务 SDK：

- [适用于 .NET 的 Azure IoT 服务 SDK][lnk-dotnet-service-sdk]
- [适用于 Node.js 的 Azure IoT 服务 SDK][lnk-node-service-sdk]
- [适用于 Java 的 Azure IoT 服务 SDK][lnk-java-service-sdk]
- [适用于 Python 的 Azure IoT 服务 SDK][lnk-python-service-sdk]

> [AZURE.NOTE]
有关使用语言和平台特定的程序包管理器在开发计算机上安装二进制文件和依赖项的信息，请参阅 GitHub 存储库中的自述文件。
> 
> 

## Azure IoT 网关 SDK
此 Azure IoT 网关 SDK 包含创建 IoT 网关解决方案的基础结构和模块。可以扩展此 SDK 来创建适用于任何端到端场景的网关。

可以从 GitHub 下载 [Azure IoT 网关 SDK][lnk-gateway-sdk]。

## 联机 API 参考文档
以下列表包含 Azure IoT 设备、服务和网关库的联机 API 参考文档的链接：

- [物联网 (IoT) .NET][lnk-dotnet-ref]
- [IoT 中心 REST][lnk-rest-ref]
- [适用于 C 的 Azure IoT 设备 SDK][lnk-c-ref]
- [适用于 Java 的 Azure IoT 设备 SDK][lnk-java-ref]
- [适用于 Java 的 Azure IoT 服务 SDK][lnk-java-service-ref]
- [适用于 Node.js 的 Azure IoT 设备 SDK][lnk-node-ref]
- [适用于 Node.js 的 Azure IoT 服务 SDK][lnk-node-service-ref]
- [Azure IoT 网关 SDK][lnk-gateway-ref]

## 后续步骤
此 IoT 中心开发人员指南中的其他参考主题包括：

- [IoT 中心终结点][lnk-devguide-endpoints]
- [设备孪生和作业的 IoT 中心查询语言][lnk-devguide-query]
- [配额和限制][lnk-devguide-quotas]
- [IoT 中心 MQTT 支持][lnk-devguide-mqtt]

<!-- Links and images -->


[lnk-c-device-sdk]: https://github.com/Azure/azure-iot-sdk-c
[lnk-dotnet-device-sdk]: https://github.com/Azure/azure-iot-sdk-csharp/tree/master/device
[lnk-java-device-sdk]: https://github.com/Azure/azure-iot-sdk-java/tree/master/device
[lnk-dotnet-service-sdk]: https://github.com/Azure/azure-iot-sdk-csharp/tree/master/service
[lnk-java-service-sdk]: https://github.com/Azure/azure-iot-sdk-java/tree/master/service
[lnk-node-device-sdk]: https://github.com/Azure/azure-iot-sdk-node/tree/master/device
[lnk-node-service-sdk]: https://github.com/Azure/azure-iot-sdk-node/tree/master/service
[lnk-python-device-sdk]: https://github.com/Azure/azure-iot-sdk-python/tree/master/device
[lnk-python-service-sdk]: https://github.com/Azure/azure-iot-sdk-python/tree/master/service
[lnk-compatibility]: /documentation/articles/iot-hub-tested-configurations/
[lnk-gateway-sdk]: https://github.com/Azure/azure-iot-gateway-sdk

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

<!---HONumber=Mooncake_0206_2017-->
<!--Update_Description:update wording and link references-->