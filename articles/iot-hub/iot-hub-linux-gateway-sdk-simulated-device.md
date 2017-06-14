<properties
    pageTitle="使用 Azure IoT Edge 模拟设备 (Linux) | Azure"
    description="如何在 Linux 上使用 Azure IoT Edge 创建模拟设备，从而通过网关将遥测数据发送到 IoT 中心。"
    services="iot-hub"
    documentationcenter=""
    author="chipalost"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="11e7bf28-ee3d-48d6-a386-eb506c7a31cf"
    ms.service="iot-hub"
    ms.devlang="cpp"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="02/08/2017"
    wacn.date="03/10/2017"
    ms.author="v-yiso" />

---
# <a name="use-azure-iot-edge-to-send-device-to-cloud-messages-with-a-simulated-device-linux"></a>使用 Azure IoT Edge，通过模拟设备发送设备到云的消息 (Linux)
[AZURE.INCLUDE [iot-hub-gateway-sdk-simulated-selector](../../includes/iot-hub-gateway-sdk-simulated-selector.md)]

## <a name="build-and-run-the-sample"></a>生成并运行示例

开始之前，必须：

* [设置开发环境][lnk-setupdevbox] ，以便在 Linux 上使用 SDK。
* [创建 IoT 中心][lnk-create-hub] ，需要中心的名称来完成本演练。 如果没有帐户，可以创建一个[试用帐户][lnk-free-trial]，只需几分钟即可完成。
* 将两个设备添加到 IoT 中心，并记下其 ID 和设备密钥。 可使用[设备资源管理器][lnk-device-explorer]或 [iothub-explorer][lnk-iothub-explorer] 工具将设备添加到在上一步中创建的 IoT 中心，并检索其密钥。

生成示例：

1. 打开 shell。
2. 浏览到 **iot-edge** 存储库本地副本中的根文件夹。
3. 运行 **tools/build.sh** 脚本。 此脚本使用 **cmake** 实用工具在 **iot-edge** 存储库本地副本的根文件夹中创建一个名为 **build** 的文件夹，并生成一个生成文件。 然后，该脚本将生成解决方案并跳过单元测试和端到端测试。 如果想要生成并运行单元测试，请添加 **--run-unittests** 参数。 如果想要生成并运行端到端测试，请添加 **--run-e2e-tests**。 

> [AZURE.NOTE]
> 每次运行 **build.sh** 脚本时，都会删除 **iot-edge** 存储库本地副本的根文件夹中的 **build** 文件夹并重新创建。
> 
> 

运行示例：

在文本编辑器中，打开 **iot-edge** 存储库本地副本中的文件 **samples/simulated_device_cloud_upload/src/simulated_device_cloud_upload_lin.json**。 此文件配置示例网关中的模块：

- **IoTHub** 模块连接到 IoT 中心。 必须配置该模块才能将数据发送到 IoT 中心。 具体而言，将 **IoTHubName** 值设置为 IoT 中心的名称，将 **IoTHubSuffix** 值设置为 **azure-devices.net**。 将 **Transport** 值设置为以下值之一：“HTTP”、“AMQP”或“MQTT”。 请注意，当前只有“HTTP”会对所有设备消息共享一个 TCP 连接。 如果将该值设置为“AMQP”或“MQTT”，则网关会为每个设备维护与 IoT 中心之间的单独 TCP 连接。
- **mapping** 模块将模拟设备的 MAC 地址映射到 IoT 中心设备 ID。 确保 **deviceId** 值与添加到 IoT 中心的两台设备的 ID 一致，确保 **deviceKey** 值包含两台设备的密钥。
- **BLE1** 和 **BLE2** 模块是模拟设备。 注意其 MAC 地址是否与 **mapping** 模块中的地址一致。
- **Logger** 模块将网关活动记录到一个文件中。
* 如下所示的 **module path** 值假定将从 **iot-edge** 存储库本地副本的根文件夹运行示例。
- JSON 文件底部的 **links** 数组将 **BLE1** 和 **BLE2** 模块连接到 **mapping** 模块，并将 **mapping** 模块连接到 **IoTHub** 模块。 它还确保 **Logger** 模块记录所有消息。

        {
            "modules": [
                {
                    "name": "IotHub",
                  "loader": {
                    "name": "native",
                    "entrypoint": {
                      "module.path": "./modules/iothub/libiothub.so"
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
                      "module.path": "./modules/identitymap/libidentity_map.so"
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
                      "module.path": "./modules/simulated_device/libsimulated_device.so"
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
                      "module.path": "./modules/simulated_device/libsimulated_device.so"
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
                      "module.path": "./modules/logger/liblogger.so"
                    }
                    },
                    "args": {
                      "filename": "deviceCloudUploadGatewaylog.log"
                    }
                  }
            ],
            "links": [
                {
                    "source": "*",
                    "sink": "Logger"
                },
                {
                    "source": "BLE1",
                    "sink": "mapping"
                },
                {
                    "source": "BLE2",
                    "sink": "mapping"
                },
                {
                    "source": "mapping",
                    "sink": "IotHub"
                }
            ]
        }


保存对配置文件所做的任何更改。

运行示例：

1. 在 shell 中，浏览到 **iot-edge/build** 文件夹。
2. 运行以下命令：

        ./samples/simulated_device_cloud_upload/simulated_device_cloud_upload_sample ./../samples/simulated_device_cloud_upload/src/simulated_device_cloud_upload_lin.json

3. 可使用[设备资源管理器][lnk-device-explorer]或 [iothub-explorer][lnk-iothub-explorer] 工具监视 IoT 中心从网关接收的消息。

## <a name="next-steps"></a>后续步骤
如果想要深入了解 Azure IoT Edge 并尝试一些代码示例，请访问以下开发人员教程和资源：

* [使用 Azure IoT Edge 从物理设备发送设备到云的消息][lnk-physical-device]
* [Azure IoT Edge][lnk-gateway-sdk]

若要进一步探索 IoT 中心的功能，请参阅：

- [IoT 中心开发人员指南][lnk-devguide]
- [从根本上保护 IoT 解决方案][lnk-securing]

<!-- Links -->
[lnk-setupdevbox]: https://github.com/Azure/iot-edge/blob/master/doc/devbox_setup.md
[lnk-free-trial]: /pricing/1rmb-trial/
[lnk-device-explorer]: https://github.com/Azure/azure-iot-sdk-csharp/tree/master/tools/DeviceExplorer
[lnk-iothub-explorer]: https://github.com/Azure/iothub-explorer/blob/master/readme.md
[lnk-gateway-sdk]: https://github.com/Azure/iot-edge/

[lnk-physical-device]: /documentation/articles/iot-hub-gateway-sdk-physical-device/

[lnk-devguide]: /documentation/articles/iot-hub-devguide/
[lnk-securing]: /documentation/articles/iot-hub-security-ground-up/
[lnk-create-hub]: /documentation/articles/iot-hub-create-through-portal/
