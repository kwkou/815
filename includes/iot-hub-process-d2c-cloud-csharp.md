## 处理设备到云的消息

在本部分中，你将创建一个 Windows 控制台应用程序，用于处理来自 IoT 中心的设备到云消息。Iot 中心公开与[事件中心]兼容的终结点，以让应用程序读取设备到云的消息。本教程使用 [EventProcessorHost] 类来处理控制台应用程序中的这些消息。有关如何处理来自事件中心的消息的详细信息，请参阅[事件中心入门]教程。

实现可靠的数据点消息存储或交互式消息转发时，所面临的主要挑战在于事件中心的事件处理是依赖消息使用者来创建进度的检查点。此外，为了达到高吞吐量，当你从事件中心读取时，应在大型批中创建检查点。如果发生失败且还原到先前的检查点，这可能会导致重复处理大量消息。在本教程中，你将了解如何将 Azure 存储空间写入和服务总线重复数据删除时间范围与 EventProcessorHost 检查点进行同步。

为了可靠地将消息写入 Azure 存储空间，本示例使用了[块 Blob][Azure Block Blobs] 的单个块提交功能。事件处理器累积内存中的消息，直到达到执行检查点的时间为止（例如在消息的累积缓冲区达到 4 Mb 的块大小上限之后，或在超过服务总线重复数据删除时间范围之后）。然后，在创建检查点之前，代码将新块提交到 Blob。

事件处理器使用事件中心消息偏移作为块 ID。这可以让它在将新块提交到存储空间之前，先执行重复数据删除检查，并处理提交块与检查点之间可能发生的崩溃。

> [AZURE.NOTE] 本教程使用单个存储帐户来写入从 IoT 中心检索的所有消息。请参阅 [Azure 存储空间可缩放性指导原则]来确定你的解决方案中是否需要使用多个 Azure 存储帐户。

应用程序使用服务总线重复数据删除功能，在处理交互式消息时避免重复项目。模拟的设备通过使用唯一的 MessageId 为每个交互式消息加上时间戳，让服务总线能够确保在指定的重复数据删除时间范围中，没有两个具有相同 MessageId 的消息传递给接收方。此重复数据删除功能和服务总线队列所提供的每一消息完成语义，使其能够很容易地实现可靠的交互消息处理。

为确保不在重复数据删除时间范围之外重新提交任何消息，代码同步 EventProcessorHost 检查点机制与服务总线队列重复数据删除时间范围。完成此操作的方法是在每次重复数据删除时间范围过去时（在本教程中为 1 小时），强制检查点至少执行一次。

> [AZURE.NOTE] 本教程使用单个分区服务总线队列来处理所有检索自 IoT 中心的交互式消息。请参阅[服务总线文档]，以获取有关如何使用服务总线队列来满足解决方案的可缩放性要求的详细信息。

### 预配 Azure 存储帐户和服务总线队列
为使用 [EventProcessorHost] 类，必须拥有 Azure 存储帐户才能让 EventProcessorHost 记录检查点信息。可以使用现有的存储帐户，或是根据[关于 Azure 存储空间]中的说明创建新的帐户。请记下存储帐户连接字符串。

> [AZURE.NOTE] 复制和粘贴存储帐户连接字符串时，请确保连接字符串中不包含空格。
你还需要服务总线队列来可靠处理交互式消息。可以编程方式使用 1 小时重复数据删除窗口创建队列，如[如何使用服务总线队列][Service Bus Queue]中所述，或遵循以下步骤使用 [Azure 门户]：

1. 单击左下角的“新建”，依次单击“应用程序服务”、“服务总线”、“队列”、“自定义创建”，输入名称 d2ctutorial，选择区域，使用现有命名空间或创建一个新的命名空间，然后在下一页选择“启用重复检测”，并将“重复检测历史记录期限”设为一小时。然后单击复选标记以保存你的队列配置。

    ![][30]

2. 在服务总线队列列表中单击“d2ctutorial”，然后单击“配置”。创建两个共享访问策略，一个名为 send，具有发送权限；另一个名为 listen，具有侦听权限。完成时单击底部的“保存”。

    ![][31]

3. 单击顶部的“仪表板”，然后单击底部的“连接信息”，并记下这两个连接字符串。

    ![][32]

### 创建事件处理器

1. 在当前的 Visual Studio 解决方案中，单击“文件”->“添加”->“新建项目”，以使用“控制台应用程序”项目模板创建新的 Visual C# Windows 项目。确保 .NET Framework 版本为 4.5.1 或更高。将项目命名为 “ProcessDeviceToCloudMessages”。

    ![][10]

2. 在“解决方案资源管理器”中，右键单击“ProcessDeviceToCloudMessages”项目，然后单击“管理 NuGet 包”。此时将显示“NuGet 包管理器”对话框。

3. 搜索“WindowsAzure.ServiceBus”，单击“安装”并接受使用条款。这样，便会下载、安装 [Azure 服务总线 NuGet 包](https://www.nuget.org/packages/WindowsAzure.ServiceBus)并添加对该程序包的引用，包括其所有依赖项。

4. 搜索“Microsoft Azure 服务总线事件中心 - EventProcessorHost”，单击“安装”，并接受使用条款。这样，便会下载、安装 [Azure 服务总线事件中心 — EventProcessorHost NuGet 程序包](https://www.nuget.org/packages/Microsoft.Azure.ServiceBus.EventProcessorHost)并添加对该程序包的引用，包括其所有依赖项。

5. 右键单击“ProcessDeviceToCloudMessages”项目，单击“添加”，然后单击“类”。将新类命名为 StoreEventProcessor，然后单击“确定”以创建该类。

6. 在 StoreEventProcessor.cs 文件的顶部添加以下语句：

        using System.IO;
        using System.Diagnostics;
        using System.Security.Cryptography;
        using Microsoft.ServiceBus.Messaging;
        using Microsoft.WindowsAzure.Storage;
        using Microsoft.WindowsAzure.Storage.Blob;

7. 用以下代码替换该类的正文：

        class StoreEventProcessor : IEventProcessor
        {
          private const int MAX_BLOCK_SIZE = 4 * 1024 * 1024;
          public static string StorageConnectionString;
          public static string ServiceBusConnectionString;
    
          private CloudBlobClient blobClient;
          private CloudBlobContainer blobContainer;
          private QueueClient queueClient;
    
          private long currentBlockInitOffset;
          private MemoryStream toAppend = new MemoryStream(MAX_BLOCK_SIZE);
    
          private Stopwatch stopwatch;
          private TimeSpan MAX_CHECKPOINT_TIME = TimeSpan.FromHours(1);
    
          public StoreEventProcessor()
          {
            var storageAccount = CloudStorageAccount.Parse(StorageConnectionString);
            blobClient = storageAccount.CreateCloudBlobClient();
            blobContainer = blobClient.GetContainerReference("d2ctutorial");
            blobContainer.CreateIfNotExists();
            queueClient = QueueClient.CreateFromConnectionString(ServiceBusConnectionString, "d2ctutorial");
          }
    
          Task IEventProcessor.CloseAsync(PartitionContext context, CloseReason reason)
          {
            Console.WriteLine("Processor Shutting Down. Partition '{0}', Reason: '{1}'.", context.Lease.PartitionId, reason);
            return Task.FromResult<object>(null);
          }
    
          Task IEventProcessor.OpenAsync(PartitionContext context)
          {
            Console.WriteLine("StoreEventProcessor initialized.  Partition: '{0}', Offset: '{1}'", context.Lease.PartitionId, context.Lease.Offset);
    
            if (!long.TryParse(context.Lease.Offset, out currentBlockInitOffset))
            {
              currentBlockInitOffset = 0;
            }
            stopwatch = new Stopwatch();
            stopwatch.Start();
    
            return Task.FromResult<object>(null);
          }
    
          async Task IEventProcessor.ProcessEventsAsync(PartitionContext context, IEnumerable<EventData> messages)
          {
            foreach (EventData eventData in messages)
            {
              byte[] data = eventData.GetBytes();
    
              if (eventData.Properties.ContainsKey("messageType") && (string) eventData.Properties["messageType"] == "interactive")
              {
                var messageId = (string) eventData.SystemProperties["message-id"];
    
                var queueMessage = new BrokeredMessage(new MemoryStream(data));
                queueMessage.MessageId = messageId;
                queueMessage.Properties["messageType"] = "interactive";
                await queueClient.SendAsync(queueMessage);
    
                WriteHighlightedMessage(string.Format("Received interactive message: {0}", messageId));
                continue;
              }
    
              if (toAppend.Length + data.Length > MAX_BLOCK_SIZE || stopwatch.Elapsed > MAX_CHECKPOINT_TIME)
              {
                await AppendAndCheckpoint(context);
              }
              await toAppend.WriteAsync(data, 0, data.Length);
    
              Console.WriteLine(string.Format("Message received.  Partition: '{0}', Data: '{1}'",
                context.Lease.PartitionId, Encoding.UTF8.GetString(data)));
            }
          }
    
          private async Task AppendAndCheckpoint(PartitionContext context)
          {
            var blockIdString = String.Format("startSeq:{0}", currentBlockInitOffset.ToString("0000000000000000000000000"));
            var blockId = Convert.ToBase64String(ASCIIEncoding.ASCII.GetBytes(blockIdString));
            toAppend.Seek(0, SeekOrigin.Begin);
            byte[] md5 = MD5.Create().ComputeHash(toAppend);
            toAppend.Seek(0, SeekOrigin.Begin);
    
            var blobName = String.Format("iothubd2c_{0}", context.Lease.PartitionId);
            var currentBlob = blobContainer.GetBlockBlobReference(blobName);
    
            if (await currentBlob.ExistsAsync())
            {
              await currentBlob.PutBlockAsync(blockId, toAppend, Convert.ToBase64String(md5));
              var blockList = await currentBlob.DownloadBlockListAsync();
              var newBlockList = new List<string>(blockList.Select(b => b.Name));
    
              if (newBlockList.Count() > 0 && newBlockList.Last() != blockId)
              {
                newBlockList.Add(blockId);
                WriteHighlightedMessage(String.Format("Appending block id: {0} to blob: {1}", blockIdString, currentBlob.Name));
              }
              else
              {
                WriteHighlightedMessage(String.Format("Overwriting block id: {0}", blockIdString));
              }
              await currentBlob.PutBlockListAsync(newBlockList);
            }
            else
            {
              await currentBlob.PutBlockAsync(blockId, toAppend, Convert.ToBase64String(md5));
              var newBlockList = new List<string>();
              newBlockList.Add(blockId);
              await currentBlob.PutBlockListAsync(newBlockList);
    
              WriteHighlightedMessage(String.Format("Created new blob", currentBlob.Name));
            }
    
            toAppend.Dispose();
            toAppend = new MemoryStream(MAX_BLOCK_SIZE);
    
            // checkpoint.
            await context.CheckpointAsync();
            WriteHighlightedMessage(String.Format("Checkpointed partition: {0}", context.Lease.PartitionId));
    
            currentBlockInitOffset = long.Parse(context.Lease.Offset);
            stopwatch.Restart();
          }
    
          private void WriteHighlightedMessage(string message)
          {
            Console.ForegroundColor = ConsoleColor.Yellow;
            Console.WriteLine(message);
            Console.ResetColor();
          }
        }

    EventProcessorHost 类调用此类以处理从 IoT 中心收到的设备到云消息。此类中的代码实现逻辑，以在 Blob 容器中可靠地存储消息，并将交互式消息转送到服务总线队列。OpenAsync 方法初始化 currentBlockInitOffset 变量，此变量可跟踪此事件处理器所读取的第一个消息的当前偏移。请记住，每个处理器负责单个分区。
    
    ProcessEventsAsync 方法可从 IoT 中心接收一批消息，并按以下方式处理消息：发送交互式消息到服务总线队列，并将数据点消息附加到名为 toAppend 的内存缓冲区。如果内存缓冲区达到 4 Mb 块限制，或者距离上一个检查点的时间已超过服务总线重复数据删除时间范围（在本教程中为 1 小时），则触发检查点。

    AppendAndCheckpoint 方法先为要附加的块生成 blockId。Azure 存储空间要求所有块 ID 都具有相同的长度，此方法才能以前置零来填补偏移 - `currentBlockInitOffset.ToString("0000000000000000000000000")`。如果具有此 ID 的块已在 Blob 中，此方法将使用缓冲区的当前内容将它覆盖。

    > [AZURE.NOTE] 为了简化代码，本教程使用了每个分区的单个 Blob 文件来存储消息。实际的解决方案将实现文件滚动更新，使用的方法是在文件到达特定大小时（请注意，Azure 块 Blob 大小上限为 195 Gb），或在某段时间之后创建其他文件。

8. 在 Program 类中，在顶部添加以下 using 语句：

        using Microsoft.ServiceBus.Messaging;

9. 如下所示修改 Program 类中的 Main 方法，使用名为 d2ctutorial 之队列的“发送”权限替换 IoT 中心 iothubowner 连接字符串（来自 [IoT 中心入门]教程），存储连接字符串，以及服务总线连接字符串：

        static void Main(string[] args)
        {
          string iotHubConnectionString = "{iot hub connection string}";
          string iotHubD2cEndpoint = "messages/events";
          StoreEventProcessor.StorageConnectionString = "{storage connection string}";
          StoreEventProcessor.ServiceBusConnectionString = "{service bus send connection string}";
    
          string eventProcessorHostName = Guid.NewGuid().ToString();
          EventProcessorHost eventProcessorHost = new EventProcessorHost(eventProcessorHostName, iotHubD2cEndpoint, EventHubConsumerGroup.DefaultGroupName, iotHubConnectionString, StoreEventProcessor.StorageConnectionString, "messages-events");
          Console.WriteLine("Registering EventProcessor...");
          eventProcessorHost.RegisterEventProcessorAsync<StoreEventProcessor>().Wait();
    
          Console.WriteLine("Receiving. Press enter key to stop worker.");
          Console.ReadLine();
          eventProcessorHost.UnregisterEventProcessorAsync().Wait();
        }
        
    > [AZURE.NOTE] 为简单起见，本教程使用了 [EventProcessorHost] 类的单个实例。有关详细信息，请参阅[事件中心编程指南]。

## 接收交互式消息
在本部分中，你将编写一个 Windows 控制台应用程序，用于接收来自服务总线队列的交互式消息。有关如何使用服务总线构建解决方案的详细信息，请参阅[使用服务总线构建多层应用程序][]。

1. 在当前的 Visual Studio 解决方案中，使用“控制台应用程序”项目模板创建一个新的 Visual C# Windows 项目。将项目命名为 “ProcessD2CInteractiveMessages”。

2. 在“解决方案资源管理器”中，右键单击“ProcessD2CInteractiveMessages”项目，然后单击“管理 NuGet 包”。此时将显示“NuGet 包管理器”窗口。

3. 搜索“WindowsAzure.Service Bus”，单击“安装”并接受使用条款。这样，便会下载、安装 [Azure 服务总线](https://www.nuget.org/packages/WindowsAzure.ServiceBus)并添加对该程序包的引用，包括其所有依赖项。

4. 在 **Program.cs** 文件的顶部添加以下 **using** 语句：

        using System.IO;
        using Microsoft.ServiceBus.Messaging;

5. 最后，将以下几行添加到 **Main** 方法，将连接字符串替换为名为 **d2ctutorial** 的队列的**侦听**权限：

        Console.WriteLine("Process D2C Interactive Messages app\n");
    
        string connectionString = "{service bus listen connection string}";
        QueueClient Client = QueueClient.CreateFromConnectionString(connectionString, "d2ctutorial");
    
        OnMessageOptions options = new OnMessageOptions();
        options.AutoComplete = false;
        options.AutoRenewTimeout = TimeSpan.FromMinutes(1);
    
        Client.OnMessage((message) =>
        {
          try
          {
            var bodyStream = message.GetBody<Stream>();
            bodyStream.Position = 0;
            var bodyAsString = new StreamReader(bodyStream, Encoding.ASCII).ReadToEnd();
    
            Console.WriteLine("Received message: {0} messageId: {1}", bodyAsString, message.MessageId);
    
            message.Complete();
          }
          catch (Exception)
          {
            message.Abandon();
          }
        }, options);
    
        Console.WriteLine("Receiving interactive messages from SB queue...");
        Console.WriteLine("Press any key to exit.");
        Console.ReadLine();

<!-- Links -->
[关于 Azure 存储空间]: /documentation/articles/storage-create-storage-account/#create-a-storage-account
[Azure IoT - Service SDK NuGet package]: https://www.nuget.org/packages/Microsoft.Azure.Devices/
[事件中心入门]: /documentation/articles/event-hubs-csharp-ephcs-getstarted/
[IoT Hub Developer Guide - Identity Registry]: /documentation/articles/iot-hub-devguide/#identityregistry
[Azure 存储空间可缩放性指导原则]: /documentation/articles/storage-scalability-targets/
[Azure Block Blobs]: https://msdn.microsoft.com/zh-cn/library/azure/ee691964.aspx
[事件中心]: /documentation/articles/event-hubs-overview/
[Scaled out event processing]: https://code.msdn.microsoft.com/windowsazure/Service-Bus-Event-Hub-45f43fc3
[EventProcessorHost]: http://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.eventprocessorhost(v=azure.95).aspx
[事件中心编程指南]: /documentation/articles/event-hubs-programming-guide/
[Transient Fault Handling]: https://msdn.microsoft.com/zh-cn/library/hh680901(v=pandp.50).aspx
[Azure 门户]: https://manage.windowsazure.cn/
[Service Bus Queue]: /documentation/articles/service-bus-dotnet-how-to-use-queues/
[使用服务总线构建多层应用程序]: /documentation/articles/service-bus-dotnet-multi-tier-app-using-service-bus-queues/
[IoT 中心入门]: /documentation/articles/iot-hub-csharp-csharp-getstarted/
[服务总线文档]: /documentation/services/service-bus/

<!-- Images -->
[10]: ./media/iot-hub-process-d2c-cloud-csharp/create-identity-csharp1.png
[12]: ./media/iot-hub-process-d2c-cloud-csharp/create-identity-csharp3.png

[20]: ./media/iot-hub-process-d2c-cloud-csharp/create-storage1.png
[21]: ./media/iot-hub-process-d2c-cloud-csharp/create-storage2.png
[22]: ./media/iot-hub-process-d2c-cloud-csharp/create-storage3.png

[30]: ./media/iot-hub-process-d2c-cloud-csharp/createqueue2.png
[31]: ./media/iot-hub-process-d2c-cloud-csharp/createqueue3.png
[32]: ./media/iot-hub-process-d2c-cloud-csharp/createqueue4.png

<!---HONumber=Mooncake_0321_2016-->