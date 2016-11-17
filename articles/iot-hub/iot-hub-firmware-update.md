<properties
 pageTitle="如何进行固件更新 | Azure"
 description="本教程演示如何进行固件更新"
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


# 教程：如何进行固件更新（预览版）

## 介绍
在[设备管理入门][lnk-dm-getstarted]教程中，了解了如何使用[设备克隆][lnk-devtwin]和[云到设备 (C2D) 方法][lnk-c2dmethod]基元来远程重新启动设备。本教程使用相同的 IoT 中心基元，提供指南，并演示如何进行端到端模拟固件更新。此模式在用于 Intel Edison 设备示例的固件更新实现中使用。

本教程演示如何：

- 创建一个控制台应用程序，它通过 IoT 中心在模拟设备上调用 firmwareUpdate 直接方法。
- 创建一个模拟设备，它实现的 firmwareUpdate 直接方法会执行等待下载固件映像、下载固件映像以及最后应用固件映像的多阶段过程。在执行每个阶段的整个过程中，设备使用克隆报告属性更新进度。

本教程最后提供两个 Node.js 控制台应用程序：

**dmpatterns\_fwupdate\_service.js**，它在模拟设备上调用直接方法、显示响应以及定期（每 500 毫秒）显示更新的设备克隆报告属性。

**dmpatterns\_fwupdate\_device.js**，它使用早前创建的设备标识连接到 IoT 中心、接收 firmwareUpdate 直接方法、运行一个多状态过程以模拟固件更新，包括：等待映像下载、下载新映像以及最后应用映像。


若要完成本教程，您需要以下各项：

Node.js 版本 0.12.x 或更高版本，<br/>[准备开发环境][lnk-dev-setup]介绍了如何在 Windows 或 Linux 上安装本教程所用的 Node.js。

有效的 Azure 帐户。（如果你没有帐户，只需花费几分钟就能创建一个试用帐户。有关详细信息，请参阅 [Azure 试用][lnk-free-trial]。）

按照[设备管理入门](/documentation/articles/iot-hub-device-management-get-started/)一文创建 IoT 中心并获取连接字符串。

[AZURE.INCLUDE [iot-hub-get-started-create-hub-pp](../../includes/iot-hub-get-started-create-hub-pp.md)]

## 创建模拟设备应用程序

在此部分中，会创建一个 Node.js 控制台应用，它响应云调用的直接方法，这会触发模拟设备固件更新，并使用设备克隆报告属性使设备克隆查询可以识别设备以及它们上次重新启动的时间。

1. 新建名为 **manageddevice** 的空文件夹。在 **manageddevice** 文件夹的命令提示符处，使用以下命令创建 package.json 文件。接受所有默认值：

    ```
    npm init
    ```
    
2. 在命令提示符下的 **manageddevice** 文件夹中，运行以下命令以安装 **azure-iot-device@dtpreview** 设备 SDK 包和 **azure-iot-device-mqtt@dtpreview** 包：

    ```
    npm install azure-iot-device@dtpreview azure-iot-device-mqtt@dtpreview --save
    ```

3. 在 **manageddevice** 文件夹中，利用文本编辑器创建新的 **dmpatterns\_fwupdate\_device.js** 文件。

4. 在 **dmpatterns\_fwupdate\_device.js** 文件开头添加以下“require”语句：

    ```
    'use strict';

    var Client = require('azure-iot-device').Client;
    var Protocol = require('azure-iot-device-mqtt').Mqtt;
    ```

5. 添加 **connectionString** 变量，并用其创建设备客户端。

    ```
    var connectionString = 'HostName={youriothostname};DeviceId=myDeviceId;SharedAccessKey={yourdevicekey}';
    var client = Client.fromConnectionString(connectionString, Protocol);
    ```

6. 添加用于更新克隆报告属性的以下函数

    ```
    var reportFWUpdateThroughTwin = function(twin, firmwareUpdateValue) {
      var patch = {
          iothubDM : {
            firmwareUpdate : firmwareUpdateValue
          }
      };

      twin.properties.reported.update(patch, function(err) {
        if (err) throw err;
        console.log('twin state reported')
      });
    };
    ```
    
7. 添加模拟固件映像的下载和应用的以下函数。

    ```
    var simulateDownloadImage = function(imageUrl, callback) {
      var error = null;
      var image = "[fake image data]";
      
      console.log("Downloading image from " + imageUrl);
      
      callback(error, image);
    }

    var simulateApplyImage = function(imageData, callback) {
      var error = null;
      
      if (!imageData) {
        error = {message: 'Apply image failed because of missing image data.'};
      }
      
      callback(error);
    }
    ```

8. 添加通过克隆报告属性将固件更新状态更新为正在等待下载的以下函数。通常，设备会收到有关可用更新的通知，并且管理员定义的策略会使设备开始下载和应用更新。这是用于启用该策略的逻辑运行的位置。为简单起见，我们会延迟 4 秒，然后继续下载固件映像。

    ```
    var waitToDownload = function(twin, fwPackageUriVal, callback) {
      var now = new Date();
      
      reportFWUpdateThroughTwin(twin, {
        fwPackageUri: fwPackageUriVal,
        status: 'waiting',
        error : null,
        startedWaitingTime : now.toISOString()
      });
      setTimeout(callback, 4000);
    };
    ```
    
9. 添加通过克隆报告属性将固件更新状态更新为正在下载固件映像的以下函数。它会追踪模拟固件下载，最后更新固件更新状态以告知下载成功或失败。

    ```
    var downloadImage = function(twin, fwPackageUriVal, callback) {
      var now = new Date();   
      
      reportFWUpdateThroughTwin(twin, {
        status: 'downloading',
      });
      
      setTimeout(function() {
        // Simulate download
        simulateDownloadImage(fwPackageUriVal, function(err, image) {
          
          if (err)
          {
            reportFWUpdateThroughTwin(twin, {
              status: 'downloadfailed',
              error: {
                code: error_code,
                message: error_message,
              }
            });
          }
          else {        
            reportFWUpdateThroughTwin(twin, {
              status: 'downloadComplete',
              downloadCompleteTime: now.toISOString(),
            });
            
            setTimeout(function() { callback(image); }, 4000);   
          }
        });
        
      }, 4000);
    }
    ```
    
10. 添加通过克隆报告属性将固件更新状态更新为正在应用固件映像的以下函数。它会追踪模拟固件映像的应用，最后更新固件更新状态以告知应用成功或失败。

    ```
    var applyImage = function(twin, imageData, callback) {
      var now = new Date();   
      
      reportFWUpdateThroughTwin(twin, {
        status: 'applying',
        startedApplyingImage : now.toISOString()
      });
      
      setTimeout(function() {
        
        // Simulate apply firmware image
        simulateApplyImage(imageData, function(err) {
          if (err) {
            reportFWUpdateThroughTwin(twin, {
              status: 'applyFailed',
              error: {
                code: err.error_code,
                message: err.error_message,
              }
            });
          } else { 
            reportFWUpdateThroughTwin(twin, {
              status: 'applyComplete',
              lastFirmwareUpdate: now.toISOString()
            });    
            
          }
        });
        
        setTimeout(callback, 4000);
        
      }, 4000);
    }
    ```

11. 添加处理 firmwareUpdate 方法并启动多阶段固件更新过程的以下函数。

    ```
    var onFirmwareUpdate = function(request, response) {
            
      // Respond the cloud app for the direct method
      response.send(200, 'FirmwareUpdate started', function(err) {
        if (!err) {
          console.error('An error occured when sending a method response:\n' + err.toString());
        } else {
          console.log('Response to method \'' + request.methodName + '\' sent successfully.');
        }
      });

      // Get the parameter from the body of the method request
      var fwPackageUri = JSON.parse(request.payload).fwPackageUri;

      // Obtain the device twin
      client.getTwin(function(err, twin) {
        if (err) {
          console.error('Could not get device twin.');
        } else {
          console.log('Device twin acquired.');
          
          // Start the multi-stage firmware update
          waitToDownload(twin, fwPackageUri, function() {
            downloadImage(twin, fwPackageUri, function(imageData) {
              applyImage(twin, imageData, function() {});    
            });  
          });

        }
      });
    }
    ```
    
12. 最后，添加作为设备连接到 IoT 中心的以下代码。

    ```
    client.open(function(err) {
      if (err) {
        console.error('Could not connect to IotHub client');
      }  else {
        console.log('Client connected to IoT Hub.  Waiting for firmwareUpdate direct method.');
      }
      
      client.onDeviceMethod('firmwareUpdate', onFirmwareUpdate(request, response));
    });
    ```

> [AZURE.NOTE] 为简单起见，本教程不实现任何重试策略。在生产代码中，你应该按 MSDN 文章 [Transient Fault Handling][lnk-transient-faults]（暂时性故障处理）中所述实施重试策略（例如指数性的回退）。

## 使用直接方法在设备上触发远程固件更新 

在此部分中，会创建一个 Node.js 控制台应用，它使用直接方法在设备上启动远程固件更新，并使用设备克隆查询定期获取该设备上活动固件更新的状态。


1. 新建名为 **triggerfwupdateondevice** 的空文件夹。在 **triggerfwupdateondevice** 文件夹的命令提示符处，使用以下命令创建 package.json 文件。接受所有默认值：

    ```
    npm init
    ```
    
2. 在 **triggerfwupdateondevice** 文件夹的命令提示符处，运行下述命令以安装 **azure-iothub** 设备 SDK 包和 **azure-iot-device-mqtt** 包：

    ```
    npm install azure-iot-hub@dtpreview --save
    ```
    
3. 在 **triggerfwupdateondevice** 文件夹中，利用文本编辑器创建新的 **dmpatterns\_getstarted\_service.js** 文件。

4. 在 **dmpatterns\_getstarted\_service.js** 文件开头添加以下“require”语句：

    ```
    'use strict';

    var Registry = require('azure-iothub').Registry;
    var Client = require('azure-iothub').Client;
    ```

5. 添加以下变量声明并替换占位符值：

    ```
    var connectionString = '{device_connectionstring}';
    var registry = Registry.fromConnectionString(connectionString);
    var client = Client.fromConnectionString(connectionString);
    var deviceToUpdate = 'myDeviceId';
    ```
    
6. 添加以下函数以查找并显示 firmwareUpdate 报告属性的值。

    ```
    var queryTwinFWUpdateReported = function() {
        registry.getTwin(deviceToUpdate, function(err, twin){
            if (err) {
              console.error('Could not query twins: ' + err.constructor.name + ': ' + err.message);
            } else {
              console.log((JSON.stringify(twin.properties.reported.iothubDM.firmwareUpdate)) + "\n");
            }
        });
    };
    ```

7. 添加以下函数以调用 firmwareUpdate 方法来重新启动目标设备：

    ```
    var startFirmwareUpdateDevice = function() {
      var params = {
          fwPackageUri: 'https://secureurl'
      };
      
      var methodName = "firmwareUpdate";
      var payloadData =  JSON.stringify(params);
      
      var methodParams = {
        methodName: methodName,
        payload: payloadData,
        timeoutInSeconds: 30
      };
      
      client.invokeDeviceMethod(deviceToUpdate, methodParams, function(err, result) {
        if (err) {
          console.error('Could not start the firmware update on the device: ' + err.message)
        } 
      });
    };
    ```

8. 最后，向代码添加以下函数以启动固件更新序列并开始定期显示克隆报告属性：

    ```
    startFirmwareUpdateDevice();
    setInterval(queryTwinFWUpdateReported, 500);
    ```
    
9. 保存并关闭 **dmpatterns\_fwupdate\_service.js** 文件。

## 运行应用程序

现在，你已准备就绪，可以运行应用程序了。

1. 在 **manageddevice** 文件夹的命令提示符处，运行以下命令以开始侦听重新启动直接方法。

    ```
    node dmpatterns_fwupdate_device.js
    ```

2. 在 **triggerfwupdateondevice** 文件夹的命令提示符处，运行以下命令以触发远程重新启动并查询设备克隆以查找上次重新启动时间。

    ```
    node dmpatterns_fwupdate_service.js
    ```

3. 可通过打印出消息查看对直接方法的反应

## 后续步骤

在本教程中，使用了直接方法在设备上触发远程固件更新，并定期使用克隆报告属性了解固件更新过程的进度。

若要了解如何扩展 IoT 解决方案并在多个设备上安排方法调用，请参阅[安排和广播作业][lnk-tutorial-jobs]教程。

[lnk-devtwin]: /documentation/articles/iot-hub-devguide-device-twins/
[lnk-c2dmethod]: /documentation/articles/iot-hub-devguide-direct-methods/
[lnk-dm-getstarted]: /documentation/articles/iot-hub-device-management-get-started/
[lnk-tutorial-jobs]: /documentation/articles/iot-hub-schedule-jobs/

[lnk-dev-setup]: https://github.com/Azure/azure-iot-sdks/blob/master/doc/get_started/node-devbox-setup.md
[lnk-free-trial]: /pricing/1rmb-trial/
[lnk-transient-faults]: https://msdn.microsoft.com/zh-cn/library/hh680901(v=pandp.50).aspx

<!---HONumber=Mooncake_1031_2016-->