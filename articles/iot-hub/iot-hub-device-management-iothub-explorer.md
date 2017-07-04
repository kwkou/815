<properties
    pageTitle="使用 iothub-explorer 进行 Azure IoT 设备管理 | Azure"
    description="使用 iothub-explorer CLI 工具进行 Azure IoT 中心设备管理，该特点是使用直接方法并提供孪生所需的属性管理选项。"
    services="iot-hub"
    documentationcenter=""
    author="shizn"
    manager="timtl"
    tags=""
    keywords="azure iot 设备管理, azure iot 中心设备管理, 设备管理 iot, iot 中心设备管理" />
<tags
    ms.assetid="b34f799a-fc14-41b9-bf45-54751163fffe"
    ms.service="iot-hub"
    ms.devlang="arduino"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="04/10/2017"
    wacn.date="06/05/2017"
    ms.author="v-yiso"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="08618ee31568db24eba7a7d9a5fc3b079cf34577"
    ms.openlocfilehash="b65765fda66304ea49c6c0fcc15a69148e8d6756"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/26/2017" />

# <a name="use-iothub-explorer-for-azure-iot-hub-device-management"></a>使用 iothub-explorer 进行 Azure IoT 中心设备管理

![端到端关系图](./media/iot-hub-get-started-e2e-diagram/2.png)

[AZURE.INCLUDE [iot-hub-get-started-note](../../includes/iot-hub-get-started-note.md)]

[iothub-explorer](https://github.com/azure/iothub-explorer) 是一种在主机上运行的 CLI 工具，用于管理 IoT 中心注册表中的设备标识。 它附带了可用于执行各种任务的管理选项。

| 管理选项          | 任务                                                                                                                            |
|----------------------------|---------------------------------------------------------------------------------------------------------------------------------|
| 直接方法             | 让设备执行操作，如开始或停止发送消息或重新启动设备。                                        |
| 孪生所需属性    | 让设备进入特定状态，例如将 LED 设置为绿色，或将遥测发送间隔设置为 30 分钟。         |
| 孪生报告属性   | 获取报告的设备状态。 例如，设备报告 LED 现在正在闪烁。                                    |
| 孪生标记                  | 将设备特定的元数据存储在云中。 例如，存储在自动售货机的部署位置。                         |
| 云到设备的消息   | 向设备发送通知。 例如，“今天很可能会下雨。 不要忘记带雨伞。”              |
| 设备孪生查询        | 查询所有设备孪生以检索符合任意条件的设备孪生，例如确定可供使用的设备。 |

有关这些选项的差异和使用指导的更详细说明，请参阅[设备到云通信指南](/documentation/articles/iot-hub-devguide-d2c-guidance/)和[云到设备通信指南](/documentation/articles/iot-hub-devguide-c2d-guidance/)。

> [AZURE.NOTE]
> 设备孪生是存储设备状态信息（元数据、配置和条件）的 JSON 文档。 IoT 中心为连接到它的每台设备保留一个设备孪生。 有关设备孪生的详细信息，请参阅[设备孪生入门](/documentation/articles/iot-hub-node-node-twin-getstarted/)。

## <a name="what-you-learn"></a>学习内容

将通过结合使用 iothub-explorer 和各种管理选项来进行了解。

## <a name="what-you-do"></a>准备工作

使用各种管理选项运行 iothub-explorer。

## <a name="what-you-need"></a>需要什么

- 已完成教程[设置设备](/documentation/articles/iot-hub-raspberry-pi-kit-node-get-started/)，其中涵盖以下要求：
  - 一个有效的 Azure 订阅。
  - 已在订阅中创建一个 Azure IoT 中心。
  - 一个可向 Azure IoT 中心发送消息的客户端应用程序。
- iothub-explorer。 （[安装 iothub-explorer](https://github.com/azure/iothub-explorer)）

## <a name="connect-to-your-iot-hub"></a>连接到 IoT 中心

通过运行以下命令连接到 IoT 中心：

    iothub-explorer login <your IoT hub connection string>

## <a name="use-iothub-explorer-with-direct-methods"></a>结合使用 iothub-explorer 和直接方法

通过运行以下命令在设备应用中调用 `start` 方法以向 IoT 中心发送消息：

    iothub-explorer device-method <your device Id> start

通过运行以下命令在设备应用中调用 `stop` 方法以停止向 IoT 中心发送消息：

    iothub-explorer device-method <your device Id> stop

## <a name="use-iothub-explorer-with-twins-desired-properties"></a>结合使用 iothub-explorer 和孪生所需的属性

通过运行以下命令将所需属性间隔设置为 3000：

    iothub-explorer update-twin mydevice {\"properties\":{\"desired\":{\"interval\":3000}}}

可通过设备读取此属性。

## <a name="use-iothub-explorer-with-twins-reported-properties"></a>结合使用 iothub-explorer 和孪生的报告属性

通过运行以下命令获取报告的设备属性：

    iothub-explorer get-twin <your device id>

其中一个属性是 $metadata.$lastUpdated，它显示该设备上次发送或接收消息的时间。

## <a name="use-iothub-explorer-with-twins-tags"></a>结合使用 iothub-explorer 和孪生的标记

通过运行以下命令显示设备的标记和属性：

    iothub-explorer get-twin <your device id>

通过运行以下命令向设备添加字段角色 = 温度和湿度：

    iothub-explorer update-twin <your device id> {\"tags\":{\"role\":\"temperature&humidity\"}}

## <a name="use-iothub-explorer-with-cloud-to-device-messages"></a>使用 iothub-explorer 发送云到设备的消息

运行以下命令，将“Hello World”消息发送到设备：

    iothub-explorer send <device-id> "Hello World"

有关使用此命令的真实应用场景，请参阅[使用 iothub-explorer 在设备与 IoT 中心之间发送和接收消息](/documentation/articles/iot-hub-explorer-cloud-device-messaging/)。

## <a name="use-iothub-explorer-with-device-twins-queries"></a>将 iothub-explorer 用于设备孪生查询

通过运行以下命令查询角色标记 =“温度和湿度”的设备：

    iothub-explorer query-twin "SELECT * FROM devices WHERE tags.role = 'temperature&humidity'"

通过运行以下命令查询除角色标记 =“温度和湿度”的设备以外的所有设备：

    iothub-explorer query-twin "SELECT * FROM devices WHERE tags.role != 'temperature&humidity'"

## <a name="next-steps"></a>后续步骤

现已了解如何结合使用 iothub-explorer 和各种管理选项。

[AZURE.INCLUDE [iot-hub-get-started-next-steps](../../includes/iot-hub-get-started-next-steps.md)]