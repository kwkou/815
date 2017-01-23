<properties
	pageTitle="IoT 中心网关 SDK 入门 | Azure"
	description="使用 Windows 的 Azure IoT 中心网关 SDK 演练，说明使用 Azure IoT 中心网关 SDK 时应理解的关键概念。"
	services="iot-hub"
	documentationCenter=""
	authors="chipalost"
	manager="timlt"
	editor=""/>  


<tags
     ms.service="iot-hub"
     ms.devlang="cpp"
     ms.topic="article"
     ms.tgt_pltfrm="na"
     ms.workload="na"
     ms.date="11/16/2016"
     wacn.date="01/13/2017"
     ms.author="andbuc"/>  



# Azure IoT 网关 SDK 入门 \(Windows\)

[AZURE.INCLUDE [iot-hub-gateway-sdk-getstarted-selector](../../includes/iot-hub-gateway-sdk-getstarted-selector.md)]

## 如何生成示例

开始之前，必须[设置开发环境][lnk-setupdevbox]，以便在 Windows 上使用 SDK。

1. 打开 **VS2015 开发人员命令提示**命令提示符。
2. 浏览到本地 **azure-iot-gateway-sdk** 存储库副本中的根文件夹。
3. 运行 **tools\\build.cmd** 脚本。此脚本创建 Visual Studio 解决方案文件，生成解决方案，并运行测试。你可以在本地 **azure-iot-gateway-sdk** 存储库副本的 **build** 文件夹中找到 Visual Studio 解决方案。

## 如何运行示例
1. **build.cmd** 脚本在本地存储库副本中创建一个名为 **build** 的文件夹。此文件夹中包含本示例中使用的两个模块。
   
    生成脚本将 **logger.dll** 放在 **build\\modules\\logger\\Debug** 文件夹中，将 **hello\_world.dll** 放在 **build\\modules\\hello\_world\\Debug** 文件夹中。按以下 JSON 设置文件所示，将这些路径用于 **module path** 值。
2. hello\_world\_sample 进程采用 JSON 配置文件路径作为命令行中的参数。已在 **azure-iot-gateway-sdk\\samples\\hello\_world\\src\\hello\_world\_win.json** 中将以下示例 JSON 文件作为存储库的一部分提供。除非修改生成脚本，将模块或示例可执行文件放在非默认位置，否则它会按原样运行。

   > [AZURE.NOTE]
   模块路径相对 hello\_world\_sample.exe 所在的目录。示例 JSON 配置文件默认将“log.txt”写入当前工作目录。
   

        {
          "modules": [
            {
              "name": "logger",
              "loader": {
                "name": "native",
                "entrypoint": {
                  "module.path": "..\\..\\..\\modules\\logger\\Debug\\logger.dll"
                }
              },
              "args": { "filename": "log.txt" }
            },
            {
              "name": "hello_world",
              "loader": {
                "name": "native",
                "entrypoint": {
                  "module.path": "..\\..\\..\\modules\\hello_world\\Debug\\hello_world.dll"
                }
              },
              "args": null
              }
          ],
          "links": [
            {
              "source": "hello_world",
              "sink": "logger"
            }
          ]
        }

3. 浏览到 **azure-iot-gateway-sdk** 存储库的本地副本中的根文件夹。

4. 运行以下命令：

        build\samples\hello_world\Debug\hello_world_sample.exe samples\hello_world\src\hello_world_win.json


[AZURE.INCLUDE [iot-hub-gateway-sdk-getstarted-code](../../includes/iot-hub-gateway-sdk-getstarted-code.md)]

<!-- Links -->

[lnk-setupdevbox]: https://github.com/Azure/azure-iot-gateway-sdk/blob/master/doc/devbox_setup.md

<!---HONumber=Mooncake_0109_2017-->
<!--Update_Description:update wording-->