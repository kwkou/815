<properties
    pageTitle="Azure IoT 网关 SDK 入门 (Linux) | Azure"
    description="了解如何在 Linux 计算机上构建网关，以及 Azure IoT Edge 的重要概念，如模块和 JSON 配置文件。"
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
    ms.date="03/28/2017"
    wacn.date="06/05/2017"
    ms.author="v-yiso"
    ms.custom="H1Hack27Feb2017"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="08618ee31568db24eba7a7d9a5fc3b079cf34577"
    ms.openlocfilehash="394e5ff77a9dd8ae113470a469b4394e4c913976"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/26/2017" />

# <a name="explore-azure-iot-edge-architecture-on-linux"></a>在 Linux 上浏览 Azure IoT Edge 体系结构

[AZURE.INCLUDE [iot-hub-gateway-sdk-getstarted-selector](../../includes/iot-hub-gateway-sdk-getstarted-selector.md)]

## <a name="how-to-build-the-sample"></a>如何生成示例

开始之前，必须 [设置开发环境][lnk-setupdevbox] ，以便在 Linux 上使用 SDK。

1. 打开 shell。
1. 浏览到 **iot-edge** 存储库本地副本中的根文件夹。
1. 运行 **tools/build.sh** 脚本。 此脚本使用 **cmake** 实用工具在 **iot-edge** 存储库本地副本的根文件夹中创建一个名为 **build** 的文件夹，并生成一个生成文件。 然后，该脚本将生成解决方案并跳过单元测试和端到端测试。 如果想要生成并运行单元测试，请添加 **--run-unittests** 参数。 如果想要生成并运行端到端测试，请添加 **--run-e2e-tests**。

> [AZURE.NOTE]
> 每次运行 **build.sh** 脚本时，都会删除 **iot-edge** 存储库本地副本的根文件夹中的 **build** 文件夹并重新创建。
> 
> 

## <a name="how-to-run-the-sample"></a>如何运行示例

1. **build.sh** 脚本在 **iot-edge** 存储库本地副本的 **build** 文件夹中生成输出。 该输出包括此示例中使用的两个模块。

    生成脚本将 **liblogger.so** 放在 **build/modules/logger/** 文件夹中，将 **libhello\_world.so** 放在 **build/modules/hello_world/** 文件夹中。 按以下示例 JSON 设置文件中所示，将这些路径用于 **module path** 值。
1. hello\_world\_sample 过程使用 JSON 配置文件的路径作为命令行参数。 以下示例 JSON 文件在 SDK 存储库的以下路径中提供：**samples/hello\_world/src/hello\_world\_lin.json**。 除非你修改了生成脚本，将模块或示例可执行文件放置在非默认位置，否则，此配置文件可按原样工作。

   > [AZURE.NOTE]
   > 模块路径相对于从中启动 hello\_world\_sample 可执行文件的当前工作目录，而不是可执行文件所在的目录。 示例 JSON 配置文件默认为在当前工作目录中写入“log.txt”。

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

1. 导航到 **build** 文件夹。
1. 运行以下命令：

   `./samples/hello_world/hello_world_sample ./../samples/hello_world/src/hello_world_lin.json`

[AZURE.INCLUDE [iot-hub-gateway-sdk-getstarted-code](../../includes/iot-hub-gateway-sdk-getstarted-code.md)]

<!-- Links -->
[lnk-setupdevbox]: https://github.com/Azure/iot-edge/blob/master/doc/devbox_setup.md