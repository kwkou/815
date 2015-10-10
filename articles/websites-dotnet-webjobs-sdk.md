<properties 
	pageTitle="什么是 Azure WebJobs SDK" 
	description="Azure WebJobs SDK 简介。介绍 SDK 的定义，它适用的典型方案，以及代码示例。" 
	services="app-service\web, storage" 
	documentationCenter=".net" 
	authors="tdykstra" 
	manager="wpickett" 
	editor="jimbe"/>

<tags 
	ms.service="app-service-web" 
	ms.date="08/05/2015" 
	wacn.date="10/03/2015"/>

# 什么是 Azure WebJobs SDK


## 目录

- [概述](#overview)
- [方案](#scenarios)
- [代码示例](#code)
- [在 Web 作业的外部使用 WebJobs SDK](#workerrole)
- [使用 WebJobs SDK 调用任何函数](#nostorage)
- [后续步骤](#nextsteps)

## <a id="overview"></a>概述

本文说明 WebJobs SDK 是什么、了解部分适用的典型方案，以及提供在代码中的使用方式概述。

[WebJobs](/documentation/articles/websites-webjobs-resources) 是一项 Azure 网站，可让您在与 Web 应用相同的上下文中执行程序或脚本。WebJobs SDK 旨在简化作为 Web 作业运行并适用于 Azure 存储队列、Blob、表和 Service Bus 队列的代码的编写任务。

WebJobs SDK 包括以下组件：

* **NuGet 程序包**。添加到 Visual Studio 控制台应用程序项目的 NuGet 程序包提供一个框架，你的代码可通过此框架来使用 Azure 存储服务或 Service Bus 队列。   
  
* **仪表板**。Azure 网站中包含 WebJobs SDK 的一部分，它可针对使用 NuGet 程序包的程序提供丰富的监视和诊断功能。你无需编写代码就可以使用这些监视和诊断功能。

## <a id="scenarios"></a>方案

下面是 Azure WebJobs SDK 可帮助你轻松处理的部分典型方案：

* 图像处理或其他需要大量 CPU 的工作。网站的一项常见功能是上载图像或视频。在许多时候，你想要在内容上载后处理该内容，但又不想在你执行此任务时让用户等候。

* 队列处理。Web 前端与后端服务的一个常见通信方式是使用队列。当网站需要完成工作时，它会将消息推送到队列。后端服务会从队列提取消息，并完成工作。你可以在图像处理中使用队列：例如，在用户上载多个文件之后，文件名会被放置在队列消息中，由后端选取队列消息进行处理。或者，你可以使用队列来改进网站响应能力。例如，无需将目录直接写入 SQL 数据库，而可以写入队列并告知用户已完成，然后由后端服务处理高延迟的关系型数据库工作。有关使用图像处理的队列处理示例，请参阅 [WebJobs SDK 入门教程](/documentation/articles/websites-dotnet-webjobs-sdk-get-started)。

* RSS 聚合。如果你有维护 RSS 源列表的网站，你可以在后台进程中提取源中的所有文章。

* 文件维护，例如聚合或清理日志文件。你可能拥有由数个网站在不同的时间所创建的日志文件，你想要结合这些文件以便执行分析工作。或者你想要计划每周运行的任务，来清理旧的日志文件。

* 输入 Azure 表。你可能会有想要分析的存储文件和 Blob，并想要将数据存储在表中。入口函数可能会写入许多行（在某些情况下可能有上百万行），而 WebJobs SDK 让你可以轻松地实现此功能。SDK 还提供进度指示器的实时监视，例如表中的写入行数。

* 您想要在后台线程中执行的其他长时间运行任务，例如[发送电子邮件](https://github.com/victorhurdugaci/AzureWebJobsSamples/tree/master/SendEmailOnFailure)。

在许多情况下，你可能想要扩展 Web 应用以便它在多个 VM 上运行，这就需要同时运行多个 Web 作业。在某些情况下，这可能导致相同的数据被处理多次，但如果使用 WebJobs SDK 的内置队列、Blob 和服务总线触发器，则不会造成问题。该 SDK 可确保只会针对每个消息或 Blob 处理函数一次。

## <a id="code"></a>代码示例

使用 Azure 存储空间处理典型任务的代码十分简单。在控制台应用程序中，你可以针对要执行的后台任务编写方法，然后使用 WebJobs SDK 的属性加以修饰。`Main` 方法将创建可以协调调用所编写方法的 `JobHost` 对象。WebJobs SDK 框架会根据方法中所用的 WebJobs SDK 属性，知道何时调用方法。

下面是个一个简单程序，它可用来轮询队列并为收到的每个队列消息创建 Blob：

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

`JobHost` 对象是一组后台函数的容器。`JobHost` 对象可监视函数、观察触发函数的事件，并在发生触发事件时执行函数。您可以调用 `JobHost` 方法，以指示您要在当前线程或后台线程上执行容器进程。在此示例中，`RunAndBlock` 方法将在当前线程上持续运行该进程。

由于此示例中的 `ProcessQueueMessage` 方法具有 `QueueTrigger` 属性，接收新队列消息时便会触发该函数。`JobHost` 对象监视指定队列（在此示例中为“webjobsqueue”）上的新队列消息，查找到新队列消息时，此对象将调用 `ProcessQueueMessage`。

`QueueTrigger` 属性将 `inputText` 参数绑定到队列消息的值。`Blob` 将 `TextWriter` 对象绑定到“containername”容器中名为“blobname”的 Blob。

		public static void ProcessQueueMessage([QueueTrigger("webjobsqueue")]] string inputText, 
		    [Blob("containername/blobname")]TextWriter writer)

然后，该函数使用这些参数将队列消息的值写入 Blob：

		writer.WriteLine(inputText);

WebJobs SDK 的触发器和绑定器功能可大幅简化使用 Azure 存储空间和 Service Bus 队列所要编写的代码。WebJobs SDK 框架将为你完成处理队列和 Blob 处理所需的低级代码 - 该框架会创建尚不存在的队列、打开队列、读取队列消息并在处理完成后删除队列消息、创建尚不存在的 Blob 容器、写入 Blob，等等。

WebJobs SDK 提供多种使用 Azure 存储空间的方法。例如，如果使用 `QueueTrigger` 属性修饰的参数是一个字节数组或自定义类型，则会自动从 JSON 进行反序列化。每次在 Azure 存储帐户中新建了 blob 时，您可以使用 `BlobTrigger` 属性触发一个流程。（请注意，当 `QueueTrigger` 在几秒钟内查找到新队列消息时，`BlobTrigger` 可能需要 20 分钟的时间来检测新 blob。每当 `JobHost` 启动，`BlobTrigger` 就会扫描 blob，然后定期检查 Azure 存储空间日志以检测新 blob。）

## <a id="workerrole"></a>在 WebJobs 外部使用 WebJobs SDK

使用 WebJobs SDK 的程序是指可在任意位置运行的标准控制台应用程序 - 它不一定要以 Web 作业的形式运行。你可以在开发计算机上本地测试程序，而在生产环境中，可以在云服务辅助角色或 Windows 服务中运行程序（如果你偏好其中一个环境）。

但是，仪表板仅能用作 Azure 网站 Web 应用的扩展。如果您想要在 WebJob 外部运行并且仍使用仪表板，可将 Web 应用配置为使用您的 WebJobs SDK 仪表板连接字符串引用的同一存储帐户，然后，Web 应用的 WebJobs 仪表板将显示有关来自其他某处运行程序的函数执行数据。可以使用 URL https://*{webappname}*.scm.chinacloudsites.cn/azurejobs/#/functions 来访问仪表板。有关详细信息，请参阅[使用 WebJobs SDK 获取用于本地开发的仪表板](http://blogs.msdn.com/b/jmstall/archive/2014/01/27/getting-a-dashboard-for-local-development-with-the-webjobs-sdk.aspx)，但请注意，此博客文章会显示旧的连接字符串名称。

## <a id="nostorage"></a>使用 WebJobs SDK 调用任何函数

即使你不需要直接处理 Azure 存储队列、表、或 Blob 或 Service Bus 队列，WebJobs SDK 仍然提供以下优点：

* 可以从仪表板调用函数。
* 可以从仪表板重放函数。
* 您可以在仪表板中查看日志，链接到特定的 WebJob（使用 Console.Out、Console.Error、Trace 等编写的应用程序日志）或链接到生成它们的特定函数调用（使用此 SDK 传递给函数作为参数的 `TextWriter` 对象编写的日志）。 

* 有关详细信息，请参阅[如何手动调用函数](/documentation/articles/websites-dotnet-webjobs-sdk-storage-queues-how-to#manual)和[如何编写日志](/documentation/articles/websites-dotnet-webjobs-sdk-storage-queues-how-to#logs)

## <a id="nextsteps"></a>后续步骤

有关 WebJobs SDK 的详细信息，请参阅[Azure WebJobs 推荐资源](/documentation/articles/websites-webjobs-resources/
)。
 

<!---HONumber=71-->