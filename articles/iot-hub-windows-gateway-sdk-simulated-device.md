<properties
	pageTitle="使用网关 SDK 模拟设备 | Azure"
	description="使用 Windows 的 Azure IoT 中心网关 SDK 演练，说明如何使用 Azure IoT 中心网关 SDK 从模拟设备发送遥测数据。"
	services="iot-hub"
	documentationCenter=""
	authors="chipalost"
	manager="timlt"
	editor=""/>

<tags
     ms.service="iot-hub"
     ms.date="04/20/2016"
     wacn.date="07/04/2016"/>


# IoT 网关 SDK（Beta 版）– 使用 Windows 通过模拟设备发送设备至云消息

[AZURE.INCLUDE [iot-hub-gateway-sdk-simulated-selector](../includes/iot-hub-gateway-sdk-simulated-selector.md)]

## 生成并运行示例

开始之前，必须：

- [设置开发环境][lnk-setupdevbox]，以便在 Windows 上使用 SDK。
- 在 Azure 订阅中[创建 IoT 中心][lnk-create-hub]，将需要中心的名称来完成此演练。如果还没有 Azure 订阅，可以获取一个[帐户][lnk-free-trial]。
- 将两个设备添加到 IoT 中心，并记下其 ID 和设备密钥。可使用[设备资源管理器或 iothub-explorer][lnk-explorer-tools] 工具来将设备添加到在上一步中创建的 IoT 中心，并检索其密钥。

生成示例：

1. 打开 **VS2015 开发人员命令提示**命令提示符。
2. 浏览到本地 **azure-iot-gateway-sdk** 存储库副本中的根文件夹。
3. 运行 **tools\\build.cmd** 脚本。此脚本创建 Visual Studio 解决方案文件，生成解决方案，并运行测试。你可以在本地 **azure-iot-gateway-sdk** 存储库副本的 **build** 文件夹中找到 Visual Studio 解决方案。

运行示例：

在文本编辑器中，打开本地 **azure-iot-gateway-sdk** 存储库副本中的文件 **samples\\simulated\_device\_cloud\_upload\\src\\simulated\_device\_cloud\_upload\_win.json**。此文件配置示例网关中的模块：

- **IoTHub** 模块连接到 IoT 中心。必须配置该模块才能将数据发送到 IoT 中心。具体而言，将 **IoTHubName** 值设置为 IoT 中心的名称，将 **IoTHubSuffix** 值设置为 **azure-devices.cn**。
- **mapping** 模块将模拟设备的 MAC 地址映射到 IoT 中心设备 ID。确保 **deviceId** 值与添加到 IoT 中心的两台设备的 ID 一致，确保 **deviceKey** 值包含两台设备的密钥。
- **BLE1** 和 **BLE2** 模块是模拟设备。注意其 MAC 地址是否与 **mapping** 模块中的地址一致。
- **Logger** 模块将网关活动记录到一个文件中。
- 如下所示的 **module path** 值假定你已将网关 SDK 存储库克隆到了 **C:** 盘的根目录。如果将该存储库下载到其他位置，则需要相应地调整 **module path** 值。

```
{
    "modules" :
    [ 
        {
            "module name" : "IoTHub",
            "module path" : "C:\\azure-iot-gateway-sdk\\modules\\iothubhttp\\Debug\\iothubhttp_hl.dll",
            "args" : 
            {
                "IoTHubName" : "{Your IoT hub name}",
                "IoTHubSuffix" : "azure-devices.cn"
            }
        },
        {
            "module name" : "mapping",
            "module path" : "C:\\azure-iot-gateway-sdk\\modules\\identitymap\\Debug\\identitymap_hl.dll",
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
            "module path" : "C:\\azure-iot-gateway-sdk\\modules\\ble_fake\\Debug\\ble_fake_hl.dll",
            "args":
            {
                "macAddress" : "01-01-01-01-01-01"
            }
        },
        {
            "module name":"BLE2",
            "module path" : "C:\\azure-iot-gateway-sdk\\modules\\ble_fake\\Debug\\ble_fake_hl.dll",
            "args":
            {
                "macAddress" : "02-02-02-02-02-02"
            }
        },
        {
            "module name":"Logger",
            "module path" : "C:\\azure-iot-gateway-sdk\\modules\\logger\\Debug\\logger_hl.dll",
            "args":
            {
                "filename":"C:\\azure-iot-gateway-sdk\\deviceCloudUploadGatewaylog.log"
            }
        }
    ]
}
```

保存对配置文件所做的任何更改。

运行示例：

1. 在命令提示符下，浏览到本地 **azure-iot-gateway-sdk** 存储库副本中的根文件夹。
2. 运行以下命令：
  
    ```
    build\samples\simulated_device_cloud_upload\Debug\simulated_device_cloud_upload_sample.exe samples\simulated_device_cloud_upload\src\simulated_device_cloud_upload_win.json
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