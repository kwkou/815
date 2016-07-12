## 从应用程序后端发送云到设备的消息

在本部分中，你将编写一个 Windows 控制台应用程序，用于将云到设备的消息发送到模拟设备应用程序。

1. 在当前的 Visual Studio 解决方案中，使用“控制台应用程序”项目模板创建一个新的 Visual C# 桌面应用项目。将项目命名为 SendCloudToDevice。

   	![][20]

2. 在“解决方案资源管理器”中，右键单击该解决方案，然后单击“为解决方案管理 NuGet 包...”。

	此时将显示“管理 NuGet 包”窗口。

3. 搜索 `Microsoft Azure Devices`、单击“安装”，并接受使用条款。

	这将下载、安装 [Azure IoT - 服务 SDK NuGet 包]并添加对它的引用。

4. 在 **Program.cs** 文件顶部添加以下 `using` 语句：

		using Microsoft.Azure.Devices;

5. 将以下字段添加到 **Program** 类，并将占位符值替换为在 [IoT 中心入门]中获取的 IoT 中心连接字符串：

		static ServiceClient serviceClient;
        static string connectionString = "{iot hub connection string}";

6. 将以下方法添加到 **Program** 类：

		private async static Task SendCloudToDeviceMessageAsync()
        {
            var commandMessage = new Message(Encoding.ASCII.GetBytes("Cloud to device message."));
            await serviceClient.SendAsync("myFirstDevice", commandMessage);
        }

	此方法会将新的云到设备消息发送到 ID 为 `myFirstDevice` 的设备。如果你在 [IoT 中心入门]中使用的参数，请相应地更改此参数。

7. 最后，在 **Main** 方法中添加以下行：

        Console.WriteLine("Send Cloud-to-Device message\n");
        serviceClient = ServiceClient.CreateFromConnectionString(connectionString);

        Console.WriteLine("Press any key to send a C2D message.");
        Console.ReadLine();
        SendCloudToDeviceMessageAsync().Wait();
        Console.ReadLine();

8. 在 Visual Studio 中，右键单击你的解决方案并选择“设置启动项目...”。选择“多个启动项目”，然后同时针对 “ProcessDeviceToCloudMessages”、“SimulatedDevice” 和 “SendCloudToDevice” 选择“启动”操作。

9.  按 “F5”，你应会看到所有三个应用程序启动。选择“SendCloudToDevice”窗口并按 “Enter”：你应会看到模拟应用程序正在接收消息。

    ![][21]

## 接收送达反馈
可以从 IoT 中心请求每个云到设备消息的送达（或过期）确认。这样，云后端就能轻松地通知重试或补偿逻辑。有关云到设备的反馈的详细信息，请参阅 [IoT 中心开发人员指南][IoT Hub Developer Guide - C2D]。

在本部分中，你将修改 “SendCloudToDevice” 应用程序以请求反馈，并接收来自 IoT 中心的反馈。

1. 在 Visual Studio 中的 “SendCloudToDevice” 项目内，将以下方法添加到 “Program” 类。
   
        private async static void ReceiveFeedbackAsync()
        {
            var feedbackReceiver = serviceClient.GetFeedbackReceiver();

            Console.WriteLine("\nReceiving c2d feedback from service");
            while (true)
            {
                var feedbackBatch = await feedbackReceiver.ReceiveAsync();
                if (feedbackBatch == null) continue;

                Console.ForegroundColor = ConsoleColor.Yellow;
                Console.WriteLine("Received feedback: {0}", string.Join(", ", feedbackBatch.Records.Select(f => f.StatusCode)));
                Console.ResetColor();

                await feedbackReceiver.CompleteAsync(feedbackBatch);
            }
        }

    请注意，此处的接收模式与用于从设备应用程序接收云到设备消息的模式相同。

2. 将以下方法添加到 “Main” 方法的 `serviceClient = ServiceClient.CreateFromConnectionString(connectionString)` 行之后：

        ReceiveFeedbackAsync();

3. 若要请求针对传递云到设备消息的反馈，必须在 "SendCloudToDeviceMessageAsync" 方法中指定一个属性。将以下行添加到 `var commandMessage = new Message(...);` 行之后：

        commandMessage.Ack = DeliveryAcknowledgement.Full;

4.  按 "F5" 运行应用程序，你应会看到这三个应用程序启动。选择“SendCloudToDevice”窗口并按 **Enter**：你应会看到模拟应用程序收到消息，几秒钟后，**SendCloudToDevice** 应用程序将收到反馈消息。

    ![][22]

> [AZURE.NOTE] 为简单起见，本教程不实现任何重试策略。在生产代码中，建议按 MSDN 文章[暂时性故障处理]中所述实现重试策略（例如指数退让）。

<!-- Links -->

[IoT Hub Developer Guide - C2D]: /documentation/articles/iot-hub-devguide/#c2d
[Azure IoT - 服务 SDK NuGet 包]: https://www.nuget.org/packages/Microsoft.Azure.Devices/
[暂时性故障处理]: https://msdn.microsoft.com/zh-cn/library/hh680901(v=pandp.50).aspx
[IoT 中心入门]: /documentation/articles/iot-hub-csharp-csharp-getstarted/

<!-- Images -->
[20]: ./media/iot-hub-c2d-cloud-csharp/create-identity-csharp1.png
[21]: ./media/iot-hub-c2d-cloud-csharp/sendc2d1.png
[22]: ./media/iot-hub-c2d-cloud-csharp/sendc2d2.png

<!---HONumber=Mooncake_0321_2016-->