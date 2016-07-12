<properties
	pageTitle="IoT 中心设备管理入门 | Azure"
	description="面向 C# 的 Azure IoT 中心设备管理入门教程。配合  Azure IoT SDK 使用 Azure IoT 中心和 C# 来实现设备管理。"
	services="iot-hub"
	documentationCenter=".net"
	authors="juanjperez"
	manager="timlt"
	editor=""/>

<tags
 ms.service="iot-hub"
 ms.date="04/29/2016"
 wacn.date="07/04/2016"/>

# 使用 node.js 进行 Azure IoT 中心设备管理入门（预览版）

[AZURE.INCLUDE [iot-hub-device-management-get-started-selector](../includes/iot-hub-device-management-get-started-selector.md)]

## 介绍
若要开始进行 Azure IoT 中心设备管理，需要创建 Azure IoT 中心、在 IoT 中心预配设备、启动多台模拟设备以及查看设备管理示例 UI 中的设备。本教程将指导你完成这些步骤。

> [AZURE.NOTE]  即使有现有的 IoT 中心，也需要创建一个新的 IoT 中心来启用设备管理功能，因为现有 IoT 中心尚不具备设备管理功能。公开发布设备管理功能后，将升级全部现有 IoT 中心，以获得设备管理功能。

## 先决条件

本教程假定你正在使用 Ubuntu Linux 开发计算机。

若要完成这些步骤，需要安装以下软件：

- Git

- GCC（4.9 版或更高版本）。可以使用 `gcc --version` 命令验证安装在环境中的当前版本。有关如何在 Ubuntu 14.04 上升级你的 GCC 版本的信息，请参阅 <http://askubuntu.com/questions/466651/how-do-i-use-the-latest-gcc-4-9-on-ubuntu-14-04>。

- [CMake](https://cmake.org/download/)（2.8 版或更高版本）。可以使用 `cmake --version` 命令验证安装在环境中的当前版本。

- Node.js 6.1.0 或更高版本。从 <https://nodejs.org/> 安装适用于平台的 Node.js。

- 一个有效的 Azure 订阅。如果你没有帐户，只需花费几分钟就能创建一个试用帐户。有关详细信息，请参阅 [Azure 试用][lnk-free-trial]。

## 创建支持设备管理的 IoT 中心

需要针对要连接到的模拟设备创建支持设备管理的 IoT 中心。以下步骤说明如何使用 Azure 门户完成此任务。

1.  登录到 [Azure 门户]。
2.  在跳转栏中，依次单击“新建”、“物联网”和“Azure IoT 中心”。

	![][img-new-hub]

3.  在“IoT 中心”边栏选项卡中，选择 IoT 中心的配置。

	![][img-configure-hub]

  -   在“名称”框中，输入 IoT 中心的名称。如果该“名称”有效且可用，“名称”框中会出现绿色的勾选标记。
  -   选择“定价和缩放层”。本教程不需要特定的层。
  -   在“资源组”中，创建新资源组或选择现有的资源组。有关详细信息，请参阅[使用资源组管理 Azure 资源]。
  -   选中“启用设备管理”复选框。
  -   在“位置”中，选择托管 IoT 中心的位置。在公开预览期间，仅在美国东部、北欧和东亚提供 IoT 中心设备管理功能。未来将在所有区域提供该功能。

  > [AZURE.NOTE]  如果不选中“启用设备管理”复选框，示例将不起作用。

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

1.  打开 shell。

2.  克隆 GitHub 存储库。**确保克隆的目录中不包含任何空格。**

	  ```
	  git clone --recursive --branch dmpreview https://github.com/Azure/azure-iot-sdks.git
	  ```

3.  从克隆 **azure-iot-sdks** 存储库的根文件夹，导航到 **azure-iot-sdks/c/build\_all/linux** 目录并执行以下命令以安装系统必备组件包和依赖库：

	  ```
	  ./setup.sh
	  ```


4.  从克隆 **azure-iot-sdks** 存储库的根文件夹，导航到 **azure-iot-sdks/node/service/samples** 目录并执行以下命令以将占位符值替换为上一部分中的连接字符串：

	  ```
	  ./setup.sh <IoT Hub Connection String>
	  ```

此脚本执行以下任务：

1.  运行 **cmake**，以创建生成模拟设备所需的生成文件。可执行文件位于 **azure-iot-sdks/node/service/samples/cmake/iotdm\_client/samples/iotdm\_simple\_sample** 中。注意，源文件位于文件夹 **azure-iot-sdks/c/iotdm\_client/samples/iotdm\_simple\_sample** 中。

2.  生成模拟设备可执行文件 **iotdm\_simple\_sample**。

3.  运行 `npm install` 以安装所需的包。

4.  运行 `node generate_devices.js` 以在 IoT 中心预配设备标识。**sampledevices.json** 中对设备进行了说明。预配设备后，凭据存储在 **devicecreds.txt** 文件中（该文件位于 **azure-iot-sdks/node/service/samples** 目录中）。

## 启动模拟设备

设备已添加到设备注册表中，现在即可启动模拟托管设备。Azure IoT 中心预配的每个设备标识都需要启动一个模拟设备。

使用 shell，导航到 **azure-iot-sdks/node/service/samples** 目录并运行：

  ```
  ./simulate.sh
  ```

此脚本为 **devicecreds.txt** 文件中列出的每台设备输出启动 **iotdm\_simple\_sample** 需要运行的命令。请从单独的终端窗口为每台模拟设备单独运行这些命令。模拟设备将继续运行，直至关闭命令窗口。

**iotdm\_simple\_sample** 应用程序使用面向 C 的 Azure IoT 中心设备管理客户端库生成，该客户端库支持创建可由 Azure IoT 中心托管的 IoT 设备。设备制造商可以使用此库来报告设备属性，并实现设备作业要求的执行操作。此库是作为开放源代码 Azure IoT 中心 SDK 的一部分提供的组件。

运行 **simulate.sh** 时，可在输出窗口中看到数据流。此输出显示传入和传出流量，以及应用程序特定回调函数中的 **printf** 语句。这可让你看到传入和传出流量，以及示例应用程序如何处理已解码的数据包。当设备连接到 IoT 中心时，该服务将自动启动，以观察设备上的资源。然后，IoT 中心 DM 客户端库将调用设备回调，以从设备检索最新值。

以下是 **iotdm\_simple\_sample** 示例应用程序的输出。在顶部，可以看到成功的“已注册”消息，该消息显示 ID 为 **Device11-7ce4a850** 的设备正在连接到 IoT 中心。

> [AZURE.NOTE]  如果需要不太详细的输出，请生成并运行零售配置。

![][img-output]

确保在完成以下部分时，保持所有模拟设备处于运行状态。

## 运行设备管理示例 UI

现在，你已设置了一个 IoT 中心且运行并注册管理了多个模拟设备，因此你可以部署设备管理示例 UI。设备管理示例 UI 为你提供关于如何使用设备管理 API 来构建交互式 UI 体验的工作示例。有关设备管理示例 UI 的详细信息，包括[已知问题](https://github.com/Azure/azure-iot-device-management#knownissues)，请参阅 [Azure IoT 设备管理 UI][lnk-dm-github] GitHub 存储库。

若要检索、构建和运行设备管理示例 UI，请执行以下步骤：

1. 打开 shell。

2. 通过键入 `node --version`，确认你已根据系统必备部分安装了 Node.js 6.1.0 或更高版本。

3. 通过运行 shell 中的以下命令来克隆 Azure IoT 设备管理 UI GitHub 存储库：

	```
	git clone https://github.com/Azure/azure-iot-device-management.git
	```
	
4. 在 Azure IoT 设备管理 UI 存储库的克隆副本的根文件夹中，运行以下命令来检索相关程序包：

	```
	npm install
	```

5. npm 安装命令完成后，运行 shell 中的以下命令以生成代码：

	```
	npm run build
	```

6. 使用文本编辑器在克隆文件夹的根目录中打开 user-config.json 文件。使用上一部分的 IoT 中心连接字符串替换文本“&lt;YOUR CONNECTION STRING HERE&gt;”并保存该文件。

7. 在 shell 中，运行以下命令启动设备管理 UX 应用：

	```
	npm run start
	```

8. 当命令提示符处报告“服务已启动”时，打开 Web 浏览器，然后通过以下 URL 导航到设备管理应用查看你的模拟设备：<http://127.0.0.1:3003>。

	![][img-dm-ui]

转到下一章设备管理教程时，使模拟设备和设备管理应用保持运行。


## 后续步骤

若要继续了解 Azure IoT 中心设备管理功能，请参阅[使用示例 UI 了解 Azure IoT 中心设备管理][lnk-sample-ui]教程。

<!-- images and links -->
[img-new-hub]: ./media/iot-hub-device-management-get-started/image1.png
[img-configure-hub]: ./media/iot-hub-device-management-get-started/image2.png
[img-monitor]: ./media/iot-hub-device-management-get-started/image3.png
[img-keys]: ./media/iot-hub-device-management-get-started/image4.png
[img-connection]: ./media/iot-hub-device-management-get-started/image5.png
[img-output]: ./media/iot-hub-device-management-get-started/image6.png
[img-dm-ui]: media/iot-hub-device-management-get-started-node/dmui.png

[lnk-free-trial]: /pricing/1rmb-trial/
[Azure 门户]: https://portal.azure.cn/
[使用资源组管理 Azure 资源]: /documentation/articles/resource-group-portal/
[lnk-dm-github]: https://github.com/Azure/azure-iot-device-management
[lnk-sample-ui]: /documentation/articles/iot-hub-device-management-ui-sample/

<!---HONumber=Mooncake_0523_2016-->