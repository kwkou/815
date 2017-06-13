<properties
    pageTitle="使用 Intel NUC 将网关连接到 Azure IoT 套件 | Azure"
    description="使用 Microsoft IoT 商业网关工具包和远程监控预配置解决方案。 使用网关将 SensorTag 设备连接到远程监控解决方案，将遥测数据发送到云中，并响应从解决方案仪表板调用的方法。"
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
    ms.date="05/05/2017"
    wacn.date="06/13/2017"
    ms.author="v-yiso"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="4a18b6116e37e365e2d4c4e2d144d7588310292e"
    ms.openlocfilehash="07a13fa9cf0e181700be6b16af1f4ef2284d12d1"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/19/2017" />

# <a name="connect-your-azure-iot-gateway-to-the-remote-monitoring-preconfigured-solution-and-send-telemetry-from-a-sensortag"></a>将 Azure IoT 网关连接到远程监控预配置解决方案并通过 SensorTag 发送模拟遥测数据

[AZURE.INCLUDE [iot-suite-gateway-kit-selector](../../includes/iot-suite-gateway-kit-selector.md)]

本教程介绍如何使用 Azure IoT Edge 将 SensorTag 设备中的温度和湿度数据发送到远程监控预配置解决方案。 SensorTag 使用蓝牙连接到 Intel NUC 网关。 本教程使用：

- Azure IoT Edge 实现示例网关。
- IoT 套件远程监控预配置解决方案作为基于云的后端。

## <a name="overview"></a>概述

在本教程中，将完成以下步骤：

- 将远程监控预配置解决方案的实例部署到 Azure 订阅。 此步骤会自动部署并配置多个 Azure 服务。
- 将 Intel NUC 网关设备设置为与计算机和远程监控解决方案通信。
- 将 Intel NUC 网关设置为从 SensorTag 设备接收遥测数据，并将其发送到远程监控仪表板。

[AZURE.INCLUDE [iot-suite-gateway-kit-prerequisites](../../includes/iot-suite-gateway-kit-prerequisites.md)]

[Texas Instruments BLE SensorTag][lnk-sensortag]. 本教程通过蓝牙从 SensorTag 设备检索遥测数据。

[AZURE.INCLUDE [iot-suite-provision-remote-monitoring](../../includes/iot-suite-provision-remote-monitoring.md)]

> [AZURE.WARNING]
> 远程监控解决方案在 Azure 订阅中预配了一组 Azure 服务。 部署反映实际企业体系结构。 若要避免产生不必要的 Azure 使用费用，请在使用完预配置解决方案的实例后，在 azureiotsuite.cn 上将其删除。 如果再次需要预配置解决方案，可以轻松地重新创建它。 若要详细了解如何在远程监控解决方案运行时减少消耗，请参阅[出于演示目的配置 Azure IoT 套件预配置解决方案][lnk-demo-config]。

[AZURE.INCLUDE [iot-suite-gateway-kit-view-solution](../../includes/iot-suite-gateway-kit-view-solution.md)]

[AZURE.INCLUDE [iot-suite-gateway-kit-prepare-nuc-connectivity](../../includes/iot-suite-gateway-kit-prepare-nuc-connectivity.md)]

## <a name="configure-bluetooth-connectivity"></a>配置蓝牙连接

在 Intel NUC 上配置蓝牙，使 SensorTag 设备能够建立连接和发送遥测数据。

### <a name="find-the-mac-address-of-the-sensortag"></a>查找 SensorTag 的 MAC 地址

1. 在 Intel NUC 上的 shell 中运行以下命令以取消阻止蓝牙服务：

        sudo rfkill unblock bluetooth

1. 运行以下命令在 Intel NUC 上启动蓝牙服务，然后进入蓝牙 shell：

        sudo systemctl start bluetooth
        bluetoothctl

1. 运行以下命令打开蓝牙控制器：

        power on

    打开控制器后，会看到一条消息“已成功更改电源设置”。

1. 运行以下命令扫描附近的蓝牙设备：

        scan on

1. 按 SensorTag 上的电源按钮，以便能够发现它。 绿色 LED 将会闪烁。

1. 在 shell 中看到一条指出控制器已发现 SensorTag 的消息时，请记下设备的 MAC 地址。 该 MAC 地址类似于 **A0:E6:F8:B5:F6:00**。 稍后在本教程中配置网关时需要用到该 MAC 地址。

1. 运行以下命令关闭蓝牙扫描：

        scan off

1. 运行以下命令，验证是否可以连接到 SensorTag 设备：

        connect <SensorTag MAC address>

    如果连接成功，shell 将显示消息“连接成功”，并列显有关 SensorTag 设备的信息。 如果无法连接，请检查 SensorTag 的电源是否仍已打开。

1. 现在，可通过运行以下命令，从 SensorTag 断开连接并退出蓝牙 shell：

        disconnect
        exit

[AZURE.INCLUDE [iot-suite-gateway-kit-prepare-nuc-software](../../includes/iot-suite-gateway-kit-prepare-nuc-software.md)]

## <a name="build-the-custom-gateway-module"></a>生成自定义网关模块

现在，可以生成自定义网关模块，使网关能够将消息发送到远程监控解决方案。 有关配置网关和网关模块的详细信息，请参阅 [Azure IoT Edge 概念][lnk-gateway-concepts]。

使用以下命令从 GitHub 下载自定义模块的源代码：

    cd ~
    git clone https://github.com/Azure-Samples/iot-remote-monitoring-c-intel-nuc-gateway-getting-started.git

使用以下命令生成自定义模块：

    cd ~/iot-remote-monitoring-c-intel-nuc-gateway-getting-started/basic
    chmod u+x build.sh
    sed -i -e 's/\r$//' build.sh
    ./build.sh

生成脚本将 libsensor2remotemonitoring.so 自定义模块放入 build 文件夹中。

## <a name="configure-and-run-the-gateway"></a>配置并运行网关

现在可以配置网关，将 SensorTag 设备中的遥测数据发送到远程监控仪表板。 有关配置网关和网关模块的详细信息，请参阅 [Azure IoT Edge 概念][lnk-gateway-concepts]。

> [AZURE.TIP]
> 本教程使用 Intel NUC 上的标准 `vi` 文本编辑器。 如果以前未使用过 `vi`，应完成简介教程（例如 [Unix - vi 编辑器教程][lnk-vi-tutorial]）来熟悉此编辑器。 或者，可以使用命令 `smart install nano -y` 安装更友好的 [nano](https://www.nano-editor.org/) 编辑器。

使用以下命令在 **vi** 编辑器中打开示例配置文件：

    vi ~/iot-remote-monitoring-c-intel-nuc-gateway-getting-started/basic/remote_monitoring.json

在 IoTHub 模块的配置中找到以下行：

    "args": {
      "IoTHubName": "<<Azure IoT Hub Name>>",
      "IoTHubSuffix": "<<Azure IoT Hub Suffix>>",
      "Transport": "http"
    }

将占位符值替换为在本教程开始时创建并保存的 IoT 中心信息。 IoTHubName 的值类似于 **yourrmsolution37e08**，而 IoTSuffix 的值通常为 **azure-devices.cn**。

在映射模块的配置中找到以下行：

    args": [
      {
        "macAddress": "<<AA:BB:CC:DD:EE:FF>>",
        "deviceId": "<<Azure IoT Hub Device ID>>",
        "deviceKey": "<<Azure IoT Hub Device Key>>>"
      }
    ]

将 **macAddress** 占位符替换为前面记下的 SensorTag MAC 地址。 将 **deviceID** 和 **deviceKey** 占位符替换为前面在远程监控解决方案中创建的两个设备的 ID 和密钥。

在 SensorTag 模块的配置中找到以下行：

    "args": {
      "controller_index": 0,
      "device_mac_address": "<<AA:BB:CC:DD:EE:FF>>",
      ...
    }

将 **device\_mac\_address** 占位符替换为前面记下的 SensorTag MAC 地址。

保存所做更改。

现在可以使用以下命令运行网关：

    cd ~/iot-remote-monitoring-c-intel-nuc-gateway-getting-started/basic
    /usr/share/azureiotgatewaysdk/samples/ble_gateway/ble_gateway remote_monitoring.json

网关将在 Intel NUC 上启动，并将 SensorTag 中的遥测数据发送到远程监控解决方案：

![网关发送 SensorTag 中的遥测数据][img-telemetry]

随时都可按 **Ctrl-C** 退出程序。

## <a name="view-the-telemetry"></a>查看遥测数据

网关现在正将 SensorTag 设备中的遥测数据发送到远程监控解决方案。 可以在解决方案仪表板上查看遥测数据。 此外，可以在解决方案仪表板中通过通过网关向 SensorTag 设备发送命令。

- 导航到解决方案仪表板。
- 在“要查看的设备”下拉列表中，选择在网关中配置的、代表 SensorTag 的设备。
- SensorTag 设备中的遥测数据将显示在仪表板上。

![显示 SensorTag 设备中的遥测数据][img-telemetry-display]

> [AZURE.WARNING]
> 如果让远程监控解决方案在 Azure 帐户中保持运行状态，系统会按其运行时间计费。 若要详细了解如何在远程监控解决方案运行时减少消耗，请参阅[出于演示目的配置 Azure IoT 套件预配置解决方案][lnk-demo-config]。 请在用完预配置的解决方案后将其从 Azure 帐户中删除。


## <a name="next-steps"></a>后续步骤

有关 Azure IoT 的更多示例和文档，请访问 [Azure IoT 开发人员中心](/develop/iot/)。

[img-telemetry]: ./media/iot-suite-gateway-kit-get-started-sensortag/appoutput.png


[img-telemetry-display]: ./media/iot-suite-gateway-kit-get-started-sensortag/telemetry.png

[lnk-demo-config]: https://github.com/Azure/azure-iot-remote-monitoring/blob/master/Docs/configure-preconfigured-demo.md
[lnk-vi-tutorial]: http://www.tutorialspoint.com/unix/unix-vi-editor.htm
[lnk-sensortag]: http://www.ti.com/ww/en/wireless_connectivity/sensortag/
[lnk-gateway-concepts]: https://docs.microsoft.com/azure/iot-hub/iot-hub-linux-gateway-sdk-get-started