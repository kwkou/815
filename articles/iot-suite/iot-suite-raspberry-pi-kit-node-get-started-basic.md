<properties
    pageTitle="使用 Node.js 与实际传感器将 Raspberry Pi 连接到 Azure IoT 套件 | Azure"
    description="使用适用于 Raspberry Pi 3 的 AzureIoT 初学者工具包和 Azure IoT 套件。 使用 Node.js 将 Raspberry Pi 连接到远程监控解决方案，从传感器将遥测数据发送到云，并响应从解决方案仪表板调用的方法。"
    services=""
    suite="iot-suite"
    documentationcenter=""
    author="dominicbetts"
    manager="timlt"
    editor="" />
<tags
    ms.service="iot-suite"
    ms.devlang="nodejs"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="04/24/2017"
    wacn.date="06/13/2017"
    ms.author="v-yiso"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="4a18b6116e37e365e2d4c4e2d144d7588310292e"
    ms.openlocfilehash="e241e2501d70526f939dc5d8a4ab8cab0c06f8ee"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/19/2017" />

# <a name="connect-your-raspberry-pi-3-to-the-remote-monitoring-solution-and-send-telemetry-from-a-real-sensor-using-nodejs"></a>使用 Node.js 将 Raspberry Pi 3 连接到远程监控解决方案，并从实际传感器发送遥测数据

[AZURE.INCLUDE [iot-suite-raspberry-pi-kit-selector](../../includes/iot-suite-raspberry-pi-kit-selector.md)]

本教程演示如何使用适用于 Raspberry Pi 3 的 AzureIoT 初学者工具包开发可与云通信的温度和湿度读取器。 本教程使用：

- Raspbian OS、Node.js 编程语言和 Azure IoT SDK for Node.js 实现示例设备。
- IoT 套件远程监控预配置解决方案作为基于云的后端。

## <a name="overview"></a>概述

在本教程中，将完成以下步骤：

- 将远程监控预配置解决方案的实例部署到 Azure 订阅。 此步骤会自动部署并配置多个 Azure 服务。
- 将设备和传感器设置为与计算机和远程监控解决方案通信。
- 更新用于连接到远程监控解决方案的示例设备代码，并发送可在解决方案仪表板上查看的遥测数据。

[AZURE.INCLUDE [iot-suite-raspberry-pi-kit-prerequisites](../../includes/iot-suite-raspberry-pi-kit-prerequisites.md)]

[AZURE.INCLUDE [iot-suite-provision-remote-monitoring](../../includes/iot-suite-provision-remote-monitoring.md)]

> [AZURE.WARNING]
> 远程监控解决方案在 Azure 订阅中预配一组 Azure 服务。 部署反映实际企业体系结构。 若要避免产生不必要的 Azure 使用费用，请在使用完预配置解决方案的实例后，在 azureiotsuite.cn 上将其删除。 如果再次需要预配置解决方案，可以轻松地重新创建它。 若要详细了解如何在远程监控解决方案运行时减少消耗，请参阅[出于演示目的配置 Azure IoT 套件预配置解决方案][lnk-demo-config]。

[AZURE.INCLUDE [iot-suite-raspberry-pi-kit-view-solution](../../includes/iot-suite-raspberry-pi-kit-view-solution.md)]

[AZURE.INCLUDE [iot-suite-raspberry-pi-kit-prepare-pi](../../includes/iot-suite-raspberry-pi-kit-prepare-pi.md)]

## <a name="download-and-configure-the-sample"></a>下载并配置示例

现在，可以在 Raspberry Pi 上下载并配置远程监控客户端应用程序。

### <a name="install-nodejs"></a>安装 Node.js

在 Raspberry Pi 上安装 Node.js。 IoT SDK for Node.js 需要版本 0.11.5 或更高版本的 Node.js。 以下步骤演示如何在 Raspberry Pi 上安装 Node.js v6.10.2：

1. 使用以下命令更新 Raspberry Pi：

    `sudo apt-get update`

1. 使用以下命令将 Node.js 二进制文件下载到 Raspberry Pi 上：

    `wget https://nodejs.org/dist/v6.10.2/node-v6.10.2-linux-armv7l.tar.gz`

1. 使用以下命令安装二进制文件：

    `sudo tar -C /usr/local --strip-components 1 -xzf node-v6.10.2-linux-armv7l.tar.gz`

1. 使用以下命令验证是否已成功安装 Node.js v6.10.2：

    `node --version`

### <a name="clone-the-repositories"></a>克隆存储库

如果尚未这样做，请通过在 Pi 上运行以下命令，克隆所需的存储库：

`cd ~`

`git clone --recursive https://github.com/Azure-Samples/iot-remote-monitoring-node-raspberrypi-getstartedkit.git`

### <a name="update-the-device-connection-string"></a>更新设备连接字符串

使用以下命令在 **nano** 编辑器中打开示例源文件：

`nano ~/iot-remote-monitoring-node-raspberrypi-getstartedkit/basic/remote_monitoring.js`

查找行：

    var connectionString = 'HostName=[Your IoT hub name].azure-devices.cn;DeviceId=[Your device id];SharedAccessKey=[Your device key]';

将占位符值替换为在本教程开始时创建并保存的设备和 IoT 中心信息。 保存所做的更改（按 **Ctrl-O**，然后按 **Enter**），然后退出编辑器（按 **Ctrl-X**）。

## <a name="run-the-sample"></a>运行示例

运行以下命令以安装示例的必备组件包：

`cd ~/iot-remote-monitoring-node-raspberrypi-getstartedkit/basic`

`npm install`

现在可以在 Raspberry Pi 上运行示例程序。 输入以下命令：

`sudo node ~/iot-remote-monitoring-node-raspberrypi-getstartedkit/basic/remote_monitoring.js`

以下示例输出是在 Raspberry Pi 的命令提示符下看到的输出示例：

![Raspberry Pi 应用的输出][img-raspberry-output]

随时都可按 **Ctrl-C** 退出程序。

[AZURE.INCLUDE [iot-suite-raspberry-pi-kit-view-telemetry](../../includes/iot-suite-raspberry-pi-kit-view-telemetry.md)]

## <a name="next-steps"></a>后续步骤

有关 Azure IoT 的更多示例和文档，请访问 [Azure IoT 开发人员中心](/develop/iot/)。


[img-raspberry-output]: ./media/iot-suite-raspberry-pi-kit-node-get-started-basic/app-output.png

[lnk-demo-config]: https://github.com/Azure/azure-iot-remote-monitoring/blob/master/Docs/configure-preconfigured-demo.md