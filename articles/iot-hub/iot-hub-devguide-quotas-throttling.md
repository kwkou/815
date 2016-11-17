<properties
 pageTitle="开发人员指南 - 配额和限制 | Azure"
 description="Azure IoT 中心开发人员指南 - 适用于 IoT 中心的配额和预期的限制行为的说明"
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
 wacn.date="11/07/2016" 
 ms.author="dobett"/>  


# 参考 - 配额和限制

## 配额和限制

每个 Azure 订阅最多可以有 10 个 IoT 中心。

每个 IoT 中心预配了特定 SKU 的特定单位数（有关详细信息，请参阅 [Azure IoT 中心定价][lnk-pricing]）。SKU 和单位数目确定可以发送的消息的每日配额上限。

SKU 还确定了 IoT 中心对所有操作强制实施的限制。

## 操作限制

操作限制是在分钟范围内应用的速率限制，主要是为了避免不当使用。IoT 中心会尽可能避免返回错误，但如果违反限制太久，就会开始返回异常。

以下是强制实施的限制列表。值与单个中心相关。

| 限制 | 每个中心的值 |
| -------- | ------------- |
| 标识注册表操作（创建、检索、列出、更新、删除） | 5000/分钟/单位（适用于 S3）<br/> 100/分钟/单位（适用于 S1 和 S2）。 |
| 设备连接 | 6000/秒/单位（适用于 S3），120/秒/单位（适用于 S2），12/秒/单位（适用于 S1）。<br/>最小值为 100/秒。<br/>例如，两个 S1 单位是 2\*12 = 24/秒，但是在所有单位中至少有 100/秒。如果有 9 个 S1 单位，则所有单位就有 108/秒 (9\*12)。 |
| 设备到云的发送 | 6000/秒/单位（适用于 S3），120/秒/单位（适用于 S2），12/秒/单位（适用于 S1）。<br/>最小值为 100/秒。<br/>例如，两个 S1 单位是 2\*12 = 24/秒，但是在所有单位中至少有 100/秒。如果有 9 个 S1 单位，则所有单位就有 108/秒 (9\*12)。 |
| 云到设备的发送 | 5000/分钟/单位（适用于 S3），100/分钟/单位（适用于 S1 和 S2）。 |
| 云到设备的接收 | 50000/分钟/单位（适用于 S3），1000/分钟/单位（适用于 S1 和 S2）。 |
| 文件上载操作 | 5000 个文件上载通知/分钟/单位（适用于 S3），100 个文件上载通知/分钟/单位（适用于 S1 和 S2）。<br/> 存储帐户一次可传出 10000 个 SAS URI。<br/> 一次可传出 10 个 SAS URI/设备。 | 

必须清楚地知道， *设备连接* 限制控制可与 IoT 中心建立新设备连接的速率，而不是控制同时连接的设备数上限。该限制取决于为中心预配的单位数。

例如，如果你购买的是单一 S1 单位，则限制为每秒 100 个连接。这意味着，若要连接 100,000 台设备，至少需要花费 1000 秒（大约 16 分钟）。但是，同时连接的设备数可与你在设备标识注册表中注册的设备数相同。


>[AZURE.NOTE] 无论何时，都可以通过增加 IoT 中心预配的单位来提高配额或限制。

>[AZURE.IMPORTANT] 标识注册表操作用于设备管理与预配方案中的运行时使用。通过[导入和导出作业][lnk-importexport]可以支持读取或更新大量的设备标识。

## 后续步骤

此 IoT 中心开发人员指南中的其他参考主题包括：

- [IoT 中心终结点][lnk-devguide-endpoints]
- [克隆、方法和作业的查询语言][lnk-devguide-query]
- [IoT 中心 MQTT 支持][lnk-devguide-mqtt]

[lnk-pricing]: /pricing/details/iot-hub
[lnk-throttle-blog]: https://azure.microsoft.com/blog/iot-hub-throttling-and-you/
[lnk-importexport]: /documentation/articles/iot-hub-devguide-identity-registry/#import-and-export-device-identities

[lnk-devguide-endpoints]: /documentation/articles/iot-hub-devguide-endpoints/
[lnk-devguide-query]: /documentation/articles/iot-hub-devguide-query-language/
[lnk-devguide-mqtt]: /documentation/articles/iot-hub-mqtt-support/

<!---HONumber=Mooncake_1031_2016-->