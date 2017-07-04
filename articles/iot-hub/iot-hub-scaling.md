<properties
    pageTitle="Azure IoT 中心缩放 | Azure"
    description="如何缩放 IoT 中心来支持预期的消息吞吐量。 概括介绍了分片选项和每层支持的吞吐量"
    services="iot-hub"
    documentationCenter=""
    authors="fsautomata"
    manager="timlt"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.service="iot-hub"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="09/19/2016"
    wacn.date="04/24/2017"
    ms.author="elioda"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="7c8281cb060e06d612623fc95d6e061cca8d6abb"
    ms.lasthandoff="04/14/2017" />

# <a name="scale-your-iot-hub-solution"></a>缩放 IoT 中心解决方案
Azure IoT 中心可支持多达一百万台设备同时连接。 有关详细信息，请参阅 [IoT 中心定价][lnk-pricing]。 每个 IoT 中心计价单位允许特定数量的日常消息。

为了正确缩放解决方案，必须考虑 IoT 中心的特定用法。尤其要考虑以下类别的操作所需的高峰吞吐量：

* 设备到云的消息
* 云到设备的消息
* 标识注册表操作

除了此吞吐量信息，另请参阅 [IoT 中心配额和限制][IoT 中心配额和限制]并相应地设计解决方案。

## <a name="device-to-cloud-and-cloud-to-device-message-throughput"></a>设备到云和云到设备的消息吞吐量
若要基于每个单位评估流量，最佳方式是调整 IoT 中心解决方案的大小。

设备到云的消息遵循以下持续吞吐量指导原则。

| 层 | 持续吞吐量 | 持续发送速率 |
| ---- | -------------------- | ------------------- |
| S1 | 每个单元最多 1111 KB/分钟<br/>（1.5 GB/天/单元） | 每个单元平均 278 条消息/分钟<br/>（400000 条消息/天/单元） |
| S2 | 每个单元最多 16 MB/分钟<br/>（22.8 GB/天/单元） | 每个单元平均 4167 条消息/分钟<br/>（600 万条消息/天/单元） |
| S3 | 每个单元最多 814 MB/分钟<br/>（1144.4 GB/天/单元） | 每个单元平均 208,333 条消息/分钟<br/>（3 亿条消息/天/单元） |

## <a name="identity-registry-operation-throughput"></a>标识注册表操作吞吐量

由于大多数 IoT 中心标识注册表操作都与设备预配相关，因此不认为这些操作是运行时操作。

有关具体的突发性能数字，请参阅 [IoT 中心配额和限制][]。

## <a name="sharding"></a>分片
尽管单个 IoT 中心可以扩展到数百万个设备，但有时解决方案所需的具体性能特征无法由单个 IoT 中心提供保证。 在此情况下，建议将设备分区成多个 IoT 中心。 多个 IoT 中心可以缓解流量喷发，并获得所需的吞吐量或操作速率。

## <a name="next-steps"></a>后续步骤
若要进一步探索 IoT 中心的功能，请参阅：

- [IoT 中心开发人员指南][lnk-devguide]
- [使用 IoT 网关 SDK 模拟设备][lnk-gateway]

[lnk-pricing]: /pricing/details/iot-hub/
[IoT 中心配额和限制]: /documentation/articles/iot-hub-devguide-quotas-throttling/

[lnk-devguide]: /documentation/articles/iot-hub-devguide/
[lnk-gateway]: /documentation/articles/iot-hub-linux-gateway-sdk-simulated-device/

<!--Update_Description:update wording-->