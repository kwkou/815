<properties
    pageTitle="Azure IoT 中心的云到设备的消息 (Node) | Azure"
    description="如何使用 Azure IoT SDK for Node.js 将云到设备的消息从 Azure IoT 中心发送到设备。修改模拟设备应用以接收云到设备消息，并修改后端应用以发送云到设备消息。"
    services="iot-hub"
    documentationcenter="nodejs"
    author="dominicbetts"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="3ca8a78f-ade2-46e8-8a49-d5d599cdf1f1"
    ms.service="iot-hub"
    ms.devlang="javascript"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="03/16/2017"
    wacn.date="04/24/2017"
    ms.author="dobett" />  


# 使用 IoT 中心发送云到设备的消息 \(Node\)

[AZURE.INCLUDE [iot-hub-selector-c2d](../../includes/iot-hub-selector-c2d.md)]

## 介绍
Azure IoT 中心是一项完全托管的服务，有助于在数百万台设备和单个解决方案后端之间实现安全可靠的双向通信。[IoT 中心入门]教程介绍了如何创建 IoT 中心和在其中预配设备标识，并介绍了如何编写用于发送设备到云消息的模拟设备应用。

本教程以 [IoT 中心入门]为基础。其中了说明了如何：

* 通过 IoT 中心，将云到设备的消息从解决方案后端发送到单个设备。
* 在设备上接收云到设备的消息。
* 通过解决方案后端，请求确认收到从 IoT 中心发送到设备的消息（*反馈*）。

若要深入了解云到设备的消息，可参阅 [IoT 中心开发人员指南][IoT Hub developer guide - C2D]。

在本教程结束时，你将运行两个 Node.js 控制台应用：

* **SimulatedDevice**，这是在 [IoT 中心入门]中创建的应用的修改版本，可连接到 IoT 中心并接收云到设备的消息。
* **SendCloudToDeviceMessage**，它通过 IoT 中心将云到设备的消息发送到模拟设备应用，然后接收其传送确认。

> [AZURE.NOTE] IoT 中心通过 Azure IoT 设备 SDK 对许多设备平台和语言（包括 C、Java 和 Javascript）提供 SDK 支持。有关如何将设备连接到本教程中的代码（通常是连接到 Azure IoT 中心）的逐步说明，请参阅 [Azure IoT Developer Center]（Azure IoT 开发人员中心）。

若要完成本教程，您需要以下各项：

+ Node.js 0.10.x 或更高版本。

+ 有效的 Azure 帐户。（如果没有帐户，只需花费几分钟就能创建一个[帐户][lnk-free-trial]。）

## 在模拟设备应用上接收消息
在本部分中，将修改 [IoT 中心入门]中创建的模拟设备应用，接收来自 IoT 中心的云到设备消息。

1. 使用文本编辑器打开 SimulatedDevice.js 文件。
2. 修改 **connectCallback** 函数以处理 IoT 中心发来的消息。在本示例中，设备始终调用 **complete** 函数，以通知 IoT 中心它已处理消息。**connectCallback** 函数的新版本类似于以下片段：
   
    
        var connectCallback = function (err) {
          if (err) {
            console.log('Could not connect: ' + err);
          } else {
            console.log('Client connected');
            client.on('message', function (msg) {
              console.log('Id: ' + msg.messageId + ' Body: ' + msg.data);
              client.complete(msg, printResultFor('completed'));
            });
            // Create a message and send it to the IoT Hub every second
            setInterval(function(){
                var windSpeed = 10 + (Math.random() * 4);
                var data = JSON.stringify({ deviceId: 'myFirstNodeDevice', windSpeed: windSpeed });
                var message = new Message(data);
                console.log("Sending message: " + message.getData());
                client.sendEvent(message, printResultFor('send'));
            }, 1000);
          }
        };
    

   > [AZURE.NOTE]如果使用 HTTP（而不使用 MQTT 或 AMQP）作为传输，**DeviceClient** 实例不会经常检查 IoT 中心发来的消息（时间间隔小于 25 分钟）。有关 MQTT、AMQP 和 HTTP 支持之间的差异，以及 IoT 中心限制的详细信息，请参阅 [IoT 中心开发人员指南][IoT Hub Developer Guide - C2D]。

## 发送云到设备的消息
在本部分中，你将创建一个 Node.js 控制台应用，用于将云到设备的消息发送到模拟设备应用。需使用 [IoT 中心入门]教程中添加的设备的设备 ID。还需要中心的 IoT 中心连接字符串（位于 [Azure 门户预览]）。

1. 创建名为 **sendcloudtodevicemessage** 的空文件夹。在命令提示符下的 **sendcloudtodevicemessage** 文件夹中，使用以下命令创建 package.json 文件。接受所有默认值：
   
    
        npm init
    
2. 在命令提示符下的 **sendcloudtodevicemessage** 文件夹中，运行以下命令以安装 **azure-iothub** 包：
   
    
        npm install azure-iothub --save
    
3. 通过文本编辑器，在 **sendcloudtodevicemessage** 文件夹中创建一个 **SendCloudToDeviceMessage.js** 文件。
4. 在 **SendCloudToDeviceMessage.js** 文件的开头添加以下 `require` 语句：
   
    
        'use strict';
       
        var Client = require('azure-iothub').Client;
        var Message = require('azure-iot-common').Message;
    
5. 将以下代码添加到 **SendCloudToDeviceMessage.js** 文件。将 IoT 中心连接字符串占位符值替换为在 [IoT 中心入门]教程中创建的中心的 IoT 中心连接字符串。将目标设备占位符替换为在 [IoT 中心入门]教程中所添加的设备的设备 ID：
   
    
        var connectionString = '{iot hub connection string}';
        var targetDevice = '{device id}';
       
        var serviceClient = Client.fromConnectionString(connectionString);
    
6. 添加以下函数，以便在控制台中列显操作结果：
   
    
        function printResultFor(op) {
          return function printResult(err, res) {
            if (err) console.log(op + ' error: ' + err.toString());
            if (res) console.log(op + ' status: ' + res.constructor.name);
          };
        }
    
7. 添加以下函数，以便在控制台中列显送达反馈消息：
   
    
        function receiveFeedback(err, receiver){
          receiver.on('message', function (msg) {
            console.log('Feedback message:')
            console.log(msg.getData().toString('utf-8'));
          });
        }
    
8. 添加以下代码，以便在设备确认收到云到设备的消息时将消息发送到设备，并处理反馈消息：
   
    
        serviceClient.open(function (err) {
          if (err) {
            console.error('Could not connect: ' + err.message);
          } else {
            console.log('Service client connected');
            serviceClient.getFeedbackReceiver(receiveFeedback);
            var message = new Message('Cloud to device message.');
            message.ack = 'full';
            message.messageId = "My Message ID";
            console.log('Sending message: ' + message.getData());
            serviceClient.send(targetDevice, message, printResultFor('send'));
          }
        });
    
9. 保存并关闭 **SendCloudToDeviceMessage.js** 文件。

## 运行应用程序
现在，你已准备就绪，可以运行应用程序了。

1. 在 **simulateddevice** 文件夹的命令提示符处运行以下命令，以便将遥测发送到 IoT 中心并侦听云到设备的消息：
   
    
        node SimulatedDevice.js 
    
   
    ![运行模拟设备应用][img-simulated-device]  

2. 在 **sendcloudtodevicemessage** 文件夹的命令提示符处运行以下命令，以便发送云到设备的消息并等待确认反馈：
   
    
        node SendCloudToDeviceMessage.js 
    
   
    ![运行应用以发送云到设备的命令][img-send-command]  

   
    > [AZURE.NOTE] 为简单起见，本教程不实现任何重试策略。在生产代码中，应按 MSDN 文章 [Transient Fault Handling]（暂时性故障处理）中所述实施重试策略（例如指数性的回退）。

## 后续步骤
在本教程中，你已学习如何发送和接收云到设备的消息。

<!-- To see examples of complete end-to-end solutions that use IoT Hub, see [Azure IoT Suite]. -->


若要深入了解如何使用 IoT 中心开发解决方案，请参阅 [IoT 中心开发人员指南]。

<!-- Images -->
[img-simulated-device]: ./media/iot-hub-node-node-c2d/receivec2d.png
[img-send-command]: ./media/iot-hub-node-node-c2d/sendc2d.png

<!-- Links -->


[IoT 中心入门]: /documentation/articles/iot-hub-node-node-getstarted/
[IoT Hub Developer Guide - C2D]: /documentation/articles/iot-hub-devguide-messaging/
[IoT 中心开发人员指南]: /documentation/articles/iot-hub-devguide/
[Azure IoT Developer Center]: /develop/iot/
[lnk-free-trial]: /pricing/1rmb-trial/
[lnk-dev-setup]: https://github.com/Azure/azure-iot-sdks/blob/master/doc/get_started/node-devbox-setup.md
[Transient Fault Handling]: https://msdn.microsoft.com/zh-cn/library/hh680901(v=pandp.50).aspx
[Azure 门户预览]: https://portal.azure.cn
[Azure IoT Suite]: /documentation/services/iot-suite/

<!--Update_Description:update meta properties-->