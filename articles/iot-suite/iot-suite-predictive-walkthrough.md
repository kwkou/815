<properties
 pageTitle="预见性维护演练 | Azure"
 description="Azure IoT 预见性维护预配置解决方案演练。"
 services=""
 suite="iot-suite"
 documentationCenter=""
 authors="aguilaaj"
 manager="timlt"
 editor=""/>  


<tags
 ms.service="iot-suite"
 ms.date="05/16/2016"
 wacn.date="08/22/2016"/>

# 预见性维护预配置解决方案演练

## 介绍

IoT 套件预见性维护预配置解决方案是一个用于商业应用场景的端到端解决方案，可预测可能发生故障的时间点。你可以主动对优化维护等活动运用此预配置解决方案。此解决方案结合主要的 Azure IoT 套件服务，包括带有实验的 [Azure 机器学习][lnk_machine_learning]工作区，可根据公用示例数据集预测飞机引擎的剩余使用年限 (RUL)。此解决方案提供完整的商业应用场景实现作为你规划和实施此类型 IoT 解决方案的起点，以满足你自己的特定业务需求。

## 逻辑体系结构

下图概述该预配置解决方案的逻辑组件：

![][img-architecture]

蓝色项是你在预配该预配置解决方案时选择的位置中预配的 Azure 服务。你可以在中国东部或中国北部区域预配该预配置解决方案。

某些资源不适用于你预配该预配置解决方案的区域。图表中的橙色项表示在给出选定区域的情况下，在最接近的可用区域（中国北部、中国东部）预配的 Azure 服务。

绿色项是表示飞机引擎的模拟设备。你可以在下面了解有关这些模拟设备的详细信息。

灰色项表示实现设备管理功能的组件。当前的预见性维护预配置解决方案版本不会预配这些资源。若要了解有关设备管理的详细信息，请参阅[远程监视预配置解决方案][lnk-remote-monitoring]。

## 模拟设备

在该预配置解决方案中，模拟设备代表飞机引擎。该解决方案预配了 2 个映射到单架飞机的引擎。每个引擎发出 4 种遥测：传感器 9、传感器 11、传感器 14 和传感器 15，以提供机器学习模型计算该引擎的剩余使用年限 (RUL) 所需的数据。每个模拟设备会将下列遥测消息发送到 IoT 中心：

周期计数。一个周期表示 2 至 10 小时不等的已完成飞行，而在飞行时间内会每半小时捕获一次遥测数据。

遥测。有 4 个代表引擎属性的传感器。这些传感器一般标记为传感器 9、传感器 11、传感器 14 和传感器 15。这 4 个传感器代表足以从 RUL 的机器学习模型获取有用结果的遥测。此模型根据包含实际引擎传感器数据的公用数据集创建而来。有关如何根据原始数据集创建该模型的详细信息，请参阅 [Cortana Intelligence Gallery Predictive Maintenance Template][lnk-cortana-analytics]（Cortana Intelligence 库预见性维护模板）。

模拟设备可以处理从 IoT 中心发送的下列命令：

| 命令 | 说明 |
|---------|-------------|
| StartTelemetry | 控制模拟的状态。<br/>使设备开始发送遥测 |
| StopTelemetry | 控制模拟的状态。<br/>使设备停止发送遥测 |

IoT 中心会提供设备命令确认。

## Azure 流分析作业

**作业：遥测**会使用两个语句来操作传入设备遥测流。第一个语句会从设备选择所有遥测，然后将这些数据从在 Web 应用中可视化的位置发送到 blob 存储。第二个语句会通过两分钟的滑动窗口计算平均传感器值，然后通过事件中心将这些数据发送到**事件处理器**。

## 事件处理器

**事件处理器**会采用已完成周期的平均传感器值，并将这些值传递到可公开机器学习定型模型的 API，以便计算引擎的 RUL。

## Azure 机器学习

有关如何根据原始数据集创建该模型的详细信息，请参阅 [Cortana Intelligence Gallery Predictive Maintenance Template][lnk-cortana-analytics]（Cortana Intelligence 库预见性维护模板）。

## 让我们开始

本节将逐步解说解决方案的组件、说明预期的用例，并提供示例。

### 预见性维护仪表板

Web 应用程序中的此页面会使用 PowerBI JavaScript 控件（请参阅 [PowerBI-visuals repository][lnk-powerbi]（PowerBI 可视化效果存储库））以可视化方式呈现：

- blob 存储中流分析作业的输出数据。
- 每个飞机引擎的 RUL 和周期计数。

### 观察云解决方案的行为

你可以查看已预配的资源，方法是浏览到 Azure 门户，然后导航到具有你选定的解决方案名称的资源组。

![][img-resource-group]

预配该预配置解决方案时，你会收到一封电子邮件，其中包含机器学习工作区的链接。如果此机器学习工作区处于“就绪”状态，你也可以从已预配解决方案的 [azureiotsuite.cn][lnk-azureiotsuite] 页面导航到此工作区。

![][img-machine-learning]

在解决方案门户中，你可以看到本示例预配了四个模拟设备，表示各有 2 个引擎的 2 架飞机，而每个引擎有 4 个传感器。当你第一次导航到解决方案门户时，模拟便会停止。

![][img-simulation-stopped]

单击“开始模拟”即可开始模拟，而你将看到传感器历史记录、RUL、周期和 RUL 历史记录填充仪表板。

![][img-simulation-running]

当 RUL 小于 160 时（出于演示目的而选择的任意阈值），解决方案门户会在 RUL 旁边显示警告符号，并将图片中的飞机引擎标成黄色。你会注意到 RUL 值有整体向下的趋势，但倾向于上下波动。这是因为周期长度和模型精确度不同的缘故。

![][img-simulation-warning]

完整模拟需要约 35 分钟的时间才能完成 148 个周期。160 RUL 阈值第一次在大约 5 分钟的时候达到，而这两个引擎在大约 8 分钟的时候同时达到阈值。

模拟会彻底运行 148 个周期的完整数据集并确定最终的 RUL 和周期值。

你可以随时停止模拟，但单击“开始模拟”会从数据集的开头重播模拟。

## 后续步骤

你已经运行预见性维护预配置解决方案，现在可能想要对其进行修改，请参阅[预配置解决方案自定义指南][lnk-customize]。

[IoT Suite - Under The Hood - Predictive Maintenance（IoT 套件 - 幕后 -预见性维护）](http://social.technet.microsoft.com/wiki/contents/articles/33527.iot-suite-under-the-hood-predictive-maintenance.aspx)TechNet 博客文章提供有关预见性维护预配置解决方案的其他详细信息。

  
[img-architecture]: ./media/iot-suite-predictive-walkthrough/architecture.png
[img-resource-group]: ./media/iot-suite-predictive-walkthrough/resource-group.png
[img-machine-learning]: ./media/iot-suite-predictive-walkthrough/machine-learning.png
[img-simulation-stopped]: ./media/iot-suite-predictive-walkthrough/simulation-stopped.png
[img-simulation-running]: ./media/iot-suite-predictive-walkthrough/simulation-running.png
[img-simulation-warning]: ./media/iot-suite-predictive-walkthrough/simulation-warning.png

[lnk-powerbi]: https://www.github.com/Microsoft/PowerBI-visuals
[lnk_machine_learning]: /home/features/machine-learning/
[lnk-remote-monitoring]: /documentation/articles/iot-suite/iot-suite-remote-monitoring-sample-walkthrough/
[lnk-cortana-analytics]: http://gallery.cortanaintelligence.com/Collection/Predictive-Maintenance-Template-3
[lnk-azureiotsuite]: https://www.azureiotsuite.cn/
[lnk-customize]: /documentation/articles/iot-suite/iot-suite-guidance-on-customizing-preconfigured-solutions/

<!---HONumber=Mooncake_0815_2016-->