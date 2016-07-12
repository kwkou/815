## 从模拟设备上载文件

在本部分中，你将修改在 [使用 IoT 中心发送云到设备的消息] 中创建的模拟设备应用程序，以接收来自 IoT 中心的云到设备消息。

1. 在 Visual Studio 中，右键单击“SimulatedDevice”项目，然后单击“管理 NuGet 包...”。 

    此时将显示“管理 NuGet 包”窗口。

2. 搜索 `WindowsAzure.Storage`、单击“安装”，并接受使用条款。

    这将下载、安装 [Microsoft Azure 存储空间 SDK](https://www.nuget.org/packages/WindowsAzure.Storage/) 并添加对它的引用。

3. 在 Program.cs 文件的顶部添加以下语句：

        using System.IO;
        using Microsoft.WindowsAzure.Storage;
        using Microsoft.WindowsAzure.Storage.Blob;

4. 在 Program 类中，按以下方式更改 ReceiveC2dAsync 方法：
         
        private static async void ReceiveC2dAsync()
        {
            Console.WriteLine("\nReceiving cloud to device messages from service");
            while (true)
            {
                Message receivedMessage = await deviceClient.ReceiveAsync();
                if (receivedMessage == null) continue;

                if (receivedMessage.Properties.ContainsKey("command") && receivedMessage.Properties["command"] == "FileUpload")
                {
                    UploadFileToBlobAsync(receivedMessage);
                    continue;
                }

                Console.ForegroundColor = ConsoleColor.Yellow;
                Console.WriteLine("Received message: {0}", Encoding.ASCII.GetString(receivedMessage.GetBytes()));
                Console.ResetColor();
            }
        }

    这使 ReceiveC2dAsync 可以区分 `command` 属性设为 `FileUpload` 的消息，这些消息将由 UploadFileToBlobAsync 方法进行处理。

    添加以下方法来处理文件上载命令。
   
        private static async Task UploadFileToBlobAsync(Message fileUploadCommand)
        {
            var fileUri = fileUploadCommand.Properties["fileUri"];
            var blob = new CloudBlockBlob(new Uri(fileUri));

            byte[] data = new byte[10 * 1024 * 1024];
            Random rng = new Random();
            rng.NextBytes(data);

            MemoryStream msWrite = new MemoryStream(data);
            msWrite.Position = 0;
            using (msWrite)
            {
                await blob.UploadFromStreamAsync(msWrite);
            }
            Console.ForegroundColor = ConsoleColor.Yellow;
            Console.WriteLine("Uploaded file to: {0}", fileUri);
            Console.ResetColor();

            await deviceClient.CompleteAsync(fileUploadCommand);
        }

    此方法使用 Azure 存储空间 SDK，将随机生成的 10Mb Blob 上载到指定的 URI。有关如何上载 Blob 的详细信息，请参阅 [Azure 存储空间 - 如何使用 Blob]。

> [AZURE.NOTE] 请注意，此模拟设备的实现，只在上载 Blob 后，完成云到设备的消息。此方法可简化后端中已上载文件的处理作业，由于传递通知即表示上载的文件已可供处理。如 [IoT 中心开发人员指南][IoT Hub Developer Guide - C2D]中所述，但在“可见性超时”（通常是 1 分钟）前未完成的消息放回设备队列，而 ReceiveAsync() 方法再次接收该消息。在文件上载时间较长的情况下，可能较适合为模拟设备将当前的上载工作保持长期存放。这可让模拟设备在文件上载完毕之前，完成云到设备的消息，然后发送云到设备消息，以对后端通知工作已完成。

<!-- Links -->
[IoT Hub Developer Guide - C2D]: /documentation/articles/iot-hub-devguide/#c2d
[Azure 存储空间 - 如何使用 Blob]: /documentation/articles/storage-dotnet-how-to-use-blobs/#upload-a-blob-into-a-container

[使用 IoT 中心发送云到设备的消息]: /documentation/articles/iot-hub-csharp-csharp-c2d/

<!-- Images -->

<!---HONumber=Mooncake_0321_2016-->