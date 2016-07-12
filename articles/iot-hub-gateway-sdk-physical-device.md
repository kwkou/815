<properties
	pageTitle="通过网关 SDK 使用实际设备 | Azure"
	description="Azure IoT 中心网关 SDK 演练使用 Texas Instruments SensorTag 通过 Intel Edison 计算模块上运行的网关将数据发送到 IoT 中心"
	services="iot-hub"
	documentationCenter=""
	authors="chipalost"
	manager="timlt"
	editor=""/>

<tags
     ms.service="iot-hub"
     ms.date="05/31/2016"
     wacn.date="07/04/2016"/>


# IoT 网关 SDK（Beta 版）– 使用 Linux 通过实际设备发送设备至云消息

本演练的[蓝牙低功耗示例][lnk-ble-samplecode]演示如何使用 [Azure IoT 网关 SDK][lnk-sdk] 从物理设备将设备到云遥测转发到 IoT 中心以及如何从 IoT 中心将命令路由到物理设备。

本文介绍的内容包括：

* **体系结构**：与蓝牙低功耗示例有关的重要体系结构信息。

* **生成并运行**：生成并运行示例所需的步骤。

## 体系结构

本演练演示如何在运行 Linux 的 Intel Edison 计算模块上生成和运行 IoT 网关。该网关是使用 IoT 网关 SDK 生成的。此示例使用 Texas Instruments SensorTag 蓝牙低功耗 (BLE) 设备收集温度数据。

当你运行网关时，它：

- 使用蓝牙低功耗 (BLE) 协议连接到 SensorTag 设备。
- 使用 AMQP 协议连接到 IoT 中心。
- 从 SensorTag 设备将遥测转发到 IoT 中心。
- 从 IoT 中心将命令路由到 SensorTag 设备。

该网关包含以下模块：

- BLE 模块，与 BLE 设备相连接，从设备接收温度数据并将命令发送到设备。
- 记录器模块，用于生成消息总线诊断。
- 标识映射模块，用于在 BLE 设备 MAC 地址和 Azure IoT 中心设备标识之间进行转换。
- IoT 中心 HTTP 模块，用于将遥测数据上载到 IoT 中心并接收来自 IoT 中心的设备命令。
- BLE 打印机模块，用于解释 BLE 设备的遥测，并将格式化数据输出到控制台，以启用故障排除和调试。

### 数据如何流经网关

以下块图说明了遥测上载数据流管道：

![](./media/iot-hub-gateway-sdk-physical-device/gateway_ble_upload_data_flow.png)

通过以下步骤将遥测项从 BLE 设备传输到 IoT 中心：

1. BLE 设备生成温度样本并将其通过蓝牙发送到网关的 BLE 模块。
2. BLE 模块接收该样本，并将其与设备的 MAC 地址一起发布到消息总线。
3. 标识映射模块从消息总线提取此消息，并使用内部表将设备的 MAC 地址转换为 IoT 中心设备标识（设备 ID 和设备密钥）。然后，它将新消息（包含温度样本数据、设备的 MAC 地址、设备 ID 和设备密钥）发布到消息总线。
4. IoT 中心 HTTP 模块从消息总线接收此新消息（由标识映射模块生成），并将其发布到 IoT 中心。
5. 记录器模块将消息总线中的所有消息记录到磁盘文件中。

以下块图说明了设备命令数据流管道：

![](./media/iot-hub-gateway-sdk-physical-device/gateway_ble_command_data_flow.png)

1. IoT 中心 HTTP 模块会定期在 IoT 中心中轮询新的命令消息。
2. 当 IoT 中心 HTTP 模块收到新的命令消息时，它会将其发布到消息总线。
3. 标识映射模块从消息总线提取该命令消息，并使用内部表将 IoT 中心设备 ID 转换为设备 MAC 地址。然后，它将新消息（在消息的属性映射中包括目标设备的 MAC 地址）发布到消息总线。
4. BLE 模块通过 BLE 设备进行通信提取该消息并执行 I/O 指令。
5. 记录器模块将消息总线中的所有消息记录到磁盘文件中。

## 准备硬件

本教程假定你使用的是已连接到 Intel Edison 板的 [Texas Instruments SensorTag](http://www.ti.com/ww/en/wireless_connectivity/sensortag2015/index.html) 设备。

### 设置 Edison 板

在开始之前，你应确保可以将 Edison 设备连接到无线网络。若要设置 Edison 设备，需要将其连接到主计算机。Intel 针对以下操作系统提供了入门指南：

- [Get Started with the Intel Edison Development Board on Windows 64-bit（Windows 64 位上的 Intel Edison 开发板入门）][lnk-setup-win64]。
- [Get Started with the Intel Edison Development Board on Windows 32-bit（Windows 32 位上的 Intel Edison 开发板入门）][lnk-setup-win32]。
- [Get Started with the Intel Edison Development Board on Mac OS X（Mac OS X 上的 Intel Edison 开发板入门）][lnk-setup-osx]。
- [Getting Started with the Intel® Edison Board on Linux（Linux 上的 Intel® Edison 板入门）][lnk-setup-linux]。

若要设置 Edison 设备并熟悉它，应完成所有这些“入门”文章中除最后一步外的所有步骤，最后一步“选择 IDE”不是当前教程所必需的。在 Edison 设置过程结束时，你应已：

- 使用最新固件刷写 Edison。
- 建立从主机到 Edison 的串行连接。
- 运行 **configure\_edison** 脚本以设置密码，并在 Edison 上启用 WiFi。

### 从 Edison 板启用到 SensorTag 设备的连接

运行示例之前，需要确认 Edison 板可以连接到 SensorTag 设备。

首先，需要更新 Edison 上 BlueZ 软件的版本。请注意，即使你已安装 5.37 版本，也应完成以下步骤以确保安装已完成：

1. 停止当前正在运行的蓝牙守护程序。
    
    ```
    systemctl stop bluetooth
    ```

2. 下载并解压缩 BlueZ 版本 5.37 的[源代码](http://www.kernel.org/pub/linux/bluetooth/bluez-5.37.tar.xz)。
    
    ```
    wget http://www.kernel.org/pub/linux/bluetooth/bluez-5.37.tar.xz
    tar -xvf bluez-5.37.tar.xz
    cd bluez-5.37
    ```

3. 生成并安装 BlueZ。
    
    ```
    ./configure --disable-udev --disable-systemd --enable-experimental
    make
    make install
    ```

4. 通过编辑文件 **/lib/systemd/system/bluetooth.service** 更改蓝牙的 *systemd* 服务配置，使其指向新的蓝牙守护程序。替换 **ExecStart** 属性的值，使其如下所示：
    
    ```
    ExecStart=/usr/local/libexec/bluetooth/bluetoothd -E
    ```

5. 重新启动 Edison。

接下来，需要确认 Edison 可以连接到 SensorTag 设备。

1. 在 Edison 上取消阻止蓝牙，并检查版本号是否为 **5.37**。
    
    ```
    rfkill unblock bluetooth
    bluetoothctl --version
    ```

2. 执行 **bluetoothctl** 命令。你应看到如下输出：
    
    ```
    [NEW] Controller 98:4F:EE:04:1F:DF edison [default]
    ```

3. 现在，你已在交互式蓝牙外壳程序中。输入命令 **scan on** 以扫描蓝牙设备。你应看到如下输出：
    
    ```
    Discovery started
    [CHG] Controller 98:4F:EE:04:1F:DF Discovering: yes
    ```

4. 通过按小按钮（绿色 LED 应闪烁）使 SensorTag 设备可检测到。Edison 应发现 SensorTag 设备：
    
    ```
    [NEW] Device A0:E6:F8:B5:F6:00 CC2650 SensorTag
    [CHG] Device A0:E6:F8:B5:F6:00 TxPower: 0
    [CHG] Device A0:E6:F8:B5:F6:00 RSSI: -43
    ```
    
    在此示例中，你可以看到 SensorTag 设备的 MAC 地址为 **A0:E6:F8:B5:F6:00**。

5. 输入 **scan off** 命令以关闭扫描。
    
    ```
    [CHG] Controller 98:4F:EE:04:1F:DF Discovering: no
    Discovery stopped
    ```

6. 通过输入 **connect <MAC address>** 使用其 MAC 地址连接到 SensorTag 设备。请注意，下面的示例输出已节略：
    
    ```
    Attempting to connect to A0:E6:F8:B5:F6:00
    [CHG] Device A0:E6:F8:B5:F6:00 Connected: yes
    Connection successful
    [CHG] Device A0:E6:F8:B5:F6:00 UUIDs: 00001800-0000-1000-8000-00805f9b34fb
    ...
    [NEW] Primary Service
            /org/bluez/hci0/dev_A0_E6_F8_B5_F6_00/service000c
            Device Information
    ...
    [CHG] Device A0:E6:F8:B5:F6:00 GattServices: /org/bluez/hci0/dev_A0_E6_F8_B5_F6_00/service000c
    ...
    [CHG] Device A0:E6:F8:B5:F6:00 Name: SensorTag 2.0
    [CHG] Device A0:E6:F8:B5:F6:00 Alias: SensorTag 2.0
    [CHG] Device A0:E6:F8:B5:F6:00 Modalias: bluetooth:v000Dp0000d0110
    ```
    
    注意：你可以使用 **list-attributes** 命令重新列出设备的 GATT 特征。

7. 现在可以使用 **disconnect** 命令与设备断开连接，然后使用 **quit** 命令退出蓝牙外壳程序：
    
    ```
    Attempting to disconnect from A0:E6:F8:B5:F6:00
    Successful disconnected
    [CHG] Device A0:E6:F8:B5:F6:00 Connected: no
    ```

你现在可以在 Edison 设备上运行 BLE 网关示例。

## 运行 BLE 网关示例

若要在 Edison 上运行 BLE 示例，需要完成三个任务：

- 在 IoT 中心中配置两个示例设备。
- 在 Edison 设备上生成网关 SDK。
- 在 Edison 设备上配置和运行 BLE 示例。

编写本文时，网关 SDK 仅支持在 Linux 上使用 BLE 模块的网关。

### 在 IoT 中心中配置两个示例设备

- 在 Azure 订阅中[创建 IoT 中心][lnk-create-hub]，你需要中心的名称才能完成此演练。如果还没有 Azure 订阅，可以获取一个[试用帐户][lnk-free-trial]。
- 将一台名为 **SensorTag\_01** 的设备添加到 IoT 中心，并记下其 ID 和设备密钥。可使用[设备资源管理器或 iothub-explorer][lnk-explorer-tools] 工具将此设备添加到在上一步中创建的 IoT 中心，并检索其密钥。配置网关时，可将此设备映射到 SensorTag 设备。

### 在 Edison 设备上生成网关 SDK

Edsion 上的 **git** 版本不支持子模块。若要将网关 SDK 的完整源代码下载到 Edison，有两个选择：

- 选项 #1：在 Edison 上克隆 [Azure IoT 网关 SDK][lnk-sdk]，然后手动克隆每个子模块的存储库。
- 选项 #2：在其中 **git** 支持子模块的桌面设备上克隆 [Azure IoT 网关 SDK][lnk-sdk] 存储库，然后将包含子模块的整个存储库复制到 Edison 中。

如果你选择选项 #2，请使用以下 **git** 命令克隆网关 SDK 及其所有子模块：

```
git clone --recursive https://github.com/Azure/azure-iot-gateway-sdk.git 
git submodule update --init --recursive
```

然后应将整个本地存储库压缩成单个存档文件，然后再将其复制到 Edison。可以使用实用程序（如 **Putty** 附带的 **pscp**）将存档文件复制到 Edison。例如：

```
pscp .\gatewaysdk.zip root@192.168.0.45:/home/root
```

当 Edison 上有了网关 SDK 存储库的完整副本时，可以从包含该 SDK 的文件夹使用以下命令生成它：

```
./tools/build.sh
```

### 在 Edison 设备上配置和运行 BLE 示例

若要启动和运行示例，需要配置参与网关的每个模块。在 JSON 文件中提供了此配置，你需要配置所有五个参与模块。存储库中提供了一个名为 **gateway\_sample.json** 的示例 JSON 文件，你可以使用它作为生成自己的配置文件的起点。此文件位于网关 SDK 存储库的本地副本中的 **samples/ble\_gateway\_hl/src** 文件夹。

下列各节描述如何为 BLE 示例编辑此配置文件，并假设网关 SDK 存储库位于 Edison 设备上的 **/home/root/azure-iot-gateway-sdk/** 文件夹中。如果存储库在其他位置，则应相应地调整路径：


#### 记录器配置

假定网关存储库位于 **/home/root/azure-iot-gateway-sdk/** 文件夹中，请按如下所示配置记录器模块：

```json
{
    "module name": "logger",
    "module path": "/home/root/azure-iot-gateway-sdk/build/modules/logger/liblogger_hl.so",
    "args":
    {
        "filename":"/home/root/gw_logger.log"
    }
}
```

#### BLE 模块配置

BLE 设备的示例配置假定使用 Texas Instruments SensorTag 设备。任何可以作为 GATT 外围设备运行的标准 BLE 设备都应可以使用，但你需要更新 GATT 特征 ID 和数据（用于编写说明）。添加 SensorTag 设备的 MAC 地址：

```json
{
  "module name": "SensorTag",
  "module path": "/home/root/azure-iot-gateway-sdk/build/modules/ble/libble_hl.so",
  "args": {
    "controller_index": 0,
    "device_mac_address": "<<AA:BB:CC:DD:EE:FF>>",
    "instructions": [
      {
        "type": "read_once",
        "characteristic_uuid": "00002A24-0000-1000-8000-00805F9B34FB"
      },
      {
        "type": "read_once",
        "characteristic_uuid": "00002A25-0000-1000-8000-00805F9B34FB"
      },
      {
        "type": "read_once",
        "characteristic_uuid": "00002A26-0000-1000-8000-00805F9B34FB"
      },
      {
        "type": "read_once",
        "characteristic_uuid": "00002A27-0000-1000-8000-00805F9B34FB"
      },
      {
        "type": "read_once",
        "characteristic_uuid": "00002A28-0000-1000-8000-00805F9B34FB"
      },
      {
        "type": "read_once",
        "characteristic_uuid": "00002A29-0000-1000-8000-00805F9B34FB"
      },
      {
        "type": "write_at_init",
        "characteristic_uuid": "F000AA02-0451-4000-B000-000000000000",
        "data": "AQ=="
      },
      {
        "type": "read_periodic",
        "characteristic_uuid": "F000AA01-0451-4000-B000-000000000000",
        "interval_in_ms": 1000
      },
      {
        "type": "write_at_exit",
        "characteristic_uuid": "F000AA02-0451-4000-B000-000000000000",
        "data": "AA=="
      }
    ]
  }
}
```

#### IoT 中心 HTTP 模块

添加 IoT 中心的名称。后缀值通常是 **azure-devices.cn**：

```json
{
  "module name": "IoTHub",
  "module path": "/home/root/azure-iot-gateway-sdk/build/modules/iothubhttp/libiothubhttp_hl.so",
  "args": {
    "IoTHubName": "<<Azure IoT Hub Name>>",
    "IoTHubSuffix": "<<Azure IoT Hub Suffix>>"
  }
}
```

#### 标识映射模块配置

添加已添加到 IoT 中心的 SensorTag 设备的 MAC 地址和 **SensorTag\_01** 设备的设备 ID 和密钥：

```json
{
  "module name": "mapping",
  "module path": "/home/root/azure-iot-gateway-sdk/build/modules/identitymap/libidentity_map_hl.so",
  "args": [
    {
      "macAddress": "<<AA:BB:CC:DD:EE:FF>>",
      "deviceId": "<<Azure IoT Hub Device ID>>",
      "deviceKey": "<<Azure IoT Hub Device Key>>"
    }
  ]
}
```

#### BLE 打印机模块配置

```json
{
    "module name": "BLE Printer",
    "module path": "/home/root/azure-iot-gateway-sdk/build/samples/ble_gateway_hl/ble_printer/libble_printer.so",
    "args": null
}
```

若要运行示例，需运行 **ble\_gateway\_hl** 二进制文件，以将路径传递给 JSON 配置文件。如果已使用 **gateway\_sample.json** 文件，则要执行的命令如下所示：

```
./build/samples/ble_gateway_hl/ble_gateway_hl ./samples/ble_gateway_hl/src/gateway_sample.json
```

在运行示例之前，可能需要按 SensorTag 上的小按钮，使其可被发现。

运行示例时，可以使用[设备资源管理器或 iothub-explorer][lnk-explorer-tools] 工具监视网关从 SensorTag 设备转发的消息。

## 发送云到设备的消息

BLE 模块还支持从 Azure IoT 中心将指令发送到设备。可以使用 [Azure IoT 中心设备资源管理器](https://github.com/Azure/azure-iot-sdks/blob/master/tools/DeviceExplorer/doc/how_to_use_device_explorer.md)或 [IoT 中心资源管理器] https://github.com/Azure/azure-iot-sdks/tree/master/tools/iothub-explorer 将 BLE 网关模块传递的 JSON 消息发送到 BLE 设备。例如，如果你使用的是 Texas Instruments SensorTag 设备，则可以从 IoT 中心将以下 JSON 消息发送到设备。

- 重置所有 LED 和蜂鸣器（将它们关闭）

    ```json
    {
      "type": "write_once",
      "characteristic_uuid": "F000AA65-0451-4000-B000-000000000000",
      "data": "AA=="
    }
    ```

- 将 I/O 配置为“远程”

    ```json
    {
      "type": "write_once",
      "characteristic_uuid": "F000AA66-0451-4000-B000-000000000000",
      "data": "AQ=="
    }
    ```

- 打开红色 LED

    ```json
    {
      "type": "write_once",
      "characteristic_uuid": "F000AA65-0451-4000-B000-000000000000",
      "data": "AQ=="
    }
    ```

- 打开绿色 LED

    ```json
    {
      "type": "write_once",
      "characteristic_uuid": "F000AA65-0451-4000-B000-000000000000",
      "data": "Ag=="
    }
    ```

- 打开蜂鸣器

    ```json
    {
      "type": "write_once",
      "characteristic_uuid": "F000AA65-0451-4000-B000-000000000000",
      "data": "BA=="
    }
    ```

使用 HTTP 协议连接到 IoT 中心的设备的默认行为是每隔 25 分钟检查一次新命令。因此，如果你发送多条单独的命令，则设备接收每条命令时，你都需要等待 25 分钟。

> [AZURE.NOTE] 每次网关启动时，网关也会检查新命令，因此你可以停止并启动网关以强制它处理命令。

## 后续步骤

有关详细信息，请参阅 [Azure IoT 网关 SDK][lnk-sdk]。

<!-- Links -->
[lnk-ble-samplecode]: https://github.com/Azure/azure-iot-gateway-sdk/blob/master/samples/ble_gateway_hl
[lnk-setupdevbox]: https://github.com/Azure/azure-iot-gateway-sdk/blob/master/doc/devbox_setup.md
[lnk-create-hub]: /documentation/articles/iot-hub-manage-through-portal/
[lnk-free-trial]: /pricing/1rmb-trial/
[lnk-explorer-tools]: https://github.com/Azure/azure-iot-sdks/blob/master/doc/manage_iot_hub.md
[lnk-gateway-sdk]: https://github.com/Azure/azure-iot-gateway-sdk/
[lnk-setup-win64]: https://software.intel.com/get-started-edison-windows
[lnk-setup-win32]: https://software.intel.com/get-started-edison-windows-32
[lnk-setup-osx]: https://software.intel.com/get-started-edison-osx
[lnk-setup-linux]: https://software.intel.com/get-started-edison-linux
[lnk-sdk]: https://github.com/Azure/azure-iot-gateway-sdk/
<!---HONumber=Mooncake_0627_2016-->