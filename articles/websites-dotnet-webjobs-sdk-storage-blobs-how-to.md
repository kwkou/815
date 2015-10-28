<properties 
	pageTitle="如何结合使用 Azure blob 存储和 WebJobs SDK" 
	description="了解如何结合使用 Azure blob 存储和 WebJobs SDK。在新的 blob 出现在容器中时触发进程，并处理“病毒 blob”。" 
	services="app-service\web, storage" 
	documentationCenter=".net" 
	authors="tdykstra" 
	manager="wpickett" 
	editor="jimbe"/>

<tags 
	ms.service="app-service-web" 
	ms.date="06/08/2015" 
	wacn.date="08/29/2015"/>

# 如何结合使用 Azure blob 存储和 WebJobs SDK

## 概述

本指南包含 C# 代码示例，展示了如何在创建或更新 Azure blob 时触发进程。这些代码示例使用 [WebJobs SDK](/documentation/articles/websites-dotnet-webjobs-sdk) 版本 1.x。

有关展示如何创建 blob 的代码示例，请参阅[如何结合使用 Azure 队列存储和 WebJobs SDK](/documentation/articles/websites-dotnet-webjobs-sdk-storage-queues-how-to)。
		
若要查看本指南，您需要知道[如何在 Visual Studio 中创建 Web 作业项目，其中包含指向存储帐户的连接字符串](/documentation/articles/websites-dotnet-webjobs-sdk-get-started)。

## 目录

-   [如何在创建或更新 blob 时触发函数](#trigger)
	- blob 名称和扩展名的单个占位符
	- 单独的 blob 名称和扩展名占位符
-   [BlobTrigger 适用的类型](#types)
-   [通过绑定到字符串获取文本 blob 内容](#string)
-   [使用 ICloudBlobStreamBinder 获取序列化 blob 内容](#icbsb)
-   [如何处理病毒 blob](#poison)
-   [Blob 轮询算法](#polling)
-   [Blob 回执](#receipts)
-   [队列文章涉及的相关主题](#queues)
-   [后续步骤](#nextsteps)

## <a id="trigger"></a>如何在创建或更新 blob 时触发函数

此部分介绍了如何使用 `BlobTrigger` 属性。

> **注意：**WebJobs SDK 会扫描日志文件，监控新的或更改后的 blob。此过程非常缓慢；在创建 blob 后，需要经过数分钟或更长时间才可能会触发函数。如果您的应用程序需要立即处理 blob，推荐的方法是在创建 blob 时创建一个队列消息，并对处理 blob 的函数使用 [QueueTrigger](/documentation/articles/websites-dotnet-webjobs-sdk-storage-queues-how-to#trigger) 属性（而非 `BlobTrigger` 属性）。

### blob 名称和扩展名的单个占位符  

以下代码示例将*输入*容器中显示的文本 blob 复制到*输出*容器中：

		public static void CopyBlob([BlobTrigger("input/{name}")] TextReader input,
		    [Blob("output/{name}")] out string output)
		{
		    output = input.ReadToEnd();
		}

属性构造函数捕获指定了容器名称的字符串参数和 blob 名称占位符。在此示例中，如果在*输入*容器中创建了名为 *Blob1.txt* 的 blob，则函数会在*输出*容器中创建名为 *Blob1.txt* 的 blob。

您可以指定包含 blob 名称占位符的名称模式，如以下代码示例所示：

		public static void CopyBlob([BlobTrigger("input/original-{name}")] TextReader input,
		    [Blob("output/copy-{name}")] out string output)
		{
		    output = input.ReadToEnd();
		}

此代码只会复制名称以“original-”开头的 blob。例如，将*输入*容器中的 *original-Blob1.txt* 复制为*输出*容器中的 *copy-Blob1.txt*。

如果您需要为名称中包含大括号的 blob 名称指定名称模式，请使用双重大括号。例如，如果您想在*映像*容器中查找具有以下类似名称的 blob：

		{20140101}-soundfile.mp3

请对您的模式使用以下代码：

		images/{{20140101}}-{name}

在此示例中，*名称*占位符值将为 *soundfile.mp3*。

### 单独的 blob 名称和扩展名占位符

以下代码示例在将*输入*容器中显示的 blob 复制到*输出*容器中时更改文件扩展名。此代码记录*输入* blob 的扩展名，并将*输出* blob 的扩展名设置为 *.txt*。

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

## <a id="types"></a>可绑定到 blob 的类型

可以对以下类型使用 `BlobTrigger` 属性：

* `string`
* `TextReader`
* `Stream`
* `ICloudBlob`
* `CloudBlockBlob`
* `CloudPageBlob`
* [ICloudBlobStreamBinder](#icbsb) 反序列化的其他类型 

如果您想直接使用 Azure 存储帐户，则还可以向方法签名添加 `CloudStorageAccount` 参数。

## <a id="string"></a>通过绑定到字符串获取文本 blob 内容

如果需要的是文本 blob，则可将 `BlobTrigger` 应用到 `string` 参数。以下代码示例将文本 blob 绑定到名为 `logMessage` 的 `string` 参数。函数使用相应的参数将 blob 内容写入 WebJobs SDK 仪表板。
 
		public static void WriteLog([BlobTrigger("input/{name}")] string logMessage,
		    string name, 
		    TextWriter logger)
		{
		     logger.WriteLine("Blob name: {0}", name);
		     logger.WriteLine("Content:");
		     logger.WriteLine(logMessage);
		}

## <a id="icbsb"></a>使用 ICloudBlobStreamBinder 获取序列化 blob 内容

以下代码示例使用实施 `ICloudBlobStreamBinder` 的类，让 `BlobTrigger` 属性将 blob 绑定到 `WebImage` 类型。

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

`WebImage` 绑定代码由派生自 `ICloudBlobStreamBinder` 的 `WebImageBinder` 类提供。

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

## <a id="poison"></a>如何处理病毒 blob

当 `BlobTrigger` 函数失败时，如果失败是由暂时性错误所导致，则 SDK 会再次调用此函数。如果失败是由 blob 内容所导致，则此函数每次尝试处理 blob 时都会失败。默认情况下，对于给定的 blob，SDK 最多调用一个函数 5 次。如果第 5 次尝试失败，则 SDK 会将消息添加到 *webjobs-blobtrigger-poison* 队列中。

重试次数上限是可配置的。处理病毒 blob 和病毒队列消息所使用的是同一个 [MaxDequeueCount](/documentation/articles/websites-dotnet-webjobs-sdk-storage-queues-how-to#configqueue) 设置。

病毒 blob 的队列消息是包含以下属性的 JSON 对象：

* FunctionId（格式为 *{WebJob name}*.Functions.*{Function name}*，例如：WebJob1.Functions.CopyBlob）
* BlobType（“BlockBlob”或“PageBlob”）
* ContainerName
* BlobName
* ETag（blob 版本标识符，例如：“0x8D1DC6E70A277EF”）

在下面的代码示例中，`CopyBlob` 函数的代码导致它每次被调用时都失败。在 SDK 调用它的次数达到重试次数上限后，病毒 blob 队列中会创建一条消息，并由 `LogPoisonBlob` 函数处理此消息。

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

SDK 会自动反序列化 JSON 消息。下面是 `PoisonBlobMessage` 类：

		public class PoisonBlobMessage
		{
		    public string FunctionId { get; set; }
		    public string BlobType { get; set; }
		    public string ContainerName { get; set; }
		    public string BlobName { get; set; }
		    public string ETag { get; set; }
		}

### <a id="polling"></a>Blob 轮询算法

在应用程序启动时，WebJobs SDK 会扫描 `BlobTrigger` 属性指定的所有容器。在大型存储帐户中，此扫描可能需要一段时间才能完成，因此可能需要过一会儿才能找到新的 blob 和执行 `BlobTrigger` 函数。

若要在应用程序启动后检测新的或更改后的 blob，则 SDK 会定期读取 blob 存储日志。blob 日志会进行缓冲，仅每隔 10 分钟左右获取一次物理写入，因此创建或更新 blob 后可能会经历很长的一段延迟，然后才会执行相应的 `BlobTrigger` 函数。

对于您使用 `Blob` 属性创建的 blob，存在例外情况。新建 blob 时，WebJobs SDK 会立即将新的 blob 传递给所有匹配的 `BlobTrigger` 函数。因此，如果您有一系列 blob 输入和输出，则 SDK 可以高效地处理它们。不过，如果您希望通过其他方式创建或更新的 blob 的处理进程函数具有低延迟，我们建议您使用 `QueueTrigger`（而不是 `BlobTrigger`）。

### <a id="receipts"></a>Blob 回执

对于同一新的或更新后的 blob，WebJobs SDK 可确保不会多次调用 `BlobTrigger` 函数。为此，它会维护 *blob 回执*，以确定是否已处理过给定的 blob 版本。

Blob 回执存储在 *azure-webjobs-hosts* 容器内，此容器位于 AzureWebJobsStorage 连接字符串指定的 Azure 存储帐户中。blob 回执包含以下信息：

* 为 blob 调用的函数（“*{WebJob name}*.Functions.*{Function name}*”，例如：“WebJob1.Functions.CopyBlob”）
* 容器名称
* blob 类型（“BlockBlob”或“PageBlob”）
* blob 名称
* ETag（blob 版本标识符，例如：“0x8D1DC6E70A277EF”）

如果您想强行重新处理某个 blob，则可以从 *azure-webjobs-hosts* 容器中手动删除此 blob 的 blob 回执。

## <a id="queues"></a>队列文章涉及的相关主题

若要了解如何处理队列消息触发的 blob 处理进程，或了解非 blob 处理进程 WebJobs SDK 方案，请参阅[如何结合使用 Azure 队列存储和 WebJobs SDK](/documentation/articles/websites-dotnet-webjobs-sdk-storage-queues-how-to)。

这篇文章涉及的相关主题包括：

* 异步函数
* 多个实例
* 正常关闭
* 在函数主体中使用 WebJobs SDK 属性
* 在代码中设置 SDK 连接字符串。
* 在代码中设置 WebJobs SDK 构造函数参数的值
* 配置 `MaxDequeueCount` 处理病毒 blob。
* 手动触发函数
* 写入日志

## <a id="nextsteps"></a>后续步骤

本指南中包含的代码示例展示了如何处理常见方案来结合使用 Azure blob。若要详细了解如何使用 Azure WebJobs 和 WebJobs SDK，请参阅[有关 Azure WebJobs 的推荐资源](/documentation/articles/websites-webjobs-resources)。
 

<!---HONumber=67-->