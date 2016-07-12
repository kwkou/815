<properties
 pageTitle="IoT 中心诊断度量值"
 description="概述 Azure IoT 中心度量值，使用户能够评估其资源的总体运行状况"
 services="iot-hub"
 documentationCenter=""
 authors="nberdy"
 manager="timlt"
 editor=""/>

<tags
 ms.service="iot-hub"
 ms.date="02/03/2016"
 wacn.date="07/04/2016"/>

# 诊断度量值简介

诊断度量值是 Azure 范围的度量值推送内容的一部分，目的是提供有关订阅中 Azure 资源状态的更清晰数据。用户可以使用度量值评估服务以及连接到服务的设备的总体运行状况。面向用户的统计信息非常重要，因为它们可以帮助用户了解其 IoT 中心的情况，帮助用户在不联系 Azure 支持人员的情况下解决根本问题。

可以从 Azure 门户启用诊断度量值。

## 如何启用诊断度量值

1. 创建 IoT 中心。可以在[入门][lnk-get-started]指南中找到有关如何创建 IoT 中心的说明。

2. 打开 IoT 中心的边栏选项卡。在此处单击“所有设置”，然后单击“诊断”。

    ![][1]

3. 通过将状态设置为“打开”并选择要用于存储诊断数据的存储帐户来配置诊断。选中度量值复选框，然后按“保存”。请注意，必须提前创建存储帐户，并且你需要单独为存储付费。

    ![][2]

4. 设置诊断后，返回到 IoT 中心的主边栏选项卡。边栏选项卡的“监视”部分中填充了度量值信息。单击图表打开度量值窗格，可以在此处查看 IoT 中心的度量值信息摘要，并编辑图表中显示的所选度量值。你还可以根据度量值配置警报。

    ![][3]

## 度量值及其用法

IoT 中心提供多个度量值，使你大致了解中心的运行状况及连接到中心的设备总数。可以结合多个度量值的信息，更清楚地了解 IoT 中心的状态。下表描述了每个 IoT 中心所跟踪的度量值，以及每个度量值与 IoT 中心总体状态的关联。

| 度量值 | 度量值说明 | 度量值用途 |
| ---- | ---- | ---- |
| d2c.telemetry.ingress.allProtocol | 所有设备上发送的消息数目 | 有关消息发送操作的概述数据 |
| d2c.telemetry.ingress.success | 成功传入中心的消息总数 | 成功传入中心的消息的概述 |
| c2d.commands.egress.complete.success | 接收设备在所有设备上完成的所有命令消息计数 | 结合有关放弃或拒绝的度量值提供 C2D 命令总体成功率的概述 |
| c2d.commands.egress.abandon.success | 接收设备在所有设备上成功放弃的消息总数 | 如果消息被放弃的频率超出预期，则突显潜在问题 |
| c2d.commands.egress.reject.success | 接收设备在所有设备上成功拒绝的消息总数 | 如果消息被拒绝的频率超出预期，则突显潜在问题 |
| devices.totalDevices | 向 IoT 中心注册的设备的平均数目、最小数目和最大数目 | 向中心注册的设备数目 |
| devices.connectedDevices.allProtocol | 同时连接的设备的平均数目、最小数目和最大数目 | 连接到中心的设备数概述 |

## 后续步骤

现在你已了解诊断度量值的概述，接下来请单击以下链接来了解更多信息：

- [IoT 中心操作监视][lnk-operations-monitoring]
- [缩放 IoT 中心][lnk-scaling]
- [IoT 中心高可用性和灾难恢复][lnk-dr]

<!-- Links and images -->
[1]: ./media/iot-hub-metrics/enable-metrics-1.png
[2]: ./media/iot-hub-metrics/enable-metrics-2.png
[3]: ./media/iot-hub-metrics/enable-metrics-3.png

[lnk-get-started]: /documentation/articles/iot-hub-csharp-csharp-getstarted/
[lnk-operations-monitoring]: /documentation/articles/iot-hub-operations-monitoring/
[lnk-scaling]: /documentation/articles/iot-hub-scaling/
[lnk-dr]: /documentation/articles/iot-hub-ha-dr/

<!---HONumber=Mooncake_0307_2016-->