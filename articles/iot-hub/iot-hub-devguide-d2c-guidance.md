<properties
	pageTitle="Azure IoT 中心从设备到云通信指南"
	description="Azure IoT 中心开发人员指南 - 指导用户何时使用从设备到云的消息、设备克隆的报告属性或文件上载。"
	services="iot-hub"
	documentationcenter=".net"
	author="fsautomata"
	manager="timlt"
	editor=""/>  


<tags
	ms.service="iot-hub"
	ms.date="09/30/2016"
	wacn.date="12/19/2016"/>  

# 从设备到云通信指南
将信息从设备应用发送到后端时，IoT 中心会公开三个选项：

* [从设备到云 (D2C) 的消息][lnk-d2c]：适用于时间系列遥测和警报；
* [克隆的报告属性][lnk-twins]：适用于报告设备状态信息，例如可用功能、条件以及长时间运行的工作流（例如配置和软件更新）的状态；
* [文件上载][lnk-fileupload]：适用于媒体文件和大型遥测批文件，可在上载时通过间歇性地连接到设备或者压缩节省带宽。

下面详细比较了各种从设备到云的通信选项。

| | D2C 消息 | 克隆的报告属性 | 文件上载 |
| ---- | ------- | ---------- | ---- |
| 方案 | 遥测时间系列和警报，例如 256KB 的传感器数据批处理，每隔 5 分钟发送一次。 | 可用的功能和条件，例如设备当前的连接模式（移动电话网络或 Wifi）。同步长时间运行的工作流，例如配置和软件更新。 | 媒体文件。大型（通常为压缩状态）遥测批处理。 |
| 存储和检索 | 通过 IoT 中心临时进行存储，最多存储 7 天。仅顺序读取。 | 通过 IoT 中心存储在设备克隆中。可通过 [SQL 类查询语言][lnk-query]检索。 | 存储在用户提供的 Azure 存储帐户中。 |
| 大小 | 最多 256KB 的消息。 | 报告的属性大小最大为 8KB。 | Azure Blob 存储支持的最大文件大小。 |
| 频率 | 高。有关详细信息，请参阅 [IoT 中心限制][lnk-quotas]。 | 中。有关详细信息，请参阅 [IoT 中心限制][lnk-quotas]。 | 低。有关详细信息，请参阅 [IoT 中心限制][lnk-quotas]。 |
| 协议 | 在所有协议上可用。 | 目前仅在使用 MQTT 时提供。 | 不管使用什么协议都提供，但要求在设备上启用 HTTP。 |

> [AZURE.NOTE]
可能出现的情况是，应用程序既要求将信息以遥测时间系列或警报方式发送，又要求以克隆方式提供。不管什么情况，既可以通过设备应用发送 D2C 消息并报告属性更改，也可以通过后端在收到消息时将信息存储在克隆的标记中。由于 D2C 消息允许的吞吐量远高于克隆更新，因此在有些时候，最好是避免为每条 D2C 消息更新克隆。
> 
> 

[lnk-twins]: /documentation/articles/iot-hub-devguide-device-twins/
[lnk-fileupload]: /documentation/articles/iot-hub-devguide-file-upload/
[lnk-quotas]: /documentation/articles/iot-hub-devguide-quotas-throttling/
[lnk-query]: /documentation/articles/iot-hub-devguide-query-language/
[lnk-d2c]: /documentation/articles/iot-hub-devguide-messaging/#device-to-cloud-messages

<!---HONumber=Mooncake_1212_2016-->