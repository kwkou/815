<properties
	pageTitle="使用 IoT 中心从设备上载文件 | Microsoft Azure"
	description="遵照本教程了解如何将 Azure IoT 中心与 C# 配合使用，以从设备上载文件。"
	services="iot-hub"
	documentationCenter=".net"
	authors="fsautomata"
	manager="timlt"
	editor=""/>

<tags
     ms.service="iot-hub"
     ms.date="09/29/2015"
     wacn.date="03/18/2016"/>

# 教程：如何使用 IoT 中心将文件从设备上载到云中

## 介绍

Azure IoT 中心是一项完全托管的服务，可在数百万个 IoT 设备和一个应用程序后端之间实现安全可靠的双向通信。以前的教程（[IoT 中心入门]和[使用 IoT 中心发送云到设备的消息]）演示了 IoT 中心的设备到云和云到设备的基本消息传送功能，以及如何从设备和云组件访问这些消息。[处理设备到云的消息]介绍了在 Azure Blob 存储中可靠存储设备到云消息的方法。然而，在某些情况下，来自设备的数据无法轻松映射到相对较小的设备到云消息。包含图片、视频、高频震动数据样本，或者包含某种形式的预处理数据的大型文件就是这种数据的一些例子。这些文件通常使用工具（例如[Hadoop] 堆栈）进行批处理。如果你偏好通过从设备上载文件来发送事件，仍可以使用 IoT 中心的安全性与可靠性功能。

本教程在[使用 IoT 中心发送云到设备的消息]中所述代码的基础上制作，说明如何使用云到设备消息来安全地向设备提供 Azure Blob URI 用于上载文件，以及如何使用 IoT 中心传送确认来触发来自应用后端的文件的处理。此方法的优点是重复使用 IoT 中心设备标识以及云到设备消息的传送确认，使应用后端知道文件已经成功上载。

> [AZURE.NOTE] 设备可以使用此处所用的方法安全地从云下载文件。

可以在[ IoT 中心开发人员指南]中找到有关云到设备的消息和 IoT 中心安全性的详细信息。

在本教程最后，你将运行两个 Windows 控制台应用程序：

* **SimulatedDevice**，这是[使用 IoT 中心发送云到设备的消息]中创建的应用的修改版本，可连接到 IoT 中心、接收包含 Azure Blob URI 的云到设备消息。对于每个收到的云到设备消息，它会触发将文件上载到指定 Blob URI 的操作。
* **SendCloudToDevice**，构建 Azure Blob URI（如[创建 SAS 并将其用于 Blob 服务](/documentation/articles/storage-dotnet-shared-access-signature-part-2/)中所述），在云到设备消息中通过 IoT 中心将 Azure Blob URI 发送到模拟设备，然后接收其传送确认。

> [AZURE.NOTE] IoT 中心通过 Azure IoT 设备 SDK 对许多设备平台和语言（包括 C、Java 和 Javascript）提供 SDK 支持。有关如何将设备连接到本教程中的代码（通常是连接到 Azure IoT 中心）的逐步说明，请参阅 [Azure IoT 开发人员中心]。适用于 Java 和 Node 的 Azure IoT 服务 SDK 即将推出。

为了完成本教程，你需要有：

+ Microsoft Visual Studio 2015。

+ 有效的 Azure 帐户。<br/>如果你没有帐户，只需花费几分钟就能创建一个试用帐户。有关详细信息，请参阅 [Azure 试用](/pricing/1rmb-trial/)。


[AZURE.INCLUDE [iot-hub-file-upload-cloud-csharp](../includes/iot-hub-file-upload-cloud-csharp.md)]


[AZURE.INCLUDE [iot-hub-file-upload-device-csharp](../includes/iot-hub-file-upload-device-csharp.md)]

## 运行应用程序

现在，你已准备就绪，可以运行应用程序了。

1.  在 Visual Studio 中，右键单击你的解决方案并选择“设置启动项目...”。选择“多个启动项目”，然后同时针对 **SimulatedDevice** 和 **SendCloudToDevice** 应用选择“启动”操作。

2.  按 **F5**，你应会看到所有应用程序启动。选择 **SendCloudToDevice** 窗口并按某个键。当模拟设备上载文件后，你会看到模拟设备输出消息，并且 **SendCloudToDevice** 应用显示已成功接收反馈。可以使用 [Azure 门户]或 Visual Studio 服务器资源管理器来检查存储帐户中的文件是否存在。

  ![][50]


## 后续步骤

在本教程中，你已学习如何利用云到设备的消息来简化从设备上载文件的操作。可以使用以下教程继续探索 IoT 中心功能和方案：

- [处理设备到云的消息]说明了如何可靠处理来自设备的遥测数据和交互消息。

有关 IoT 中心的更多信息：

* [IoT 中心概述]
* [IoT 中心开发人员指南]
* [IoT 中心指南]
* [Azure IoT 开发人员中心]

<!-- Images. -->

[50]: ./media/iot-hub-csharp-csharp-file-upload/run-apps1.png

<!-- Links -->

[Send Cloud-to-Device messages with IoT Hub]: /documentation/articles/iot-hub-csharp-csharp-c2d/

[Azure 门户]: https://manage.windowsazure.cn


[Hadoop]: /documentation/services/hdinsight/

[Get started with IoT Hub]: /documentation/articles/iot-hub-csharp-csharp-getstarted/
[使用 IoT 中心发送云到设备的消息]: /documentation/articles/iot-hub-csharp-csharp-c2d/
[处理设备到云的消息]: /documentation/articles/iot-hub-csharp-csharp-process-d2c/
[Uploading files from devices]: /documentation/articles/iot-hub-csharp-csharp-file-upload/

[IoT 中心概述]: /documentation/articles/iot-hub-what-is-iot-hub/
[IoT 中心指南]: /documentation/articles/iot-hub-guidance/
[ IoT 中心开发人员指南]: /documentation/articles/iot-hub-devguide/
[IoT 中心开发人员指南]: /documentation/articles/iot-hub-devguide/
[IoT Hub Supported Devices]: /documentation/articles/iot-hub-supported-devices/
[IoT 中心入门]: /documentation/articles/iot-hub-csharp-csharp-getstarted/
[Supported devices]: https://github.com/Azure/azure-iot-sdks/blob/master/doc/tested_configurations.md
[Azure IoT 开发人员中心]: /develop/iot

<!---HONumber=Mooncake_0307_2016-->