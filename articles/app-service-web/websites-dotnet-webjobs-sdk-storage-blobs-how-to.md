<properties 
	pageTitle="如何通过 WebJobs SDK 使用 Azure Blob 存储" 
	description="了解如何通过 WebJobs SDK 使用 Azure Blob 存储。在新 Blob 出现在容器中时触发进程并处理“有害 Blob”。" 
	services="app-service\web, storage" 
	documentationCenter=".net" 
	authors="tdykstra" 
	manager="wpickett" 
	editor="jimbe"/>

<tags 
	ms.service="app-service-web" 
	ms.workload="web" 
	ms.tgt_pltfrm="na" 
	ms.devlang="dotnet" 
	ms.topic="article" 
	ms.date="06/01/2016" 
	wacn.date="12/16/2016" 
	ms.author="tdykstra"/>

# 如何通过 WebJobs SDK 使用 Azure Blob 存储

[AZURE.INCLUDE [azure-sdk-developer-differences](../../includes/azure-sdk-developer-differences.md)]

## 概述

本指南提供 C# 代码示例，用于演示如何在创建或更新 Azure Blob 后触发进程。这些代码示例使用 [WebJobs SDK](/documentation/articles/websites-dotnet-webjobs-sdk/) 版本 1.x。

有关演示如何创建 Blob 的代码示例，请参阅[如何通过 WebJobs SDK 使用 Azure 队列存储](/documentation/articles/websites-dotnet-webjobs-sdk-storage-queues-how-to/)。
		
本指南假设你了解[如何使用指向存储帐户的连接字符串在 Visual Studio 中创建 WebJob 项目](/documentation/articles/websites-dotnet-webjobs-sdk-get-started/)或创建[多个存储帐户](https://github.com/Azure/azure-webjobs-sdk/blob/master/test/Microsoft.Azure.WebJobs.Host.EndToEndTests/MultipleStorageAccountsEndToEndTests.cs)。

## <a id="trigger"></a> 如何在创建或更新 Blob 后触发函数

本部分说明如何使用 `BlobTrigger` 属性。

> [AZURE.NOTE] WebJobs SDK 会扫描日志文件，以观察新的或更改的 Blob。此过程非常缓慢；创建 Blob 之后数分钟或更长时间内可能仍不会触发函数。如果你的应用程序需要立即处理 Blob，推荐的方法是在创建该 Blob 时创建队列消息，并在处理该 Blob 的函数上使用 [QueueTrigger](/documentation/articles/websites-dotnet-webjobs-sdk-storage-queues-how-to/#trigger) 属性（而非 `BlobTrigger` 属性）。

### Blob 名称和扩展名的单个占位符  

以下代码示例将 *input* 容器中显示的文本 Blob 复制到 *output* 容器中：

		public static void CopyBlob([BlobTrigger("input/{name}")] TextReader input,
		    [Blob("output/{name}")] out string output)
		{
		    output = input.ReadToEnd();
		}

属性构造函数采用指定容器名称的字符串参数和 Blob 名称的占位符。在此示例中，如果在 *input* 容器中创建了名为 *Blob1.txt* 的 Blob，则该函数将在 *output* 容器中创建名为 *Blob1.txt* 的 Blob。

你可以指定包含 Blob 名称占位符的名称模式，如以下代码示例中所示：

		public static void CopyBlob([BlobTrigger("input/original-{name}")] TextReader input,
		    [Blob("output/copy-{name}")] out string output)
		{
		    output = input.ReadToEnd();
		}

此代码只会复制名称以“original-”开头的 Blob。例如，将 *input* 容器中的 *original-Blob1.txt* 复制到 *output* 容器中的 *copy-Blob1.txt*。

如果你需要为名称中包含大括号的 Blob 名称指定名称模式，则使用双倍的大括号。例如，如果你想要在 *images* 容器中查找具有以下类似名称的 Blob：

		{20140101}-soundfile.mp3

为模式使用以下代码：

		images/{{20140101}}-{name}

在示例中，*name* 占位符值将为 *soundfile.mp3*。

### 单独的 Blob 名称和扩展名占位符

以下代码示例在将 *input* 容器中显示的 Blob 复制到 *output* 容器中时更改文件扩展名。该代码将记录 *input* Blob 的扩展名，并将 *output* Blob 的扩展名设置为 *.txt*。

		public static void CopyBlobToTxtFile([BlobTrigger("input/{name}.{ext}")] TextReader input,
		    [Blob("output/{name}.txt")] out string output,
		    string name,
		    string ext,
		    TextWriter logger)
		{
		    logger.WriteLine("Blob name:" + name);
		    logger.WriteLine("Blob extension:" + ext);
		    output = input.ReadToEnd();
		}

## <a id="types"></a> 可绑定到 Blob 的类型

可对以下类型使用 `BlobTrigger` 属性：

* `string`
* `TextReader`
* `Stream`
* `ICloudBlob`
* `CloudBlockBlob`
* `CloudPageBlob`
* `CloudBlobContainer`
* `CloudBlobDirectory`
* `IEnumerable<CloudBlockBlob>`
* `IEnumerable<CloudPageBlob>`
* [ICloudBlobStreamBinder](#icbsb) 反序列化的其他类型 

如果你想要直接使用 Azure 存储帐户，则还可以向方法签名添加 `CloudStorageAccount` 参数。

有关示例，请参阅 [GitHub.com 上 azure-webjobs-sdk 存储库中的 Blob 绑定代码](https://github.com/Azure/azure-webjobs-sdk/blob/master/test/Microsoft.Azure.WebJobs.Host.EndToEndTests/BlobBindingEndToEndTests.cs)。

## <a id="string"></a> 通过绑定到字符串获取文本 Blob 内容

如果需要文本 Blob，可将 `BlobTrigger` 应用到 `string` 参数。以下代码示例将文本 Blob 绑定到名为 `logMessage` 的 `string` 参数。函数使用该参数将 Blob 的内容写入 WebJobs SDK 仪表板。
 
		public static void WriteLog([BlobTrigger("input/{name}")] string logMessage,
		    string name, 
		    TextWriter logger)
		{
		     logger.WriteLine("Blob name: {0}", name);
		     logger.WriteLine("Content:");
		     logger.WriteLine(logMessage);
		}

## <a id="icbsb"></a> 使用 ICloudBlobStreamBinder 获取序列化 Blob 内容

以下代码示例使用实现 `ICloudBlobStreamBinder` 的类来启用 `BlobTrigger` 属性，将 Blob 绑定到 `WebImage` 类型。

		public static void WaterMark(
		    [BlobTrigger("images3/{name}")] WebImage input,
		    [Blob("images3-watermarked/{name}")] out WebImage output)
		{
		    output = input.AddTextWatermark("WebJobs SDK", 
		        horizontalAlign: "Center", verticalAlign: "Middle",
		        fontSize: 48, opacity: 50);
		}
		public static void Resize(
		    [BlobTrigger("images3-watermarked/{name}")] WebImage input,
		    [Blob("images3-resized/{name}")] out WebImage output)
		{
		    var width = 180;
		    var height = Convert.ToInt32(input.Height * 180 / input.Width);
		    output = input.Resize(width, height);
		}

派生自 `ICloudBlobStreamBinder` 的 `WebImageBinder` 类中提供 `WebImage` 绑定代码。

		public class WebImageBinder : ICloudBlobStreamBinder<WebImage>
		{
		    public Task<WebImage> ReadFromStreamAsync(Stream input, 
		        System.Threading.CancellationToken cancellationToken)
		    {
		        return Task.FromResult<WebImage>(new WebImage(input));
		    }
		    public Task WriteToStreamAsync(WebImage value, Stream output,
		        System.Threading.CancellationToken cancellationToken)
		    {
		        var bytes = value.GetBytes();
		        return output.WriteAsync(bytes, 0, bytes.Length, cancellationToken);
		    }
		}

## <a id="poison"></a> 如何处理有害 Blob

当 `BlobTrigger` 函数失败时，如果失败是暂时性错误导致的，则 SDK 会再次调用该函数。如果失败是由 Blob 的内容导致的，则该函数每次尝试处理 Blob 时都会失败。默认情况下，对于给定的 Blob，SDK 调用一个函数最多 5 次。如果第五次尝试失败，SDK 会将消息添加到名为 *webjobs-blobtrigger-poison* 的队列中。

最大尝试次数可配置。将使用相同的 [MaxDequeueCount](/documentation/articles/websites-dotnet-webjobs-sdk-storage-queues-how-to/#configqueue) 设置处理有害 Blob 和有害队列消息。

有害 Blob 的队列消息是包含以下属性的 JSON 对象：

* FunctionId（格式为 *{WebJob name}*.Functions.*{Function name}*，例如 WebJob1.Functions.CopyBlob）
* BlobType（"BlockBlob" 或 "PageBlob"）
* ContainerName
* BlobName
* ETag（Blob 版本标识符，例如："0x8D1DC6E70A277EF"）

在下面的代码示例中，`CopyBlob` 函数的代码导致它每次调用时都失败。SDK 进行最大重试次数的调用之后，有害 Blob 队列中会创建消息，该消息通过 `LogPoisonBlob` 函数进行处理。

		public static void CopyBlob([BlobTrigger("input/{name}")] TextReader input,
		    [Blob("textblobs/output-{name}")] out string output)
		{
		    throw new Exception("Exception for testing poison blob handling");
		    output = input.ReadToEnd();
		}
		
		public static void LogPoisonBlob(
		[QueueTrigger("webjobs-blobtrigger-poison")] PoisonBlobMessage message,
		    TextWriter logger)
		{
		    logger.WriteLine("FunctionId: {0}", message.FunctionId);
		    logger.WriteLine("BlobType: {0}", message.BlobType);
		    logger.WriteLine("ContainerName: {0}", message.ContainerName);
		    logger.WriteLine("BlobName: {0}", message.BlobName);
		    logger.WriteLine("ETag: {0}", message.ETag);
		}

SDK 自动反序列化 JSON 消息。下面是 `PoisonBlobMessage` 类：

		public class PoisonBlobMessage
		{
		    public string FunctionId { get; set; }
		    public string BlobType { get; set; }
		    public string ContainerName { get; set; }
		    public string BlobName { get; set; }
		    public string ETag { get; set; }
		}

### <a id="polling"></a> Blob 轮询算法

启动应用程序时，WebJobs SDK 将扫描 `BlobTrigger` 属性指定的所有容器。在大型存储帐户中，此扫描可能需要一些时间，因此在查找新 Blob 和执行 `BlobTrigger` 函数之前，可能需要一段时间。

若要在应用程序启动后检测新的或已更改的 Blob，SDK 会定期从 Blob 存储日志读取。Blob 日志将进行缓冲，仅每隔 10 分钟左右进行物理写入，因此创建或更新 Blob 后可能存在很长的延迟，然后才会执行对应的 `BlobTrigger` 函数。

使用 `Blob` 属性创建的 Blob 出现异常。当 WebJobs SDK 创建新 Blob 时，会立即将新的 Blob 传递给任何匹配的 `BlobTrigger` 函数。因此，如果建立了 Blob 输入和输出的链接，则 SDK 可以高效地处理它们。但是，如果你想要对通过其他方式创建或更新的 Blob 降低运行 Blob 处理函数的延迟时间，我们建议使用 `QueueTrigger`（而不是 `BlobTrigger`）。

### <a id="receipts"></a> Blob 回执

WebJobs SDK 确保没有为相同的新 Blob 或更新 Blob 多次调用 `BlobTrigger` 函数。为此，它会维护 *Blob 回执*，以确定是否已处理给定的 Blob 版本。

Blob 回执存储在 AzureWebJobsStorage 连接字符串指定的 Azure 存储帐户中名为 *azure-webjobs-hosts* 的容器内。Blob 回执包含以下信息：

* 为 Blob 调用的函数（"*{WebJob name}*.Functions.*{Function name}*"，例如 "WebJob1.Functions.CopyBlob"）
* 容器名称
* Blob 类型（"BlockBlob" 或 "PageBlob"）
* Blob 名称
* ETag（Blob 版本标识符，例如："0x8D1DC6E70A277EF"）

如果你想要强制重新处理某个 Blob，则可以从 *azure-webjobs-hosts* 容器中手动删除该 Blob 的 Blob 回执。

## <a id="queues"></a> 队列文章涵盖的相关主题

有关如何处理队列消息触发的 Blob 处理，或者不特定于 Blob 处理的 WebJobs SDK 方案的信息，请参阅[如何通过 WebJobs SDK 使用 Azure 队列存储](/documentation/articles/websites-dotnet-webjobs-sdk-storage-queues-how-to/)。

该文章涵盖的相关主题包括：

* 异步函数
* 多个实例
* 正常关闭
* 在函数正文中使用 WebJobs SDK 属性
* 在代码中设置 SDK 连接字符串。
* 在代码中设置 WebJobs SDK 构造函数参数的值
* 为有害 Blob 处理配置 `MaxDequeueCount`。
* 手动触发函数
* 写入日志

## <a id="nextsteps"></a>后续步骤

本指南提供的代码示例演示了如何处理常见方案以操作 Azure Blob。有关如何使用 Azure WebJobs 和 WebJobs SDK 的详细信息，请参阅 [Azure WebJobs 推荐资源](/documentation/articles/websites-webjobs-resources/)。
 

<!---HONumber=Mooncake_Quality_Review_1202_2016-->