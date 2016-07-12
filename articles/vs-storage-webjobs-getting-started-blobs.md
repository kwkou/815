<properties
	pageTitle="Blob 存储和 Visual Studio 连接服务（Web 作业项目）入门 | Azure"
	description="在使用 Visual Studio 连接服务连接到 Azure 存储后，如何开始使用 WebJob 项目中的 Blob 存储"
	services="storage"
	documentationCenter=""
	authors="TomArcher"
	manager="douge"
	editor=""/>

<tags
	ms.service="storage"
	ms.date="05/08/2016"
	wacn.date="06/13/2016"/>

# 开始使用 Azure Blob 存储和 Visual Studio 连接服务（WebJob 项目）

## 概述

本文章提供 C# 代码示例，用于演示如何在创建或更新 Azure blob 后触发进程。这些代码示例使用 [WebJobs SDK](/documentation/articles/websites-dotnet-webjobs-sdk/) 版本 1.x。当你使用 Visual Studio“添加连接服务”对话框将存储帐户添加到 WebJob 项目中时，会安装相应的 Azure 存储 NuGet 包，相应的.NET 引用会添加到项目中，并会在 App.config 文件中更新存储帐户的连接字符串。



## 如何在创建或更新 Blob 后触发函数

本部分说明如何使用 **BlobTrigger** 属性。

 **注意：**WebJobs SDK 会扫描日志文件，以观察新的或更改的 blob。此过程非常缓慢；创建 blob 之后数分钟或更长时间内可能仍不会触发函数。如果你的应用程序需要立即处理 blob，推荐的方法是在创建该 blob 时创建队列消息，并在处理该 blob 的函数上使用 [QueueTrigger](/documentation/articles/websites-dotnet-webjobs-sdk-storage-queues-how-to/#trigger) 属性（而非 **BlobTrigger** 属性）。

### Blob 名称和扩展名的单个占位符  

以下代码示例将输入容器中显示的文本 blob 复制到输出容器中：

		public static void CopyBlob([BlobTrigger("input/{name}")] TextReader input,
		    [Blob("output/{name}")] out string output)
		{
		    output = input.ReadToEnd();
		}

属性构造函数采用指定容器名称的字符串参数和 Blob 名称的占位符。在此示例中，如果在输入容器中创建了名为 *Blob1.txt* 的 blob，则该函数将在输出容器中创建名为 *Blob1.txt* 的 blob。

你可以指定包含 Blob 名称占位符的名称模式，如以下代码示例中所示：

		public static void CopyBlob([BlobTrigger("input/original-{name}")] TextReader input,
		    [Blob("output/copy-{name}")] out string output)
		{
		    output = input.ReadToEnd();
		}

此代码只会复制名称以“original-”开头的 Blob。例如，将输入容器中的 *original-Blob1.txt* 复制到输出容器中的 *copy-Blob1.txt*。

如果你需要指定的名称中包含大括号的 Blob 名称的名称模式，增加大括号。例如，如果你想要在映像容器中查找具有以下类似名称的 blob：

		{20140101}-soundfile.mp3

为模式使用以下代码：

		images/{{20140101}}-{name}

在示例中，名称占位符值将为 *soundfile.mp3*。

### 单独的 Blob 名称和扩展名占位符

以下代码示例在将输入容器中显示的 blob 复制到输出容器中时更改文件扩展名。该代码将记录输入 blob 的扩展名，并将输出 blob 的扩展名设置为 *.txt*。

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

## <a id="types"></a> 可绑定到 blob 的类型

可对以下类型使用 **BlobTrigger** 属性：

* **string**
* **TextReader**
* **Stream**
* **ICloudBlob**
* **CloudBlockBlob**
* **CloudPageBlob**
* [ICloudBlobStreamBinder](#getting-serialized-blob-content-by-using-icloudblobstreambinder) 反序列化的其他类型

如果你想要直接使用 Azure 存储帐户，则还可以向方法签名添加 **CloudStorageAccount** 参数。

## <a id="string"></a> 通过绑定到字符串获取文本 blob 内容

如果需要文本 blob，可将 **BlobTrigger** 应用到 **string** 参数。以下代码示例将文本 blob 绑定到名为 **logMessage** 的 **string** 参数。函数使用该参数将 Blob 的内容写入 WebJobs SDK 仪表板。

		public static void WriteLog([BlobTrigger("input/{name}")] string logMessage,
		    string name,
		    TextWriter logger)
		{
		     logger.WriteLine("Blob name: {0}", name);
		     logger.WriteLine("Content:");
		     logger.WriteLine(logMessage);
		}

## <a id="getting-serialized-blob-content-by-using-icloudblobstreambinder"></a> 使用 ICloudBlobStreamBinder 获取序列化 blob 内容

以下代码示例使用实现 **ICloudBlobStreamBinder** 的类来启用 **BlobTrigger** 属性，将 blob 绑定到 **WebImage** 类型。

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

**WebImage** 绑定代码在 **WebImageBinder** 类中提供，该类派生自 **ICloudBlobStreamBinder**。

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

## <a id="poison"></a> 如何处理有害 blob

当 **BlobTrigger** 函数失败时，如果失败是暂时性错误导致的，则 SDK 会再次调用该函数。如果失败是由 Blob 的内容导致的，则该函数每次尝试处理 Blob 时都会失败。默认情况下，对于给定的 Blob，SDK 调用一个函数最多 5 次。如果第五次尝试失败，SDK 会将消息添加到名为 *webjobs-blobtrigger-poison* 的队列中。

最大尝试次数可配置。将使用相同的 [MaxDequeueCount](/documentation/articles/websites-dotnet-webjobs-sdk-storage-queues-how-to/#configqueue) 设置处理有害 blob 和有害队列消息。

有害 Blob 的队列消息是包含以下属性的 JSON 对象：

* FunctionId（格式为 *{WebJob name}*.Functions.*{Function name}*，例如 WebJob1.Functions.CopyBlob）
* BlobType（"BlockBlob" 或 "PageBlob"）
* ContainerName
* BlobName
* ETag（blob 版本标识符，例如："0x8D1DC6E70A277EF"）

在下面的代码示例中，**CopyBlob** 函数的代码导致它每次调用时都失败。SDK 进行最大重试次数的调用之后，有害 blob 队列中会创建消息，该消息通过 **LogPoisonBlob** 函数进行处理。

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

SDK 自动反序列化 JSON 消息。下面是 **PoisonBlobMessage** 类：

		public class PoisonBlobMessage
		{
		    public string FunctionId { get; set; }
		    public string BlobType { get; set; }
		    public string ContainerName { get; set; }
		    public string BlobName { get; set; }
		    public string ETag { get; set; }
		}

### <a id="polling"></a> Blob 轮询算法

启动应用程序时，WebJobs SDK 将扫描 **BlobTrigger** 属性指定的所有容器。在大型存储帐户中，此扫描可能需要一些时间，因此在查找新 blob 和执行 **BlobTrigger** 函数之前，可能需要一段时间。

若要在应用程序启动后检测新的或已更改的 Blob，SDK 会定期读取从 Blob 存储日志。blob 日志将进行缓冲，仅每隔 10 分钟左右获取物理写入，因此创建或更新 blob 后可能存在很长的延迟，然后才会执行对应的 **BlobTrigger** 函数。

使用 **Blob** 属性创建的 blob 出现异常。当 WebJobs SDK 创建新 blob 时，会立即将新的 blob 传递给任何匹配的 **BlobTrigger** 函数。因此，如果建立了 Blob 输入和输出的链接，则 SDK 可以高效地处理它们。但是，如果你想要对通过其他方式创建或更新的 blob 降低运行 blob 处理功能的延迟时间，我们建议使用 **QueueTrigger**（而不是 **BlobTrigger**）。

### <a id="receipts"></a> Blob 回执

WebJobs SDK 确保没有为相同的新 blob 或更新 blob 多次调用 **BlobTrigger** 函数。为此，它会维护blob 回执，以确定是否已处理给定的 blob 版本。

Blob 回执在 AzureWebJobsStorage 连接字符串指定的 Azure 存储帐户中名为 *azure-webjobs-hosts* 的容器内存储。Blob 回执包含以下信息：

* 为 blob 调用的函数（"*{WebJob name}*.Functions.*{Function name}*"，例如 "WebJob1.Functions.CopyBlob"）
* 容器名称
* Blob 类型（"BlockBlob" 或 "PageBlob"）
* Blob 名称
* ETag（blob 版本标识符，例如："0x8D1DC6E70A277EF"）

如果您想要强制重新处理某个 blob，则可以从 *azure-webjobs-hosts* 容器中手动删除该 blob 的 blob 回执。

## <a id="queues"></a> 队列文章涵盖的相关主题

有关如何处理队列消息触发的 blob 处理，或者不特定于 blob 处理的 WebJobs SDK 方案的信息，请参阅[如何通过 WebJobs SDK 使用 Azure 队列存储](/documentation/articles/websites-dotnet-webjobs-sdk-storage-queues-how-to/)。

该文章涵盖的相关主题包括：

* 异步函数
* 多个实例
* 正常关闭
* 在函数正文中使用 WebJobs SDK 属性
* 在代码中设置 SDK 连接字符串。
* 在代码中设置 WebJobs SDK 构造函数参数的值
* 配置 **MaxDequeueCount**，以便进行有害 blob 的处理。
* 手动触发函数
* 写入日志

## <a id="nextsteps"></a>后续步骤

本文提供了代码示例，演示如何处理用于操作 Azure blob 的常见方案。有关如何使用 Azure WebJobs 和 WebJobs SDK 的详细信息，请参阅 [Azure WebJobs 文档资源](/documentation/articles/websites-webjobs-resources/)。
 
<!---HONumber=Mooncake_0606_2016-->