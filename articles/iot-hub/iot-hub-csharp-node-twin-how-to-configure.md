<properties
    pageTitle="使用 Azure IoT 中心设备孪生属性 (.NET/Node) | Azure"
    description="如何使用 Azure IoT 中心设备孪生配置设备。 使用适用于 Node.js 的 Azure IoT 设备 SDK 实现模拟设备应用，并使用适用于 .NET 的 Azure IoT 服务 SDK 实现可使用设备孪生更改设备配置的服务应用。"
    services="iot-hub"
    documentationcenter=".net"
    author="fsautomata"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="3c627476-f982-43c9-bd17-e0698c5d236d"
    ms.service="iot-hub"
    ms.devlang="node"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="03/30/2017"
    wacn.date="06/05/2017"
    ms.author="v-yiso"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="08618ee31568db24eba7a7d9a5fc3b079cf34577"
    ms.openlocfilehash="f5dfda908b19a262ca9202db52ef784110424d49"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/26/2017" />

# <a name="use-desired-properties-to-configure-devices"></a>使用所需属性配置设备
[AZURE.INCLUDE [iot-hub-selector-twin-how-to-configure](../../includes/iot-hub-selector-twin-how-to-configure.md)]

在本教程结束时，将会创建两个控制台应用：

* **SimulateDeviceConfiguration.js**，一个模拟设备应用，它等待所需配置更新并报告模拟配置更新过程的状态。
* **SetDesiredConfigurationAndQuery**，一个 .NET 后端应用，用于在设备上设置所需配置并查询配置更新过程。

> [AZURE.NOTE]
> [Azure IoT SDK][lnk-hub-sdks] 文章介绍了可用于构建设备和后端应用的 Azure IoT SDK。
> 
> 

完成本教程需具备以下条件：

+ Visual Studio 2015 或 Visual Studio 2017。

+ Node.js 版本 0.10.x 或更高版本。

+ 有效的 Azure 帐户。 （如果你没有帐户，只需花费几分钟就能创建一个[试用帐户][lnk-free-trial]。）

如果已按照 [设备孪生入门][lnk-twin-tutorial] 教程执行了操作，则现在已有一个 IoT 中心和一个名为 **myDeviceId** 的设备标识。 在这种情况下，可以跳到 [创建模拟设备应用][lnk-how-to-configure-createapp] 部分。

[AZURE.INCLUDE [iot-hub-get-started-create-hub](../../includes/iot-hub-get-started-create-hub.md)]

[AZURE.INCLUDE [iot-hub-get-started-create-device-identity](../../includes/iot-hub-get-started-create-device-identity.md)]

## <a name="create-the-simulated-device-app"></a>创建模拟设备应用

在此部分，用户需创建一个 Node.js 控制台应用，该应用可作为 **myDeviceId** 连接到中心并等待所需配置更新，然后针对模拟配置更新过程报告更新。

1. 新建名为 **simulatedeviceconfiguration** 的空文件夹。在命令提示符下的 **simulatedeviceconfiguration** 文件夹中，使用以下命令创建新的 package.json 文件。接受所有默认值：
   
    
        npm init
    
1. 在 **simulatedeviceconfiguration** 文件夹的命令提示符处，运行下述命令以安装 **azure-iot-device** 和 **azure-iot-device-mqtt** 包：
   
    
        npm install azure-iot-device azure-iot-device-mqtt --save
    
3. 使用文本编辑器，在 **simulatedeviceconfiguration** 文件夹中创建新的 **SimulateDeviceConfiguration.js** 文件。
4. 将以下代码添加到 **SimulateDeviceConfiguration.js** 文件，并将 **{device connection string}** 占位符替换为创建 **myDeviceId** 设备标识时复制的设备连接字符串：
   
        'use strict';
        var Client = require('azure-iot-device').Client;
        var Protocol = require('azure-iot-device-mqtt').Mqtt;
   
        var connectionString = '{device connection string}';
        var client = Client.fromConnectionString(connectionString, Protocol);
   
        client.open(function(err) {
            if (err) {
                console.error('could not open IotHub client');
            } else {
                client.getTwin(function(err, twin) {
                    if (err) {
                        console.error('could not get twin');
                    } else {
                        console.log('retrieved device twin');
                        twin.properties.reported.telemetryConfig = {
                            configId: "0",
                            sendFrequency: "24h"
                        }
                        twin.on('properties.desired', function(desiredChange) {
                            console.log("received change: "+JSON.stringify(desiredChange));
                            var currentTelemetryConfig = twin.properties.reported.telemetryConfig;
                            if (desiredChange.telemetryConfig &&desiredChange.telemetryConfig.configId !== currentTelemetryConfig.configId) {
                                initConfigChange(twin);
                            }
                        });
                    }
                });
            }
        });
   
    **客户端**对象公开从设备与设备孪生进行交互所需的所有方法。 此代码将初始化 **Client** 对象，然后检索 **myDeviceId** 的设备孪生，并在所需属性上附加用于更新的处理程序。 该处理程序通过比较 configId 验证是否有实际的配置更改，然后调用用于启动配置更改的方法。
   
    请注意，为了简单起见，此代码使用初始配置的硬编码默认值。 实际的应用可能会从本地存储加载该配置。
   
       > [AZURE.IMPORTANT]
       > 连接设备时，始终发出一次所需的属性更改事件。 执行任何操作之前，请确保检查所需属性是否进行过实际更改。
       > 
       > 
       
1. 在 `client.open()` 调用前添加以下方法：
   
            var initConfigChange = function(twin) {
                var currentTelemetryConfig = twin.properties.reported.telemetryConfig;
                currentTelemetryConfig.pendingConfig = twin.properties.desired.telemetryConfig;
                currentTelemetryConfig.status = "Pending";
       
                var patch = {
                telemetryConfig: currentTelemetryConfig
                };
                twin.properties.reported.update(patch, function(err) {
                    if (err) {
                        console.log('Could not report properties');
                    } else {
                        console.log('Reported pending config change: ' + JSON.stringify(patch));
                        setTimeout(function() {completeConfigChange(twin);}, 60000);
                    }
                });
            }
       
            var completeConfigChange =  function(twin) {
                var currentTelemetryConfig = twin.properties.reported.telemetryConfig;
                currentTelemetryConfig.configId = currentTelemetryConfig.pendingConfig.configId;
                currentTelemetryConfig.sendFrequency = currentTelemetryConfig.pendingConfig.sendFrequency;
                currentTelemetryConfig.status = "Success";
                delete currentTelemetryConfig.pendingConfig;
       
                var patch = {
                    telemetryConfig: currentTelemetryConfig
                };
                patch.telemetryConfig.pendingConfig = null;
       
                twin.properties.reported.update(patch, function(err) {
                    if (err) {
                        console.error('Error reporting properties: ' + err);
                    } else {
                        console.log('Reported completed config change: ' + JSON.stringify(patch));
                    }
                });
            };
            
   
    **initConfigChange** 方法使用配置更新请求更新本地设备孪生对象的报告属性，并将状态设置为“等待中”，然后更新服务的设备孪生。 成功更新设备孪生后，它会模拟在执行 **completeConfigChange** 期间终止的长时间运行的进程。 此方法更新本地报告属性，将状态设置为“成功”并删除 **pendingConfig** 对象。 然后，它会更新服务的设备孪生。

    请注意，为了节省带宽，仅通过指定要修改的属性（在上述代码中名为 **patch**）而不是替换整个文档来更新报告属性。

    > [AZURE.NOTE]
    > 本教程不模拟并发配置更新的任何行为。 某些配置更新进程在更新运行过程中可能能够适应目标配置的更改，某些配置更新进程则可能必须将它们排队，某些配置更新进程会拒绝它们并显示错误情况。 请确保考虑你的特定配置过程所需的行为，并在开始配置更改之前添加相应的逻辑。
    > 
    > 
       

6. 运行设备应用：
    
        node SimulateDeviceConfiguration.js
   
    此时会显示消息`retrieved device twin`。使应用保持运行状态。

## <a name="create-the-service-app"></a>创建服务应用
在本节中，用户需创建一个 .NET 控制台应用，以便通过新的遥测配置对象在与 *myDeviceId* 关联的设备孪生上更新 **所需属性** 。 然后，它查询存储在 IoT 中心的设备孪生，并显示该设备的所需配置和报告配置之间的差异。

1. 在 Visual Studio 中，使用“ **控制台应用程序** ”项目模板将 Visual C# Windows 经典桌面项目添加到当前解决方案。 **SetDesiredConfigurationAndQuery**。

    ![新的 Visual C# Windows 经典桌面项目][img-createapp]
    
2. 在“解决方案资源管理器”中，右键单击“**SetDesiredConfigurationAndQuery**”项目，然后单击“**管理 NuGet 包**”。
1. 在“NuGet 包管理器”窗口中，选择“浏览”，搜索 **microsoft.azure.devices**，选择“安装”以安装 **Microsoft.Azure.Devices** 包，然后接受使用条款。 该过程将下载、安装 [Azure IoT 服务 SDK][lnk-nuget-service-sdk] Nuget 包及其依赖项并添加对它的引用。

    ![NuGet 包管理器窗口][img-servicenuget]
    
4. 在 **Program.cs** 文件顶部添加以下 `using` 语句：
   
        using Microsoft.Azure.Devices;
        using System.Threading;
        using Newtonsoft.Json;
5. 将以下字段添加到 **Program** 类。将占位符值替换为在上一部分中为中心创建的 IoT 中心连接字符串。
   
        static RegistryManager registryManager;
        static string connectionString = "{iot hub connection string}";
6. 将以下方法添加到 **Program** 类：
   
        static private async Task SetDesiredConfigurationAndQuery()
        {
            var twin = await registryManager.GetTwinAsync("myDeviceId");
            var patch = new {
                    properties = new {
                        desired = new {
                            telemetryConfig = new {
                                configId = Guid.NewGuid().ToString(),
                                sendFrequency = "5m"
                            }
                        }
                    }
                };
   
            await registryManager.UpdateTwinAsync(twin.DeviceId, JsonConvert.SerializeObject(patch), twin.ETag);
            Console.WriteLine("Updated desired configuration");
   
            while (true)
            {
                var query = registryManager.CreateQuery("SELECT * FROM devices WHERE deviceId = 'myDeviceId'");
                var results = await query.GetNextAsTwinAsync();
                foreach (var result in results)
                {
                    Console.WriteLine("Config report for: {0}", result.DeviceId);
                    Console.WriteLine("Desired telemetryConfig: {0}", JsonConvert.SerializeObject(result.Properties.Desired["telemetryConfig"], Formatting.Indented));
                    Console.WriteLine("Reported telemetryConfig: {0}", JsonConvert.SerializeObject(result.Properties.Reported["telemetryConfig"], Formatting.Indented));
                    Console.WriteLine();
                }
                Thread.Sleep(10000);
            }
        }
   

    **Registry** 对象公开从服务与设备孪生进行交互所需的所有方法。 此代码将初始化 **Registry** 对象，然后检索 **myDeviceId** 的设备孪生，并使用新的遥测配置对象更新其所需属性。
    然后，该代码会每隔 10 秒钟查询一次存储在 IoT 中心的设备孪生，并打印所需遥测配置和报告遥测配置。 若要了解如何在所有设备中生成丰富的报告，请参阅 [IoT 中心查询语言][lnk-query]。
   
    > [AZURE.IMPORTANT]
    > 为进行说明，此应用程序每 10 秒查询 IoT 中心一次。 使用查询跨多个设备生成面向用户的报表，而不检测更改。 如果解决方案需要设备事件的实时通知，请使用[孪生通知][lnk-twin-notifications]。
    > 
    > 
    
7. 最后，在 **Main** 方法中添加以下行：
   
            registryManager = RegistryManager.CreateFromConnectionString(connectionString);
            SetDesiredConfigurationAndQuery();
            Console.WriteLine("Press any key to quit.");
            Console.ReadLine();
            
8. 在 **SimulateDeviceConfiguration.js** 处于运行状态的情况下，使用 **F5** 从 Visual Studio 运行 .NET 应用程序，此时会看到报告的配置状态从 **Success** 变为 **Pending**，再变为 **Success**，采用的全新活动发送频率为 5 分钟而非 24 小时。
   
       > [AZURE.IMPORTANT]
       设备报告操作与查询结果之间最多存在一分钟的延迟。这是为了使查询基础结构可以采用非常大的规模来工作。若要检索单个设备孪生的一致视图，请使用 **Registry** 类中的 **getDeviceTwin** 方法。
       > 
       > 

## 后续步骤
在本教程中，用户已从解决方案后端将所需配置设置为*所需属性*，此外还编写了一个设备应用来检测该更改并模拟多步骤更新过程（通过报告属性报告其状态）。

充分利用以下资源：

- 通过 [Get started with IoT Hub][lnk-iothub-getstarted]（IoT 中心入门）教程学习如何从设备发送遥测；
- 通过 [Use direct methods][lnk-methods-tutorial]（使用直接方法）教程学习如何以交互方式控制设备（例如如何从用户控制的应用打开风扇）。

<!-- images -->
[img-servicenuget]: ./media/iot-hub-csharp-node-twin-how-to-configure/servicesdknuget.png
[img-createapp]: ./media/iot-hub-csharp-node-twin-how-to-configure/createnetapp.png
[img-deviceconfigured]: ./media/iot-hub-csharp-node-twin-how-to-configure/deviceconfigured.png

<!-- links -->

[lnk-hub-sdks]: /documentation/articles/iot-hub-devguide-sdks/
[lnk-free-trial]: /pricing/1rmb-trial/
[lnk-nuget-service-sdk]: https://www.nuget.org/packages/Microsoft.Azure.Devices/1.1.0/

[lnk-devguide-jobs]: /documentation/articles/iot-hub-devguide-jobs/
[lnk-query]: /documentation/articles/iot-hub-devguide-query-language/
[lnk-twin-notifications]: /documentation/articles/iot-hub-devguide-device-twins/#back-end-operations
[lnk-methods]: /documentation/articles/iot-hub-devguide-direct-methods/
[lnk-dm-overview]: /documentation/articles/iot-hub-device-management-overview/
[lnk-twin-tutorial]: /documentation/articles/iot-hub-node-node-twin-getstarted/
[lnk-schedule-jobs]: /documentation/articles/iot-hub-node-node-schedule-jobs/
[lnk-dev-setup]: https://github.com/Azure/azure-iot-sdk-node/blob/master/doc/node-devbox-setup.md
[lnk-connect-device]: /develop/iot/
[lnk-device-management]: /documentation/articles/iot-hub-node-node-device-management-get-started/
[lnk-iothub-getstarted]: /documentation/articles/iot-hub-node-node-getstarted/
[lnk-methods-tutorial]: /documentation/articles/iot-hub-node-node-direct-methods/

[lnk-guid]: https://en.wikipedia.org/wiki/Globally_unique_identifier

[lnk-how-to-configure-createapp]: /documentation/articles/iot-hub-node-node-twin-how-to-configure/#create-the-simulated-device-app
