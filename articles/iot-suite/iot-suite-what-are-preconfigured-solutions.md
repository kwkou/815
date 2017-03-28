<properties
 pageTitle="Azure IoT 预配置解决方案 | Azure"
 description="Azure IoT 预配置解决方案及其体系结构描述，以及指向其他资源的链接。"
 services=""
 suite="iot-suite"
 documentationCenter=""
 author="dominicbetts"
 manager="timlt"
 editor=""/>

<tags
 ms.service="iot-suite"
 ms.devlang="na"
 ms.topic="get-started-article"
 ms.tgt_pltfrm="na"
 ms.workload="na"
 ms.date="02/15/2017"
 ms.author="dobett"
 wacn.date="03/28/2017"/>  


# Azure IoT 套件预配置解决方案是什么？

Azure IoT 套件预配置解决方案是你可以使用订阅部署到 Azure 的常见 IoT 解决方案模式的实现。你可以使用预配置解决方案实现以下目的：

- 将其用作自己的 IoT 解决方案的起点。
- 了解 IoT 解决方案设计和开发中的常见模式。

每个预配置解决方案是使用模拟设备生成遥测数据的完整的端到端实现。

除了在 Azure 中部署和运行解决方案，你还可以下载完整的源代码，然后自定义并扩展解决方案以满足特定的 IoT 要求。

> [AZURE.NOTE] 要部署其中一个预配置的解决方案，请访问 [Azure IoT 套件][lnk-azureiotsuite]。[IoT 预配置解决方案入门][lnk-getstarted-preconfigured]这篇文章介绍了有关如何部署并运行其中一个解决方案的详细信息。

下表显示了如何将解决方案映射到特定的 IoT 功能：

| 解决方案 | 数据引入 | 设备标识 | 设备管理 | 命令和控制 | 规则和操作 | 预测分析 |
| --- | --- | --- | --- | --- | --- | --- |
| [远程监视][lnk-getstarted-preconfigured] |是 |是 |是 |是 |是 |- | 
| [主动维护][lnk-predictive-maintenance] |是 |是 |- |是 |是 |是 |

* *数据提取*：向云中大规模输入数据。
* *设备标识*：管理唯一设备标识，并控制对解决方案的设备访问权限。
* *设备管理*：管理设备元数据，并执行设备重新启动和固件升级等操作。
* *命令和控制*：从云中向设备发送消息，使设备采取操作。
* *规则和操作*：解决方案后端使用规则作用于特定设备到云的数据。
* *预测分析*：解决方案后端对设备到云数据进行分析，由此预测应何时采取特定操作。例如，分析飞机发动机遥测数据来确定发动机维护何时到期。

## 远程监控预配置解决方案概述

我们选择在本文中讨论远程监控预配置解决方案，因为它介绍了许多其他解决方案共享的常见设计元素。

下图说明了远程监控解决方案的关键要素。以下各节介绍了有关这些元素的详细信息。

![远程监控预配置解决方案体系结构][img-remote-monitoring-arch]

## 设备
部署远程监控预配置解决方案时，在该解决方案中预配置了四个模拟冷却设备的设备。这些模拟设备具有内置的可发出遥测数据的温度和湿度模型。包含这些模拟设备的目的在于：
- 演示贯穿整个解决方案的端到端数据流。
- 提供方便的遥测源。
- 如果你是后端开发人员，且将解决方案用作自定义实现的起始点，则可为其提供方法或命令的目标。

解决方案中的模拟设备可以响应以下云到设备的通信：

- *方法（[直接方法][lnk-direct-methods]）*：双向通信方法，使用此方法，设备可立即作出响应。
- *命令（云到设备的消息）*：单向通信方法，使用此方法，设备会从持久队列中检索命令。

有关这些不同方法的比较，请参阅[云到设备的通信指南][lnk-c2d-guidance]。

在预配置解决方案中，当设备首次连接到 IoT 中心时，会向中心发送设备信息消息，该消息枚举了设备可以响应的方法。在远程监视预配置解决方案中，模拟设备支持以下方法：

* *启动固件更新*：此方法在设备上启动异步任务以执行固件更新。此异步任务使用报告的属性将状态更新传送给解决方案仪表板。
* *重新启动*：此方法使模拟设备重新启动。
* *FactoryReset*：此方法会触发模拟设备上的出厂设置恢复。

在预配置解决方案中，当设备首次连接到 IoT 中心时，会向中心发送设备信息消息，该消息枚举设备可以响应的命令。在远程监视预配置解决方案中，模拟设备支持以下命令：

* *Ping 设备*：设备通过确认响应此命令。此命令对于检查设备是否仍然活动且正在侦听很有用。
* *开始遥测*：指示设备开始发送遥测数据。
* *停止遥测*：指示设备停止发送遥测数据。
* *更改设置点温度*：控制设备发送的模拟的温度遥测值。此命令对于测试后端逻辑很有用。
* *诊断遥测*：控制设备是否应将外部温度作为遥测数据发送。
* *更改设备状态*：设置设备报告的设备状态的元数据属性。此命令对于测试后端逻辑很有用。

可以在该解决方案中添加更多可以发出相同遥测数据和对相同方法和命令作出响应的模拟设备。

除响应命令和方法外，解决方案还使用[设备孪生][lnk-device-twin]。设备使用设备孪生向解决方案后端报告属性值。解决方案仪表板使用设备孪生在设备上设置新的所需属性值。例如，固件更新过程中，模拟设备将使用报告的属性报告更新状态。

## IoT 中心
在此预配置的解决方案中，IoT 中心实例对应于典型的 [IoT 解决方案体系结构][lnk-what-is-azure-iot]中的*云网关*。

IoT 中心接收单个终结点上的设备的遥测数据。IoT 中心还维护特定于设备的终结点，终结点中的每个设备可以检索发送给它的命令。

IoT 中心通过服务端遥测数据读取终结点使收到的遥测数据可用。

使用 IoT 中心的设备管理功能，可以从解决方案门户中管理设备属性，并计划执行以下操作的作业：

- 重新启动设备
- 更改设备状态
- 固件更新

## Azure 流分析
此预配置的解决方案使用三种 [Azure 流分析][lnk-asa] (ASA) 作业筛选设备的遥测数据流：

* *DeviceInfo 作业* - 将数据输出到事件中心，该中心将特定于设备注册的消息路由到解决方案设备注册表（一种 DocumentDB 数据库）。设备第一次连接或者响应**更改设备状态**命令时会发送此消息。
* *遥测数据作业* - 将所有原始遥测数据发送到 Azure blob 存储进行冷存储，并计算在解决方案仪表板中显示的遥测汇总数据。
* *规则作业* -筛选超出了任何规则阈值的遥测数据流，并将数据输出到事件中心。触发规则时，解决方案门户仪表板视图会在警报历史记录表中将此事件显示为新行。这些规则也可以基于解决方案门户中“规则”和“操作”视图上定义的设置来触发操作。

在此预配置的解决方案中，ASA 作业是典型的 [IoT 解决方案体系结构][lnk-what-is-azure-iot]中 **IoT 解决方案后端**的组成部分。

## 事件处理器

在此预配置的解决方案中，事件处理器是典型的 [IoT 解决方案体系结构][lnk-what-is-azure-iot]中 **IoT 解决方案后端**的组成部分。

**DeviceInfo** 和**规则** ASA 作业将其输出发送到事件中心以传递到其他后端服务。该解决方案使用 [Web 作业][lnk-web-job]中运行的 [EventPocessorHost][lnk-event-processor] 实例从这些事件中心读取消息。**EventProcessorHost**：
- 使用 **DeviceInfo** 数据更新 DocumentDB 数据库中的设备数据。
- 使用**规则**数据调用逻辑应用，并更新解决方案门户中显示的警报。

## <a name="device-identity-registry-and-documentdb"></a> 设备标识注册表和 DocumentDB
每个 IoT 中心都包括存储设备密钥的[设备标识注册表][lnk-identity-registry]。IoT 中心使用此信息对设备进行身份验证 - 设备必须已注册，并具有有效的密钥，然后才能连接到中心。

[设备孪生][lnk-device-twin]是由 IoT 中心管理的 JSON 文档。设备的设备孪生包含：

- 设备向中心发送的报告属性。可以在解决方案门户中查看这些属性。
- 想要发送到设备的所需属性。可以在解决方案门户中设置这些属性。
- 只有设备孪生中存在标记，设备上没有。可以使用这些标记来筛选解决方案门户中的设备列表。

此解决方案使用设备孪生来管理设备元数据。解决方案还使用 DocumentDB 数据库来存储特定于解决方案的其他设备数据，例如每个设备支持的命令和命令历史记录。

该解决方案必须保持设备标识注册表中的信息与 DocumentDB 数据库的内容同步。**EventProcessorHost** 使用来自 **DeviceInfo** 流分析作业的数据来管理同步。

## 解决方案门户
![解决方案门户][img-dashboard]

解决方案门户是基于 Web 的 UI，作为预配置解决方案的一部分部署到云。通过解决方案门户你可以：

* 查看仪表板中的遥测数据和警报历史记录。
* 设置新设备。
* 管理和监视设备。
* 将命令发送到特定设备。
* 调用特定设备上的方法。
* 管理规则和操作。
* 计划要在一台或多台设备上运行的作业。

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
[lnk-c2d-guidance]: /documentation/articles/iot-hub-devguide-c2d-guidance/
[lnk-device-twin]: /documentation/articles/iot-hub-devguide-device-twins/
[lnk-direct-methods]: /documentation/articles/iot-hub-devguide-direct-methods/

<!---HONumber=Mooncake_0327_2017-->