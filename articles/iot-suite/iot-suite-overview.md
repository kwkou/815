<properties
	pageTitle="Azure IoT 套件概述 | Azure"
	description="概述 Azure IoT 套件如何提供物联网预配置解决方案，以收集、分析和存储数据，提供可视化效果，以及与其他系统集成。"
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


# Azure IoT 套件是什么？

Azure 物联网 (IoT) 服务提供有各种功能。这些企业级服务让你能够：

- 从设备收集数据
- 分析移动中的数据流
- 存储和查询大型数据集
- 可视化实时和历史数据
- 与后端办公系统集成

为了提供这些功能，Azure IoT 套件将多个 Azure 服务与自定义扩展打包在一起作为 *预配置解决方案* 。预配置解决方案是常见 IoT 解决方案模式的基础实现，可帮助你减少交付 IoT 解决方案所花费的时间。通过使用 [IoT 软件开发工具包][lnk-sdks]，你可以自定义和扩展这些解决方案来满足你自己的需求。也可以使用这些解决方案作为开发新 IoT 解决方案时的示例或模板。

> [AZURE.NOTE] Azure IoT 套件暂时只有远程监控功能上线。
此服务暂时只对签订世纪互联在线服务高级协议的企业客户开放。

## Azure IoT 套件中的 Azure IoT 服务

预配置解决方案通常使用下列服务：

- Azure IoT 套件的核心是 [Azure IoT 中心][lnk-iot-hub]服务。该服务提供设备到云和云到设备的消息传送功能，并充当云和其他主要 IoT 套件服务的网关。该服务允许你从你的设备大量接收消息，并将命令发送给你的设备。

- [Azure 流分析][lnk-asa]提供移动中的数据分析。IoT 套件利用该服务来处理传入遥测、执行聚合以及检测事件。预配置解决方案也会使用流分析来处理包含数据（例如元数据或来自设备的命令响应）的信息消息。这些解决方案使用流分析来处理来自你设备的消息，并将这些消息传送给其他服务。

- [Azure 存储空间][lnk-azure-storage]和 [Azure DocumentDB][lnk-document-db] 提供数据存储功能。预配置解决方案使用 blob 存储来存储遥测数据并使其可用于分析。这些解决方案使用 DocumentDB 来存储设备元数据，以及启用解决方案的设备管理功能。

- [Microsoft Power BI][lnk-power-bi] 提供数据可视化功能。借助 Power BI 的灵活性，你可以快速生成自己的交互式仪表板（使用 IoT 套件数据）。

有关典型 IoT 解决方案体系结构的概述，请参阅 [Azure 和物联网 (IoT)][iot-suite-what-is-azure-iot]。

## 预配置解决方案

IoT 套件包含预配置解决方案，可让你快速地开始使用，并浏览 Azure IoT 套件使其可行的常见 IoT 方案，例如 *远程监视* 和 *预见性维护* 。你可以将这些解决方案部署到 Azure 订阅，然后运行完整的端到端 IoT 方案。

## 后续步骤

现在，你已概要了解 IoT 套件可以执行哪些操作以及其主要组件是什么，接下来你可以详细了解 IoT 套件中的预配置解决方案，请参阅[什么是 Azure IoT 预配置解决方案？][lnk-what-are-preconfig]

[lnk-sdks]: /documentation/articles/iot-hub-sdks-summary/
[lnk-iot-hub]: /documentation/services/iot-hub/
[lnk-asa]: /documentation/services/stream-analytics/
[lnk-azure-storage]: /documentation/services/storage/
[lnk-document-db]: /documentation/services/documentdb/
[lnk-power-bi]: https://powerbi.microsoft.com/
[iot-suite-what-is-azure-iot]: /documentation/articles/iot-suite-what-is-azure-iot/
[lnk-what-are-preconfig]: /documentation/articles/iot-suite-what-are-preconfigured-solutions/

<!---HONumber=Mooncake_0815_2016-->