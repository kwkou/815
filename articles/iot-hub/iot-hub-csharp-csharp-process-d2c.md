<properties
	pageTitle="处理 IoT 中心设备到云的消息 (.Net) | Azure"
	description="遵照本教程了解处理 IoT 中心设备到云消息的有用模式。"
	services="iot-hub"
	documentationCenter=".net"
	authors="dominicbetts"
	manager="timlt"
	editor=""/>  


<tags
     ms.service="iot-hub"
     ms.devlang="csharp"
     ms.topic="article"
     ms.tgt_pltfrm="na"
     ms.workload="na"
     ms.date="10/05/2016"
     wacn.date="01/04/2017"
     ms.author="dobett"/>

# 教程：如何使用 .Net 处理 IoT 中心设备到云的消息

[AZURE.INCLUDE [iot-hub-selector-process-d2c](../../includes/iot-hub-selector-process-d2c.md)]

## 介绍

Azure IoT 中心是一项完全托管的服务，可在数百万个 IoT 设备和一个应用程序后端之间实现安全可靠的双向通信。其他教程（[IoT 中心入门]和[使用 IoT 中心发送云到设备的消息][lnk-c2d]）介绍了如何使用 IoT 中心的设备到云和云到设备的基本消息传递功能。

本教程以 [IoT 中心入门]中演示的代码为基础，呈现两种可用于处理设备到云消息的可缩放的模式：

- 在 [Azure Blob 存储]中可靠地存储设备到云的消息。常见的是 *冷路径* 分析，该情况下在 blob 中存储要输入到分析进程中的遥测数据。这些进程可由 [HDInsight (Hadoop)] 堆栈等工具驱动。

- 可靠地处理设备到云的*交互式*消息。如果设备到云的消息因为应用程序后端中的一组操作而立即触发，则表示这些消息是交互式的。例如，设备可能将发送一条警报消息，触发在 CRM 系统中插入票证。与此相反，*数据点*消息则仅送入分析引擎。例如，设备中存储以供将来分析使用的温度遥测是数据点消息。

IoT 中心公开[事件中心][lnk-event-hubs]兼容的终结点来接收设备到云的消息，因此本教程使用 [EventProcessorHost] 实例。此实例：

* 在 Azure blob 存储中可靠地存储*数据点*消息。
* 将设备到云的*交互式*消息转发到 Azure [服务总线队列]进行即时处理。

服务总线可以帮助确保可靠处理交互式消息，因为它提供了各消息的检查点，以及基于时间范围的重复数据删除。

> [AZURE.NOTE] **EventProcessorHost** 实例只是其中一种处理交互式消息的方法。其他选项包括 [Azure Service Fabric][lnk-service-fabric] 和 [Azure 流分析][lnk-stream-analytics]。

在本教程最后，会运行 3 个 Windows 控制台应用：

* **SimulatedDevice**（[IoT 中心入门]教程中创建的应用的修改版本）每秒发送一次设备到云的数据点消息，每 10 秒发送一次设备到云的交互式消息。此应用使用 AMQPS 协议来与 IoT 中心通信。
* **ProcessDeviceToCloudMessages** 使用 [EventProcessorHost] 类从与事件中心兼容的终结点检索消息。然后将数据点消息可靠地存储在 Azure blob 存储中，并将交互式消息转发到服务总线队列。
* **ProcessD2CInteractiveMessages** 从服务总线队列中剔除交互式消息。

> [AZURE.NOTE] IoT 中心对许多设备平台和语言（包括 C、Java 和 JavaScript）提供 SDK 支持。若要了解如何将本教程中的模拟设备替换为物理设备，以及如何将设备连接到 IoT 中心，请参阅 [Azure IoT 开发人员中心]。

本教程直接适用于使用事件中心兼容消息的其他方式，如 [HDInsight (Hadoop)] 项目。有关详细信息，请参阅 [Azure IoT 中心开发人员指南 - 设备到云]。

若要完成本教程，您需要以下各项：

+ Microsoft Visual Studio 2015。

+ 有效的 Azure 帐户。<br/>如果没有 Azure 订阅，只需几分钟即可创建一个[帐户](/pricing/1rmb-trial/)。

应具备 [Azure 存储]和 [Azure 服务总线]的一些基础知识。


## 从模拟设备发送交互式消息

在本部分中，会修改 [IoT 中心入门]教程中创建的模拟设备应用程序，向 IoT 中心发送设备到云的交互式消息。

1. 在 Visual Studio 的 **SimulatedDevice** 项目内，将以下方法添加到 **Program** 类。

    
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
    

    此方法类似于 **SimulatedDevice** 项目中的 **SendDeviceToCloudMessagesAsync** 方法。唯一的区别在于现在设置 **MessageId** 系统属性和 **messageType** 用户属性。
    代码将向 **MessageId** 属性分配全局唯一标识符 (GUID)。服务总线可使用此标识符来删除收到的重复消息。本示例使用 **messageType** 属性来区分交互式消息和数据点消息。应用程序将在消息属性而不是在消息正文中传递此信息，因此事件处理器不需要反序列化消息来执行消息路由。

    > [AZURE.NOTE] 有必要在设备代码中创建用于删除重复交互式消息的 **MessageId**。间歇性网络通信或其他故障可能会导致多次重复传输来自该设备的相同消息。你也可以使用语义消息 ID（例如相关消息数据字段的哈希）来取代 GUID。

2. 将以下方法添加到 **Main** 方法的 `Console.ReadLine()` 行的前面：

    `
	    SendDeviceToCloudInteractiveMessagesAsync();
    `

    > [AZURE.NOTE] 为简单起见，本教程不实现任何重试策略。在生产代码中，应按 MSDN 文章 [Transient Fault Handling]（暂时性故障处理）中所述实施指数退让等重试策略。

## 处理设备到云的消息

在本部分中，将创建一个 Windows 控制台应用，用于处理来自 IoT 中心的设备到云的消息。Iot 中心公开与[事件中心]兼容的终结点，让应用程序能读取设备到云的消息。本教程使用 [EventProcessorHost] 类来处理控制台应用中的这些消息。若要深入了解如何处理来自事件中心的消息，请参阅[事件中心入门]教程。

实现数据点消息的可靠存储或交互式消息的转发时，遇到的挑战是事件处理依赖消息使用者来提供进度的检查点。此外，为达到高吞吐量，应在从事件中心读取时大批量地提供检查点。如果失败且还原到先前的检查点，此方法可能重复处理大量消息。本教程将介绍如何将 Azure 存储空间写入及服务总线重复数据删除时间范围与 **EventProcessorHost** 检查点进行同步。

为了可靠地将消息写入 Azure 存储空间，本示例使用[块 blob][Azure Block Blobs] 的单个块提交功能。事件处理器将消息累积在内存中，直到应该提供检查点。例如，在消息的累积缓冲区达到 4 MB 的最大块大小之后，或者在超过服务总线重复数据删除时间范围之后。然后，在检查点之前，代码将新块提交到 Blob。

事件处理器使用事件中心消息偏移作为块 ID。借助此机制，事件处理器可在向存储空间提交新块之前执行重复数据删除检查，处理提交块和检查点之间可能发生的崩溃。

> [AZURE.NOTE] 本教程使用单个存储帐户来写入从 IoT 中心检索的所有消息。若要确定解决方案中是否需使用多个 Azure 存储帐户，请参阅 [Azure 存储可缩放性指导原则]。

应用程序利用服务总线重复数据删除功能，在处理交互式消息时避免重复项。模拟的设备向每个交互式消息标记唯一的 **MessageId**。借助这些 ID，服务总线可确保在指定的重复数据删除时间范围内，仅向接收方发送一条带相同 **MessageId** 的消息。此重复数据删除功能和服务总线队列所提供的每一消息完成语义，使其能够很容易地实现可靠的交互消息处理。

为确保未在重复数据删除时间范围外重新提交任何消息，代码会将 **EventProcessorHost** 检查点机制与服务总线队列重复数据删除时间范围进行同步。同步方式是在每次超出重复数据删除时间范围时（本教程中为 1 小时），至少强制执行一次检查点。

> [AZURE.NOTE] 本教程使用单个分区服务总线队列来处理所有检索自 IoT 中心的交互式消息。若要深入了解如何使用服务总线队列来满足解决方案的扩展性要求，请参阅 [Azure 服务总线]文档。

### 预配 Azure 存储帐户和服务总线队列
若要使用 [EventProcessorHost] 类，必须具有 Azure 存储帐户以使该类能记录检查点信息。可使用现有的存储帐户，或按照[关于 About Azure 存储]中的说明来创建新帐户。请记下存储帐户连接字符串。

> [AZURE.NOTE] 复制和粘贴存储帐户连接字符串时，请务必不要包含空格。

你还需要服务总线队列来可靠处理交互式消息。可在 1 小时的重复数据删除时间范围内，以编程方式创建队列，如[如何使用服务总线队列][Service Bus queue]中所述。还可按以下步骤使用 [Azure 经典管理门户][lnk-classic-portal]：

1. 单击左下角的“新建”。然后单击“应用程序服务”>“服务总线”>“队列”>“自定义创建”。输入名称 **d2ctutorial**，选择区域，再使用现有命名空间或新建一个。在下一页中，选择“启用重复检测”，再将“重复检测历史记录期限”设为 1 小时。然后单击右下角的复选标记保存你的队列配置。

    ![在 Azure 门户中创建队列][30]

2. 在服务总线队列列表中单击“d2ctutorial”，然后单击“配置”。创建分别名为 **send** 和 **listen** 的共享访问策略，前者带“发送”权限，后者带“侦听”权限。完成后，单击底部的“保存”。

    ![在 Azure 门户中配置队列][31]

3. 单击顶部的“仪表板”，再单击底部的“连接信息”。记下两个连接字符串。

    ![Azure 门户中的队列仪表板][32]

### 创建事件处理器

1. 在当前的 Visual Studio 解决方案中，若要使用“控制台应用程序”项目模板创建 Visual C# Windows 项目，请单击“文件”>“添加”>“新建项目”。确保 .NET Framework 版本为 4.5.1 或更高。将项目命名为 **ProcessDeviceToCloudMessages**，然后单击“确定”。

    ![Visual Studio 中的新项目][10]

2. 在解决方案资源管理器中，右键单击“ProcessDeviceToCloudMessages”项目，然后单击“管理 NuGet 包”。随即显示“NuGet 包管理器”对话框。

3. 搜索 **WindowsAzure.ServiceBus**，单击“安装”并接受使用条款。此操作会下载和安装 [Azure 服务总线 Nuget 包](https://www.nuget.org/packages/WindowsAzure.ServiceBus)，并添加对其的引用（包括所有依赖项）。

4. 搜索 **Microsoft.Azure.ServiceBus.EventProcessorHost**，单击“安装”并接受使用条款。此操作会下载和安装 [Azure 服务总线事件中心 - EventProcessorHost NuGet 包](https://www.nuget.org/packages/Microsoft.Azure.ServiceBus.EventProcessorHost)及其所有依赖项，并添加对它的引用。

5. 右键单击“ProcessDeviceToCloudMessages”项目，再依次单击“添加”和“类”。将新类命名为 **StoreEventProcessor**，然后单击“确定”以创建该类。

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
    

    **EventProcessorHost** 类调用此类以处理从 IoT 中心收到的设备到云消息。此类中的代码实现逻辑，以在 Blob 容器中可靠地存储消息，并将交互式消息转送到服务总线队列。

    **OpenAsync** 方法初始化 **currentBlockInitOffset** 变量，此变量可跟踪此事件处理器读取的第一个消息的当前偏移。请记住，每个处理器负责单个分区。

    **ProcessEventsAsync** 方法可从 IoT 中心接收一批消息，并按以下方式处理：发送交互式消息到服务总线队列，并将数据点消息附加到 **toAppend** 内存缓冲区。如果内存缓冲区达到 4 MB 的限制，或者超过重复数据删除时间范围（本教程中为检查点后的 1 小时），应用程序则会触发检查点。

    **AppendAndCheckpoint** 方法先为要附加的块生成 blockId。Azure 存储要求所有块 ID 的长度均相同，以便此方法用前置零填补偏移 - `currentBlockInitOffset.ToString("0000000000000000000000000")`。如果具有此 ID 的块已在 Blob 中，此方法将使用缓冲区的当前内容将它覆盖。

    > [AZURE.NOTE] 为了简化代码，本教程使用了每个分区的单个 Blob 文件来存储消息。实际上，会在某段时间后或在文件达到特定大小后创建其他文件，从而实现文件滚动。请记住，Azure 块 blob 最多可容纳 195 GB 的数据。

8. 在 **Program** 类的顶部添加以下 **using** 语句：

    
	    using Microsoft.ServiceBus.Messaging;
    

9. 如下所示修改 **Program** 类中的 **Main** 方法。将 **{iot hub connection string}** 替换为 [IoT 中心入门]教程中提供的 **iothubowner** 连接字符串。将存储连接字符串替换为在本部分开始时记录的连接字符串。将带“发送”权限的服务总线连接字符串替换为在本部分开始时记录的 **d2ctutorial** 队列：

    
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
    

    > [AZURE.NOTE] 为简单起见，本教程使用 [EventProcessorHost] 类的单个实例。有关详细信息，请参阅 [Event Hubs Programming Guide]（事件中心编程指南）。

## 接收交互式消息
在本部分，将编写一个 Windows 控制台应用，用于接收来自服务总线队列的交互式消息。若要深入了解如何使用服务总线构建解决方案，请参阅[使用服务总线构建多层应用程序][Build multi-tier applications with Service Bus]。

1. 在当前的 Visual Studio 解决方案中，使用“控制台应用程序”项目模板创建 Visual C# Windows 项目。将项目命名为 **ProcessD2CInteractiveMessages**。

2. 在解决方案资源管理器中，右键单击“ProcessD2CInteractiveMessages”项目，然后单击“管理 NuGet 包”。此时将显示“NuGet 包管理器”窗口。

3. 搜索 **WindowsAzure.ServiceBus**，单击“安装”并接受使用条款。此操作会下载和安装 [Azure 服务总线](https://www.nuget.org/packages/WindowsAzure.ServiceBus)，并添加对其的引用（包括所有依赖项）。

4. 在 **Program.cs** 文件的顶部添加以下 **using** 语句：

    
	    using System.IO;
	    using Microsoft.ServiceBus.Messaging;
    

5. 最后，在 **Main** 方法中添加以下行。将连接字符串替换为 **d2ctutorial** 队列的“侦听”权限：

    
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
    

## 运行应用程序

现在，你已准备就绪，可以运行应用程序了。

1.	在 Visual Studio 的解决方案资源管理器中，右键单击你的解决方案并选择“设置启动项目”。选择“多个启动项目”，然后选择“启动”以启动 **ProcessDeviceToCloudMessages**、**SimulatedDevice** 和 **ProcessD2CInteractiveMessages** 项目。

2.	按 **F5** 启动 3 个控制台应用程序。**ProcessD2CInteractiveMessages** 应用程序应处理 **SimulatedDevice** 应用程序发出的每条交互式消息。

    ![3 个控制台应用程序][50]  


> [AZURE.NOTE] 若要查看 blob 文件中的更新，需要将 **StoreEventProcessor** 类中的 **MAX\_BLOCK\_SIZE** 常量降为较小值，例如 **1024**。此更改很有用，原因是模拟设备发出的数据需要一些时间才能达到块大小限制。块大小更小时，可更快查看正创建和更新的 blob。但是，使用较大的块可以提高应用程序的可缩放性。

## 后续步骤

本教程介绍了如何使用 [EventProcessorHost] 类可靠地处理设备到云的数据点消息和交互式消息。

[如何使用 IoT 中心发送云到设备的消息][lnk-c2d]介绍如何从后端向设备发送消息。

若要查看使用 IoT 中心完成端到端解决方案的示例，请参阅 [Azure IoT 套件][lnk-suite]。

若要深入了解如何使用 IoT 中心开发解决方案，请参阅 [IoT 中心开发人员指南]。

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

[Azure IoT 中心开发人员指南 - 设备到云]: /documentation/articles/iot-hub-devguide-messaging/

[Azure 存储]: /documentation/services/storage/
[Azure 服务总线]: /documentation/services/service-bus/

[IoT 中心开发人员指南]: /documentation/articles/iot-hub-devguide/
[IoT 中心入门]: /documentation/articles/iot-hub-csharp-csharp-getstarted/
[Azure IoT 开发人员中心]: /develop/iot
[lnk-service-fabric]: /documentation/services/service-fabric/
[lnk-stream-analytics]: /documentation/services/stream-analytics/
[lnk-event-hubs]: /documentation/services/event-hubs/
[Transient Fault Handling]: https://msdn.microsoft.com/zh-cn/library/hh675232.aspx

<!-- Links -->

[关于 About Azure 存储]: /documentation/articles/storage-create-storage-account/#create-a-storage-account
[事件中心入门]: /documentation/articles/event-hubs-csharp-ephcs-getstarted/
[Azure 存储可缩放性指导原则]: /documentation/articles/storage-scalability-targets/
[Azure Block Blobs]: https://msdn.microsoft.com/zh-cn/library/azure/ee691964.aspx
[事件中心]: /documentation/articles/event-hubs-overview/
[EventProcessorHost]: http://msdn.microsoft.com/zh-cn/library/azure/microsoft.servicebus.messaging.eventprocessorhost(v=azure.95).aspx
[Event Hubs Programming Guide]: /documentation/articles/event-hubs-programming-guide/
[Transient Fault Handling]: https://msdn.microsoft.com/zh-cn/library/hh680901(v=pandp.50).aspx
[Build multi-tier applications with Service Bus]: /documentation/articles/service-bus-dotnet-multi-tier-app-using-service-bus-queues/

[lnk-classic-portal]: https://manage.windowsazure.cn
[lnk-c2d]: /documentation/articles/iot-hub-csharp-csharp-process-d2c/
[lnk-suite]: /documentation/services/iot-suite/

<!---HONumber=Mooncake_Quality_Review_1230_2016-->