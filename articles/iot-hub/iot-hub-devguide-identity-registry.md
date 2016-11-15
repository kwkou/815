<properties
 pageTitle="开发人员指南 - 设备标识注册表 | Azure"
 description="Azure IoT 中心开发人员指南 - 说明设备标识注册表以及如何使用它来管理设备"
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
 ms.author="dobett"
 wacn.date="11/07/2016"/>  


# 管理 IoT 中心中的设备标识

## 概述

每个 IoT 中心都有一个设备标识注册表，存储允许连接到中心的设备的相关信息。中心的设备标识注册表中必须先有设备的条目，然后该设备才能连接到中心。设备还必须基于设备标识注册表中存储的凭据向中心进行身份验证。

概括地说，设备标识注册表是支持 REST 的设备标识资源集合。向此注册表中添加条目时，IoT 中心将在服务中创建一组每设备资源，如包含未送达云到设备消息的队列。

### 何时使用

当需要预配可连接到 IoT 中心的设备和需要控制每个设备对中心内面向设备的终结点的访问权限时，请使用设备标识注册表。

> [AZURE.NOTE] 设备标识注册表不包含任何应用程序特定的元数据。

## 设备标识注册表操作

IoT 中心设备标识注册表公开以下操作：

* 创建设备标识
* 更新设备标识
* 按 ID 检索设备标识
* 删除设备标识
* 最多列出 1000 个标识
* 将所有标识导出到 Blob 存储
* 从 Blob 存储导入标识

上述所有操作均可以使用 [RFC7232][lnk-rfc7232] 中指定的乐观并发。

> [AZURE.IMPORTANT] 如果要检索中心标识注册表中的所有标识，唯一方法是使用[导出][lnk-export]功能。

IoT 中心设备标识注册表：

- 不包含任何应用程序元数据。
- 可以将 **deviceId** 作为键进行访问，就像字典一样。
- 不支持表达性查询。

IoT 解决方案通常具有不同的解决方案特定存储，其中包含应用程序特定的元数据。例如，智能建筑物解决方案中的解决方案特定存储将记录部署温度感应器的房间信息。

> [AZURE.IMPORTANT] 只将设备标识注册表用于设备管理和预配操作。运行时的高吞吐量操作不应依赖于在设备标识注册表中执行操作。例如，在发送命令前先检查设备的连接状态就是不支持的模式。请务必检查设备标识注册表的[限制速率][lnk-quotas]以及[设备检测信号][lnk-guidance-heartbeat]模式。

## 禁用设备

可以通过更新注册表中标识的**状态**属性来禁用设备。通常在两种情况下使用此属性：

- 在预配协调过程中。有关详细信息，请参阅 [Device Provisioning][lnk-guidance-provisioning]（设备预配）。
- 你出于任何原因认为设备遭到入侵或未经授权。

## <a name="import-and-export-device-identities"></a> 导入和导出设备标识

可以使用 [IoT 中心资源提供程序终结点][lnk-endpoints]上的异步操作，从 IoT 中心的标识注册表批量导出设备标识。导出是长时间运行的作业，它使用客户提供的 blob 容器来保存从标识注册表读取的设备标识数据。

可以使用 [IoT 中心资源提供程序终结点][lnk-endpoints]上的异步操作，将设备标识批量导入 IoT 中心的标识注册表。导入是长时间运行的作业，它使用客户提供的 blob 容器中的数据，将设备标识数据写入设备标识注册表。

- 有关导入和导出 API 的详细信息，请参阅 [Azure IoT 中心 - 资源提供程序 API][lnk-resource-provider-apis]。
- 若要了解有关如何运行导入和导出作业的详细信息，请参阅 [Bulk management of IoT Hub device identities][lnk-bulk-identity]（批量管理 IoT 中心的设备标识）。

## <a name="device-provisioning"></a> 设备预配

给定的 IoT 解决方案存储的设备数据取决于该解决方案的特定要求。但是，解决方案必须至少存储设备标识和身份验证密钥。Azure IoT 中心包含标识注册表，可以存储每个设备的值，例如 ID、身份验证密钥和状态代码。解决方案可以使用其他 Azure 服务（例如表、Blob 或 Azure DocumentDB）来存储任何其他设备数据。

*设备预配* 是将初始设备数据添加到解决方案中存储中的过程。若要使新设备能够连接到中心，必须将新设备 ID 和密钥添加到 IoT 中心标识注册表。在预配过程中，你可能需要初始化其他解决方案存储中的设备特定数据。

## <a name="device-heartbeat"></a> 设备检测信号

IoT 中心标识注册表包含名为 **connectionState** 的字段。你只应在开发和调试期间使用 **connectionState** 字段，IoT 解决方案不应在运行时查询该字段（例如，为了检查设备是否已连接以确定是否要发送云到设备的消息或短信）。

如果 IoT 解决方案需要知道设备是否已连接（在运行时，或在比 **connectionState** 属性提供的值更精确时），解决方案应该实施 *检测信号模式* 。

在检测信号模式下，设备每隔固定时间至少发送一次设备到云的消息（例如，每小时至少一次）。这意味着，即使设备没有任何要发送的数据，仍会发送空的设备到云的消息（通常具有可供识别其属于检测信号的属性）。在服务端，解决方案维护一份图表，其中包含每个设备所收到的最后一次检测信号，并假设如果设备没有在预期时间内收到检测信号消息，即表示设备有问题。

更复杂的实现可包含来自[操作监视][lnk-devguide-opmon]的信息，以便识别尝试连接或通信但失败的设备。实施检测信号模式时，请务必查看 [IoT 中心配额与限制][lnk-quotas]。

> [AZURE.NOTE] 如果 IoT 解决方案只根据设备连接状态来决定是否发送云到设备的消息，并且没有把消息广播到大量设备，则可以考虑使用更简单的模式，即使用较短的到期时间。它达到的效果与使用检测信号模式维护设备连接状态达到的效果一样，而且更加有效。IoT 中心还可以通过请求消息确认来通知哪些设备可以接收消息、哪些设备脱机或不能接收消息。

## 参考主题：

以下参考主题提供有关设备标识注册表的详细信息。

## <a name="device-identity-properties"></a> 设备标识属性

设备识别以包含以下属性的 JSON 文档表示。

| 属性 | 选项 | 说明 |
| -------- | ------- | ----------- |
| deviceId | 必需，更新时只读 | ASCII 7 位字母数字字符 + `{'-', ':', '.', '+', '%', '_', '#', '*', '?', '!', '(', ')', ',', '=', '@', ';', '$', '''}` 的区分大小写字符串（最长为 128 个字符）。 |
| generationId | 必需，只读 | 中心生成的区分大小写字符串，最长为 128 个字符。在删除并重新创建设备时用于区分具有相同 **deviceId** 的设备。 |
| etag | 必需，只读 | 一个字符串，根据 [RFC7232][lnk-rfc7232] 表示设备标识的弱 etag。|
| auth | 可选 | 包含身份验证信息和安全材料的复合对象。 |
| auth.symkey | 可选 | 包含主密钥和辅助密钥的复合对象，以 base64 格式存储。 |
| status | 必填 | 访问指示器。可以是 **Enabled** 或 **Disabled**。如果是 **Enabled**，则允许设备连接。如果是 **Disabled**，则此设备无法访问任何面向设备的终结点。 |
| statusReason | 可选 | 128 个字符的字符串，用于存储设备标识状态的原因。允许所有 UTF-8 字符。 |
| statusUpdateTime | 只读 | 临时指示器，显示上次状态更新的日期和时间。 |
| connectionState | 只读 | 指示连接状态的字段：**Connected** 或 **Disconnected**。此字段表示设备连接状态的 IoT 中心视图。**重要说明**：此字段只用于开发/调试目的。仅使用 MQTT 或 AMQP 的设备才更新连接状态。此外，它基于协议级别的 ping（MQTT ping 或 AMQP ping），并且最多只有 5 分钟的延迟。出于这些原因，可能会发生误报，例如，将设备报告为已连接，但实际上已断开连接。 |
| connectionStateUpdatedTime | 只读 | 临时指示器，显示上次更新连接状态的日期和时间。 |
| lastActivityTime | 只读 | 临时指示器，显示设备上次连接、接收或发送消息的日期和时间。 |

> [AZURE.NOTE] 连接状态只能表示连接状态的 IoT 中心视图。根据网络状态和配置，可能会延迟此状态的更新。

## 其他参考资料

开发人员指南中的其他参考主题包括：

- [IoT 中心终结点][lnk-endpoints]说明每个 IoT 中心针对运行时和管理操作公开的各种终结点。
- [限制和配额][lnk-quotas]说明适用于 IoT 中心服务的配额，以及使用服务时可预期的限制行为。
- [IoT 中心设备和服务 SDK][lnk-sdks] 列出在开发与 IoT 中心交互的设备和服务应用程序时使用的各种语言 SDK。
- [克隆、方法和作业的查询语言][lnk-query]描述可用于从 IoT 中心检索有关设备克隆、方法和作业的信息的查询语言。
- [IoT 中心 MQTT 支持][lnk-devguide-mqtt]提供有关 IoT 中心对 MQTT 协议的支持的详细信息。

## 后续步骤

现在已了解如何使用 IoT 中心设备标识注册表，你可能对以下开发人员指南主题感兴趣：

- [控制 IoT 中心的访问权限][lnk-devguide-security]
- [使用设备克隆来同步状态和配置][lnk-devguide-device-twins]
- [在设备上调用直接方法][lnk-devguide-directmethods]
- [在多台设备上计划作业][lnk-devguide-jobs]

如果要尝试本文中介绍的一些概念，你可能对以下 IoT 中心教程感兴趣：

- [Azure IoT 中心入门][lnk-getstarted-tutorial]


<!-- Links and images -->


[lnk-endpoints]: /documentation/articles/iot-hub-devguide-endpoints/
[lnk-quotas]: /documentation/articles/iot-hub-devguide-quotas-throttling/
[lnk-sdks]: /documentation/articles/iot-hub-devguide-sdks/
[lnk-query]: /documentation/articles/iot-hub-devguide-query-language/
[lnk-devguide-mqtt]: /documentation/articles/iot-hub-mqtt-support/
[lnk-resource-provider-apis]: https://msdn.microsoft.com/zh-cn/library/mt548492.aspx
[lnk-guidance-provisioning]: /documentation/articles/iot-hub-devguide-identity-registry/#device-provisioning
[lnk-guidance-heartbeat]: /documentation/articles/iot-hub-devguide-identity-registry/#device-heartbeat
[lnk-rfc7232]: https://tools.ietf.org/html/rfc7232
[lnk-bulk-identity]: /documentation/articles/iot-hub-bulk-identity-mgmt/
[lnk-export]: /documentation/articles/iot-hub-devguide-identity-registry/#import-and-export-device-identities
[lnk-devguide-opmon]: /documentation/articles/iot-hub-operations-monitoring/

[lnk-devguide-security]: /documentation/articles/iot-hub-devguide-security/
[lnk-devguide-device-twins]: /documentation/articles/iot-hub-devguide-device-twins/
[lnk-devguide-directmethods]: /documentation/articles/iot-hub-devguide-direct-methods/
[lnk-devguide-jobs]: /documentation/articles/iot-hub-devguide-jobs/

[lnk-getstarted-tutorial]: /documentation/articles/iot-hub-csharp-csharp-getstarted/

<!---HONumber=Mooncake_1031_2016-->