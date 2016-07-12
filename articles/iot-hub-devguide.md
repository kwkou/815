<properties
 pageTitle="IoT 中心开发人员指南主题 | Azure"
 description="Azure IoT 中心开发人员指南，其中介绍了 IoT 中心终结点、安全性、设备标识注册表和消息传送"
 services="iot-hub"
 documentationCenter=".net"
 authors="dominicbetts"
 manager="timlt"
 editor=""/>

<tags
 ms.service="iot-hub"
 ms.date="05/29/2016" 
 wacn.date="07/04/2016"/>

# Azure IoT 中心开发人员指南

Azure IoT 中心是一项完全托管的服务，有助于在数百万个 IoT 设备和一个应用程序后端之间实现安全可靠的双向通信。

Azure IoT 中心可帮助提供以下内容：

* 使用每个设备的安全凭据和访问控制来保护通信安全。
* 提供可靠的设备到云和云到设备的超大规模消息传送。
* 通过最流行语言和平台的设备库来方便建立设备连接。

本文涵盖以下主题：

- [终结点](#endpoints)。此部分说明每个 IoT 中心针对运行时和管理操作公开的各种终结点。
- [设备标识注册表](#device-identity-registry)。此部分说明各 IoT 中心的设备标识注册表存储的信息，以及访问和修改这些信息的方式。
- [安全性](#security)。此部分说明用于向设备和云组件授予 IoT 中心功能访问权限的安全模型。
- [消息传送](#messaging)。此部分说明 IoT 中心公开的消息传送功能（设备到云和云到设备）。
- [配额和限制](#throttling)。此部分汇总了 IoT 中心的使用配额。

## 终结点<a id="endpoints"></a>

Azure IoT 中心属于多租户服务，向各种执行组件公开功能。下图显示了 IoT 中心公开的各种终结点。

![IoT 中心终结点][img-endpoints]

以下是终结点的说明：

* **资源提供程序**。IoT 中心资源提供程序公开 [Azure Resource Manager][lnk-arm] 界面，可让 Azure 订阅所有者创建和删除 IoT 中心以及更新 IoT 中心属性。IoT 中心属性可管理中心级别的安全策略，相对于设备级别的访问控制（请参阅本文后面的[访问控制](#accesscontrol)一节）和云到设备及设备到云的消息传送的功能选项。资源提供程序还可让你[导出设备标识](#importexport)。
* **设备标识管理**。每个 IoT 中心公开一组用于管理设备标识的 HTTP REST 终结点（创建、检索、更新和删除）。设备标识用于设备身份验证和访问控制。有关详细信息，请参阅[设备标识注册表](#device-identity-registry)。
* **设备终结点**。对于设备标识注册表中预配的每个设备，IoT 中心公开设备可用来发送和接收消息的一组终结点：
    - 发送设备到云的消息。使用此终结点发送设备到云的消息。有关详细信息，请参阅[设备到云的消息传送](#d2c)。
    - 接收云到设备的消息设备。使用此终结点来接收目标云到设备的消息。有关详细信息，请参阅[云到设备的消息传送](#c2d)。
    - 启动文件上传。设备使用此终结点接收来自 IoT 中心的 Azure 存储空间 SAS URI，以便上传文件。有关详细信息，请参阅[文件上传](#fileupload)。 

    这些终结点是使用 HTTP 1.1、[MQTT v3.1.1][lnk-mqtt] 和 [AMQP 1.0][lnk-amqp] 协议公开的。请注意，也可以通过端口 443 上的 [WebSockets][lnk-websockets] 来实现 AMQP。
* **服务终结点**。每个 IoT 中心公开应用程序后端可用来与设备通信的一组终结点。目前仅使用 [AMQP][lnk-amqp] 协议公开这些终结点。
    - 接收设备到云的消息。此终结点与 [Azure 事件中心][lnk-event-hubs]兼容。后端服务可用它来读取由设备发送的所有设备到云的消息。有关详细信息，请参阅[设备到云的消息传递](#d2c)。
    - 发送云到设备的消息并接收传递确认。这些终结点可让应用程序后端发送可靠的云到设备消息，以及接收对应的传送或过期确认。有关详细信息，请参阅[云到设备的消息传递](#c2d)。
    - 接收文件通知。此消息传递终结点允许你在设备成功上传文件时接收通知。 

[IoT 中心 API 和 SDK][lnk-apis-sdks] 一文介绍了可用于访问这些终结点的各种方式。

最后请务必注意，所有的 IoT 中心终结点都使用 [TLS][lnk-tls] 协议，且绝不会在未加密/不安全的通道上公开任何终结点。

### 如何从事件中心兼容的终结点读取信息<a id="eventhubcompatible"></a>

当你使用[适用于 .NET 的 Azure 服务总线 SDK](https://www.nuget.org/packages/WindowsAzure.ServiceBus) 或[事件中心 - 事件处理器主机][]时，可以将任何 IoT 中心连接字符串与正确的权限配合使用，然后使用**消息/事件**作为事件中心名称。

使用无法识别 IoT 中心的 SDK（或产品集成）时，必须从 [Azure 门户][]的 IoT 中心配置中检索事件中心兼容终结点和事件中心名称：

1. 在 IoT 中心边栏选项卡中单击“设置”>“消息传递”。
2. 在**设备到云的设置**部分中，你可以看到以下值：**事件中心兼容的终结点**、**事件中心兼容的名称**和**分区**。

    ![设备到云的设置][img-eventhubcompatible]

> [AZURE.NOTE] 如果 SDK 需要**主机名**或**命名空间**值，请从**事件中心兼容的终结点**中删除方案。例如，如果事件中心兼容的终结点为 **sb://iothub-ns-myiothub-1234.servicebus.chinacloudapi.cn/**，则**主机名**为 **iothub-ns-myiothub-1234.servicebus.chinacloudapi.cn**，**命名空间**为 **iothub-ns-myiothub-1234**。

然后，可以使用具有 **ServiceConnect** 权限的任何共享访问安全策略连接到指定的事件中心。

如果需要通过使用以前的信息构建事件中心连接字符串，请使用以下模式：

```
Endpoint={Event Hub-compatible endpoint};SharedAccessKeyName={iot hub policy name};SharedAccessKey={iot hub policy key}
```

以下是可以配合 IoT 中心公开的事件中心兼容终结点使用的 SDK 和集成项目列表：

* [Java 事件中心客户端](https://github.com/hdinsight/eventhubs-client)
* [Apache Storm Spout](/documentation/articles/hdinsight-storm-develop-csharp-event-hub-topology/)。可以在 GitHub 上查看 [Spout 源代码](https://github.com/apache/storm/tree/master/external/storm-eventhubs)。
* [Apache Spark 集成](/documentation/articles/hdinsight-apache-spark-eventhub-streaming/)

## 设备标识注册表

每个 IoT 中心都包含一个设备标识注册表。可以使用此注册表在服务中创建各设备资源，如包含进行中云到设备的消息的队列。还可使用此注册表允许访问面向设备的终结点，如[访问控制](#accesscontrol)一节中所述。

概括地说，设备标识注册表是支持 REST 的设备标识资源集合。以下部分将详细说明设备标识资源属性，以及支持对注册表中的标识执行的操作。

> [AZURE.NOTE] 有关可用来与设备标识注册表交互的 HTTP 协议和 SDK 的更多详细信息，请参阅 [IoT 中心 API 和 SDK][lnk-apis-sdks]。

### 设备标识属性 <a id="deviceproperties"></a>

设备识别以包含以下属性的 JSON 文档表示。

| 属性 | 选项 | 说明 |
| -------- | ------- | ----------- |
| deviceId | 必需，更新时只读 | ASCII 7 位字母数字字符 + `{'-', ':', '.', '+', '%', '_', '#', '*', '?', '!', '(', ')', ',', '=', '@', ';', '$', '''}` 的区分大小写字符串（最长为 128 个字符）。 |
| generationId | 必需，只读 | 中心生成的区分大小写字符串，最长为 128 个字符。在删除并重新创建设备时用于区分具有相同 **deviceId** 的设备。 |
| etag | 必需，只读 | 一个字符串，根据 [RFC7232][lnk-rfc7232] 表示设备标识的弱 etag。|
| auth | 可选 | 包含身份验证信息和安全材料的复合对象。 |
| auth.symkey | 可选 | 包含主密钥和辅助密钥的复合对象，以 base64 格式存储。 |
| status | 必填 | 访问指示器。可以是**已启用**或**已禁用**。如果是**已启用**，则允许设备连接。如果是**已禁用**，则此设备无法访问任何面向设备的终结点。 |
| statusReason | 可选 | 128 个字符的字符串，用于存储设备标识状态的原因。允许所有 UTF-8 字符。 |
| statusUpdateTime | 只读 | 临时指示器，显示上次状态更新的日期和时间。 |
| connectionState | 只读 | 指示连接状态的字段：**已连接**或**已断开**。此字段表示设备连接状态的 IoT 中心视图。**重要说明**：此字段只用于开发/调试目的。只有使用 AMQP 或 MQTT 的设备更新连接状态。此外，它基于协议级别的 ping（MQTT ping 或 AMQP ping），并且最多只有 5 分钟的延迟。出于这些原因，可能会发生误报，例如，将设备报告为已连接，但实际上已断开连接。 |
| connectionStateUpdatedTime | 只读 | 临时指示器，显示上次更新连接状态的日期和时间。 |
| lastActivityTime | 只读 | 临时指示器，显示设备上次连接、接收或发送消息的日期和时间。 |

> [AZURE.NOTE] 连接状态只能表示连接状态的 IoT 中心视图。根据网络状态和配置，可能会延迟此状态的更新。

### 设备标识操作

IoT 中心设备标识注册表公开以下操作：

* 创建设备标识
* 更新设备标识
* 按 ID 检索设备标识
* 删除设备标识
* 最多列出 1000 个标识
* 将所有标识导出到 Blob 存储
* 从 Blob 存储导入标识

上述所有操作允许使用 [RFC7232][lnk-rfc7232] 中指定的乐观并发。

> [AZURE.IMPORTANT] 如果要检索中心标识注册表中的所有标识，唯一方法是使用[导出](#importexport)功能。

IoT 中心设备标识注册表：

- 不包含任何应用程序元数据。
- 可以将 **deviceId** 作为键进行访问，就像字典一样。
- 不支持表达性查询。

IoT 解决方案通常具有不同的解决方案特定存储，其中包含应用程序特定的元数据。例如，智能建筑物解决方案中的解决方案特定存储将记录部署温度感应器的房间信息。

> [AZURE.IMPORTANT] 请只将设备标识注册表用于设备管理和预配操作。运行时的高吞吐量操作不应依赖于在设备标识注册表中执行操作。例如，在发送命令前先检查设备的连接状态就是不支持的模式。请务必检查设备标识注册表的[限制速率](#throttling)以及[设备检测信号][lnk-guidance-heartbeat]模式。

### 禁用设备

可以通过更新注册表中标识的**状态**属性来禁用设备。通常在两种情况下使用此操作：

- 在预配协调过程中。有关详细信息，请参阅[设计你的解决方案 - 设备预配][lnk-guidance-provisioning]。
- 你出于任何原因认为设备遭到入侵或未经授权。

### 导入和导出设备标识 <a id="importexport"></a>

可以使用 [IoT 中心资源提供程序终结点](#endpoints)上的异步操作，从 IoT 中心的标识注册表批量导出设备标识。导出是长时间运行的作业，它使用客户提供的 blob 容器来保存从标识注册表读取的设备标识数据。

可以使用 [IoT 中心资源提供程序终结点](#endpoints)上的异步操作，将设备标识批量导入 IoT 中心的标识注册表。导入是长时间运行的作业，它使用客户提供的 blob 容器中的数据，将设备标识数据写入设备标识注册表。

- 有关导入和导出 API 的详细信息，请参阅 [Azure IoT 中心 — 资源提供程序 API][lnk-resource-provider-apis]。
- 若要了解有关如何运行导入和导出作业的详细信息，请参阅[批量管理 IoT 中心的设备标识][lnk-bulk-identity]。

## 安全 <a id="security"></a>

本部分介绍用于保护 Azure IoT 中心的选项。

### 访问控制 <a id="accesscontrol"></a>

IoT 中心使用以下一组*权限*向每个 IoT 中心的终结点授予访问权限。权限可根据功能限制对 IoT 中心的访问。

* **RegistryRead**。授予对设备标识注册表的读取访问权限。有关详细信息，请参阅[设备标识注册表](#device-identity-registry)。
* **RegistryReadWrite**。授予对设备标识注册表的读取和写入访问权限。有关详细信息，请参阅[设备标识注册表](#device-identity-registry)。
* **ServiceConnect**。授予对面向云服务的通信和监视终结点的访问权限。例如，它授权后端云服务接收设备到云的消息、发送云到设备的消息，以及检索对应的传送确认。
* **DeviceConnect**。授予对面向设备的通信终结点的访问权限。例如，它授予发送设备到云的消息和接收云到设备的消息的权限。此权限由设备使用。

可以通过以下方式授予权限：

* **中心级别的共享访问策略**。共享访问策略可以授予前一部分中所列权限的任意组合。你可以在 [Azure 门户][lnk-management-portal]中定义策略，或使用 [Azure IoT 中心资源提供程序 API][lnk-resource-provider-apis] 以编程方式定义策略。新建的 IoT 中心有以下默认策略：

    - **iothubowner**：包含所有权限的策略。
    - **service**：包含 ServiceConnect 权限的策略。
    - **device**：包含 DeviceConnect 权限的策略。
    - **registryRead**：包含 RegistryRead 权限的策略。
    - **registryReadWrite**：包含 RegistryRead 和 RegistryWrite 权限的策略。


* **每个设备的安全凭据**。每个 IoT 中心都包含一个[设备标识注册表](#device-identity-registry)。对于此注册表中的每个设备，可以配置安全凭据，以相应设备终结点为范围来授予 **DeviceConnect** 权限。

例如，在典型的 IoT 解决方案中：
- 设备管理组件使用 registryReadWrite 策略。
- 事件处理器组件使用 service 策略。
- 运行时设备业务逻辑组件使用 service 策略。
- 各个设备的连接使用 IoT 中心的标识注册表中存储的凭据。

有关 IoT 中心安全性主题的指导，请参阅[设计你的解决方案][lnk-guidance-security]中的“安全”部分。

### 身份验证

Azure IoT 中心可根据共享访问策略和设备标识注册表安全凭据来验证令牌，以授予对终结点的访问权限。

安全凭据（例如对称密钥）永远不会通过网络发送。

> [AZURE.NOTE] 如同 [Azure Resource Manager][lnk-azure-resource-manager] 中的所有提供程序一样，Azure IoT 中心资源提供程序也通过 Azure 订阅受到保护。

有关如何构造和使用安全令牌的详细信息，请参阅 [IoT 中心安全令牌][lnk-sas-tokens]。

#### 协议详情

每个支持的协议（例如 AMQP、MQTT 和 HTTP）以不同的方式传输令牌。


HTTP 通过在**授权**请求标头中包含有效的令牌来实施身份验证。

使用 [AMQP][lnk-amqp] 时，IoT 中心支持 [SASL PLAIN][lnk-sasl-plain] 和 [AMQP 基于声明的安全性][lnk-cbs]。

对于 AMQP 基于声明的安全性，标准将指定如何传输这些令牌。

对于 SASL PLAIN，**用户名**可以是：

* 如果是中心级别的令牌，则为 `{policyName}@sas.root.{iothubName}`。
* 如果是设备范围的令牌，则为 `{deviceId}`。

在这两种情况下，密码字段都包含令牌，如 [IoT 中心安全令牌][lnk-sas-tokens]一文中所述。

使用 MQTT 时，CONNECT 数据包具有用作 ClientId 的 deviceId，在 Username 字段中具有 {iothubhostname}/{deviceId}，在 Password 字段中具有 SAS 令牌。{iothubhostname} 应是 IoT 中心的完整 CName（例如，contoso.azure-devices.cn）。

##### 示例： #####

用户名（DeviceId 区分大小写）：
`iothubname.azure-devices.cn/DeviceId`

密码（使用设备资源管理器生成 SAS）：`SharedAccessSignature sr=iothubname.azure-devices.cn%2fdevices%2fDeviceId&sig=kPszxZZZZZZZZZZZZZZZZZAhLT%2bV7o%3d&se=1487709501`

> [AZURE.NOTE] [Azure IoT 中心 SDK][lnk-apis-sdks] 在连接到服务时自动生成令牌。在某些情况下，SDK 不支持所有的协议或所有身份验证方法。

#### 有关 SASL PLAIN 的特殊注意事项

使用 SASL PLAIN 时，连接到 IoT 中心的客户端可为每个 TCP 连接使用单个令牌。当令牌过期时，TCP 将从服务断开连接，并触发重新连接。此行为虽然不对应用程序后端组件造成问题，但对设备端应用程序极为不利，原因如下：

*  网关通常代表许多设备连接。使用 SASL PLAIN 时，它们必须针对连接到 IoT 中心的每个设备创建不同的 TCP 连接。这会大幅提高电源与网络资源的消耗并增大每个设备连接的延迟。
* 在每个令牌过期后，增加使用要重新连接的资源通常会对资源受限的设备造成不良影响。

### 设置中心级凭据的范围

可以通过使用受限资源 URI 创建令牌，来设置中心级安全策略的范围。例如，要从设备发送设备到云的消息的终结点是 **/devices/{deviceId}/messages/events**。还可以使用包含 **DeviceConnect** 权限的中心级别共享访问策略对 resourceURI 为 **/devices/{deviceId}** 的令牌进行签名。这将创建一个令牌，仅可用于发送代表设备的 **deviceId**。

此机制类似于[事件中心发布者策略][lnk-event-hubs-publisher-policy]，可用于实施自定义身份验证方法。有关详细信息，请参阅[设计你的解决方案][lnk-guidance-security]的“安全”部分。

## 消息传递

IoT 中心提供消息传送基元进行通信：
- [云到设备](#c2d)：从应用程序后端（服务或云）。
- [设备到云](#d2c)：从设备到应用程序后端。
- 从设备到关联的 Azure 存储帐户的[文件上传](#fileupload)。 

IoT 中心消息传送功能的核心属性是消息的可靠性和持久性。这可在设备端上恢复间歇性连接，以及在云恢复事件处理的负载高峰。IoT 中心对设备到云和云到设备的消息传送实施至少一次传送保证。

IoT 中心支持多个面向设备的协议（例如 MQTT、AMQP 和 HTTP）。为了支持无缝的跨协议互操作性，IoT 中心定义了所有面向设备的协议均可支持的通用消息格式。

### 消息格式 <a id="messageformat"></a>

IoT 中心消息包含：

* 一组系统属性。这些是 IoT 中心解释或设置的属性。此集合是预先确定的。
* 一组应用程序属性。这是应用程序可以定义的字符串属性字典，而不需将消息正文反序列化即可进行访问。IoT 中心永不修改这些属性。
* 不透明的二进制正文。

有关在不同的协议中如何对消息进行编码的详细信息，请参阅 [IoT 中心 API 和 SDK][lnk-apis-sdks]。

这是 IoT 中心消息中的系统属性集。

| 属性 | 说明 |
| -------- | ----------- |
| MessageId | 用户可设置的消息标识符，通常用于请求-答复模式。格式：ASCII 7 位字母数字字符 + `{'-', ':',’.', '+', '%', '_', '#', '*', '?', '!', '(', ')', ',', '=', '@', ';', '$', '''}` 的区分大小写字符串（最长为 128 个字符）。 |
| 序列号 | IoT 中心分配给每条云到设备消息的编号（对每个设备队列是唯一的）。 |
| 如果 | [云到设备](#c2d)消息中指定的目标。 |
| ExpiryTimeUtc | 消息过期的日期和时间。 |
| EnqueuedTime | IoT 中心收到消息的日期和时间。 |
| CorrelationId | 响应消息中的字符串属性，通常包含采用“请求-答复”模式的请求的 MessageId。 |
| UserId | 用于指定消息的源的 ID。如果消息是由 IoT 中心生成的，则设置为 `{iot hub name}`。 |
| Ack | 反馈消息生成器。此属性在云到设备的消息中用于请求 IoT 中心因为设备使用消息而生成反馈消息。可能的值：**none**（默认值）：不生成任何反馈消息；**positive**：如果消息已完成，则接收反馈消息；**negative**：如果消息未由设备完成就过期（或已达到最大传送计数），则收到反馈消息；**full**：positive 和 negative。有关详细信息，请参阅[消息反馈](#feedback)。 |
| ConnectionDeviceId | IoT 中心对设备到云的消息设置的 ID。它包含发送消息的设备的 **deviceId**。 |
| ConnectionDeviceGenerationId | IoT 中心对设备到云的消息设置的 ID。它包含发送消息的设备的 **generationId**（根据[设备标识属性](#deviceproperties)）。 |
| ConnectionAuthMethod | 由 IoT 中心对设备到云的消息设置的身份验证方法。此属性包含用于验证发送消息的设备的身份验证方法的相关信息。有关详细信息，请参阅[设备到云的反欺骗技术](#antispoofing)。|

### 选择通信协议 <a id="amqpvshttp"></a>

在设备端通信方面，Iot 中心支持 [AMQP][lnk-amqp]、基于 WebSockets 的 AMQP、MQTT 和 HTTP/1 协议。关于它们的用途，请考虑以下几点。

* **云到设备模式**。HTTP/1 不会以有效的方式实现服务器推送。因此，使用 HTTP/1 时，设备将在 IoT 中心轮询云到设备的消息。这对于设备和 IoT 中心而言是非常低效的。根据当前 HTTP/1 准则，每个设备每 25 分钟或更长时间进行轮询。另一方面，接受云到设备的消息时，AMQP 和 MQTT 支持服务器推送。它们将启用从 IoT 中心到设备的直接消息推送。如果传送延迟是一大考虑因素，最好使用 AMQP 或 MQTT 协议。对于很少连接的设备，HTTP/1 也适用。
* **现场网关**。使用 HTTP/1 和 MQTT 时，无法使用相同的 TLS 连接来连接到多个设备（各有自身的设备凭据）。因此，对于[现场网关方案][lnk-azure-gateway-guidance]，这些并不是最理想的协议，因为对于每个连接到现场网关的设备，这些协议需要在现场网关和 IoT 中心之间建立一个 TLS 连接。
* **低资源设备**。MQTT 和 HTTP/1 库的占用空间比 AMQP 库更小。因此，如果设备拥有的资源很少（例如，小于 1 MB RAM），这些协议可能是唯一可用的协议实现。
* **网络遍历**。MQTT 标准在端口 8883 上侦听。这可能会导致对非 HTTP 协议关闭的网络发生问题。HTTP 和 AMQP（基于 WebSockets）均可用于此方案。
* **有效负载大小**。AMQP 和 MQTT 是二进制协议，因此明显比 HTTP/1 更精简。

一般说来，应尽可能使用 AMQP（或基于 WebSockets 的 AMQP），而且仅当资源的约束阻止使用 AMQP 时才使用 MQTT。应仅在网络遍历和网络配置都阻止使用 MQTT 和 AMQP 时才使用 HTTP/1。此外，在使用 HTTP/1 时，每个设备应该每隔 25 分钟或以上轮询一次云到设备的消息。

> [AZURE.NOTE] 在开发期间可以 25 分钟以下的时间间隔进行轮询。

<a id="mqtt-support"></a>
#### 有关 MQTT 支持的说明
IoT 中心实现 MQTT v3.1.1 协议，但具有以下限制和特定行为：

  * **不支持 QoS 2**。如果设备客户端使用 **QoS 2** 发布消息，IoT 中心将关闭网络连接。当设备客户端使用 **QoS 2** 订阅主题时，IoT 中心将在 **SUBACK** 数据包中授予最大 QoS 级别 1。
  * **不会保留保留消息**。如果设备客户端发布 RETAIN 标志设置为 1 的消息，IoT 中心将在消息中添加 **x-opt-retain** 应用程序属性。这意味着 IoT 中心不会持续保留消息，而是将它传递到后端应用程序。

有关详细信息，请参阅 [IoT 中心 MQTT 支持][lnk-mqtt-support]。

最后一个注意事项，你应查看 [Azure IoT 协议网关][lnk-azure-protocol-gateway]。这使你能够部署与 IoT 中心直接连接的高性能自定义协议网关。Azure IoT 协议网关可让你自定义设备协议，以适应要重建的 MQTT 部署或其他自定义协议。但是，这种方法确实要求你自承载并运行自定义协议网关。

### 设备到云 <a id="d2c"></a>

详见[终结点](#endpoints)一节，通过面向设备的终结点 (**/devices/{deviceId}/messages/events**) 发送设备到云的消息。通过面向服务的终结点 (**/messages/events**) 接收消息，该终结点与[事件中心][lnk-event-hubs]兼容。因此，你可以使用标准事件中心集成和 SDK 来接收设备到云的消息。

IoT 中心使用类似于[事件中心][lnk-event-hubs]的方式实现设备到云的消息传递。IoT 中心的设备到云的消息更像是事件中心事件，而不是[服务总线][lnk-servicebus]消息。

这种实现具有以下含义：

* 与事件中心事件类似，设备到云的消息可持久保留在 IoT 中心多达 7 天（请参阅[设备到云的配置选项](#d2cconfiguration)）。
* 设备到云的消息分区存放到创建时设置的一组固定分区中（请参阅[设备到云的配置选项](#d2cconfiguration)）。
* 与事件中心类似，读取设备到云消息的客户端必须处理数据分区和检查点。请参阅[事件中心 — 使用事件][lnk-event-hubs-consuming-events]。
* 如同事件中心的事件，设备到云的消息最大可为 256 KB，而且可分成多个批以优化发送。批最大可为 256 KB，最多包含 500 条消息。

不过，IoT 中心的设备到云的消息传递与事件中心之间还有一些重要差异：

* 如[安全](#security)部分中所述，IoT 中心允许基于设备的身份验证和访问控制。
* IoT 中心允许同时连接数百万个设备（请参阅[配额和限制](#throttling)），而事件中心则限制为每个命名空间 5000 个 AMQP 连接。
* IoT 中心不允许使用 **PartitionKey** 任意分区。设备到云的消息根据其源于的 **deviceId** 进行分区。
* IoT 中心的缩放方式与事件中心稍有不同。有关详细信息，请参阅[缩放 IoT 中心][lnk-guidance-scale]。

请注意，这并不表示在所有情况下都能使用 IoT 中心代替事件中心。例如，在某些事件处理计算中，可能需要在分析数据流之前，根据不同属性或字段重新分区事件。在这种情况下，你可以使用事件中心来减少流处理管道的两个部分。有关详细信息，请参阅 [Azure 事件中心概述][lnk-eventhub-partitions]中的分区。

有关如何使用设备到云的消息传送的详细信息，请参阅 [IoT 中心 API 和 SDK][lnk-apis-sdks]。

> [AZURE.NOTE] 使用 HTTP 发送设备到云的消息时，属性名称和值只能包含 ASCII 字母数字字符加上 ``{'!', '#', '$', '%, '&', "'", '*', '*', '+', '-', '.', '^', '_', '`', '|', '~'}``。

#### 非遥测流量

在许多情况下，除了遥测数据点之外，设备还会发送消息以及要求执行和处理应用程序业务逻辑层的请求。例如，必须在后端触发特定操作的关键警报，或者对由后端发送的命令做出的设备响应。

有关这类消息的最佳处理方式的详细信息，请参阅[设备到云的处理][lnk-guidance-d2c-processing]。

#### 设备到云的配置选项 <a id="d2cconfiguration"></a>

IoT 中心公开以下属性让你控制设备到云的消息传送。

* **分区计数**。在创建时设置此属性，以便为设备到云事件引入定义分区数。
* **保留时间**。此属性指定设备到云消息的保留时间。默认值为一天，但可以增加到七天。

此外，类似于事件中心，IoT 中心也可让你管理设备到云接收终结点上的“使用者组”。

可以通过 [Azure IoT 中心 - 资源提供程序 API][lnk-management-portal] 以编程方式修改上述所有属性，或使用 [Azure 门户][lnk-resource-provider-apis]进行修改。

#### 反欺骗属性 <a id="antispoofing"></a>

为了避免设备到云的消息中出现设备欺骗，IoT 中心使用以下属性在所有消息上加上戳记：

* **ConnectionDeviceId**
* **ConnectionDeviceGenerationId**
* **ConnectionAuthMethod**

根据[设备标识属性](#deviceproperties)，前两个属性包含源设备的 **deviceId** 和 **generationId**。

**ConnectionAuthMethod** 属性包含具有以下属性的 JSON 序列化对象：

```
{
  "scope": "{ hub | device}",
  "type": "{ symkey | sas}",
  "issuer": "iothub"
}
```

### 云到设备 <a id="c2d"></a>

如[终结点](#endpoints)部分所详述，你可以通过面向服务的终结点 (**/messages/devicebound**) 发送云到设备的消息。设备可以通过特定于设备的终结点 (**/devices/{deviceId}/messages/devicebound**) 接收它们。

每个云到设备的消息都以单个设备为目标，并将 **to** 属性设置为 **/devices/{deviceId}/messages/devicebound**。

>[AZURE.IMPORTANT] 每个设备队列最多可以保留 50 条云到设备的消息。尝试将更多消息传送到同一设备将导致错误。

> [AZURE.NOTE] 发送云到设备的消息时，属性名称和值只能包含 ASCII 字母数字字符加上 ``{'!', '#', '$', '%, '&', "'", '*', '*', '+', '-', '.', '^', '_', '`', '|', '~'}``。

#### 消息生命周期 <a id="message lifecycle"></a>

为了至少一次实现消息传递保证，云到设备的消息将保留在每个设备队列中。设备必须显式确认完成，IoT 中心才会从队列中将其删除。这是为了保证连接失败和设备故障时能够恢复。

下图显示了云到设备消息的生命周期状态图。

![云到设备的消息生命周期][img-lifecycle]

当服务发送消息时，该消息被视为已排队。当设备想要接收消息时，IoT 中心将锁定该消息（将状态设置为**不可见**），以便让同一设备上的其他线程开始接收其他消息。当设备线程完成消息的处理后，将通过完成消息来通知 IoT 中心。

设备还可以：

- 拒绝消息，这会使 IoT 中心将此消息设置为**死信**状态。
- 放弃消息，这会使 IoT 中心将消息放回队列，并将状态设置为**已排队**。

线程可能无法处理消息，且不通知 IoT 中心。在此情况下，在可见性（或锁定）超时时间之后，消息将从**不可见**状态自动转换回**已排队**状态。此超时的默认值为一分钟。

消息可以在**已排队**与**不可见**状态之间转换的次数，以 IoT 中心上**最大传递计数**属性中指定的次数为上限。在该转换次数之后，IoT 中心会将消息的状态设置为**死信**。同样，IoT 中心也会在消息的到期时间之后（请参阅[生存时间](#ttl)），将消息的状态设置为**死信**。

有关云到设备的消息的教程，请参阅 [Azure IoT 中心云到设备的消息入门][lnk-getstarted-c2d-tutorial]。有关不同 API 和 SDK 如何公开云到设备功能的参考主题，请参阅 [IoT 中心 API 和 SDK][lnk-apis-sdks]。

> [AZURE.NOTE] 通常只要丢失消息不影响应用程序逻辑，就会完成云到设备的消息。例如，消息内容已成功保留在本地存储空间中，或已成功执行某操作。该消息还可能携带暂时性信息，该信息的丢失不会影响应用程序的功能。有时，对于长时间运行的任务，你可以在将任务说明保留到本地存储空间后完成该云到设备的消息。然后，在作业进度的不同阶段，你可以使用一条或多条设备到云的消息通知应用程序后端。

#### 消息有效期（生存时间）<a id="ttl"></a>

每条云到设备的消息都有过期时间。该时间可以由服务（在 **ExpiryTimeUtc** 属性中）显式设置，或者由 IoT 中心使用指定为 IoT 中心属性的默认生存时间来设置。请参阅[云到设备的配置选项](#c2dconfiguration)。

> [AZURE.NOTE] 利用消息到期时间的常见方法是设置较短的生存时间值，以避免将消息发送到已断开连接的设备。此做法可达到与维护设备连接状态一样的效果，而且更加有效。通过请求消息确认，IoT 中心可以通知哪些设备可以接收消息、哪些设备脱机或不能接收消息。

#### 消息反馈 <a id="feedback"></a>

当你发送云到设备的消息时，服务可以请求传送每条消息的反馈（关于该消息的最终状态）。

- 如果将 **Ack** 属性设置为 **positive**，则当且仅当云到设备的消息达到**已完成**状态时，IoT 中心才生成反馈消息。
- 如果将 **Ack** 属性设置为 **negative**，则当且仅当云到设备的消息达到**死信**状态时，IoT 中心才生成反馈消息。
- 如果将 **Ack** 属性设置为 **full**，则 IoT 中心在上述任一情况下都会生成反馈消息。

> [AZURE.NOTE] 如果 **Ack** 为 **full**，且未收到反馈消息，则意味着反馈消息已过期。该服务无法了解原始消息的经历。实际上，服务应该确保它可以在反馈过期之前对其进行处理。最长过期时间是两天，因此当发生失败时，有相当充裕的时间让服务再次运行。

如[终结点](#endpoints)中所述，IoT 中心通过面向服务的终结点 (**/messages/servicebound/feedback**) 以消息方式传送反馈。接收反馈的语义与云到设备的消息的语义相同，并且具有相同的 [消息生命周期](#message lifecycle)。可能的话，消息反馈将放入单个消息中，其格式如下。

设备从反馈终结点检索的每条消息具有以下属性：

| 属性 | 说明 |
| -------- | ----------- |
| EnqueuedTime | 指示创建消息时的时间戳。 |
| UserId | `{iot hub name}` |
| ContentType | `application/vnd.microsoft.iothub.feedback.json` |

正文是记录的 JSON 序列化数组，每条记录具有以下属性：

| 属性 | 说明 |
| -------- | ----------- |
| EnqueuedTimeUtc | 指示消息结果出现时的时间戳。例如，设备已完成或消息已过期。 |
| OriginalMessageId | 此反馈信息所属的云到设备的消息的 **MessageId**。 |
| StatusCode | 必须是整数。在 IoT 中心生成的反馈消息中使用。<br/> 0 = 成功 <br/> 1 = 消息已过期 <br/> 2 = 超出最大传送计数 <br/> 3 = 消息被拒绝 |
| 说明 | **StatusCode** 的字符串值。 |
| DeviceId | 此反馈信息所属的云到设备的消息的目标设备的 **DeviceId**。 |
| DeviceGenerationId | 此反馈信息所属的云到设备的消息的目标设备的 **DeviceGenerationId**。 |


>[AZURE.IMPORTANT] 服务必须指定云到设备的消息的 **MessageId**，才能将其反馈与原始消息相关联。

以下示例演示了反馈消息的正文。

```
[
  {
    "OriginalMessageId": "0987654321",
    "EnqueuedTimeUtc": "2015-07-28T16:24:48.789Z",
    "StatusCode": 0
    "Description": "Success",
    "DeviceId": "123",
    "DeviceGenerationId": "abcdefghijklmnopqrstuvwxyz"
  },
  {
    ...
  },
  ...
]
```

#### 云到设备的配置选项 <a id="c2dconfiguration"></a>

每个 IoT 中心都针对云到设备的消息传送公开以下配置选项。

| 属性 | 说明 | 范围和默认值 |
| -------- | ----------- | ----------------- |
| defaultTtlAsIso8601 | 云到设备消息的默认 TTL。 | ISO\_8601 间隔高达 2D（最小为 1 分钟）。默认值：1 小时。 |
| maxDeliveryCount | 每个设备队列的云到设备最大传送计数。 | 1 到 100。默认值：10。 |
| feedback.ttlAsIso8601 | 服务绑定反馈消息的保留时间。 | ISO\_8601 间隔高达 2D（最小为 1 分钟）。默认值：1 小时。 |
| feedback.maxDeliveryCount | 反馈队列的最大传送计数。 | 1 到 100。默认值：100。 |

有关详细信息，请参阅[管理 IoT 中心][lnk-manage]。

### 文件上传 <a id="fileupload"></a>

如[终结点](#endpoints)一节所详述，设备可以通过面向设备的终结点 (**/devices/{deviceId}/files**) 发送通知以启动文件上传。当设备通知 IoT 中心已完成上传时，IoT 中心将生成文件上传通知，你可以通过面向服务的终结点 (**/messages/servicebound/filenotifications**) 将其作为消息接收。

IoT 中心本身不中转消息，而是充当关联 Azure 存储帐户的调度程序。设备请求来自 IoT 中心的存储令牌，该令牌特定于设备要上传的文件。设备使用 SAS URI 将文件上传到存储空间，上传完成后，设备将完成通知发送到 IoT 中心。IoT 中心验证该文件已上传，然后将文件上传通知添加到新的面向服务的文件通知消息传递终结点。

#### 将 Azure 存储帐户与 IoT 中心相关联

若要使用文件上传功能，必须首先将 Azure 存储帐户链接到 IoT 中心。可以通过 [Azure 门户][lnk-management-portal]实现此操作，或通过 [Azure IoT 中心 — 资源提供程序 API][lnk-resource-provider-apis] 以编程方式实现此操作。将存储帐户与 IoT 中心关联后，当设备启动文件上传请求时，此服务将向该设备返回 SAS URI。

> [AZURE.NOTE] [Azure IoT 中心 SDK][lnk-apis-sdks] 自动处理检索 SAS URI、上传文件和通知 IoT 中心已完成上传。

#### 初始化文件上传

IoT 中心有两个 REST 终结点支持文件上传，一个用于获取存储空间的 SAS URI，另一个用于通知 IoT 中心已完成上传。设备通过在 `{iot hub}.azure-devices.cn/devices/{deviceId}/files/{filename}` 向 IoT 中心发送 GET 来启动文件上传过程。该中心将返回特定于要上传的文件的 SAS URI，以及上传完成时要使用的相关性 ID。

#### 通知 IoT 中心已完成文件上传

设备负责使用 Azure 存储 SDK 将文件上传到存储空间。上传完成后，设备将使用从初始 GET 收到的相关性 ID 在 `{iot hub}.azure-devices.cn/devices/{deviceId}/files/notifications/{correlationId}` 向 IoT 中心发送 POST。

#### 文件上传通知

当设备上传文件并通知 IoT 中心上传完成时，该服务将选择性地生成包含该文件名称和存储位置的通知消息。

如[终结点](#endpoints)中所述，IoT 中心通过面向服务的终结点 (**/messages/servicebound/feedback**) 以消息的形式传递文件上传通知。上传通知的接收语义与云到设备的消息的接收语义相同，并且具有相同的 [消息生命周期](#message lifecycle)。从文件上传通知终结点检索到的每条消息都是具有以下属性的 JSON 记录：

| 属性 | 说明 |
| -------- | ----------- |
| EnqueuedTimeUtc | 指示通知创建时间的时间戳。 |
| DeviceId | 上传文件的设备的 **DeviceId**。 |
| BlobUri | 已上传文件的 URI。 |
| BlobName | 已上传文件的名称。 |
| LastUpdatedTime | 指示文件更新时间的时间戳。 |
| BlobSizeInBytes | 已上传文件的大小。 |

**示例**。这是文件上传通知消息的示例正文。

```
{
	"deviceId":"mydevice",
	"blobUri":"https://{storage account}.blob.core.chinacloudapi.cn/{container name}/mydevice/myfile.jpg",
	"blobName":"mydevice/myfile.jpg",
	"lastUpdatedTime":"2016-06-01T21:22:41+00:00",
	"blobSizeInBytes":1234,
	"enqueuedTimeUtc":"2016-06-01T21:22:43.7996883Z"
}
```

#### 文件上传通知配置选项 <a id="c2dconfiguration"></a>

每个 IoT 中心都为文件上传通知公开以下配置选项：

| 属性 | 说明 | 范围和默认值 |
| -------- | ----------- | ----------------- |
| **enableFileUploadNotifications** | 控制是否将文件上传通知写入文件通知终结点。 | 布尔型。默认值：True。 |
| **fileNotifications.ttlAsIso8601** | 文件上传通知的默认 TTL。 | ISO\_8601 间隔上限为 48 小时（下限为 1 分钟）。默认值：1 小时。 |
| **fileNotifications.lockDuration** | 文件上传通知队列的锁定持续时间。 | 5 到 300 秒（最小为 5 秒）。默认值：60 秒。 |
| **fileNotifications.maxDeliveryCount** | 文件上传通知队列的最大传递计数。 | 1 到 100。默认值：100。 |

有关更多信息，请参阅[管理 IoT 中心][lnk-manage]。

## 配额和限制 <a id="throttling"></a>

每个 Azure 订阅最多可以有 10 个 IoT 中心。

每个 IoT 中心预配了特定 SKU 的特定单位数（有关详细信息，请参阅 [Azure IoT 中心定价][lnk-pricing]）。SKU 和单位数目确定可以发送的消息的每日配额上限。

SKU 还确定了 IoT 中心对所有操作强制实施的限制。

### 操作限制

操作限制是在分钟范围内应用的速率限制，主要是为了避免不当使用。IoT 中心会尽可能避免返回错误，但如果违反限制太久，就会开始返回异常。

以下是强制实施的限制列表。值与单个中心相关。

| 限制 | 每个中心的值 |
| -------- | ------------- |
| 标识注册表操作（创建、检索、列出、更新、删除） | 100/分钟/单位，最高 5000/分钟 |
| 设备连接 | 120/秒/单位（适用于 S2）, 12/秒/单位（适用于 S1）。<br/>最小值为 100/秒。<br/>例如，两个 S1 单位是 2\*12 = 24/秒，但是在所有单位中至少有 100/秒。如果有 9 个 S1 单位，则所有单位就有 108/秒 (9\*12)。 |
| 设备到云的发送 | 120/秒/单位（适用于 S2）, 12/秒/单位（适用于 S1）。<br/>最小值为 100/秒。<br/>例如，两个 S1 单位是 2\*12 = 24/秒，但是在所有单位中至少有 100/秒。如果有 9 个 S1 单位，则所有单位就有 108/秒 (9\*12)。 |
| 云到设备的发送 | 100/分钟/单位 |
| 云到设备的接收 | 1000/分钟/单位 |
| 文件上载操作 | 100 个文件上传通知/分钟/单位 <br/> 一个存储帐户一次可发出 10000 个 SAS URI <br/> 一次可发出 10 个 SAS URI/设备 | 

必须清楚地知道，设备连接限制控制可与 IoT 中心建立新设备连接的速率，而不是控制同时连接的设备数上限。该限制取决于为中心预配的单位数。

例如，如果你购买的是单一 S1 单位，则限制为每秒 100 个连接。这意味着，若要连接 100,000 台设备，至少需要花费 1000 秒（大约 16 分钟）。但是，同时连接的设备数可与你在设备标识注册表中注册的设备数相同。

有关 IoT 中心限制行为的深入讨论，请参阅博客文章 [IoT Hub throttling and you（IoT 中心限制与你息息相关）][lnk-throttle-blog]。

>[AZURE.NOTE] 无论何时，都可以通过增加 IoT 中心预配的单位来提高配额或限制。

>[AZURE.IMPORTANT] 标识注册表操作用于设备管理与预配方案中的运行时使用。通过[导入/导出作业](#importexport)可以支持读取或更新大量的设备标识。

## 后续步骤

现在你已了解开发 IoT 中心的概述，请查看以下内容以了解详细信息：

- [IoT 中心入门（教程）][lnk-get-started]
- [操作系统平台和硬件兼容性][lnk-compatibility]
- [Azure IoT 开发人员中心][lnk-iotdev]
- [设计你的解决方案][lnk-guidance]

[事件中心 - 事件处理器主机]: http://blogs.msdn.com/b/servicebus/archive/2015/01/16/event-processor-host-best-practices-part-1.aspx

[Azure 门户]: https://manage.windowsazure.cn

[img-endpoints]: ./media/iot-hub-devguide/endpoints.png
[img-lifecycle]: ./media/iot-hub-devguide/lifecycle.png
[img-eventhubcompatible]: ./media/iot-hub-devguide/eventhubcompatible.png

[lnk-compatibility]: /documentation/articles/iot-hub-tested-configurations/
[lnk-apis-sdks]: /documentation/articles/iot-hub-sdks-summary/
[lnk-pricing]: /pricing/details/iot-hub
[lnk-resource-provider-apis]: https://msdn.microsoft.com/zh-cn/library/mt548492.aspx

[lnk-sas-tokens]: /documentation/articles/iot-hub-sas-tokens/
[lnk-azure-gateway-guidance]: /documentation/articles/iot-hub-guidance/#field-gateways
[lnk-guidance-provisioning]: /documentation/articles/iot-hub-guidance/#provisioning
[lnk-guidance-scale]: /documentation/articles/iot-hub-scaling/
[lnk-guidance-security]: /documentation/articles/iot-hub-guidance/#customauth
[lnk-guidance-heartbeat]: /documentation/articles/iot-hub-guidance/#heartbeat

[lnk-azure-protocol-gateway]: /documentation/articles/iot-hub-protocol-gateway/
[lnk-get-started]: /documentation/articles/iot-hub-csharp-csharp-getstarted/
[lnk-guidance]: /documentation/articles/iot-hub-guidance/
[lnk-getstarted-c2d-tutorial]: /documentation/articles/iot-hub-csharp-csharp-c2d/

[lnk-amqp]: https://www.amqp.org/
[lnk-mqtt]: http://mqtt.org/
[lnk-websockets]: https://tools.ietf.org/html/rfc6455
[lnk-arm]: /documentation/articles/resource-group-overview/
[lnk-azure-resource-manager]: /documentation/articles/resource-group-overview/
[lnk-cbs]: https://www.oasis-open.org/committees/download.php/50506/amqp-cbs-v1%200-wd02%202013-08-12.doc
[lnk-createuse-sas]: /documentation/articles/storage-dotnet-shared-access-signature-part-2/
[lnk-event-hubs-publisher-policy]: https://code.msdn.microsoft.com/Service-Bus-Event-Hub-99ce67ab
[lnk-event-hubs]: /documentation/services/event-hubs/
[lnk-event-hubs-consuming-events]: /documentation/articles/event-hubs-programming-guide/#event-consumers
[lnk-guidance-d2c-processing]: /documentation/articles/iot-hub-csharp-csharp-process-d2c/
[lnk-management-portal]: https://manage.windowsazure.cn
[lnk-rfc7232]: https://tools.ietf.org/html/rfc7232
[lnk-sasl-plain]: http://tools.ietf.org/html/rfc4616
[lnk-servicebus]: /documentation/services/service-bus/
[lnk-tls]: https://tools.ietf.org/html/rfc5246
[lnk-iotdev]: /develop/iot/
[lnk-bulk-identity]: /documentation/articles/iot-hub-bulk-identity-mgmt/
[lnk-eventhub-partitions]: /documentation/articles/event-hubs-overview/#partitions
[lnk-manage]: /documentation/articles/iot-hub-manage-through-portal/
[lnk-mqtt-support]: /documentation/articles/iot-hub-mqtt-support/
[lnk-throttle-blog]: https://azure.microsoft.com/blog/iot-hub-throttling-and-you/
<!---HONumber=Mooncake_0627_2016-->