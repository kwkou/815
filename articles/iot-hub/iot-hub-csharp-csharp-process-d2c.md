<properties
	pageTitle="处理 IoT 中心设备到云的消息 | Azure"
	description="遵照本教程了解处理 IoT 中心设备到云消息的有用模式。"
	services="iot-hub"
	documentationCenter=".net"
	authors="dominicbetts"
	manager="timlt"
	editor=""/>

<tags
     ms.service="iot-hub"
     ms.date="04/29/2016"
     wacn.date="08/08/2016"/>

# 教程：如何处理 IoT 中心设备到云的消息

## 介绍

Azure IoT 中心是一项完全托管的服务，可在数百万个 IoT 设备和一个应用程序后端之间实现安全可靠的双向通信。其他教程（[IoT 中心入门]和[使用 IoT 中心发送云到设备的消息][lnk-c2d]）说明了如何使用 IoT 中心的设备到云和云到设备的基本消息传送功能。

本教程以 [IoT 中心入门]中演示的代码为基础，呈现两种可用于处理设备到云消息的可缩放的模式：

- 在 [Azure Blob 存储]中可靠地存储设备到云的消息。一种很常见的情况是*冷路径*分析，因为你会在其中将 blob 中用作输入的遥测数据存储到分析进程中。这些分析进程可由[HDInsight (Hadoop)] 堆栈等工具驱动。

- 可靠处理*交互式*设备到云的消息。如果设备到云的消息因为应用程序后端中的一组操作而立即触发，则表示这些消息是交互式的。例如，设备可能将发送一条警报消息，触发在 CRM 系统中插入票证。与此相反，*数据点*消息则只送入分析引擎。例如，设备中存储以供将来分析使用的温度遥测是数据点消息。

由于 IoT 中心公开[事件中心][lnk-event-hubs]兼容的终结点来接收设备到云的消息，因此本教程使用了 [EventProcessorHost] 实例。此实例：

* 在 Azure blob 存储中可靠地存储*数据点*消息。
* 将*交互式*设备到云的消息转发到 Azure [服务总线队列]以立即进行处理。

服务总线可以帮助确保可靠处理交互式消息，因为它提供了各消息的检查点，以及基于时间范围的重复数据删除。

> [AZURE.NOTE] **EventProcessorHost** 实例只是其中一种处理交互式消息的方法。其他选项包括 [Azure Service Fabric][lnk-service-fabric] 和 [Azure 流分析][lnk-stream-analytics]。

在本教程最后，你将运行三个 Windows 控制台应用：

* **SimulatedDevice**（[Get started with IoT Hub]（IoT 中心入门）教程中创建的应用的修改版本）每秒发送数据点设备到云的消息一次，每 10 秒发送交互式设备到云的消息一次。此应用使用 AMQPS 协议来与 IoT 中心通信。
* **ProcessDeviceToCloudMessages** 使用 [EventProcessorHost] 类从与事件中心兼容的终结点检索消息。然后将数据点消息可靠地存储在 Azure blob 存储中，并将交互式消息转发到服务总线队列。
* **ProcessD2CInteractiveMessages** 将交互式消息从服务总线队列中取消排队。

> [AZURE.NOTE] IoT 中心对许多设备平台和语言（包括 C、Java 和 JavaScript）提供 SDK 支持。有关如何将本教程中所述的模拟设备替换为物理设备，以及一般情况下如何将设备连接到 IoT 中心的分步说明，请参阅 [Azure IoT 开发人员中心]。

本教程直接适用于使用事件中心兼容消息的其他方案，例如 [HDInsight (Hadoop)] 项目。有关详细信息，请参阅 [Azure IoT 中心开发人员指南 - 设备到云]。

若要完成本教程，你需要以下各项：

+ Microsoft Visual Studio 2015。

+ 有效的 Azure 帐户。<br/>如果没有 Azure 订阅，只需要花费几分钟就能创建一个[帐户](/pricing/1rmb-trial/)。

你应该了解 [Azure 存储空间]和 [Azure 服务总线]的一些基本知识。


## 从模拟设备发送交互式消息

在本部分中，你将修改在 [Get started with IoT Hub]（IoT 中心入门）教程中创建的模拟设备应用程序，以便向 IoT 中心发送设备到云的消息。

1. 在 Visual Studio 中的 **SimulatedDevice** 项目内，将以下方法添加到 **Program** 类。

    ```
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
    ```

    此方法非常类似于 **SimulatedDevice** 项目中的 **SendDeviceToCloudMessagesAsync** 方法。唯一的差别是你现在要设置 **MessageId** 系统属性，以及名为 **messageType** 的用户属性。
    代码将向 **MessageId** 属性分配全局唯一的标识符 (GUID)。服务总线可以用它来删除重复接收的消息。本示例使用 **messageType** 属性来区分交互式消息与数据点消息。应用程序将在消息属性而不是在消息正文中传递此信息，因此事件处理器不需要反序列化消息来执行消息路由。

    > [AZURE.NOTE] 在设备代码中创建用于删除重复交互式消息的 **MessageId** 很重要。间歇性网络通信或其他故障可能会造成从设备多次重新传输相同的消息。你也可以使用语义消息 ID（例如相关消息数据字段的哈希）来取代 GUID。

2. 将以下方法添加到 **Main** 方法的 `Console.ReadLine()` 行的前面：

    ````
    SendDeviceToCloudInteractiveMessagesAsync();
    ````

    > [AZURE.NOTE] 为简单起见，本教程不实现任何重试策略。在生产代码中，你应该按 MSDN 文章 [Transient Fault Handling]（暂时性故障处理）中所述实现重试策略（例如指数退让）。

## 处理设备到云的消息

在本部分中，你将创建一个 Windows 控制台应用程序，用于处理来自 IoT 中心的设备到云消息。Iot 中心公开与[事件中心]兼容的终结点，以让应用程序读取设备到云的消息。本教程使用 [EventProcessorHost] 类来处理控制台应用程序中的这些消息。有关如何处理来自事件中心的消息的详细信息，请参阅 [Get Started with Event Hubs]（事件中心入门）教程。

实现可靠的数据点消息存储或交互式消息转发时，所面临的主要挑战在于事件中心的事件处理是依赖消息使用者来提供其进度的检查点。此外，为了达到高吞吐量，当你从事件中心读取时，应在大型批中提供检查点。如果发生失败且还原到先前的检查点，这可能会导致重复处理大量消息。在本教程中，你将了解如何将 Azure 存储空间写入和服务总线重复数据删除时间范围与 **EventProcessorHost** 检查点进行同步。

为了可靠地将消息写入 Azure 存储空间，本示例使用了[块 Blob][Azure Block Blobs] 的单个块提交功能。事件处理器累积内存中的消息，直到达到提供检查点的时间为止（例如在消息的累积缓冲区达到 4 MB 的块大小上限之后，或在超过服务总线重复数据删除时间范围之后）。然后，在检查点之前，代码将新块提交到 Blob。

事件处理器使用事件中心消息偏移作为块 ID。这可以让它在将新块提交到存储空间之前，先执行重复数据删除检查，并处理提交块与检查点之间可能发生的崩溃。

> [AZURE.NOTE] 本教程使用单个存储帐户来写入从 IoT 中心检索的所有消息。若要确定你的解决方案中是否需要使用多个 Azure 存储帐户，请参阅 [Azure Storage scalability Guidelines]（Azure 存储空间可缩放性指导原则）。

应用程序使用服务总线重复数据删除功能，在处理交互式消息时避免重复项目。模拟的设备通过使用唯一的 **MessageId** 为每个交互式消息加上时间戳。这样，服务总线便可确保在指定的重复数据删除时间范围中，没有两个具有相同 **MessageId** 的消息传递给接收方。此重复数据删除功能和服务总线队列所提供的每一消息完成语义，使其能够很容易地实现可靠的交互消息处理。

为确保不在重复数据删除时间范围之外重新提交任何消息，代码同步 **EventProcessorHost** 检查点机制与服务总线队列重复数据删除时间范围。完成此操作的方法是在每次重复数据删除时间范围过去时（在本教程中，该时间范围为 1 小时），强制检查点至少执行一次。

> [AZURE.NOTE] 本教程使用单个分区服务总线队列来处理所有检索自 IoT 中心的交互式消息。有关如何使用服务总线队列来满足解决方案的可缩放性要求的详细信息，请参阅 [Azure Service Bus]（Azure 服务总线）文档。

### 预配 Azure 存储帐户和服务总线队列
若要使用 [EventProcessorHost] 类，必须拥有 Azure 存储帐户才能让 **EventProcessorHost** 记录检查点信息。可以使用现有的存储帐户，或根据 [About Azure Storage]（关于 Azure 存储空间）中的说明创建新的帐户。请记下存储帐户连接字符串。

> [AZURE.NOTE] 复制和粘贴存储帐户连接字符串时，请务必不要包含空格。

你还需要服务总线队列来可靠处理交互式消息。可以编程方式使用 1 小时重复数据删除时间范围创建队列，如[如何使用服务总线队列][Service Bus queue]中所述。或者，也可以遵循以下步骤使用 [Azure 经典门户][lnk-classic-portal]

1. 单击左下角的“新建”。然后单击“应用程序服务”>“服务总线”>“队列”>“自定义创建”。输入名称 **d2ctutorial**，选择区域，并使用现有命名空间或创建一个新的命名空间。在下一页选择“启用重复检测”，并将“重复检测历史记录期限”设为一小时。然后单击右下角的复选标记保存你的队列配置。

    ![在 Azure 门户中创建队列][30]

2. 在服务总线队列列表中单击“d2ctutorial”，然后单击“配置”。创建两个共享访问策略，一个名为 **send**，具有“发送”权限；另一个名为 **listen**，具有“侦听”权限。完成后，单击底部的“保存”。

    ![在 Azure 门户中配置队列][31]

3. 单击顶部的“仪表板”，然后单击底部的“连接信息”。记下两个连接字符串。

    ![Azure 门户中的队列仪表板][32]

### 创建事件处理器

1. 在当前的 Visual Studio 解决方案中，若要使用“控制台应用程序”项目模板创建一个新的 Visual C# Windows 项目，请单击“文件”>“添加”>“新建项目”。确保 .NET Framework 版本为 4.5.1 或更高。将项目命名为 **ProcessDeviceToCloudMessages**，然后单击“确定”。

    ![Visual Studio 中的新项目][10]

2. 在“解决方案资源管理器”中，右键单击“ProcessDeviceToCloudMessages”项目，然后单击“管理 NuGet 包”。此时将显示“NuGet 包管理器”对话框。

3. 搜索 **WindowsAzure.ServiceBus**，单击“安装”并接受使用条款。这样，便会下载、安装 [Azure 服务总线 NuGet 包](https://www.nuget.org/packages/WindowsAzure.ServiceBus)并添加对该程序包的引用，包括其所有依赖项。

4. 搜索 **Microsoft Azure 服务总线事件中心 - EventProcessorHost**，单击“安装”，并接受使用条款。这样，便会下载、安装 [Azure 服务总线事件中心 — EventProcessorHost NuGet 程序包](https://www.nuget.org/packages/Microsoft.Azure.ServiceBus.EventProcessorHost)并添加对该程序包的引用，包括其所有依赖项。

5. 右键单击“ProcessDeviceToCloudMessages”项目，单击“添加”，然后单击“类”。将新类命名为 **StoreEventProcessor**，然后单击“确定”以创建该类。

6. 在 StoreEventProcessor.cs 文件的顶部添加以下语句：

    ```
    using System.IO;
    using System.Diagnostics;
    using System.Security.Cryptography;
    using Microsoft.ServiceBus.Messaging;
    using Microsoft.WindowsAzure.Storage;
    using Microsoft.WindowsAzure.Storage.Blob;
    ```

7. 用以下代码替换该类的正文：

    ```
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
        queueClient = QueueClient.CreateFromConnectionString(ServiceBusConnectionString);
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
    ```

    **EventProcessorHost** 类调用此类以处理从 IoT 中心收到的设备到云消息。此类中的代码实现逻辑，以在 Blob 容器中可靠地存储消息，并将交互式消息转送到服务总线队列。

    **OpenAsync** 方法初始化 **currentBlockInitOffset** 变量，此变量可跟踪此事件处理器所读取的第一个消息的当前偏移。请记住，每个处理器负责单个分区。

    **ProcessEventsAsync** 方法可从 IoT 中心接收一批消息，并按以下方式处理消息：发送交互式消息到服务总线队列，并将数据点消息附加到名为 **toAppend** 的内存缓冲区。如果内存缓冲区达到 4 MB 块限制，或者距离上一个检查点的时间已超过服务总线重复数据删除时间范围（在本教程中为 1 小时），则触发检查点。

    **AppendAndCheckpoint** 方法先为要附加的块生成 blockId。Azure 存储空间要求所有块 ID 都具有相同的长度，此方法才能以前置零来填补偏移 - `currentBlockInitOffset.ToString("0000000000000000000000000")`。如果具有此 ID 的块已在 Blob 中，此方法将使用缓冲区的当前内容将它覆盖。

    > [AZURE.NOTE] 为了简化代码，本教程使用了每个分区的单个 Blob 文件来存储消息。实际的解决方案将实现文件滚动更新，使用的方法是在某段时间之后，或者在文件到达特定大小时（请注意，Azure 块 Blob 大小上限为 195 GB）时创建其他文件。

8. 在 **Program** 类中，在顶部添加以下 **using** 语句：

    ```
    using Microsoft.ServiceBus.Messaging;
    ```

9. 如下所示修改 **Program** 类中的 **Main** 方法。使用名为 **d2ctutorial** 的队列的“发送”权限替换 IoT 中心 **iothubowner** 连接字符串（来自 [IoT 中心入门]教程），存储连接字符串，以及服务总线连接字符串：

    ```
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
    ```

    > [AZURE.NOTE] 为简单起见，本教程使用了 [EventProcessorHost] 类的单个实例。有关详细信息，请参阅 [Event Hubs Programming Guide]（事件中心编程指南）。

## 接收交互式消息
在本部分中，你将编写一个 Windows 控制台应用程序，用于接收来自服务总线队列的交互式消息。有关如何使用服务总线构建解决方案的详细信息，请参阅 [Build multi-tier applications with Service Bus][]（使用服务总线构建多层应用程序）。

1. 在当前的 Visual Studio 解决方案中，使用“控制台应用程序”项目模板创建一个新的 Visual C# Windows 项目。将项目命名为 **ProcessD2CInteractiveMessages**。

2. 在“解决方案资源管理器”中，右键单击“ProcessD2CInteractiveMessages”项目，然后单击“管理 NuGet 包”。此时将显示“NuGet 包管理器”窗口。

3. 搜索 **WindowsAzure.Service Bus**，单击“安装”并接受使用条款。这样，便会下载、安装 [Azure 服务总线](https://www.nuget.org/packages/WindowsAzure.ServiceBus)并添加对该程序包的引用，包括其所有依赖项。

4. 在 **Program.cs** 文件的顶部添加以下 **using** 语句：

    ```
    using System.IO;
    using Microsoft.ServiceBus.Messaging;
    ```

5. 最后，在 **Main** 方法中添加以下行。将连接字符串替换为名为 **d2ctutorial** 的队列的“侦听”权限：

    ```
    Console.WriteLine("Process D2C Interactive Messages app\n");

    string connectionString = "{service bus listen connection string}";
    QueueClient Client = QueueClient.CreateFromConnectionString(connectionString);

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
    ```

## 运行应用程序

现在，你已准备就绪，可以运行应用程序了。

1.	在 Visual Studio 的解决方案资源管理器中右键单击你的解决方案，然后选择“设置启动项目”。选择“多个启动项目”，然后针对 **ProcessDeviceToCloudMessages**、**SimulatedDevice** 和 **ProcessD2CInteractiveMessages** 项目选择“启动”作为操作。

2.	按 **F5** 启动三个控制台应用程序。**ProcessD2CInteractiveMessages** 应用程序应处理从 **SimulatedDevice** 应用程序发送的每条交互式消息。

  ![三个控制台应用程序][50]

> [AZURE.NOTE] 若要查看 Blob 文件中的更新，需要将 **StoreEventProcessor** 类中的 **MAX\_BLOCK\_SIZE** 常量降低为较小值，例如 **1024**。这是因为达到模拟设备所发送的数据块大小限制需要一段时间。如果块大小比较小，则就无需等待这么久才能查看所创建和更新的 Blob。但是，使用较大的块可以提高应用程序的可缩放性。

## 后续步骤

在本教程中，你已学习如何通过使用 [EventProcessorHost] 类可靠地处理数据点与交互式设备到云的消息。

[How to send cloud-to-device messages with IoT Hub][lnk-c2d]（如何使用 IoT 中心发送云到设备的消息）说明了如何从后端向设备发送消息。


若要了解有关使用 IoT 中心开发解决方案的详细信息，请参阅 [IoT Hub Developer Guide]（IoT 中心开发人员指南）。

<!-- Images. -->
[50]: ./media/iot-hub-csharp-csharp-process-d2c/run1.png
[10]: ./media/iot-hub-csharp-csharp-process-d2c/create-identity-csharp1.png

[30]: ./media/iot-hub-csharp-csharp-process-d2c/createqueue2.png
[31]: ./media/iot-hub-csharp-csharp-process-d2c/createqueue3.png
[32]: ./media/iot-hub-csharp-csharp-process-d2c/createqueue4.png

<!-- Links -->

[Azure Blob 存储]: /documentation/articles/storage-dotnet-how-to-use-blobs/

[HDInsight (Hadoop)]: /documentation/services/hdinsight/
[Service Bus Queue]: /documentation/articles/service-bus-dotnet-get-started-with-queues/
[服务总线队列]: /documentation/articles/service-bus-dotnet-get-started-with-queues/



[Azure IoT 中心开发人员指南 - 设备到云]: /documentation/articles/iot-hub-devguide/#d2c

[Azure 存储空间]: /documentation/services/storage/
[Azure Service Bus]: /documentation/services/service-bus/
[Azure 服务总线]: /documentation/services/service-bus/

[IoT Hub Developer Guide]: /documentation/articles/iot-hub-devguide/
[Get started with IoT Hub]: /documentation/articles/iot-hub-csharp-csharp-getstarted/
[IoT 中心入门]: /documentation/articles/iot-hub-csharp-csharp-getstarted/
[Azure IoT 开发人员中心]: /develop/iot
[lnk-service-fabric]: /documentation/services/service-fabric/
[lnk-stream-analytics]: /documentation/services/stream-analytics/
[lnk-event-hubs]: /documentation/services/event-hubs/
[Transient Fault Handling]: https://msdn.microsoft.com/zh-cn/library/hh675232.aspx

<!-- Links -->
[About Azure Storage]: /documentation/articles/storage-create-storage-account/#create-a-storage-account
[Get Started with Event Hubs]: /documentation/articles/event-hubs-csharp-ephcs-getstarted/
[Azure Storage scalability Guidelines]: /documentation/articles/storage-scalability-targets/
[Azure Block Blobs]: https://msdn.microsoft.com/zh-cn/library/azure/ee691964.aspx
[事件中心]: /documentation/articles/event-hubs-overview/
[EventProcessorHost]: http://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.eventprocessorhost(v=azure.95).aspx
[Event Hubs Programming Guide]: /documentation/articles/event-hubs-programming-guide/
[Transient Fault Handling]: https://msdn.microsoft.com/zh-cn/library/hh680901(v=pandp.50).aspx
[Build multi-tier applications with Service Bus]: /documentation/articles/service-bus-dotnet-multi-tier-app-using-service-bus-queues/

[lnk-classic-portal]: https://manage.windowsazure.cn
[lnk-c2d]: /documentation/articles/iot-hub-csharp-csharp-process-d2c/

<!---HONumber=Mooncake_0801_2016-->