<properties
   pageTitle="使用 Node.js 连接设备 | Azure"
   description="介绍如何使用以 Node.js 编写的应用程序将设备连接到 Azure IoT 套件预配置远程监视解决方案。"
   services=""
   suite="iot-suite"
   documentationCenter="na"
   authors="dominicbetts"
   manager="timlt"
   editor=""/>  


<tags
   ms.service="iot-suite"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="na"
   ms.date="02/20/2017"
   ms.author="dobett"
   wacn.date="03/28/2017"/>  



# 将设备连接到远程监视预配置解决方案 (Node.js)

[AZURE.INCLUDE [iot-suite-selector-connecting](../../includes/iot-suite-selector-connecting.md)]

## 创建 node.js 示例解决方案

请确保已在开发计算机上安装 Node.js 0.10.x 或更高版本。若要检查版本，可在命令行中运行 `node --version`。

1. 在开发计算机上创建名为 **RemoteMonitoring** 的文件夹。在命令行环境中导航到此文件夹。

1. 运行以下命令下载并安装完成示例应用所需的包：


	    npm init
	    npm install azure-iot-device azure-iot-device-mqtt --save


1. 在 **RemoteMonitoring** 文件夹中创建名为 **remote\_monitoring.js** 的文件。在文本编辑器中打开此文件。

1. 在 **remote\_monitoring.js** 文件中添加以下 `require` 语句：


	    'use strict';

	    var Protocol = require('azure-iot-device-mqtt').Mqtt;
	    var Client = require('azure-iot-device').Client;
	    var ConnectionString = require('azure-iot-device').ConnectionString;
	    var Message = require('azure-iot-device').Message;


1. 在 `require` 语句之后添加以下变量声明。将占位符值 [Device Id] 和 [Device Key] 替换为在远程监视解决方案仪表板中记下的设备值。使用解决方案仪表板中的 IoT 中心主机名替换 [IoTHub Name]。例如，如果 IoT 中心主机名是 **contoso.azure-devices.cn**，则将 [IoTHub Name] 替换为 **contoso**：


	    var connectionString = 'HostName=[IoTHub Name].azure-devices.cn;DeviceId=[Device Id];SharedAccessKey=[Device Key]';
	    var deviceId = ConnectionString.parse(connectionString).DeviceId;


1. 添加以下变量用于定义一些基本遥测数据：


	    var temperature = 50;
	    var humidity = 50;
	    var externalTemperature = 55;


1. 添加以下帮助器函数用于列显操作结果：

    
	    function printErrorFor(op) {
	        return function printError(err) {
	            if (err) console.log(op + ' error: ' + err.toString());
	        };
	    }
    

1. 添加以下帮助器函数用于随机化遥测值：

    
	    function generateRandomIncrement() {
	        return ((Math.random() * 2) - 1);
	    }
    

1. 为设备在启动时发送的 **DeviceInfo** 对象添加以下定义：

    
	    var deviceMetaData = {
	        'ObjectType': 'DeviceInfo',
	        'IsSimulatedDevice': 0,
	        'Version': '1.0',
	        'DeviceProperties': {
	            'DeviceID': deviceId,
	            'HubEnabledState': 1
	        }
	    };
    

1. 为设备孪生报告的值添加以下定义。此定义包括设备支持的直接方法的说明：

    
	    var reportedProperties = {
	        "Device": {
	            "DeviceState": "normal",
	            "Location": {
	                "Latitude": 47.642877,
	                "Longitude": -122.125497
	            }
	        },
	        "Config": {
	            "TemperatureMeanValue": 56.7,
	            "TelemetryInterval": 45
	        },
	        "System": {
	            "Manufacturer": "Contoso Inc.",
	            "FirmwareVersion": "2.22",
	            "InstalledRAM": "8 MB",
	            "ModelNumber": "DB-14",
	            "Platform": "Plat 9.75",
	            "Processor": "i3-9",
	            "SerialNumber": "SER99"
	        },
	        "Location": {
	            "Latitude": 47.642877,
	            "Longitude": -122.125497
	        },
	        "SupportedMethods": {
	            "Reboot": "Reboot the device",
	            "InitiateFirmwareUpdate--FwPackageURI-string": "Updates device Firmware. Use parameter FwPackageURI to specifiy the URI of the firmware file"
	        },
	    }
    

1. 添加以下函数用于处理 **Reboot** 直接方法调用：

    
	    function onReboot(request, response) {
	        // Implement actual logic here.
	        console.log('Simulated reboot...');

	        // Complete the response
	        response.send(200, "Rebooting device", function(err) {
	            if(!!err) {
	                console.error('An error ocurred when sending a method response:\n' + err.toString());
	            } else {
	                console.log('Response to method \'' + request.methodName + '\' sent successfully.' );
	            }
	        });
	    }
    

1. 添加以下函数用于处理 **InitiateFirmwareUpdate** 直接方法调用。此直接方法使用参数指定要下载的固件映像的位置，并在设备上异步启动固件更新：

    
	    function onInitiateFirmwareUpdate(request, response) {
	        console.log('Simulated firmware update initiated, using: ' + request.payload.FwPackageURI);

	        // Complete the response
	        response.send(200, "Firmware update initiated", function(err) {
	            if(!!err) {
	                console.error('An error ocurred when sending a method response:\n' + err.toString());
	            } else {
	                console.log('Response to method \'' + request.methodName + '\' sent successfully.' );
	            }
	        });

	        // Add logic here to perform the firmware update asynchronously
	    }
    

1. 添加以下代码用于创建客户端实例：

    
	    var client = Client.fromConnectionString(connectionString, Protocol);
    

1. 添加以下代码来执行下述操作：

   * 打开连接。
   * 发送 **DeviceInfo** 对象。
   * 设置所需属性的处理程序。
   * 发送报告的属性。
   * 为直接方法注册处理程序。
   * 开始发送遥测数据。

    
    	    client.open(function (err) {
    	        if (err) {
    	            printErrorFor('open')(err);
    	        } else {
    	            console.log('Sending device metadata:\n' + JSON.stringify(deviceMetaData));
    	            client.sendEvent(new Message(JSON.stringify(deviceMetaData)), printErrorFor('send metadata'));
    
    	            // Create device twin
    	            client.getTwin(function(err, twin) {
    	                if (err) {
    	                    console.error('Could not get device twin');
    	                } else {
    	                    console.log('Device twin created');
    
    	                    twin.on('properties.desired', function(delta) {
    	                        console.log('Received new desired properties:');
    	                        console.log(JSON.stringify(delta));
    	                    });
    
    	                    // Send reported properties
    	                    twin.properties.reported.update(reportedProperties, function(err) {
    	                        if (err) throw err;
    	                        console.log('twin state reported');
    	                    });
    
    	                    // Register handlers for direct methods
    	                    client.onDeviceMethod('Reboot', onReboot);
    	                    client.onDeviceMethod('InitiateFirmwareUpdate', onInitiateFirmwareUpdate);
    	                }
    	            });
    
    	            // Start sending telemetry
    	            var sendInterval = setInterval(function () {
    	                temperature += generateRandomIncrement();
    	                externalTemperature += generateRandomIncrement();
    	                humidity += generateRandomIncrement();
    
    	                var data = JSON.stringify({
    	                    'DeviceID': deviceId,
    	                    'Temperature': temperature,
    	                    'Humidity': humidity,
    	                    'ExternalTemperature': externalTemperature
    	                });
    
    	                console.log('Sending device event data:\n' + data);
    	                client.sendEvent(new Message(data), printErrorFor('send event'));
    	            }, 5000);
    
    	            client.on('error', function (err) {
    	                printErrorFor('client')(err);
    	                if (sendInterval) clearInterval(sendInterval);
    	                client.close(printErrorFor('client.close'));
    	            });
    	        }
    	    });
    

1. 保存对 **remote\_monitoring.js** 文件所做的更改。

1. 在命令提示符下运行以下命令，启动示例应用程序：
   
    
	    node remote_monitoring.js
    

[AZURE.INCLUDE [iot-suite-visualize-connecting](../../includes/iot-suite-visualize-connecting.md)]

[lnk-github-repo]: https://github.com/azure/azure-iot-sdk-node
[lnk-github-prepare]: https://github.com/Azure/azure-iot-sdk-node/blob/master/doc/node-devbox-setup.md

<!---HONumber=Mooncake_0306_2017-->