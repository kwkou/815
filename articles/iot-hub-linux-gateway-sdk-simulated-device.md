<properties
	pageTitle="使用网关 SDK 模拟设备 | Azure"
	description="使用 Linux 的 Azure IoT 中心网关 SDK 演练，说明如何使用 Azure IoT 中心网关 SDK 从模拟设备发送遥测数据。"
	services="iot-hub"
	documentationCenter=""
	authors="chipalost"
	manager="timlt"
	editor=""/>

<tags
     ms.service="iot-hub"
     ms.date="04/20/2016"
     wacn.date="07/04/2016"/>


# IoT 网关 SDK（Beta 版）– 使用 Linux 通过模拟设备发送设备至云消息

[AZURE.INCLUDE [iot-hub-gateway-sdk-simulated-selector](../includes/iot-hub-gateway-sdk-simulated-selector.md)]

## 生成并运行示例

开始之前，必须：

- [设置开发环境][lnk-setupdevbox]，以便在 Linux 上使用 SDK。
- 在 Azure 订阅中创建 IoT 中心，将需要中心的名称来完成此演练。如果还没有 Azure 订阅，可以获取一个[帐户][lnk-free-trial]。
- 将两个设备添加到 IoT 中心，并记下其 ID 和设备密钥。可使用[设备资源管理器或 iothub-explorer][lnk-explorer-tools] 工具来将设备添加到在上一步中创建的 IoT 中心，并检索其密钥。

生成示例：

1. 打开 shell。
2. 浏览到本地 **azure-iot-gateway-sdk** 存储库副本中的根文件夹。
3. 运行 **tools/build.sh** 脚本。此脚本使用 **cmake** 实用工具在本地 **azure-iot-gateway-sdk** 存储库副本的根文件夹中创建一个名为 **build** 的文件夹，并生成一个生成文件。然后，该脚本将生成解决方案并运行测试。

> [AZURE.NOTE]  每次运行 **build.sh** 脚本时，都会删除本地 **azure-iot-gateway-sdk** 存储库副本的根文件夹中的 **build** 文件夹并重新生成。

运行示例：

在文本编辑器中，打开本地 **azure-iot-gateway-sdk** 存储库副本中的文件 **samples/simulated\_device\_cloud\_upload/src/simulated\_device\_cloud\_upload\_lin.json**。此文件配置示例网关中的模块：

- **IoTHub** 模块连接到 IoT 中心。必须配置该模块才能将数据发送到 IoT 中心。具体而言，将 **IoTHubName** 值设置为 IoT 中心的名称，将 **IoTHubSuffix** 值设置为 **azure-devices.cn**。
- **mapping** 模块将模拟设备的 MAC 地址映射到 IoT 中心设备 ID。确保 **deviceId** 值与添加到 IoT 中心的两台设备的 ID 一致，确保 **deviceKey** 值包含两台设备的密钥。
- **BLE1** 和 **BLE2** 模块是模拟设备。注意其 MAC 地址是否与 **mapping** 模块中的地址一致。
- **Logger** 模块将网关活动记录到一个文件中。
- 如下所示的 **module path** 值假定你将从本地 **azure-iot-gateway-sdk** 存储库副本的根文件夹运行示例。

```
{
    "modules" :
    [ 
        {
            "module name" : "IoTHub",
            "module path" : "./build/modules/iothubhttp/libiothubhttp_hl.so",
            "args" : 
            {
                "IoTHubName" : "{Your IoT hub name}",
                "IoTHubSuffix" : "azure-devices.cn"
            }
        },
        {
            "module name" : "mapping",
            "module path" : "./build/modules/identitymap/libidentitymap_hl.so",
            "args" : 
            [
                {
                    "macAddress" : "01-01-01-01-01-01",
                    "deviceId"   : "GW-ble1-demo",
                    "deviceKey"  : "{Device key}"
                },
                {
                    "macAddress" : "02-02-02-02-02-02",
                    "deviceId"   : "GW-ble2-demo",
                    "deviceKey"  : "{Device key}"
                }
            ]
        },
        {
            "module name":"BLE1",
            "module path" : "./build/modules/simulated_device/libsimulated_device_hl.so",
            "args":
            {
                "macAddress" : "01-01-01-01-01-01"
            }
        },
        {
            "module name":"BLE2",
            "module path" : "./build/modules/simulated_device/libsimulated_device_hl.so",
            "args":
            {
                "macAddress" : "02-02-02-02-02-02"
            }
        },
        {
            "module name":"Logger",
            "module path" : "./build/modules/logger/liblogger_hl.so",
            "args":
            {
                "filename":"./deviceCloudUploadGatewaylog.log"
            }
        }
    ]
}

```

保存对配置文件所做的任何更改。

运行示例：

1. 在 shell 中，浏览到本地 **azure-iot-gateway-sdk** 存储库副本中的根文件夹。
2. 运行以下命令：

    ```
    ./build/samples/simulated_device_cloud_upload/simulated_device_cloud_upload_sample ./samples/simulated_device_cloud_upload/src/simulated_device_cloud_upload_lin.json
    ```

3. 可使用[设备资源管理器或 iothub-explorer][lnk-explorer-tools] 工具来监视 IoT 中心从网关接收的消息。

## 后续步骤

若要了解如何使用网关 SDK，请参阅 GitHub 上的 [Azure IoT 网关 SDK][lnk-gateway-sdk]。

<!-- Links -->
[lnk-setupdevbox]: https://github.com/Azure/azure-iot-gateway-sdk/blob/master/doc/devbox_setup.md
[lnk-create-hub]: /documentation/articles/iot-hub-manage-through-portal/
[lnk-free-trial]: /pricing/1rmb-trial/
[lnk-explorer-tools]: https://github.com/Azure/azure-iot-sdks/blob/master/doc/manage_iot_hub.md
[lnk-gateway-sdk]: https://github.com/Azure/azure-iot-gateway-sdk/




<!---HONumber=Mooncake_0523_2016-->