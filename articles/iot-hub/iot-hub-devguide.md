<properties
    pageTitle="Azure IoT 中心开发人员指南 | Azure"
    description="Azure IoT 中心开发人员指南讨论了终结点、安全性、标识注册表、设备管理、直接方法、设备孪生、文件上传、作业、IoT 中心查询语言以及消息传送。"
    services="iot-hub"
    documentationcenter=".net"
    author="dominicbetts"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="d534ff9d-2de5-4995-bb2d-84a02693cb2e"
    ms.service="iot-hub"
    ms.devlang="multiple"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="01/31/2017"
    wacn.date="03/10/2017"
    ms.author="dobett" />

# Azure IoT 中心开发人员指南
Azure IoT 中心是一项完全托管的服务，有助于在数百万台设备和单个解决方案后端之间实现安全可靠的双向通信。

Azure IoT 中心提供：

* 使用每个设备的安全凭据和访问控制来保护通信安全。
* 多个设备到云和云到设备的超大规模通信选项。
* 各设备状态信息和元数据的可查询存储。
* 通过最流行语言和平台的设备库来方便建立设备连接。

本 IoT 中心开发人员指南包括以下文章：

* [使用 IoT 中心发送和接收消息][devguide-messaging]说明 IoT 中心公开的消息传送功能（设备到云和云到设备）。此文章还介绍了以下主题：消息路由、消息格式、支持的通信协议及其使用的端口号等。
* [设备到云通信指南][lnk-d2c-guidance]可帮助在设备到云消息、设备孪生的报告属性和文件上传之间做出选择。
* [云到设备的通信指南][lnk-c2d-guidance]有助于在直接方法、设备孪生的所需属性和云到设备的消息之间做出选择。
* [从设备上传文件][devguide-upload]介绍如何从设备上传文件。此文章还介绍了上传过程可发送的通知等主题。
* [管理 IoT 中心中的设备标识][devguide-identities]介绍了每个 IoT 中心的标识注册表存储哪些信息，并介绍了如何访问和修改此信息。
* [控制对 IoT 中心的访问][devguide-security]说明用于向设备和云组件授予 IoT 中心功能访问权限的安全模型。此文章包括有关使用令牌和 X.509 证书的信息，以及可以授予的权限的详细信息。
* [使用设备孪生同步状态和配置][devguide-device-twins]介绍了*设备孪生*的概念及其公开的功能（如使用设备孪生同步设备）。此文章包括有关设备孪生中存储的数据的信息。
* [在设备上调用直接方法][devguide-directmethods]说明直接方法的生命周期，介绍有关如何从后端应用调用设备方法以及在设备上处理直接方法的信息。
* [在多台设备上计划作业][devguide-jobs]介绍如何在多台设备上计划作业。此文章介绍了如何提交作业，这些作业在执行直接方法和利用设备孪生更新设备时执行任务。它还介绍如何查询作业的状态。
* [参考 - IoT 中心终结点][devguide-endpoints]说明了每个 IoT 中心针对运行时和管理操作公开的各种终结点。此文章还介绍如何在 IoT 中心创建附加终结点，以及如何使用现场网关使设备连接到非标准方案中的 IoT 中心终结点。
* [参考 - 设备孪生和作业的 IoT 中心查询语言][devguide-query]介绍了可用于在中心检索设备孪生和作业相关信息的 IoT 中心查询语言。
* [参考 - 配额和限制][devguide-quotas]总结了 IoT 中心服务中设置的配额，以及当超过配额时可以预期看到的限制行为。
* [参考 - 设备和服务 SDK][devguide-sdks] 列出了开发与 IoT 中心交互的设备和服务应用时可使用的 Azure IoT SDK。此文章包括指向联机 API 文档的链接。
* [参考 - IoT 中心 MQTT 支持][devguide-mqtt]详细介绍了 IoT 中心如何支持 MQTT 协议。此文章介绍了到 Azure IoT SDK 的 MQTT 协议内置支持，并阐述了如何直接使用 MQTT 协议。
* [词汇表][devguide-glossary]是与 IoT 中心相关的常见术语的列表。



[devguide-messaging]: /documentation/articles/iot-hub-devguide-messaging/
[devguide-upload]: /documentation/articles/iot-hub-devguide-file-upload/
[devguide-identities]: /documentation/articles/iot-hub-devguide-identity-registry/
[devguide-security]: /documentation/articles/iot-hub-devguide-security/
[devguide-device-twins]: /documentation/articles/iot-hub-devguide-device-twins/
[devguide-directmethods]: /documentation/articles/iot-hub-devguide-direct-methods/
[devguide-jobs]: /documentation/articles/iot-hub-devguide-jobs/
[devguide-endpoints]: /documentation/articles/iot-hub-devguide-endpoints/
[devguide-quotas]: /documentation/articles/iot-hub-devguide-quotas-throttling/
[devguide-query]: /documentation/articles/iot-hub-devguide-query-language/
[devguide-sdks]: /documentation/articles/iot-hub-devguide-sdks/
[devguide-mqtt]: /documentation/articles/iot-hub-mqtt-support/
[devguide-glossary]: /documentation/articles/iot-hub-devguide-glossary/
[lnk-c2d-guidance]: /documentation/articles/iot-hub-devguide-c2d-guidance/
[lnk-d2c-guidance]: /documentation/articles/iot-hub-devguide-d2c-guidance/

<!---HONumber=Mooncake_0306_2017-->
<!--Update_Description:update wording-->