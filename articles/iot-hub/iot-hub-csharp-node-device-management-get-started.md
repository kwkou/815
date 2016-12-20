<properties
	pageTitle = "Azure IoT 中心设备管理入门"
	description = "本教程演示如何开始使用 Azure IoT 中心的设备管理"
	services = "iot-hub"
	documentationcenter = "net"
	author = "juanjperez"
	manager = "timlt"
	editor = "" />  


<tags
	ms.service = "iot-hub"
	ms.date = "11/17/2016"
	wacn.date="12/19/2016"/>  


# 教程：设备管理入门

[AZURE.INCLUDE [iot-hub-selector-dm-getstarted](../../includes/iot-hub-selector-dm-getstarted.md)]
## 介绍
IoT 后端应用可以使用 Azure IoT 中心的基元（即设备克隆和直接方法）远程启动和监视设备上的设备管理操作。此文章提供有关 IoT 后端应用和设备如何使用 IoT 中心协同工作来启动和监视远程设备重新启动的指导和代码。

若要从基于云的后端应用远程启动和监视设备上的设备管理操作，请使用 IoT 中心基元（如[设备克隆][lnk-devtwin]和[从云到设备 (C2D) 的方法][lnk-c2dmethod]）。本教程说明后端应用和设备如何协同工作，实现从 IoT 中心启动和监视远程设备重新启动。

使用直接方法可以从云中的后端应用启动设备管理操作（如重新启动、恢复出厂设置和固件更新）。设备负责以下操作：

* 处理从 IoT 中心发送的方法请求。
* 在设备上启动相应的设备特定操作。
* 通过设备克隆向 IoT 中心报告的属性，提供状态更新。

可以使用云中的后端应用运行设备克隆查询，以报告设备管理操作的进度。

本教程演示如何：

* 使用 Azure 门户创建 IoT 中心，以及如何在 IoT 中心创建设备标识。
* 创建一个模拟设备，以便云可以调用其直接方法实现重新启动。
* 创建一个控制台应用程序，以便通过 IoT 中心调用模拟设备上的重新启动直接方法。

本教程结束时，用户会有一个 Node.js 控制台设备应用，以及一个 .NET (C#) 控制台后端应用：

**dmpatterns\_getstarted\_device.js**，它使用先前创建的设备标识连接到 IoT 中心，接收重新启动直接方法，模拟物理重新启动，并报告上次重新启动的时间。

**TriggerReboot**：在模拟设备上调用直接方法、显示响应以及显示更新的设备克隆报告属性。

若要完成本教程，您需要以下各项：

* Microsoft Visual Studio 2015。
* Node.js 版本 0.12.x 或更高版本，<br/>[准备开发环境][lnk-dev-setup]介绍了如何在 Windows 或 Linux 上安装本教程所用的 Node.js。
* 有效的 Azure 帐户。（如果没有帐户，只需花费几分钟就能创建一个[帐户][lnk-free-trial]。）

[AZURE.INCLUDE [iot-hub-get-started-create-hub](../../includes/iot-hub-get-started-create-hub.md)]

[AZURE.INCLUDE [iot-hub-get-started-create-device-identity](../../includes/iot-hub-get-started-create-device-identity.md)]

## 使用直接方法在设备上触发远程重新启动
在此部分中，会创建一个 .NET 控制台应用（使用 C#），它使用直接方法在设备上启动远程重新启动，并使用设备克隆查询找到该设备上次重新启动的时间。

1. 在 Visual Studio 中，使用“控制台应用程序”项目模板将 Visual C# Windows 经典桌面项目添加到当前解决方案。将项目命名为 **TriggerReboot**。

    ![新的 Visual C# Windows 经典桌面项目][img-createapp]  


2. 在解决方案资源管理器中，右键单击“TriggerReboot”项目，然后单击“管理 NuGet 包”。
3. 在“Nuget 包管理器”窗口中，选择“浏览”，搜索 **microsoft.azure.devices**，选择“安装”以安装 **Microsoft.Azure.Devices** 包，然后接受使用条款。此过程将下载、安装 [Microsoft Azure IoT Service SDK][lnk-nuget-service-sdk]（Microsoft Azure IoT 服务 SDK）NuGet 包及其依赖项并添加对它的引用。

    ![“NuGet 包管理器”窗口][img-servicenuget]  

4. 在 **Program.cs** 文件顶部添加以下 `using` 语句：
   
        using Microsoft.Azure.Devices;
        
5. 将以下字段添加到 **Program** 类。将占位符值替换为在上一部分和目标设备中为 IoT 中心创建的连接字符串。
   
        static RegistryManager registryManager;
        static string connString = "{iot hub connection string}";
        static ServiceClient client;
        static JobClient jobClient;
        static string targetDevice = "{deviceIdForTargetDevice}";
        
6. 将以下方法添加到 **Program** 类。此代码获取重新启动设备的克隆并输出报告的属性。
   
        public static async Task QueryTwinRebootReported()
        {
            Twin twin = await registryManager.GetTwinAsync(targetDevice);
            Console.WriteLine(twin.Properties.Reported.ToJson());
        }
        
7. 将以下方法添加到 **Program** 类。此代码使用直接方法在设备上发起重新启动操作。

        public static async Task StartReboot()
        {
            client = ServiceClient.CreateFromConnectionString(connString);
            CloudToDeviceMethod method = new CloudToDeviceMethod("reboot");
            method.ResponseTimeout = TimeSpan.FromSeconds(30);

            CloudToDeviceMethodResult result = await client.InvokeDeviceMethodAsync(targetDevice, method);

            Console.WriteLine("Invoked firmware update on device.");
        }

7. 最后，在 **Main** 方法中添加以下行：
   
        registryManager = RegistryManager.CreateFromConnectionString(connString);
        StartReboot().Wait();
        QueryTwinRebootReported().Wait();
        Console.WriteLine("Press ENTER to exit.");
        Console.ReadLine();
        
8. 生成解决方案。

## 创建模拟设备应用程序
在本部分，用户需

- 创建一个 Node.js 控制台应用，用于响应通过云调用的直接方法
- 触发模拟的设备重启
- 使用设备克隆报告的属性，允许通过设备克隆查询标识设备及其上次重启的时间


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
7. 添加以下代码，打开与 IoT 中心的连接并启动直接方法侦听器：
   
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


## 运行应用
现在，已准备就绪，可以运行应用。

1. 在 **manageddevice** 文件夹的命令提示符处，运行以下命令以开始侦听重新启动直接方法。
   
    ```
    node dmpatterns_getstarted_device.js
    ```
2. 运行 C# 控制台应用 **TriggerReboot** - 右键单击 **TriggerReboot** 项目，选择“调试”和“启动新实例”。

3. 可以在控制台中看到设备对直接方法的响应。

## 自定义和扩展设备管理操作
IoT 解决方案可以扩展已定义的设备管理模式集，或通过使用设备克隆和从云到设备方法基元启用自定义模式。设备管理操作的其他示例包括恢复出厂设置、固件更新、软件更新、电源管理、网络和连接管理以及数据加密。

## 设备维护时段
通常情况下，将设备配置为在某一时间执行操作，以最大程度减少中断和停机时间。设备维护时段是一种常用模式，用于定义设备应更新其配置的时间。后端解决方案使用设备克隆所需属性在设备上定义并激活策略，以启用维护时段。当设备收到维护时段策略时，它可以使用设备克隆报告属性报告策略状态。然后，后端应用可以使用设备克隆查询来证明设备和每个策略的符合性。

## 后续步骤
在本教程中，使用直接方法来触发设备的远程重新启动，使用设备克隆报告属性来报告设备上次重新启动时间，并查询设备克隆来从云中发现设备上次重新启动时间。

若要继续完成 IoT 中心和设备管理模式（如远程无线固件更新）的入门内容，请参阅：

[教程：如何进行固件更新][lnk-fwupdate]

若要了解如何扩展 IoT 解决方案并在多个设备上计划方法调用，请参阅 [Schedule and broadcast jobs][lnk-tutorial-jobs]（计划和广播作业）教程。

若要继续完成 IoT 中心的入门内容，请参阅 [IoT 网关 SDK 入门][lnk-gateway-SDK]。

<!-- images and links -->

[img-output]: ./media/iot-hub-get-started-with-dm/image6.png
[img-dm-ui]: ./media/iot-hub-get-started-with-dm/dmui.png
[img-servicenuget]: ./media/iot-hub-csharp-node-device-management-get-started/servicesdknuget.png
[img-createapp]: ./media/iot-hub-csharp-node-device-management-get-started/createnetapp.png

[lnk-dev-setup]: https://github.com/Azure/azure-iot-sdks/blob/master/doc/get_started/node-devbox-setup.md

[lnk-free-trial]: /pricing/1rmb-trial/
[lnk-fwupdate]: /documentation/articls/iot-hub-node-node-firmware-update/
[Azure portal]: https://portal.azure.cn/
[Using resource groups to manage your Azure resources]: /documentation/articls/resource-group-portal/
[lnk-dm-github]: https://github.com/Azure/azure-iot-device-management
[lnk-tutorial-jobs]: /documentation/articls/iot-hub-node-node-schedule-jobs/
[lnk-gateway-SDK]: /documentation/articls/iot-hub-linux-gateway-sdk-get-started/

[lnk-devtwin]: /documentation/articls/iot-hub-devguide-device-twins/
[lnk-c2dmethod]: /documentation/articls/iot-hub-devguide-direct-methods/
[lnk-transient-faults]: https://msdn.microsoft.com/zh-cn/library/hh680901(v=pandp.50).aspx
[lnk-nuget-service-sdk]: https://www.nuget.org/packages/Microsoft.Azure.Devices/

<!---HONumber=Mooncake_1212_2016-->