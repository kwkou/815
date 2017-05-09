<properties
    pageTitle="了解 Azure IoT 中心配额和限制 | Azure"
    description="开发人员指南 - 介绍适用于 IoT 中心的配额和预期限制行为。"
    services="iot-hub"
    documentationcenter=".net"
    author="dominicbetts"
    manager="timlt"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid="425e1b08-8789-4377-85f7-c13131fae4ce"
    ms.service="iot-hub"
    ms.devlang="multiple"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="03/27/2017"
    wacn.date="05/08/2017"
    ms.author="dobett"
    ms.sourcegitcommit="2c4ee90387d280f15b2f2ed656f7d4862ad80901"
    ms.openlocfilehash="ad1f72de5cd0ffdd9e4b8d593f76a346a8a208ac"
    ms.lasthandoff="04/28/2017" />

# <a name="reference---iot-hub-quotas-and-throttling"></a>参考 - IoT 中心配额和限制

## <a name="quotas-and-throttling"></a>配额和限制
每个 Azure 订阅最多可以拥有 10 个 IoT 中心和 1 个免费中心。

每个 IoT 中心预配了特定 SKU 的特定单位数（有关详细信息，请参阅 [Azure IoT 中心定价][lnk-pricing]）。 SKU 和单位数目确定可以发送的消息的每日配额上限。

SKU 还确定了 IoT 中心对所有操作强制实施的限制。

## <a name="operation-throttles"></a>操作限制
操作限制是在分钟范围内应用的速率限制，主要是为了避免不当使用。 IoT 中心会尽可能避免返回错误，但如果违反限制太久，就会开始返回异常。

以下是强制实施的限制列表。 值与单个中心相关。

| 限制 | 免费和 S1 中心 | S2 中心 | S3 中心 | 
| -------- | ------- | ------- | ------- |
| 标识注册表操作（创建、检索、列出、更新、删除） | 1.67/秒/单位（100/分钟/单位） | 1.67/秒/单位（100/分钟/单位） | 83.33/秒/单位（5000/分钟/单位） |
| 设备连接 | 最大值为 100/秒或 12/秒/单位 <br/> 例如，两个 S1 单位是 2\*12 = 24/秒，但是在所有单位中至少有 100/秒。 如果有 9 个 S1 单位，则所有单位就有 108/秒 (9\*12)。 | 120/秒/单位 | 6000/秒/单位 |
| 设备到云的发送 | 最大值为 100/秒或 12/秒/单位 <br/> 例如，两个 S1 单位是 2\*12 = 24/秒，但是在所有单位中至少有 100/秒。 如果有 9 个 S1 单位，则所有单位就有 108/秒 (9\*12)。 | 120/秒/单位 | 6000/秒/单位 |
| 云到设备的发送 | 1.67/秒/单位（100/分钟/单位） | 1.67/秒/单位（100/分钟/单位） | 83.33/秒/单位（5000/分钟/单位） |
| 云到设备的接收 <br/> （仅当设备使用 HTTP 时）| 16.67/秒/单位（1000/分钟/单位） | 16.67/秒/单位（1000/分钟/单位） | 833.33/秒/单位（50000/分钟/单位） |
| 文件上载 | 1.67 文件上载通知/秒/单位（100/分钟/单位） | 1.67 文件上载通知/秒/单位（100/分钟/单位） | 83.33 文件上载通知/秒/单位（5000/分钟/单位） |
| 直接方法 | 10/秒/单位 | 30/秒/单位 | 1500/秒/单位 | 
| 设备孪生读取 | 10/秒 | 最大值为 10/秒或 1/秒/单位 | 50/秒/单位 |
| 设备孪生更新 | 10/秒 | 最大值为 10/秒或 1/秒/单位 | 50/秒/单位 |
| 作业操作 <br/> （创建、更新、列表、删除） | 1.67/秒/单位（100/分钟/单位） | 1.67/秒/单位（100/分钟/单位） | 83.33/秒/单位（5000/分钟/单位） |
| 作业每设备操作吞吐量 | 10/秒 | 最大值为 10/秒或 1/秒/单位 | 50/秒/单位 |

必须清楚地知道，*设备连接*限制控制可与 IoT 中心建立新设备连接的速率，而不是控制同时连接的设备数上限。该限制取决于为 IoT 中心预配的单位数。

例如，如果你购买的是单一 S1 单位，则限制为每秒 100 个连接。因此，若要连接 100,000 台设备，至少需要花费 1000 秒（大约 16 分钟）。但是，同时连接的设备数可与用户在标识注册表中注册的设备数相同。


>[AZURE.NOTE]
> 无论何时，都可以通过增加 IoT 中心预配的单位来提高配额或限制。
> 

>[AZURE.IMPORTANT]
> 标识注册表操作用于设备管理与预配方案中的运行时使用。 通过 [导入和导出作业][lnk-importexport]可以支持读取或更新大量的设备标识。
> 
> 

## <a name="other-limits"></a>其他限制

IoT 中心对其不同的功能实施其他限制。

| 操作 | 限制 |
| --------- | ----- |
| 文件上传 URI | 存储帐户一次可传出 10000 个 SAS URI。 <br/> 每台设备一次可传出 10 个 SAS URI。 |
| 作业 | 作业历史记录最多保留 30 天 <br/> 最大并发作业数为 1（适用于免费版和 S1）、5（适用于 S2）、10（适用于 S3）。 |
| 额外终结点 | 付费 SKU 中心可能有 10 个额外终结点。 免费 SKU 中心可能有 1 个额外终结点。 |
| 消息路由规则 | 付费 SKU 中心可能有 100 条路由规则。 免费 SKU 中心可包含 5 条路由规则。 |

> [AZURE.NOTE]
> 目前，可以连接到单个 IoT 中心的设备的最大数目是 500,000。 如果想要增加此限制，请联系 [Azure 支持](/support/contact/)。

## <a name="next-steps"></a>后续步骤

此 IoT 中心开发人员指南中的其他参考主题包括：

- [IoT 中心终结点][lnk-devguide-endpoints]
- [设备孪生和作业的 IoT 中心查询语言][lnk-devguide-query]
- [IoT 中心 MQTT 支持][lnk-devguide-mqtt]

[lnk-pricing]: /pricing/details/iot-hub
[lnk-throttle-blog]: https://azure.microsoft.com/blog/iot-hub-throttling-and-you/
[lnk-importexport]: /documentation/articles/iot-hub-devguide-identity-registry/#import-and-export-device-identities

[lnk-devguide-endpoints]: /documentation/articles/iot-hub-devguide-endpoints/
[lnk-devguide-query]: /documentation/articles/iot-hub-devguide-query-language/
[lnk-devguide-mqtt]: /documentation/articles/iot-hub-mqtt-support/

<!--Update_Description:update wording-->