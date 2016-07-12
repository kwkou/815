<properties 
	pageTitle="如何通过 WebJobs SDK 使用 Azure 队列存储" 
	description="了解如何通过 WebJobs SDK 使用 Azure 队列存储。创建和删除队列；插入、扫视、获取和删除队列消息，等等。" 
	services="app-service\web, storage" 
	documentationCenter=".net" 
	authors="tdykstra" 
	manager="wpickett" 
	editor="jimbe"/>

<tags
	ms.service="app-service-web"
	ms.date="02/29/2016"
	wacn.date="05/30/2016"/>

# 如何通过 WebJobs SDK 使用 Azure 队列存储

## 概述

本指南提供了 C# 代码示例，用于演示如何在 Azure 队列存储服务中使用 Azure WebJobs SDK 版本 1.x。

本指南假设你了解[如何使用指向存储帐户的连接字符串在 Visual Studio 中创建 WebJob 项目](/documentation/articles/websites-dotnet-webjobs-sdk-get-started/#configure-storage)或创建[多个存储帐户](https://github.com/Azure/azure-webjobs-sdk/blob/master/test/Microsoft.Azure.WebJobs.Host.EndToEndTests/MultipleStorageAccountsEndToEndTests.cs)。

大多数代码段只显示函数，不同于创建 `JobHost` 对象的代码，如以下示例所示：

		static void Main(string[] args)
		{
		    JobHost host = new JobHost();
		    host.RunAndBlock();
		}
		
本指南涉及以下主题：

-   [如何在接收队列消息时触发函数](#trigger)
	- 字符串队列消息
	- POCO 队列消息
	- 异步函数
	- QueueTrigger 属性适用的类型
	- 轮询算法
	- 多个实例
	- 并行执行
	- 获取队列或队列消息元数据
	- 正常关闭
-   [如何在处理队列消息时创建队列消息](#createqueue)
	- 字符串队列消息
	- POCO 队列消息
	- 在异步函数中创建多个消息
	- Queue 属性适用的类型
	- 在函数正文中使用 WebJobs SDK 属性
-   [如何在处理队列消息时读取和写入 Blob](#blobs)
	- 字符串队列消息
	- POCO 队列消息
	- Blob 属性适用的类型
-   [如何处理有害消息](#poison)
	- 自动处理有害消息
	- 手动处理有害消息
-   [如何设置配置选项](#config)
	- 在代码中设置 SDK 连接字符串
	- 配置 QueueTrigger 设置
	- 在代码中设置 WebJobs SDK 构造函数参数的值
-   [如何手动触发函数](#manual)
-   [如何写入日志](#logs) 
-   [如何处理错误和配置超时](#errors)
-   [后续步骤](#nextsteps)

## <a id="trigger"></a>如何在收到队列消息时触发函数

若要编写在收到队列消息时 WebJobs SDK 调用的函数，请使用 `QueueTrigger` 属性。该属性构造函数使用一个字符串参数来指定要轮询的队列名称。你也可以[动态设置队列名称](#config)。

### 字符串队列消息

在下面的示例中，队列中包含一个字符串消息，因此，`QueueTrigger` 已应用到包含队列消息内容的 `logMessage` 字符串参数。该函数[向仪表板写入一条日志消息](#logs)。
 

		public static void ProcessQueueMessage([QueueTrigger("logqueue")] string logMessage, TextWriter logger)
		{
		    logger.WriteLine(logMessage);
		}

除了 `string` 以外，参数还可以是字节数组、`CloudQueueMessage` 对象或您定义的 POCO。

### POCO[（普通旧 CLR 对象](http://en.wikipedia.org/wiki/Plain_Old_CLR_Object)）队列消息

在下面的示例中，队列消息包含 `BlobInformation` 对象的 JSON，该对象包含一个 `BlobName` 属性。SDK 会自动反序列化该对象。

		public static void WriteLogPOCO([QueueTrigger("logqueue")] BlobInformation blobInfo, TextWriter logger)
		{
		    logger.WriteLine("Queue message refers to blob: " + blobInfo.BlobName);
		}

SDK 使用 [Newtonsoft.Json NuGet 包](http://www.nuget.org/packages/Newtonsoft.Json)序列化和反序列化消息。如果你在不使用 WebJobs SDK 的程序中创建队列消息，则可以如以下示例所示编写代码，以创建 SDK 可以分析的 POCO 队列消息。

		BlobInformation blobInfo = new BlobInformation() { BlobName = "log.txt" };
		var queueMessage = new CloudQueueMessage(JsonConvert.SerializeObject(blobInfo));
		logQueue.AddMessage(queueMessage);

### 异步函数

以下异步函数[将日志写入仪表板](#logs)。

		public async static Task ProcessQueueMessageAsync([QueueTrigger("logqueue")] string logMessage, TextWriter logger)
		{
		    await logger.WriteLineAsync(logMessage);
		}

异步函数可以采用[取消标记](http://www.asp.net/mvc/overview/performance/using-asynchronous-methods-in-aspnet-mvc-4#CancelToken)，如以下用于复制 blob 的示例中所示。（有关 `queueTrigger` 占位符的说明，请参阅 [Blob](#blobs) 部分）。

		public async static Task ProcessQueueMessageAsyncCancellationToken(
		    [QueueTrigger("blobcopyqueue")] string blobName, 
		    [Blob("textblobs/{queueTrigger}",FileAccess.Read)] Stream blobInput,
		    [Blob("textblobs/{queueTrigger}-new",FileAccess.Write)] Stream blobOutput,
		    CancellationToken token)
		{
		    await blobInput.CopyToAsync(blobOutput, 4096, token);
		}

### <a id="qtattributetypes"></a>QueueTrigger 属性适用的类型

可以对 `QueueTrigger` 使用以下类型：

* `string`
* 序列化为 JSON 的 POCO 类型
* `byte[]`
* `CloudQueueMessage`

### <a id="polling"></a>轮询算法

SDK 实现了随机指数退让算法，以降低空闲队列轮询对存储事务成本造成的影响。当找到消息时，SDK 将等待两秒钟，然后检查另一条消息；如果未找到消息，它将等待大约四秒，然后重试。如果后续尝试获取队列消息失败，则等待时间会继续增加，直到达到最长等待时间（默认为 1 分钟）。[最长等待时间是可配置的](#config)。

### <a id="instances"></a>多个实例

如果 Web 应用在多个实例上运行，则每台计算机上都会运行一个连续 Web 作业，并且每台计算机将等待触发器并尝试运行函数。WebJobs SDK 队列触发器会自动阻止函数多次处理队列消息；函数无需编写为幂等函数。但是，如果你想要确保即使有多个主机 Web 应用的实例，也只有一个函数实例运行，可以使用 `Singleton` 属性。

### <a id="parallel"></a>并行执行

如果有多个函数在侦听不同的队列，SDK 将在同时接收消息时并行调用这些函数。

接收单个队列的多个消息时，也是如此。默认情况下，SDK 每次获取包含 16 个队列消息的批，然后并行执行处理这些消息的函数。[批大小是可配置的](#config)。当处理的数量达到批大小的一半时，SDK 将获取另一个批，并开始处理这些消息。因此，每个函数处理的最大并发消息数是批大小的 1.5 倍。此限制分别应用于各个包含 `QueueTrigger` 属性的函数。

如果不希望对队列上收到的消息并行执行，可以将批大小设置为 1。另请参阅 [Azure WebJobs SDK 1.1.0 RTM](http://azure.microsoft.com/zh-cn/blog/azure-webjobs-sdk-1-1-0-rtm/) 中的**更好地控制队列处理**。

### <a id="queuemetadata"></a>获取队列或队列消息元数据

你可以通过将参数添加到方法签名来获取以下消息属性：

* `DateTimeOffset` expirationTime
* `DateTimeOffset` insertionTime
* `DateTimeOffset` nextVisibleTime
* `string` queueTrigger（包含消息文本）
* `string` id
* `string` popReceipt
* `int` dequeueCount

如果您想直接使用 Azure 存储 API，则还可以添加 `CloudStorageAccount` 参数。

下面的示例将所有这些元数据写入 INFO 应用程序日志。在该示例中，logMessage 和 queueTrigger 包含队列消息的内容。

		public static void WriteLog([QueueTrigger("logqueue")] string logMessage,
		    DateTimeOffset expirationTime,
		    DateTimeOffset insertionTime,
		    DateTimeOffset nextVisibleTime,
		    string id,
		    string popReceipt,
		    int dequeueCount,
		    string queueTrigger,
		    CloudStorageAccount cloudStorageAccount,
		    TextWriter logger)
		{
		    logger.WriteLine(
		        "logMessage={0}\n" +
			"expirationTime={1}\ninsertionTime={2}\n" +
		        "nextVisibleTime={3}\n" +
		        "id={4}\npopReceipt={5}\ndequeueCount={6}\n" +
		        "queue endpoint={7} queueTrigger={8}",
		        logMessage, expirationTime,
		        insertionTime,
		        nextVisibleTime, id,
		        popReceipt, dequeueCount,
		        cloudStorageAccount.QueueEndpoint,
		        queueTrigger);
		}

下面是示例代码编写的示例日志：

		logMessage=Hello world!
		expirationTime=10/14/2014 10:31:04 PM +00:00
		insertionTime=10/7/2014 10:31:04 PM +00:00
		nextVisibleTime=10/7/2014 10:41:23 PM +00:00
		id=262e49cd-26d3-4303-ae88-33baf8796d91
		popReceipt=AgAAAAMAAAAAAAAAfc9H0n/izwE=
		dequeueCount=1
		queue endpoint=https://contosoads.queue.core.chinacloudapi.cn/
		queueTrigger=Hello world!

### <a id="graceful"></a>正常关闭

在连续 Web 作业中运行的函数可以接受 `CancellationToken` 参数，以便操作系统能够在 Web 作业即将终止时通知此函数。你可以使用此通知来确保该函数不会意外终止，导致数据处于不一致状态。

下面的示例演示了如何在函数中检查即将发生的 Web 作业终止。

	public static void GracefulShutdownDemo(
	            [QueueTrigger("inputqueue")] string inputText,
	            TextWriter logger,
	            CancellationToken token)
	{
	    for (int i = 0; i < 100; i++)
	    {
	        if (token.IsCancellationRequested)
	        {
	            logger.WriteLine("Function was cancelled at iteration {0}", i);
	            break;
	        }
	        Thread.Sleep(1000);
	        logger.WriteLine("Normal processing for queue message={0}", inputText);
	    }
	}

**注意**：仪表板可能会错误显示已关闭函数的的状态和输出。
 
有关详细信息，请参阅 [Web 作业正常关闭](http://blog.amitapple.com/post/2014/05/webjobs-graceful-shutdown/#.VCt1GXl0wpR)。

## <a id="createqueue"></a>如何在处理队列消息时创建队列消息

若要编写用于新建队列消息的函数，请使用 `Queue` 属性。与 `QueueTrigger` 一样，你可以传入字符串形式的队列名称，或者[动态设置队列名称](#config)。

### 字符串队列消息

下面的非异步代码示例在名为“outputqueue”的队列中创建新的队列消息，该消息的内容与名为“inputqueue”的队列中收到的队列消息相同。（对于异步函数，请按照本部分稍后将介绍的方法使用 `IAsyncCollector<T>`。）


		public static void CreateQueueMessage(
		    [QueueTrigger("inputqueue")] string queueMessage,
		    [Queue("outputqueue")] out string outputQueueMessage )
		{
		    outputQueueMessage = queueMessage;
		}
  
### POCO[（普通旧 CLR 对象](http://en.wikipedia.org/wiki/Plain_Old_CLR_Object)）队列消息

若要创建包含 POCO（而不是字符串）的队列消息，请将 POCO 类型作为输出参数传递给 `Queue` 属性构造函数。
 
		public static void CreateQueueMessage(
		    [QueueTrigger("inputqueue")] BlobInformation blobInfoInput,
		    [Queue("outputqueue")] out BlobInformation blobInfoOutput )
		{
		    blobInfoOutput = blobInfoInput;
		}

SDK 会自动将对象序列化为 JSON。即使对象为 null，也始终会创建队列消息。

### 在异步函数中创建多个消息

若要创建多个消息，请设置输出队列 `ICollector<T>` 或 `IAsyncCollector<T>` 的参数类型，如以下示例所示。

		public static void CreateQueueMessages(
		    [QueueTrigger("inputqueue")] string queueMessage,
		    [Queue("outputqueue")] ICollector<string> outputQueueMessage,
		    TextWriter logger)
		{
		    logger.WriteLine("Creating 2 messages in outputqueue");
		    outputQueueMessage.Add(queueMessage + "1");
		    outputQueueMessage.Add(queueMessage + "2");
		}

调用 `Add` 方法时，将立即创建每个队列消息。

### Queue 属性适用的类型

可对以下参数类型使用 `Queue` 属性：

* `out string`（函数结束时，如果参数值为非 null，则创建队列消息）
* `out byte[]`（工作方式类似于 `string`） 
* `out CloudQueueMessage`（工作方式类似于 `string`） 
* `out POCO`（一种可序列化类型，如果在函数结束时参数为 null，则创建包含 null 对象的消息）
* `ICollector`
* `IAsyncCollector`
* `CloudQueue`（用于直接通过 Azure 存储 API 手动创建消息）

### <a id="ibinder"></a>在函数主体中使用 WebJobs SDK 属性

如果您需要在使用 `Queue`、`Blob` 或 `Table` 等 WebJobs SDK 属性之前在函数中执行某项操作，则可以使用 `IBinder` 接口。

下面的示例采用一个输入队列消息，并在输出队列中创建具有相同内容的新消息。输出队列名称由函数正文中的代码设置。

		public static void CreateQueueMessage(
		    [QueueTrigger("inputqueue")] string queueMessage,
		    IBinder binder)
		{
		    string outputQueueName = "outputqueue" + DateTime.Now.Month.ToString();
		    QueueAttribute queueAttribute = new QueueAttribute(outputQueueName);
		    CloudQueue outputQueue = binder.Bind<CloudQueue>(queueAttribute);
		    outputQueue.AddMessage(new CloudQueueMessage(queueMessage));
		}

`IBinder` 接口也可以与 `Table` 和 `Blob` 属性一起使用。

## <a id="blobs"></a>如何在处理队列消息时读取和写入 blob 和表

可以使用 `Blob` 和 `Table` 属性来读取和写入 blob 和表。本部分中的示例适用于 Blob。有关展示如何在创建或更新 blob 时触发进程的代码示例，请参阅[如何结合使用 Azure blob 存储和 WebJobs SDK](/documentation/articles/websites-dotnet-webjobs-sdk-storage-blobs-how-to/)；有关用于读取和写入表的代码示例，请参阅[如何结合使用 Azure 表存储和 WebJobs SDK](/documentation/articles/websites-dotnet-webjobs-sdk-storage-tables-how-to/)。

### 触发 Blob 操作的字符串队列消息

对于包含字符串的队列消息，`queueTrigger` 是占位符，可以用于包含消息内容的 `Blob` 属性的 `blobPath` 参数。

下面的示例使用 `Stream` 对象读取和写入 blob。队列消息是位于 textBlobs 容器中的 Blob 名称。将在同一个容器中创建 Blob 的副本，并在其名称后面附加“-new”。

		public static void ProcessQueueMessage(
		    [QueueTrigger("blobcopyqueue")] string blobName, 
		    [Blob("textblobs/{queueTrigger}",FileAccess.Read)] Stream blobInput,
		    [Blob("textblobs/{queueTrigger}-new",FileAccess.Write)] Stream blobOutput)
		{
		    blobInput.CopyTo(blobOutput, 4096);
		}

`Blob` 属性构造函数采用指定容器和 blob 名称的 `blobPath` 参数。有关此占位符的详细信息，请参阅[如何结合使用 Azure blob 存储和 WebJobs SDK](/documentation/articles/websites-dotnet-webjobs-sdk-storage-blobs-how-to/)。

当属性修饰 `Stream` 对象时，另一个构造函数参数会将 `FileAccess` 模式指定为读取、写入或读取/写入。

下面的示例使用 `CloudBlockBlob` 对象删除 blob。队列消息是 Blob 的名称。

		public static void DeleteBlob(
		    [QueueTrigger("deleteblobqueue")] string blobName,
		    [Blob("textblobs/{queueTrigger}")] CloudBlockBlob blobToDelete)
		{
		    blobToDelete.Delete();
		}

### <a id="pocoblobs"></a>POCO[（普通旧 CLR 对象](http://en.wikipedia.org/wiki/Plain_Old_CLR_Object)）队列消息

对于在队列消息中存储为 JSON 的 POCO，您可以使用占位符来命名 `Queue` 属性的 `blobPath` 参数中的对象属性。还可以将[队列元数据属性名称](#queuemetadata)用作占位符。

下面的示例将 Blob 复制到具有不同扩展名的新 Blob。队列消息是包含 `BlobName` 和 `BlobNameWithoutExtension` 属性的 `BlobInformation` 对象。属性名称用作 `Blob` 属性的 blob 路径中的占位符。
 
		public static void CopyBlobPOCO(
		    [QueueTrigger("copyblobqueue")] BlobInformation blobInfo,
		    [Blob("textblobs/{BlobName}", FileAccess.Read)] Stream blobInput,
		    [Blob("textblobs/{BlobNameWithoutExtension}.txt", FileAccess.Write)] Stream blobOutput)
		{
		    blobInput.CopyTo(blobOutput, 4096);
		}

SDK 使用 [Newtonsoft.Json NuGet 包](http://www.nuget.org/packages/Newtonsoft.Json)序列化和反序列化消息。如果你在不使用 WebJobs SDK 的程序中创建队列消息，则可以如以下示例所示编写代码，以创建 SDK 可以分析的 POCO 队列消息。

		BlobInformation blobInfo = new BlobInformation() { BlobName = "boot.log", BlobNameWithoutExtension = "boot" };
		var queueMessage = new CloudQueueMessage(JsonConvert.SerializeObject(blobInfo));
		logQueue.AddMessage(queueMessage);

如果在将 blob 绑定到对象之前，您需要在函数中执行某项操作，则可以在函数主体中使用属性，[如前面的 Queue 属性所示](#ibinder)。

### <a id="blobattributetypes"></a>Blob 属性适用的类型
 
`Blob` 属性适用以下类型：

* `Stream`（读取或写入，通过使用 FileAccess 构造函数参数指定）
* `TextReader`
* `TextWriter`
* `string`（读取）
* `out string`（写入；仅当字符串参数在函数返回结果时为非 null 的情况下，才创建 blob）
* POCO（读取）
* out POCO（写入；始终创建 Blob，如果函数返回时 POCO 参数为 null，则创建 null 对象）
* `CloudBlobStream`（写入）
* `ICloudBlob`（读取或写入）
* `CloudBlockBlob`（读取或写入） 
* `CloudPageBlob`（读取或写入） 

## <a id="poison"></a>如何处理病毒消息

内容导致函数失败的消息称为*病毒消息*。当函数失败时，将不删除并最终再次选择队列消息，从而导致周期重复。在达到限制的迭代次数后，SDK 可自动中断周期，你也可以手动中断。

### 自动处理有害消息

SDK 在处理一个队列消息时最多会调用某个函数 5 次。如果第五次尝试失败，消息将移到有害队列。[最大重试次数可配置](#config)。

病毒队列的名称为 *{originalqueuename}*-poison。你可以编写一个函数来处理有害队列中的消息，并记录这些消息，或者发送需要注意的通知。

在下面的示例中，如果队列消息包含不存在的 blob 名称，则 `CopyBlob` 函数会失败。在这种情况，消息将从 copyBlobqueue 队列移到 copyBlobqueue-poison 队列。然后，`ProcessPoisonMessage` 将记录有害消息。

		public static void CopyBlob(
		    [QueueTrigger("copyblobqueue")] string blobName,
		    [Blob("textblobs/{queueTrigger}", FileAccess.Read)] Stream blobInput,
		    [Blob("textblobs/{queueTrigger}-new", FileAccess.Write)] Stream blobOutput)
		{
		    blobInput.CopyTo(blobOutput, 4096);
		}
		
		public static void ProcessPoisonMessage(
		    [QueueTrigger("copyblobqueue-poison")] string blobName, TextWriter logger)
		{
		    logger.WriteLine("Failed to copy blob, name=" + blobName);
		}

下图显示了处理有害消息时这些函数的控制台输出。

![处理病毒消息时的控制台输出](./media/websites-dotnet-webjobs-sdk-storage-queues-how-to/poison.png)

### 手动处理有害消息

您可以向您的函数添加名为 `dequeueCount` 的 `int` 参数，获取选择处理某消息的次数。然后，你可以检查函数代码中的取消排队计数，并在处理次数超过阈值时执行自己的有害消息处理，如以下示例中所示。

		public static void CopyBlob(
		    [QueueTrigger("copyblobqueue")] string blobName, int dequeueCount,
		    [Blob("textblobs/{queueTrigger}", FileAccess.Read)] Stream blobInput,
		    [Blob("textblobs/{queueTrigger}-new", FileAccess.Write)] Stream blobOutput,
		    TextWriter logger)
		{
		    if (dequeueCount > 3)
		    {
		        logger.WriteLine("Failed to copy blob, name=" + blobName);
		    }
		    else
		    {
		    blobInput.CopyTo(blobOutput, 4096);
		    }
		}

## <a id="config"></a>如何设置配置选项

您可以使用 `JobHostConfiguration` 类型设置以下配置选项：

* 在代码中设置 SDK 连接字符串。
* 配置 `QueueTrigger` 设置，例如最大取消排队计数。
* 从配置中获取队列名称。

### <a id="setconnstr"></a>在代码中设置 SDK 连接字符串

在代码中设置 SDK 连接字符串可以在配置文件或环境变量中使用自己的连接字符串名称，如以下示例中所示。

		static void Main(string[] args)
		{
		    var _storageConn = ConfigurationManager
		        .ConnectionStrings["MyStorageConnection"].ConnectionString;
		
		    var _dashboardConn = ConfigurationManager
		        .ConnectionStrings["MyDashboardConnection"].ConnectionString;
		
		    var _serviceBusConn = ConfigurationManager
		        .ConnectionStrings["MyServiceBusConnection"].ConnectionString;
		
		    JobHostConfiguration config = new JobHostConfiguration();
		    config.StorageConnectionString = _storageConn;
		    config.DashboardConnectionString = _dashboardConn;
		    config.ServiceBusConnectionString = _serviceBusConn;
		    JobHost host = new JobHost(config);
		    host.RunAndBlock();
		}

### <a id="configqueue"></a>配置 QueueTrigger 设置

你可以配置以下用于处理队列消息的设置：

- 同时选择的、要并行执行的最大队列消息数（默认值为 16）。
- 在将队列消息发送到有害队列之前要重试的最大次数（默认值为 5）。
- 当队列为空时，再次轮询之前要等待的最长时间（默认值为 1 分钟）。

下面的示例演示如何配置这些设置：

		static void Main(string[] args)
		{
		    JobHostConfiguration config = new JobHostConfiguration();
		    config.Queues.BatchSize = 8;
		    config.Queues.MaxDequeueCount = 4;
		    config.Queues.MaxPollingInterval = TimeSpan.FromSeconds(15);
		    JobHost host = new JobHost(config);
		    host.RunAndBlock();
		}

### <a id="setnamesincode"></a>在代码中设置 WebJobs SDK 构造函数参数的值

有时，你想要在代码中指定队列名称、Blob 名称、容器或表名称，而不是进行硬编码。例如，您可能想在配置文件或环境变量中指定 `QueueTrigger` 的队列名称。

为此，您可以向 `JobHostConfiguration` 类型传递 `NameResolver` 对象。此时，你可以在 WebJobs SDK 属性构造函数参数中包含以百分号 (%) 括住的特殊占位符，你的 `NameResolver` 代码将指定要用于取代这些占位符的实际值。

例如，假设你要在测试环境中使用名为 logqueuetest 的队列，并在生产环境中使用名为 logqueueprod 的队列。您希望在具有实际队列名称的 `appSettings` 集合中指定条目名称，而不是硬编码的队列名称。如果 `appSettings` 键为 logqueue，则函数如以下示例所示。

		public static void WriteLog([QueueTrigger("%logqueue%")] string logMessage)
		{
		    Console.WriteLine(logMessage);
		}

然后，`NameResolver` 类可以从 `appSettings` 获取队列名称，如以下示例所示：

		public class QueueNameResolver : INameResolver
		{
		    public string Resolve(string name)
		    {
		        return ConfigurationManager.AppSettings[name].ToString();
		    }
		}

将 `NameResolver` 类传入 `JobHost` 对象，如以下示例中所示。

		static void Main(string[] args)
		{
		    JobHostConfiguration config = new JobHostConfiguration();
		    config.NameResolver = new QueueNameResolver();
		    JobHost host = new JobHost(config);
		    host.RunAndBlock();
		}
 
**注意：**每次调用函数，都会解析队列名称、表名称和 blob 名称，但 blob 容器名称只会在应用程序启动时进行解析。在作业运行时，您无法更改 blob 容器名称。

## <a id="manual"></a>如何手动触发函数

若要手动触发函数，请对 `JobHost` 对象使用 `Call` 或 `CallAsync` 方法，并对函数使用 `NoAutomaticTrigger` 属性，如以下示例所示。

		public class Program
		{
		    static void Main(string[] args)
		    {
		        JobHost host = new JobHost();
		        host.Call(typeof(Program).GetMethod("CreateQueueMessage"), new { value = "Hello world!" });
		    }
		
		    [NoAutomaticTrigger]
		    public static void CreateQueueMessage(
		        TextWriter logger, 
		        string value, 
		        [Queue("outputqueue")] out string message)
		    {
		        message = value;
		        logger.WriteLine("Creating queue message: ", message);
		    }
		}

## <a id="logs"></a>如何写入日志

仪表板在两个位置显示日志：针对 Web 作业的页，以及针对特定 Web 作业调用的页。

![WebJob 页中的日志](./media/websites-dotnet-webjobs-sdk-storage-queues-how-to/dashboardapplogs.png)

![函数调用页中的日志](./media/websites-dotnet-webjobs-sdk-storage-queues-how-to/dashboardlogs.png)

在函数或 `Main()` 方法中调用的控制台方法的输出在 Web 作业的仪表板页面上显示，而不是在特定方法调用页面上显示。从方法签名的参数中获取的 TextWriter 对象的输出在方法调用的仪表板页中显示。

无法将控制台输出链接到特定的方法调用，因为控制台是单线程的，而许多作业函数可能同时运行。正因如此，SDK 为每个函数调用提供了自身唯一的日志写入器对象。

若要写入应用程序跟踪日志，请使用 `Console.Out`（创建标记为 INFO 的日志）和 `Console.Error`（创建标记为 ERROR 的日志）。或者，您可以使用 [Trace 或 TraceSource](http://blogs.msdn.com/b/mcsuksoldev/archive/2014/09/04/adding-trace-to-azure-web-sites-and-web-jobs.aspx)，它除了提供“信息”和“错误”外，还提供“详细”、“警告”和“严重级别”。应用程序跟踪日志将显示在 Web 应用日志文件、Azure 表或 Azure Blob 中，具体取决于你如何配置 Azure Web 应用。与所有控制台输出一样，最近的 100 条应用程序日志也会显示在 Web 作业的仪表板页中，而不是显示在函数调用的页中。

仅当程序在 Azure Web 作业中运行（而不是在本地运行或者在其他某个环境中运行）时，控制台输出才显示在仪表板中。

为高吞吐量方案禁用仪表板日志记录。默认情况下，SDK 将日志写入存储，此活动会在处理的消息较多时降低性能。若要禁用日志记录，请按以下示例中所示，将仪表板连接字符串设置为 null。

		JobHostConfiguration config = new JobHostConfiguration();       
		config.DashboardConnectionString = "";        
		JobHost host = new JobHost(config);
		host.RunAndBlock();

下面的示例演示了写入日志的多种方法：

		public static void WriteLog(
		    [QueueTrigger("logqueue")] string logMessage,
		    TextWriter logger)
		{
		    Console.WriteLine("Console.Write - " + logMessage);
		    Console.Out.WriteLine("Console.Out - " + logMessage);
		    Console.Error.WriteLine("Console.Error - " + logMessage);
		    logger.WriteLine("TextWriter - " + logMessage);
		}

在 WebJobs SDK 仪表板中，当您转到特定函数调用页面并单击“切换输出”时，您会看到 `TextWriter` 对象的输出：

![单击函数调用链接](./media/websites-dotnet-webjobs-sdk-storage-queues-how-to/dashboardinvocations.png)

![函数调用页中的日志](./media/websites-dotnet-webjobs-sdk-storage-queues-how-to/dashboardlogs.png)

在 WebJobs SDK 仪表板中，当您转到 Web 作业（而不是函数调用）页面并单击“切换输出”时，您会看到最近的 100 行控制台输出。
 
![单击“切换输出”](./media/websites-dotnet-webjobs-sdk-storage-queues-how-to/dashboardapplogs.png)

在连续 Web 作业中，应用程序日志显示在 Web 应用文件系统的 /data/jobs/continuous/*{webjobname}*/job\_log.txt 中。

		[09/26/2014 21:01:13 > 491e54: INFO] Console.Write - Hello world!
		[09/26/2014 21:01:13 > 491e54: ERR ] Console.Error - Hello world!
		[09/26/2014 21:01:13 > 491e54: INFO] Console.Out - Hello world!

在 Azure blob 中，应用程序日志如下所示：
		2014-09-26T21:01:13,Information,contosoadsnew,491e54,635473620738373502,0,17404,17,Console.Write - Hello world!,
		2014-09-26T21:01:13,Error,contosoadsnew,491e54,635473620738373502,0,17404,19,Console.Error - Hello world!,
		2014-09-26T21:01:13,Information,contosoadsnew,491e54,635473620738529920,0,17404,17,Console.Out - Hello world!,

在 Azure 表中，`Console.Out` 和 `Console.Error` 日志如下所示：

![表中的信息日志](./media/websites-dotnet-webjobs-sdk-storage-queues-how-to/tableinfo.png)

![表中的错误日志](./media/websites-dotnet-webjobs-sdk-storage-queues-how-to/tableerror.png)

如果要插入自己的记录器，请参阅[此示例](http://github.com/Azure/azure-webjobs-sdk-samples/blob/master/BasicSamples/MiscOperations/Program.cs)。

## <a id="errors"></a>如何处理错误和配置超时

WebJobs SDK 还包括 [Timeout](http://github.com/Azure/azure-webjobs-sdk-samples/blob/master/BasicSamples/MiscOperations/Functions.cs) 属性，可用于在函数未在指定的时间内完成时取消函数。而且，如果你想要在指定的时间段内发生太多错误时引发警报，可以使用 `ErrorTrigger` 属性。下面是 [ErrorTrigger 示例](https://github.com/Azure/azure-webjobs-sdk-extensions/wiki/Error-Monitoring)。

	
	public static void ErrorMonitor(
	[ErrorTrigger("00:01:00", 1)] TraceFilter filter, TextWriter log,
	[SendGrid(
	    To = "admin@emailaddress.com",
	    Subject = "Error!")]
	 SendGridMessage message)
	{
	    // log last 5 detailed errors to the Dashboard
	   log.WriteLine(filter.GetDetailedMessage(5));
	   message.Text = filter.GetDetailedMessage(1);
	}
	

你还可以使用配置开关（可以是应用设置或环境变量名称）动态地禁用和启用函数以控制是否可以触发它们。有关示例代码，请参阅 [WebJobs SDK 示例存储库](https://github.com/Azure/azure-webjobs-sdk-samples/blob/master/BasicSamples/MiscOperations/Functions.cs)中的 `Disable` 属性。

## <a id="nextsteps"></a>后续步骤

本指南提供的代码示例演示了如何处理常见方案以操作 Azure 队列。若要详细了解如何使用 Azure WebJobs 和 WebJobs SDK，请参阅[有关 Azure WebJobs 的推荐资源](/documentation/articles/websites-webjobs-resources/)。
 

<!---HONumber=Mooncake_0118_2016-->