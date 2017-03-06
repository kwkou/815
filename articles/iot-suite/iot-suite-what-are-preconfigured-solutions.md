<properties
 pageTitle="Azure IoT 预配置解决方案 | Azure"
 description="Azure IoT 预配置解决方案及其体系结构描述，以及指向其他资源的链接。"
 services=""
 suite="iot-suite"
 documentationCenter=""
 authors="dominicbetts"
 manager="timlt"
 editor=""/>

<tags
 ms.service="iot-suite"
 ms.devlang="na"
 ms.topic="get-started-article"
 ms.tgt_pltfrm="na"
 ms.workload="na"
 ms.date="11/16/2016"
 wacn.date="12/05/2016"
 ms.author="dobett"/>  


# Azure IoT 套件预配置解决方案是什么？

Azure IoT 套件预配置解决方案是你可以使用订阅部署到 Azure 的常见 IoT 解决方案模式的实现。你可以使用预配置解决方案实现以下目的：

- 将其用作自己的 IoT 解决方案的起点。
- 了解 IoT 解决方案设计和开发中的常见模式。

每个预配置解决方案是使用模拟设备生成遥测数据的完整的端到端实现。

除了在 Azure 中部署和运行解决方案，你还可以下载完整的源代码，然后自定义并扩展解决方案以满足特定的 IoT 要求。

> [AZURE.NOTE] 要部署其中一个预配置的解决方案，请访问 [Azure IoT 套件][lnk-azureiotsuite]。[IoT 预配置解决方案入门][lnk-getstarted-preconfigured]这篇文章介绍了有关如何部署并运行其中一个解决方案的详细信息。

下表显示了如何将解决方案映射到特定的 IoT 功能：

| 解决方案 | 数据提取 | 设备标识 | 命令和控制 | 规则和操作 | 预测分析 |
|------------------------|-----|-----|-----|-----|-----|
| [远程监视][lnk-getstarted-preconfigured] |是 |是 |是 |是 |- |
| [主动维护][lnk-predictive-maintenance] |是 |是 |是 |是 |是 |

- *数据提取*：向云中大规模输入数据。
- *设备标识*：管理每个连接设备的唯一标识。
- *命令和控制*：从云中向设备发送消息，使设备采取某些操作。
- *规则和操作*：解决方案后端使用规则作用于特定设备到云的数据。
- *预测分析*：解决方案后端将对设备到云数据的分析应用到预测应何时发生特定操作。例如，分析飞机发动机遥测数据来确定发动机维护何时到期。

## 远程监控预配置解决方案概述

我们选择在本文中讨论远程监控预配置解决方案，因为它介绍了许多其他解决方案共享的常见设计元素。

下图说明了远程监控解决方案的关键要素。以下各节介绍了有关这些元素的详细信息。

![远程监控预配置解决方案体系结构][img-remote-monitoring-arch]

## 设备

部署远程监控预配置解决方案时，在该解决方案中预配置了四个模拟冷却设备的设备。这些模拟设备具有内置的可发出遥测数据的温度和湿度模型。这些模拟设备用于说明解决方案中端到端的数据流，以及如果你是将该解决方案用作自定义实现的起点的后端开发人员，这些模拟设备还用于提供便利的遥测数据源和命令的目标。

在远程监控预配置解决方案中，当设备首次连接到 IoT 中心时，发送到 IoT 中心的设备信息消息列出了设备可以响应的命令的清单。在远程监控预配置解决方案中，这些命令为：

- *Ping 设备*：设备通过确认响应此命令。这对于检查设备是否仍然活动且正在侦听很有用。
- *开始遥测*：指示设备开始发送遥测数据。
- *停止遥测*：指示设备停止发送遥测数据。
- *更改设置点温度*：控制设备发送的模拟的温度遥测值。这对于测试后端逻辑很有用。
- *诊断遥测*：控制设备是否应将外部温度作为遥测数据发送。
- *更改设备状态*：设置设备报告的设备状态的元数据属性。这对于测试后端逻辑很有用。

你可以在该解决方案中添加更多可以发出相同遥测数据和对相同命令作出响应的模拟设备。

## IoT 中心

在此预配置的解决方案中，IoT 中心实例对应于典型的 [IoT 解决方案体系结构][lnk-what-is-azure-iot]中的 *云网关* 。

IoT 中心接收单个终结点上的设备的遥测数据。IoT 中心还维护特定于设备的终结点，终结点中的每个设备可以检索发送给它的命令。

IoT 中心通过服务端遥测数据读取终结点使收到的遥测数据可用。

## Azure 流分析

此预配置的解决方案使用三种 [Azure 流分析][lnk-asa] (ASA) 作业筛选设备的遥测数据流：


- *DeviceInfo 作业* - 将数据输出到事件中心，该中心将第一次连接设备或者设备响应**更改设备状态**命令时发送的特定于设备注册的消息路由到解决方案设备注册表（一种 DocumentDB 数据库）。
- *遥测数据作业* - 将所有原始遥测数据发送到 Azure blob 存储进行冷存储，并计算在解决方案仪表板中显示的遥测汇总数据。
- *规则作业* -筛选超出了任何规则阈值的遥测数据流，并将数据输出到事件中心。当规则触发时，解决方案门户仪表板视图在警报历史记录表中将该事件作为新行显示，并根据解决方案门户中的规则和操作视图上定义的设置触发操作。

在此预配置的解决方案中，ASA 作业是典型的 [IoT 解决方案体系结构][lnk-what-is-azure-iot]中 **IoT 解决方案后端**的组成部分。

## 事件处理器

在此预配置的解决方案中，事件处理器是典型的 [IoT 解决方案体系结构][lnk-what-is-azure-iot]中 **IoT 解决方案后端**的组成部分。

**DeviceInfo** 和**规则** ASA 作业将其输出发送到事件中心以传递到其他后端服务。该解决方案使用 [Web 作业][lnk-web-job]中运行的 [EventPocessorHost][lnk-event-processor] 实例从这些事件中心读取消息。**EventProcessorHost** 使用 **DeviceInfo** 数据更新 DocumentDB 数据库中的设备数据，使用**规则**数据调用逻辑应用和更新解决方案门户中显示的警报。

## <a name="device-identity-registry-and-documentdb"></a> 设备标识注册表和 DocumentDB

每个 IoT 中心都包括存储设备密钥的[设备标识注册表][lnk-identity-registry]。IoT 中心使用此信息对设备进行身份验证 - 设备必须已注册，并具有有效的密钥，然后才能连接到中心。

此解决方案还存储有关设备的其他信息，例如设备的状态、它们支持的命令，以及其他元数据。该解决方案使用 DocumentDB 数据库来存储特定于该解决方案的设备数据，并且该解决方案门户在 DocumentDB 数据库中检索要显示和编辑的数据。

该解决方案必须保持设备标识注册表中的信息与 DocumentDB 数据库的内容同步。**EventProcessorHost** 使用来自 **DeviceInfo** 流分析作业的数据来管理同步。

## 解决方案门户

![解决方案仪表板][img-dashboard]

解决方案门户是基于 Web 的 UI，作为预配置解决方案的一部分部署到云。通过解决方案门户你可以：

- 查看仪表板中的遥测数据和警报历史记录。
- 设置新设备。
- 管理和监视设备。
- 将命令发送到特定设备。
- 管理规则和操作。

在此预配置的解决方案中，解决方案门户是典型的 [IoT 解决方案体系结构][lnk-what-is-azure-iot]中 **IoT 解决方案后端**和**处理和业务连接**的组成部分。

## 后续步骤

有关 IoT 解决方案体系结构的详细信息，请参阅 [Azure IoT services: Reference Architecture（Azure IoT 服务：参考体系结构）][lnk-refarch]。

现在你已了解什么是预配置解决方案，接下来你可以通过部署*远程监视* 预配置解决方案来开始入门，请参阅：[Get started with the preconfigured solutions][lnk-getstarted-preconfigured]（预配置解决方案入门）。

[img-remote-monitoring-arch]: ./media/iot-suite-what-are-preconfigured-solutions/remote-monitoring-arch1.png
[img-dashboard]: ./media/iot-suite-what-are-preconfigured-solutions/dashboard.png
[lnk-what-is-azure-iot]: /documentation/articles/iot-suite-what-is-azure-iot/
[lnk-asa]: /documentation/services/stream-analytics/
[lnk-event-processor]: /documentation/articles/event-hubs-programming-guide/#event-processor-host
[lnk-web-job]: /documentation/articles/web-sites-create-web-jobs/
[lnk-identity-registry]: /documentation/articles/iot-hub-devguide-identity-registry/
[lnk-predictive-maintenance]: /documentation/articles/iot-suite-predictive-overview/
[lnk-azureiotsuite]: https://www.azureiotsuite.cn/
[lnk-refarch]: http://download.microsoft.com/download/A/4/D/A4DAD253-BC21-41D3-B9D9-87D2AE6F0719/Microsoft_Azure_IoT_Reference_Architecture.pdf
[lnk-getstarted-preconfigured]: /documentation/articles/iot-suite-getstarted-preconfigured-solutions/

<!---HONumber=Mooncake_0815_2016-->