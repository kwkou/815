<properties
	pageTitle="适用于 Java 的 Azure IoT 中心入门 | Azure"
	description="适用于 Java 的 Azure IoT 中心入门教程。配合 Azure IoT SDK 使用 Azure IoT 中心和 Java 来实施物联网解决方案。"
	services="iot-hub"
	documentationCenter="java"
	authors="dominicbetts"
	manager="timlt"
	editor=""/>

<tags
     ms.service="iot-hub"
     ms.date="03/22/2016"
     wacn.date="07/04/2016"/>

# 适用于 Java 的 Azure IoT 中心入门

[AZURE.INCLUDE [iot-hub-selector-get-started](../includes/iot-hub-selector-get-started.md)]

## 介绍

Azure IoT 中心是一项完全托管的服务，可在数百万个物联网 (IoT) 设备和一个解决方案后端之间实现安全可靠的双向通信。IoT 项目面临的最大挑战之一是如何可靠且安全地将设备连接到解决方案后端。为了解决此难题，IoT 中心：

- 提供可靠的设备到云和云到设备的超大规模消息传送。
- 使用每个设备的安全凭据和访问控制来实现安全通信。
- 包含最流行语言和平台的设备库。

本教程演示如何：

- 使用 Azure 门户创建 IoT 中心。
- 在 IoT 中心内创建设备标识。
- 创建一个模拟设备用于将遥测数据发送到云后端。

本教程最后提供了三个 Java 控制台应用程序：

* **create-device-identity**，用于创建设备标识和关联的安全密钥以连接模拟设备。
* **read-d2c-messages**，显示模拟设备发送的遥测数据。
* **simulated-device**，它使用前面创建的设备标识连接到 IoT 中心，并使用 AMQPS 协议每秒发送遥测消息一次。

> [AZURE.NOTE] [IoT 中心 SDK][lnk-hub-sdks] 一文提供了有关多种 SDK 的信息，你可以使用这些 SDK 构建可在设备和解决方案后端上运行的应用程序。

若要完成本教程，你需要以下各项：

+ Java SE 8。<br/>[准备开发环境][lnk-dev-setup]介绍了如何在 Windows 或 Linux 上安装本教程所用的 Java。

+ Maven 3。<br/>[准备开发环境][lnk-dev-setup]介绍了如何在 Windows 或 Linux 上安装本教程所用的 Maven。

+ 有效的 Azure 帐户。<br/>如果你没有帐户，只需花费几分钟就能创建一个试用帐户。有关详细信息，请参阅 [Azure 试用][lnk-free-trial]。


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

7. 在 IoT 中心边栏选项卡上单击“设置”，然后在“设置”边栏选项卡上单击“消息传送”。在“消息传送”边栏选项卡上，记下“与事件中心兼容的名称”和“与事件中心的兼容终结点”。创建 **read-d2c-messages** 应用程序时，将要用到这些值。

    ![][6]

现在，你已创建 IoT 中心并获取了 IoT 中心主机名、IoT 中心连接字符串、与事件中心兼容的名称及与事件中心兼容的终结点，接下来需要完成本教程的余下部分。

[AZURE.INCLUDE [iot-hub-get-started-cloud-java](../includes/iot-hub-get-started-cloud-java.md)]


[AZURE.INCLUDE [iot-hub-get-started-device-java](../includes/iot-hub-get-started-device-java.md)]


## 运行应用程序

现在，你已准备就绪，可以运行应用程序了。

1. 在 read-d2c 文件夹中的命令提示符下，运行以下命令开始监视 IoT 中心：

    ```
    mvn exec:java -Dexec.mainClass="com.mycompany.app.App" 
    ```

    ![][7]

2. 在 simulated-device 文件夹中的命令提示符下，运行以下命令开始将遥测数据发送到 IoT 中心：

    ```
    mvn exec:java -Dexec.mainClass="com.mycompany.app.App" 
    ```

    ![][8]

3. [Azure 门户][lnk-portal]中的“使用情况”磁贴显示了发送到中心的消息数：

    ![][43]

## 后续步骤

在本教程中，你已在门户中配置了新的 IoT 中心，然后在中心的标识注册表中创建了设备标识。你已使用此设备标识来让模拟设备应用向中心发送设备到云的消息，并创建了用于显示中心所接收消息的应用。可以使用以下教程继续探索 IoT 中心功能和其他 IoT 方案：

- [使用 IoT 中心发送云到设备的消息][lnk-c2d-tutorial]说明了如何将消息发送到设备，并处理 IoT 中心生成的传送反馈。
- [处理设备到云的消息][lnk-process-d2c-tutorial]说明了如何可靠处理来自设备的遥测数据和交互消息。
- [从设备上载文件][lnk-upload-tutorial]介绍了使用云到设备的消息来帮助从设备上载文件的模式。

<!-- Images. -->
[1]: ./media/iot-hub-java-java-getstarted/create-iot-hub1.png
[2]: ./media/iot-hub-java-java-getstarted/create-iot-hub2.png
[3]: ./media/iot-hub-java-java-getstarted/create-iot-hub3.png
[4]: ./media/iot-hub-java-java-getstarted/create-iot-hub4.png
[5]: ./media/iot-hub-java-java-getstarted/create-iot-hub5.png
[6]: ./media/iot-hub-java-java-getstarted/create-iot-hub6.png
[7]: ./media/iot-hub-java-java-getstarted/runapp1.png
[8]: ./media/iot-hub-java-java-getstarted/runapp2.png
[43]: ./media/iot-hub-csharp-csharp-getstarted/usage.png

<!-- Links -->
[lnk-dev-setup]: https://github.com/Azure/azure-iot-sdks/blob/master/java/device/doc/devbox_setup.md
[lnk-c2d-tutorial]: /documentation/articles/iot-hub-csharp-csharp-c2d
[lnk-process-d2c-tutorial]: /documentation/articles/iot-hub-csharp-csharp-process-d2c
[lnk-upload-tutorial]: /documentation/articles/iot-hub-csharp-csharp-file-upload

[lnk-hub-sdks]: /documentation/articles/iot-hub-sdks-summary
[lnk-free-trial]: /pricing/1rmb-trial/
[lnk-resource-groups]: /documentation/articles/resource-group-portal
[lnk-portal]: https://manage.windowsazure.cn

<!---HONumber=Mooncake_0307_2016-->