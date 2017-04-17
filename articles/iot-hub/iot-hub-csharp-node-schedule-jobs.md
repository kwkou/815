<properties
    pageTitle="通过 Azure IoT 中心 (.NET/Node) 安排作业 | Azure"
    description="如何安排 Azure IoT 中心作业实现多台设备上的直接方法调用。 使用适用于 Node.js 的 Azure IoT 设备 SDK 实现模拟设备应用，并使用适用于 .NET 的 Azure IoT 服务 SDK 实现用于运行作业的服务应用。"
    services="iot-hub"
    documentationcenter=".net"
    author="juanjperez"
    manager="timlt"
    editor=""
    translationtype="Human Translation" />
<tags
    ms.assetid="2233356e-b005-4765-ae41-3a4872bda943"
    ms.service="iot-hub"
    ms.devlang="multiple"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="02/24/2017"
    wacn.date="04/17/2017"
    ms.author="juanpere"
    ms.sourcegitcommit="7cc8d7b9c616d399509cd9dbdd155b0e9a7987a8"
    ms.openlocfilehash="2f1199d1cded08ddd99b5d0a8f7f8ccafc697576"
    ms.lasthandoff="04/07/2017" />

# <a name="schedule-and-broadcast-jobs"></a>计划和广播作业
[AZURE.INCLUDE [iot-hub-selector-schedule-jobs](../../includes/iot-hub-selector-schedule-jobs.md)]

## <a name="introduction"></a>介绍
Azure IoT 中心是一项完全托管的服务，允许后端应用创建和跟踪用于计划和更新数百万个设备的作业。  作业可以用于以下操作：

* 更新所需属性
* 更新标记
* 调用直接方法

从概念上讲，作业包装这些操作之一，并跟踪针对一组设备执行（由设备孪生查询定义）的进度。例如，通过使用作业，后端应用可以对 10,000 个设备调用重新启动方法（由设备孪生查询指定并安排在将来执行）。该应用程序随后可以在其中每个设备接收和执行重新启动方法时跟踪进度。

可在以下文章中了解有关所有这些功能的详细信息：

* 设备孪生和属性：[设备孪生入门][lnk-get-started-twin]和[教程：如何使用设备孪生属性][lnk-twin-props]
* 直接方法：[IoT 中心开发人员指南 - 直接方法][lnk-dev-methods]和[教程：使用直接方法][lnk-c2d-methods]

本教程演示如何：

* 创建一个模拟设备应用，它具有实现可以由后端应用调用的 **lockDoor** 的直接方法。
* 创建 .NET 控制台应用，它使用作业在模拟设备应用中调用 **lockDoor** 直接方法，并使用设备作业更新所需属性。

本教程结束时，用户会有一个 Node.js 控制台设备应用，以及一个 .NET (C#) 控制台后端应用：

**simDevice.js**，它使用设备标识连接到 IoT 中心，并接收 **lockDoor** 直接方法。

**ScheduleJob**，它在模拟设备应用中调用直接方法，并通过作业更新设备孪生所需的属性。

要完成本教程，需要具备以下先决条件：

* Microsoft Visual Studio 2015。
* Node.js 版本 0.12.x 或更高版本。
* [准备开发环境][lnk-dev-setup]介绍了如何在 Windows 或 Linux 上安装本教程所用的 Node.js。
* 有效的 Azure 帐户。（如果没有帐户，只需花费几分钟就能创建一个[免费帐户][lnk-free-trial]。）

[AZURE.INCLUDE [iot-hub-get-started-create-hub](../../includes/iot-hub-get-started-create-hub.md)]

[AZURE.INCLUDE [iot-hub-get-started-create-device-identity](../../includes/iot-hub-get-started-create-device-identity.md)]

## <a name="schedule-jobs-for-calling-a-direct-method-and-updating-a-device-twins-properties"></a>安排作业，用于调用直接方法和更新设备孪生的属性
该部分会创建一个 .NET 控制台应用（使用 C#），它直接在设备上启动远程 **lockDoor** 并更新设备孪生的属性。

1. 在 Visual Studio 中，使用“ **控制台应用程序** ”项目模板将 Visual C# Windows 经典桌面项目添加到当前解决方案。 将该项目命名为 **ScheduleJob**。

	![新的 Visual C# Windows 经典桌面项目][img-createapp]  


2. 在“解决方案资源管理器”中，右键单击“ScheduleJob”项目，然后单击“管理 NuGet 包”。
3. 在“NuGet 包管理器”窗口中，选择“浏览”，搜索 **microsoft.azure.devices**，选择“安装”以安装 **Microsoft.Azure.Devices** 包，然后接受使用条款。 该过程将下载、安装 [Azure IoT 服务 SDK][lnk-nuget-service-sdk] NuGet 包及其依赖项并添加对它的引用。

	![“NuGet 包管理器”窗口][img-servicenuget]  

4. 在 **Program.cs** 文件顶部添加以下 `using` 语句：
   
        using Microsoft.Azure.Devices;
        
5. 将以下字段添加到 **Program** 类。将占位符替换为在上一部分中为中心创建的 IoT 中心连接字符串。
   
        static string connString = "{iot hub connection string}";
        static ServiceClient client;
        static JobClient jobClient;
        
6. 将以下方法添加到 **Program** 类：
   
        public static async Task MonitorJob(string jobId)
        {
            JobResponse result;
            do
            {
                result = await jobClient.GetJobAsync(jobId);
                Console.WriteLine("Job Status : " + result.Status.ToString());
                Thread.Sleep(2000);
            } while ((result.Status != JobStatus.Completed) && (result.Status != JobStatus.Failed));
        }
                
7. 将以下方法添加到 **Program** 类：

        public static async Task StartMethodJob(string jobId)
        {
            CloudToDeviceMethod directMethod = new CloudToDeviceMethod("lockDoor", TimeSpan.FromSeconds(5), TimeSpan.FromSeconds(5));

            JobResponse result = await jobClient.ScheduleDeviceMethodAsync(jobId,
                "deviceId='myDeviceId'",
                directMethod,
                DateTime.Now,
                10);

            Console.WriteLine("Started Method Job");
        }

8. 将以下方法添加到 **Program** 类：

        public static async Task StartTwinUpdateJob(string jobId)
        {
            var twin = new Twin();
            twin.Properties.Desired["Building"] = "43";
            twin.Properties.Desired["Floor"] = "3";
            twin.ETag = "*";

            JobResponse result = await jobClient.ScheduleTwinUpdateAsync(jobId,
                "deviceId='myDeviceId'",
                twin,
                DateTime.Now,
                10);

            Console.WriteLine("Started Twin Update Job");
        }
 

9. 最后，在 **Main** 方法中添加以下行：
   
        jobClient = JobClient.CreateFromConnectionString(connString);

        string methodJobId = Guid.NewGuid().ToString();

        StartMethodJob(methodJobId);
        MonitorJob(methodJobId).Wait();
        Console.WriteLine("Press ENTER to run the next job.");
        Console.ReadLine();

        string twinUpdateJobId = Guid.NewGuid().ToString();

        StartTwinUpdateJob(twinUpdateJobId);
        MonitorJob(twinUpdateJobId).Wait();
        Console.WriteLine("Press ENTER to exit.");
        Console.ReadLine();
                   
10. 生成解决方案。

## <a name="create-a-simulated-device-app"></a>创建模拟设备应用程序
在本部分中，创建响应云调用的直接方法的 Node.js 控制台应用，这将触发模拟的设备重新启动并使用报告属性启用设备孪生查询，以标识设备和及其上次重新启动的时间。

1. 新建名为 **simDevice** 的空文件夹。在 **simDevice** 文件夹的命令提示符处，使用以下命令创建 package.json 文件。接受所有默认值：
   
    
        npm init
    
2. 在 **simDevice** 文件夹的命令提示符处，运行下述命令以安装 **azure-iot-device** 设备 SDK 包和 **azure-iot-device-mqtt** 包：
   
    
        npm install azure-iot-device azure-iot-device-mqtt --save

3. 在 **simDevice.js** 文件夹中，利用文本编辑器创建新的 **simDevice** 文件。
4. 在 **simDevice.js** 文件的开头添加以下“require”语句：
   
    
        'use strict';
       
        var Client = require('azure-iot-device').Client;
        var Protocol = require('azure-iot-device-mqtt').Mqtt;

5. 添加 **connectionString** 变量，并使用它创建一个**客户端**实例。  

        var connectionString = 'HostName={youriothostname};DeviceId={yourdeviceid};SharedAccessKey={yourdevicekey}';
        var client = Client.fromConnectionString(connectionString, Protocol);
    
6. 添加以下函数以处理 **lockDoor** 方法。
   
    
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
    
7. 添加以下代码以注册 **lockDoor** 方法的处理程序。
   
    
        client.open(function(err) {
            if (err) {
                console.error('Could not connect to IotHub client.');
            }  else {
            	console.log('Client connected to IoT Hub.  Waiting for lockDoor direct method.');
                client.onDeviceMethod('lockDoor', onLockDoor);
            }
        });
    
8. 保存并关闭 **simDevice.js** 文件。

> [AZURE.NOTE]
> 为简单起见，本教程不实现任何重试策略。 在生产代码中，应该按 MSDN 文章 [Transient Fault Handling][lnk-transient-faults]（暂时性故障处理）中所述实施重试策略（例如指数性的回退）。
> 
> 

## <a name="run-the-apps"></a>运行应用
现在，已准备就绪，可以运行应用。

1. 在 **simDevice** 文件夹的命令提示符处，运行以下命令以开始侦听重新启动直接方法。
   
    
        node simDevice.js

2. 运行 C# 控制台应用 **ScheduleJob** - 右键单击“ScheduleJob”项目，然后依次选择“调试”和“启动新实例”。

3. 会出现设备和后端应用的输出。

## <a name="next-steps"></a>后续步骤
在本教程中，使用了作业来安排用于设备的直接方法以及设备孪生属性的更新。

若要继续完成 IoT 中心和设备管理模式（如远程无线固件更新）的入门内容，请参阅：

[教程：如何进行固件更新][lnk-fwupdate]

若要继续完成 IoT 中心的入门内容，请参阅 [IoT 网关 SDK 入门][lnk-gateway-SDK]。

<!-- images -->
[img-servicenuget]: ./media/iot-hub-csharp-node-schedule-jobs/servicesdknuget.png
[img-createapp]: ./media/iot-hub-csharp-node-schedule-jobs/createnetapp.png

[lnk-get-started-twin]: /documentation/articles/iot-hub-node-node-twin-getstarted/
[lnk-twin-props]: /documentation/articles/iot-hub-node-node-twin-how-to-configure/
[lnk-c2d-methods]: /documentation/articles/iot-hub-node-node-direct-methods/
[lnk-dev-methods]: /documentation/articles/iot-hub-devguide-direct-methods/
[lnk-fwupdate]: /documentation/articles/iot-hub-node-node-firmware-update/
[lnk-gateway-SDK]: /documentation/articles/iot-hub-linux-gateway-sdk-get-started/
[lnk-dev-setup]: https://github.com/Azure/azure-iot-sdk-node/blob/master/doc/node-devbox-setup.md
[lnk-free-trial]: /pricing/1rmb-trial/
[lnk-transient-faults]: https://msdn.microsoft.com/zh-cn/library/hh680901(v=pandp.50).aspx
[lnk-nuget-service-sdk]: https://www.nuget.org/packages/Microsoft.Azure.Devices/


<!--Update_Description:update wording and code-->