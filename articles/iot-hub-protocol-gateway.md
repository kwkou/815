<properties
   pageTitle="Azure IoT 协议网关 | Azure"
   description="介绍如何使用 Azure IoT 协议网关来扩展 Azure IoT 中心的功能和协议支持。"
   services="iot-hub"
   documentationCenter=""
   authors="kdotchkoff"
   manager="timlt"
   editor=""/>

<tags
   ms.service="iot-hub"
   ms.date="02/03/2016"
   wacn.date="03/18/2016"/>

# 支持 IoT 中心的其他协议

Azure IoT 中心通过 AMQP、MQTT 和 HTTP/1 协议以本机方式支持通信。在某些情况下，设备或现场网关可能无法使用这些标准协议的其中一个，且需要协议自适应。在这种情况下，你可以使用自定义网关。自定义网关可以桥接进出 IoT 中心的流量，从而为 IoT 中心终结点启用协议自适应。你可以使用 [Azure IoT 协议网关](https://github.com/Azure/azure-iot-protocol-gateway/blob/master/README.md)作为自定义网关，来为 IoT 中心启用协议自适应。

## Azure IoT 协议网关

Azure IoT 协议网关是协议自适应的框架，旨在用来与 IoT 中心进行高缩放性双向设备通信。协议网关是一种传递组件，通过特定的协议接受设备连接。它通过 AMQP 1.0 桥接发往 IoT 中心的流量。IoT 协议网关以开源软件项目的形式提供使用，它具有一定的弹性，可用于添加对各种协议和协议版本的支持。

你可以使用 Azure 云服务辅助角色，以高度缩放的方式在 Azure 中部署协议网关。此外，可将协议网关部署在本地环境中，例如现场网关。

Azure IoT 协议网关包含可让你根据需要自定义 MQTT 协议行为的 MQTT 协议适配器。由于 IoT 中心对 MQTT v3.1.1 协议提供内置支持，因此，当你需要进行协议自定义或者对其他功能有具体要求时，应只考虑使用 MQTT 协议适配器。

MQTT 适配器还会演示用来为其他协议构建协议适配器的编程模型。此外，IoT 协议网关编程模型可让你对专门化处理插入自定义组件，例如自定义身份验证、消息转换、压缩/解压缩，或加密/解密设备与 IoT 中心之间的流量。

为了提供弹性，协议网关和 MQTT 实现在开源软件项目中提供。这样，你可就可以根据自定义该实现。

## 后续步骤

若要了解有关 Azure IoT 协议网关的详细信息以及如何使用并将其部署为 IoT 解决方案的一部分，请参阅：

* [GitHub 上的 Azure IoT 协议网关存储库](https://github.com/Azure/azure-iot-protocol-gateway/blob/master/README.md)
* [Azure IoT 协议网关开发人员指南](https://github.com/Azure/azure-iot-protocol-gateway/blob/master/docs/DeveloperGuide.md)

<!---HONumber=Mooncake_0307_2016-->