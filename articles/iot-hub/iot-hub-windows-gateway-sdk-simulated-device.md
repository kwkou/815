<properties
    pageTitle="使用 Azure IoT 网关 SDK 模拟设备 (Windows) | Azure"
    description="如何在 Windows 上使用 Azure IoT 网关 SDK 创建模拟设备，从而将遥测数据通过网关发送到 IoT 中心"
    services="iot-hub"
    documentationCenter=""
    author="chipalost"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="6a2aeda0-696a-4732-90e1-595d2e2fadc6"
    ms.service="iot-hub"
    ms.devlang="cpp"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="03/29/2017"
    wacn.date="05/15/2017"
    ms.author="andbuc"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="457fc748a9a2d66d7a2906b988e127b09ee11e18"
    ms.openlocfilehash="5cb04f3d6da77ab42404d9fca9f6ed1edbd8473b"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/05/2017" />

# <a name="use-the-azure-iot-gateway-sdk-to-send-device-to-cloud-messages-with-a-simulated-device-windows"></a>使用 Azure IoT 网关 SDK，通过模拟设备发送设备到云消息 (Windows)

[AZURE.INCLUDE [iot-hub-gateway-sdk-simulated-selector](../../includes/iot-hub-gateway-sdk-simulated-selector.md)]

## <a name="build-and-run-the-sample"></a>生成并运行示例

开始之前，必须：

- [设置开发环境][lnk-setupdevbox]，以便在 Windows 上使用 SDK。
- 若要在 Azure 订阅中[创建 IoT 中心][lnk-create-hub]，需要使用中心的名称来完成本演练。 如果没有帐户，只需花费几分钟就能创建一个[帐户][lnk-free-trial]。
- 将两个设备添加到 IoT 中心，并记下其 ID 和设备密钥。 可使用[设备资源管理器][lnk-device-explorer]或 [iothub-explorer][lnk-iothub-explorer] 工具将设备添加到在上一步中创建的 IoT 中心，并检索其密钥。

生成示例：

1. 打开“VS 2015 开发人员命令提示”或“VS 2017 开发人员命令提示”命令提示符。
2. 浏览到本地 **azure-iot-gateway-sdk** 存储库副本中的根文件夹。
3. 运行 **tools\\build.cmd** 脚本。 此脚本创建 Visual Studio 解决方案文件并生成解决方案。 你可以在本地 **azure-iot-gateway-sdk** 存储库副本的 **build** 文件夹中找到 Visual Studio 解决方案。 可为脚本提供其他参数，用于生成和运行单元测试和端到端测试。 这些参数分别是 **--run-unittests** 和 **--run-e2e-tests**。

运行示例：

在文本编辑器中，打开本地 **azure-iot-gateway-sdk** 存储库副本中的文件 **samples\\simulated_device_cloud_upload\\src\\simulated_device_cloud_upload_win.json**。 此文件配置示例网关中的模块：

* **IoTHub** 模块连接到 IoT 中心。 将该模块配置为将数据发送到 IoT 中心。 具体而言，将 **IoTHubName** 值设置为 IoT 中心的名称，将 **IoTHubSuffix** 值设置为 **azure-devices.net**。 将“传输”值设置为“HTTP”、“AMQP”或“MQTT”其中的一个。 目前只有“HTTP”会针对所有设备消息共享一个 TCP 连接。 如果将值设置为“AMQP”或“MQTT”，则网关将为每个设备维护与 IoT 中心的单独 TCP 连接。
* **mapping** 模块将模拟设备的 MAC 地址映射到 IoT 中心设备 ID。 确保 **deviceId** 值与添加到 IoT 中心的两台设备的 ID 一致，确保 **deviceKey** 值包含两台设备的密钥。
* **BLE1** 和 **BLE2** 模块是模拟设备。 注意模块 MAC 地址如何与“映射”模块中的地址匹配。
* **Logger** 模块将网关活动记录到一个文件中。
* 以下示例中显示的 **module path** 值假定已将 IoT 网关 SDK 存储库克隆到 **C:** 驱动器的根目录。 如果将该存储库下载到其他位置，则需要相应地调整 **module path** 值。
* JSON 文件底部的 **links** 数组将 **BLE1** 和 **BLE2** 模块连接到 **mapping** 模块，并将 **mapping** 模块连接到 **IoTHub** 模块。 它还确保 **Logger** 模块记录所有消息。

        {
            "modules" :
            [
              {
                "name": "IotHub",
                "loader": {
                  "name": "native",
                  "entrypoint": {
                    "module.path": "..\\..\\..\\modules\\iothub\\Debug\\iothub.dll"
                  }
                  },
                  "args": {
                    "IoTHubName": "<<insert here IoTHubName>>",
                    "IoTHubSuffix": "<<insert here IoTHubSuffix>>",
                    "Transport": "HTTP"
                  }
                },
              {
                "name": "mapping",
                "loader": {
                  "name": "native",
                  "entrypoint": {
                    "module.path": "..\\..\\..\\modules\\identitymap\\Debug\\identity_map.dll"
                  }
                  },
                  "args": [
                    {
                      "macAddress": "01:01:01:01:01:01",
                      "deviceId": "<<insert here deviceId>>",
                      "deviceKey": "<<insert here deviceKey>>"
                    },
                    {
                      "macAddress": "02:02:02:02:02:02",
                      "deviceId": "<<insert here deviceId>>",
                      "deviceKey": "<<insert here deviceKey>>"
                    }
                  ]
                },
              {
                "name": "BLE1",
                "loader": {
                  "name": "native",
                  "entrypoint": {
                    "module.path": "..\\..\\..\\modules\\simulated_device\\Debug\\simulated_device.dll"
                  }
                  },
                  "args": {
                    "macAddress": "01:01:01:01:01:01"
                  }
                },
              {
                "name": "BLE2",
                "loader": {
                  "name": "native",
                  "entrypoint": {
                    "module.path": "..\\..\\..\\modules\\simulated_device\\Debug\\simulated_device.dll"
                  }
                  },
                  "args": {
                    "macAddress": "02:02:02:02:02:02"
                  }
                },
              {
                "name": "Logger",
                "loader": {
                  "name": "native",
                  "entrypoint": {
                    "module.path": "..\\..\\..\\modules\\logger\\Debug\\logger.dll"
                  }
                },
                "args": {
                  "filename": "deviceCloudUploadGatewaylog.log"
                }
              }
            ],
            "links" : [
                { "source" : "*", "sink" : "Logger" },
                { "source" : "BLE1", "sink" : "mapping" },
                { "source" : "BLE2", "sink" : "mapping" },
                { "source" : "mapping", "sink" : "IotHub" }
            ]
        }


保存对配置文件所做的任何更改。

运行示例：

1. 在命令提示符下，浏览到本地 **azure-iot-gateway-sdk** 存储库副本中的根文件夹。
2. 运行以下命令：
   
    
        build\samples\simulated_device_cloud_upload\Debug\simulated_device_cloud_upload_sample.exe samples\simulated_device_cloud_upload\src\simulated_device_cloud_upload_win.json

3. 可使用[设备资源管理器][lnk-device-explorer]或 [iothub-explorer][lnk-iothub-explorer] 工具监视 IoT 中心从网关接收的消息。

## <a name="next-steps"></a>后续步骤
如果想要深入了解 IoT 网关 SDK 并尝试一些代码示例，请访问以下开发人员教程和资源：

- [使用 IoT 网关 SDK 从物理设备发送设备到云的消息][lnk-physical-device]
- [Azure IoT 网关 SDK][lnk-gateway-sdk]

若要进一步探索 IoT 中心的功能，请参阅：

- [IoT 中心开发人员指南][lnk-devguide]
- [从根本上保护 IoT 解决方案][lnk-securing]

<!-- Links -->

[lnk-setupdevbox]: https://github.com/Azure/azure-iot-gateway-sdk/blob/master/doc/devbox_setup.md
[lnk-free-trial]: /pricing/1rmb-trial/
[lnk-device-explorer]: https://github.com/Azure/azure-iot-sdk-csharp/tree/master/tools/DeviceExplorer
[lnk-iothub-explorer]: https://github.com/Azure/iothub-explorer/blob/master/readme.md
[lnk-gateway-sdk]: https://github.com/Azure/azure-iot-gateway-sdk/

[lnk-physical-device]: /documentation/articles/iot-hub-gateway-sdk-physical-device/

[lnk-devguide]: /documentation/articles/iot-hub-devguide/
[lnk-create-hub]: /documentation/articles/iot-hub-create-through-portal/
[lnk-securing]: /documentation/articles/iot-hub-security-ground-up/

<!---HONumber=Mooncake_0306_2017-->
<!--Update_Description:update wording-->