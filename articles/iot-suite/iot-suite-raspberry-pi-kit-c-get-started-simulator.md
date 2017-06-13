<properties
    pageTitle="使用 C 与模拟遥测将 Raspberry Pi 连接到 Azure IoT 套件 | Azure"
    description="使用适用于 Raspberry Pi 3 的 Microsoft Azure IoT 初学者工具包和 Azure IoT 套件。 使用 C 将 Raspberry Pi 连接到远程监控解决方案，将模拟遥测数据发送到云，并响应从解决方案仪表板调用的方法。"
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
    ms.date="04/24/2017"
    wacn.date="06/13/2017"
    ms.author="v-yiso"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="4a18b6116e37e365e2d4c4e2d144d7588310292e"
    ms.openlocfilehash="c518bf28b262bda2373a809d2fed6aeb7489ed4f"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/19/2017" />

# <a name="connect-your-raspberry-pi-3-to-the-remote-monitoring-solution-and-send-simulated-telemetry-using-c"></a>使用 C 将 Raspberry Pi 3 连接到远程监控解决方案，并发送模拟遥测数据

[AZURE.INCLUDE [iot-suite-raspberry-pi-kit-selector](../../includes/iot-suite-raspberry-pi-kit-selector.md)]

本教程演示如何使用 Raspberry Pi 3 模拟要发送到云的温度和湿度数据。 本教程使用：

- Raspbian OS、C 编程语言和 Microsoft Azure IoT SDK for C 实现示例设备。
- IoT 套件远程监控预配置解决方案作为基于云的后端。

[AZURE.INCLUDE [iot-suite-raspberry-pi-kit-overview-simulator](../../includes/iot-suite-raspberry-pi-kit-overview-simulator.md)]

[AZURE.INCLUDE [iot-suite-provision-remote-monitoring](../../includes/iot-suite-provision-remote-monitoring.md)]

> [AZURE.WARNING]
> 远程监控解决方案在 Azure 订阅中预配一组 Azure 服务。 部署反映实际企业体系结构。 若要避免产生不必要的 Azure 使用费用，请在使用完预配置解决方案的实例后，在 azureiotsuite.cn 上将其删除。 如果再次需要预配置解决方案，可以轻松地重新创建它。 若要详细了解如何在远程监控解决方案运行时减少消耗，请参阅[出于演示目的配置 Azure IoT 套件预配置解决方案][lnk-demo-config]。

[AZURE.INCLUDE [iot-suite-raspberry-pi-kit-view-solution](../../includes/iot-suite-raspberry-pi-kit-view-solution.md)]

[AZURE.INCLUDE [iot-suite-raspberry-pi-kit-prepare-pi-simulator](../../includes/iot-suite-raspberry-pi-kit-prepare-pi-simulator.md)]

## <a name="download-and-configure-the-sample"></a>下载并配置示例

现在，可以在 Raspberry Pi 上下载并配置远程监控客户端应用程序。

### <a name="clone-the-repositories"></a>克隆存储库

如果尚未这样做，请通过在 Pi 上的终端中运行以下命令，克隆所需的存储库：

`cd ~`

`git clone --recursive https://github.com/Azure-Samples/iot-remote-monitoring-c-raspberrypi-getstartedkit.git`

### <a name="update-the-device-connection-string"></a>更新设备连接字符串

使用以下命令在 **nano** 编辑器中打开示例源文件：

`nano ~/iot-remote-monitoring-c-raspberrypi-getstartedkit/simulator/remote_monitoring/remote_monitoring.c`

找到以下行：

    static const char* deviceId = "[Device Id]";
    static const char* connectionString = "HostName=[IoTHub Name].azure-devices.cn;DeviceId=[Device Id];SharedAccessKey=[Device Key]";

将占位符值替换为在本教程开始时创建并保存的设备和 IoT 中心信息。 保存所做的更改（按 **Ctrl-O**，然后按 **Enter**），然后退出编辑器（按 **Ctrl-X**）。

## <a name="build-the-sample"></a>生成示例

通过在 Raspberry Pi 上的终端中运行以下命令，为适用于 C 的 Microsoft Azure IoT 设备 SDK 安装必备组件包：

`sudo apt-get update`

`sudo apt-get install g++ make cmake git libcurl4-openssl-dev libssl-dev uuid-dev`

现在可以在 Raspberry Pi 上生成已更新的示例解决方案：

`chmod +x ~/iot-remote-monitoring-c-raspberrypi-getstartedkit/simulator/build.sh`

`~/iot-remote-monitoring-c-raspberrypi-getstartedkit/simulator/build.sh`

现在可以在 Raspberry Pi 上运行示例程序。 输入以下命令：

  `sudo ~/cmake/remote_monitoring/remote_monitoring`

以下示例输出是在 Raspberry Pi 的命令提示符下看到的输出示例：

![Raspberry Pi 应用的输出][img-raspberry-output]

随时都可按 **Ctrl-C** 退出程序。

[AZURE.INCLUDE [iot-suite-raspberry-pi-kit-view-telemetry-simulator](../../includes/iot-suite-raspberry-pi-kit-view-telemetry-simulator.md)]

## <a name="next-steps"></a>后续步骤

有关 Azure IoT 的更多示例和文档，请访问 [Azure IoT 开发人员中心](/develop/iot/)。


[img-raspberry-output]: ./media/iot-suite-raspberry-pi-kit-c-get-started-simulator/appoutput.png

[lnk-demo-config]: https://github.com/Azure/azure-iot-remote-monitoring/blob/master/Docs/configure-preconfigured-demo.md