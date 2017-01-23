<properties
    pageTitle="Azure IoT 网关 SDK 入门 (Linux) | Azure"
    description="了解如何在 Linux 计算机上生成网关，并了解 Azure IoT 网关 SDK（如模块）和 JSON 配置文件中的重要概念。"
    services="iot-hub"
    documentationcenter=""
    author="chipalost"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="cf537bdd-2352-4bb1-96cd-a283fcd3d6cf"
    ms.service="iot-hub"
    ms.devlang="cpp"
    ms.topic="get-started-article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="11/23/2016"
    wacn.date="01/13/2017"
    ms.author="andbuc" />  


# Azure IoT 网关 SDK 入门 \(Linux\)

[AZURE.INCLUDE [iot-hub-gateway-sdk-getstarted-selector](../../includes/iot-hub-gateway-sdk-getstarted-selector.md)]

## 如何生成示例
开始之前，必须[设置开发环境][lnk-setupdevbox]，以便在 Linux 上使用 SDK。

1. 打开 shell。
2. 浏览到本地 **azure-iot-gateway-sdk** 存储库副本中的根文件夹。
3. 运行 **tools/build.sh --skip-unittests** 脚本。此脚本使用 **cmake** 实用工具在本地 **azure-iot-gateway-sdk** 存储库副本的根文件夹中创建一个名为 **build** 的文件夹，并生成一个生成文件。然后，该脚本将生成解决方案，跳过单元测试。若想生成并运行单元测试，请删除 **--skip-unittests** 参数。

> [AZURE.NOTE]  每次运行 **build.sh** 脚本时，都会删除本地 **azure-iot-gateway-sdk** 存储库副本的根文件夹中的 **build** 文件夹并重新生成。

## 如何运行示例

1. **build.sh** 脚本在本地 **azure-iot-gateway-sdk** 存储库副本的 **build** 文件夹中生成输出。该文件夹中包含本示例中使用的两个模块。
   
    生成脚本将 **liblogger.so** 放在 **build/modules/logger/** 文件夹中，将 **libhello\_world.so** 放在 **build/modules/hello\_world/** 文件夹中。按如下 JSON 设置文件所示，将这些路径用于 **module path** 值。
2. hello\_world\_sample 进程采用 JSON 配置文件路径作为命令行中的参数。已在 **azure-iot-gateway-sdk/samples/hello\_world/src/hello\_world\_win.json** 中将示例 JSON 文件作为存储库的一部分提供，并复制在下方。除非修改生成脚本，将模块或示例可执行文件放在非默认位置，否则它会按原样运行。

    > [AZURE.NOTE]
   模块路径相对于当前工作目录（hello\_world\_sample 可执行文件的启动位置），而不相对于可执行文件所在的目录。示例 JSON 配置文件默认将“log.txt”写入当前工作目录。
   

        {
            "modules" :
            [
                {
                  "name" : "logger",
                  "loader": {
                    "name": "native",
                    "entrypoint": {
                      "module.path": "./modules/logger/liblogger.so"
                    }
                  },
                  "args" : {"filename":"log.txt"}
                },
                {
                    "name" : "hello_world",
                  "loader": {
                    "name": "native",
                    "entrypoint": {
                      "module.path": "./modules/hello_world/libhello_world.so"
                    }
                  },
                    "args" : null
                }
            ],
            "links": 
            [
                {
                    "source": "hello_world",
                    "sink": "logger"
                }
            ]
        }

3. 浏览到 **azure-iot-gateway-sdk/build** 文件夹。
4. 运行以下命令：
   
   ```
   ./build/samples/hello_world/hello_world_sample ./samples/hello_world/src/hello_world_lin.json
   ``` 

[AZURE.INCLUDE [iot-hub-gateway-sdk-getstarted-code](../../includes/iot-hub-gateway-sdk-getstarted-code.md)]

<!-- Links -->

[lnk-setupdevbox]: https://github.com/Azure/azure-iot-gateway-sdk/blob/master/doc/devbox_setup.md

<!---HONumber=Mooncake_0109_2017-->
<!--Update_Description:update wording-->