<properties
 pageTitle="IoT 中心解决方案指南 | Microsoft Azure"
 description="有关网关、设备预配和身份验证的指导主题，帮助用户使用 Azure IoT 中心开发 IoT 解决方案。"
 services="iot-hub"
 documentationCenter=""
 authors="dominicbetts"
 manager="timlt"
 editor=""/>

<tags
 ms.service="iot-hub"
 ms.date="02/03/2016"
 wacn.date="03/18/2016"/>

# 设计你的解决方案

本文提供有关如何在物联网 (IoT) 解决方案中设计以下功能的指导：

- 设备预配
- 现场网关
- 设备身份验证

## 设备预配

IoT 解决方案存储有关单个设备的数据，例如：

- 设备标识和身份验证密钥
- 设备硬件类型和版本
- 服务状态
- 软件版本和功能
- 设备命令历史记录

给定的 IoT 解决方案存储的设备数据取决于该解决方案的特定要求。但是，解决方案必须至少存储设备标识和身份验证密钥。Azure IoT 中心包含[标识注册表][lnk-devguide-identityregistry]，可以存储每个设备的值，例如 ID、身份验证密钥和状态代码。解决方案可以使用其他 Azure 服务（例如表、Blob 或 Azure DocumentDB）来存储任何其他设备数据。

*设备预配*是将初始设备数据添加到解决方案中存储中的过程。若要使新设备能够连接到中心，必须将新设备 ID 和密钥添加到 [IoT 中心标识注册表][lnk-devguide-identityregistry]。在预配过程中，你可能需要初始化其他解决方案存储中的设备特定数据。

[IoT 中心设备管理指南][lnk-device-management]一文描述了设备预配的一些常用策略。[IoT 中心标识注册表 API][lnk-devguide-identityregistry] 可让你将 IoT 中心集成到预配过程。

## 现场网关

在 IoT 解决方案中，*现场网关*位于设备和 IoT 中心之间。它通常位于靠近设备的位置。设备使用设备支持的协议，直接与现场网关通信。现场网关使用 IoT 中心支持的协议来与 IoT 中心通信。现场网关可以是专用的独立设备，也可以是在现有硬件部件上运行的软件。

现场网关与简单的流量路由设备（例如网络地址转换 (NAT) 设备或防火墙）不同，因为它通常在解决方案中管理访问和信息流中扮演主动的角色。例如，现场网关可以：

- 管理本地设备。例如，现场网关可以执行事件规则处理，并将命令发送到设备以响应特定遥测数据。
- 筛选或聚合遥测数据，然后将数据转发到 IoT 中心。这可以减少发送到 IoT 中心的数据量，还可能降低解决方案的成本。
- 帮助预配设备。
- 转换遥测数据，以简化解决方案后端的处理。
- 执行协议转换可让设备与 IoT 中心进行通信，即使它们未使用 IoT 中心支持的传输协议。

> [AZURE.NOTE] 尽管你通常会将现场网关部署到设备本地，在某些情况下，也可以在云中部署[协议网关][lnk-gateway]。

### 现场网关的类型

现场网关可以是*透明的*或*不透明的*：

| &nbsp; | 透明网关 | 不透明网关|
|--------|-------------|--------|
| 存储在 IoT 中心标识注册表中的标识 | 所有连接设备的标识 | 仅现场网关的标识 |
| IoT 中心可以提供[设备标识反欺骗][lnk-devguide-antispoofing] | 是 | 否 |
| [限制和配额][lnk-throttles-quotas] | 应用到每个设备 | 应用到现场网关 |

> [AZURE.IMPORTANT]  使用不透明的网关模式时，所有通过该网关连接的设备共享同一云到设备的队列，此队列中最多可包含 50 条消息。因此，不透明的网关模式的使用时机应该是只有很少的设备通过每个现场网关连接，且其云到设备的流量很低。

### 其他注意事项

可以使用 [Azure IoT 设备 SDK][lnk-device-sdks] 来实现现场网关。某些设备 SDK 提供的特定功能可帮助实现现场网关，例如，能够将来自多个设备到通往 IoT 中心的相同连接上的通信多路复用的功能。如同 [IoT 中心开发人员指南 - 选择协议][lnk-devguide-protocol]中所述，应该避免使用 HTTP/1 作为现场网关的传输协议。

## 自定义设备身份验证

可以使用 IoT 中心[设备标识注册表][lnk-devguide-identityregistry]来配置每个设备的安全凭据和访问控制。如果 IoT 解决方案已经大幅投资自定义设备标识注册表和/或身份验证方案，可以通过创建*令牌服务*，将此现有基础结构与 IoT 中心集成。这样，便可以在解决方案中使用其他 IoT 功能。

令牌服务是自定义云服务。它使用包含 **DeviceConnect** 权限的 IoT 中心*共享访问策略*创建*设备范围的*令牌。这些令牌可让设备连接到 IoT 中心。

  ![令牌服务模式的步骤][img-tokenservice]

下面是令牌服务模式的主要步骤：

1. 为 IoT 中心创建包含 [DeviceConnect][lnk-devguide-security] 权限的 **IoT 中心共享访问策略**。可以在 [Azure 门户][lnk-portal]或以编程方式创建此策略。令牌服务使用此策略为它创建的令牌签名。
2. 当设备需要访问 IoT 中心时，将向令牌服务请求已签名的令牌。设备可以使用自定义设备标识注册表/身份验证方案来进行身份验证，以确定令牌服务用来创建令牌的设备标识。
3. 令牌服务返回令牌。使用 `/devices/{deviceId}` 作为 `resourceURI`（其中 `deviceId` 是要身份验证的设备），并根据 [IoT 中心开发人员指南][lnk-devguide-security]的安全部分创建令牌。令牌服务使用共享访问策略来构造令牌。
4. 设备直接通过 IoT 中心使用令牌。

> [AZURE.NOTE] 可以使用 .NET 类 [SharedAccessSignatureBuilder][lnk-dotnet-sas] 或 Java 类 [IotHubServiceSasToken][lnk-java-sas] 在令牌服务中创建令牌。

令牌服务可以根据需要设置令牌过期日期。令牌过期时，IoT 中心将断开设备连接。然后，设备必须向令牌服务请求新令牌。如果使用过短的过期时间，会增加设备与令牌服务上的负载。

为了让设备连接到中心，你仍必须将它添加 IoT 中心设备标识注册表，即使设备使用令牌而不是设备密钥来连接。因此，你可以在设备使用令牌身份验证时，通过在 [IoT 中心识别注册表][lnk-devguide-identityregistry]中启用或禁用设备标识，来继续使用基于设备的访问控制。这可以减轻使用较长过期时间令牌的风险。

### 与自定义网关的比较

令牌服务模式是使用 IoT 中心实现自定义标识注册表/身份验证方案的建议方式。提供这种建议是因为 IoT 中心继续处理大部分解决方案流量。但在某些情况下，自定义身份验证方案和协议过度交织，因此需要可处理所有流量（*自定义网关*）的服务。[传输层安全 (TLS) 和预共享密钥 (PSK)][lnk-tls-psk] 就是这种服务的例子。有关详细信息，请参阅[协议网关][lnk-gateway]主题。

## 设备检测信号 <a id="heartbeat"></a>

[IoT 中心标识注册表][lnk-devguide-identityregistry]包含名为 **connectionState** 的字段。你只应在开发和调试期间使用 **connectionState** 字段，IoT 解决方案不应在运行时查询该字段（例如，为了检查设备是否已连接以确定是否要发送云到设备的消息或 SMS）。如果 IoT 解决方案需要知道设备是否已连接（在运行时，或在比 **connectionState** 属性提供的值更精确时），解决方案应该实施*检测信号模式*。

在检测信号模式下，设备每隔固定时间至少发送一次设备到云的消息（例如，每小时至少一次）。这意味着，即使设备没有任何要发送的数据，仍会发送空的设备到云的消息（通常具有可供识别其属于检测信号的属性）。在服务端，解决方案维护一份图表，其中包含每个设备所收到的最后一次检测信号，并假设如果设备没有在预期时间内收到检测信号消息，即表示设备有问题。

更复杂的实现可以包含来自[操作监视][lnk-devguide-opmon]的信息，以便识别尝试连接或通信但失败的设备。实施检测信号模式时，请务必查看 [IoT 中心配额与限制][]。

## 后续步骤

若要了解有关 Azure IoT 中心的详细信息，请参阅以下链接：

- [IoT 中心入门（教程）][lnk-get-started]
- [Azure IoT 中心是什么？][lnk-what-is-hub]

[img-tokenservice]: ./media/iot-hub-guidance/tokenservice.png

[lnk-devguide-identityregistry]: /documentation/articles/iot-hub-devguide/#identityregistry
[lnk-device-management]: /documentation/articles/iot-hub-device-management
[lnk-devguide-opmon]: /documentation/articles/iot-hub-operations-monitoring

[lnk-device-sdks]: /documentation/articles/iot-hub-sdks-summary
[lnk-devguide-security]: /documentation/articles/iot-hub-devguide/#security
[lnk-tls-psk]: https://tools.ietf.org/html/rfc4279
[lnk-gateway]: /documentation/articles/iot-hub-protocol-gateway

[lnk-get-started]: /documentation/articles/iot-hub-csharp-csharp-getstarted
[lnk-what-is-hub]: /documentation/articles/iot-hub-what-is-iot-hub
[lnk-portal]: https://manage.windowsazure.cn
[lnk-throttles-quotas]: /documentation/articles/azure-subscription-service-limits/#iot-hub-limits
[lnk-devguide-antispoofing]: /documentation/articles/iot-hub-devguide/#antispoofing
[lnk-devguide-protocol]: /documentation/articles/iot-hub-devguide/#amqpvshttp
[lnk-dotnet-sas]: https://msdn.microsoft.com/zh-cn/library/microsoft.azure.devices.common.security.sharedaccesssignaturebuilder.aspx
[lnk-java-sas]: http://azure.github.io/azure-iot-sdks/java/service/api_reference/com/microsoft/azure/iot/service/auth/IotHubServiceSasToken.html
[IoT 中心配额与限制]: /documentation/articles/iot-hub-devguide/#throttling

<!---HONumber=Mooncake_0307_2016-->