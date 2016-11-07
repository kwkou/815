<properties
 pageTitle="如何安排作业 | Azure"
 description="本教程演示如何安排作业"
 services="iot-hub"
 documentationCenter=".net"
 authors="juanjperez"
 manager="timlt"
 editor=""/>  


<tags
 ms.service="iot-hub"
 ms.devlang="multiple"
 ms.topic="article"
 ms.tgt_pltfrm="na"
 ms.workload="na"
 ms.date="09/30/2016" 
 ms.author="juanpere"
 wacn.date="11/07/2016"/>  


# 教程：安排和广播作业（预览版）

## 介绍
Azure IoT 中心是一个完全托管的服务，使应用程序后端可以创建和跟踪用于安排和更新数百万个设备的作业。作业可以用于以下操作：

- 更新设备克隆所需属性
- 更新设备克隆标记
- 调用云到设备的方法

从概念上讲，作业包装这些操作之一，并跟踪针对一组设备进行的执行（由克隆查询定义）的进度。例如，通过使用作业，应用程序后端可以对 10,000 个设备调用重新启动方法（由克隆查询指定并安排在将来执行）。该应用程序随后可以在其中每个设备接收和执行重新启动方法时跟踪进度。

可在以下文章中了解有关所有这些功能的详细信息：

- 设备克隆和属性：[克隆入门][lnk-get-started-twin]和[教程：如何使用克隆属性][lnk-twin-props]
- 云到设备的方法：[开发人员指南 - 直接方法][lnk-dev-methods]和[教程：C2D 方法][lnk-c2d-methods]

本教程演示如何：

- 创建一个模拟设备，它具有实现可以由应用程序后端调用的 lockDoor 的直接方法。
- 创建一个控制台应用程序，它使用作业对模拟设备调用 lockDoor 直接方法，并使用设备作业更新克隆所需属性。

本教程最后提供两个 Node.js 控制台应用程序：

**simDevice.js**，它使用设备标识连接到 IoT 中心，并接收 lockDoor 直接方法。

**scheduleJobService.js**，它对模拟设备调用直接方法，并使用作业更新克隆的所需属性。

若要完成本教程，您需要以下各项：

Node.js 版本 0.12.x 或更高版本，<br/>[准备开发环境][lnk-dev-setup]介绍了如何在 Windows 或 Linux 上安装本教程所用的 Node.js。

有效的 Azure 帐户。（如果你没有帐户，可以创建一个试用帐户，只需几分钟即可完成。有关详细信息，请参阅 [Azure 试用][lnk-free-trial]。）

[AZURE.INCLUDE [iot-hub-get-started-create-hub-pp](../../includes/iot-hub-get-started-create-hub-pp.md)]

## 创建模拟设备应用程序

在此部分中，会创建一个 Node.js 控制台应用，它响应云调用的直接方法，这会触发模拟设备重新启动，并使用设备克隆报告属性使设备克隆查询可以识别设备以及它们上次重新启动的时间。

1. 新建名为 **simDevice** 的空文件夹。在 **simDevice** 文件夹的命令提示符处，使用以下命令创建 package.json 文件。接受所有默认值：

    ```
    npm init
    ```
    
2. 在 **simDevice** 文件夹的命令提示符处，运行下述命令以安装 **azure-iot-device** 设备 SDK 包和 **azure-iot-device-mqtt** 包：

    ```
    npm install azure-iot-device@dtpreview azure-iot-device-mqtt@dtpreview --save
    ```

3. 在 **simDevice** 文件夹中，利用文本编辑器创建新的 **simDevice.js** 文件。

4. 在 **simDevice.js** 文件的开头添加以下“require”语句：

    ```
    'use strict';

    var Client = require('azure-iot-device').Client;
    var Protocol = require('azure-iot-device-mqtt').Mqtt;
    ```

5. 添加 **connectionString** 变量，并用其创建设备客户端。

    ```
    var connectionString = 'HostName={youriothostname};DeviceId={yourdeviceid};SharedAccessKey={yourdevicekey}';
    var client = Client.fromConnectionString(connectionString, Protocol);
    ```

6. 添加以下函数以处理 lockDoor 方法。

    ```
    var onLockDoor = function(request, response) {
        
        // Respond the cloud app for the direct method
        response.send(200, function(err) {
            if (!err) {
                console.error('An error occured when sending a method response:\n' + err.toString());
            } else {
                console.log('Response to method \'' + request.methodName + '\' sent successfully.');
            }
        });
        
        console.log('Locking Door!');
    };
    ```

7. 添加以下代码以注册 lockDoor 方法的处理程序。

    ```
    client.open(function(err) {
        if (err) {
            console.error('Could not connect to IotHub client.');
        }  else {
            console.log('Client connected to IoT Hub.  Waiting for reboot direct method.');
            client.onDeviceMethod('lockDoor', onLockDoor);
        }
    });
    ```
    
8. 保存并关闭 **simDevice.js** 文件。

> [AZURE.NOTE] 为简单起见，本教程不实现任何重试策略。在生产代码中，你应该按 MSDN 文章 [Transient Fault Handling][lnk-transient-faults]（暂时性故障处理）中所述实施重试策略（例如指数性的回退）。

## 安排作业以便调用直接方法并更新克隆的属性

在此部分中，会创建一个 Node.js 控制台应用，它使用直接方法对设备启动远程 lockDoor 并更新克隆的属性。

1. 新建名为 **scheduleJobService** 的空文件夹。在 **scheduleJobService** 文件夹的命令提示符处，使用以下命令创建 package.json 文件。接受所有默认值：

    ```
    npm init
    ```
    
2. 在 **scheduleJobService** 文件夹的命令提示符处，运行下述命令以安装 **azure-iothub** 设备 SDK 包和 **azure-iot-device-mqtt** 包：

    ```
    npm install azure-iothub@dtpreview uuid --save
    ```
    
3. 在 **scheduleJobService** 文件夹中，利用文本编辑器创建新的 **scheduleJobService.js** 文件。

4. 在 **dmpatterns\_gscheduleJobServiceetstarted\_service.js** 文件开头添加以下“require”语句：

    ```
    'use strict';

    var uuid = require('uuid');
    var JobClient = require('azure-iothub').JobClient;
    ```

5. 添加以下变量声明并替换占位符值：

    ```
    var connectionString = '{iothubconnectionstring}';
    var deviceArray = ['myDeviceId'];
    var startTime = new Date();
    var maxExecutionTimeInSeconds =  3600;
    var jobClient = JobClient.fromConnectionString(connectionString);
    ```
    
6. 添加用于监视作业执行的以下函数：

    ```
    function monitorJob (jobId, callback) {
        var jobMonitorInterval = setInterval(function() {
            jobClient.getJob(jobId, function(err, result) {
            if (err) {
                console.error('Could not get job status: ' + err.message);
            } else {
                console.log('Job: ' + jobId + ' - status: ' + result.status);
                if (result.status === 'completed' || result.status === 'failed' || result.status === 'cancelled') {
                clearInterval(jobMonitorInterval);
                callback(null, result);
                }
            }
            });
        }, 5000);
    }
    ```

7. 添加以下代码以安排调用设备方法的作业：

    ```
    var methodParams = {
        methodName: 'lockDoor',
        payload: null,
        timeoutInSeconds: 45
    };

    var methodJobId = uuid.v4();
    console.log('scheduling Device Method job with id: ' + methodJobId);
    jobClient.scheduleDeviceMethod(methodJobId,
                                deviceArray,
                                methodParams,
                                startTime,
                                maxExecutionTimeInSeconds,
                                function(err) {
        if (err) {
            console.error('Could not schedule device method job: ' + err.message);
        } else {
            monitorJob(methodJobId, function(err, result) {
                if (err) {
                    console.error('Could not monitor device method job: ' + err.message);
                } else {
                    console.log(JSON.stringify(result, null, 2));
                }
            });
        }
    });
    ```
        
8. 添加以下代码以安排更新克隆的作业：

    ```
    var twinPatch = {
        etag: '*',
        desired: {
            building: '43',
            floor: 3
        }
    };

    var twinJobId = uuid.v4();

    console.log('scheduling Twin Update job with id: ' + twinJobId);
    jobClient.scheduleTwinUpdate(twinJobId,
                                deviceArray,
                                twinPatch,
                                startTime,
                                maxExecutionTimeInSeconds,
                                function(err) {
        if (err) {
            console.error('Could not schedule twin update job: ' + err.message);
        } else {
            monitorJob(twinJobId, function(err, result) {
                if (err) {
                    console.error('Could not monitor twin update job: ' + err.message);
                } else {
                    console.log(JSON.stringify(result, null, 2));
                }
            });
        }
    });
    ```
    
9. 保存并关闭 **scheduleJobService.js** 文件。

## 运行应用程序

现在，你已准备就绪，可以运行应用程序了。

1. 在 **simDevice** 文件夹的命令提示符处，运行以下命令以开始侦听重新启动直接方法。

    ```
    node simDevice.js
    ```

2. 在 **scheduleJobService** 文件夹的命令提示符处，运行以下命令以触发远程重新启动并查询设备克隆以查找上次重新启动时间。

    ```
    node scheduleJobService.js
    ```

3. 你会看到来自设备和后端应用程序的输出。


## 后续步骤

在本教程中，使用了作业来安排用于设备的直接方法以及设备克隆属性的更新。

若要继续完成 IoT 中心和设备管理模式（如远程无线固件更新）的入门内容，请参阅：

[教程：如何进行固件更新][lnk-fwupdate]

若要继续完成 IoT 中心的入门内容，请参阅[网关 SDK 入门][lnk-gateway-SDK]。

[lnk-get-started-twin]: /documentation/articles/iot-hub-node-node-twin-getstarted/
[lnk-twin-props]: /documentation/articles/iot-hub-node-node-twin-how-to-configure/
[lnk-c2d-methods]: /documentation/articles/iot-hub-c2d-methods/
[lnk-dev-methods]: /documentation/articles/iot-hub-devguide-direct-methods/
[lnk-fwupdate]: /documentation/articles/iot-hub-firmware-update/
[lnk-gateway-SDK]: /documentation/articles/iot-hub-linux-gateway-sdk-get-started/
[lnk-dev-setup]: https://github.com/Azure/azure-iot-sdks/blob/master/doc/get_started/node-devbox-setup.md
[lnk-free-trial]: /pricing/1rmb-trial/
[lnk-transient-faults]: https://msdn.microsoft.com/zh-cn/library/hh680901(v=pandp.50).aspx

<!---HONumber=Mooncake_1031_2016-->