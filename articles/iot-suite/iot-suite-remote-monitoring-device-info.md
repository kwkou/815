<properties
    pageTitle="远程监控解决方案中的设备信息元数据 | Azure"
    description="介绍 Azure IoT 预配置解决方案远程监控及其体系结构。"
    services=""
    suite="iot-suite"
    documentationCenter=""
    authors="dominicbetts"
    manager="timlt"
    editor="" />
<tags
    ms.service="iot-suite"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="05/15/2017"
    ms.author="v-yiso"
    wacn.date="06/13/2017"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="4a18b6116e37e365e2d4c4e2d144d7588310292e"
    ms.openlocfilehash="ef150381c700bffc8aded8edf69816fe77b8710d"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/19/2017" />

# <a name="device-information-metadata-in-the-remote-monitoring-preconfigured-solution"></a>远程监控预配置解决方案中的设备信息元数据
Azure IoT 套件远程监控预配置解决方案演示了管理设备元数据的方法。 本文将概述此解决方案为了帮助你理解内容而采用的方法：

- 解决方案存储哪些设备元数据。
- 解决方案如何管理设备元数据。

## <a name="context"></a>上下文
远程监控预配置解决方案使用 [Azure IoT 中心][lnk-iot-hub] ，使设备能够将数据发送到云。 该解决方案在三个不同的位置存储有关设备的信息：

| 位置 | 存储的信息 | 实现 |
| -------- | ------------------ | -------------- |
| 标识注册表 | 设备 ID、身份验证密钥、启用状态 | 内置于 IoT 中心 |
| 设备孪生 | 元数据：报告的属性、所需的属性、标记 | 内置于 IoT 中心 |
| DocumentDB | 命令和方法历史记录 | 为解决方案自定义 |

IoT 中心提供[设备标识注册表][lnk-identity-registry]用于管理对 IoT 中心的访问，使用[设备孪生][lnk-device-twin]来管理设备元数据。此外，还提供一个远程监控解决方案特定的*设备注册表*用于存储命令和方法历史记录。远程监控解决方案使用 [DocumentDB][lnk-docdb] 数据库实现命令和方法历史记录的自定义存储。

> [AZURE.NOTE]
> 远程监控预配置解决方案可使设备标识注册表与 DocumentDB 数据库中的信息保持同步。 这两个注册表使用相同的设备 ID 来唯一标识连接到 IoT 中心的每个设备。
> 
> 

## <a name="device-metadata"></a>设备元数据
IoT 中心为连接到远程监控解决方案的每个模拟设备和物理设备维护[设备孪生][lnk-device-twin]。 解决方案使用设备孪生来管理与设备关联的元数据。 设备孪生是 IoT 中心维护的 JSON 文档。解决方案使用 IoT 中心 API 来与设备孪生交互。

设备孪生存储三种类型的元数据：

- *报告的属性*由设备发送到 IoT 中心。在远程监控解决方案中，模拟设备在启动时以及在响应“更改设备状态”命令和方法时，将发送报告的属性。可以在解决方案门户上的“设备列表”和“设备详细信息”中查看报告的属性。报告的属性是只读的。
- *所需的属性*由设备从 IoT 中心检索。设备负责对自身进行任何必要的配置更改。此外，设备还需负责以报告属性的形式向中心报告更改。可以通过解决方案门户设置所需的属性值。
- *标记*仅在设备孪生中存在，永远不会与设备同步。可以在解决方案门户设置标记值，并在筛选设备列表时使用这些值。解决方案还使用标记在解决方案门户中标识设备的显示图标。

来自模拟设备的报告属性示例包括制造商、型号、纬度和经度。模拟设备也以报告属性的形式返回受支持方法的列表。

> [AZURE.NOTE]
> 模拟设备代码仅使用所需的属性 Desired.Config.TemperatureMeanValue 和 Desired.Config.TelemetryInterval 来更新发回给 IoT 中心的报告的属性。 将忽略所有其他的所需属性更改请求。

存储在设备注册表 DocumentDB 数据库中的设备信息元数据 JSON 文档具有以下结构：


		{
		  "DeviceProperties": {
		    "DeviceID": "deviceid1",
		    "HubEnabledState": null,
		    "CreatedTime": "2016-04-25T23:54:01.313802Z",
		    "DeviceState": "normal",
		    "UpdatedTime": null
		    },
		  "SystemProperties": {
		    "ICCID": null
		  },
		  "Commands": [],
		  "CommandHistory": [],
		  "IsSimulatedDevice": false,
		  "id": "fe81a81c-bcbc-4970-81f4-7f12f2d8bda8"
		}



> [AZURE.NOTE]
> 设备信息还可能包含元数据，用于描述设备发送给 IoT 中心的遥测数据。远程监控解决方案使用此遥测元数据来自定义仪表板显示[动态遥测数据][lnk-dynamic-telemetry]的方式。
> 
> 

## <a name="lifecycle"></a>生命周期

首次在解决方案门户中创建设备时，解决方案将在 DocumentDB 数据库中创建一个项来存储命令和方法历史记录。 此时，解决方案还会在设备标识注册表中为设备创建一个条目，用于生成设备向 IoT 中心进行身份验证时所用的密钥。 它还会创建设备孪生。

当设备首次连接到解决方案时，将发送报告的属性和设备信息消息。报告的属性值自动保存在设备孪生中。报告的属性包括设备制造商、型号、序列号和受支持方法的列表。设备信息消息包含设备支持的命令列表，其中包括有关任何命令参数的信息。当解决方案收到此消息时，将更新 DocumentDB 数据库中的设备信息。

### <a name="view-and-edit-device-information-in-the-solution-portal"></a>在解决方案门户中查看和编辑设备信息
默认情况下，解决方案门户中的设备列表以列的形式显示以下设备属性：“状态”、“设备 ID”、“制造商”、“型号”、“序号”、“固件”、“平台”、“处理器”和“已安装的 RAM”。 可以通过单击“列编辑器”来自定义这些列。 设备属性“纬度”和“经度”驱动仪表板上必应地图中的位置。

![设备列表中的列编辑器][img-device-list]  

在解决方案门户上的“设备详细信息”窗格中，可以编辑所需的属性和标记（报告的属性是只读的）。

![设备详细信息窗格][img-device-edit]  


可以使用解决方案门户从解决方案中删除设备。删除某个设备时，解决方案将从标识注册表中删除该设备项，然后删除设备孪生。解决方案还会从 DocumentDB 数据库中删除与该设备相关的信息。必须禁用设备才可删除它。

![删除设备][img-device-remove]

## <a name="device-information-message-processing"></a>设备信息消息处理

设备发出的设备信息消息不同于遥测消息。 设备信息消息包括设备可响应的命令和任何命令历史记录。 IoT 中心本身不知道设备信息消息中包含的元数据，它以处理任何设备到云消息的相同方式处理消息。 在远程监控解决方案中，[Azure 流分析][lnk-stream-analytics] (ASA) 作业读取来自 IoT 中心的消息。 DeviceInfo 流分析作业筛选包含 "ObjectType": "DeviceInfo" 的消息，并将这些消息转发到 Web 作业中运行的 EventProcessorHost 主机实例。 EventProcessorHost 实例中的逻辑使用设备 ID 来查找特定设备的 DocumentDB 记录并更新记录。

> [AZURE.NOTE]
> 设备信息消息是标准的设备到云消息。解决方案使用 ASA 查询来区分设备信息消息与遥测消息。
> 
> 

## <a name="next-steps"></a>后续步骤
现在你已学习如何自定义预配置解决方案，接下来你可以浏览 IoT 套件预配置的解决方案的一些其他特性和功能：

- [预测性维护预配置解决方案概述][lnk-predictive-overview]
- [有关 IoT 套件的常见问题][lnk-faq]
- [从头开始保障 IoT 安全][lnk-security-groundup]



<!-- Images and links -->
[img-device-list]: ./media/iot-suite-remote-monitoring-device-info/image1.png
[img-device-edit]: ./media/iot-suite-remote-monitoring-device-info/image2.png
[img-device-remove]: ./media/iot-suite-remote-monitoring-device-info/image3.png

[lnk-iot-hub]: /documentation/services/iot-hub/
[lnk-identity-registry]: /documentation/articles/iot-hub-devguide-identity-registry/
[lnk-device-twin]: /documentation/articles/iot-hub-devguide-device-twins/
[lnk-docdb]: /documentation/services/documentdb/
[lnk-stream-analytics]: /documentation/services/stream-analytics/
[lnk-dynamic-telemetry]: /documentation/articles/iot-suite-dynamic-telemetry/
[lnk-predictive-overview]: /documentation/articles/iot-suite-predictive-overview/
[lnk-faq]: /documentation/articles/iot-suite-faq/
[lnk-security-groundup]: /documentation/articles/securing-iot-ground-up/
