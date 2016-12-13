<properties
 pageTitle="Azure IoT 中心与 Azure 事件中心的比较 | Azure"
 description="将 Azure IoT 中心与 Azure 事件中心服务进行比较，重点为功能和用例方面的差异。"
 services="iot-hub"
 documentationCenter=""
 authors="fsautomata"
 manager="timlt"
 editor=""/>  


<tags
 ms.service="iot-hub"
 ms.devlang="na"
 ms.topic="article"
 ms.tgt_pltfrm="na"
 ms.workload="na"
 ms.date="06/06/2016"
 wacn.date="12/12/2016"
 ms.author="elioda"/>


# IoT 中心与事件中心的比较

Azure IoT 中心的主要用例之一是从设备收集遥测数据。因此，我们经常在 IoT 中心与 [Azure 事件中心][]之间进行比较。与 IoT 中心一样，事件中心是一种事件处理服务，用于向云提供大规模的事件与遥测数据入口，并且具有较低的延迟和较高的可靠性。

但是，这两个服务之间有许多差异，下表对此做了详述。

| 区域 | IoT 中心 | 事件中心 |
| --- | --- | --- |
| 通信模式 | 启用设备到云通信（消息传递、文件上传及设备克隆报告属性）和云到设备通信（直接方法、设备克隆所需属性、消息传递）。 |仅支持事件引入（通常视为设备到云的方案）。 |
| 设备状态信息 | [设备克隆][lnk-twins]可存储和查询设备状态信息。 | 没有可存储的设备状态信息。 |
| 设备协议支持 |支持 MQTT、基于 WebSockets 的 MQTT、AMQP、基于 WebSockets 的 AMQP 和 HTTP。此外，IoT 中心还可使用 [Azure IoT 协议网关][lnk-azure-protocol-gateway]，这是一种支持自定义协议的可自定义协议网关实现。 |支持 AMQP、基于 WebSockets 的 AMQP 和 HTTP。 |
| “安全” |提供每个设备的标识与可吊销的访问控制权限。请参阅 [IoT 中心开发人员指南的“安全性”部分]。 |提供事件中心范围的[共享访问策略][事件中心 - 安全]，通过[发布者策略][事件中心发布者策略]提供有限的权限吊销支持。IoT 解决方案通常需要实现自定义解决方案来支持每个设备的凭据以及防欺骗措施。 |
| 操作监视 |允许 IoT 解决方案订阅丰富的设备标识管理和连接事件集，例如单个设备的身份验证错误、限制和错误格式异常。这些事件可让你快速识别单个设备级别的连接问题。 |仅公开聚合度量值。 |
| 缩放 |已经过优化，可支持数百万个同时连接的设备。 |可支持的同时连接数具有更大的限制：根据 [Azure 服务总线配额][Azure Service Bus quotas]，最多只支持 5,000 个 AMQP 连接。另一方面，事件中心可让你为每个发送的消息指定分区。 |
| 设备 SDK |除直接 MQTT、AMQP 和 HTTP API 外，还为各种平台和语言提供[设备 SDK][Azure IoT Hub SDKs]。 |在 .NET、Java、C 以及 AMQP 和 HTTP 发送接口上受支持。 |
| 文件上载 |可让 IoT 解决方案将文件从设备上载到云。包含一个用于集成工作流的文件通知终结点，以及一个用于支持调试的操作监视类别。 | 不支持。 |

总而言之，即使唯一的用例是设备到云的遥测引入，IoT 中心也能专门为 IoT 设备连接提供一种服务。对于这些使用 IoT 特定功能的方案，它会持续完善价值主张。无论对于数据中心之间还是数据中心内部的方案，事件中心主要用于大规模事件引入。

在同一解决方案中同时使用 IoT 中心和事件中心的情况并不常见，因为 IoT 中心处理的是设备到云的通信，而事件中心处理的是将后期事件引入实时处理引擎。

## 后续步骤

若要深入了解如何规划 IoT 中心部署，请参阅[缩放、HA 和 DR][lnk-scaling]。

若要进一步探索 IoT 中心的功能，请参阅：

- [开发人员指南][lnk-devguide]
- [使用 IoT 网关 SDK 模拟设备][lnk-gateway]

[lnk-twins]: /documentation/articles/iot-hub-devguide-device-twins/
[Azure 事件中心]: /documentation/articles/event-hubs-what-is-event-hubs/
[IoT 中心开发人员指南的“安全性”部分]: /documentation/articles/iot-hub-devguide-security/
[Event Hub - security]: /documentation/articles/event-hubs-authentication-and-security-model-overview/
[Event Hub publisher policies]: /documentation/articles/event-hubs-overview/#common-publisher-tasks
[Azure 服务总线配额]: /documentation/articles/service-bus-quotas/
[Azure IoT Hub SDKs]: https://github.com/Azure/azure-iot-sdks/blob/master/readme.md
[lnk-azure-protocol-gateway]: /documentation/articles/iot-hub-protocol-gateway/

[lnk-scaling]: /documentation/articles/iot-hub-scaling/
[lnk-devguide]: /documentation/articles/iot-hub-devguide/
[lnk-gateway]: /documentation/articles/iot-hub-linux-gateway-sdk-simulated-device/
[lnk-portal]: /documentation/articles/iot-hub-manage-through-portal/

<!---HONumber=Mooncake_0801_2016-->