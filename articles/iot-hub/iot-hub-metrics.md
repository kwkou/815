<properties
    pageTitle="Azure IoT 中心度量值 | Azure"
    description="如何使用 Azure IoT 中心度量值评估和监视 IoT 中心的总体运行状况。"
    services="iot-hub"
    documentationcenter=""
    author="nberdy"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="a47108fd-f994-4105-b21d-5b8f697b699c"
    ms.service="iot-hub"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="11/16/2016"
    wacn.date="01/13/2017"
    ms.author="nberdy" />  


# IoT 中心度量值
IoT 中心度量值提供更棒的数据，清晰显示 Azure 订阅中的 Azure IoT 资源状态。通过 IoT 中心度量值，可评估 IoT 中心服务及其所连接的设备的总体运行状况。面向用户的统计信息非常重要，因为它们可以帮助了解 IoT 中心的情况，并可以帮助在不联系 Azure 支持人员的情况下解决根本问题。

默认启用度量值。可在 Azure 门户预览中查看 IoT 中心度量值。

## 如何查看 IoT 中心度量值
1. 创建 IoT 中心。有关如何创建 IoT 中心的说明，请参阅[入门][lnk-get-started]指南。
2. 打开 IoT 中心的边栏选项卡。在此处单击“度量值”。
   
    ![][1]  

3. 在“度量值”边栏选项卡中，可查看 IoT 中心的度量值并创建度量值的自定义视图。单击“诊断设置”，即可选择将度量值数据发送到事件中心终结点或 Azure 存储帐户。
   
    ![][2]  


## IoT 中心度量值及其用法
IoT 中心提供多个度量值，帮助你大致了解中心的运行状况以及所连接的设备总数。可以结合多个度量值的信息，更清楚地了解 IoT 中心的状态。下表描述了每个 IoT 中心所跟踪的度量值，以及每个度量值与 IoT 中心总体状态的关联。

| 度量值 | 度量值说明 | 度量值用途 |
| --- | --- | --- |
| d2c.telemetry.ingress.allProtocol | 所有设备上发送的消息数目 | 简要介绍有关消息发送操作的数据 |
| d2c.telemetry.ingress.success | 成功传入中心的消息总数 | 成功传入中心的消息的概述 |
| d2c.telemetry.egress.success | 所有成功写入终结点的遥测消息数 | 简要介绍基于用户路由的消息扇出操作 |
| d2c.telemetry.egress.invalid | 由于与终结点不兼容而未传递的消息计数 | 简要介绍要写入用户的终结点集的故障数。高值可能表示终结点配置错误。 |
| d2c.telemetry.egress.dropped | 由于终结点不正常而删除的消息数 | 简要介绍 IoT 中心的当前配置下所丢弃的消息数 |
| d2c.telemetry.egress.fallback | 符合回退路由的消息计数 | 对于通过管道将所有消息传递到其他终结点（非内置终结点）的用户，此度量值将显示路由设置中的差距 |
| d2c.telemetry.egress.orphaned | 不匹配任何路由（包括回退路由）的消息计数 | 简要介绍 IoT 中心的当前配置下所孤立的消息数 |
| d2c.endpoints.latency.eventHubs | 消息进入 IoT 中心与进入事件中心终结点之间的平均延迟（毫秒） | 此传递可帮助用户识别不佳的终结点配置 |
| d2c.endpoints.latency.serviceBusQueues | 消息进入 IoT 中心与进入服务总线队列终结点之间的平均延迟（毫秒） | 此传递可帮助用户识别不佳的终结点配置 |
| d2c.endpoints.latency.serviceBusTopic | 消息进入 IoT 中心与进入服务总线主题终结点之间的平均延迟（毫秒） | 此传递可帮助用户识别不佳的终结点配置 |
| d2c.endpoints.latency.builtIn.events | 消息进入 IoT 中心与进入内置终结点（消息/事件）之间的平均延迟（毫秒） | 此传递可帮助用户识别不佳的终结点配置 |
| c2d.commands.egress.complete.success | 接收设备在所有设备上完成的所有命令消息计数 |结合有关放弃或拒绝的度量值，可提供云到设备命令总体成功率的概述 |
| c2d.commands.egress.abandon.success | 接收设备在所有设备上成功放弃的消息总数 |如果消息被放弃的频率超出预期，则突显潜在问题 |
| c2d.commands.egress.reject.success | 接收设备在所有设备上成功拒绝的消息总数 |如果消息被拒绝的频率超出预期，则突显潜在问题 |
| devices.totalDevices | 向 IoT 中心注册的设备计数 |向中心注册的设备数目 |
| devices.connectedDevices.allProtocol | 同时连接的设备计数 |连接到中心的设备数概述 |

## 后续步骤
现已大致了解了 IoT 中心度量值，请单击此链接，深入了解如何管理 Azure IoT 中心：

- [操作监视][lnk-monitor]

若要进一步探索 IoT 中心的功能，请参阅：

- [IoT 中心开发人员指南][lnk-devguide]
- [使用 IoT 网关 SDK 模拟设备][lnk-gateway]

<!-- Links and images -->

[1]: ./media/iot-hub-metrics/enable-metrics-1.png
[2]: ./media/iot-hub-metrics/enable-metrics-2.png

[lnk-get-started]: /documentation/articles/iot-hub-csharp-csharp-getstarted/
[lnk-operations-monitoring]: /documentation/articles/iot-hub-operations-monitoring/
[lnk-scaling]: /documentation/articles/iot-hub-scaling/
[lnk-dr]: /documentation/articles/iot-hub-ha-dr/

[lnk-monitor]: /documentation/articles/iot-hub-operations-monitoring/

[lnk-devguide]: /documentation/articles/iot-hub-devguide/
[lnk-gateway]: /documentation/articles/iot-hub-linux-gateway-sdk-simulated-device/

<!---HONumber=Mooncake_0109_2017-->
<!--Update_Description:update wording-->