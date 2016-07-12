## 从模拟设备发送交互式消息

在本部分中，你将修改在 [IoT 中心入门]中创建的模拟设备应用程序，以便向 IoT 中心发送设备到云的消息。

1. 在 Visual Studio 中的 **SimulatedDevice** 项目内，将以下方法添加到 **Program** 类。

        private static async void SendDeviceToCloudInteractiveMessagesAsync()
        {
          while (true)
          {
            var interactiveMessageString = "Alert message!";
            var interactiveMessage = new Message(Encoding.ASCII.GetBytes(interactiveMessageString));
            interactiveMessage.Properties["messageType"] = "interactive";
            interactiveMessage.MessageId = Guid.NewGuid().ToString();
    
            await deviceClient.SendEventAsync(interactiveMessage);
            Console.WriteLine("{0} > Sending interactive message: {1}", DateTime.Now, interactiveMessageString);
    
            Task.Delay(10000).Wait();
          }
        }

    此方法非常类似于 **SimulatedDevice** 项目中的 **SendDeviceToCloudMessagesAsync** 方法。唯一的差别是你现在要设置 **MessageId** 系统属性，以及名为 **messageType** 的用户属性。代码将向 **MessageId** 属性分配全局唯一的标识符 (guid)，服务总线可以用它来删除重复接收的消息。本示例使用 **messageType** 属性来区分交互式消息与数据点消息。应用程序将在消息属性而不是在消息正文中传递此信息，因此事件处理器不需要反序列化消息来执行消息路由。

    > [AZURE.NOTE] 在设备代码中创建用于删除重复交互式消息的 **MessageId** 很重要，因为间歇性网络通信或其他失败可能会造成从设备多次重新传输相同的消息。你也可以使用语义消息 ID（例如相关消息数据字段的哈希）来取代 guid。

2. 将以下方法添加到 **Main** 方法的 `Console.ReadLine()` 行的前面：

    
        SendDeviceToCloudInteractiveMessagesAsync();


    > [AZURE.NOTE] 为简单起见，本教程不实现任何重试策略。在生产代码中，你应该按 MSDN 文章[暂时性故障处理]中所述实现重试策略（例如指数退让）。

<!-- Links -->
[暂时性故障处理]: https://msdn.microsoft.com/zh-cn/library/hh675232.aspx
[IoT 中心入门]: /documentation/articles/iot-hub-csharp-csharp-getstarted/

<!---HONumber=Mooncake_0321_2016-->