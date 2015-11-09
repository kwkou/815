<properties
	pageTitle="教程 - 适用于 .NET 的 Azure Batch Apps 库入门"
	description="了解 Azure Batch Apps 的基本概念，以及如何使用一个简单方案进行开发"
	services="batch"
	documentationCenter=".net"
	authors="yidingzhou"
	manager="timlt"
	editor=""/>

<tags
	ms.service="batch"
	ms.date="07/21/2015"
	wacn.date="10/30/2015"/>




# 适用于 .NET 的 Azure Batch Apps 库入门
本教程说明如何在 Azure 批处理( Batch )上，使用 Batch Apps 服务运行并行计算工作负载。

Batch Apps 是 Azure  批处理( Batch )的一项功能，提供以应用程序为中心的方法来管理和执行批处理( Batch )工作负载。使用 Batch Apps SDK 可以创建包，以便能够 批处理( Batch )应用程序，并将这些包部署在自己的帐户中，或提供给其他 批处理( Batch )用户使用。批处理( Batch )还提供数据管理、作业监视、内置的诊断和日志记录，以及任务间依赖关系支持。此外，它还包含一个管理门户，你可以在其中管理作业、查看日志和下载输出，而无需编写自己的客户端。

在 Batch Apps 方案中，可以使用 Batch Apps 云 SDK 编写代码，将作业分割成并行任务、描述这些任务之间的任何依赖关系，以及指定如何执行每个任务。此代码将部署到 批处理( Batch )帐户。然后，客户端只需指定作业类型和 REST API 的输入文件，即可执行作业。

>[AZURE.NOTE]若要完成本教程，你需要一个 Azure 帐户。只需几分钟即可创建一个免费试用帐户。有关详细信息，请参阅 [Azure 试用](http://www.windowsazure.cn/zh-cn/pricing/1rmb-trial/)。可以使用 NuGet 来获取 <a href="http://www.nuget.org/packages/Microsoft.Azure.Batch.Apps.Cloud/">Batch Apps 云</a>程序集和 <a href="http://www.nuget.org/packages/Microsoft.Azure.Batch.Apps/">Batch Apps 客户端</a>程序集。在 Visual Studio 中创建项目后，可在“解决方案资源管理器”中右键单击该项目，然后选择“管理 NuGet 包”。你还可以下载 Batch Apps 的 Visual Studio 扩展，其中包含可启用应用程序云功能的项目模板和部署应用程序的功能。若要下载该扩展，请单击<a href="https://visualstudiogallery.msdn.microsoft.com/8b294850-a0a5-43b0-acde-57a07f17826a">此处</a>，或者在 Visual Studio 中通过“扩展和更新”菜单项搜索 **Batch Apps**。你也可以查找 <a href="https://go.microsoft.com/fwLink/?LinkID=512183&clcid=0x409">MSDN 上的端到端示例。</a>
>

###Azure Batch Apps 基础知识
 批处理( Batch )旨在用于处理现有的计算密集型应用程序。它利用现有的应用代码，并在动态、虚拟化、通用环境中运行。为了使应用程序能够与 Batch Apps 配合运行，必须先完成一些准备工作：

1.	准备现有应用程序可执行文件（在传统服务器场或群集中运行的相同可执行文件）的 zip 文件，以及它所需的任何支持文件。然后，使用管理门户或 REST API 将此 zip 文件上载到 批处理( Batch )帐户。
2.	创建“云程序集”（用于将工作负载分派给应用程序）的 zip 文件，然后通过管理门户或 REST API 上载它。云程序集包含两个针对云 SDK 构建的组件：
	1.	作业拆分器 - 将作业分解成可独立处理的任务。例如，在动画案例中，作业拆分器将使用某个影片作业，并将影片分割成若干帧。
	2.	任务处理器– 调用给定任务的应用程序可执行文件。例如，在动画案例中，任务处理器将调用一个渲染程序，以渲染现有任务指定的单个帧。
3.	提供一种方式以将作业提交给 Azure 中启用的应用程序。这可能是应用程序 UI 中的插件、Web 门户，甚至是执行管道中的无人值守服务。有关示例，请参阅 MSDN 上的<a href="https://go.microsoft.com/fwLink/?LinkID=512183&clcid=0x409">示例</a>。



###Batch Apps 的重要概念
Batch Apps 的编程和使用模式围绕以下重要概念：

####作业
**作业**是用户提交的一项工作。提交作业后，用户将指定作业的类型、该作业的任何设置，以及作业所需的数据。启用的实现可以代表用户确定这些详细信息，或者，在某些情况下，用户可以通过客户端显式提供此信息。作业具有返回的结果。每个作业都有主要输出，有时还有预览输出。如果需要，作业还可以返回额外的输出。

####任务
**任务**是要在作业中完成的一项工作。在用户提交作业后，该作业将分解为一些较小的任务。然后，服务将处理其中的各个任务，再将任务结果组合成完整的作业输出。任务的性质取决于作业的类型。作业拆分器根据应用程序主要处理的任务部分，定义如何将作业细分为任务。下载任务输出时，也可以单独下载，这在某些情况下可能很有用，例如，用户可能想要从动画作业中下载单个任务。

####合并任务
**合并任务**是将单个任务的结果组合成最终作业结果的特殊任务。在影片渲染作业中，合并任务可能会将渲染的帧组合成影片，或将所有渲染的帧压缩成单个文件。即使不需要实际进行任何“合并”，每个作业仍会有合并任务。

####文件
**文件**是输入到作业中的数据片段。作业可能没有关联的输入文件，也可能有一或多个关联的输入文件。相同的文件也可以用于多个作业，例如，对于影片渲染作业，文件可能是纹理、模型，等等。对于数据分析作业，文件可能是一组观察值或测量值。

###启用云应用程序
应用程序必须包含静态字段或属性，该字段或属性包含应用程序的所有详细信息。它指定应用程序的名称，以及应用程序所处理的作业类型。使用 SDK（可通过 Visual Studio 库下载）中的模板时，便会提供这个字段或属性。

下面是并行工作负载的云应用程序声明的一个示例：

	public static class ApplicationDefinition
	{
	    public static readonly CloudApplication App = new ParallelCloudApplication
	    {
	        ApplicationName = "ApplicationName",
	        JobType = "ApplicationJobType",
	        JobSplitterType = typeof(MyJobSplitter),
	        TaskProcessorType = typeof(MyTaskProcessor),
	    };
	}

####实现作业拆分器和任务处理器
对于易并行工作负载，必须实现一个作业拆分器和一个任务处理器。

####实现 JobSplitter.SplitIntoTasks
SplitIntoTasks 实现必须任务序列。每个任务代表一项独立的工作，它将被排入队列供计算节点处理。每个任务都必须是独立的，并且必须设有任务处理器所需的全部信息。

作业拆分器指定的任务以 TaskSpecifier 对象表示。TaskSpecifier 有一些可以在返回任务之前设置的属性。


-	TaskIndex 将被忽略，但可供任务处理器使用。你可以使用此属性将索引传递给任务处理器。如果不需要传递索引，则不需要设置此属性。
-	默认情况下，Parameters 是一个空集合。可以将项目添加给它，或将它替换为新集合。可以使用 WithJobParameters 或 WithAllJobParameters 方法从作业参数集合中复制条目。  


默认情况下，RequiredFiles 是一个空集合。可以将项目添加给它，或将它替换为新集合。可以使用 RequiringJobFiles 或 RequiringAllJobFiles 方法从作业文件集合中复制条目。

可以指定依赖于另一个特定任务的任务。为此，请设置 TaskSpecifier.DependsOn 属性，并传递它所依赖的任务 ID：

	new TaskSpecifier {
	    DependsOn = TaskDependency.OnId(5)
	}

任务必须等到依赖的任务生成输出之后才会运行。

你还可以指定整组任务必须等到另一组任务完成之后才开始处理。在此情况下，可以设置 TaskSpecifier.Stage 属性。具有给定 Stage 值的任务必须等到 Stage 值更小的所有任务完成之后，才会开始处理。例如，必须等到 Stage 0、 1 或 2 的所有任务完成之后，具有 Stage 3 的任务才会开始处理。Stage 必须从 0 开始，各个 Stage 的顺序必须没有间隔，并且 SplitIntoTasks 必须依 Stage 顺序返回任务：例如，在返回 Stage 为 1 的任务之后返回 Stage 为 0 的任务，即为错误。

每个并行操作以一项称为合并任务的特殊任务来结束。合并任务的作业是将并行处理任务的结果组合成最终结果。Batch Apps 会自动为你创建合并任务。

在极少数情况下，你可能想要显式控制合并任务，例如，自定义其参数。在此情况下，可以返回 TaskSpecifier 并将其 IsMerge 属性设置为 true，以指定合并任务。这会覆盖自动合并任务。如果要创建显式合并任务：

-	可以只创建一个显式合并任务
-	它必须是序列中的最后一个任务
-	它必须将 IsMerge 设置为 true，而且其他每个任务必须将 IsMerge 设置为 false  


但请记住，通常不需要显式创建合并任务。

以下代码演示了 SplitIntoTasks 的一个简单实现。

	protected override IEnumerable<TaskSpecifier> SplitIntoTasks(
	    IJob job,
	    JobSplitSettings settings)
	{
	    int start = Int32.Parse(job.Parameters["startIndex"]);
	    int end = Int32.Parse(job.Parameters["endIndex"]);
	    int count = (end - start) + 1;

	    // Processing tasks
	    for (int i = 0; i < count; ++i)
	    {
	        yield return new TaskSpecifier
	        {
	            TaskIndex = start + i,
	        }.RequiringAllJobFiles();
	    }
	}
####实现 ParallelTaskProcessor.RunExternalTaskProcess

从作业拆分器返回的每个非合并任务都会调用 RunExternalTaskProcess。它应该使用适当的参数来调用你的应用程序，并返回需要保留以供将来使用的输出集合。

以下代码段演示了如何调用名为 application.exe 的程序。请注意，可以使用 ExecutablePath 帮助器方法来创建绝对文件路径。

云 SDK 中的 ExternalProcess 类提供了有用的帮助器逻辑来运行应用程序可执行文件。ExternalProcess 可以处理取消操作、将退出代码转换为异常、捕获标准输出和标准错误，以及设置环境变量。此外，你也可以根据需要直接使用 .NET 进程类来运行程序。

RunExternalTaskProcess 方法返回 TaskProcessResult，其中包含输出文件的列表。至少必须包含合并所需的所有文件。可以选择性地返回日志文件、预览文件和中间文件（例如，用于在任务失败时进行诊断）。请注意，你的方法将返回文件路径，而不是文件内容。

每个文件都必须以它所包含的输出类型来标识：output（即，最终作业输出的一部分）、preview、log 或 intermediate。这些值来自 TaskOutputFileKind 枚举。以下代码段将返回单个任务输出，而不返回预览或日志文件。TaskProcessResult.FromExternalProcessResult 方法可以简化从命令行程序中捕获退出代码、处理器输出和输出文件的常见方案：

以下代码演示了 ParallelTaskProcessor.RunExternalTaskProcess 的一个简单实现。

	protected override TaskProcessResult RunExternalTaskProcess(
	    ITask task,
	    TaskExecutionSettings settings)
	{
	    var inputFile = LocalPath(task.RequiredFiles[0].Name);
	    var outputFile = LocalPath(task.TaskId.ToString() + ".jpg");

	    var exePath = ExecutablePath(@"application\application.exe");
	    var arguments = String.Format("-in:{0} -out:{1}", inputFile, outputFile);

	    var result = new ExternalProcess {
	        CommandPath = exePath,
	        Arguments = arguments,
	        WorkingDirectory = LocalStoragePath,
	        CancellationToken = settings.CancellationToken
	    }.Run();

	    return TaskProcessResult.FromExternalProcessResult(result, outputFile);
	}
####实现 ParallelTaskProcessor.RunExternalMergeProcess

这是为合并任务调用的方法。它应该调用应用程序来组合先前任务的输出（以适合应用程序的任何方式），然后返回组合的输出。

RunExternalMergeProcess 的实现非常类似于 RunExternalTaskProcess，差别在于：

-	RunExternalMergeProcess 使用先前任务的输出而不是用户输入文件。因此，应该根据任务 ID 来判断想要处理的文件的名称，而不是从 Task.RequiredFiles 集合来判断。
-	RunExternalMergeProcess 返回 JobOutput 文件，并选择性地返回 JobPreview 文件。


以下代码演示了 ParallelTaskProcessor.RunExternalMergeProcess 的一个简单实现。

	protected override JobResult RunExternalMergeProcess(
	    ITask mergeTask,
	    TaskExecutionSettings settings)
	{
	    var outputFile =  "output.zip";

	    var exePath =  ExecutablePath(@"application\application.exe");
	    var arguments = String.Format("a -application {0} *.jpg", outputFile);

	    new ExternalProcess {
	        CommandPath = exePath,
	        Arguments = arguments,
	        WorkingDirectory = LocalStoragePath
	    }.Run();

	    return new JobResult {
	        OutputFile = outputFile
	    };
	}

###将作业提交到 Batch Apps
作业描述要运行的工作负载，需要包含有关工作负载的所有信息（文件内容除外）。例如，作业包含从客户端流向作业拆分器，再流向任务的配置设置。MSDN 上提供的示例演示了如何提交作业并提供多个客户端，包括 Web 门户和命令行客户端。

<!---HONumber=66-->