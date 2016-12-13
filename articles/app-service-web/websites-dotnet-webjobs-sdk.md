<properties 
	pageTitle="什么是 Azure WebJobs SDK" 
	description="Azure WebJobs SDK 简介。介绍 SDK，适用的典型方案以及代码示例。" 
	services="app-service\web, storage" 
	documentationCenter=".net" 
	authors="tdykstra" 
	manager="wpickett" 
	editor="jimbe"/>

<tags 
	ms.service="app-service-web" 
	ms.workload="web" 
	ms.tgt_pltfrm="na" 
	ms.devlang="na" 
	ms.topic="article" 
	ms.date="06/01/2016" 
	wacn.date="12/12/2016" 
	ms.author="tdykstra"/>

# 什么是 Azure WebJobs SDK

[AZURE.INCLUDE [azure-sdk-developer-differences](../../includes/azure-sdk-developer-differences.md)]

## <a id="overview"></a>概述

本文解释什么是 WebJobs SDK，回顾部分适用的典型方案并概述如何在代码中使用它。

[Web 作业](/documentation/articles/websites-webjobs-resources/)是 Azure 应用服务的一项功能，可以在与 Web 应用、API 应用或移动应用相同的上下文中运行程序或脚本。[WebJobs SDK](/documentation/articles/websites-webjobs-resources/) 用于简化针对 Web 作业执行的常见任务（如图像处理、队列处理、RSS 聚合、文件维护和发送电子邮件）编写的代码。WebJobs SDK 的内置功能使用 Azure 存储空间和 Service Bus，用于计划任务和处理错误，以及用于许多其他常见方案。此外，还可以扩展其设计。[WebJobs SDK 是开源代码](https://github.com/Azure/azure-webjobs-sdk/)，包含[扩展的开源代码存储库](https://github.com/Azure/azure-webjobs-sdk-extensions/wiki/Binding-Extensions-Overview)。

WebJobs SDK 包括以下组件：

* **NuGet 程序包**。添加到 Visual Studio 控制台应用程序项目的 NuGet 包可提供一个框架，代码可通过使用 WebJobs SDK 属性修饰方法使用此框架。
  
* **仪表板**。Azure 应用服务中包含 WebJobs SDK 的一部分，它可针对使用 NuGet 程序包的程序提供丰富的监视和诊断功能。无需编写代码即可使用这些监视和诊断功能。

## <a id="scenarios"></a>方案

下面是 Azure WebJobs SDK 帮助轻松处理的部分典型方案：

* 图像处理或其他需要频繁使用 CPU 的工作。网站的一项常见功能是上载图像或视频。通常，在上载内容后要进行处理，但又不想在执行此操作时让用户等候。

* 队列处理。Web 前端与后端服务进行通信的常见方式是使用队列。当网站需要完成工作时，它会将消息推送到队列。后端服务会从队列提取消息，并完成工作。可以在图像处理中使用队列：例如，在用户上载多个文件后，会将文件名放置在由后端选取队列消息处理的队列消息中。或者，可以使用队列提高网站响应能力。例如，无需将目录直接写入 SQL 数据库，而可以写入队列并告知用户已完成，然后由后端服务处理高延迟的关系型数据库工作。有关使用图像处理队列的示例，请参阅 [WebJobs SDK 入门教程](/documentation/articles/websites-dotnet-webjobs-sdk-get-started/)。

* RSS 聚合。如果有维护 RSS 源列表的网站，可以在后台进程中提取源中的所有文章。

* 文件维护，例如聚合或清理日志文件。可能拥有由多个站点在不同时间创建的日志文件，结合这些文件便于执行分析工作。或者，要计划每周运行的任务，以清理旧的日志文件。

* 输入 Azure 表。可能拥有要分析的存储文件和 Blob，并要将数据存储在表中。入口函数可能会写入许多行（在某些情况下可能有上百万行），而 WebJobs SDK 可以轻松实现此功能。SDK 还可实时监视进度指示器，例如表中的写入行数。

* 要在后台线程中运行其他长时间运行任务，如[发送电子邮件](https://github.com/victorhurdugaci/AzureWebJobsSamples/tree/master/SendEmailOnFailure)。

* 要按计划运行的任何任务，如每晚执行备份操作。

在许多情况下，可能要扩展 Web 应用以便在多个 VM 上运行，这需要同时运行多个 Web 作业。在某些情况下，这可能导致多次处理相同数据；但是，如果使用 WebJobs SDK 的内置队列、Blob 和服务总线触发器，可轻松实现。SDK 可确保针对每个消息或 Blob 仅处理一次函数。

使用 WebJobs SDK 还可轻松处理常见的错误处理方案。可设置警报以便在函数失败时发送通知，还可设置超时以便在函数未在指定时间限制内完成时自动取消函数。

## <a id="code"></a>代码示例

使用 Azure 存储空间处理典型任务的代码十分简单。在控制台应用程序的 `Main` 方法中创建 `JobHost` 对象，协调调用编写的方法。WebJobs SDK 框架根据方法中使用的 WebJobs SDK 属性，了解调用方法的时间和要使用的参数值。SDK 提供指定调用函数的条件的*触发器*，并提供指定如何获取传入和传出方法参数信息的*绑定器*。

例如，[QueueTrigger](/documentation/articles/websites-dotnet-webjobs-sdk-storage-queues-how-to/) 属性导致在队列中收到消息时调用函数，如果消息格式为字节数组或自定义类型的 JSON，该消息将自动反序列化。每次在 Azure 存储帐户中新建一个 blob 时，[BlobTrigger](/documentation/articles/websites-dotnet-webjobs-sdk-storage-blobs-how-to/) 属性都会触发一个流程。

以下是用于轮询队列并为收到的每个队列消息创建 Blob 的简单程序：

		public static void Main()
		{
		    JobHost host = new JobHost();
		    host.RunAndBlock();
		}

		public static void ProcessQueueMessage([QueueTrigger("webjobsqueue")] string inputText, 
            [Blob("containername/blobname")]TextWriter writer)
		{
		    writer.WriteLine(inputText);
		}

`JobHost` 对象是一组后台函数的容器。`JobHost` 对象可监视函数，观察触发函数的事件，并在发生触发事件时执行函数。可调用 `JobHost` 方法，指示要在当前线程或后台线程中执行容器进程。在此示例中，`RunAndBlock` 方法将在当前线程中持续运行该进程。

由于此示例中的 `ProcessQueueMessage` 方法具有 `QueueTrigger` 属性，因此接收新队列消息时会触发该函数。`JobHost` 对象监视指定队列（在此示例中为“webjobsqueue”）中的新队列消息，发现新队列消息后，此对象将调用 `ProcessQueueMessage`。

`QueueTrigger` 属性将 `inputText` 参数绑定到队列消息的值。`Blob` 将 `TextWriter` 对象绑定到“containername”容器中名为“blobname”的 Blob。

		public static void ProcessQueueMessage([QueueTrigger("webjobsqueue")]] string inputText, 
		    [Blob("containername/blobname")]TextWriter writer)

然后，该函数使用这些参数将队列消息的值写入 Blob：

		writer.WriteLine(inputText);

WebJobs SDK 的触发器和绑定器功能可大幅简化编写代码。处理队列、blob 或文件，或启动计划任务所需的低级代码由 WebJobs SDK 框架编写。例如，该框架可创建尚不存在的队列、打开队列、读取队列消息并在处理完成后删除队列消息、创建尚不存在的 Blob 容器、写入 Blob 等。

以下代码示例显示了一个 Web 作业中的各种触发器：`QueueTrigger`、`FileTrigger`、`WebHookTrigger` 和 `ErrorTrigger`。

    public class Functions
    {
        public static void ProcessQueueMessage([QueueTrigger("queue")] string message,
        TextWriter log)
        {
            log.WriteLine(message);
        }

        public static void ProcessFileAndUploadToBlob(
            [FileTrigger(@"import\{name}", "*.*", autoDelete: true)] Stream file,
            [Blob(@"processed/{name}", FileAccess.Write)] Stream output,
            string name,
            TextWriter log)
        {
            output = file;
            file.Close();
            log.WriteLine(string.Format("Processed input file '{0}'!", name));
        }

        [Singleton]
        public static void ProcessWebHookA([WebHookTrigger] string body, TextWriter log)
        {
            log.WriteLine(string.Format("WebHookA invoked! Body: {0}", body));
        }

        public static void ProcessGitHubWebHook([WebHookTrigger] string body, TextWriter log)
        {
            dynamic issueEvent = JObject.Parse(body);
            log.WriteLine(string.Format("GitHub WebHook invoked! ('{0}', '{1}')",
                issueEvent.issue.title, issueEvent.action));
        }

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
    }

## <a id="schedule"></a> 计划

使用 `TimerTrigger` 属性可以触发要按计划运行的函数。可通过 Azure 从整体上计划 Web 作业，也可使用 WebJobs SDK `TimerTrigger` 计划 Web 作业的各个函数。下面是代码示例。

	public class Functions
	{
    	public static void ProcessTimer([TimerTrigger("*/15 * * * * *", RunOnStartup = true)]
    	TimerInfo info, [Queue("queue")] out string message)
    	{
        	message = info.FormatNextOccurrences(1);
    	}
	}

有关更多示例代码，请参阅 GitHub.com 上 azure-webjobs-sdk-extensions 存储库中的 [TimerSamples.cs](https://github.com/Azure/azure-webjobs-sdk-extensions/blob/master/src/ExtensionsSample/Samples/TimerSamples.cs)。

## 可扩展性

不限于内置功能 - WebJobs SDK 允许编写自定义触发器和绑定器。例如，可以为缓存事件和定期计划编写触发器。[开源存储库](https://github.com/Azure/azure-webjobs-sdk-extensions)包含[有关 WebJobs SDK 可扩展性的详细指南](https://github.com/Azure/azure-webjobs-sdk-extensions/wiki/Binding-Extensions-Overview)和示例代码，有帮助于用户开始编写自己的触发器和绑定器。

## <a id="workerrole"></a>在 WebJobs 外部使用 WebJobs SDK

使用 WebJobs SDK 的程序是指可在任意位置运行的标准控制台应用程序 - 不一定要以 Web 作业的形式运行。在开发计算机上，可本地测试程序；在生产环境中，可在云服务辅助角色或 Windows 服务中运行程序（如果喜欢其中一个环境）。

但是，只能将仪表板用作 Azure 应用服务 Web 应用的扩展。如果要在 WebJob 外部运行且仍然使用仪表板，可将 Web 应用配置为使用 WebJobs SDK 仪表板连接字符串引用的同一存储帐户，然后 Web 应用的 WebJobs 仪表板将显示有关其他位置运行程序的函数执行数据。可以使用 URL https://*{webappname}*.scm.chinacloudsites.cn/azurejobs/#/functions 访问仪表板。有关详细信息，请参阅[使用 WebJobs SDK 获取用于本地开发的仪表板](http://blogs.msdn.com/b/jmstall/archive/2014/01/27/getting-a-dashboard-for-local-development-with-the-webjobs-sdk.aspx)，但请注意，此博客文章会显示旧的连接字符串名称。

## <a id="nostorage"></a>仪表板功能

即使不使用 WebJobs SDK 触发器或绑定器，WebJobs SDK 也具有下列优点：

* 可以从仪表板调用函数。
* 可以从仪表板重放函数。
* 可在仪表板中查看日志，链接到特定的 WebJob（使用 Console.Out、Console.Error、Trace 等编写的应用程序日志）或链接到生成它们的特定函数调用（使用 SDK 传递给函数作为参数的 `TextWriter` 对象编写的日志）。

有关详细信息，请参阅[如何手动调用函数](/documentation/articles/websites-dotnet-webjobs-sdk-storage-queues-how-to/#manual)和[如何编写日志](/documentation/articles/websites-dotnet-webjobs-sdk-storage-queues-how-to/#logs)

## <a id="nextsteps"></a>后续步骤

有关 WebJobs SDK 的详细信息，请参阅[Azure WebJobs 推荐资源](/documentation/articles/websites-webjobs-resources/)。

有关 WebJobs SDK 的最新增强功能的信息，请参阅[发行说明](https://github.com/Azure/azure-webjobs-sdk/wiki/Release-Notes)。
 

<!---HONumber=Mooncake_Quality_Review_1118_2016-->