<properties
    pageTitle="使用 Azure IoT 中心的直接方法 (.NET/Node) | Azure"
    description="如何使用 Azure IoT 中心直接方法。使用适用于 Node.js 的 Azure IoT 设备 SDK 实现包含直接方法的模拟设备应用，并使用适用于 .NET 的 Azure IoT 服务 SDK 实现调用直接方法的服务应用。"
    services="iot-hub"
    documentationcenter=""
    author="nberdy"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="ab035b8e-bff8-4e12-9536-f31d6b6fe425"
    ms.service="iot-hub"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="10/05/2016"
    wacn.date="01/13/2017"
    ms.author="nberdy" />  


# 使用直接方法\(.NET/Node\)
[AZURE.INCLUDE [iot-hub-selector-c2d-methods](../../includes/iot-hub-selector-c2d-methods.md)]

在本教程结束时，你将拥有一个 .NET 控制台应用和一个 Node.js 控制台应用：

* **CallMethodOnDevice.sln**：一个 .NET 后端应用，可调用模拟设备应用上的方法并显示响应。
* **SimulatedDevice.js**：一个 Node.js 应用，它模拟使用早前创建的设备标识连接到 IoT 中心的设备，并响应通过云调用的方法。

> [AZURE.NOTE]
[Azure IoT SDK][lnk-hub-sdks] 文章介绍了 Azure IoT SDK，这些 SDK 可用于构建在设备和解决方案后端运行的应用程序。
> 
> 

完成本教程需具备以下条件：

* Microsoft Visual Studio 2015。
* Node.js 0.10.x 或更高版本。
* 有效的 Azure 帐户。（如果没有帐户，只需花费几分钟就能创建一个[帐户][lnk-free-trial]。）

[AZURE.INCLUDE [iot-hub-get-started-create-hub](../../includes/iot-hub-get-started-create-hub.md)]

[AZURE.INCLUDE [iot-hub-get-started-create-device-identity](../../includes/iot-hub-get-started-create-device-identity.md)]

## 创建模拟设备应用程序
在本部分，用户需创建一个 Node.js 控制台应用，用于响应解决方案后端调用的方法。

1. 新建名为 **simulateddevice** 的空文件夹。在 **simulateddevice** 文件夹的命令提示符处，使用以下命令创建 package.json 文件。接受所有默认值：
   
    
        npm init
    
2. 在 **simulateddevice** 文件夹的命令提示符处，运行下述命令以安装 **azure-iot-device** 和 **azure-iot-device-mqtt** 包：
   
    
        npm install azure-iot-device azure-iot-device-mqtt --save
    
3. 在 **simulateddevice** 文件夹中，利用文本编辑器创建新的 **SimulatedDevice.js** 文件。
4. 在 **SimulatedDevice.js** 文件的开头添加以下 `require` 语句：
   
    
        'use strict';
       
        var Mqtt = require('azure-iot-device-mqtt').Mqtt;
        var DeviceClient = require('azure-iot-device').Client;
    
5. 添加 **connectionString** 变量，并使用它创建 **DeviceClient** 实例。将 **{device connection string}** 替换为在“创建设备标识”部分创建的设备连接字符串：
   
    
        var connectionString = '{device connection string}';
        var client = DeviceClient.fromConnectionString(connectionString, Mqtt);
    
6. 添加以下函数，实现设备上的直接方法：
   
    
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
    
7. 打开与 IoT 中心的连接并初始化方法侦听器：
   
    
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

## 在设备上调用直接方法
在此部分，用户需创建一个 .NET 控制台应用，以便调用模拟设备应用上的方法，然后显示响应。

1. 在 Visual Studio 中，使用“控制台应用程序”项目模板将 Visual C# Windows 经典桌面项目添加到当前解决方案。确保 .NET Framework 版本为 4.5.1 或更高。将项目命名为 **CallMethodOnDevice**。
   
    ![新的 Visual C# Windows 经典桌面项目][10]  

2. 在“解决方案资源管理器”中，右键单击“CallMethodOnDevice”项目，然后单击“管理 NuGet 包”。
3. 在“NuGet 包管理器”窗口中，选择“浏览”，搜索 **microsoft.azure.devices**，选择“安装”以安装 **Microsoft.Azure.Devices** 包，然后接受使用条款。此过程将下载、安装 [Azure IoT 服务 SDK][lnk-nuget-service-sdk] NuGet 包及其依赖项并添加对它的引用。
   
    ![“NuGet 包管理器”窗口][11]  


4. 在 **Program.cs** 文件顶部添加以下 `using` 语句：
   
        using System.Threading.Tasks;
        using Microsoft.Azure.Devices;
5. 将以下字段添加到 **Program** 类。将占位符值替换为在上一部分中为中心创建的 IoT 中心连接字符串。
   
        static ServiceClient serviceClient;
        static string connectionString = "{iot hub connection string}";
        
6. 将以下方法添加到 **Program** 类：
   
        private static async Task InvokeMethod()
        {
            var methodInvocation = new CloudToDeviceMethod("writeLine") { ResponseTimeout = TimeSpan.FromSeconds(30) };
            methodInvocation.SetPayloadJson("'a line to be written'");

            var response = await serviceClient.InvokeDeviceMethodAsync("myDeviceId", methodInvocation);

            Console.WriteLine("Response status: {0}, payload:", response.Status);
            Console.WriteLine(response.GetPayloadAsJson());
        }
   
    此方法在 `myDeviceId` 设备上调用名为 `writeLine` 的直接方法，然后将设备提供的响应写入到控制台。请注意如何才能为要响应的设备指定一个超时值。
    
7. 最后，在 **Main** 方法中添加以下行：
   
        serviceClient = ServiceClient.CreateFromConnectionString(connectionString);
        InvokeMethod().Wait();
        Console.WriteLine("Press Enter to exit.");
        Console.ReadLine();

## 运行应用程序
现在，你已准备就绪，可以运行应用程序了。

1. 在 **simulateddevice** 文件夹的命令提示符处运行以下命令，开始侦听从 IoT 中心发出的方法调用：
   
    
        node SimulatedDevice.js
    
   
    ![][7]  

2. 设备已连接，正在等待方法调用，此时可运行 .NET **CallMethodOnDevice** 应用，在模拟设备应用中调用方法。此时会看到写入控制台的设备响应。
   
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

[7]: ./media/iot-hub-csharp-node-direct-methods/run-simulated-device.png
[8]: ./media/iot-hub-csharp-node-direct-methods/netserviceapp.png
[9]: ./media/iot-hub-csharp-node-direct-methods/methods-output.png

[10]: ./media/iot-hub-csharp-node-direct-methods/create-identity-csharp1.png
[11]: ./media/iot-hub-csharp-node-direct-methods/create-identity-csharp2.png

<!-- Links -->

[lnk-transient-faults]: https://msdn.microsoft.com/zh-cn/library/hh680901(v=pandp.50).aspx

[lnk-dev-setup]: https://github.com/Azure/azure-iot-sdks/blob/master/doc/get_started/node-devbox-setup.md

[lnk-hub-sdks]: /documentation/articles/iot-hub-devguide-sdks/
[lnk-free-trial]: /pricing/1rmb-trial/
[lnk-portal]: https://portal.azure.cn/
[lnk-nuget-service-sdk]: https://www.nuget.org/packages/Microsoft.Azure.Devices/

[lnk-devguide-jobs]: /documentation/articles/iot-hub-devguide-jobs/
[lnk-tutorial-jobs]: /documentation/articles/iot-hub-node-node-schedule-jobs/
[lnk-devguide-methods]: /documentation/articles/iot-hub-devguide-direct-methods/
[lnk-devguide-mqtt]: /documentation/articles/iot-hub-mqtt-support/

[Send Cloud-to-Device messages with IoT Hub]: /documentation/articles/iot-hub-csharp-csharp-c2d/
[Process Device-to-Cloud messages]: /documentation/articles/iot-hub-csharp-csharp-process-d2c/
[IoT 中心入门]: /documentation/articles/iot-hub-node-node-getstarted/

<!---HONumber=Mooncake_0109_2017-->
<!--Update_Description:update wording-->