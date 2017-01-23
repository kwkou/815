<properties
    pageTitle="通过 Azure IoT 中心进行设备固件更新 (.NET/Node) | Azure"
    description="如何使用 Azure IoT 中心上的设备管理进行设备固件更新。使用适用于 Node.js 的 Azure IoT 设备 SDK 实现模拟设备应用，并使用适用于 .NET 的 Azure IoT 服务 SDK 实现可触发固件更新的服务应用。"
    services="iot-hub"
    documentationcenter=".net"
    author="juanjperez"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="70b84258-bc9f-43b1-b7cf-de1bb715f2cf"
    ms.service="iot-hub"
    ms.devlang="multiple"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="11/17/2016"
    wacn.date="01/13/2017"
    ms.author="juanpere" />  


# 使用设备管理启动设备固件更新 (.NET/Node)
[AZURE.INCLUDE [iot-hub-selector-firmware-update](../../includes/iot-hub-selector-firmware-update.md)]

## 介绍
在[设备管理入门][lnk-dm-getstarted]教程中，了解了如何使用[设备孪生][lnk-devtwin]和[直接方法][lnk-c2dmethod]基元来远程重新启动设备。本教程使用相同的 IoT 中心基元，介绍如何进行端到端模拟固件更新。

本教程演示如何：

* 创建一个 .NET 控制台应用，其通过 IoT 中心在模拟设备应用上调用 firmwareUpdate 直接方法。
* 创建一个模拟设备应用，它实现的 firmwareUpdate 直接方法会执行等待下载固件映像、下载固件映像以及最后应用固件映像的多阶段过程。在执行各阶段的整个过程中，设备使用报告属性更新进度。

本教程结束时，用户会有一个 Node.js 控制台设备应用，以及一个 .NET (C#) 控制台后端应用：

**dmpatterns\_fwupdate\_service.js**，它在模拟设备应用上调用直接方法、显示响应并定期（每 500 毫秒）显示更新后的报告属性。

**TriggerFWUpdate**：使用早前创建的设备标识连接到 IoT 中心、接收 firmwareUpdate 直接方法、运行一个多状态过程以模拟固件更新，包括：等待映像下载、下载新映像以及最后应用映像。

若要完成本教程，您需要以下各项：

* Microsoft Visual Studio 2015。
* Node.js 版本 0.12.x 或更高版本，<br/>
* 有效的 Azure 帐户。（如果没有帐户，只需花费几分钟就能创建一个[帐户][lnk-free-trial]。）

按照[设备管理入门](/documentation/articles/iot-hub-csharp-node-device-management-get-started/)一文创建 IoT 中心并获取连接字符串。

[AZURE.INCLUDE [iot-hub-get-started-create-hub](../../includes/iot-hub-get-started-create-hub.md)]

[AZURE.INCLUDE [iot-hub-get-started-create-device-identity](../../includes/iot-hub-get-started-create-device-identity.md)]

## 使用直接方法在设备上触发远程固件更新
在此部分中，会创建一个 .NET 控制台应用（使用 C#），它使用直接方法在设备上启动远程固件更新，并使用设备孪生查询定期获取该设备上活动固件更新的状态。

1. 在 Visual Studio 中，使用“控制台应用程序”项目模板将 Visual C# Windows 经典桌面项目添加到当前解决方案。将项目命名为 **TriggerFWUpdate**。

    ![新的 Visual C# Windows 经典桌面项目][img-createapp]  


2. 在解决方案资源管理器中，右键单击“TriggerFWUpdate”项目，然后单击“管理 NuGet 包”。
3. 在“NuGet 包管理器”窗口中，选择“浏览”，搜索 **microsoft.azure.devices**，选择“安装”以安装 **Microsoft.Azure.Devices** 包，然后接受使用条款。此过程将下载、安装 [Azure IoT 服务 SDK][lnk-nuget-service-sdk] NuGet 包及其依赖项并添加对它的引用。

    ![“NuGet 包管理器”窗口][img-servicenuget]  

4. 在 **Program.cs** 文件顶部添加以下 `using` 语句：
   
        using Microsoft.Azure.Devices;
        
5. 将以下字段添加到 **Program** 类。将多个占位符值替换为在上一部分为中心创建的 IoT 中心连接字符串。
   
        static RegistryManager registryManager;
        static string connString = "{iot hub connection string}";
        static ServiceClient client;
        static JobClient jobClient;
        static string targetDevice = "{deviceIdForTargetDevice}";
        
6. 将以下方法添加到 **Program** 类：
   
        public static async Task QueryTwinFWUpdateReported()
        {
            Twin twin = await registryManager.GetTwinAsync(targetDevice);
            Console.WriteLine(twin.Properties.Reported.ToJson());
        }
        
7. 将以下方法添加到 **Program** 类：

        public static async Task StartFirmwareUpdate()
        {
            client = ServiceClient.CreateFromConnectionString(connString);
            CloudToDeviceMethod method = new CloudToDeviceMethod("firmwareUpdate");
            method.ResponseTimeout = TimeSpan.FromSeconds(30);
            method.SetPayloadJson(
                @"{
                    fwPackageUri : 'https://someurl'
                }");

            CloudToDeviceMethodResult result = await client.InvokeDeviceMethodAsync(targetDevice, method);

            Console.WriteLine("Invoked firmware update on device.");
        }

7. 最后，在 **Main** 方法中添加以下行：
   
        registryManager = RegistryManager.CreateFromConnectionString(connString);
        StartFirmwareUpdate().Wait();
        QueryTwinFWUpdateReported().Wait();
        Console.WriteLine("Press ENTER to exit.");
        Console.ReadLine();
        
8. 生成解决方案。

## 创建模拟设备应用程序
在本部分，用户需

* 创建一个 Node.js 控制台应用，用于响应通过云调用的直接方法
* 触发模拟的固件更新
* 使用报告属性，允许通过设备孪生查询标识设备及其上次完成固件更新的时间



1. 新建名为 **manageddevice** 的空文件夹。在 **manageddevice** 文件夹的命令提示符处，使用以下命令创建 package.json 文件。接受所有默认值：
   
    
        npm init
    
2. 在 **manageddevice** 文件夹的命令提示符处，运行下述命令以安装 **azure-iot-device** 设备 SDK 包和 **azure-iot-device-mqtt** 包：
   
    
        npm install azure-iot-device azure-iot-device-mqtt --save
    
3. 在 **manageddevice** 文件夹中，利用文本编辑器创建新的 **dmpatterns\_fwupdate\_device.js** 文件。
4. 在 **dmpatterns\_fwupdate\_device.js** 文件开头添加以下“require”语句：
   
    
        'use strict';
       
        var Client = require('azure-iot-device').Client;
        var Protocol = require('azure-iot-device-mqtt').Mqtt;
    
5. 添加 **connectionString** 变量，并使用它创建**客户端**实例。
   
    
        var connectionString = 'HostName={youriothostname};DeviceId=myDeviceId;SharedAccessKey={yourdevicekey}';
        var client = Client.fromConnectionString(connectionString, Protocol);
    
6. 添加用于更新报告属性的下述函数
   
    
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
    
7. 添加以下函数，模拟固件映像的下载和应用。
   
    
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
    
8. 添加以下函数，它通过报告属性将固件更新状态更新为“正在等待下载”。通常，设备会收到有关可用更新的通知，并且管理员定义的策略会使设备开始下载和应用更新。这是用于启用该策略的逻辑运行的位置。为简单起见，我们会延迟 4 秒，然后继续下载固件映像。
   
    
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
    
9. 添加以下函数，它通过报告属性将固件更新状态更新为“正在下载固件映像”。它会追踪模拟固件下载，最后更新固件更新状态以告知下载成功或失败。
   
    
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
    
10. 添加以下函数，它通过报告属性将固件更新状态更新为“正在应用固件映像”。它会追踪模拟固件映像的应用，最后更新固件更新状态以告知应用成功或失败。
    
    
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
    
11. 添加以下函数，处理 **firmwareUpdate** 方法并启动多阶段固件更新过程。
    
    
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
    
12. 最后，添加以下代码，以便作为设备连接到 IoT 中心。
    
    
        client.open(function(err) {
          if (err) {
            console.error('Could not connect to IotHub client');
          }  else {
            console.log('Client connected to IoT Hub.  Waiting for firmwareUpdate direct method.');
          }
        
          client.onDeviceMethod('firmwareUpdate', onFirmwareUpdate(request, response));
        });
    

> [AZURE.NOTE]
为简单起见，本教程不实现任何重试策略。在生产代码中，你应该按 MSDN 文章 [Transient Fault Handling][lnk-transient-faults]（暂时性故障处理）中所述实施重试策略（例如指数性的回退）。
> 
> 

## 运行应用
现在，已准备就绪，可以运行应用。

1. 在 **manageddevice** 文件夹的命令提示符处，运行以下命令以开始侦听重新启动直接方法。
   
    
        node dmpatterns_fwupdate_device.js
        
2. 运行 C# 控制台应用 **TriggerFWUpdate** - 右键单击 **TriggerFWUpdate** 项目，选择“调试”和“启动新实例”。

3. 可以在控制台中看到设备对直接方法的响应。

## 后续步骤
在本教程中，直接触发了设备的远程固件更新，并定期使用报告属性了解了固件更新过程的进度。

若要了解如何扩展 IoT 解决方案并在多个设备上计划方法调用，请参阅 [Schedule and broadcast jobs][lnk-tutorial-jobs]（计划和广播作业）教程。

<!-- images -->

[img-servicenuget]: ./media/iot-hub-csharp-node-firmware-update/servicesdknuget.png
[img-createapp]: ./media/iot-hub-csharp-node-firmware-update/createnetapp.png

[lnk-devtwin]: /documentation/articles/iot-hub-devguide-device-twins/
[lnk-c2dmethod]: /documentation/articles/iot-hub-devguide-direct-methods/
[lnk-dm-getstarted]: /documentation/articles/iot-hub-node-node-device-management-get-started/
[lnk-tutorial-jobs]: /documentation/articles/iot-hub-node-node-schedule-jobs/

[lnk-free-trial]: /pricing/1rmb-trial/
[lnk-transient-faults]: https://msdn.microsoft.com/zh-cn/library/hh680901(v=pandp.50).aspx
[lnk-nuget-service-sdk]: https://www.nuget.org/packages/Microsoft.Azure.Devices/

<!---HONumber=Mooncake_0109_2017-->
<!--Update_Description:update wording-->
