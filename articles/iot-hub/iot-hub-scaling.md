<properties
 pageTitle="Azure IoT 中心缩放 | Azure"
 description="介绍如何缩放 Azure IoT 中心。"
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
 ms.date="09/19/2016"
 ms.author="elioda"
 wacn.date="11/07/2016"/>  


# 缩放 IoT 中心

Azure IoT 中心可支持多达一百万台设备同时连接。有关详细信息，请参阅 [IoT 中心定价][lnk-pricing]。每个 IoT 中心计价单位允许特定数量的日常消息。

为了正确缩放解决方案，必须考虑 IoT 中心的特定用法。尤其要考虑以下类别的操作所需的高峰吞吐量：

* 设备到云的消息
* 云到设备的消息
* 标识注册表操作

除此吞吐量信息以外，请参阅 [IoT 中心配额和限制][]来相应地设计你的解决方案。

## 设备到云和云到设备的消息吞吐量

若要基于每个单位评估流量，最佳方式是调整 IoT 中心解决方案的大小。

设备到云的消息遵循以下持续吞吐量指导原则。

| 层 | 持续吞吐量 | 持续发送速率 |
| ---- | -------------------- | ------------------- |
| S1 | 每个计价单位最多 1111 KB/分钟<br/>（1.5 GB/天/计价单位） | 每个计价单位平均 278 条消息/分钟<br/>（400,000 条消息/天/计价单位） |
| S2 | 每个计价单位最多 16 MB/分钟<br/>（22.8 GB/天/计价单位） | 每个计价单位平均 4167 条消息/分钟<br/>（600 万条消息/天/计价单位） |
| S3 | 每个计价单位最多 814 MB/分钟<br/>（1144.4 GB/天/计价单位） | 每个计价单位平均 208,333 条消息/分钟<br/>（3 亿条消息/天/计价单位） |

## 标识注册表操作吞吐量

由于大多数 IoT 中心标识注册表操作都与设备预配相关，因此不认为这些操作是运行时操作。

有关具体的突发性能数字，请参阅 [IoT 中心配额和限制][]。

## 分片

尽管单个 IoT 中心可以扩展到数百万个设备，但有时解决方案所需的具体性能特征无法由单个 IoT 中心提供保证。在此情况下，建议将设备分区成多个 IoT 中心。多个 IoT 中心可以缓解流量喷发，并获得所需的吞吐量或操作速率。

## 后续步骤

若要进一步探索 IoT 中心的功能，请参阅：

- [开发人员指南][lnk-devguide]
- [使用网关 SDK 模拟设备][lnk-gateway]

[lnk-pricing]: /pricing/details/iot-hub/
[IoT 中心配额和限制]: /documentation/articles/iot-hub-devguide-quotas-throttling/

[lnk-devguide]: /documentation/articles/iot-hub-devguide/
[lnk-gateway]: /documentation/articles/iot-hub-linux-gateway-sdk-simulated-device/

<!---HONumber=Mooncake_0307_2016-->