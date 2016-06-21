<properties
	pageTitle="IoT 中心设备管理入门 | Azure"
	description="面向 C# 的 Azure IoT 中心设备管理入门教程。配合  Azure IoT SDK 使用 Azure IoT 中心和 C# 来实现设备管理。"
	services="iot-hub"
	documentationCenter=".net"
	authors="ellenfosborne"
	manager="timlt"
	editor=""/>

<tags
 ms.service="iot-hub"
 ms.date="04/29/2016"
 wacn.date="05/30/2016"/>

# 使用 node.js 进行 Azure IoT 中心设备管理入门（预览版）

[AZURE.INCLUDE [iot-hub-device-management-get-started-selector](../includes/iot-hub-device-management-get-started-selector.md)]

## 介绍
若要开始进行 Azure IoT 中心设备管理，需要创建 Azure IoT 中心、在 IoT 中心预配设备，以及启动多台模拟设备。本教程将指导你完成这些步骤。

> [AZURE.NOTE]  即使有现有的 IoT 中心，也需要创建一个新的 IoT 中心来启用设备管理功能，因为现有 IoT 中心尚不具备设备管理功能。公开发布设备管理功能后，将升级全部现有 IoT 中心，以获得设备管理功能。

## 先决条件

若要完成这些步骤，需要安装以下各项：

- Git
- 节点
- npm
- CMake（2.8 版或更高版本）。从 <https://cmake.org/download/> 安装 CMake。确保选中“将 CMake 添加到当前用户 PATH 变量”的复选框。
- 一个有效的 Azure 订阅。

	如果你没有帐户，只需花费几分钟就能创建一个试用帐户。有关详细信息，请参阅 [Azure 试用][lnk-free-trial]。

## 创建支持设备管理的 IoT 中心

需要针对要连接到的模拟设备创建支持设备管理的 IoT 中心。以下步骤说明如何使用 Azure 门户完成此任务。

1.  登录到 [Azure 门户]。
2.  在跳转栏中，依次单击“新建”、“物联网”、“Azure IoT 中心”。

	![][img-new-hub]

3.  在“IoT 中心”边栏选项卡中，选择 IoT 中心的配置。

	![][img-configure-hub]

  -   在“名称”框中，输入 IoT 中心的名称。如果该名称有效且可用，“名称”框中会出现绿色的勾选标记。
  -   选择“定价层和缩放层”。本教程不需要特定的层。
  -   在“资源组”中，创建新资源组或选择现有的资源组。有关详细信息，请参阅[使用资源组管理 Azure 资源]。
  -   选中“启用设备管理”的复选框。
  -   在“位置”中，选择托管 IoT 中心的位置。在公开预览期间，仅在美国东部、北欧和东亚提供 IoT 中心设备管理功能。未来将在所有区域提供该功能。

  > [AZURE.NOTE]  如果不选中“启用设备管理”的复选框，示例将不起作用。

4.  选择 IoT 中心配置选项后，单击“创建”。Azure 可能需要几分钟时间来创建 IoT 中心。若要检查状态，可以在“启动板”或“通知”面板中监视进度。

	![][img-monitor]

5.  成功创建 IoT 中心后，请打开新 IoT 中心的边栏选项卡，记下“主机名”，然后单击“钥匙”图标。

	![][img-keys]

6.  单击“iothubowner”策略，然后复制并记下“iothubowner”边栏选项卡中的连接字符串。将该连接字符串复制到稍后可访问的位置，因为完成本教程的其余部分需要连接字符串。

 	> [AZURE.NOTE] 在生产方案中，切勿使用 **iothubowner** 凭据。

	![][img-connection]

现在就已经创建好支持设备管理的 IoT 中心。将需要连接字符串来完成本教程的其余部分。

## 生成示例并在 IoT 中心预配设备

在本部分，将运行一个脚本来生成模拟设备和示例，并在 IoT 中心的设备注册表中预配一组新的设备标识。设备无法连接到 IoT 中心，除非它在设备注册表中具有相应的条目。

若要生成示例并在 IoT 中心预配设备，请遵循以下步骤进行操作：

1.  打开终端。

2.  克隆 GitHub 存储库。**确保克隆的目录中不包含任何空格。**

	  ```
	  git clone --recursive --branch dmpreview https://github.com/Azure/azure-iot-sdks.git
	  ```

3.  从克隆 **azure-iot-sdks** 存储库的根文件夹，浏览到 **azure-iot-sdks/node/service/samples** 目录并运行，以将占位符值替换为上一部分中的连接字符串：

	  ```
	  setup.bat <IoT Hub Connection String>
	  ```

此脚本执行以下任务：

1.  运行 **cmake**，以创建生成模拟设备所需的生成文件。可执行文件位于 **azure-iot-sdks/node/service/samples/cmake/iotdm\_client/samples/iotdm\_simple\_sample** 中。注意，源文件位于文件夹 **azure-iot-sdks/c/iotdm\_client/samples/iotdm\_simple\_sample** 中。

2.  生成模拟设备可执行文件 **iotdm\_simple\_sample**。

3.  运行 ``` npm install ``` 以安装所需的包。

4.  运行 ```node generate_devices.js``` 以在 IoT 中心预配设备标识。设备在 **sampledevices.json** 中说明。预配设备后，凭据存储在 **devicecreds.txt** 文件中（该文件位于 **azure-iot-sdks/node/service/samples** 目录下）。

## 启动模拟设备

设备已添加到设备注册表中，现在即可启动模拟托管设备。Azure IoT 中心预配的每个设备标识都需要启动一个模拟设备。

使用终端，在 **azure-iot-sdks/node/service/samples** 目录下运行：

  ```
  simulate.sh
  ```

此脚本输出为 **devicecreds.txt** 文件中列出的每台设备启动 **iotdm\_simple\_sample** 而需要运行的命令。请从单独的终端窗口为每台模拟设备单独运行这些命令。模拟设备将继续运行，直至关闭命令窗口。

**iotdm\_simple\_sample** 应用程序使用面向 C 的 Azure IoT 中心设备管理客户端库生成，该客户端库支持创建可由 Azure IoT 中心托管的 IoT 设备。设备制造商可以使用此库来报告设备属性，并实现设备作业要求的执行操作。此库是作为开放源代码 Azure IoT 中心 SDK 的一部分提供的组件。

运行 **simulate.sh** 时，可在输出窗口中看到数据流。此输出显示传入和传出流量，以及应用程序特定的回调函数中的 **printf** 语句。这可让你看到传入和传出流量，以及示例应用程序如何处理已解码的数据包。当设备连接到 IoT 中心时，该服务将自动启动，以观察设备上的资源。然后，IoT 中心 DM 客户端库将调用设备回调，以从设备检索最新值。

以下是 **iotdm\_simple\_sample** 示例应用程序的输出。在顶部，可以看到成功的“已注册”消息，该消息显示 ID 为 **Device11-7ce4a850** 的设备已连接到 IoT 中心。

> [AZURE.NOTE]  如果需要不太详细的输出，请生成并运行零售配置。

![][img-output]

确保在完成“后续步骤”中的教程时，保持所有模拟设备处于运行状态。

## 后续步骤

若要了解有关 Azure IoT 中心设备管理功能的详细信息，可学习以下教程：

- [如何使用设备克隆][lnk-tutorial-twin]

- [如何使用查询查找设备克隆][lnk-tutorial-queries]

- [如何使用设备作业更新设备固件][lnk-tutorial-jobs]


<!-- images and links -->
[img-new-hub]: ./media/iot-hub-device-management-get-started/image1.png
[img-configure-hub]: ./media/iot-hub-device-management-get-started/image2.png
[img-monitor]: ./media/iot-hub-device-management-get-started/image3.png
[img-keys]: ./media/iot-hub-device-management-get-started/image4.png
[img-connection]: ./media/iot-hub-device-management-get-started/image5.png
[img-output]: ./media/iot-hub-device-management-get-started/image6.png

[lnk-free-trial]: /pricing/1rmb-trial/
[Azure 门户]: https://portal.azure.cn/
[使用资源组管理 Azure 资源]: /documentation/articles/resource-group-portal
[lnk-tutorial-twin]: /documentation/articles/iot-hub-device-management-device-twin
[lnk-tutorial-queries]: /documentation/articles/iot-hub-device-management-device-query
[lnk-tutorial-jobs]: /documentation/articles/iot-hub-device-management-device-jobs

<!---HONumber=Mooncake_0523_2016-->