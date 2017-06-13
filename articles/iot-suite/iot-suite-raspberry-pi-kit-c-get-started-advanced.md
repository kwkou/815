<properties
    pageTitle="使用 C 将 Raspberry Pi 连接到 Azure IoT 套件以支持固件更新 | Azure"
    description="使用适用于 Raspberry Pi 3 的 Microsoft Azure IoT 初学者工具包和 Azure IoT 套件。 使用 C 将 Raspberry Pi 连接到远程监控解决方案，从传感器将遥测数据发送到云，并执行远程固件更新。"
    services=""
    suite="iot-suite"
    documentationcenter=""
    author="dominicbetts"
    manager="timlt"
    editor="" />
<tags
    ms.service="iot-suite"
    ms.devlang="c"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="04/25/2017"
    wacn.date="06/13/2017"
    ms.author="v-yiso"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="4a18b6116e37e365e2d4c4e2d144d7588310292e"
    ms.openlocfilehash="267e969be49aa20476e06dfe659f91c5aea4e929"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/19/2017" />

# <a name="connect-your-raspberry-pi-3-to-the-remote-monitoring-solution-and-enable-remote-firmware-updates-using-c"></a>使用 C 将 Raspberry Pi 3 连接到远程监控解决方案并启用远程固件更新

[AZURE.INCLUDE [iot-suite-raspberry-pi-kit-selector](../../includes/iot-suite-raspberry-pi-kit-selector.md)]

本教程演示如何使用适用于 Raspberry Pi 3 的 Azure IoT 初学者工具包完成以下任务：

* 开发可与云通信的温度和湿度读取器。
* 启用并执行远程固件更新以更新 Raspberry Pi 上的客户端应用程序。

本教程使用：

- Raspbian OS、C 编程语言和 Azure IoT SDK for C 实现示例设备。
- IoT 套件远程监控预配置解决方案作为基于云的后端。

## <a name="overview"></a>概述

在本教程中，将完成以下步骤：

- 将远程监控预配置解决方案的实例部署到 Azure 订阅。 此步骤会自动部署并配置多个 Azure 服务。
- 将设备和传感器设置为与计算机和远程监控解决方案通信。
- 更新用于连接到远程监控解决方案的示例设备代码，并发送可在解决方案仪表板上查看的遥测数据。
- 使用示例设备代码更新客户端应用程序。

[AZURE.INCLUDE [iot-suite-raspberry-pi-kit-prerequisites](../../includes/iot-suite-raspberry-pi-kit-prerequisites.md)]

[AZURE.INCLUDE [iot-suite-provision-remote-monitoring](../../includes/iot-suite-provision-remote-monitoring.md)]

> [AZURE.WARNING]
> 远程监控解决方案在 Azure 订阅中预配一组 Azure 服务。 部署反映实际企业体系结构。 若要避免产生不必要的 Azure 使用费用，请在使用完预配置解决方案的实例后，在 azureiotsuite.cn 上将其删除。 如果再次需要预配置解决方案，可以轻松地重新创建它。 若要详细了解如何在远程监控解决方案运行时减少消耗，请参阅[出于演示目的配置 Azure IoT 套件预配置解决方案][lnk-demo-config]。

[AZURE.INCLUDE [iot-suite-raspberry-pi-kit-view-solution](../../includes/iot-suite-raspberry-pi-kit-view-solution.md)]

[AZURE.INCLUDE [iot-suite-raspberry-pi-kit-prepare-pi](../../includes/iot-suite-raspberry-pi-kit-prepare-pi.md)]

## <a name="download-and-configure-the-sample"></a>下载并配置示例

现在，可以在 Raspberry Pi 上下载并配置远程监控客户端应用程序。

### <a name="clone-the-repositories"></a>克隆存储库

如果尚未这样做，请通过在 Pi 上运行以下命令，克隆所需的存储库：

`cd ~`

`git clone --recursive https://github.com/Azure-Samples/iot-remote-monitoring-c-raspberrypi-getstartedkit.git`

### <a name="update-the-device-connection-string"></a>更新设备连接字符串

使用以下命令在 **nano** 编辑器中打开示例配置文件：

`nano ~/iot-remote-monitoring-c-raspberrypi-getstartedkit/advanced/config/deviceinfo`

将占位符值替换为在本教程开始时创建并保存的设备 ID 和 IoT 中心信息。

完成后，deviceinfo 文件的内容应如以下示例所示：

    yourdeviceid
    HostName=youriothubname.azure-devices.cn;DeviceId=yourdeviceid;SharedAccessKey=yourdevicekey

保存所做的更改（按 **Ctrl-O**，然后按 **Enter**），然后退出编辑器（按 **Ctrl-X**）。

## <a name="build-the-sample"></a>生成示例

如果尚未这样做，请通过在 Raspberry Pi 上的终端中运行以下命令，为适用于 C 的 Microsoft Azure IoT 设备 SDK 安装必备组件包：

`sudo apt-get update`

`sudo apt-get install g++ make cmake git libcurl4-openssl-dev libssl-dev uuid-dev`

现在可以在 Raspberry Pi 上生成示例解决方案：

`chmod +x ~/iot-remote-monitoring-c-raspberrypi-getstartedkit/advanced/1.0/build.sh`

`~/iot-remote-monitoring-c-raspberrypi-getstartedkit/advanced/1.0/build.sh`

现在可以在 Raspberry Pi 上运行示例程序。 输入以下命令：

  `sudo ~/cmake/remote_monitoring/remote_monitoring`

以下示例输出是在 Raspberry Pi 的命令提示符下看到的输出示例：

![Raspberry Pi 应用的输出][img-raspberry-output]

随时都可按 **Ctrl-C** 退出程序。

[AZURE.INCLUDE [iot-suite-raspberry-pi-kit-view-telemetry-advanced](../../includes/iot-suite-raspberry-pi-kit-view-telemetry-advanced.md)]

1. 在解决方案仪表板中，单击“设备”即可访问“设备”页。 在“设备列表”中选择 Raspberry Pi。 然后选择“方法”：

    ![在仪表板中列出设备][img-list-devices]

1. 在“调用方法”页的“方法”下拉列表中，选择“InitiateFirmwareUpdate”。

1. 在“FWPackageURI”字段中，输入 **https://github.com/Azure-Samples/iot-remote-monitoring-c-raspberrypi-getstartedkit/raw/master/advanced/2.0/package/remote_monitoring.zip**。 此存档文件包含固件版本 2.0 的实现。

1. 选择“InvokeMethod”。 Raspberry Pi 上的应用会将确认发送回解决方案仪表板。 然后，它通过下载新版本的固件启动固件更新过程：

    ![显示方法历史记录][img-method-history]

## <a name="observe-the-firmware-update-process"></a>观察固件更新过程

可以在设备运行固件更新过程时观察该更新过程，可通过在解决方案仪表板中查看报告的属性来实现此目的：

1. 可以在 Raspberry Pi 上查看更新过程的进度：

    ![显示更新进度][img-update-progress]

    > [AZURE.NOTE]
    > 更新完成时，远程监控应用将以无提示方式重新启动。 使用 `ps -ef` 命令验证它是否正在运行。 如果要终止该过程，请使用带过程 ID 的 `kill` 命令。

1. 可以在解决方案门户中查看设备报告的固件更新状态。 以下屏幕截图显示更新过程的每个阶段的状态和持续时间，以及新的固件版本：

    ![显示作业状态][img-job-status]

    如果导航回仪表板，则可以验证固件更新后设备是否仍发送遥测数据。

> [AZURE.WARNING]
> 如果让远程监控解决方案在 Azure 帐户中保持运行状态，系统会按其运行时间计费。 若要详细了解如何在远程监控解决方案运行时减少消耗，请参阅[出于演示目的配置 Azure IoT 套件预配置解决方案][lnk-demo-config]。 请在用完预配置的解决方案后将其从 Azure 帐户中删除。

## <a name="next-steps"></a>后续步骤

有关 Azure IoT 的更多示例和文档，请访问 [Azure IoT 开发人员中心](/develop/iot/)。


[img-raspberry-output]: ./media/iot-suite-raspberry-pi-kit-c-get-started-advanced/app-output.png
[img-update-progress]: ./media/iot-suite-raspberry-pi-kit-c-get-started-advanced/updateprogress.png
[img-job-status]: ./media/iot-suite-raspberry-pi-kit-c-get-started-advanced/jobstatus.png
[img-list-devices]: ./media/iot-suite-raspberry-pi-kit-c-get-started-advanced/listdevices.png
[img-method-history]: ./media/iot-suite-raspberry-pi-kit-c-get-started-advanced/methodhistory.png

[lnk-demo-config]: https://github.com/Azure/azure-iot-remote-monitoring/blob/master/Docs/configure-preconfigured-demo.md