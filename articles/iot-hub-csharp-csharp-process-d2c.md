<properties
	pageTitle="处理 IoT 中心设备到云的消息 | Azure"
	description="遵照本教程了解处理 IoT 中心设备到云消息的有用模式。"
	services="iot-hub"
	documentationCenter=".net"
	authors="dominicbetts"
	manager="timlt"
	editor=""/>

<tags
     ms.service="iot-hub"
     ms.date="04/29/2016"
     wacn.date="07/04/2016"/>

# 教程：如何处理 IoT 中心设备到云的消息

## 介绍

Azure IoT 中心是一项完全托管的服务，可在数百万个 IoT 设备和一个应用程序后端之间实现安全可靠的双向通信。其他教程（[IoT 中心入门]和[使用 IoT 中心发送云到设备的消息]）说明了如何使用 IoT 中心的设备到云和云到设备的基本消息传送功能。

本教程以 [IoT 中心入门]中演示的代码为基础，呈现两种可用于处理设备到云消息的可缩放的模式：

- 在 [Azure Blob 存储]中可靠地存储设备到云的消息。一种很常见的情况是冷路径分析，因为你会在其中将 blob 中用作输入的遥测数据存储到分析进程中。这些分析进程可由[HDInsight (Hadoop)] 堆栈等工具驱动。

- 可靠处理交互式设备到云的消息。如果设备到云的消息因为应用程序后端中的一组操作而立即触发，则表示这些消息是交互式的。例如，设备可能将发送一条警报消息，触发在 CRM 系统中插入票证。与此相反，数据点消息则只送入分析引擎。例如，设备中存储以供将来分析使用的温度遥测是数据点消息。

由于 IoT 中心公开[事件中心][lnk-event-hubs]兼容的终结点来接收设备到云的消息，因此本教程使用了 [EventProcessorHost] 实例。此实例：

* 在 Azure blob 存储中可靠地存储数据点消息。
* 将交互式设备到云的消息转发到 Azure 服务总线队列以立即进行处理。

服务总线可以帮助确保可靠处理交互式消息，因为它提供了各消息的检查点，以及基于时间范围的重复数据删除。

> [AZURE.NOTE] **EventProcessorHost** 实例只是其中一种处理交互式消息的方法。其他选项包括 [Azure Service Fabric][lnk-service-fabric] 和 [Azure 流分析][lnk-stream-analytics]。

在本教程最后，你将运行三个 Windows 控制台应用：

* **SimulatedDevice**（[IoT 中心入门]教程中创建的应用的修改版本）每秒发送数据点设备到云的消息一次，每 10 秒发送交互式设备到云的消息一次。此应用使用 AMQPS 协议来与 IoT 中心通信。
* **ProcessDeviceToCloudMessages** 使用 [EventProcessorHost] 类从与事件中心兼容的终结点检索消息。然后将数据点消息可靠地存储在 Azure blob 存储中，并将交互式消息转发到服务总线队列。
* **ProcessD2CInteractiveMessages** 将交互式消息从服务总线队列中取消排队。

> [AZURE.NOTE] IoT 中心对许多设备平台和语言（包括 C、Java 和 JavaScript）提供 SDK 支持。有关如何将本教程中所述的模拟设备替换为物理设备，以及一般情况下如何将设备连接到 IoT 中心的分步说明，请参阅 [Azure IoT 开发人员中心]。

本教程直接适用于使用事件中心兼容消息的其他方案，例如 [HDInsight (Hadoop)] 项目。有关详细信息，请参阅 [Azure IoT 中心开发人员指南 - 设备到云]。

若要完成本教程，你需要以下各项：

+ Microsoft Visual Studio 2015。

+ 有效的 Azure 帐户。<br/>如果你没有帐户，只需花费几分钟就能创建一个试用帐户。有关详细信息，请参阅 [Azure 试用](/pricing/1rmb-trial)。

你应该了解 [Azure 存储空间]和 [Azure 服务总线]的一些基本知识。


[AZURE.INCLUDE [iot-hub-process-d2c-device-csharp](../includes/iot-hub-process-d2c-device-csharp.md)]


[AZURE.INCLUDE [iot-hub-process-d2c-cloud-csharp](../includes/iot-hub-process-d2c-cloud-csharp.md)]

## 运行应用程序

现在，你已准备就绪，可以运行应用程序了。

1.	在 Visual Studio 的解决方案资源管理器中右键单击你的解决方案，然后选择“设置启动项目”。选择“多个启动项目”，然后针对 **ProcessDeviceToCloudMessages**、**SimulatedDevice** 和 **ProcessD2CInteractiveMessages** 项目选择“启动”作为操作。

2.	按 **F5** 启动三个控制台应用程序。**ProcessD2CInteractiveMessages** 应用程序应处理从 **SimulatedDevice** 应用程序发送的每条交互式消息。

  ![三个控制台应用程序][50]

> [AZURE.NOTE] 若要查看 Blob 文件中的更新，需要将 **StoreEventProcessor** 类中的 **MAX\_BLOCK\_SIZE** 常量降低为较小值，例如 **1024**。这是因为达到模拟设备所发送的数据块大小限制需要一段时间。如果块大小比较小，则就无需等待这么久才能查看所创建和更新的 Blob。但是，使用较大的块可以提高应用程序的可缩放性。

## 后续步骤

在本教程中，你已学习如何通过使用 [EventProcessorHost] 类可靠地处理数据点与交互式设备到云的消息。

[从设备上传文件]教程通过使用模拟消息处理逻辑以本教程为基础编写。该教程还说明了一种使用云到设备的消息来帮助从设备上传文件的模式。

有关 IoT 中心的更多信息：

* [IoT 中心概述]
* [IoT 中心开发人员指南]
* [IoT 中心指南]
* [支持的设备平台和语言][Supported devices]
* [Azure IoT 开发人员中心]

<!-- Images. -->
[50]: ./media/iot-hub-csharp-csharp-process-d2c/run1.png


<!-- Links -->

[Azure Blob 存储]: /documentation/articles/storage-dotnet-how-to-use-blobs/

[HDInsight (Hadoop)]: /documentation/services/hdinsight/
[Service Bus Queue]: /documentation/articles/service-bus/service-bus-dotnet-get-started-with-queues
[EventProcessorHost]: http://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.eventprocessorhost(v=azure.95).aspx


[Azure IoT 中心开发人员指南 - 设备到云]: /documentation/articles/iot-hub-devguide/#d2c

[Azure 存储空间]: /documentation/services/storage/
[Azure 服务总线]: /documentation/services/service-bus/



[使用 IoT 中心发送云到设备的消息]: /documentation/articles/iot-hub-csharp-csharp-c2d/
[从设备上传文件]: /documentation/articles/iot-hub-csharp-csharp-file-upload/

[IoT 中心概述]: /documentation/articles/iot-hub-what-is-iot-hub/
[IoT 中心指南]: /documentation/articles/iot-hub-guidance/
[IoT 中心开发人员指南]: /documentation/articles/iot-hub-devguide/
[IoT 中心入门]: /documentation/articles/iot-hub-csharp-csharp-getstarted/
[Supported devices]: /documentation/articles/iot-hub-tested-configurations/
[Azure IoT 开发人员中心]: /develop/iot
[lnk-service-fabric]: https://azure.microsoft.com/documentation/services/service-fabric/
[lnk-stream-analytics]: https://azure.microsoft.com/documentation/services/stream-analytics/
[lnk-event-hubs]: https://azure.microsoft.com/documentation/services/event-hubs/
<!---HONumber=Mooncake_0425_2016-->