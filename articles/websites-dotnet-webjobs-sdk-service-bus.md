<properties 
	pageTitle="如何通过 WebJobs SDK 使用 Azure Service Bus" 
	description="了解如何通过 WebJobs SDK 使用 Azure Service Bus 队列和主题。" 
	services="app-service\web, service-bus" 
	documentationCenter=".net" 
	authors="tdykstra" 
	manager="wpickett" 
	editor="jimbe"/>

<tags
	ms.service="app-service-web"
	ms.date="02/29/2016"
	wacn.date="04/18/2016"/>

# 如何通过 WebJobs SDK 使用 Azure Service Bus

## 概述

本指南提供 C# 代码示例，用于演示如何在创建或更新 Azure Blob 后触发进程。这些代码示例使用 [WebJobs SDK](/documentation/articles/websites-dotnet-webjobs-sdk/) 版本 1.x。

本指南假设您了解[如何使用指向存储帐户的连接字符串在 Visual Studio 中创建 WebJob 项目](/documentation/articles/websites-dotnet-webjobs-sdk-get-started/)。

代码段只显示函数，不同于创建 `JobHost` 对象的代码（如以下示例所示）：

	public class Program
	{
   		public static void Main()
   		{
      			JobHostConfiguration config = new JobHostConfiguration();
      			config.UseServiceBus();
      			JobHost host = new JobHost(config);
      			host.RunAndBlock();
   		}
	}

在 GitHub.com 上的 azure-webjobs-sdk-samples 存储库中有[完整的服务总线代码示例](https://github.com/Azure/azure-webjobs-sdk-samples/blob/master/BasicSamples/ServiceBus/Program.cs)。

## <a id="prerequisites"></a>先决条件

你必须先安装 [Microsoft.Azure.WebJobs.ServiceBus](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.ServiceBus/) NuGet 包和其他 WebJobs SDK 包，然后才能使用服务总线。

你还必须设置 AzureWebJobsServiceBus 连接字符串，以及存储连接字符串。你可以在 App.config 文件的 `connectionStrings` 部分中执行此操作，如以下示例所示：

		<connectionStrings>
		    <add name="AzureWebJobsDashboard" connectionString="DefaultEndpointsProtocol=https;AccountName=[accountname];AccountKey=[accesskey]"/>
		    <add name="AzureWebJobsStorage" connectionString="DefaultEndpointsProtocol=https;AccountName=[accountname];AccountKey=[accesskey]"/>
		    <add name="AzureWebJobsServiceBus" connectionString="Endpoint=sb://[yourServiceNamespace].servicebus.chinacloudapi.cn/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=[yourKey]"/>
		</connectionStrings>

有关在 App.config 文件中包含服务总线连接字符串设置的示例项目，请参阅[服务总线示例](https://github.com/Azure/azure-webjobs-sdk-samples/tree/master/BasicSamples/ServiceBus)。

也可以在 Azure 运行时环境中设置连接字符串，当 Web 作业在 Azure 中运行时，这些设置将覆盖 App.config 设置；有关详细信息，请参阅 [WebJobs SDK 入门](/documentation/articles/websites-dotnet-webjobs-sdk-get-started/#configure-the-web-app-to-use-your-azure-sql-database-and-storage-account)。

## <a id="trigger"></a>如何在接收服务总线队列消息时触发函数

若要编写接收队列消息时 WebJobs SDK 调用的函数，请使用 `ServiceBusTrigger` 属性。该属性构造函数使用一个参数来指定要轮询的队列名称。

### ServicebusTrigger 工作原理

SDK 接收 `PeekLock` 模式的消息。如果函数成功完成，则对此消息调用 `Complete`；如果函数失败，则调用 `Abandon`。如果函数的运行时间长于 `PeekLock` 超时时间，则会自动续订锁定。

服务总线会自行执行有害队列处理，因此不需要由 WebJobs SDK 控制或配置。

### 字符串队列消息

以下代码示例读取包含字符串的队列消息，并将字符串写入 WebJobs SDK 仪表板。

		public static void ProcessQueueMessage([ServiceBusTrigger("inputqueue")] string message, 
		    TextWriter logger)
		{
		    logger.WriteLine(message);
		}

**注意：**如果你在未使用 WebJobs SDK 的应用程序中创建队列消息，请务必将 [BrokeredMessage.ContentType](http://msdn.microsoft.com/zh-cn/library/microsoft.servicebus.messaging.brokeredmessage.contenttype.aspx) 设置为 “text/plain”。

### POCO 队列消息

SDK 会自动反序列化包含 POCO[（普通旧 CLR 对象](http://en.wikipedia.org/wiki/Plain_Old_CLR_Object)）类型 JSON 的队列消息。以下代码示例读取包含 `BlobInformation` 对象（具有 `BlobName` 属性）的队列消息：

		public static void WriteLogPOCO([ServiceBusTrigger("inputqueue")] BlobInformation blobInfo,
		    TextWriter logger)
		{
		    logger.WriteLine("Queue message refers to blob: " + blobInfo.BlobName);
		}

有关展示如何使用 POCO 属性在同一函数中处理 blob 和表的代码示例，请参阅[这篇文章的存储队列版本](/documentation/articles/websites-dotnet-webjobs-sdk-storage-queues-how-to/#pocoblobs)。

如果创建队列消息的代码不使用 WebJobs SDK，请使用类似于以下示例的代码：

		var client = QueueClient.CreateFromConnectionString(ConfigurationManager.ConnectionStrings["AzureWebJobsServiceBus"].ConnectionString, "blobadded");
		BlobInformation blobInformation = new BlobInformation () ;
		var message = new BrokeredMessage(blobInformation);
		client.Send(message);

### ServiceBusTrigger 适用的类型

除了 `string` 和 POCO 类型以外，你还可以使用具有字节数组或 `BrokeredMessage` 对象的 `ServiceBusTrigger` 属性。

## <a id="create"></a>如何创建服务总线队列消息

若要编写用于新建队列消息的函数，请使用 `ServiceBus` 属性，并将队列名称传递给属性构造函数。


### 在非异步函数中创建单个队列消息

以下代码示例使用输出参数在名为“outputqueue”的队列中创建新的消息，该消息的内容与名为“inputqueue”的队列中收到的队列消息相同。

		public static void CreateQueueMessage(
		    [ServiceBusTrigger("inputqueue")] string queueMessage,
		    [ServiceBus("outputqueue")] out string outputQueueMessage)
		{
		    outputQueueMessage = queueMessage;
		}

用于创建单个队列消息的输出参数可以是以下任何类型：

* `string`
* `byte[]`
* `BrokeredMessage`
* 你定义的可序列化 POCO 类型。自动序列化为 JSON。

对于 POCO 类型参数，当函数结束时始终创建队列消息；如果参数为 null，则 SDK 将创建在接收和反序列化消息时返回 null 的队列消息。对于其他类型，如果该参数为 null，则不创建队列消息。

### 在异步函数中创建多个队列消息

若要创建多个消息，请使用包含 `ICollector<T>` 或 `IAsyncCollector<T>` 的 `ServiceBus` 属性，如以下代码示例所示：

		public static void CreateQueueMessages(
		    [ServiceBusTrigger("inputqueue")] string queueMessage,
		    [ServiceBus("outputqueue")] ICollector<string> outputQueueMessage,
		    TextWriter logger)
		{
		    logger.WriteLine("Creating 2 messages in outputqueue");
		    outputQueueMessage.Add(queueMessage + "1");
		    outputQueueMessage.Add(queueMessage + "2");
		}

调用 `Add` 方法时，将立即创建每个队列消息。

## <a id="topics"></a>如何处理服务总线主题

若要编写 SDK 在收到服务总线主题消息时调用的函数，请使用 `ServiceBusTrigger` 属性以及捕获主题名称和订阅名称的构造函数，如以下代码示例所示：

		public static void WriteLog([ServiceBusTrigger("outputtopic","subscription1")] string message,
		    TextWriter logger)
		{
		    logger.WriteLine("Topic message: " + message);
		}

若要创建某主题的消息，请使用 `ServiceBus` 属性和主题名称，过程与使用此属性和队列名称一样。

## 1\.1 版中的新增功能

在 1.1 版中添加了以下功能：

* 允许通过 `ServiceBusConfiguration.MessagingProvider` 对消息处理进行深层自定义。
* `MessagingProvider` 支持服务总线 `MessagingFactory` 和 `NamespaceManager` 的自定义。
* `MessageProcessor` 策略模式允许你为每个队列/主题指定处理器。
* 默认情况下支持消息处理并发。 
* 可以轻松通过 `ServiceBusConfiguration.MessageOptions` 对 `OnMessageOptions` 进行自定义。
* 允许在 `ServiceBusTriggerAttribute`/`ServiceBusAttribute` 上指定 [AccessRights](https://github.com/Azure/azure-webjobs-sdk-samples/blob/master/BasicSamples/ServiceBus/Functions.cs#L71)（适用于你可能不具有管理权限的情况）。 

## <a id="queues"></a>存储队列操作说明文章涉及的相关主题

若要了解非服务总线专用 WebJobs SDK 方案，请参阅[如何结合使用 Azure 队列存储和 WebJobs SDK](/documentation/articles/websites-dotnet-webjobs-sdk-storage-queues-how-to/)。

该文章涵盖的主题包括：

* 异步函数
* 多个实例
* 正常关闭
* 在函数正文中使用 WebJobs SDK 属性
* 在代码中设置 SDK 连接字符串。
* 在代码中设置 WebJobs SDK 构造函数参数的值
* 手动触发函数
* 写入日志

## <a id="nextsteps"></a>后续步骤

本指南中包含的代码示例展示了如何处理常见方案来结合使用 Azure 服务总线。有关如何使用 Azure WebJobs 和 WebJobs SDK 的详细信息，请参阅 [Azure WebJobs 推荐资源](/documentation/articles/websites-webjobs-resources/)。
 

<!---HONumber=Mooncake_0411_2016-->