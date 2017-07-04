<properties
    pageTitle="Azure IoT 协议网关 | Azure"
    description="如何使用 Azure IoT 协议网关扩展 IoT 中心功能及协议支持，使设备利用不由 IoT 中心本机支持的协议即可连接到该中心。"
    services="iot-hub"
    documentationcenter=""
    author="kdotchkoff"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="555e59ae-3136-4533-8ba8-f3a3b6acf648"
    ms.service="iot-hub"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="01/11/2017"
    wacn.date="02/10/2017"
    ms.author="kdotchko" />  


# 支持 IoT 中心的其他协议
Azure IoT 中心通过 MQTT、AMQP 和 HTTP 协议以本机方式支持通信。在某些情况下，设备或现场网关可能无法使用这些标准协议的其中一个，且需要协议自适应。在这种情况下，可以使用自定义网关。自定义网关可以桥接进出 IoT 中心的流量，从而为 IoT 中心终结点启用协议自适应。你可以使用 [Azure IoT 协议网关](https://github.com/Azure/azure-iot-protocol-gateway/blob/master/README.md)作为自定义网关，为 IoT 中心启用协议自适应。

## Azure IoT 协议网关
Azure IoT 协议网关是协议自适应的框架，旨在用来与 IoT 中心进行高缩放性双向设备通信。协议网关是一种传递组件，通过特定的协议接受设备连接。它通过 AMQP 1.0 桥接发往 IoT 中心的流量。Azure IoT 协议网关以开源软件项目的形式提供，它具有一定的灵活性，可用于添加对各种协议和协议版本的支持。

可以使用 Azure Service Fabric、Azure 云服务辅助角色或 Windows 虚拟机，以高度可缩放的方式在 Azure 中部署协议网关。此外，可将协议网关部署在本地环境中，例如现场网关。

Azure IoT 协议网关包含可让用户根据需要自定义 MQTT 协议行为的 MQTT 协议适配器。由于 IoT 中心对 MQTT v3.1.1 协议提供内置支持，因此，需要进行协议自定义或者对其他功能有具体要求时，应只考虑使用 MQTT 协议适配器。

MQTT 适配器还会演示用来为其他协议构建协议适配器的编程模型。此外，借助 Azure IoT 协议网关编程模型，可对专门化处理插入自定义组件，例如自定义身份验证、消息转换、压缩/解压缩，或加密/解密设备与 IoT 中心之间的流量。

为了提供弹性，协议网关和 MQTT 实现在开源软件项目中提供。这样，你可就可以根据需要自定义实现。

## 后续步骤

若要了解有关 Azure IoT 协议网关的详细信息以及如何使用并将其部署为 IoT 解决方案的一部分，请参阅：

* [GitHub 上的 Azure IoT 协议网关存储库](https://github.com/Azure/azure-iot-protocol-gateway/blob/master/README.md)
* [Azure IoT 协议网关开发人员指南](https://github.com/Azure/azure-iot-protocol-gateway/blob/master/docs/DeveloperGuide.md)

若要深入了解如何规划 IoT 中心部署，请参阅：

- [与事件中心比较][lnk-compare]
- [缩放、HA 和 DR][lnk-scaling]

- [IoT 中心开发人员指南][lnk-devguide]

[lnk-compare]: /documentation/articles/iot-hub-compare-event-hubs/
[lnk-scaling]: /documentation/articles/iot-hub-scaling/
[lnk-devguide]: /documentation/articles/iot-hub-devguide/

<!---HONumber=Mooncake_0206_2017-->
<!--Update_Description:update wording-->