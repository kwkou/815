<properties
	pageTitle="适用于 C# 的 Azure IoT 中心入门 | Azure"
	description="适用于 C# 的 Azure IoT 中心入门教程。配合 Azure IoT SDK 使用 Azure IoT 中心和 C# 来实施物联网解决方案。"
	services="iot-hub"
	documentationCenter=".net"
	authors="dominicbetts"
	manager="timlt"
	editor=""/>

<tags
     ms.service="iot-hub"
     ms.date="06/16/2016"
     wacn.date="07/04/2016"/>

# 适用于 .NET 的 Azure IoT 中心入门

[AZURE.INCLUDE [iot-hub-selector-get-started](../includes/iot-hub-selector-get-started.md)]

本教程最后提供了三个 Windows 控制台应用程序：

* **CreateDeviceIdentity**，用于创建设备标识和关联的安全密钥以连接模拟设备。
* **ReadDeviceToCloudMessages**，显示模拟设备发送的遥测数据。
* **SimulatedDevice**，它使用前面创建的设备标识连接到 IoT 中心，并使用 AMQPS 协议每秒发送一次遥测消息。

> [AZURE.NOTE] 有关各种 SDK 的信息（你可以使用这些 SDK 构建可在设备和解决方案后端上运行的应用程序），请参阅 [IoT 中心 SDK][lnk-hub-sdks]。

若要完成本教程，你需要以下各项：

+ Microsoft Visual Studio 2015。

+ 有效的 Azure 帐户。<br/>如果你没有帐户，只需花费几分钟就能创建一个试用帐户。有关详细信息，请参阅 [Azure 试用][lnk-free-trial]。

[AZURE.INCLUDE [iot-hub-get-started-create-hub](../includes/iot-hub-get-started-create-hub.md)]

现已创建 IoT 中心，因此已具有完成本教程的其余部分所需的主机名和连接字符串。

## 创建设备标识

在本部分中，你将创建一个 Windows 控制台应用程序，用于在 IoT 中心的标识注册表中创建新的设备标识。设备无法连接到 IoT 中心，除非它在设备标识注册表中具有条目。有关详细信息，请参阅 [IoT 中心开发人员指南][lnk-devguide-identity]的“设备标识注册表”部分。当你运行此控制台应用时，它将生成唯一的设备 ID 和密钥，当设备向 IoT 中心发送设备到云的消息时，可以用于标识设备本身。

1. 在 Visual Studio 中，使用“控制台应用程序”项目模板将新的 Visual C# Windows 经典桌面项目添加到当前解决方案。确保 .NET Framework 版本为 4.5.1 或更高。将项目命名为 **CreateDeviceIdentity**。

	![新的 Visual C# Windows 经典桌面项目][10]

2. 在“解决方案资源管理器”中，右键单击“CreateDeviceIdentity”项目，然后单击“管理 NuGet 包”。

3. 在“NuGet 包管理器”窗口中，选择“浏览”，搜索 **microsoft.azure.devices**，选择“安装”以安装 **Microsoft.Azure.Devices** 包，然后接受使用条款。这将下载、安装 [Microsoft Azure IoT 服务 SDK][lnk-nuget-service-sdk] NuGet 包及其依赖项并添加对它的引用。

	![“NuGet 包管理器”窗口][11]

4. 在 **Program.cs** 文件顶部添加以下 `using` 语句：

		using Microsoft.Azure.Devices;
        using Microsoft.Azure.Devices.Common.Exceptions;

5. 将以下字段添加到 **Program** 类。将占位符值替换为在上一部分中为 IoT 中心创建的连接字符串。

		static RegistryManager registryManager;
        static string connectionString = "{iothub connection string}";

6. 将以下方法添加到 **Program** 类：

		private static async Task AddDeviceAsync()
        {
            string deviceId = "myFirstDevice";
            Device device;
            try
            {
                device = await registryManager.AddDeviceAsync(new Device(deviceId));
            }
            catch (DeviceAlreadyExistsException)
            {
                device = await registryManager.GetDeviceAsync(deviceId);
            }
            Console.WriteLine("Generated device key: {0}", device.Authentication.SymmetricKey.PrimaryKey);
        }

	此方法将创建 ID 为 **myFirstDevice** 的新设备标识。（如果该设备 ID 已经存在于注册表中，代码就只检索现有的设备信息。） 然后，应用程序将显示该标识的主密钥。你将在此模拟设备中使用此密钥连接到 IoT 中心。

7. 最后，在 **Main** 方法中添加以下行：

		registryManager = RegistryManager.CreateFromConnectionString(connectionString);
        AddDeviceAsync().Wait();
        Console.ReadLine();

8. 运行此应用程序并记下设备密钥。

    ![应用程序生成的设备密钥][12]

> [AZURE.NOTE] IoT 中心标识注册表只存储设备标识，以启用对中心的安全访问。它存储设备 ID 和密钥作为安全凭据，以及启用或禁用标志让你禁用对单个设备的访问。如果应用程序需要存储其他特定于设备的元数据，则应使用特定于应用程序的存储。有关详细信息，请参阅 [IoT 中心开发人员指南][lnk-devguide-identity]。

## 接收设备到云的消息

在本部分中，你将创建一个 Windows 控制台应用程序，用于读取来自 IoT 中心的设备到云消息。IoT 中心公开与 [Azure 事件中心][lnk-event-hubs-overview]兼容的终结点，以让你读取设备到云的消息。为了简单起见，本教程创建的基本读取器不适用于高吞吐量部署。若要了解如何大规模处理设备到云的消息，请参阅[处理设备到云的消息][lnk-process-d2c-tutorial]教程。有关如何处理来自事件中心的消息的更多信息，请参阅[事件中心入门][lnk-eventhubs-tutorial]教程。（本教程适用于 IoT 中心上与事件中心兼容的终结点）。

> [AZURE.NOTE] 读取设备到云消息的事件中心兼容终结点始终使用 AMQPS 协议。

1. 在 Visual Studio 中，使用“控制台应用程序”项目模板将新的 Visual C# Windows 经典桌面项目添加到当前解决方案。确保 .NET Framework 版本为 4.5.1 或更高。将项目命名为 **ReadDeviceToCloudMessages**。

    ![新的 Visual C# Windows 经典桌面项目][10]

2. 在“解决方案资源管理器”中，右键单击“ReadDeviceToCloudMessages”项目，然后单击“管理 NuGet 包”。

3. 在“NuGet 包管理器”窗口中，搜索 **WindowsAzure.ServiceBus**，选择“安装”并接受使用条款。这将下载、安装 [Azure 服务总线][lnk-servicebus-nuget]及其所有依赖项并添加对它的引用。此包可让应用程序连接到 IoT 中心上与事件中心兼容的终结点。

4. 在 **Program.cs** 文件顶部添加以下 `using` 语句：

        using Microsoft.ServiceBus.Messaging;
        using System.Threading;

5. 将以下字段添加到 **Program** 类。将占位符值替换为在“创建 IoT 中心”部分中为 IoT 中心创建的连接字符串。

        static string connectionString = "{iothub connection string}";
        static string iotHubD2cEndpoint = "messages/events";
        static EventHubClient eventHubClient;

6. 将以下方法添加到 **Program** 类：

        private static async Task ReceiveMessagesFromDeviceAsync(string partition, CancellationToken ct)
        {
            var eventHubReceiver = eventHubClient.GetDefaultConsumerGroup().CreateReceiver(partition, DateTime.UtcNow);
            while (true)
            {
                if (ct.IsCancellationRequested) break;
                EventData eventData = await eventHubReceiver.ReceiveAsync();
                if (eventData == null) continue;

                string data = Encoding.UTF8.GetString(eventData.GetBytes());
                Console.WriteLine("Message received. Partition: {0} Data: '{1}'", partition, data);
            }
        }

    此方法使用 **EventHubReceiver** 实例接收来自所有 IoT 中心设备到云接收分区的消息。请注意在创建 **EventHubReceiver** 对象时传递 `DateTime.Now` 参数的方式，使它只收到它启动后发送的消息。这很适合测试环境，因为这样可以看到当前的消息集，但在生产环境中，代码应该要确保它能处理所有消息。有关详细信息，请参阅[如何处理 IoT 中心设备到云的消息][lnk-process-d2c-tutorial]教程。

7. 最后，在 **Main** 方法中添加以下行：

        Console.WriteLine("Receive messages. Ctrl-C to exit.\n");
        eventHubClient = EventHubClient.CreateFromConnectionString(connectionString, iotHubD2cEndpoint);

        var d2cPartitions = eventHubClient.GetRuntimeInformation().PartitionIds;

        CancellationTokenSource cts = new CancellationTokenSource();

        System.Console.CancelKeyPress += (s, e) =>
        {
          e.Cancel = true;
          cts.Cancel();
          Console.WriteLine("Exiting...");
        };

        var tasks = new List<Task>();
        foreach (string partition in d2cPartitions)
        {
           tasks.Add(ReceiveMessagesFromDeviceAsync(partition, cts.Token));
        }  
        Task.WaitAll(tasks.ToArray());

## 创建模拟设备应用程序

在本部分中，你将创建一个 Windows 控制台应用程序，用于模拟向 IoT 中心发送设备到云消息的设备。

1. 在 Visual Studio 中，使用“控制台应用程序”项目模板将新的 Visual C# Windows 经典桌面项目添加到当前解决方案。确保 .NET Framework 版本为 4.5.1 或更高。将项目命名为 **SimulatedDevice**。

    ![新的 Visual C# Windows 经典桌面项目][10]

2. 在“解决方案资源管理器”中，右键单击“SimulatedDevice”项目，然后单击“管理 NuGet 包”。

3. 在“NuGet 包管理器”窗口中，选择“浏览”，搜索 **Microsoft.Azure.Devices.Client**，选择“安装”以安装 **Microsoft.Azure.Devices.Client** 包，然后接受使用条款。这将下载、安装 [Azure IoT - 设备 SDK NuGet 包][lnk-device-nuget]及其依赖项并添加对它的引用。

4. 在 **Program.cs** 文件顶部添加以下 `using` 语句：

		using Microsoft.Azure.Devices.Client;
        using Newtonsoft.Json;

5. 将以下字段添加到 **Program** 类。将占位符值替换为在“创建 IoT 中心”部分中检索到的 IoT 中心主机名，以及在“创建设备标识”部分中检索到的设备密钥。

		static DeviceClient deviceClient;
        static string iotHubUri = "{iot hub hostname}";
        static string deviceKey = "{device key}";

6. 将以下方法添加到 **Program** 类：

		private static async void SendDeviceToCloudMessagesAsync()
        {
            double avgWindSpeed = 10; // m/s
            Random rand = new Random();

            while (true)
            {
                double currentWindSpeed = avgWindSpeed + rand.NextDouble() * 4 - 2;

                var telemetryDataPoint = new
                {
                    deviceId = "myFirstDevice",
                    windSpeed = currentWindSpeed
                };
                var messageString = JsonConvert.SerializeObject(telemetryDataPoint);
                var message = new Message(Encoding.ASCII.GetBytes(messageString));

                await deviceClient.SendEventAsync(message);
                Console.WriteLine("{0} > Sending message: {1}", DateTime.Now, messageString);

                Task.Delay(1000).Wait();
            }
        }

	此方法每隔一秒发送一条新的设备到云消息。该消息包含一个具有设备 ID 的 JSON 序列化对象和一个随机生成的编号，用于模拟风速传感器。

7. 最后，在 **Main** 方法中添加以下行：

        Console.WriteLine("Simulated device\n");
        deviceClient = DeviceClient.Create(iotHubUri, new DeviceAuthenticationWithRegistrySymmetricKey("myFirstDevice", deviceKey));

        SendDeviceToCloudMessagesAsync();
        Console.ReadLine();

  默认情况下，**Create** 方法将创建使用 AMQP 协议来与 IoT 中心通信的 **DeviceClient** 实例。若要使用 HTTPS 协议，请使用 **Create** 方法的重写，它可以让你指定协议。如果使用 HTTPS 协议，则还应在项目中添加 **Microsoft.AspNet.WebApi.Client** NuGet 包，以包含 **System.Net.Http.Formatting** 命名空间。

本教程指导你完成创建 IoT 中心设备客户端的步骤。你也可以使用 [Azure IoT 中心的连接服务][lnk-connected-service] Visual Studio 扩展将所需的代码添加到设备客户端应用程序。


> [AZURE.NOTE] 为简单起见，本教程不实现任何重试策略。在生产代码中，你应该按 MSDN 文章 [Transient Fault Handling][lnk-transient-faults]（暂时性故障处理）中所述实施重试策略（例如指数性的回退）。
## 运行应用程序

现在，你已准备就绪，可以运行应用程序了。

1.	在 Visual Studio 的“解决方案资源管理器”中右键单击你的解决方案，然后单击“设置启动项目”。选择“多个启动项目”，然后针对 **ProcessDeviceToCloudMessages** 和 **SimulatedDevice** 项目选择“启动”作为操作。

   	![启动项目属性][41]

2.	按 **F5** 启动这两个应用，使其运行。来自 **SimulatedDevice** 应用的控制台输出会显示模拟设备发送给 IoT 中心的消息。来自 **ProcessDeviceToCloudMessages** 应用的控制台输出则会显示 IoT 中心接收的消息。

   	![来自应用的控制台输出][42]

3. [Azure 门户][lnk-portal]中的“使用情况”磁贴显示发送到中心的消息数：

    ![Azure 门户的“使用情况”磁贴][43]


## 后续步骤

在本教程中，你已在门户中配置了新的 IoT 中心，然后在中心的标识注册表中创建了设备标识。你已使用此设备标识来让模拟设备应用向中心发送设备到云的消息，并创建了用于显示中心所接收消息的应用。可以使用以下教程继续探索 IoT 中心功能和其他 IoT 方案：

- [使用 IoT 中心发送云到设备的消息][lnk-c2d-tutorial]说明如何将消息发送到设备，并处理 IoT 中心生成的传送反馈。
- [处理设备到云的消息][lnk-process-d2c-tutorial]说明如何可靠地处理来自设备的遥测数据和交互消息。
- [从设备上载文件][lnk-upload-tutorial]介绍了一种模式，该模式利用云到设备的消息来帮助从设备上载文件。

<!-- Images. -->
[1]: ./media/iot-hub-csharp-csharp-getstarted/create-iot-hub1.png
[2]: ./media/iot-hub-csharp-csharp-getstarted/create-iot-hub2.png
[3]: ./media/iot-hub-csharp-csharp-getstarted/create-iot-hub3.png
[4]: ./media/iot-hub-csharp-csharp-getstarted/create-iot-hub4.png
[5]: ./media/iot-hub-csharp-csharp-getstarted/create-iot-hub5.png
[41]: ./media/iot-hub-csharp-csharp-getstarted/run-apps1.png
[42]: ./media/iot-hub-csharp-csharp-getstarted/run-apps2.png
[43]: ./media/iot-hub-csharp-csharp-getstarted/usage.png
[10]: ./media/iot-hub-csharp-csharp-getstarted/create-identity-csharp1.png
[11]: ./media/iot-hub-csharp-csharp-getstarted/create-identity-csharp2.png
[12]: ./media/iot-hub-csharp-csharp-getstarted/create-identity-csharp3.png

<!-- Links -->
[lnk-c2d-tutorial]: /documentation/articles/iot-hub-csharp-csharp-c2d/
[lnk-process-d2c-tutorial]: /documentation/articles/iot-hub-csharp-csharp-process-d2c/
[lnk-upload-tutorial]: /documentation/articles/iot-hub-csharp-csharp-file-upload/

[lnk-hub-sdks]: /documentation/articles/iot-hub-sdks-summary/
[lnk-free-trial]: /pricing/1rmb-trial/
[lnk-portal]: https://manage.windowsazure.cn
[lnk-eventhubs-tutorial]: /documentation/articles/event-hubs-csharp-ephcs-getstarted/
[lnk-devguide-identity]: /documentation/articles/iot-hub-devguide/#identityregistry
[lnk-servicebus-nuget]: https://www.nuget.org/packages/WindowsAzure.ServiceBus
[lnk-event-hubs-overview]: /documentation/articles/event-hubs-overview/

[lnk-nuget-service-sdk]: https://www.nuget.org/packages/Microsoft.Azure.Devices/
[lnk-device-nuget]: https://www.nuget.org/packages/Microsoft.Azure.Devices.Client/
[lnk-transient-faults]: https://msdn.microsoft.com/zh-cn/library/hh680901(v=pandp.50).aspx
[lnk-connected-service]: https://visualstudiogallery.msdn.microsoft.com/e254a3a5-d72e-488e-9bd3-8fee8e0cd1d6

<!---HONumber=Mooncake_0627_2016-->