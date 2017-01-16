<properties
    pageTitle="Azure IoT 中心设备孪生入门 (.NET/Node) | Azure"
    description="如何使用 Azure IoT 中心设备孪生添加标记，然后使用 IoT 中心查询。使用适用于 Node.js 的 Azure IoT 设备 SDK 实现模拟设备应用，并使用适用于 .NET 的 Azure IoT 服务 SDK 实现可添加标记并运行 IoT 中心查询的服务应用。"
    services="iot-hub"
    documentationcenter="node"
    author="fsautomata"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="f7e23b6e-bfde-4fba-a6ec-dbb0f0e005f4"
    ms.service="iot-hub"
    ms.devlang="node"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="na"
    ms.date="09/13/2016"
    wacn.date="01/13/2017"
    ms.author="elioda" />  


# 设备孪生入门 \(.NET/Node\)
[AZURE.INCLUDE [iot-hub-selector-twin-get-started](../../includes/iot-hub-selector-twin-get-started.md)]

在本教程结束时，你将拥有一个 .NET 控制台应用和一个 Node.js 控制台应用：

* **AddTagsAndQuery.sln**，一个 .NET 后端应用，用于添加标记并查询设备孪生。
* **TwinSimulatedDevice.js**，一个 Node.js 应用，它模拟使用早前创建的设备标识连接到 IoT 中心的设备，并报告其连接条件。

> [AZURE.NOTE]
[Azure IoT SDK][lnk-hub-sdks] 文章介绍了可用于构建设备和后端应用的 Azure IoT SDK。
> 
> 

完成本教程需具备以下条件：

+ Microsoft Visual Studio 2015。

+ Node.js 0.10.x 或更高版本。

+ 有效的 Azure 帐户。（如果没有帐户，只需花费几分钟就能创建一个[帐户][lnk-free-trial]。）

[AZURE.INCLUDE [iot-hub-get-started-create-hub](../../includes/iot-hub-get-started-create-hub.md)]

[AZURE.INCLUDE [iot-hub-get-started-create-device-identity](../../includes/iot-hub-get-started-create-device-identity.md)]

## 创建服务应用
在此部分，会创建一个 Node.js 控制台应用，将位置元数据添加到与 **myDeviceId** 关联的设备孪生。该应用随后会选择位于中国的设备来查询存储在 IoT 中心的设备孪生，然后查询报告手机网络连接的设备孪生。

1. 在 Visual Studio 中，使用“控制台应用程序”项目模板将 Visual C# Windows 经典桌面项目添加到当前解决方案。将项目命名为 **AddTagsAndQuery**。
   
    ![新的 Visual C# Windows 经典桌面项目][img-createapp]  

2. 在“解决方案资源管理器”中，右键单击“AddTagsAndQuery”项目，然后单击“管理 NuGet 包”。
3. 在“Nuget 包管理器”窗口中，选择“浏览”，搜索 **microsoft.azure.devices**，选择“安装”以安装 **Microsoft.Azure.Devices** 包，然后接受使用条款。此过程将下载、安装 [Azure IoT 服务 SDK][lnk-nuget-service-sdk] NuGet 包及其依赖项并添加对它的引用。
   
    ![“NuGet 包管理器”窗口][img-servicenuget]  

4. 在 **Program.cs** 文件顶部添加以下 `using` 语句：
   
        using Microsoft.Azure.Devices;
5. 将以下字段添加到 **Program** 类。将占位符值替换为在上一部分中为中心创建的 IoT 中心连接字符串。
   
        static RegistryManager registryManager;
        static string connectionString = "{iot hub connection string}";
6. 将以下方法添加到 **Program** 类：
   
        public static async Task AddTagsAndQuery()
        {
            var twin = await registryManager.GetTwinAsync("myDeviceId");
            var patch =
                @"{
                    tags: {
                        location: {
                            region: 'US',
                            plant: 'Redmond43'
                        }
                    }
                }";
            await registryManager.UpdateTwinAsync(twin.DeviceId, patch, twin.ETag);
   
            var query = registryManager.CreateQuery("SELECT * FROM devices WHERE tags.location.plant = 'Redmond43'", 100);
            var twinsInRedmond43 = await query.GetNextAsTwinAsync();
            Console.WriteLine("Devices in Redmond43: {0}", string.Join(", ", twinsInRedmond43.Select(t => t.DeviceId)));
   
            query = registryManager.CreateQuery("SELECT * FROM devices WHERE tags.location.plant = 'Redmond43' AND properties.reported.connectivity.type = 'cellular'", 100);
            var twinsInRedmond43UsingCellular = await query.GetNextAsTwinAsync();
            Console.WriteLine("Devices in Redmond43 using cellular network: {0}", string.Join(", ", twinsInRedmond43UsingCellular.Select(t => t.DeviceId)));
        }
   
    **RegistryManager** 类会公开从服务与设备孪生进行交互所需的所有方法。上面的代码首先初始化 **registryManager** 对象，然后检索 **myDeviceId** 的设备孪生，最后使用所需位置信息更新其标记。
   
    更新后，该代码将执行两个查询：第一个只选择位于 **Redmond43** 工厂中的设备的设备孪生，第二个会优化查询，只选择也通过手机网络连接的设备。
   
    请注意，前面的代码在创建 **query** 对象时会指定最大返回文档数。**query** 对象包含 **HasMoreResults** 布尔属性，可以用于多次调用 **GetNextAsTwinAsync** 方法以检索所有结果。名为 **GetNextAsJson** 的方法可用于不是设备孪生的结果（例如聚合查询的结果）。
7. 最后，在 **Main** 方法中添加以下行：
   
        registryManager = RegistryManager.CreateFromConnectionString(connectionString);
        AddTagsAndQuery().Wait();
        Console.WriteLine("Press Enter to exit.");
        Console.ReadLine();
8. 运行此应用程序，对于寻找位于 **Redmond43** 的所有设备的查询，应在结果中看到一个设备，而对于将结果限制为使用手机网络连接的设备的查询，不会看到任何设备。
   
    ![窗口中的查询结果][img-addtagapp]  


在下一部分，用户创建的设备应用会报告连接信息并更改上一部分中查询的结果。

## 创建设备应用
在此部分，用户需创建一个 Node.js 控制台应用作为 **myDeviceId** 连接到中心，然后更新其报告属性，以包含它使用手机网络进行连接的信息。

1. 新建名为 **reportconnectivity** 的空文件夹。在 **reportconnectivity** 文件夹的命令提示符处，使用以下命令创建新的 package.json 文件。接受所有默认值：
   
    
        npm init
    
2. 在 **reportconnectivity** 文件夹的命令提示符处，运行以下命令来安装 **azure-iot-device** 和 **azure-iot-device-mqtt** 包：
   
    
        npm install azure-iot-device azure-iot-device-mqtt --save
    
3. 在 **reportconnectivity** 文件夹中使用文本编辑器新建 **ReportConnectivity.js** 文件。
4. 将以下代码添加到 **ReportConnectivity.js** 文件，并将 **{device connection string}** 占位符替换为创建 **myDeviceId** 设备标识时复制的设备连接字符串：
   
        'use strict';
        var Client = require('azure-iot-device').Client;
        var Protocol = require('azure-iot-device-mqtt').Mqtt;
   
        var connectionString = '{device connection string}';
        var client = Client.fromConnectionString(connectionString, Protocol);
   
        client.open(function(err) {
        if (err) {
            console.error('could not open IotHub client');
        }  else {
            console.log('client opened');
   
            client.getTwin(function(err, twin) {
            if (err) {
                console.error('could not get twin');
            } else {
                var patch = {
                    connectivity: {
                        type: 'cellular'
                    }
                };
   
                twin.properties.reported.update(patch, function(err) {
                    if (err) {
                        console.error('could not update twin');
                    } else {
                        console.log('twin state reported');
                        process.exit();
                    }
                });
            }
            });
        }
        });
   
    **Client** 对象会公开从设备与设备孪生进行交互所需的所有方法。前面的代码在初始化 **Client** 对象后会检索 **myDeviceId** 的设备孪生，并使用连接信息更新其报告属性。
5. 运行设备应用
   
        node ReportConnectivity.js
   
    此时会显示消息`twin state reported`。
6. 现在设备报告了其连接信息，该信息应出现在两个查询中。运行 .NET **AddTagsAndQuery** 应用即可再次运行查询。这次 **myDeviceId** 应出现在两个查询结果中。
   
    ![][img-addtagapp2]  


## 后续步骤
本教程中，在 Azure 门户预览中配置了新的 IoT 中心，然后在 IoT 中心的标识注册表中创建了设备标识。已从后端应用以标记形式添加了设备元数据，并编写了模拟的设备应用，用于报告设备孪生中的设备连接信息。还学习了如何使用类似 SQL 的 IoT 中心查询语言来查询此信息。

充分利用以下资源：

- 通过 [Get started with IoT Hub][lnk-iothub-getstarted]（IoT 中心入门）教程学习如何从设备发送遥测；
- 通过[使用所需属性配置设备][lnk-twin-how-to-configure]教程学习如何使用设备孪生的所需属性配置设备，
- 通过 [Use direct methods][lnk-methods-tutorial]（使用直接方法）教程学习如何以交互方式控制设备（例如如何从用户控制的应用打开风扇）。

<!-- images -->

[img-servicenuget]: ./media/iot-hub-csharp-node-twin-getstarted/servicesdknuget.png
[img-createapp]: ./media/iot-hub-csharp-node-twin-getstarted/createnetapp.png
[img-addtagapp]: ./media/iot-hub-csharp-node-twin-getstarted/addtagapp.png
[img-addtagapp2]: ./media/iot-hub-csharp-node-twin-getstarted/addtagapp2.png

<!-- links -->

[lnk-hub-sdks]: /documentation/articles/iot-hub-devguide-sdks/
[lnk-free-trial]: /pricing/1rmb-trial/
[lnk-nuget-service-sdk]: https://www.nuget.org/packages/Microsoft.Azure.Devices/

[lnk-d2c]: /documentation/articles/iot-hub-devguide-messaging/#device-to-cloud-messages
[lnk-methods]: /documentation/articles/iot-hub-devguide-direct-methods/
[lnk-twins]: /documentation/articles/iot-hub-devguide-device-twins/
[lnk-query]: /documentation/articles/iot-hub-devguide-query-language/
[lnk-identity]: /documentation/articles/iot-hub-devguide-identity-registry/

[lnk-iothub-getstarted]: /documentation/articles/iot-hub-node-node-getstarted/
[lnk-methods-tutorial]: /documentation/articles/iot-hub-node-node-direct-methods/
[lnk-twin-how-to-configure]: /documentation/articles/iot-hub-csharp-node-twin-how-to-configure/

[lnk-dev-setup]: https://github.com/Azure/azure-iot-sdks/blob/master/doc/get_started/node-devbox-setup.md

<!---HONumber=Mooncake_0109_2017-->
<!--Update_Description:update wording-->