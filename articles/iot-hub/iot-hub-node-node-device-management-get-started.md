<properties
	pageTitle="设备管理入门"
	description="本教程演示如何开始使用 Azure IoT 中心的设备管理"
	services="iot-hub"
	documentationcenter=".net"
	author="juanjperez"
	manager=" timlt"
	editor=""/>  


<tags
	ms.service="iot-hub"
	ms.date="09/30/2016"
	wacn.date="12/19/2016"/>  


# 教程：设备管理入门
## 介绍
IoT 云应用程序可以使用 Azure IoT 中心中的基元（即设备孪生和直接方法）远程启动和监视设备上的设备管理操作。此文章提供有关 IoT 云应用程序和设备如何使用 IoT 中心协同工作来启动和监视远程设备重新启动的指导和代码。

若要从基于云的后端应用远程启动和监视设备上的设备管理操作，请使用 Azure IoT 中心基元（如[设备孪生][lnk-devtwin]和[直接方法][lnk-c2dmethod]）。本教程说明后端应用和设备如何协同工作，实现从 IoT 中心启动和监视远程设备重新启动。

使用直接方法可以从云中的后端应用启动设备管理操作（如重新启动、恢复出厂设置和固件更新）。设备负责以下操作：

* 处理从 IoT 中心发送的方法请求。
* 在设备上启动相应的设备特定操作。
* 通过设备孪生向 IoT 中心报告的属性，提供状态更新。

可以使用云中的后端应用运行设备孪生查询，以报告设备管理操作的进度。

本教程演示如何：

* 使用 Azure 门户创建 IoT 中心，以及如何在 IoT 中心创建设备标识。
* 创建一个模拟设备，以便云可以调用其直接方法实现重新启动。
* 创建一个控制台应用程序，以便通过 IoT 中心调用模拟设备上的重新启动直接方法。

在本教程结束时，用户有两个 Node.js 控制台应用程序：

**dmpatterns\_getstarted\_device.js**，它使用先前创建的设备标识连接到 IoT 中心，接收重新启动直接方法，模拟物理重新启动，并报告上次重新启动的时间。

**dmpatterns\_getstarted\_service.js**，它在模拟设备上调用直接方法、显示响应以及显示更新的设备孪生报告属性。

若要完成本教程，您需要以下各项：

* Node.js 版本 0.12.x 或更高版本，<br/>[准备开发环境][lnk-dev-setup]介绍了如何在 Windows 或 Linux 上安装本教程所用的 Node.js。
* 有效的 Azure 帐户。（如果没有帐户，只需花费几分钟就能创建一个[帐户][lnk-free-trial]。）

[AZURE.INCLUDE [iot-hub-get-started-create-hub](../../includes/iot-hub-get-started-create-hub.md)]

[AZURE.INCLUDE [iot-hub-get-started-create-device-identity](../../includes/iot-hub-get-started-create-device-identity.md)]

## 创建模拟设备应用程序
在本部分，用户需

  - 创建一个 Node.js 控制台应用，用于响应通过云调用的直接方法
  - 触发模拟的设备重启
  - 使用设备孪生报告的属性，允许通过设备孪生查询标识设备及其上次重启的时间



1. 新建名为 **manageddevice** 的空文件夹。在 **manageddevice** 文件夹的命令提示符处，使用以下命令创建 package.json 文件。接受所有默认值：
   
    ```
    npm init
    ```
2. 在 **manageddevice** 文件夹的命令提示符处，运行下述命令以安装 **azure-iot-device** 设备 SDK 包和 **azure-iot-device-mqtt** 包：
   
    ```
    npm install azure-iot-device azure-iot-device-mqtt --save
    ```
3. 在 **manageddevice** 文件夹中，利用文本编辑器创建新的 **dmpatterns\_getstarted\_device.js** 文件。
4. 在 **dmpatterns\_getstarted\_device.js** 文件开头添加以下“require”语句：
   
    ```
    'use strict';
   
    var Client = require('azure-iot-device').Client;
    var Protocol = require('azure-iot-device-mqtt').Mqtt;
    ```
5. 添加 **connectionString** 变量，并用其创建设备客户端。将连接字符串替换为设备连接字符串。
   
    ```
    var connectionString = 'HostName={youriothostname};DeviceId=myDeviceId;SharedAccessKey={yourdevicekey}';
    var client = Client.fromConnectionString(connectionString, Protocol);
    ```
6. 添加以下函数，实现设备上的直接方法
   
    ```
    var onReboot = function(request, response) {
   
        // Respond the cloud app for the direct method
        response.send(200, 'Reboot started', function(err) {
            if (!err) {
                console.error('An error occured when sending a method response:\n' + err.toString());
            } else {
                console.log('Response to method \'' + request.methodName + '\' sent successfully.');
            }
        });
   
        // Report the reboot before the physical restart
        var date = new Date();
        var patch = {
            iothubDM : {
                reboot : {
                    lastReboot : date.toISOString(),
                }
            }
        };
   
        // Get device Twin
        client.getTwin(function(err, twin) {
            if (err) {
                console.error('could not get twin');
            } else {
                console.log('twin acquired');
                twin.properties.reported.update(patch, function(err) {
                    if (err) throw err;
                    console.log('Device reboot twin state reported')
                });  
            }
        });
   
        // Add your device's reboot API for physical restart.
        console.log('Rebooting!');
    };
    ```
7. 打开与 IoT 中心的连接并启动直接方法侦听器：
   
    ```
    client.open(function(err) {
        if (err) {
            console.error('Could not open IotHub client');
        }  else {
            console.log('Client opened.  Waiting for reboot method.');
            client.onDeviceMethod('reboot', onReboot);
        }
    });
    ```
8. 保存并关闭 **dmpatterns\_getstarted\_device.js** 文件。
   
   [AZURE.NOTE] 为简单起见，本教程不实现任何重试策略。在生产代码中，你应该按 MSDN 文章 [Transient Fault Handling][lnk-transient-faults]（暂时性故障处理）中所述实施重试策略（例如指数性的回退）。

## 使用直接方法在设备上触发远程重新启动
在此部分中，会创建一个 Node.js 控制台应用，它使用直接方法在设备上启动远程重新启动，并使用设备孪生查询找到该设备上次重新启动时间。

1. 新建名为 **triggerrebootondevice** 的空文件夹。在 **triggerrebootondevice** 文件夹的命令提示符处，使用以下命令创建 package.json 文件。接受所有默认值：
   
    ```
    npm init
    ```
2. 在 **triggerrebootondevice** 文件夹的命令提示符处，运行下述命令以安装 **azure-iothub** 设备 SDK 包和 **azure-iot-device-mqtt** 包：
   
    ```
    npm install azure-iothub --save
    ```
3. 在 **triggerrebootondevice** 文件夹中，使用文本编辑器创建新的 **dmpatterns\_getstarted\_service.js** 文件。
4. 在 **dmpatterns\_getstarted\_service.js** 文件开头添加以下“require”语句：
   
    ```
    'use strict';
   
    var Registry = require('azure-iothub').Registry;
    var Client = require('azure-iothub').Client;
    ```
5. 添加以下变量声明并替换占位符值：
   
    ```
    var connectionString = '{iothubconnectionstring}';
    var registry = Registry.fromConnectionString(connectionString);
    var client = Client.fromConnectionString(connectionString);
    var deviceToReboot = 'myDeviceId';
    ```
6. 添加以下函数以调用设备方法来重新启动目标设备：
   
    ```
    var startRebootDevice = function(twin) {
   
        var methodName = "reboot";
   
        var methodParams = {
            methodName: methodName,
            payload: null,
            timeoutInSeconds: 30
        };
   
        client.invokeDeviceMethod(deviceToReboot, methodParams, function(err, result) {
            if (err) { 
                console.error("Direct method error: "+err.message);
            } else {
                console.log("Successfully invoked the device to reboot.");  
            }
        });
    };
    ```
7. 添加以下函数以查询设备并获取上次重新启动时间：
   
    ```
    var queryTwinLastReboot = function() {
   
        registry.getTwin(deviceToReboot, function(err, twin){
   
            if (twin.properties.reported.iothubDM != null)
            {
                if (err) {
                    console.error('Could not query twins: ' + err.constructor.name + ': ' + err.message);
                } else {
                    var lastRebootTime = twin.properties.reported.iothubDM.reboot.lastReboot;
                    console.log('Last reboot time: ' + JSON.stringify(lastRebootTime, null, 2));
                }
            } else 
                console.log('Waiting for device to report last reboot time.');
        });
    };
    ```
8. 添加以下代码以调用函数，将触发重新启动直接方法并查询上次重新启动时间：
   
    ```
    startRebootDevice();
    setInterval(queryTwinLastReboot, 2000);
    ```
9. 保存并关闭 **dmpatterns\_getstarted\_service.js** 文件。

## 运行应用
现在，已准备就绪，可以运行应用。

1. 在 **manageddevice** 文件夹的命令提示符处，运行以下命令以开始侦听重新启动直接方法。
   
    ```
    node dmpatterns_getstarted_device.js
    ```
2. 在 **triggerrebootondevice** 文件夹的命令提示符处，运行以下命令以触发远程重新启动并查询设备孪生以查找上次重新启动时间。
   
    ```
    node dmpatterns_getstarted_service.js
    ```
3. 可以在控制台中看到设备对直接方法的响应。

## 自定义和扩展设备管理操作
IoT 解决方案可以扩展已定义的设备管理模式集，或通过使用设备孪生和直接方法基元启用自定义模式。设备管理操作的其他示例包括恢复出厂设置、固件更新、软件更新、电源管理、网络和连接管理以及数据加密。

## 设备维护时段
通常情况下，将设备配置为在某一时间执行操作，以最大程度减少中断和停机时间。设备维护时段是一种常用模式，用于定义设备应更新其配置的时间。后端解决方案使用设备孪生所需属性在设备上定义并激活策略，以启用维护时段。当设备收到维护时段策略时，它可以使用设备孪生报告属性报告策略状态。然后，后端应用可以使用设备孪生查询来证明设备和每个策略的符合性。

## 后续步骤
在本教程中，使用直接方法来触发设备的远程重新启动，使用设备孪生报告属性来报告设备上次重新启动时间，并查询设备孪生来从云中发现设备上次重新启动时间。

若要继续完成 IoT 中心和设备管理模式（如远程无线固件更新）的入门内容，请参阅：

[教程：如何进行固件更新][lnk-fwupdate]

若要了解如何扩展 IoT 解决方案并在多个设备上计划方法调用，请参阅 [Schedule and broadcast jobs][lnk-tutorial-jobs]（计划和广播作业）教程。

若要继续完成 IoT 中心的入门内容，请参阅 [IoT 网关 SDK 入门][lnk-gateway-SDK]。

<!-- images and links -->

[img-output]: ./media/iot-hub-get-started-with-dm/image6.png
[img-dm-ui]: ./media/iot-hub-get-started-with-dm/dmui.png

[lnk-dev-setup]: https://github.com/Azure/azure-iot-sdks/blob/master/doc/get_started/node-devbox-setup.md

[lnk-free-trial]: /pricing/1rmb-trial/
[lnk-fwupdate]: /documentation/articles/iot-hub-node-node-firmware-update/
[Azure portal]: https://portal.azure.cn/
[Using resource groups to manage your Azure resources]: /documentation/articles/resource-group-portal/
[lnk-dm-github]: https://github.com/Azure/azure-iot-device-management
[lnk-tutorial-jobs]: /documentation/articles/iot-hub-node-node-schedule-jobs/
[lnk-gateway-SDK]: /documentation/articles/iot-hub-linux-gateway-sdk-get-started/

[lnk-devtwin]: /documentation/articles/iot-hub-devguide-device-twins/
[lnk-c2dmethod]: /documentation/articles/iot-hub-devguide-direct-methods/
[lnk-transient-faults]: https://msdn.microsoft.com/zh-cn/library/hh680901(v=pandp.50).aspx

<!---HONumber=Mooncake_1212_2016-->