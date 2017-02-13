<properties
    pageTitle="Azure IoT 中心直接方法 (Node) | Azure"
    description="如何使用 Azure IoT 中心直接方法。使用 Azure IoT SDK for Node.js 实现包含直接方法的模拟设备应用和调用直接方法的服务应用。"
    services="iot-hub"
    documentationcenter=""
    author="nberdy"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="ea9c73ca-7778-4e38-a8f1-0bee9d142f04"
    ms.service="iot-hub"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="01/11/2017"
    wacn.date="02/10/2017"
    ms.author="nberdy" />  


# 使用直接方法 \(Node\)
[AZURE.INCLUDE [iot-hub-selector-c2d-methods](../../includes/iot-hub-selector-c2d-methods.md)]

在本教程结束时，你将拥有两个 Node.js 控制台应用：

* **CallMethodOnDevice.js**，用于在模拟设备应用上调用方法并显示响应。
* **SimulatedDevice.js**，可使用前面创建的设备标识连接到 IoT 中心，并响应通过云调用的方法。

> [AZURE.NOTE]
[Azure IoT SDK][lnk-hub-sdks] 文章介绍了 Azure IoT SDK，这些 SDK 可用于构建在设备和解决方案后端运行的应用程序。
> 
> 

要完成本教程，需要具备以下先决条件：

* Node.js 0.10.x 或更高版本。
* 有效的 Azure 帐户。（如果没有帐户，只需花费几分钟就能创建一个[帐户][lnk-free-trial]。）

[AZURE.INCLUDE [iot-hub-get-started-create-hub](../../includes/iot-hub-get-started-create-hub.md)]

[AZURE.INCLUDE [iot-hub-get-started-create-device-identity](../../includes/iot-hub-get-started-create-device-identity.md)]

## 创建模拟设备应用程序
在本部分，用户需创建一个 Node.js 控制台应用，用于响应通过云调用的方法。

1. 新建名为 **simulateddevice** 的空文件夹。在 **simulateddevice** 文件夹的命令提示符处，使用以下命令创建 package.json 文件。接受所有默认值：
   
    
        npm init
    
2. 在 **simulateddevice** 文件夹的命令提示符处，运行下述命令以安装 **azure-iot-device** 设备 SDK 包和 **azure-iot-device-mqtt** 包：
   
    
        npm install azure-iot-device azure-iot-device-mqtt --save
    
3. 在 **simulateddevice** 文件夹中，利用文本编辑器创建新的 **SimulatedDevice.js** 文件。
4. 在 **SimulatedDevice.js** 文件的开头添加以下 `require` 语句：
   
    
        'use strict';
       
        var Mqtt = require('azure-iot-device-mqtt').Mqtt;
        var DeviceClient = require('azure-iot-device').Client;
    
5. 添加 **connectionString** 变量，并使用它创建 **DeviceClient** 实例。将 **{device connection string}** 替换为在“创建设备标识”部分创建的设备连接字符串：
   
    
        var connectionString = '{device connection string}';
        var client = DeviceClient.fromConnectionString(connectionString, Mqtt);
    
6. 添加以下函数，实现设备上的方法：
   
    
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
    
7. 打开与 IoT 中心的连接并开始初始化方法侦听器：
   
    
        client.open(function(err) {
            if (err) {
                console.error('could not open IotHub client');
            }  else {
                console.log('client opened');
                client.onDeviceMethod('writeLine', onWriteLine);
            }
        });
    
8. 保存并关闭 **SimulatedDevice.js** 文件。

> [AZURE.NOTE]
为简单起见，本教程不实现任何重试策略。在生产代码中，应按 MSDN 文章 [Transient Fault Handling][lnk-transient-faults]（暂时性故障处理）中所述实施重试策略（例如连接重试）。
> 
> 

## 调用设备上的方法
在此部分中，会创建一个 Node.js 控制台应用，该应用在模拟设备应用上调用方法并随后显示响应。

1. 新建名为 **callmethodondevice** 的空文件夹。在 **callmethodondevice** 文件夹的命令提示符处，使用以下命令创建 package.json 文件。接受所有默认值：
   
    
        npm init
    
2. 在 **callmethodondevice** 文件夹的命令提示符处，运行以下命令以安装 **azure-iothub** 包：
   
    
        npm install azure-iothub --save
    
3. 使用文本编辑器，在 **callmethodondevice** 文件夹中创建 **CallMethodOnDevice.js** 文件。
4. 在 **CallMethodOnDevice.js** 文件的开头添加以下 `require` 语句：
   
    
        'use strict';
       
        var Client = require('azure-iothub').Client;
    
5. 添加以下变量声明，并将占位符值替换为中心的 IoT 中心连接字符串：
   
    
        var connectionString = '{iothub connection string}';
        var methodName = 'writeLine';
        var deviceId = 'myDeviceId';
    
6. 创建客户端，以便打开到 IoT 中心的连接。
   
    
        var client = Client.fromConnectionString(connectionString);
    
7. 添加以下函数，以便调用设备方法并将设备响应输出到控制台：
   
    
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
    
8. 保存并关闭 **CallMethodOnDevice.js** 文件。

## 运行应用
现在，已准备就绪，可以运行应用。

1. 在 **simulateddevice** 文件夹的命令提示符处运行以下命令，开始侦听从 IoT 中心发出的方法调用：
   
    
        node SimulatedDevice.js
    
   
    ![][7]  

2. 在 **callmethodondevice** 文件夹的命令提示符处运行以下命令，开始监视 IoT 中心：
   
    
        node CallMethodOnDevice.js 
    
   
    ![][8]  

3. 此时会看到设备通过输出消息对方法进行响应，而调用该方法的应用程序则会显示来自设备的响应：
   
    ![][9]  


## 后续步骤
本教程中，在 Azure 门户预览中配置了新的 IoT 中心，然后在 IoT 中心的标识注册表中创建了设备标识。你已通过此设备标识启用模拟设备应用的相关功能，使之能够响应通过云调用的方法。你还创建了一个应用，用于调用设备上的方法并显示来自设备的响应。

若要继续了解 IoT 中心入门知识并浏览其他 IoT 方案，请参阅：

* [IoT 中心入门]
* [Schedule jobs on multiple devices（在多台设备上计划作业）][lnk-devguide-jobs]

若要了解如何扩展 IoT 解决方案并在多个设备上计划方法调用，请参阅 [Schedule and broadcast jobs][lnk-tutorial-jobs]（计划和广播作业）教程。

<!-- Images. -->

[7]: ./media/iot-hub-node-node-direct-methods/run-simulated-device.png
[8]: ./media/iot-hub-node-node-direct-methods/run-callmethodondevice.png
[9]: ./media/iot-hub-node-node-direct-methods/methods-output.png

<!-- Links -->

[lnk-transient-faults]: https://msdn.microsoft.com/zh-cn/library/hh680901(v=pandp.50).aspx

[lnk-dev-setup]: https://github.com/Azure/azure-iot-sdks/blob/master/doc/get_started/node-devbox-setup.md

[lnk-hub-sdks]: /documentation/articles/iot-hub-devguide-sdks/
[lnk-free-trial]: /pricing/1rmb-trial/
[lnk-portal]: https://portal.azure.cn/

[lnk-devguide-jobs]: /documentation/articles/iot-hub-devguide-jobs/
[lnk-tutorial-jobs]: /documentation/articles/iot-hub-node-node-schedule-jobs/
[lnk-devguide-methods]: /documentation/articles/iot-hub-devguide-direct-methods/
[lnk-devguide-mqtt]: /documentation/articles/iot-hub-mqtt-support/

[Send Cloud-to-Device messages with IoT Hub]: /documentation/articles/iot-hub-csharp-csharp-c2d/
[Process Device-to-Cloud messages]: /documentation/articles/iot-hub-csharp-csharp-process-d2c/
[IoT 中心入门]: /documentation/articles/iot-hub-node-node-getstarted/

<!---HONumber=Mooncake_0109_2017-->
<!--Update_Description:update meta properties-->