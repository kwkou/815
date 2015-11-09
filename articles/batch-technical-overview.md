<properties
	pageTitle="Azure  批处理( Batch )技术概述"
	description="了解 Azure  批处理( Batch )服务的概念、工作流和方案"
	services="batch"
	documentationCenter=""
	authors="dlepow"
	manager="timlt"
	editor=""/>

<tags
	ms.service="batch"
	ms.date="07/13/2015"
	wacn.date="10/30/2015"/>


#Azure  批处理( Batch )技术概述
Azure 批处理( Batch )可帮助你在云中有效运行大规模并行和高性能计算 (HPC) 应用程序。它是一个平台服务，提供作业计划，并可以自动缩放受控的虚拟机 (VM) 集合以运行作业。通过使用批处理( Batch )服务，你可以将批处理( Batch )工作负荷配置为按需或按计划在 Azure 中运行，而无需担心配置和管理 HPC 群集、VM 和作业计划程序的复杂性。

>[AZURE.NOTE]若要使用批处理( Batch )，你需要一个 Azure 帐户。如果你没有帐户，只需花费几分钟就能创建一个免费试用帐户。有关详细信息，请参阅[创建 Azure 帐户](/documentation/articles/php-create-account)。


## 用例

 批处理( Batch )利用云的弹性和规模，帮助你进行* 批处理( Batch )*或* 批处理( Batch )计算* - 运行大量类似的任务以获得某个所需的结果。命令行程序或脚本接受一组数据文件作为输入，以一系列的任务来处理数据，然后生成一组输出文件。输出文件可能是最终结果，也可能是较大工作流中的中间步骤。

在按计划或按需处理、转换和分析大量数据的组织中， 批处理( Batch )计算是一种常见的模式。其中包括生命周期终止处理，例如银行的每日风险报表，或必须按计划完成的工资单。它还包括大型商业、科学和工程应用程序，通常需要计算群集或网格的工具和资源。应用范围包括传统的 HPC 应用，例如流体动力学仿真以及专业化工作负荷（从数码内容创作、金融服务到生命科学研究等各种领域）。

 批处理( Batch )很适合处理本质上并行（有时称为“易并行”）的应用程序或工作负载，而这些本身就适合在多部计算机上以并行任务运行，例如  批处理( Batch )服务管理的计算 VM。请参阅图 1。

![并行任务][parallel]

**图 1.在多台计算机上运行的并行任务**

示例包括：

* 金融风险建模
* 图像呈现和图像处理
* 媒体编码和转码
* 基因序列分析
* 软件测试

你还可以使用批处理( Batch )来执行并行计算（最后加上归纳步骤），以及其他更复杂的并行工作负荷。

>[AZURE.NOTE]目前，只能在批处理( Batch )服务上运行 Windows Server 工作负荷。此外，批处理( Batch )目前不支持消息传递接口 (MPI) 应用程序。

## 开发人员方案

  批处理( Batch )支持不同的开发人员方案，可帮助你利用批处理( Batch )服务来配置和运行大规模并行工作负荷。这些方案利用 API 创建和管理 VM 池（计算节点），以及计划对 VM 运行的作业和任务。有关批处理( Batch )概念的详细信息，请参阅 [Azure 批处理( Batch ) API 基础知识](/documentation/articles/batch-api-basics)。

以下部分列出了典型的批处理( Batch )开发人员方案。

### 向外扩展并行工作负荷

使用批处理( Batch ) API 向外扩展本质并行的任务，例如，对包含多达数千个计算核心的池执行图像渲染。你可以自动计划大型计算作业，并相应地扩大和缩小计算 VM 池来运行这些作业，而不需设置计算群集或编写代码来排队和计划作业，并移动所需的输入和输出数据。你可以编写客户端应用和前端来按需、按计划或者在执行工具管理的大型工作流期间中运行作业和任务<!-- such as [Azure Data Factory](https://azure.microsoft.com/documentation/services/data-factory/)-->。

图 2 显示了将应用程序提交到要在其中分发以进行处理的池的简化工作流。

![工作项工作流][work_item_workflow]

**图 2.在批处理( Batch )服务上向外扩展并行工作负荷**

1.	将作业所需的输入文件（例如源数据或图像）上载到 Azure 存储帐户。这些文件必须在存储帐户中，批处理( Batch )服务才能够访问它们。任务运行时，  批处理( Batch )服务会将文件加载到计算节点。
2.	将相关二进制文件上载到存储帐户。二进制文件包括由任务运行的程序和相关程序集。这些文件也必须从存储访问，并已加载到计算节点。
3.	创建计算节点的池，指定属性，例如其 VM 大小和其运行的操作系统。你还可以定义如何增加或减少池中的节点数目，以响应任务负荷。任务运行时，系统从这个池分配节点给它。
4.	定义要对池运行的作业。
5.	将任务添加到作业。每个任务都使用所上载的程序处理已上载的文件中的信息。
6.	运行该应用程序并监视输出结果。


### 启用云功能的计算密集型应用

你可以使用预览版 批处理（ Batch ） Apps API 来包装现有的应用程序，使其在  批处理( Batch )在后台管理的计算节点池上作为服务运行。该应用程序可以是目前在客户端工作站或计算群集上运行的应用程序。你可以开发服务，让用户将高峰工作卸载到云，或完全在云中运行其任务。Batch Apps 框架将处理输入和输出文件的移动，将作业拆分成任务，执行作业和任务处理，以及保存数据。

>[AZURE.IMPORTANT]Azure 只以预览版形式提供批处理（ Batch ） Apps API。你只应该针对测试或概念证明项目开发它。在将来的服务版本中，关键的 批处理（ Batch ） Apps 功能将集成到批处理( Batch ) API。

图 3 显示的基本工作流使用 Batch Apps API 发布应用程序，然后允许用户将作业提交到应用程序。

![应用程序发布工作流][app_pub_workflow]

**图 3.使用 批处理( Batch ) Apps 发布和运行应用程序的工作流**

1.	准备**应用程序映像**，这是一个 zip 文件，包含现有应用程序可执行文件及其所需的任何支持文件。这些可能是在传统服务器场或群集中运行的相同可执行文件。
2.	创建**云程序集**的 zip 文件，该程序集会调用任务负载并将其分派给批处理( Batch )服务。这包含两个组件：

	a.**作业拆分器** - 将作业分解成可独立处理的任务。例如，在动画案例中，作业拆分器将使用影片渲染作业，并将影片分割成单个帧。

	b.**任务处理器** – 调用给定任务的应用程序可执行文件。例如，在动画案例中，任务处理器将调用一个渲染程序，以渲染任务指定的单个帧。

3.	使用批处理( Batch ) Apps API 或开发人员工具将前两个步骤中准备的 zip 文件上载到 Azure 存储帐户。这些文件必须在存储帐户中，  批处理( Batch )服务才能够访问它们。此任务通常由服务管理员为每个应用程序执行一次。
4.	提供一种方式以将作业提交给 Azure 中启用的应用程序服务。这可能是应用程序 UI 中的插件、Web 门户，或后端系统中的无人值守服务。

	运行作业：

	a.上载特定于用户作业的输入文件（例如源数据或图像）。这些文件必须在存储帐户中，  批处理( Batch )服务才能够访问它们。

	b.连同所需的参数和文件列表一起提交作业。

	c.使用 API 或 Batch Apps 门户监视作业进度。



## <a id="BKMK_Account">批处理( Batch )帐户</a>
你需要创建一个或多个唯一的** 批处理( Batch )帐户**来使用批处理( Batch )服务以及进行开发。对批处理( Batch )服务发出的所有请求必须使用帐户的名称及其访问密钥进行身份验证。

可以使用 Azure 预览门户或[ 批处理( Batch ) PowerShell cmdlet](/documentation/articles/batch-powershell-cmdlets-get-started) 中创建批处理( Batch )帐户和管理该帐户的访问密钥。

在门户中创建批处理( Batch )帐户：

在门户中创建批处理( Batch )帐户：

<!-- 登录到 [Azure 门户](https://manage.windowsazure.cn)。

  单击“新建”>“计算”>“应用商店”>“所有”，然后在搜索框中输入“批处理( Batch )”。

	![应用商店中的 批处理( Batch )][marketplace_portal]

在搜索结果中单击“批处理( Batch )服务”，然后单击“创建”。-->

1. 登录到 [Azure 门户](https://manage.windowsazure.cn)。 
2. 单击“新建”>“计算”>“ 批处理服务”>“快速创建”，输入以下信息：
    a.在“帐户名称”中，输入要在批处理( Batch )帐户 URL 中使用的唯一名称。

	>[AZURE.NOTE] 批处理( Batch )帐户名在 Azure 中必须是唯一的，长度为 3 到 24 个字符，并且只能包含小写字母和数字。

	b.在“区域”中，选择批处理( Batch )已上市的 Azure 区域。


 3. 单击“创建账户”完成帐户创建。


创建帐户之后，你可以在“批处理服务”中找到它，并将它用于管理访问密钥和其他设置。



## 其他资源

* [适用于 .NET 的 Azure  批处理( Batch )库入门](/documentation/articles/batch-dotnet-get-started)
* [Azure 批处理( Batch )开发库和工具](/documentation/articles/batch-development-libraries-tools)
* [Azure 批处理( Batch ) REST API 参考](https://msdn.microsoft.com/zh-cn/library/azure/dn820158.aspx)
* [Azure 批处理( Batch ) Apps REST API 参考](https://msdn.microsoft.com/zh-cn/library/azure/dn820126.aspx)

[parallel]: ./media/batch-technical-overview/parallel.png
[marketplace_portal]: ./media/batch-technical-overview/marketplace_batch.PNG
[account_portal]: ./media/batch-technical-overview/batch_acct_portal.png
[account_keys]: ./media/batch-technical-overview/account_keys.PNG
[work_item_workflow]: ./media/batch-technical-overview/work_item_workflow.png
[app_pub_workflow]: ./media/batch-technical-overview/app_pub_workflow.png

<!---HONumber=66-->