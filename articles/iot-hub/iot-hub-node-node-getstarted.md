<properties
	pageTitle="适用于 Node.js 的 Azure IoT 中心入门 | Azure"
	description="适用于 Node.js 的 Azure IoT 中心入门教程。配合 Azure IoT SDK 使用 Azure IoT 中心和 Node.js 来实施物联网解决方案。"
	services="iot-hub"
	documentationCenter="nodejs"
	authors="dominicbetts"
	manager="timlt"
	editor=""/>  


<tags
     ms.service="iot-hub"
     ms.devlang="javascript"
     ms.topic="hero-article"
     ms.tgt_pltfrm="na"
     ms.workload="na"
     ms.date="09/12/2016"
     wacn.date="01/04/2017"
     ms.author="dobett"/>


# 适用于 Node.js 的 Azure IoT 中心入门

[AZURE.INCLUDE [iot-hub-selector-get-started](../../includes/iot-hub-selector-get-started.md)]

本教程最后提供 3 个 Node.js 控制台应用程序：

* **CreateDeviceIdentity.js**，用于创建设备标识和关联的安全密钥以连接模拟设备。
* **ReadDEviceToCloudMessages.js**，显示模拟设备发送的遥测数据。
* **SimulatedDevice.js**，它使用前面创建的设备标识连接到 IoT 中心，并使用 AMQPS 协议每秒发送一次遥测消息。

> [AZURE.NOTE] [IoT 中心 SDK][lnk-hub-sdks] 一文提供了各种 SDK 的相关信息，你可以使用这些 SDK 构建可在设备和解决方案后端上运行的应用程序。

若要完成本教程，您需要以下各项：

+ Node.js 版本 0.12.x 或更高版本。<br/>

+ 有效的 Azure 帐户。（如果没有帐户，只需花费几分钟就能创建一个[帐户][lnk-free-trial]。）

[AZURE.INCLUDE [iot-hub-get-started-create-hub](../../includes/iot-hub-get-started-create-hub.md)]

现在已创建 IoT 中心。你已获得完成本教程的其余部分所需的 IoT 中心主机名和 IoT 中心连接字符串。

## 创建设备标识
本部分将创建一个 Node.js 控制台应用，用于在 IoT 中心的标识注册表中创建设备标识。设备无法连接到 IoT 中心，除非它在设备标识注册表中具有条目。有关详细信息，请参阅 [IoT Hub Developer Guide][lnk-devguide-identity]（IoT 中心开发人员指南）的 **Device Identity Registry**（设备标识注册表）部分。当你运行此控制台应用程序时，它将生成唯一的设备 ID 和密钥，当设备向 IoT 中心发送设备到云的消息时，可以使用这些信息标识设备本身。

1. 新建名为 **createdeviceidentity** 的空文件夹。在 **createdeviceidentity** 文件夹的命令提示符处，使用以下命令创建 package.json 文件。接受所有默认值：
   
    
	    npm init
    
2. 在 **createdeviceidentity** 文件夹的命令提示符处，运行下述命令以安装 **azure-iothub** 服务 SDK 包：
   
    
	    npm install azure-iothub --save
    
3. 在 **createdeviceidentity** 文件夹中，利用文本编辑器创建 **CreateDeviceIdentity.js** 文件。
4. 在 **CreateDeviceIdentity.js** 文件的开头添加以下 `require` 语句：
   
    
	    'use strict';
   
	    var iothub = require('azure-iothub');
    
5. 将以下代码添加到 **CreateDeviceIdentity.js** 文件，并将占位符值替换为在上一节中为 IoT 中心创建的连接字符串：
   
    
	    var connectionString = '{iothub connection string}';
   
	    var registry = iothub.Registry.fromConnectionString(connectionString);
    
6. 添加以下代码，在 IoT 中心的设备标识注册表中创建设备标识。如果注册表中没有设备 ID，此代码会创建一个设备；否则返回现有设备的密钥：
   
    
	    var device = new iothub.Device(null);
	    device.deviceId = 'myFirstNodeDevice';
	    registry.create(device, function(err, deviceInfo, res) {
	      if (err) {
	        registry.get(device.deviceId, printDeviceInfo);
	      }
	      if (deviceInfo) {
	        printDeviceInfo(err, deviceInfo, res)
	      }
	    });
   
	    function printDeviceInfo(err, deviceInfo, res) {
	      if (deviceInfo) {
	        console.log('Device id: ' + deviceInfo.deviceId);
	        console.log('Device key: ' + deviceInfo.authentication.symmetricKey.primaryKey);
	      }
	    }
    
7. 保存并关闭 **CreateDeviceIdentity.js** 文件。
8. 若要运行 **createdeviceidentity** 应用程序，请在命令提示符下的 createdeviceidentity 文件夹中执行以下命令：
   
    
	    node CreateDeviceIdentity.js 
    
9. 记下**设备 ID** 和**设备密钥**。稍后在创建作为设备连接到 IoT 中心的应用程序时，需要这些值。

> [AZURE.NOTE] IoT 中心标识注册表只存储设备标识，以启用对中心的安全访问。它存储设备 ID 和密钥作为安全凭据，以及启用或禁用标志（可用于禁用对单个设备的访问）。如果应用程序需要存储其他特定于设备的元数据，则应使用特定于应用程序的存储。有关详细信息，请参阅 [IoT Hub Developer Guide][lnk-devguide-identity]（IoT 中心开发人员指南）。

## 接收设备到云的消息
本部分将创建一个 Node.js 控制台应用，用于读取来自 IoT 中心的设备到云消息。IoT 中心公开与[事件中心][lnk-event-hubs-overview]兼容的终结点，以便你可读取设备到云的消息。为了简单起见，本教程创建的基本读取器不适用于高吞吐量部署。[Process device-to-cloud messages][lnk-process-d2c-tutorial]（处理设备到云的消息）教程介绍了如何大规模处理设备到云的消息。[事件中心入门][lnk-eventhubs-tutorial]教程更详细地介绍了如何处理来自事件中心的消息，此教程也适用于与 IoT 中心事件中心兼容的终结点。

> [AZURE.NOTE] 读取设备到云消息的事件中心兼容终结点始终使用 AMQPS 协议。

1. 新建名为 **readdevicetocloudmessages** 的空文件夹。在 **readdevicetocloudmessages** 文件夹的命令提示符处，使用以下命令创建 package.json 文件。接受所有默认值：
   
    
	    npm init
    
2. 在命令提示符下的 **readdevicetocloudmessages** 文件夹中，运行以下命令以安装 **azure-event-hubs** 包：
   
    
	    npm install azure-event-hubs --save
    
3. 在 **readdevicetocloudmessages** 文件夹中，利用文本编辑器创建 **ReadDeviceToCloudMessages.js** 文件。
4. 在 **ReadDeviceToCloudMessages.js** 文件的开头添加以下 `require` 语句：
   
    
	    'use strict';
   
	    var EventHubClient = require('azure-event-hubs').Client;
    
5. 添加以下变量声明，并将占位符值替换为你的 IoT 中心的连接字符串：
   
    
	    var connectionString = '{iothub connection string}';
    
6. 添加以下两个函数用于在控制台中列显输出：
   
    
	    var printError = function (err) {
	      console.log(err.message);
	    };
   
	    var printMessage = function (message) {
	      console.log('Message received: ');
	      console.log(JSON.stringify(message.body));
	      console.log('');
	    };
    
7. 添加以下代码以创建 **EventHubClient**，打开与 IoT 中心的连接，并为每个分区创建接收器。在创建开始运行后只读取发送到 IoT 中心的消息的接收方时，此应用程序将使用筛选器。此筛选器仅显示当前的消息集，很适合测试环境。在生产环境中，代码应确保它能处理所有消息。有关详细信息，请参阅[如何处理 IoT 中心设备到云的消息][lnk-process-d2c-tutorial]教程：
   
    
	    var client = EventHubClient.fromConnectionString(connectionString);
	    client.open()
	        .then(client.getPartitionIds.bind(client))
	        .then(function (partitionIds) {
	            return partitionIds.map(function (partitionId) {
	                return client.createReceiver('$Default', partitionId, { 'startAfterTime' : Date.now()}).then(function(receiver) {
	                    console.log('Created partition receiver: ' + partitionId)
	                    receiver.on('errorReceived', printError);
	                    receiver.on('message', printMessage);
	                });
	            });
	        })
	        .catch(printError);
    
8. 保存并关闭 **ReadDeviceToCloudMessages.js** 文件。

## 创建模拟设备应用程序
本部分将创建一个 Node.js 控制台应用，用于模拟向 IoT 中心发送设备到云消息的设备。

1. 新建名为 **simulateddevice** 的空文件夹。在 **simulateddevice** 文件夹的命令提示符处，使用以下命令创建 package.json 文件。接受所有默认值：
   
    
	    npm init
    
2. 在 **simulateddevice** 文件夹的命令提示符处，运行下述命令以安装 **azure-iot-device** 设备 SDK 包和 **azure-iot-device-amqp** 包：
   
    
	    npm install azure-iot-device azure-iot-device-amqp --save
    
3. 在 **simulateddevice** 文件夹中，利用文本编辑器创建新的 **SimulatedDevice.js** 文件。
4. 在 **SimulatedDevice.js** 文件的开头添加以下 `require` 语句：
   
    
	    'use strict';
   
	    var clientFromConnectionString = require('azure-iot-device-amqp').clientFromConnectionString;
	    var Message = require('azure-iot-device').Message;
    
5. 添加 **connectionString** 变量，并用其创建设备客户端。将 **{youriothostname}** 替换为在*创建 IoT 中心*部分中创建的 IoT 中心名称。将 **{yourdevicekey}** 替换为在*创建设备标识*部分中生成的设备密钥值：
   
    
	    var connectionString = 'HostName={youriothostname};DeviceId=myFirstNodeDevice;SharedAccessKey={yourdevicekey}';
   
	    var client = clientFromConnectionString(connectionString);
    
6. 添加以下函数以显示应用程序的输出：
   
    
	    function printResultFor(op) {
	      return function printResult(err, res) {
	        if (err) console.log(op + ' error: ' + err.toString());
	        if (res) console.log(op + ' status: ' + res.constructor.name);
	      };
	    }
    

7. 创建回调，使用 **setInterval** 函数每隔一秒向 IoT 中心发送一条新消息：
   
    
	    var connectCallback = function (err) {
	      if (err) {
	        console.log('Could not connect: ' + err);
	      } else {
	        console.log('Client connected');
   
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
    
8. 与 IoT 中心建立连接并开始发送消息：
   
    
	    client.open(connectCallback);
    
9. 保存并关闭 **SimulatedDevice.js** 文件。

> [AZURE.NOTE] 为简单起见，本教程不实现任何重试策略。在生产代码中，你应该按 MSDN 文章 [Transient Fault Handling][lnk-transient-faults]（暂时性故障处理）中所述实施重试策略（例如指数性的回退）。



## 运行应用
现在，已准备就绪，可以运行应用。

1. 在 **readdevicetocloudmessages** 文件夹的命令提示符处，运行下述命令以开始监视 IoT 中心：
   
    
	    node ReadDeviceToCloudMessages.js 
    
   
    ![用于监视设备到云消息的 Node.js IoT 中心服务客户端应用程序][7]  

2. 在 **simulateddevice** 文件夹的命令提示符处，运行下述命令以开始向 IoT 中心发送遥测数据：
   
    
	    node SimulatedDevice.js
    
   
    ![用于发送设备到云消息的 Node.js IoT 中心设备客户端应用程序][8]  

3. [Azure 门户预览][lnk-portal]中的“使用情况”磁贴显示发送到中心的消息数：
   
    ![显示发送到 IoT 中心的消息数量的 Azure 门户“使用情况”磁贴][43]

## 后续步骤
在本教程中，在 Azure 门户中配置了新的 IoT 中心，然后在中心的标识注册表中创建了设备标识。你已使用此设备标识来让模拟设备应用向中心发送设备到云的消息，并创建了用于显示中心所接收消息的应用。

若要继续了解 IoT 中心入门知识并浏览其他 IoT 方案，请参阅：

- [连接你的设备][lnk-connect-device]
- [IoT 网关 SDK 入门][lnk-gateway-SDK]

若要了解如何扩展 IoT 解决方案和如何大规模处理设备到云的消息，请参阅[处理设备到云的消息][lnk-process-d2c-tutorial]教程。

<!-- Images. -->
[6]: ./media/iot-hub-node-node-getstarted/create-iot-hub6.png
[7]: ./media/iot-hub-node-node-getstarted/runapp1.png
[8]: ./media/iot-hub-node-node-getstarted/runapp2.png
[43]: ./media/iot-hub-csharp-csharp-getstarted/usage.png

<!-- Links -->

[lnk-transient-faults]: https://msdn.microsoft.com/zh-cn/library/hh680901(v=pandp.50).aspx

[lnk-eventhubs-tutorial]: /documentation/articles/event-hubs-csharp-ephcs-getstarted/
[lnk-devguide-identity]: /documentation/articles/iot-hub-devguide-identity-registry/
[lnk-event-hubs-overview]: /documentation/articles/event-hubs-overview/


[lnk-process-d2c-tutorial]: /documentation/articles/iot-hub-csharp-csharp-process-d2c/

[lnk-hub-sdks]: /documentation/articles/iot-hub-devguide-sdks/
[lnk-free-trial]: /pricing/1rmb-trial/
[lnk-portal]: https://portal.azure.cn/

[lnk-gateway-SDK]: /documentation/articles/iot-hub-linux-gateway-sdk-get-started/
[lnk-connect-device]: /develop/iot/

<!---HONumber=Mooncake_Quality_Review_1230_2016-->