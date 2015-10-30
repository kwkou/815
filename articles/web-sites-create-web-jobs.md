<properties 
	pageTitle="使用 WebJobs 运行后台任务" 
	description="了解如何在 Azure Web Apps 中运行后台任务。" 
	services="app-service\web" 
	documentationCenter="" 
	authors="tdykstra" 
	manager="wpickett" 
	editor="jimbe"/>

<tags 
	ms.service="app-service-web" 
	ms.date="09/16/2015" 
	wacn.date="10/22/2015"/>
# 使用 WebJobs 运行后台任务

## 概述
你可以使用三种方式在网站上运行程序或脚本，即：按需、连续或按计划。使用 WebJobs 无需支付额外的费用。

本文说明如何使用 Azure 管理门户来部署 Web 作业。有关如何使用 Visual Studio 或连续交付过程进行部署的信息，请参阅[如何将 Azure WebJobs 部署到 Azure 网站](/documentation/articles/websites-dotnet-deploy-webjobs)。

Azure WebJobs SDK 简化了许多 Web 作业编程任务。有关详细信息，请参阅[什么是 WebJobs SDK](/documentation/articles/websites-dotnet-webjobs-sdk)。

## 目录 ##
- [可接受的脚本文件类型](#acceptablefiles)
- [创建按需运行任务](#CreateOnDemand)
- [创建连续运行任务](#CreateContinuous)
- [创建计划任务](#CreateScheduled)
	- [计划作业和 Azure 计划程序](#Scheduler)
- [查看作业历史记录](#ViewJobHistory)
- [说明](#WHPNotes)
- [后续步骤](#NextSteps)

## <a name="acceptablefiles"></a>可接受的脚本或程序文件类型

接受以下文件类型：

* .cmd、.bat、.exe（使用 Windows cmd）
* .ps1（使用 Powershell）
* .sh（使用 Bash）
* .php（使用 php）
* .py（使用 Python）
* .js（使用 Node）
* .jar（使用 java）

## <a name="CreateOnDemand"></a>在门户中创建按需 Web 作业

1. 在“WebJobs”页的命令栏中，单击“添加”。此时将显示“新作业”对话框。
	
	![按需运行任务][OnDemandWebJob]
	
2. 在“名称”下，提供任务的名称。名称必须以字母或数字开头，并且不能包含除“-”和“\_”以外的任何特殊字符。
	
3. 在“内容(Zip 文件 – 最大 100MB)”框中，浏览到包含你的脚本的 zip 文件。Zip 文件应包含可执行文件 (.exe .cmd .bat .sh .php .py .js)，以及运行程序或脚本所需的任何支持文件。
	
4. 在“如何运行”框中，选择“按需运行”。
	
5. 选中对话框右下角的复选标记，以将该脚本上载到你的网站。为任务指定的名称将显示在列表中：
	
	![任务列表][WebJobsList]
	
6. 若要运行脚本，请在列表中选择其名称，然后在门户页面底部的命令栏中单击“运行一次”。
	
	![运行一次][RunOnce]

## <a name="CreateContinuous"></a>创建连续运行任务

1. 若要创建连续执行的任务，请按照与创建运行一次的任务相同的步骤进行操作，但在“如何运行”框中，请选择“连续运行”。
	
	![新的连续运行任务][NewContinuousJob]
	
2. 若要启动或停止连续运行的任务，请在列表中选择任务，然后在命令栏中单击“启动”或“停止”。

> [AZURE.NOTE]如果你的网站在多个实例上运行，则连续运行的任务将在所有实例上运行。按需运行任务和计划任务在 Windows Azure 针对负载平衡所选择的单个实例上运行。

> [AZURE.NOTE]对于连续运行任务，建议你在“配置”页上为网站启用“始终打开”。Always On 功能（在基本和标准模式下可用）可防止网站被卸载，即使网站已空闲一段时间也是如此。如果你的网站始终处于加载状态，则连续运行的任务可能会更可靠地运行。

## <a name="CreateScheduled"></a>创建计划任务

1. 若要创建计划任务，请执行前述相同步骤，但在“如何运行”框中，请选择“按计划运行”。
	
	![新的计划作业][NewScheduledJob]
	
2. 为作业选择“计划程序区域”，然后单击对话框右下角的箭头进入下一个屏幕。

3. 在“创建作业”对话框中，选择想要的“定期”类型：“一次作业”或“重复作业”。
	
	![安排定期计划][SchdRecurrence]
	
4. 此外，选择“开始”时间：“现在”或“特定时间”。
	
	![计划开始时间][SchdStart]
	
5. 如果你想要在特定时间开始，请在“开始时间”下选择开始时间值。
	
	![计划在特定时间启动][SchdStartOn]
	
6. 如果你选择重复作业，请使用“重复间隔”选项指定重复执行的频率，并使用“结束时间”选项指定结束时间。
	
	![安排定期计划][SchdRecurEvery]
	
7. 如果选择“周”，则可以选中“按特定计划”框，并指定你想要在星期几运行作业。
	
	![计划每周的每一天][SchdWeeksOnParticular]
	
8. 如果选择“月”并选中“按特定计划”框，则可以设置在每月的特定编号“日”运行作业。
	
	![计划每月的特定日期][SchdMonthsOnPartDays]
	
9. 如果选择“工作日”，则可以选择想要在一个月中的哪个或哪些工作日运行作业。
	
	![计划一个月中的特定工作日][SchdMonthsOnPartWeekDays]
	
10. 最后，还可以使用“次数”选项，选择想要在一个月中的哪一周（第一周、第二周、第三周等）的指定工作日运行作业。
	
	![计划一个月中的特定周的特定工作日][SchdMonthsOnPartWeekDaysOccurences]
	
11. 创建一个或多个作业后，其名称将与其状态、计划类型和其他信息一起显示在“Web 作业”选项卡中。将保留最近 30 个 WebJobs 的历史信息。
	
	![作业列表][WebJobsListWithSeveralJobs]
	
### <a name="Scheduler"></a>计划作业和 Azure 计划程序

可以在 Azure 计划程序[门户(http://manage.windowsazure.cn)中进一步配置计划作业。

1.	在“Web 作业”页上，单击作业的“计划”链接，以导航到 Azure 计划程序门户页。 
	
	![链接到 Azure 计划程序][LinkToScheduler]
	
2. 在计划程序页上，单击该作业。
	
	![计划程序门户页上的作业][SchedulerPortal]
	
3. 将打开“作业操作”页，从中可以进一步配置作业。
	
	![作业操作 PageInScheduler][JobActionPageInScheduler]
	
## <a name="ViewJobHistory"></a>查看作业历史记录

1. 若要查看作业的执行历史记录（包括使用 WebJobs SDK 创建的作业），请单击“Web 作业”边栏选项卡的“日志”列下方的相应链接。（如果需要，你可以使用剪贴板图标将日志文件页的 URL 复制到剪贴板中）
	
	![日志链接](./media/web-sites-create-web-jobs/wjbladelogslink.png)
		
2. 单击链接将打开 WebJob 的详细信息页。此页显示了命令运行的名称、它运行的最后时间及其成功或失败状态。在“最近的作业运行”下，单击某个时间以查看更多详细信息。
	
	![WebJobDetails][WebJobDetails]
	
3. 此时将显示“Web 作业运行详细信息”页。单击“切换输出”以查看日志内容的文本。输出日志使用文本格式。
	
	![Web 作业运行详细信息][WebJobRunDetails]
	
4. 若要在单独的浏览器窗口中查看输出文本，请单击“下载”链接。若要下载文本本身，请右键单击该链接，并使用你的浏览器选项来保存该文件的内容。
	
	![下载日志输出][DownloadLogOutput]
	
5. 页面顶部的“Web 作业”链接提供了一种简便方法，可用于访问历史记录仪表板上的 Web 作业列表。
	
	![链接到 Web 作业列表][WebJobsLinkToDashboardList]
	
	![历史记录仪表板中的 WebJobs 列表][WebJobsListInJobsDashboard]
	
	单击其中一条链接可将你带到所选作业的 Web 作业详细信息页。


## <a name="WHPNotes"></a>说明
	
- 如果没有对 scm（部署）站点发出任何请求，并且未在 Azure 中打开 Web 应用的门户，则免费模式下的 Web Apps 可能会在 20 分钟之后超时。对实际网站发出的请求不会对此进行重置。
- 需要对连续作业的代码进行编写，才能使其在无穷循环中运行。
- 仅当 Web 应用启动后连续作业才能连续运行。
- 基本和标准模式提供了“始终打开”功能，启用该功能可防止 Web Apps 进入空闲状态。
- 仅可以调试连续运行 WebJobs。不支持按计划或按需调试 WebJobs。

[PSonWebJobs]: http://blogs.msdn.com/b/nicktrog/archive/2014/01/22/running-powershell-web-jobs-on-azure-websites.aspx

[OnDemandWebJob]: ./media/web-sites-create-web-jobs/01aOnDemandWebJob.png
[WebJobsList]: ./media/web-sites-create-web-jobs/02aWebJobsList.png
[NewContinuousJob]: ./media/web-sites-create-web-jobs/03aNewContinuousJob.png
[NewScheduledJob]: ./media/web-sites-create-web-jobs/04aNewScheduledJob.png
[SchdRecurrence]: ./media/web-sites-create-web-jobs/05SchdRecurrence.png
[SchdStart]: ./media/web-sites-create-web-jobs/06SchdStart.png
[SchdStartOn]: ./media/web-sites-create-web-jobs/07SchdStartOn.png
[SchdRecurEvery]: ./media/web-sites-create-web-jobs/08SchdRecurEvery.png
[SchdWeeksOnParticular]: ./media/web-sites-create-web-jobs/09SchdWeeksOnParticular.png
[SchdMonthsOnPartDays]: ./media/web-sites-create-web-jobs/10SchdMonthsOnPartDays.png
[SchdMonthsOnPartWeekDays]: ./media/web-sites-create-web-jobs/11SchdMonthsOnPartWeekDays.png
[SchdMonthsOnPartWeekDaysOccurences]: ./media/web-sites-create-web-jobs/12SchdMonthsOnPartWeekDaysOccurences.png
[RunOnce]: ./media/web-sites-create-web-jobs/13RunOnce.png
[WebJobsListWithSeveralJobs]: ./media/web-sites-create-web-jobs/13WebJobsListWithSeveralJobs.png
[WebJobLogs]: ./media/web-sites-create-web-jobs/14WebJobLogs.png
[WebJobDetails]: ./media/web-sites-create-web-jobs/15WebJobDetails.png
[WebJobRunDetails]: ./media/web-sites-create-web-jobs/16WebJobRunDetails.png
[DownloadLogOutput]: ./media/web-sites-create-web-jobs/17DownloadLogOutput.png
[WebJobsLinkToDashboardList]: ./media/web-sites-create-web-jobs/18WebJobsLinkToDashboardList.png
[WebJobsListInJobsDashboard]: ./media/web-sites-create-web-jobs/19WebJobsListInJobsDashboard.png
[LinkToScheduler]: ./media/web-sites-create-web-jobs/31LinkToScheduler.png
[SchedulerPortal]: ./media/web-sites-create-web-jobs/32SchedulerPortal.png
[JobActionPageInScheduler]: ./media/web-sites-create-web-jobs/33JobActionPageInScheduler.png

<!---HONumber=74-->