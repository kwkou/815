<properties
	pageTitle="使用 IoT 中心发送云到设备的消息 | Microsoft Azure"
	description="遵照本教程了解如何将 Azure IoT 中心与 C# 配合使用，以发送云到设备的消息。"
	services="iot-hub"
	documentationCenter=".net"
	authors="fsautomata"
	manager="timlt"
	editor=""/>

<tags
     ms.service="iot-hub"
     ms.date="02/03/2016"
     wacn.date="07/04/2016"/>

# 教程：如何使用 IoT 中心发送云到设备的消息

## 介绍

Azure IoT 中心是一项完全托管的服务，有助于在数百万个 IoT 设备和一个应用程序后端之间实现安全可靠的双向通信。[IoT 中心入门]教程说明如何创建 IoT 中心、在其中预配设备标识，以及编写模拟设备用于发送设备到云的消息。

本教程是在 [IoT 中心入门]的基础上制作的，介绍如何将云到设备的消息发送到单个设备、如何请求 IoT 中心的传送确认（反馈），以及如何从应用程序云后端接收消息。

可以在[ IoT 中心开发人员指南][IoT Hub Developer Guide - C2D]中找到有关云到设备的消息的详细信息。

在本教程最后，你将运行两个 Windows 控制台应用程序：

* **SimulatedDevice**，这是在 [IoT 中心入门]中创建的应用程序的修改版本，可连接到 IoT 中心并接收云到设备的消息。
* **SendCloudToDevice**，它将云到设备的消息通过 IoT 中心发送到模拟设备，然后接收 IoT 中心的传送确认。

> [AZURE.NOTE] IoT 中心通过 Azure IoT 设备 SDK 对许多设备平台和语言（包括 C、Java 和 Javascript）提供 SDK 支持。有关如何将设备连接到本教程中的代码（通常是连接到 Azure IoT 中心）的逐步说明，请参阅 [Azure IoT 开发人员中心]。

若要完成本教程，你需要以下各项：

+ Microsoft Visual Studio 2015。

+ 有效的 Azure 帐户。<br/>如果你没有帐户，只需花费几分钟就能创建一个试用帐户。有关详细信息，请参阅 [Azure 试用](/pricing/1rmb-trial)。

[AZURE.INCLUDE [iot-hub-c2d-device-csharp](../includes/iot-hub-c2d-device-csharp.md)]


[AZURE.INCLUDE [iot-hub-c2d-cloud-csharp](../includes/iot-hub-c2d-cloud-csharp.md)]

## 后续步骤

在本教程中，你已学习如何发送和接收云到设备的消息。可以使用以下教程继续探索 IoT 中心功能和方案：

- [处理设备到云的消息]说明如何可靠地处理来自设备的遥测数据和交互消息。
- [从设备上载文件]介绍了一种模式，该模式利用云到设备的消息来帮助从设备上载文件。

有关 IoT 中心的更多信息：

* [IoT 中心概述]
* [IoT 中心开发人员指南]
* [IoT 中心指南]
* [支持的设备平台和语言][Supported devices]
* [Azure IoT 开发人员中心]

<!-- Images. -->

<!-- Links -->

[Get started with IoT Hub]: /documentation/articles/iot-hub-csharp-csharp-getstarted/

[IoT Hub Developer Guide - C2D]: /documentation/articles/iot-hub-devguide/#c2d

[Azure portal]: https://manage.windowsazure.com

[Send Cloud-to-Device messages with IoT Hub]: /documentation/articles/iot-hub-csharp-csharp-c2d/
[处理设备到云的消息]: /documentation/articles/iot-hub-csharp-csharp-process-d2c/
[从设备上载文件]: /documentation/articles/iot-hub-csharp-csharp-file-upload/

[IoT 中心概述]: /documentation/articles/iot-hub-what-is-iot-hub/
[IoT 中心指南]: /documentation/articles/iot-hub-guidance/
[IoT 中心开发人员指南]: /documentation/articles/iot-hub-devguide/
[IoT Hub Supported Devices]: /documentation/articles/iot-hub-supported-devices/
[IoT 中心入门]: /documentation/articles/iot-hub-csharp-csharp-getstarted/
[Supported devices]: https://github.com/Azure/azure-iot-sdks/blob/master/doc/tested_configurations
[Azure IoT 开发人员中心]: /develop/iot

<!---HONumber=Mooncake_0307_2016-->