## 创建模拟设备应用程序

在本部分中，你将创建一个 Windows 控制台应用程序，用于模拟向 IoT 中心发送设备到云消息的设备。

1. 在 Visual Studio 中，使用“控制台应用程序”项目模板将新的 Visual C# Windows 经典桌面项目添加到当前解决方案。将项目命名为 **SimulatedDevice**。

   	![][30]

2. 在“解决方案资源管理器”中，右键单击“SimulatedDevice”项目，然后单击“管理 NuGet 程序包”。

3. 在“NuGet 包管理器”窗口中，搜索“Microsoft Azure 设备客户端”，然后单击“安装”并接受使用条款。

	这将下载、安装 [Azure IoT - 设备 SDK NuGet 包][lnk-device-nuget]并添加对它的引用。

4. 在 **Program.cs** 文件顶部添加以下 `using` 语句：

		using Microsoft.Azure.Devices.Client;
        using Newtonsoft.Json;
        using System.Threading;

5. 将以下字段添加到 **Program** 类，并将占位符值替换为你在*创建 IoT 中心*部分中检索的 IoT 中心主机名，以及你在创建设备标识部分中检索的设备密钥：

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

                Thread.Sleep(1000);
            }
        }

	此方法每隔一秒发送一条新的设备到云消息。该消息包含具有 deviceId 的 JSON 序列化对象和一个随机生成的编号，用于模拟风速传感器。

7. 最后，在 **Main** 方法中添加以下行：

        Console.WriteLine("Simulated device\n");
        deviceClient = DeviceClient.Create(iotHubUri, new DeviceAuthenticationWithRegistrySymmetricKey("myFirstDevice", deviceKey));

        SendDeviceToCloudMessagesAsync();
        Console.ReadLine();

  默认情况下，**Create** 方法将创建使用 AMQP 协议来与 IoT 中心通信的 **DeviceClient**。若要使用 HTTPS 协议，请使用 **Create** 方法的重写，它可以让你指定协议。如果选择使用 HTTPS 协议，则还应在项目中添加 **Microsoft.AspNet.WebApi.Client** NuGet 包，以包含 **System.Net.Http.Formatting** 命名空间。


> [AZURE.NOTE] 为简单起见，本教程不实现任何重试策略。在生产代码中，你应该按 MSDN 文章[暂时性故障处理][lnk-transient-faults]中所述实现重试策略（例如指数退让）。

<!-- Links -->

[lnk-device-nuget]: https://www.nuget.org/packages/Microsoft.Azure.Devices.Client/
[lnk-transient-faults]: https://msdn.microsoft.com/zh-cn/library/hh680901(v=pandp.50).aspx

<!-- Images -->
[30]: ./media/iot-hub-getstarted-device-csharp/create-identity-csharp1.png

<!---HONumber=Mooncake_0321_2016-->