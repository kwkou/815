<properties
	pageTitle="适用于 C# 的 Azure IoT 中心入门 | Microsoft Azure"
	description="遵照本教程开始将 Azure IoT 中心与 C# 配合使用。"
	services="iot-hub"
	documentationCenter=".net"
	authors="dominicbetts"
	manager="timlt"
	editor=""/>

<tags
     ms.service="iot-hub"
     ms.date="12/14/2015"
     wacn.date="03/18/2016"/>

# 适用于 .NET 的 Azure IoT 中心入门

[AZURE.INCLUDE [iot-hub-selector-get-started](../includes/iot-hub-selector-get-started.md)]

## 介绍

Azure IoT 中心是一项完全托管的服务，可在数百万个 IoT 设备和一个解决方案后端之间实现安全可靠的双向通信。IoT 项目面临的最大挑战之一是如何可靠且安全地将设备连接到解决方案后端。为了解决此难题，IoT 中心：

- 提供可靠的设备到云和云到设备的超大规模消息传送。
- 使用每个设备的安全凭据和访问控制来实现安全通信。
- 包含最流行语言和平台的设备库。

本教程演示如何：

<!-- - 使用 Azure 门户创建 IoT 中心。 -->
- 在 IoT 中心内创建设备标识。
- 创建一个模拟设备，用于将遥测数据发送到云后端，以及从云后端接收命令。

本教程最后提供了三个 Windows 控制台应用程序：

* **CreateDeviceIdentity**，用于创建设备标识和关联的安全密钥以连接模拟设备。
* **ReadDeviceToCloudMessages**，显示模拟设备发送的遥测数据。
* **SimulatedDevice**，它使用前面创建的设备标识连接到 IoT 中心，并每秒发送遥测消息一次。

> [AZURE.NOTE] [IoT 中心 SDK][lnk-hub-sdks] 一文提供了有关多种 SDK 的信息，你可以使用这些 SDK 构建可在设备和解决方案后端上运行的应用程序。

若要完成本教程，你需要以下各项：

+ Microsoft Visual Studio 2015。

+ 有效的 Azure 帐户。<br/>如果你没有帐户，只需花费几分钟就能创建一个免费试用帐户。有关详细信息，请参阅 [Azure 免费试用][lnk-free-trial]。

<!--
## 创建 IoT 中心

需要为模拟设备创建 IoT 中心以供连接。以下步骤说明如何使用 Azure 门户完成此任务。

1. 登录到 [Azure 门户][lnk-portal]。

2. 在跳转栏中，依次单击“新建”、“物联网”、“Azure IoT 中心”。

    ![][1]

3. 在“IoT 中心”边栏选项卡中，选择 IoT 中心的配置。

    ![][2]

* 在“名称”框中，输入 IoT 中心的名称。如果该**名称**有效且可用，“名称”框中会出现绿色的勾选标记。
* 选择**定价层和缩放层**。本教程不需要特定的层。
* 在“资源组”中，创建新资源组或选择现有的资源组。有关详细信息，请参阅[使用资源组管理 Azure 资源][lnk-resource-groups]。
* 在“位置”中，选择要托管 IoT 中心的位置。  

4. 选择 IoT 中心配置选项后，单击“创建”。Azure 可能需要几分钟时间来创建 IoT 中心。若要检查状态，可以在“启动板”或“通知”面板中监视进度。

    ![][3]

5. 成功创建 IoT 中心后，请打开新 IoT 中心的边栏选项卡，记下“主机名”，然后单击“钥匙”图标。

    ![][4]

6. 单击“iothubowner”策略，然后复制并记下“iothubowner”边栏选项卡中的连接字符串。

    ![][5]

现已创建 IoT 中心，因此已具有完成本教程的其余部分所需的主机名称和连接字符串。

[AZURE.INCLUDE [iot-hub-get-started-cloud-csharp](../includes/iot-hub-get-started-cloud-csharp.md)]


[AZURE.INCLUDE [iot-hub-get-started-device-csharp](../includes/iot-hub-get-started-device-csharp.md)]

-->

## 运行应用程序

现在，你已准备就绪，可以运行应用程序了。

1.	在 Visual Studio 的解决方案资源管理器中右键单击你的解决方案，然后单击“设置启动项目”。选择“多个启动项目”，然后针对 **ReadDeviceToCloudMessages** 和 **SimulatedDevice** 项目选择“启动”作为“操作”。

   	![][41]

2.	按下 **F5** 启动这两个应用，使其运行。来自 **SimulatedDevice** 应用的控制台输出显示模拟设备发送给 IoT 中心的消息，而来自 **ReadDeviceToCloudMessages** 应用程序的控制台输出则显示 IoT 中心接收的消息。

   	![][42]

## 后续步骤

在本教程中，你已在门户中配置了新的 IoT 中心，然后在中心的标识注册表中创建了设备标识。你已在模拟设备中使用此设备标识创建了一个用于将设备到云的消息发送到中心的应用程序，并创建了用于显示中心所接收消息的另一个应用程序。可以使用以下教程继续探索 IoT 中心功能和其他 IoT 方案：

- [使用 IoT 中心发送云到设备的消息][lnk-c2d-tutorial]说明了如何将消息发送到设备，并处理 IoT 中心生成的传送反馈。
- [处理设备到云的消息][lnk-process-d2c-tutorial]说明了如何可靠处理来自设备的遥测数据和交互消息。
- [从设备上载文件][lnk-upload-tutorial]介绍了使用云到设备的消息来帮助从设备上载文件的模式。

<!-- Images. -->
[1]: ./media/iot-hub-csharp-csharp-getstarted/create-iot-hub1.png
[2]: ./media/iot-hub-csharp-csharp-getstarted/create-iot-hub2.png
[3]: ./media/iot-hub-csharp-csharp-getstarted/create-iot-hub3.png
[4]: ./media/iot-hub-csharp-csharp-getstarted/create-iot-hub4.png
[5]: ./media/iot-hub-csharp-csharp-getstarted/create-iot-hub5.png
[41]: ./media/iot-hub-csharp-csharp-getstarted/run-apps1.png
[42]: ./media/iot-hub-csharp-csharp-getstarted/run-apps2.png

<!-- Links -->
[lnk-c2d-tutorial]: /documentation/articles/iot-hub-csharp-csharp-c2d
[lnk-process-d2c-tutorial]: /documentation/articles/iot-hub-csharp-csharp-process-d2c
[lnk-upload-tutorial]: /documentation/articles/iot-hub-csharp-csharp-file-upload

[lnk-hub-sdks]: /documentation/articles/iot-hub-sdks-summary
[lnk-free-trial]: /pricing/1rmb-trial/
[lnk-resource-groups]: /documentation/articles/resource-group-portal
[lnk-portal]: https://manage.windowsazure.cn

<!---HONumber=Mooncake_0307_2016-->