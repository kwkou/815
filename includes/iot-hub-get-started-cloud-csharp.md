## 创建设备标识

在本部分中，你将创建一个 Windows 控制台应用程序，用于在 IoT 中心的标识注册表中创建新的设备标识。设备无法连接到 IoT 中心，除非它在设备标识注册表中具有条目。有关详细信息，请参阅 [IoT 中心开发人员指南][lnk-devguide-identity]的“设备识别注册表”部分。当你运行此控制台应用程序时，它将生成唯一的设备 ID 和密钥，当设备向 IoT 中心发送设备到云的消息时，可以用于标识设备本身。

1. 在 Visual Studio 中，使用“控制台应用程序”项目模板将新的 Visual C# Windows 经典桌面项目添加到当前解决方案。确保 .NET Framework 版本为 4.5.1 或更高。将项目命名为 CreateDeviceIdentity。

	![][10]

2. 在“解决方案资源管理器”中，右键单击“CreateDeviceIdentity”项目，然后单击“管理 NuGet 包”。

3. 在“NuGet 包管理器”窗口中，选择“浏览”，搜索 “microsoft.azure.devices”，单击“安装”以安装 “Microsoft.Azure.Devices” 包，然后接受使用条款。

	![][11]

4. 这将下载、安装 [Azure IoT 服务 SDK][lnk-nuget-service-sdk] NuGet 包及其依赖项并添加对其的引用。

4. 在 **Program.cs** 文件顶部添加以下 `using` 语句：

		using Microsoft.Azure.Devices;
        using Microsoft.Azure.Devices.Common.Exceptions;

5. 将以下字段添加到 **Program** 类，并将占位符值替换为在上一部分中为 IoT 中心创建的连接字符串：

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

	此方法将创建 ID 为 **myFirstDevice** 的新设备标识（如果该设备 ID 已经存在于注册表中，代码就只检索现有的设备信息）。然后，应用程序将显示该标识的主密钥。你将在此模拟设备中使用此密钥连接到 IoT 中心。

7. 最后，在 **Main** 方法中添加以下行：

		registryManager = RegistryManager.CreateFromConnectionString(connectionString);
        AddDeviceAsync().Wait();
        Console.ReadLine();

8. 运行此应用程序并记下设备密钥。

    ![][12]

> [AZURE.NOTE] IoT 中心标识注册表只存储设备标识，以启用对中心的安全访问。它存储设备 ID 和密钥作为安全凭据，以及启用或禁用标志让你禁用对单个设备的访问。如果应用程序需要存储其他设备特定的元数据，则应使用应用程序特定的存储。有关详细信息，请参阅 [IoT 中心开发人员指南][lnk-devguide-identity]。

## 接收设备到云的消息

在本部分中，你将创建一个 Windows 控制台应用程序，用于读取来自 IoT 中心的设备到云消息。IoT 中心公开与[事件中心][lnk-event-hubs-overview]兼容的终结点，以让你读取设备到云的消息。为了简单起见，本教程创建的基本读取器不适用于高吞吐量部署。[处理设备到云的消息][lnk-processd2c-tutorial]教程说明了如何大规模处理设备到云的消息。[事件中心入门][lnk-eventhubs-tutorial]教程更详细说明了如何处理来自事件中心的消息，此教程也适用于 IoT 中心事件中心兼容的终结点。

> [AZURE.NOTE] 读取设备到云消息的事件中心兼容终结点始终使用 AMQPS 协议。

1. 在 Visual Studio 中，使用“控制台应用程序”项目模板将新的 Visual C# Windows 经典桌面项目添加到当前解决方案。确保 .NET Framework 版本为 4.5.1 或更高。将项目命名为 **ReadDeviceToCloudMessages**。

    ![][10]

2. 在“解决方案资源管理器”中，右键单击“ReadDeviceToCloudMessages”项目，然后单击“管理 NuGet 包”。

3. 在“NuGet 包管理器”窗口中，搜索“WindowsAzure.ServiceBus”，单击“安装”并接受使用条款。

    这样，便会下载、安装 [Azure 服务总线][lnk-servicebus-nuget]并添加对该程序包的引用，包括其所有依赖项。此包可让应用程序连接到 IoT 中心上与事件中心兼容的终结点。

4. 在 **Program.cs** 文件顶部添加以下 `using` 语句：

        using Microsoft.ServiceBus.Messaging;
        using System.Threading;

5. 将以下字段添加到 **Program** 类，并将占位符值替换为你在“创建 IoT 中心”部分中为 IoT 中心创建的连接字符串：

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

    此方法使用 **EventHubReceiver** 实例接收来自所有 IoT 中心设备到云接收分区的消息。请注意在创建 **EventHubReceiver** 对象时传递 `DateTime.Now` 参数的方式，使它只收到它启动后发送的消息。这很适合测试环境，因为这样可以看到当前的消息集，但在生产环境中，代码应该要确保它能处理所有消息。有关详细信息，请参阅[如何处理 IoT 中心设备到云的消息][lnk-processd2c-tutorial]教程。

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


<!-- Links -->

[lnk-eventhubs-tutorial]: /documentation/articles/event-hubs-csharp-ephcs-getstarted/
[lnk-devguide-identity]: /documentation/articles/iot-hub-devguide/#identityregistry
[lnk-servicebus-nuget]: https://www.nuget.org/packages/WindowsAzure.ServiceBus
[lnk-event-hubs-overview]: /documentation/articles/event-hubs-overview/

[lnk-nuget-service-sdk]: https://www.nuget.org/packages/Microsoft.Azure.Devices/
[lnk-processd2c-tutorial]: /documentation/articles/iot-hub-csharp-csharp-process-d2c/

<!-- Images -->
[10]: ./media/iot-hub-getstarted-cloud-csharp/create-identity-csharp1.png
[11]: ./media/iot-hub-getstarted-cloud-csharp/create-identity-csharp2.png
[12]: ./media/iot-hub-getstarted-cloud-csharp/create-identity-csharp3.png

<!---HONumber=Mooncake_0321_2016-->