<properties linkid="web-sites-create-web-jobs" urlDisplayName="Use WebJobs to run background tasks in Microsoft Azure Web Sites" pageTitle="使用 WebJobs 在 Microsoft Azure 网站中运行后台任务" metaKeywords="Microsoft Azure Web Sites, Web Jobs, background tasks" description="了解如何在 Microsoft Azure 网站中运行后台任务。" metaCanonical="" services="web-sites" documentationCenter="" title="使用 WebJobs 在 Microsoft Azure 网站中运行后台任务" authors="timamm"  solutions="" writer="timamm" manager="paulettm" editor="mollybos"  />

# 使用 WebJobs 在 Microsoft Azure 网站中运行后台任务

Microsoft Azure 网站允许您使用以下三种方式之一在网站中运行程序或脚本：按需运行、连续运行或按计划运行。除非您想要启用本文后面介绍的 Always On 功能，否则使用 Microsoft Azure WebJobs 无需额外付费。

## 目录

-   [可接受的脚本文件类型][可接受的脚本文件类型]
-   [创建按需运行任务][创建按需运行任务]
-   [创建连续运行任务][创建连续运行任务]
-   [创建计划任务][创建计划任务]

    -   [计划作业和 Azure 计划程序][计划作业和 Azure 计划程序]
-   [查看作业历史记录][查看作业历史记录]
-   [说明][说明]
-   [后续步骤][后续步骤]

    -   [使用 Microsoft Azure WebJobs SDK 提高效率][使用 Microsoft Azure WebJobs SDK 提高效率]
    -   [部署的替代方法][部署的替代方法]
    -   [其他资源][其他资源]

<a name="acceptablefiles"></a>

## 可接受的脚本文件类型

接受以下文件类型作为可运行的脚本：

.cmd、.bat、.exe（使用 Windows cmd）

.ps1（使用 Powershell）

.sh（使用 Bash）

.php（使用 php）

.py（使用 Python）

.js（使用 Node）

<a name="CreateOnDemand"></a>

## 创建按需运行任务

1.  在“WebJobs”页的命令栏中，单击“添加”。此时将显示“新作业”对话框。

    ![按需运行任务][按需运行任务]

2.  在“名称”下，提供任务的名称。名称必须以字母或数字开头，并且不能包含除“-”和“\_”以外的任何特殊字符。

3.  在“内容(Zip 文件 – 最大 100 MB)”框中，浏览到包含您的脚本的 zip 文件。Zip 文件应包含可执行文件 (.exe .cmd .bat .sh .php .py .js)，以及运行程序或脚本所需的任何支持文件。

4.  在“如何运行”框中，选择“按需运行”。

5.  选中对话框右下角的复选标记，以将该脚本上载到您的网站。为任务指定的名称将显示在列表中：

    ![任务列表][任务列表]

6.  若要运行该脚本，请在列表中选择其名称，然后在门户页面底部的命令栏中单击“运行一次”。

    ![运行一次][运行一次]

<a name="CreateContinuous"></a>

## 创建连续运行任务

1.  若要创建连续执行的任务，请按照与创建运行一次的任务相同的步骤进行操作，但在“如何运行”框中，请选择“连续运行”。

    ![新的连续运行任务][新的连续运行任务]

2.  若要启动或停止连续运行的任务，请在列表中选择任务，然后在命令栏中单击“启动”或“停止”。

> [WACOM.NOTE] 如果您的网站在多个实例上运行，则连续运行的任务将在所有实例上运行。按需运行任务和计划任务在 Microsoft Azure 针对负载平衡所选择的单个实例上运行。

> [WACOM.NOTE]
> 对于连续运行任务，建议您在“配置”页上为网站启用“Always On”。Always On 功能（在基本和标准模式下可用）可防止网站被卸载，即使网站已空闲一段时间也是如此。如果您的网站始终处于加载状态，则连续运行的任务可能会更可靠地运行。

<a name="CreateScheduled"></a>

## 创建计划任务

1.  若要创建计划任务，请执行前述相同步骤，但在“如何运行”框中，请选择“按计划运行”。

    ![新的计划作业][新的计划作业]

2.  为作业选择“计划程序区域”，然后单击对话框右下角的箭头进入下一个屏幕。

3.  在“创建作业”对话框中，选择想要的“定期”类型：“一次性作业”或“定期作业”。

    ![安排定期计划][安排定期计划]

4.  还需选择“开始”时间：“立即”或“在特定时间”。

    ![计划开始时间][计划开始时间]

5.  如果您想要在特定时间启动，请在“开始时间”下选择开始时间值。

    ![计划在特定时间启动][计划在特定时间启动]

6.  如果您选择定期作业，请使用“重复间隔”选项指定重复执行的频率，并使用“结束时间”选项指定结束时间。

    ![安排定期计划][1]

7.  如果选择“周”，则可以选中“按特定计划”框，并指定您想要在星期几运行作业。

    ![计划每周的每一天][计划每周的每一天]

8.  如果选择“月”并选中“按特定计划”框，则可以设置在每月的特定编号“日”运行作业。

    ![计划每月的特定日期][计划每月的特定日期]

9.  如果选择“工作日”，则可以选择想要在一个月中的哪个或哪些工作日运行作业。

    ![计划一个月中的特定工作日][计划一个月中的特定工作日]

10. 最后，还可以使用“次数”选项，选择想要在一个月中的哪一周（第一周、第二周、第三周等）的指定工作日运行作业。

    ![计划一个月中的特定周的特定工作日][计划一个月中的特定周的特定工作日]

11. 创建一个或多个作业后，其名称将与其状态、计划类型和其他信息一起显示在“WebJobs”选项卡中。将保留最近 30 个任务的历史信息。

    ![作业列表][作业列表]

<a name="Scheduler"></a>

### 计划作业和 Azure 计划程序

可以在 Azure 计划程序门户中进一步配置计划作业。

1.  在 WebJobs 页上，单击作业的“计划”链接以导航到 Azure 计划程序门户页。

    ![链接到 Azure 计划程序][链接到 Azure 计划程序]

2.  在计划程序页上，单击该作业。

    ![计划程序门户页上的作业][计划程序门户页上的作业]

3.  将打开“作业操作”页，从中可以进一步配置该作业。

    ![作业操作 PageInScheduler][作业操作 PageInScheduler]

<!-- ================ ViewJobHistory ================= -->

<a name="ViewJobHistory"></a>

## 查看作业历史记录

1.  若要查看作业的执行历史记录（包括使用 WebJobs SDK 创建的作业），请单击“日志”列下方的该作业的相应链接。（如果需要，您可以使用剪贴板图标将日志文件页的 URL 复制到剪贴板中）

    ![日志链接][日志链接]

2.  单击链接将打开该任务的 Web 作业详细信息页。此页显示了命令运行的名称、它运行的最后时间及其成功或失败状态。在“最近的作业运行”下，单击某个时间以查看更多详细信息。

    ![WebJobDetails][WebJobDetails]

3.  将显示“WebJob 运行详细信息”页。单击“切换输出”以查看日志内容的文本。输出日志使用文本格式。

    ![Web 作业运行详细信息][Web 作业运行详细信息]

4.  若要在单独的浏览器窗口中查看输出文本，请单击“下载”链接。若要下载文本本身，请右键单击该链接，并使用您的浏览器选项来保存该文件的内容。

    ![下载日志输出][下载日志输出]

5.  页面顶部的“WebJobs”链接提供了用于访问历史记录仪表板上的 Web 作业列表的简便方法。

    ![链接到 Web 作业列表][链接到 Web 作业列表]

    ![历史记录仪表板中的作业列表][历史记录仪表板中的作业列表]

    单击其中一条链接可将您带到所选作业的 WebJob 详细信息页。

<a name="WHPNotes"></a>

## 说明

-   从 2014 年 3 月开始，如果没有对 scm（部署）站点发出任何请求，并且未在 Azure 中打开网站的门户，则免费模式下的网站可能会在 20 分钟之后超时。对实际网站发出的请求不会对此进行重置。

-   需要对连续作业的代码进行编写，才能使其在无穷循环中运行。

-   仅当站点启动后连续作业才能连续运行。

-   基本和标准模式提供了 Always On 功能，启用该功能可防止站点进入空闲状态。

<a name="NextSteps"></a>

## 后续步骤

<a name="WebJobsSDK"></a>

### 使用 Microsoft Azure WebJobs SDK 提高效率

Microsoft Azure WebJobs SDK 简化了将后台处理添加到 Microsoft Azure 网站中的任务。SDK 集成了 Microsoft Azure 存储空间，当项目添加到队列、Blob 或表中时，将触发程序中的函数。仪表板（现已集成到 Azure 门户中）为使用 SDK 编写的程序提供了丰富的监视和诊断功能。监视和诊断功能已内置到 SDK 中并且不需要您在程序中添加任何特殊代码。

有关详细信息，请参阅教程 [Microsoft Azure WebJobs SDK 入门][Microsoft Azure WebJobs SDK 入门]。本教程提供 WebJobs SDK 的功能概述，并指导您完成创建和运行简单的 Hello World 后台进程。

若要查看使用 Microsoft Azure WebJobs SDK 创建的示例命令行应用的演练，请参阅 [Windows Azure WebJobs 简介][Windows Azure WebJobs 简介]。

<a name="AlternateDeployments"></a>

### 部署的替代方法

如果您不希望使用 WebJobs 门户页上载脚本，则可以使用 FTP、git 或 Web 部署。有关详细信息，请参阅[如何部署 Windows Azure WebJobs][如何部署 Windows Azure WebJobs] 和[使用 WebJobs 通过 Git 将 .NET 控制台应用部署到 Azure 中][使用 WebJobs 通过 Git 将 .NET 控制台应用部署到 Azure 中]。

<a name="AdditionalResources"></a>

### 其他资源

-   有关 WebJobs 功能的带批注的链接列表，请参阅[使用 Windows Azure 网站的 WebJobs 功能][使用 Windows Azure 网站的 WebJobs 功能]。

-   WebJobs 相关视频：

    [Azure WebJobs 101 – Jamie Espinosa 介绍基本 WebJobs][Azure WebJobs 101 – Jamie Espinosa 介绍基本 WebJobs]

    [Azure WebJobs 102 – Jamie Espinosa 介绍计划 WebJobs 和 WebJobs 仪表板][Azure WebJobs 102 – Jamie Espinosa 介绍计划 WebJobs 和 WebJobs 仪表板]

    [Azure Scheduler 101 – Kevin Lam 介绍如何安排计划][Azure Scheduler 101 – Kevin Lam 介绍如何安排计划]

### 入门

若要开始使用 Azure，请参阅 [Microsoft Azure 免费试用版][Microsoft Azure 免费试用版]。

<!-- LINKS --> <!-- IMAGES -->

  [可接受的脚本文件类型]: #acceptablefiles
  [创建按需运行任务]: #CreateOnDemand
  [创建连续运行任务]: #CreateContinuous
  [创建计划任务]: #CreateScheduled
  [计划作业和 Azure 计划程序]: #Scheduler
  [查看作业历史记录]: #ViewJobHistory
  [说明]: #WHPNotes
  [后续步骤]: #NextSteps
  [使用 Microsoft Azure WebJobs SDK 提高效率]: #WebJobsSDK
  [部署的替代方法]: #AlternateDeployments
  [其他资源]: #AdditionalResources
  [按需运行任务]: ./media/web-sites-create-web-jobs/01aOnDemandWebJob.png
  [任务列表]: ./media/web-sites-create-web-jobs/02aWebJobsList.png
  [运行一次]: ./media/web-sites-create-web-jobs/13RunOnce.png
  [新的连续运行任务]: ./media/web-sites-create-web-jobs/03aNewContinuousJob.png
  [新的计划作业]: ./media/web-sites-create-web-jobs/04aNewScheduledJob.png
  [安排定期计划]: ./media/web-sites-create-web-jobs/05SchdRecurrence.png
  [计划开始时间]: ./media/web-sites-create-web-jobs/06SchdStart.png
  [计划在特定时间启动]: ./media/web-sites-create-web-jobs/07SchdStartOn.png
  [1]: ./media/web-sites-create-web-jobs/08SchdRecurEvery.png
  [计划每周的每一天]: ./media/web-sites-create-web-jobs/09SchdWeeksOnParticular.png
  [计划每月的特定日期]: ./media/web-sites-create-web-jobs/10SchdMonthsOnPartDays.png
  [计划一个月中的特定工作日]: ./media/web-sites-create-web-jobs/11SchdMonthsOnPartWeekDays.png
  [计划一个月中的特定周的特定工作日]: ./media/web-sites-create-web-jobs/12SchdMonthsOnPartWeekDaysOccurences.png
  [作业列表]: ./media/web-sites-create-web-jobs/13WebJobsListWithSeveralJobs.png
  [链接到 Azure 计划程序]: ./media/web-sites-create-web-jobs/31LinkToScheduler.png
  [计划程序门户页上的作业]: ./media/web-sites-create-web-jobs/32SchedulerPortal.png
  [作业操作 PageInScheduler]: ./media/web-sites-create-web-jobs/33JobActionPageInScheduler.png
  [日志链接]: ./media/web-sites-create-web-jobs/14WebJobLogs.png
  [WebJobDetails]: ./media/web-sites-create-web-jobs/15WebJobDetails.png
  [Web 作业运行详细信息]: ./media/web-sites-create-web-jobs/16WebJobRunDetails.png
  [下载日志输出]: ./media/web-sites-create-web-jobs/17DownloadLogOutput.png
  [链接到 Web 作业列表]: ./media/web-sites-create-web-jobs/18WebJobsLinkToDashboardList.png
  [历史记录仪表板中的作业列表]: ./media/web-sites-create-web-jobs/19WebJobsListInJobsDashboard.png
  [Microsoft Azure WebJobs SDK 入门]: http://asp.net/aspnet/overview/developing-apps-with-windows-azure/getting-started-with-windows-azure-webjobs
  [Windows Azure WebJobs 简介]: http://www.hanselman.com/blog/IntroducingWindowsAzureWebJobs.aspx
  [如何部署 Windows Azure WebJobs]: http://blog.amitapple.com/post/74215124623/deploy-azure-webjobs
  [使用 WebJobs 通过 Git 将 .NET 控制台应用部署到 Azure 中]: http://blog.amitapple.com/post/73574681678/git-deploy-console-app
  [使用 Windows Azure 网站的 WebJobs 功能]: http://go.microsoft.com/fwlink/?LinkId=390226
  [Azure WebJobs 101 – Jamie Espinosa 介绍基本 WebJobs]: http://www.windowsazure.cn/zh-cn/documentation/videos/azure-webjobs-basics/
  [Azure WebJobs 102 – Jamie Espinosa 介绍计划 WebJobs 和 WebJobs 仪表板]: http://www.windowsazure.cn/zh-cn/documentation/videos/azure-webjobs-schedule-and-dashboard/
  [Azure Scheduler 101 – Kevin Lam 介绍如何安排计划]: http://www.windowsazure.cn/zh-cn/documentation/videos/azure-scheduler-how-to/
  [Microsoft Azure 免费试用版]: http://azure.microsoft.cn/zh-cn/pricing/free-trial/
