<properties
 pageTitle="使用直接方法 | Azure"
 description="本教程介绍如何使用直接方法"
 services="iot-hub"
 documentationCenter=""
 authors="nberdy"
 manager="timlt"
 editor=""/>  


<tags
 ms.service="iot-hub"
 ms.devlang="na"
 ms.topic="article"
 ms.tgt_pltfrm="na"
 ms.workload="na"
 ms.date="10/05/2016"
 wacn.date="11/07/2016"
 ms.author="nberdy"/>  


# 教程：使用直接方法

## 介绍

Azure IoT 中心是一项完全托管的服务，可在数百万个 IoT 设备和一个应用程序后端之间实现安全可靠的双向通信。以前的教程（[Get started with IoT Hub]（IoT 中心入门）和 [Send Cloud-to-Device messages with IoT Hub]（使用 IoT 中心发送云到设备的消息））介绍了 IoT 中心的设备到云和云到设备的基本消息传递功能。用户还可以通过 IoT 中心从云调用设备上的非持久方法。这些方法表示与设备进行的请求-答复式交互，类似于 HTTP 调用，因为它们不管是成功还是失败，速度都非常快（在用户指定的超时过后），会让用户知道调用的状态。[Invoke a direct method on a device][lnk-devguide-methods]（调用设备上的直接方法）更详细地介绍了各种方法，指导用户何时使用这些方法，何时使用云到设备的消息。

本教程演示如何：

- 使用 Azure 门户创建 IoT 中心，以及如何在 IoT 中心创建设备标识。
- 创建一个模拟设备，以便通过云调用其中的直接方法。
- 创建一个控制台应用程序，以便通过 IoT 中心调用模拟设备上的直接方法。

> [AZURE.NOTE] 目前只能使用 MQTT 协议从连接到 IoT 中心的设备访问直接方法。有关如何转换现有设备应用以使用 MQTT 的说明，请参阅 [MQTT 支持][lnk-devguide-mqtt]一文。

在本教程结束时，用户有两个 Node.js 控制台应用程序：

* **CallMethodOnDevice.js**，用于调用模拟设备上的方法并显示响应。
* **SimulatedDevice.js**，可使用前面创建的设备标识连接到 IoT 中心，并响应通过云调用的方法。

> [AZURE.NOTE] [IoT 中心 SDK][lnk-hub-sdks] 一文提供了各种 SDK 的相关信息，你可以使用这些 SDK 构建可在设备和解决方案后端上运行的应用程序。

若要完成本教程，您需要以下各项：

+ Node.js 0.10.x 或更高版本。

+ 有效的 Azure 帐户。（如果你没有帐户，只需花费几分钟就能创建一个试用帐户。有关详细信息，请参阅 [Azure 试用版][lnk-free-trial]。）

[AZURE.INCLUDE [iot-hub-get-started-create-hub-pp](../../includes/iot-hub-get-started-create-hub-pp.md)]

## 创建模拟设备应用程序

在本部分，用户需创建一个 Node.js 控制台应用，用于响应通过云调用的方法。

1. 新建名为 **simulateddevice** 的空文件夹。在 **simulateddevice** 文件夹的命令提示符处，使用以下命令创建 package.json 文件。接受所有默认值：

    ```
    npm init
    ```

2. 在 **simulateddevice** 文件夹的命令提示符处，运行下述命令以安装 **azure-iot-device** 设备 SDK 包和 **azure-iot-device-mqtt** 包：

    ```
    npm install azure-iot-device@dtpreview azure-iot-device-mqtt@dtpreview --save
    ```

3. 在 **simulateddevice** 文件夹中，利用文本编辑器创建新的 **SimulatedDevice.js** 文件。

4. 在 **SimulatedDevice.js** 文件的开头添加以下 `require` 语句：

    ```
    'use strict';

    var Mqtt = require('azure-iot-device-mqtt').Mqtt;
    var DeviceClient = require('azure-iot-device').Client;
    ```

5. 添加 **connectionString** 变量，并用其创建设备客户端。将 **{device connection string}** 替换为在“创建设备标识”部分创建的连接字符串：

    ```
    var connectionString = '{device connection string}';
    var client = DeviceClient.fromConnectionString(connectionString, Mqtt);
    ```

6. 添加以下函数，实现设备上的方法：

    ```
    function onWriteLine(request, response) {
        console.log(request.payload);

        response.send(200, 'Input was written to log.', function(err) {
            if(err) {
                console.error('An error ocurred when sending a method response:\n' + err.toString());
            } else {
                console.log('Response to method \'' + request.methodName + '\' sent successfully.' );
            }
        });
    }
    ```

7. 打开与 IoT 中心的连接并开始初始化方法侦听器：

	```
	client.open(function(err) {
		if (err) {
			console.error('could not open IotHub client');
		}  else {
			console.log('client opened');
			client.onDeviceMethod('writeLine', onWriteLine);
		}
	});
	```

8. 保存并关闭 **SimulatedDevice.js** 文件。

> [AZURE.NOTE] 为简单起见，本教程不实现任何重试策略。在生产代码中，应按 MSDN 文章 [Transient Fault Handling][lnk-transient-faults]（暂时性故障处理）中所述实施重试策略（例如连接重试）。

## 调用设备上的方法

在此部分，用户需创建一个 Node.js 控制台应用，以便调用模拟设备上的方法，然后显示响应。

1. 新建名为 **callmethodondevice** 的空文件夹。在 **callmethodondevice** 文件夹的命令提示符处，使用以下命令创建 package.json 文件。接受所有默认值：

    ```
    npm init
    ```

2. 在 **callmethodondevice** 文件夹的命令提示符处，运行以下命令以安装 **azure-iothub** 包：

    ```
    npm install azure-iothub@dtpreview --save
    ```

3. 使用文本编辑器，在 **callmethodondevice** 文件夹中创建 **CallMethodOnDevice.js** 文件。

4. 在 **CallMethodOnDevice.js** 文件的开头添加以下 `require` 语句：

    ```
    'use strict';

    var Client = require('azure-iothub').Client;
    ```

5. 添加以下变量声明，并将占位符值替换为你的 IoT 中心的连接字符串：

    ```
    var connectionString = '{iothub connection string}';
	var methodName = 'writeLine';
	var deviceId = 'myDeviceId';
    ```

6. 创建客户端，以便打开到 IoT 中心的连接。

    ```
    var client = Client.fromConnectionString(connectionString);
    ```
	
7. 添加以下函数，以便调用设备方法并将设备响应输出到控制台：

    ```
    var methodParams = {
        methodName: methodName,
        payload: 'a line to be written',
        timeoutInSeconds: 30
    };

    client.invokeDeviceMethod(deviceId, methodParams, function (err, result) {
        if (err) {
            console.error('Failed to invoke method \'' + methodName + '\': ' + err.message);
        } else {
            console.log(methodName + ' on ' + deviceId + ':');
            console.log(JSON.stringify(result, null, 2));
        }
    });
    ```

7. 保存并关闭 **CallMethodOnDevice.js** 文件。

## 运行应用程序

现在，你已准备就绪，可以运行应用程序了。

1. 在 **simulateddevice** 文件夹的命令提示符处运行以下命令，开始侦听向从 IoT 中心发出的方法调用：

    ```
    node SimulatedDevice.js
    ```

    ![][7]  

	
2. 在 **callmethodondevice** 文件夹中的命令提示符处运行以下命令，开始监视 IoT 中心：

    ```
    node CallMethodOnDevice.js 
    ```

	![][8]  

	
3. 此时会看到设备通过输出消息对方法进行响应，而调用该方法的应用程序则会显示来自设备的响应：

	![][9]  

	
## 后续步骤

在本教程中，你已在门户中配置了新的 IoT 中心，然后在中心的标识注册表中创建了设备标识。你已通过此设备标识启用模拟设备应用的相关功能，使之能够响应通过云调用的方法。你还创建了一个应用，用于调用设备上的方法并显示来自设备的响应。

若要继续了解 IoT 中心入门知识并浏览其他 IoT 方案，请参阅：

- [IoT 中心入门]
- [Schedule jobs on multiple devices][lnk-devguide-jobs]（在多台设备上计划作业）

若要了解如何扩展 IoT 解决方案并在多个设备上计划方法调用，请参阅 [Schedule and broadcast jobs][lnk-tutorial-jobs]（计划和广播作业）教程。

<!-- Images. -->

[7]: ./media/iot-hub-c2d-methods/run-simulated-device.png
[8]: ./media/iot-hub-c2d-methods/run-callmethodondevice.png
[9]: ./media/iot-hub-c2d-methods/methods-output.png

<!-- Links -->

[lnk-transient-faults]: https://msdn.microsoft.com/zh-cn/library/hh680901(v=pandp.50).aspx

[lnk-dev-setup]: https://github.com/Azure/azure-iot-sdks/blob/master/doc/get_started/node-devbox-setup.md

[lnk-hub-sdks]: /documentation/articles/iot-hub-devguide-sdks/
[lnk-free-trial]: /pricing/1rmb-trial/
[lnk-portal]: https://portal.azure.cn/

[lnk-devguide-jobs]: /documentation/articles/iot-hub-devguide-jobs/
[lnk-tutorial-jobs]: /documentation/articles/iot-hub-schedule-jobs/
[lnk-devguide-methods]: /documentation/articles/iot-hub-devguide-direct-methods/
[lnk-devguide-mqtt]: /documentation/articles/iot-hub-mqtt-support/

[Send Cloud-to-Device messages with IoT Hub]: /documentation/articles/iot-hub-csharp-csharp-c2d/
[Process Device-to-Cloud messages]: /documentation/articles/iot-hub-csharp-csharp-process-d2c/
[Get started with IoT Hub]: /documentation/articles/iot-hub-node-node-getstarted/
[IoT 中心入门]: /documentation/articles/iot-hub-node-node-getstarted/

<!---HONumber=Mooncake_1031_2016-->