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
     ms.date="04/20/2016"
     wacn.date="05/30/2016"/>


# IoT 网关 SDK（Beta 版）- 使用 Windows 入门

[AZURE.INCLUDE [iot-hub-gateway-sdk-getstarted-selector](../includes/iot-hub-gateway-sdk-getstarted-selector.md)]

## 如何生成示例

开始之前，必须[设置开发环境][lnk-setupdevbox]，以便在 Windows 上使用 SDK。

1. 打开 **VS2015 开发人员命令提示**命令提示符。
2. 浏览到本地 **azure-iot-gateway-sdk** 存储库副本中的根文件夹。
3. 运行 **tools\\build.cmd** 脚本。此脚本创建 Visual Studio 解决方案文件，生成解决方案，并运行测试。你可以在本地 **azure-iot-gateway-sdk** 存储库副本的 **build** 文件夹中找到 Visual Studio 解决方案。

## 如何运行示例

1. **build.cmd** 脚本在本地存储库副本中创建一个名为 **build** 的文件夹。此文件夹中包含本示例中使用的两个模块。

    生成脚本将 **logger\_hl.dll** 放在 **build\\modules\\logger\\Debug** 文件夹中，将 **hello\_world\_hl.dl** 放在 **build\\modules\\hello\_world\\Debug** 文件夹中。按如下 JSON 设置文件所示，将这些路径用于 **module path** 值。

2. **samples\\hello\_world\\src** 文件夹中的文件 **hello\_world\_win.json** 是可用于运行示例的面向 Windows 的示例 JSON 设置文件。如下所示的示例 JSON 设置假定你已将网关 SDK 存储库克隆到了 **C:** 盘的根目录。如果将该存储库下载到其他位置，则需要相应地调整 **samples\\hello\_world\\src\\hello\_world\_win.json** 文件中的 **module path** 值。

3. 对于 **logger\_hl** 模块，请在 **args** 部分，将 **filename** 值设置为包含日志数据的文件的名称和路径。

    这是面向 Windows 的 JSON 设置文件的示例，该文件将写入 **C:** 盘根目录下的 **log.txt** 文件。

    ```
    {
      "modules" :
      [
        {
          "module name" : "logger_hl",
          "module path" : "C:\\azure-iot-gateway-sdk\\build\\modules\\logger\\Debug\\logger_hl.dll",
          "args" : 
          {
            "filename":"C:\\log.txt"
          }
        },
        {
          "module name" : "hello_world",
          "module path" : "C:\\azure-iot-gateway-sdk\\build\\\modules\\hello_world\\Debug\\hello_world_hl.dll",
          "args" : null
        }
      ]
    }
    ```

3. 在命令提示符下，浏览到本地 **azure-iot-gateway-sdk** 存储库副本中的根文件夹。
4. 运行以下命令：
  
  ```
  build\samples\hello_world\Debug\hello_world_sample.exe samples\hello_world\src\hello_world_win.json
  ```

[AZURE.INCLUDE [iot-hub-gateway-sdk-getstarted-code](../includes/iot-hub-gateway-sdk-getstarted-code.md)]

<!-- Links -->
[lnk-setupdevbox]: https://github.com/Azure/azure-iot-gateway-sdk/blob/master/doc/devbox_setup.md



<!---HONumber=Mooncake_0523_2016-->