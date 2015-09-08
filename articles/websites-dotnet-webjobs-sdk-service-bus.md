<properties 
	pageTitle="如何结合使用 Azure 服务总线和 WebJobs SDK" 
	description="了解如何结合使用 Azure 服务总线队列、主题和 WebJobs SDK。" 
	services="app-service\web, service-bus" 
	documentationCenter=".net" 
	authors="tdykstra" 
	manager="wpickett" 
	editor="jimbe"/>

<tags 
	ms.service="app-service-web" 
	ms.date="06/29/2015" 
	wacn.date="08/29/2015"/>

# 如何结合使用 Azure 服务总线和 WebJobs SDK

## 概述

本指南包含 C# 代码示例，展示了如何在创建或更新 Azure blob 时触发进程。这些代码示例使用 [WebJobs SDK](/documentation/articles/websites-dotnet-webjobs-sdk) 版本 1.x。

若要查看本指南，您需要知道[如何在 Visual Studio 中创建 Web 作业项目，其中包含指向存储帐户的连接字符串](/documentation/articles/websites-dotnet-webjobs-sdk-get-started)。

代码段只显示函数，不同于创建 `JobHost` 对象的代码（如以下示例所示）：

		static void Main(string[] args)
		{
		    JobHost host = new JobHost();
		    host.RunAndBlock();
		}
		
## 目录

-   [先决条件](#prerequisites)
-   [如何在收到队列消息时触发函数](#trigger)
-   [如何创建队列消息](#create)
-   [如何处理服务总线主题](#topics)
-   [存储队列文章的涉及相关主题](#queues)
-   [后续步骤](#nextsteps)

## <a id="prerequisites"></a>先决条件

您必须先安装 [Microsoft.Azure.WebJobs.ServiceBus](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.ServiceBus/) NuGet 包和其他 WebJobs SDK 包，然后才能使用服务总线。

您还必须设置 AzureWebJobsServiceBus 连接字符串和存储连接字符串。您可以在 Web.config 文件的 `connectionStrings` 部分中执行此操作，如以下示例所示：

		<connectionStrings>
		    <add name="AzureWebJobsDashboard" connectionString="DefaultEndpointsProtocol=https;AccountName=[accountname];AccountKey=[accesskey]"/>
		    <add name="AzureWebJobsStorage" connectionString="DefaultEndpointsProtocol=https;AccountName=[accountname];AccountKey=[accesskey]"/>
		    <add name="AzureWebJobsServiceBus" connectionString="Endpoint=sb://[yourServiceNamespace].servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=[yourKey]"/>
		</connectionStrings>

有关示例项目，请参阅[服务总线示例](https://github.com/Azure/azure-webjobs-sdk-samples/tree/master/BasicSamples/ServiceBus)。有关详细信息，请参阅 [WebJobs SDK 使用入门](/documentation/articles/websites-dotnet-webjobs-sdk-get-started)。

## <a id="trigger"></a>如何在收到服务总线队列消息时触发函数

若要编写在收到队列消息时 WebJobs SDK 调用的函数，请使用 `ServiceBusTrigger` 属性。此属性构造函数捕获指定了要轮询的队列名称的参数。

### ServicebusTrigger 工作原理

SDK 接收 `PeekLock` 模式的消息。如果函数成功完成，则对此消息调用 `Complete`；如果函数失败，则调用 `Abandon`。如果函数的运行时间长于 `PeekLock` 超时时间，则会自动续订锁定。

由于服务总线会自行执行病毒队列处理，因此不需要受控于 WebJobs SDK 或在其中进行配置。

### 字符串队列消息

以下代码示例读取包含字符串的队列消息，并将字符串写入 WebJobs SDK 仪表板。

		public static void ProcessQueueMessage([ServiceBusTrigger("inputqueue")] string message, 
		    TextWriter logger)
		{
		    logger.WriteLine(message);
		}

**注意：**如果您在未使用 WebJobs SDK 的应用程序中创建队列消息，请务必将 [BrokeredMessage.ContentType](https://msdn.microsoft.com/zh-cn/library/microsoft.servicebus.messaging.brokeredmessage.contenttype.aspx) 设置为 “text/plain”。

### POCO 队列消息

SDK 会自动反序列化包含 POCO[（普通旧 CLR 对象](http://en.wikipedia.org/wiki/Plain_Old_CLR_Object)）类型 JSON 的队列消息。以下代码示例读取包含 `BlobInformation` 对象（具有 `BlobName` 属性）的队列消息：

		public static void WriteLogPOCO([ServiceBusTrigger("inputqueue")] BlobInformation blobInfo,
		    TextWriter logger)
		{
		    logger.WriteLine("Queue message refers to blob: " + blobInfo.BlobName);
		}

有关展示如何使用 POCO 属性在同一函数中处理 blob 和表的代码示例，请参阅[这篇文章的存储队列版本](/documentation/articles/websites-dotnet-webjobs-sdk-storage-queues-how-to#pocoblobs)。

### ServiceBusTrigger 适用的类型

除了 `string` 和 POCO 类型以外，您还可以使用具有字节数组或 `BrokeredMessage` 对象的 `ServiceBusTrigger` 属性。

## <a id="create"></a>如何创建服务总线队列消息

若要编写用于新建队列消息的函数，请使用 `ServiceBus` 属性，并将队列名称传递给属性构造函数。


### 使用非异步函数创建一个队列消息

以下代码示例使用输出参数在“outputqueue”队列中新建消息，此消息的内容与“inputqueue”队列中收到的消息相同。

		public static void CreateQueueMessage(
		    [ServiceBusTrigger("inputqueue")] string queueMessage,
		    [ServiceBus("outputqueue")] out string outputQueueMessage)
		{
		    outputQueueMessage = queueMessage;
		}

用于创建一个队列消息的输出参数可以是以下任意类型：

* `string`
* `byte[]`
* `BrokeredMessage`
* 您定义的可序列化 POCO 类型。自动序列化为 JSON。

对于 POCO 类型参数，当函数结束时，始终会创建队列消息；如果参数为 null，则 SDK 会创建在收到和反序列化消息时返回 null 的队列消息。对于其他类型，如果参数为 null，则不会创建任何队列消息。

### 使用异步函数创建多个队列消息

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

调用 `Add` 方法后，便可立即创建每个队列消息。

## <a id="topics"></a>如何处理服务总线主题

若要编写 SDK 在收到服务总线主题消息时调用的函数，请使用 `ServiceBusTrigger` 属性以及捕获主题名称和订阅名称的构造函数，如以下代码示例所示：

		public static void WriteLog([ServiceBusTrigger("outputtopic","subscription1")] string message,
		    TextWriter logger)
		{
		    logger.WriteLine("Topic message: " + message);
		}

若要创建某主题的消息，请使用 `ServiceBus` 属性和主题名称，过程与使用此属性和队列名称一样。

## <a id="queues"></a>存储队列操作说明文章涉及的相关主题

若要了解非服务总线专用 WebJobs SDK 方案，请参阅[如何结合使用 Azure 队列存储和 WebJobs SDK](/documentation/articles/websites-dotnet-webjobs-sdk-storage-queues-how-to)。

这篇文章涉及的主题包括：

* 异步函数
* 多个实例
* 正常关闭
* 在函数主体中使用 WebJobs SDK 属性
* 在代码中设置 SDK 连接字符串
* 在代码中设置 WebJobs SDK 构造函数参数的值
* 手动触发函数
* 写入日志

## <a id="nextsteps"></a>后续步骤

本指南中包含的代码示例展示了如何处理常见方案来结合使用 Azure 服务总线。若要详细了解如何使用 Azure WebJobs 和 WebJobs SDK，请参阅[有关 Azure WebJobs 的推荐资源](/documentation/articles/websites-webjobs-resources/)。
 

<!---HONumber=67-->