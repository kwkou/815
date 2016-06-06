## 在模拟设备上接收消息

在本部分中，你将修改在 [IoT 中心入门] 中创建的模拟设备应用程序，以接收来自 IoT 中心的云到设备消息。

1. 在 Visual Studio 中的 SimulatedDevice 项目内，将以下方法添加到 Program 类。

        private static async void ReceiveC2dAsync()
        {
            Console.WriteLine("\nReceiving cloud to device messages from service");
            while (true)
            {
                Message receivedMessage = await deviceClient.ReceiveAsync();
                if (receivedMessage == null) continue;

                Console.ForegroundColor = ConsoleColor.Yellow;
                Console.WriteLine("Received message: {0}", Encoding.ASCII.GetString(receivedMessage.GetBytes()));
                Console.ResetColor();

                await deviceClient.CompleteAsync(receivedMessage);
            }
        }

    在设备收到消息时，`ReceiveAsync` 方法以异步方式返回收到的消息。它在可指定的超时期限过后返回 null（在本例中，使用的是默认值 1 分钟）。当发生这种情况时，我们想让代码继续等待新消息。因此加上 `if (receivedMessage == null) continue` 行。

    对 `CompleteAsync()` 的调用将通知 IoT 中心，指出已成功处理消息，且可以安全地从设备队列中删除该消息。如果因故导致设备应用程序无法完成消息处理作业，IoT 中心将再传递一次；因此设备应用程序中的消息处理逻辑必须是“幂等的”，以便接收多次相同的消息可生成相同的结果。应用程序还可以暂时 `Abandon` 消息，让 IoT 中心将消息保留在队列中，以供之后使用；或者 `Reject` 消息，以永久性从队列中删除消息。有关云到设备消息生命周期的详细信息，请参阅 [IoT 中心开发人员指南][IoT Hub Developer Guide - C2D]。

    > [AZURE.NOTE] 使用 HTTP/1 而不使用 AMQP 作为传输时，`ReceiveAsync` 将立即返回。使用 HTTP/1 的云到设备消息，其支持模式是间歇连接到设备，且不常检查消息（换言之，比每隔 25 分钟更少）。发出更多 HTTP/1 接收会导致 IoT 中心限制请求。有关 AMQP 和 HTTP/1 支持之间的差异，以及 IoT 中心限制的详细信息，请参阅 [IoT 中心开发人员指南][IoT Hub Developer Guide - C2D]。

2. 将以下方法添加到 Main 方法的 `Console.ReadLine()` 行的前面：

        ReceiveC2dAsync();

    > [AZURE.NOTE] 为简单起见，本教程不实现任何重试策略。在生产代码中，建议按 MSDN 文章[暂时性故障处理]中所述实现重试策略（例如指数退让）。

<!-- Links -->
[IoT Hub Developer Guide - C2D]: /documentation/articles/iot-hub-devguide/#c2d
[暂时性故障处理]: https://msdn.microsoft.com/zh-cn/library/hh680901(v=pandp.50).aspx

<!-- Images -->

<!---HONumber=Mooncake_0321_2016-->