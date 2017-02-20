<properties
    pageTitle="预见性维护演练 | Azure"
    description="Azure IoT 预见性维护预配置解决方案演练。"
    services=""
    suite="iot-suite"
    documentationcenter=""
    author="dominicbetts"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="3c48a716-b805-4c99-8177-414cc4bec3de"
    ms.service="iot-suite"
    ms.devlang="na"
    ms.topic="get-started-article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="01/16/2017"
    wacn.date="02/10/2017"
    ms.author="dobett" />

# 预见性维护预配置解决方案演练

## 介绍

IoT 套件预见性维护预配置解决方案是一个用于商业应用场景的端到端解决方案，可预测可能发生故障的时间点。可主动使用此预配置解决方案执行维护优化等活动。该解决方案结合了重要的 Azure IoT 套件服务，例如 IoT 中心和流分析。该解决方案还包含一个基于公共示例数据集的模型，用于预测飞机引擎的剩余使用寿命 (RUL)。该模型在解决方案包含的某个虚拟机所托管的 R Server 上运行。此解决方案全面实施 loT 商业应用场景，以此为起点，你可以规划和实施满足自己特定业务需求的解决方案。

## 逻辑体系结构

下图概述该预配置解决方案的逻辑组件：

![][img-architecture]  


蓝色项是在预配该预配置解决方案时选择的区域中预配的 Azure 服务。[预配页][lnk-azureiotsuite]显示了可部署预配置解决方案的区域列表。

绿色项是表示飞机引擎的模拟设备。可在以下部分中深入了解这些模拟设备。

灰色项表示实现 *设备管理* 功能的组件。当前的预见性维护预配置解决方案版本不会预配这些资源。若要了解有关设备管理的详细信息，请参阅[远程监视预配置解决方案][lnk-remote-monitoring]。

## 模拟设备

在该预配置解决方案中，模拟设备代表飞机引擎。该解决方案预配有两个映射到单架飞机的引擎。每个引擎发出 4 种遥测数据：传感器 9、传感器 11、传感器 14 和传感器 15 为 R 模型提供所需的数据来计算引擎的 RUL。每个模拟设备会将下列遥测消息发送到 IoT 中心：

*周期计数* 。一个周期表示 2 至 10 小时不等的已完成飞行。在飞行期间，每半小时捕获一次遥测数据。

*遥测* 。有 4 个代表引擎属性的传感器。这些传感器一般标记为传感器 9、传感器 11、传感器 14 和传感器 15。这四个传感器代表足以从 RUL 模型获取有用结果的遥测装置。预配置解决方案中使用模型是基于包含实际引擎传感器数据的公共数据集创建的。有关如何根据原始数据集创建该模型的详细信息，请参阅 [Cortana Intelligence Gallery Predictive Maintenance Template][lnk-cortana-analytics]（Cortana Intelligence 库预见性维护模板）。

模拟设备可以处理在解决方案中通过 IoT 中心发送的以下命令：

| 命令 | 说明 |
| --- | --- |
| StartTelemetry |控制模拟的状态。<br/>使设备开始发送遥测 |
| StopTelemetry |控制模拟的状态。<br/>使设备停止发送遥测 |

IoT 中心会提供设备命令确认。

## Azure 流分析作业
**作业：遥测** 会使用两个语句来操作传入设备遥测流。第一个语句会从设备选择所有遥测，然后将这些数据从在 Web 应用中可视化的位置发送到 blob 存储。第二个语句会通过两分钟的滑动窗口计算平均传感器值，然后通过事件中心将这些数据发送到 **事件处理器** 。

## 事件处理器
**事件处理器主机** 在 Azure Web 作业中运行。**事件处理器**获取已完成周期的平均传感器值。然后，将这些值传递给某个 API，后者可公开用于计算引擎 RUL 的训练模型。该 API 由预配为解决方案一部分的 R Server 公开。

## R Server
R Server 实现使用派生自数据的模型，这些数据是从实际飞机引擎收集的。若要访问 R Server 和模型，请参阅 [ Azure IoT 套件][lnk-azureiotsuite]中预配解决方案的解决方案面板上的链接。


## 后续步骤
了解预见性维护预配置解决方案的关键组件后，可对其进行自定义。请参阅[预配置解决方案自定义指南][lnk-customize]。

你还可以浏览 IoT 套件预配置的解决方案的一些其他特性和功能：

* [有关 IoT 套件的常见问题][lnk-faq]
* [从头开始保障 IoT 安全][lnk-security-groundup]

[img-architecture]: ./media/iot-suite-predictive-walkthrough/architecture.png

[lnk-remote-monitoring]: /documentation/articles/iot-suite-remote-monitoring-sample-walkthrough/
[lnk-cortana-analytics]: http://gallery.cortanaintelligence.com/Collection/Predictive-Maintenance-Template-3
[lnk-azureiotsuite]: https://www.azureiotsuite.cn/
[lnk-customize]: /documentation/articles/iot-suite-guidance-on-customizing-preconfigured-solutions/
[lnk-faq]: /documentation/articles/iot-suite-faq/
[lnk-security-groundup]: /documentation/articles/securing-iot-ground-up/

<!---HONumber=Mooncake_0206_2017-->