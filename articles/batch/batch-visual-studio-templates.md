<properties
    pageTitle="Azure Batch 的 Visual Studio 模板 | Azure"
    description="了解这些 Visual Studio 项目模板如何帮助在 Azure Batch 上实现和运行计算密集型工作负荷"
    services="batch"
    documentationcenter=".net"
    author="fayora"
    manager="timlt"
    editor="" />
<tags
    ms.assetid="5e041ae2-25af-4882-a79e-3aa63c4bfb20"
    ms.service="batch"
    ms.devlang="multiple"
    ms.topic="article"
    ms.tgt_pltfrm="vm-windows"
    ms.workload="big-compute"
    ms.date="01/05/2017"
    wacn.date="02/22/2017"
    ms.author="tamram" />  


# Azure Batch 的 Visual Studio 项目模板
Batch 的**作业管理器**和**任务处理器** Visual Studio 模板提供代码来帮助以最少的精力在 Batch 上实现并运行计算密集型工作负荷。本文档介绍这些模板，并提供其用法指导。

> [AZURE.IMPORTANT]
本文只介绍适用于这两个模板的信息，假设读者熟悉与其相关的 Batch 服务和重要概念：池、计算节点、作业和任务、作业管理器任务、环境变量和其他相关信息。可以在 [Basics of Azure Batch](/documentation/articles/batch-technical-overview/)（Azure Batch 基础知识）、[Batch feature overview for developers](/documentation/articles/batch-api-basics/)（面向开发人员的 Batch 功能概述）和 [Get started with the Azure Batch library for .NET](/documentation/articles/batch-dotnet-get-started/)（用于 .NET 的 Azure Batch 库入门）中找到更多信息。
> 
> 

## 综合概述
作业管理器和任务处理器模板可用于创建两个有用的组件：

- 作业管理器任务实现作业拆分器，后者可将作业细分为多个可以并行独立运行的任务。
- 任务处理器可用于围绕应用程序命令行执行前处理和后处理。

例如，在电影渲染方案中，作业拆分器将单个电影作业转变成数百个或数千个单独处理各个帧的不同任务。相应地，任务处理器调用为了渲染每个帧所需的渲染应用程序和所有依赖进程，执行任何额外操作（例如，将渲染的帧复制到存储位置）。

> [AZURE.NOTE]
作业管理器和任务处理器模板彼此独立，因此可以根据计算作业要求和个人喜好，选择同时使用两者或只使用其中之一。
> 
> 

如下图所示，使用这些模板的计算作业经历三个阶段：

1. 客户端代码（例如，应用程序、Web 服务等）将作业提交到 Azure 上的 Batch 服务，将其作业管理器任务指定为作业管理器程序。
2. Batch 服务在计算节点上运行作业管理器任务，作业拆分器根据作业拆分器代码中的参数和规范，在任意数量的所需计算节点上启动指定数量的任务处理器任务。
3. 任务处理器任务以并行方式独立运行，处理输入数据并生成输出数据。

![显示客户端代码与 Batch 服务交互的示意图][diagram01]  


## 先决条件
若要使用 Batch 模板，需要满足以下条件：

- 已安装 Visual Studio 2015 或更高版本的计算机。
- Batch 模板，可从 [Visual Studio 库][vs_gallery]以 Visual Studio 扩展的形式获取。有两种方式可获取模板：
  
  - 使用 Visual Studio 的“扩展和更新”对话框安装模板（有关详细信息，请参阅 [Finding and Using Visual Studio Extensions][vs_find_use_ext]（查找和使用 Visual Studio 扩展））。在“扩展和更新”对话框中，搜索并下载以下两个扩展：
    
    - 随附作业拆分器的 Azure Batch 作业管理器
    - Azure Batch 任务处理器
  - 从 Visual Studio 的联机库下载模板：[Azure Batch 项目模板][vs_gallery_templates]
- 如果打算使用[应用程序包](/documentation/articles/batch-application-packages/)功能将作业管理器和任务处理器部署到 Batch 计算节点，需要将存储帐户链接到 Batch 帐户。

## 准备工作
建议创建可在其中包含作业管理器和任务处理器的解决方案，因为这样可以更轻松地在作业管理器和任务处理器程序之间共享代码。若要创建此解决方案，请遵循以下步骤：

1. 打开 Visual Studio 2015，然后选择“文件”>“新建”>“项目”。
2. 在“模板”下展开“其他项目类型”，单击“Visual Studio 解决方案”，然后选择“空白解决方案”。
3. 键入用于描述应用程序和此解决方案用途的名称（例如，“LitwareBatchTaskPrograms”）。
4. 若要创建新解决方案，请单击“确定”。

## 作业管理器模板
作业管理器模板可帮助实现作业管理器任务以执行以下操作：

- 将一个作业拆分为多个任务。
- 提交这些任务以在 Batch 上运行。

> [AZURE.NOTE]
有关作业管理器任务的详细信息，请参阅 [Batch feature overview for developers](/documentation/articles/batch-api-basics/#job-manager-task/)（面向开发人员的 Batch 功能概述）。
> 
> 

### 使用模板创建作业管理器
若要在前面创建的解决方案中添加作业管理器，请遵循以下步骤：

1. 在 Visual Studio 2015 中打开现有解决方案。
2. 在解决方案资源管理器中右键单击解决方案，然后单击“添加”>“新建项目”。
3. 在“Visual C#”下面单击“云”，然后单击“随附作业拆分器的 Azure Batch 作业管理器”。
4. 键入用于描述应用程序并将此项目标识为作业管理器的名称（例如“LitwareJobManager”）。
5. 若要创建项目，请单击“确定”。
6. 最后，生成项目来强制 Visual Studio 加载所有引用的 NuGet 包，并验证项目是否有效以便能开始对其进行修改。

### 作业管理器模板文件及其用途
使用作业管理器模板创建项目时，它生成三组代码文件：

- 主程序文件 (Program.cs)。此文件包含程序入口点和顶层异常处理。一般情况下，不需要修改此文件。
- 框架目录。此目录包含的文件负责处理作业管理器程序执行的样板工作，例如解压缩参数、在 Batch 作业中添加任务等。一般情况下，不需要修改这些文件。
- 作业拆分器文件 (JobSplitter.cs)。此处可供存放用于将作业拆分为多个任务的应用程序特定逻辑。

当然，可以根据作业拆分逻辑的复杂性，视需要添加其他文件来支持作业拆分器代码。

该模板还会生成标准 .NET 项目文件，例如 .csproj 文件、app.config、packages.config 等等。

本部分的余下内容介绍不同文件和其代码结构，并解释每个类的用途。

![显示作业管理器模板解决方案的 Visual Studio 解决方案资源管理器][solution_explorer01]  


**框架文件**

- `Configuration.cs`：封装作业配置数据的加载，例如 Batch 帐户详细信息、链接的存储帐户凭据、作业和任务信息，以及作业参数。它还通过 Configuration.EnvironmentVariable 类提供 Batch 定义的环境变量（请参阅 Batch 文档中“Environment settings for tasks”（任务的环境设置））的访问权限。
- `IConfiguration.cs`：抽象化配置类的实现，以便可以使用虚构或模拟的配置对象对作业拆分器进行单元测试。
- `JobManager.cs`：协调作业管理器程序的组件。它负责初始化作业拆分器、调用作业拆分器，以及将作业拆分器返回的任务分派给任务提交器。
- `JobManagerException.cs`：代表需要由作业管理器终止的错误。JobManagerException 用于包装可在终止过程中提供特定诊断信息的“预期”错误。
- `TaskSubmitter.cs`：此类负责将作业拆分器返回的任务添加到 Batch 作业。JobManager 类将一连串任务聚合成批，以便有效、及时地添加到作业中，然后在每一批的后台线程上调用 TaskSubmitter.SubmitTasks。

**作业拆分器**

`JobSplitter.cs`：此类包含用于将作业拆分为多个任务的应用程序特定逻辑。框架调用 JobSplitter.Split 方法以获取一连串的任务，并在方法返回任务时将方法添加到作业中。这是将在其中注入作业逻辑的类。实现拆分方法，返回一连串代表要将作业拆分成的任务的 CloudTask 实例。

**标准 .NET 命令行项目文件**

- `App.config`：标准的 .NET 应用程序配置文件。
- `Packages.config`：标准的 NuGet 包依赖项文件。
- `Program.cs`：包含程序入口点和顶层异常处理。

### 实现作业拆分器
当打开作业管理器模板项目时，项目默认情况下打开 JobSplitter.cs 文件。可按如下所示使用 Split() 方法为工作负荷中的任务实现拆分逻辑：

csharp

	/// <summary>
	/// Gets the tasks into which to split the job. This is where you inject
	/// your application-specific logic for decomposing the job into tasks.
	///
	/// The job manager framework invokes the Split method for you; you need
	/// only to implement it, not to call it yourself. Typically, your
	/// implementation should return tasks lazily, for example using a C#
	/// iterator and the "yield return" statement; this allows tasks to be added
	/// and to start running while splitting is still in progress.
	/// </summary>
	/// <returns>The tasks to be added to the job. Tasks are added automatically
	/// by the job manager framework as they are returned by this method.</returns>
	public IEnumerable<CloudTask> Split()
	{
	    // Your code for the split logic goes here.
	    int startFrame = Convert.ToInt32(_parameters["StartFrame"]);
	    int endFrame = Convert.ToInt32(_parameters["EndFrame"]);

	    for (int i = startFrame; i <= endFrame; i++)
	    {
	        yield return new CloudTask("myTask" + i, "cmd /c dir");
	    }
	}

> [AZURE.NOTE]
在 `Split()` 方法中，批注部分是作业管理器模板代码中唯一可修改的部分，方法是添加用于将作业拆分成不同任务的逻辑。如果想要修改模板的其他部分，请确定熟悉 Batch 的工作原理，并先在几个 [Batch 代码示例][github_samples]中试试看。
> 
> 

Split() 实现具有以下项的访问权限：

- 作业参数，通过 `_parameters` 字段。
- 代表作业的 CloudJob 对象，通过 `_job` 字段。
- 代表作业管理器任务的 CloudTask 对象，通过 `_jobManagerTask` 字段。

`Split()` 实现不需要直接将任务添加到作业中。相反地，代码应返回一连串的 CloudTask 对象，并由调用作业拆分器的框架类自动添加到作业中。通常使用 C# 的迭代器 (`yield return`) 功能实现作业拆分器，因为这可让任务尽快开始运行，而不是等待所有要计算的任务。

**作业拆分器失败**

如果作业拆分器发生错误，它应该：

- 使用 C# `yield break` 语句终止序列，在此情况下，将作业管理器视为成功；或者
- 引发异常，在此情况下，将作业管理器视为失败，并可能根据客户端对它的配置进行重试。

在这两种情况下，作业拆分器已返回并添加到 Batch 作业的任何任务都有资格运行。如果不想让此情况发生，可以：

- 终止作业，不让它从作业拆分器返回
- 先编写整个任务集合再将它返回（也就是返回 `ICollection<CloudTask>` 或 `IList<CloudTask>`，而不是使用 C# 迭代器实现作业拆分器）
- 使用任务依赖项让所有任务依赖于成功完成作业管理器

**作业管理器重试**

根据客户端重试设置，如果作业管理器失败，Batch 服务可能会重试。通常这很安全，因为当框架将任务添加到作业时，会忽略任何已存在的任务。但是，如果计算任务需要很高的成本，可能不希望由于重新计算已添加到作业的任务而生成成本，相反地，如果重新运行不保证生成相同的任务 ID，则“忽略重复项”的行为不会开始运行。在这些情况下，应该将作业拆分器设计为检测已完成的任务而不进行重复，例如，通过在开始生成任务之前先运行 CloudJob.ListTasks。

### 作业管理器模板中的退出代码和异常
退出代码和异常提供了机制来确定程序的运行结果，并可帮助找到任何程序执行问题。作业管理器模板实现本部分所述的退出代码和异常。

使用作业管理器模板实现的作业管理器任务返回三个可能的退出代码：

| 代码 | 说明 |
| --- | --- |
| 0 |作业管理器成功完成。作业拆分器代码已运行完成，并且所有任务都已添加到作业中。 |
| 1 |作业管理器任务失败，程序的“预期”部分有异常。异常已转换成 JobManagerException 与诊断信息，如有可能，还提供可解决失败的建议。 |
| 2 |作业管理器任务失败，发生“意外的”异常。异常已记录到标准输出，但作业管理器无法添加任何额外的诊断或补救信息。 |

在作业管理器任务失败的情况下，某些任务可能仍在错误发生之前就已添加到服务中。这些任务将正常运行。请参阅上面的“作业拆分器失败”，获取有关此代码路径的介绍。

异常返回的所有信息已写入 stdout.txt 和 stderr.txt 文件。有关详细信息，请参阅[错误处理](/documentation/articles/batch-api-basics/#error-handling/)。

### 客户端注意事项
本部分说明在根据此模板调用作业管理器时的一些客户端实现要求。请参阅 [How to pass parameters and environment variables from the client code](#pass-environment-settings)（如何从客户端代码传递参数和环境变量），获取有关传递参数和环境设置的详细信息。

**必需的凭据**

若要将任务添加到 Azure Batch 作业，作业管理器任务需要 Azure Batch 帐户 URL 和密钥。必须在名为 YOUR\_BATCH\_URL 和 YOUR\_BATCH\_KEY 的环境变量中传递这些凭据。可以在作业管理器任务的环境设置中设置这些变量。例如，在 C# 客户端中：

csharp

	job.JobManagerTask.EnvironmentSettings = new [] {
	    new EnvironmentSetting("YOUR_BATCH_URL", "https://account.region.batch.azure.com"),
	    new EnvironmentSetting("YOUR_BATCH_KEY", "{your_base64_encoded_account_key}"),
	};

**存储凭据**

一般而言，客户端不需要提供链接的存储帐户凭据给作业管理器任务，因为 (a) 大多数作业管理器不需要明确访问链接的存储帐户，并且 (b) 链接的存储帐户通常提供给所有任务，作为作业的通用环境设置。如果未通过通用环境设置提供链接的存储帐户，并且作业管理器需要访问链接的存储，则应按如下所示提供链接的存储凭据：

csharp

	job.JobManagerTask.EnvironmentSettings = new [] {
	    /* other environment settings */
	    new EnvironmentSetting("LINKED_STORAGE_ACCOUNT", "{storageAccountName}"),
	    new EnvironmentSetting("LINKED_STORAGE_KEY", "{storageAccountKey}"),
	};

**作业管理器任务设置**

客户端应该将作业管理器的 *killJobOnCompletion* 标志设置为 **false**。

客户端通常可以安全地将 *runExclusive* 设置为 **false**。

客户端应使用 *resourceFiles* 或 *applicationPackageReferences* 集合将作业管理器可执行文件（及其所需的 DLL）部署到计算节点。

默认情况下，作业管理器在失败时不重试。根据作业管理器逻辑，客户端可能需要通过 *constraints*/*maxTaskRetryCount* 启用重试。

**作业设置**

如果作业拆分器发出具有依赖项的任务，客户端必须将作业的 usesTaskDependencies 设置为 true。

在作业拆分器模型中，除了作业拆分器所创建的任务外，客户端通常不需要将任务添加到作业中。因此一般而言，客户端应该将作业的 *onAllTasksComplete* 设置为 **terminatejob**。

## 任务处理器模板
任务处理器模板可帮助实现任务处理器来执行以下操作：

- 设置要让每个 Batch 任务运行所需的信息。
- 运行每个 Batch 任务所需的所有操作。
- 将任务输出存储到持久性存储。

尽管不需要任务处理器就能在 Batch 上运行任务，但使用任务处理器的主要优点是，提供包装器以在一个位置实现所有任务执行操作。例如，如果需要在每个任务的上下文中运行多个应用程序，或者如果需要在完成各项任务之后将数据复制到持久性存储。

任务处理器执行的操作可根据工作负荷所需调整为任意复杂性和数量。此外，通过将所有任务操作实现到一个任务处理器中，可以根据应用程序或工作负荷要求的更改，轻松地更新或添加操作。但是，在某些情况下，任务处理器可能不是最适合实现的解决方案，因为它增加不必要的复杂性，例如，在运行可以从简单命令行快速启动的作业时。

### 使用模板创建任务处理器
若要在前面创建的解决方案中添加任务处理器，请遵循以下步骤：

1. 在 Visual Studio 2015 中打开现有解决方案。
2. 在解决方案资源管理器中右键单击解决方案，单击“添加”，然后单击“新建项目”。
3. 在“Visual C#”下面单击“云”，然后单击“Azure Batch 任务处理器”。
4. 键入用于描述应用程序并将此项目标识为任务处理器的名称（例如“LitwareTaskProcessor”）。
5. 若要创建项目，请单击“确定”。
6. 最后，生成项目来强制 Visual Studio 加载所有引用的 NuGet 包，并验证项目是否有效以便能开始对其进行修改。

### 任务处理器模板文件及其用途
使用任务处理器模板创建项目时，它生成三组代码文件：

- 主程序文件 (Program.cs)。此文件包含程序入口点和顶层异常处理。一般情况下，不需要修改此文件。
- 框架目录。此目录包含的文件负责处理作业管理器程序执行的样板工作，例如解压缩参数、在 Batch 作业中添加任务等。一般情况下，不需要修改这些文件。
- 任务处理器文件 (TaskProcessor.cs)。此文件可供存放用于执行任务的应用程序特定逻辑（通常是通过向外调用现有的可执行文件）。预处理和和后处理代码（例如下载额外数据或上载结果文件）也存放在此。

当然，可以根据作业拆分逻辑的复杂性，视需要添加其他文件来支持任务处理器代码。

该模板还会生成标准 .NET 项目文件，例如 .csproj 文件、app.config、packages.config 等等。

本部分的余下内容介绍不同文件和其代码结构，并解释每个类的用途。

![显示任务处理器模板解决方案的 Visual Studio 解决方案资源管理器][solution_explorer02]  


**框架文件**

- `Configuration.cs`：封装作业配置数据的加载，例如 Batch 帐户详细信息、链接的存储帐户凭据、作业和任务信息，以及作业参数。它还通过 Configuration.EnvironmentVariable 类提供 Batch 定义的环境变量（请参阅 Batch 文档中“Environment settings for tasks”（任务的环境设置））的访问权限。
- `IConfiguration.cs`：抽象化配置类的实现，以便可以使用虚构或模拟的配置对象对作业拆分器进行单元测试。
- `TaskProcessorException.cs`：代表需要由作业管理器终止的错误。TaskProcessorException 用于包装可在终止过程中提供特定诊断信息的“预期”错误。

**任务处理器**

- `TaskProcessor.cs`：运行任务。框架调用 TaskProcessor.Run 方法。这是将在其中注入任务的应用程序特定逻辑的类。实现 Run 方法以便：
  - 分析和验证任何任务参数
  - 针对要调用的任何外部程序编写命令行
  - 记录为了调试所可能需要的任何诊断信息
  - 使用该命令行启动进程
  - 等待进程退出
  - 捕获进程的退出代码以确定其成功还是失败
  - 保存所有想要保留在持久性存储中的输出文件

**标准 .NET 命令行项目文件**

- `App.config`：标准的 .NET 应用程序配置文件。
- `Packages.config`：标准的 NuGet 包依赖项文件。
- `Program.cs`：包含程序入口点和顶层异常处理。

## 实现任务处理器
当打开任务处理器模板项目时，项目默认情况下打开 TaskProcessor.cs 文件。可按如下所示使用 Run() 方法为工作负荷中的任务实现运行逻辑：

csharp

	/// <summary>
	/// Runs the task processing logic. This is where you inject
	/// your application-specific logic for decomposing the job into tasks.
	///
	/// The task processor framework invokes the Run method for you; you need
	/// only to implement it, not to call it yourself. Typically, your
	/// implementation will execute an external program (from resource files or
	/// an application package), check the exit code of that program and
	/// save output files to persistent storage.
	/// </summary>
	public async Task<int> Run()

	{
	    try
	    {
	        //Your code for the task processor goes here.
	        var command = $"compare {_parameters["Frame1"]} {_parameters["Frame2"]} compare.gif";
	        using (var process = Process.Start($"cmd /c {command}"))
	        {
	            process.WaitForExit();
	            var taskOutputStorage = new TaskOutputStorage(
	            _configuration.StorageAccount,
	            _configuration.JobId,
	            _configuration.TaskId
	            );
	            await taskOutputStorage.SaveAsync(
	            TaskOutputKind.TaskOutput,
	            @"..\stdout.txt",
	            @"stdout.txt"
	            );
	            return process.ExitCode;
	        }
	    }
	    catch (Exception ex)
	    {
	        throw new TaskProcessorException(
	        $"{ex.GetType().Name} exception in run task processor: {ex.Message}",
	        ex
	        );
	    }
	}

>[AZURE.NOTE] Run() 方法中的批注部分是任务处理器模板代码中唯一可修改的部分，方法是为工作负荷中的任务添加运行逻辑。如果想要修改模板的其他部分，请先熟悉 Batch 的工作原理，方法是查看 Batch 文档并在几个 Batch 代码示例上进行尝试。

Run() 方法负责启动命令行、启动一个或多个进程、等待所有进程完成、保存结果，最后返回退出代码。Run() 方法可供实现任务的处理逻辑。任务处理器框架调用 Run() 方法；用户不需要自行调用。

Run() 实现具有以下项的访问权限：

- 任务参数，通过 `_parameters` 字段。
- 作业和任务 ID，通过 `_jobId` 和 `_taskId` 字段。
- 任务配置，通过 `_configuration` 字段。

**任务失败**

如果发生失败，可以引发异常以退出 Run() 方法，但这会导致顶层异常处理程序继续控制任务退出代码。如果需要控制退出代码以便分辨不同类型的失败，例如为了进行诊断或由于某些失败模式应终止作业，某些则不应该，则应该通过返回非零退出代码来退出 Run() 方法。这会成为任务退出代码。

### 任务处理器模板中的退出代码和异常
退出代码和异常提供了机制来确定程序的运行结果，可帮助找到任何程序执行问题。任务处理器模板实现本部分所述的退出代码和异常。

使用任务处理器模板实现的任务处理器任务返回三个可能的退出代码：

| 代码 | 说明 |
| --- | --- |
| [Process.ExitCode][process_exitcode] |任务处理器已运行完成。请注意，这并不表示调用的程序已成功，只表示任务处理器已成功调用程序并执行了所有后期处理步骤，而没有异常。退出代码的含义取决于所调用的程序，一般而言，退出代码 0 表示程序已成功，任何其他退出代码都表示程序失败。 |
| 1 |任务处理器任务失败，程序的“预期”部分有异常。异常已转换成 `TaskProcessorException` 与诊断信息，如有可能，还提供可解决失败的建议。 |
| 2 |任务处理器任务失败，发生“意外的”异常。异常已记录到标准输出，但任务处理器无法添加任何额外的诊断或补救信息。 |

> [AZURE.NOTE]
如果调用的程序使用退出代码 1 和 2 来指出特定失败模式，则使用退出代码 1 和 2 来代表任务处理器错误将造成模棱两可的状况。可以编辑 Program.cs 文件中的异常案例，将这些任务处理器错误代码更改为可区分的退出代码。
> 
> 

异常返回的所有信息已写入 stdout.txt 和 stderr.txt 文件。有关详细信息，请参阅 Batch 文档中的“Error Handling”（错误处理）。

### 客户端注意事项
**存储凭据**

如果任务处理器使用 Azure Blob 存储来保存输出，例如使用文件约定帮助器库，则它需要访问云存储帐户凭据或包含共享访问签名 (SAS) 的 Blob 容器 URL。模板支持通过通用环境变量来提供凭据。客户端可按如下所示传递存储凭据：

csharp

	job.CommonEnvironmentSettings = new [] {
	    new EnvironmentSetting("LINKED_STORAGE_ACCOUNT", "{storageAccountName}"),
	    new EnvironmentSetting("LINKED_STORAGE_KEY", "{storageAccountKey}"),
	};


然后，可以通过 `_configuration.StorageAccount` 属性在 TaskProcessor 类中使用存储帐户。

如果想要使用具有 SAS 的容器 URL，也可以通过作业的通用环境设置传递此 URL，但任务处理器模板目前未内置支持此 URL。

**存储设置**

建议客户端或作业管理器任务先创建任务所需的任何容器，再将任务添加到作业。如果使用具有 SAS 的容器 URL 就必须这样做，因为这样的 URL 并未包含创建容器的权限。即使传递的是存储帐户凭据仍建议这样做，因为它存储每一项必须在容器上调用 CloudBlobContainer.CreateIfNotExistsAsync 的任务。

## 传递参数和环境变量 <a name="pass-environment-settings"></a>

### 传递环境设置
客户端可以环境设置的形式将信息传递给作业管理器任务。然后，作业管理器任务可在生成作为计算作业一部分来运行的任务处理器任务时使用此信息。可以环境设置形式传递的信息示例如下：

- 存储帐户名称和帐户密钥
- Batch 帐户 URL
- Batch 帐户密钥

Batch 服务提供一个简单的机制在 [Microsoft.Azure.Batch.JobManagerTask][net_jobmanagertask] 中使用 `EnvironmentSettings` 属性将环境设置传递到作业管理器任务。

例如，若要获取 Batch 帐户的 `BatchClient` 实例，可以环境变量的形式从客户端代码传递 Batch 帐户的 URL 和共享密钥凭据。同样，若要访问链接到 Batch 帐户的存储帐户，可使用环境变量的形式传递存储帐户名和存储帐户密钥。

### 将参数传递到作业管理器模板
在许多情况下，最好将每个操作的参数传递到作业管理器任务，以便控制作业拆分进程或配置作业的任务。为此，可将名为 parameters.json 的 JSON 文件上载为作业管理器任务的资源文件。然后，参数就可以在作业管理器模板的 `JobSplitter._parameters` 字段中可用。

> [AZURE.NOTE]
内置的参数处理程序只支持字符串到字符串的字典。如果想要以参数值的形式传递复杂 JSON 值，需要以字符串形式传递并在作业拆分器中进行分析，或者修改框架的 `Configuration.GetJobParameters` 方法。
> 
> 

### 将参数传递给任务处理器模板
也可以使用任务处理器模板将参数传递到所实现的各个任务。就像使用作业管理器模板一样，任务处理器模板查找名为

parameters.json 的资源文件，如果找到，则将它加载为参数字典。有几个选项可用于将参数传递给任务处理器任务：

- 重复使用作业参数 JSON。这适用于唯一的参数都是作业范围的参数时（例如渲染高度和宽度）。为此，请于在作业拆分器中创建 CloudTask 时，从作业管理器任务的 ResourceFiles (`JobSplitter._jobManagerTask.ResourceFiles`) 将 parameters.json 资源文件对象的引用添加到 CloudTask 的 ResourceFiles 集合。
- 生成和上载任务特定的 parameters.json 文档作为作业拆分器执行的一部分，并在任务的资源文件集合中引用该 Blob。如果不同的任务有不同的参数，就必须这样做。以参数形式将帧索引传递到任务的 3D 渲染方案便是可能的示例。

> [AZURE.NOTE]
内置的参数处理程序只支持字符串到字符串的字典。如果想要以参数值的形式传递复杂 JSON 值，需要以字符串形式传递并在任务处理器中进行分析，或者修改框架的 `Configuration.GetTaskParameters` 方法。
> 
> 

## 后续步骤
### 将作业和任务输出保存到 Azure 存储
在开发 Batch 解决方案时的另一个有用工具是 [Azure Batch 文件约定][nuget_package]。在 Batch .NET 应用程序中使用此 .NET 类库（目前以预览版提供）可在 Azure 存储中轻松存储和检索任务输出。[Persist Azure Batch job and task output](/documentation/articles/batch-task-output/)（保存 Azure Batch 作业和任务输出）包含该库及其用法的完整介绍。

### Batch 论坛
MSDN 上的 [Azure Batch 论坛][forum]是探讨 Batch 服务以及咨询其相关问题的不错场所。欢迎前往浏览这些帮忙解决“棘手问题”的贴子，并发布你在构建 Batch 解决方案时遇到的问题。

[forum]: https://social.msdn.microsoft.com/forums/azure/zh-cn/home?forum=azurebatch
[net_jobmanagertask]: https://msdn.microsoft.com/zh-cn/library/azure/microsoft.azure.batch.jobmanagertask.aspx
[github_samples]: https://github.com/Azure/azure-batch-samples
[nuget_package]: https://www.nuget.org/packages/Microsoft.Azure.Batch.Conventions.Files
[process_exitcode]: https://msdn.microsoft.com/zh-cn/library/system.diagnostics.process.exitcode.aspx
[vs_gallery]: https://visualstudiogallery.msdn.microsoft.com/
[vs_gallery_templates]: https://go.microsoft.com/fwlink/?linkid=820714
[vs_find_use_ext]: https://msdn.microsoft.com/zh-cn/library/dd293638.aspx

[diagram01]: ./media/batch-visual-studio-templates/diagram01.png
[solution_explorer01]: ./media/batch-visual-studio-templates/solution_explorer01.png
[solution_explorer02]: ./media/batch-visual-studio-templates/solution_explorer02.png

<!---HONumber=Mooncake_0213_2017-->
<!---Update_Description: wording update -->