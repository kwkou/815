<properties
 pageTitle="开发人员指南 - 术语表 | Azure"
 description="与 IoT 中心相关的常见术语表"
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


# IoT 中心术语表

本文列出了一些与 IoT 中心关联的常用术语。

## 高级消息队列协议 (AMQP)

[AMQP](https://www.amqp.org/) 是 IoT 中心支持的一种消息传送协议，适用于与设备通信。若要详细了解 IoT Hub 支持的消息传送协议，请参阅 [Send and receive messages with IoT Hub](/documentation/articles/iot-hub-devguide-messaging/)（使用 IoT 中心发送和接收消息）。

## 云到设备 (C2D)

通常用来指从 IoT 中心发送到已连接设备的消息。这些消息通常是命令，用于指示设备采取某项操作。有关详细信息，请参阅 [Send and receive messages with IoT Hub](/documentation/articles/iot-hub-devguide-messaging/)（使用 IoT 中心发送和接收消息）。

## 条件

即设备应用报告的设备状态信息，例如当前正在使用的连接方法。设备还可以报告其功能。可以使用设备克隆查询条件和功能。

## <a name="data-point-message"></a> 数据点消息

数据点消息是指云到设备的消息，内含遥测数据（例如风速或温度）。

## 所需属性

在设备克隆上下文中，所需属性可与 *报告属性* 结合使用，同步设备配置或状态。所需属性只能由应用程序后端设置，由设备应用遵守。

## 设备到云 (D2C)

通常用来指从已连接设备发送到 IoT 中心的消息。这些消息可以是[数据点](#data-point-message)消息，也可以是[交互式](#interactive-message)消息。有关详细信息，请参阅 [Send and receive messages with IoT Hub](/documentation/articles/iot-hub-devguide-messaging/)（使用 IoT 中心发送和接收消息）。

## 设备标识注册表

[设备标识注册表](/documentation/articles/iot-hub-devguide-identity-registry/)是 IoT 中心的内置组件，用于存储允许连接到中心的单个设备的相关信息。

## 设备

在 IoT 上下文中，设备通常是指小型、独立的计算设备，可用于收集数据或控制其他设备。例如，设备可以是环境监视设备，也可以是控制器，控制温室中的浇水和通风系统。

## 设备克隆

[设备克隆](/documentation/articles/iot-hub-devguide-device-twins/)是物理设备在 IoT 中心的条件和配置设置副本。可以使用设备克隆管理物理设备的配置。

## 直接方法

可以通过调用 IoT 中心的 API，使用[直接方法](/documentation/articles/iot-hub-devguide-direct-methods/)触发在设备上执行的方法。

## 与事件中心兼容的终结点

若要读取发送到 IoT 中心的设备到云的消息，可以先连接到中心的终结点，然后使用与事件中心兼容的方法读取这些消息。与事件中心兼容的方法可以使用事件中心 SDK 和 Azure 流分析。

## 现场网关

无法直接连接到 IoT 中心的设备可以通过现场网关进行连接，而现场网关通常与设备一起部署在本地。有关详细信息，请参阅 [What is Azure IoT Hub?](/documentation/articles/iot-hub-what-is-iot-hub/)（什么是 Azure IoT 中心？）。

## 作业

解决方案后端可以使用作业在一组注册到 IoT 中心的设备上计划和跟踪活动。活动包括更新设备克隆的所需属性、更新设备克隆标记，以及调用直接方法。

## 协议网关

协议网关通常部署在云中，为连接到 IoT 中心的设备提供协议转换服务。有关详细信息，请参阅 [What is Azure IoT Hub?](/documentation/articles/iot-hub-what-is-iot-hub/)（什么是 Azure IoT 中心？）。

## <a name="interactive-message"></a> 交互式消息

交互式消息是云到设备的消息，可在应用程序后端触发即时操作。例如，设备可能会发送故障警报，而该故障会自动记录到 CRM 系统中。

## IoT 中心

IoT 中心是一项完全托管的 Azure 服务，可在数百万个 IoT 设备和一个解决方案后端之间实现安全可靠的双向通信。有关详细信息，请参阅 [What is Azure IoT Hub?](/documentation/articles/iot-hub-what-is-iot-hub/)（什么是 Azure IoT 中心？）。

## IoT 套件

Azure IoT 套件将多个 Azure 服务和预配置解决方案打包在一起。有了这些预配置解决方案，用户就可以快速启动常见 IoT 方案的端到端实现。有关详细信息，请参阅 [What is Azure IoT Suite?](/documentation/articles/iot-suite-overview/)（什么是 Azure IoT 套件？）。

## 作业

用户可以使用 IoT 中心的[作业](/documentation/articles/iot-hub-devguide-jobs/)，跨多个连接到中心的设备执行包括固件升级在内的操作。

## MQTT

[MQTT](http://mqtt.org/) 是 IoT 中心支持的一种消息传送协议，适用于与设备通信。若要详细了解 IoT Hub 支持的消息传送协议，请参阅 [Send and receive messages with IoT Hub](/documentation/articles/iot-hub-devguide-messaging/)（使用 IoT 中心发送和接收消息）。

## 报告属性

在设备克隆上下文中，报告属性可与 *所需属性* 结合使用，同步设备配置或状态。报告属性只能由设备应用设置，可由应用程序后端读取和查询。

## 标记

在设备克隆上下文中，标记是指由应用程序后端以 JSON 文档形式存储和检索的设备元数据。标记对设备上的应用不可见。

## 系统属性

在设备克隆上下文中，系统属性为只读，其中包括与设备使用情况相关的信息，例如上次活动时间和连接状态。

<!---HONumber=Mooncake_1031_2016-->