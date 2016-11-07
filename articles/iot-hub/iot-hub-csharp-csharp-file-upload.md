<properties
	pageTitle="使用 IoT 中心从设备上载文件 | Microsoft Azure"
	description="遵照本教程了解如何将 Azure IoT 中心与 C# 配合使用，以从设备上载文件。"
	services="iot-hub"
	documentationCenter=".net"
	authors="fsautomata"
	manager="timlt"
	editor=""/>  


<tags
     ms.service="iot-hub"
     ms.devlang="dotnet"
     ms.topic="article"
     ms.tgt_pltfrm="na"
     ms.workload="na"
     ms.date="06/21/2016"
     ms.author="elioda"
     wacn.date="11/07/2016"/>  

# 教程：如何使用 IoT 中心将文件从设备上载到云中

## 介绍

Azure IoT 中心是一项完全托管的服务，可在数百万个 IoT 设备和一个应用程序后端之间实现安全可靠的双向通信。以前的教程（[Get started with IoT Hub]（IoT 中心入门）和 [Send Cloud-to-Device messages with IoT Hub]（使用 IoT 中心发送云到设备的消息））演示了 IoT 中心的设备到云和云到设备的基本消息传送功能，[Process Device-to-Cloud messages]（处理设备到云的消息）教程介绍了在 Azure Blob 存储中可靠存储设备到云消息的方法。但是，在某些情况下，你无法轻松地将设备发送的数据映射为 IoT 中心接受的相对较小的设备到云消息。例如，包含图像、视频、以高频率采样的震动数据或者某种形式的预处理数据的大型文件。通常使用 [Hadoop] 堆栈等工具在云中批处理这些文件。如果你偏好通过从设备上载文件来发送事件，仍可以使用 IoT 中心的安全性与可靠性功能。

本教程的内容基于 [Send Cloud-to-Device messages with IoT Hub]（使用 IoT 中心发送云到设备的消息）教程中所述的代码，演示如何使用 IoT 中心的文件上载功能。其中演示了如何通过 Azure Blob URI 安全地提供一个设备用于上载文件，以及如何使用 IoT 中心文件上载通知在应用程序后端触发文件处理。

在本教程最后，你将运行两个 Windows 控制台应用程序：

* **SimulatedDevice**，这是 [Send Cloud-to-Device messages with IoT Hub]（使用 IoT 中心发送云到设备的消息）教程中创建的应用的修改版本，它可以使用 IoT 中心提供的 SAS URI 将文件上载到存储。
* **ReadFileUploadNotification**，它可以接收来自 IoT 中心的文件上载通知。

> [AZURE.NOTE] IoT 中心通过 Azure IoT 设备 SDK 来支持许多设备平台和语言（包括 C、Java 和 Javascript）。有关如何将设备连接到本教程中所示的代码（通常是连接到 Azure IoT 中心）的逐步说明，请参阅 [Azure IoT Developer Center]（Azure IoT 开发人员中心）。

完成本教程需要具备以下项：

+ Microsoft Visual Studio 2015。

+ 有效的 Azure 帐户。<br/>如果你没有帐户，只需花费几分钟就能创建一个试用帐户。有关详细信息，请参阅 [Azure 试用](/pricing/1rmb-trial/)。

## 将 Azure 存储帐户关联到 IoT 中心

由于模拟设备将文件上载到 Azure 存储 Blob，因此必须拥有与 IoT 中心关联的 [Azure 存储]帐户。将存储帐户与 IoT 中心关联时，中心将生成一个 SAS URI，设备可以使用它安全地将文件上载到 Blob 容器。IoT 中心服务和设备 SDK 协调生成 SAS URI 的过程，并使其可供设备用来上载文件。

根据 [Configure file uploads using the Azure portal][lnk-configure-upload]（通过 Azure 门户配置文件上载）中的说明，将 Azure 存储帐户关联到 IoT 中心。

## 从模拟设备上载文件

在本部分中，你将修改在 [Send Cloud-to-Device messages with IoT Hub]（使用 IoT 中心发送云到设备的消息）中创建的模拟设备应用程序，以接收来自 IoT 中心的云到设备消息。

1. 在 Visual Studio 中，右键单击“SimulatedDevice”项目，单击“添加”，然后单击“现有项”。导航到某个图像文件并将它包含在项目中。本教程假设图像名称为 `image.jpg`。

2. 右键单击该图像，然后单击“属性”。确保“复制到输出目录”设置为“始终复制”。

    ![][1]

3. 在 **Program.cs** 文件的顶部添加以下语句：

        using System.IO;

4. 将以下方法添加到 **Program** 类：
         
        private static async void SendToBlobAsync()
        {
            string fileName = "image.jpg";
            Console.WriteLine("Uploading file: {0}", fileName);
            var watch = System.Diagnostics.Stopwatch.StartNew();

            using (var sourceData = new FileStream(@"image.jpg", FileMode.Open))
            {
                await deviceClient.UploadToBlobAsync(fileName, sourceData);
            }

            watch.Stop();
            Console.WriteLine("Time to upload file: {0}ms\n", watch.ElapsedMilliseconds);
        }

    `UploadToBlobAsync` 方法获取要上载的文件的文件名与流源，然后处理上载到存储的任务。控制台应用程序将显示上载文件所要花费的时间。

5. 将以下方法添加到 **Main** 方法的 `Console.ReadLine()` 行的前面：

        SendToBlobAsync();

> [AZURE.NOTE] 为简单起见，本教程不实现任何重试策略。在生产代码中，应按 MSDN 文章 [Transient Fault Handling]（暂时性故障处理）中所述实施重试策略（例如指数性的回退）。

## 接收文件上载通知

在本部分中，你将编写一个 Windows 控制台应用，用于接收来自 IoT 中心的文件上载通知消息。

1. 在当前的 Visual Studio 解决方案中，使用“控制台应用程序”项目模板创建一个新的 Visual C# Windows 项目。将项目命名为 **ReadFileUploadNotification**。

    ![Visual Studio 中的新项目][2]

2. 在“解决方案资源管理器”中，右键单击“ReadFileUploadNotification”项目，然后单击“管理 NuGet 包”。

    此时将显示“管理 NuGet 包”窗口。

2. 搜索 `Microsoft.Azure.Devices`、单击“安装”，并接受使用条款。

	随后将会下载、安装 [Azure IoT - 服务 SDK NuGet 包]并在 **ReadFileUploadNotification** 项目中添加对它的引用。

3. 在 **Program.cs** 文件的顶部添加以下语句：

        using Microsoft.Azure.Devices;

4. 将以下字段添加到 **Program** 类。将占位符值替换为在 [Get started with IoT Hub]（IoT 中心入门）中获取的 IoT 中心连接字符串：

		static ServiceClient serviceClient;
        static string connectionString = "{iot hub connection string}";
        
5. 将以下方法添加到 **Program** 类：
   
        private async static Task ReceiveFileUploadNotificationAsync()
        {
            var notificationReceiver = serviceClient.GetFileNotificationReceiver();

            Console.WriteLine("\nReceiving file upload notification from service");
            while (true)
            {
                var fileUploadNotification = await notificationReceiver.ReceiveAsync();
                if (fileUploadNotification == null) continue;

                Console.ForegroundColor = ConsoleColor.Yellow;
                Console.WriteLine("Received file upload noticiation: {0}", string.Join(", ", fileUploadNotification.BlobName));
                Console.ResetColor();

                await notificationReceiver.CompleteAsync(fileUploadNotification);
            }
        }

    请注意，此处的接收模式与用于从设备应用程序接收云到设备消息的模式相同。

6. 最后，在 **Main** 方法中添加以下行：

        Console.WriteLine("Receive file upload notifications\n");
        serviceClient = ServiceClient.CreateFromConnectionString(connectionString);
        ReceiveFileUploadNotificationAsync().Wait();
        Console.ReadLine();

## 运行应用程序

现在，你已准备就绪，可以运行应用程序了。

1. 在 Visual Studio 中，右键单击你的解决方案并选择“设置启动项目”。选择“多个启动项目”，然后针对 **ReadFileUploadNotification** 和 **SimulatedDevice** 应用选择“启动”操作。

2. 按 **F5**。这两个应用程序应该都会启动。你将在其中一个控制台应用中看到上载已完成，同时还会看到另一个控制台应用收到的上载通知消息。可以使用 [Azure 门户预览]或 Visual Studio 服务器资源管理器检查存储帐户中是否存在上载的文件。

  ![][50]


## 后续步骤

在本教程中，你已学习如何利用 IoT 中心的文件上载功能来帮助从设备上载文件。可以使用以下文章继续探索 IoT 中心功能和方案：

- [以编程方式创建 IoT 中心][lnk-create-hub]
- [C SDK 简介][lnk-c-sdk]
- [IoT 中心 SDK][lnk-sdks]

若要进一步探索 IoT 中心的功能，请参阅：

- [使用网关 SDK 模拟设备][lnk-gateway]

<!-- Images. -->


[50]: ./media/iot-hub-csharp-csharp-file-upload/run-apps1.png
[1]: ./media/iot-hub-csharp-csharp-file-upload/image-properties.png
[2]: ./media/iot-hub-csharp-csharp-file-upload/create-identity-csharp1.png

<!-- Links -->


[Azure 门户预览]: https://portal.azure.cn/


[Hadoop]: /documentation/services/hdinsight/

[Send Cloud-to-Device messages with IoT Hub]: /documentation/articles/iot-hub-csharp-csharp-c2d/
[Process Device-to-Cloud messages]: /documentation/articles/iot-hub-csharp-csharp-process-d2c/
[Get started with IoT Hub]: /documentation/articles/iot-hub-csharp-csharp-getstarted/
[Azure IoT Developer Center]: /develop/iot

[Transient Fault Handling]: https://msdn.microsoft.com/zh-cn/library/hh680901(v=pandp.50).aspx
[Azure 存储]: /documentation/articles/storage-create-storage-account/#create-a-storage-account
[lnk-configure-upload]: /documentation/articles/iot-hub-configure-file-upload/
[Azure IoT - 服务 SDK NuGet 包]: https://www.nuget.org/packages/Microsoft.Azure.Devices/
[lnk-free-trial]: /pricing/1rmb-trial/

[lnk-create-hub]: /documentation/articles/iot-hub-rm-template-powershell/
[lnk-c-sdk]: /documentation/articles/iot-hub-device-sdk-c-intro/
[lnk-sdks]: /documentation/articles/iot-hub-devguide-sdks/

[lnk-gateway]: /documentation/articles/iot-hub-linux-gateway-sdk-simulated-device/


<!---HONumber=Mooncake_0307_2016-->