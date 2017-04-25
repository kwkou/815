<properties
    pageTitle="Azure IoT 网关 SDK 入门 (Linux) | Azure"
    description="了解如何在 Linux 计算机上生成网关，并了解 Azure IoT 网关 SDK（如模块）和 JSON 配置文件中的重要概念。"
    services="iot-hub"
    documentationcenter=""
    author="chipalost"
    manager="timlt"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid="cf537bdd-2352-4bb1-96cd-a283fcd3d6cf"
    ms.service="iot-hub"
    ms.devlang="cpp"
    ms.topic="get-started-article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="11/23/2016"
    wacn.date="04/24/2017"
    ms.author="andbuc"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="60c68d81de2c89dc99b7dce10d7f3c6d965742e8"
    ms.lasthandoff="04/14/2017" />

# <a name="explore-the-iot-gateway-sdk-architecture-on-linux"></a>探索 Linux 上的 IoT 网关 SDK 体系结构
[AZURE.INCLUDE [iot-hub-gateway-sdk-getstarted-selector](../../includes/iot-hub-gateway-sdk-getstarted-selector.md)]

## <a name="how-to-build-the-sample"></a>如何生成示例
开始之前，必须 [设置开发环境][lnk-setupdevbox] ，以便在 Linux 上使用 SDK。

1. 打开 shell。
2. 浏览到本地 **azure-iot-gateway-sdk** 存储库副本中的根文件夹。
3. 运行 **tools/build.sh** 脚本。 此脚本使用 **cmake** 实用工具在 **azure-iot-gateway-sdk** 存储库本地副本的根文件夹中创建一个名为 **build** 的文件夹，并生成一个生成文件。 然后，该脚本将生成解决方案并跳过单元测试和端到端测试。 如果想要生成并运行单元测试，请添加 **--run-unittests** 参数。 如果想要生成并运行端到端测试，请添加 **--run-e2e-tests**。

> [AZURE.NOTE]
> 每次运行 **build.sh** 脚本时，都会删除 **azure-iot-gateway-sdk** 存储库本地副本的根文件夹中的 **build** 文件夹并重新生成。
> 
> 

## <a name="how-to-run-the-sample"></a>如何运行示例
1. **build.sh** 脚本在 **azure-iot-gateway-sdk** 存储库的本地副本内的 **build** 文件夹中生成输出。 该文件夹中包含本示例中使用的两个模块。

    生成脚本将 **liblogger.so** 放在 **build/modules/logger/** 文件夹中，将 **libhello_world.so** 放在 **build/modules/hello_world/** 文件夹中。 按如下 JSON 设置文件所示，将这些路径用于 **module path** 值。
2. Hello_world_sample 过程使用 JSON 配置文件的路径作为命令行中的参数。 **azure-iot-gateway-sdk/samples/hello_world/src/hello_world_win.json** 上的存储库中已提供示例 JSON 文件，下面也复制了该文件的内容。 除非你修改了生成脚本，将模块或示例可执行文件放置在非默认位置，否则，可按原样运行此脚本。

   > [AZURE.NOTE]
   > 模块路径相对于当前工作目录（hello_world_sample 可执行文件的启动位置），而不相对于可执行文件所在的目录。 示例 JSON 配置文件默认为在当前工作目录中写入“log.txt”。

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

       ./build/samples/hello_world/hello_world_sample ./samples/hello_world/src/hello_world_lin.json

[AZURE.INCLUDE [iot-hub-gateway-sdk-getstarted-code](../../includes/iot-hub-gateway-sdk-getstarted-code.md)]

<!-- Links -->
[lnk-setupdevbox]: https://github.com/Azure/azure-iot-gateway-sdk/blob/master/doc/devbox_setup.md

<!--Update_Description:update wording-->