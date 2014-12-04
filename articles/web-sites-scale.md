<properties title="如何缩放网站" pageTitle="如何缩放网站" description="必填" metaKeywords="scaling Azure web sites" services="web-sites" solutions="web" documentationCenter="" authors="timamm" manager="paulettm" editor="mollybos" videoId="" scriptId="" />

# 如何缩放网站

为提高您的网站在 Microsoft Azure 上的性能和吞吐量，您可以使用 Azure 管理门户将 Web 托管计划模式从“免费”缩放为“共享”、“基本”或“标准”。

在 Azure 网站上向上缩放包括两个相关操作：将 Web 托管计划模式更改为更高级别的服务，以及在切换到更高级别的服务之后配置某些设置。本文将介绍这两个主题。更高的服务层（如“标准”模式）在确定如何使用 Azure 上的资源方面可提供更高的可靠性和灵活性。

可在管理门户的“缩放”选项卡中轻松地更改模式并进行配置。您可以根据需要向上或向下缩放。这些更改只需数秒即可生效，可影响 Web 托管计划中的所有网站。无需更改代码或重新部署应用程序。

有关 Web 托管计划的信息，请参阅[什么是 Web 托管计划？][什么是 Web 托管计划？]和 [Azure 网站 Web 托管计划深入概述][Azure 网站 Web 托管计划深入概述]。有关各个 Web 托管计划的定价和功能的信息，请参阅[网站定价详细信息][网站定价详细信息]。

> [WACOM.NOTE] 在将网站从“免费”Web 托管计划模式切换到“基本”或“标准”Web 托管计划模式时，必须先取消网站订阅已有的支出上限。要查看或更改适用于您的 Microsoft Azure 网站订阅的选项，请参阅 [Microsoft Azure 订阅][Microsoft Azure 订阅]。

本文内容：

-   [缩放到“共享”或“基本”计划模式][缩放到“共享”或“基本”计划模式]
-   [缩放到“标准”计划模式][缩放到“标准”计划模式]
-   [缩放连接到您的网站的 SQL Server 数据库][缩放连接到您的网站的 SQL Server 数据库]
-   [开发人员功能][开发人员功能]
-   [其他功能][其他功能]

<a name="scalingsharedorbasic"></a>
<!-- ===================================== -->

## 缩放到“共享”或“基本”计划模式

<!-- ===================================== -->

1.  在浏览器中，打开“管理门户”[][]。

2.  在“网站”选项卡上，选择您的网站。

    ![选择网站][选择网站]

3.  单击“缩放”选项卡。

    ![“缩放”选项卡][“缩放”选项卡]

4.  在“Web 托管计划模式”部分中，选择“共享”或“基本”。映像中的示例选择的是“基本”。

    ![选择 Web 托管计划][选择 Web 托管计划]

    “Web 托管计划站点”部分将介绍当前计划中的简短站点列表。当前计划中的所有站点都将更改为您选择的 Web 托管计划模式。

5.  在“容量”部分中，选择“实例大小”。可用的选项包括“小”、“中”或“大”。在“共享”模式下，实例大小选项不可用。有关这些实例大小的详细信息，请参阅 [Microsoft Azure 虚拟机和云服务大小][Microsoft Azure 虚拟机和云服务大小]。

    ![“基本”模式的实例大小][“基本”模式的实例大小]

6.  使用滑块来选择所需的“实例数”。

    ![“基本”模式的实例数][“基本”模式的实例数]

7.  在命令栏中，选择“保存”。

    ![“保存”按钮][“保存”按钮]

    > [WACOM.NOTE] 您可以根据需要，分别配置和保存“Web 托管计划”、“实例大小”和“实例数”。

8.  将显示一条确认消息，提醒您当前网站所在的 Web 托管计划中的其他站点也将更改为新模式。选择“是”以完成更改。

    在示例中，计划模式已更改为“基本”：

    ![计划更改完成][计划更改完成]

<a name="scalingstandard"></a>
<!-- ================================= -->

## 缩放到“标准”计划模式

<!-- ================================= -->

> [WACOM.NOTE] 在将 Web 托管计划切换到“标准”模式之前，必须先取消 Microsoft Azure 网站订阅已有的支出上限。否则，如果在计费期结束之前达到上限，您的网站将有不可用的风险。要查看或更改适用于您的 Microsoft Azure 网站订阅的选项，请参阅 [Microsoft Azure 订阅][Microsoft Azure 订阅]。

1.  要缩放到“标准”模式，请按照缩放到“共享”或“基本”模式时采用的初始步骤，然后对“Web 托管计划模式”选择“标准”。

    ![选择标准计划][选择标准计划]

    和之前一样，“Web 托管计划站点”部分将介绍当前计划中的简短站点列表。在此情况下，当前计划中的所有站点都会更改为“标准”模式。

2.  选择“标准”并展开“容量”部分，以显示“实例大小”和“实例数”选项，这两个选项在“基本”模式下也可用。“针对时间表编辑缩放设置”和“按度量值缩放”选项仅在“标准”模式下可用。

    ![“标准”模式下的“容量”部分][“标准”模式下的“容量”部分]

3.  配置“实例大小”。可用的选项包括“小”、“中”或“大”。

    ![选择实例大小][选择实例大小]

    有关这些实例大小的详细信息，请参阅 [Microsoft Azure 虚拟机和云服务大小][Microsoft Azure 虚拟机和云服务大小]。

4.  如果要根据白天与夜间、工作日与周末和/或特定日期与时间对资源进行自动缩放，请在“针对时间表编辑缩放设置”选项下选择“设置计划时间”。

    ![设置计划时间][设置计划时间]

5.  “设置计划时间”对话框提供了大量有用的配置选择。

    ![SetUpScheduleTimesDialog][SetUpScheduleTimesDialog]

6.  在“定期计划”下，选择“白天和夜间之间的缩放级别不同”和/或“工作日和周末之间的缩放级别不同”，根据单独的白天和夜间计划和/或单独的工作日和周末计划来对资源进行缩放。

    > [WACOM.NOTE] 对于此功能，周末从星期五结束时开始（默认为晚上 8:00），在星期一开始时结束（默认为上午 8:00）。周末配置文件使用您在“时间”设置中定义的一天的开始和结束时间。

7.  在“时间”下，选择一天的开始和结束时间（以半小时为增量）以及时区。默认情况下，一天从上午 8:00 开始，晚上 8:00 结束。是否采用夏令时取决于您选择的时区。

8.  在“特定日期”下，您可以创建一个或多个要缩放资源的命名的时间范围。例如，您可能要为 Web 流量可能会出现较大峰值的销售活动或产品发布会提供额外资源。

9.  选择好之后，单击“确定”以关闭“计划时间”对话框。

10. “针对时间表编辑缩放设置”框现在根据您所做的更改显示可配置的计划或事件。选择一个定期计划或特定日期并进行配置。

    ![针对时间表编辑缩放设置][针对时间表编辑缩放设置]

    您现在可以使用“按度量值缩放”和“实例数”功能，为您选择的每个计划微调资源缩放。

11. 要根据网站负载的变动来动态调整网站使用的实例数，请通过选择“CPU”来启用“按度量值缩放”选项。

    ![按度量值缩放][按度量值缩放]

    该图表显示了过去一周所用的实例数。您可以使用该图表来监视缩放活动。

12. “按度量值缩放”可修改“实例数”功能，以便您设置自动缩放所用的虚拟机数量的最大值和最小值。Azure 永远都不会超出您设置的上限或下限。

    ![实例数][实例数]

13. “按度量值缩放”还会启用“目标 CPU”选项，以便您指定 CPU 使用率的目标范围。此范围表示网站的平均 CPU 使用率。Windows Azure 将添加或删除标准实例以使网站保持在这个范围内。

    ![目标 CPU][目标 CPU]

    **注意**：在启用了“按度量值缩放”后，Microsoft Azure 将每 5 分钟检查一次网站 CPU，并根据需要添加实例。如果 CPU 使用率较低，Microsoft Azure 将每两小时删除一次实例，以确保网站保持较高性能。通常，将最小实例计数设为 1 是适合的。但是，如果您的网站使用率突然增加，则必须确保您具有足够的最小实例数目来处理负载。例如，如果在 Microsoft Azure 检查 CPU 使用率之前的 5 分钟时间间隔内流量突增，则您的网站在该时间段可能无法响应。如果您预见到流量将突增，请将最小实例数设为更高的值以便应对这些突增。

14. 在您完成了对“针对时间表编辑缩放设置”列表中所有项目的更改之后，单击页面底部命令栏中的“保存”图标可同时保存所有计划设置（无需单独保存每个计划）。

<a name="ScalingSQLServer"></a>

## 缩放连接到您的网站的 SQL Server 数据库

如果有一个或多个 SQL Server 数据库链接到您的网站（不考虑其 Web 托管计划模式），这些数据库将列在“缩放”页面底部的“链接的资源”部分。

1.  要缩放其中一个数据库，请在“链接的资源”部分单击该数据库名称旁边的“管理此数据库的缩放”链接。

    ![链接的数据库][链接的数据库]

2.  该链接指向 Azure 管理门户的 SQL Server 选项卡，可在其中配置数据库的“版本”和“最大大小”：

    ![缩放 SQL Server 数据库][缩放 SQL Server 数据库]

    对于“版本”，请选择“Web”或“Busines”，具体取决于您需要的存储容量。“Web”版提供较小容量范围，而“企业”版则提供较大容量范围。

    您为“最大大小”选择的值将指定数据库的上限。数据库费用取决于您实际存储的数据量，因此更改“最大大小”属性本身不会影响您的数据库费用。有关详细信息，请参阅 [Microsoft Azure SQL Database 中的帐户和结算][Microsoft Azure SQL Database 中的帐户和结算]。

<a name="devfeatures"></a>

## 开发人员功能

提供了以下面向开发人员的功能，具体取决于 Web 托管计划模式：

**位数**

-   “基本”和“标准”计划模式支持 64 位和 32 位应用程序。
-   “免费”和“共享”计划模式仅支持 32 位应用程序。

**调试器支持**

-   “免费”、“共享”和“基本”Web 托管计划模式提供每个应用程序 1 个并发连接的调试器支持。
-   “标准”Web 托管计划模式提供每个应用程序 5 个并发连接的调试器支持。

<a name="OtherFeatures"></a>

## 其他功能

**Web 终结点监视**

-   “基本”和“标准”Web 托管计划模式提供 Web 终结点监视功能。有关 Web 终结点监视的详细信息，请参阅[如何监视网站][如何监视网站]。

-   有关 Web 托管计划所有其他功能（包括定价和所有用户（包括开发人员）感兴趣的功能）的详细信息，请参阅[网站定价详细信息][网站定价详细信息]。

<a name="Next Steps"></a>

## 后续步骤

-   若要开始使用 Azure，请参阅 [Microsoft Azure 免费试用版][Microsoft Azure 免费试用版]。

-   有关定价、支持和 SLA 的信息，请访问以下链接。

    [数据传输定价详细信息][数据传输定价详细信息]

<!--     [Microsoft Azure Support Plans](http://azure.microsoft.com/zh-cn/support/plans/) -->

    [Service Level Agreements](http://www.windowsazure.cn/zh-cn/support/legal/sla/)

    [SQL Database Pricing Details](http://www.windowsazure.cn/zh-cn/pricing/overview/#sql_database)

    [Virtual Machine and Cloud Service Sizes for Microsoft Azure][vmsizes]

    [Web Sites Pricing Details](http://azure.microsoft.com/zh-cn/pricing/details/web-sites/)

    [Web Sites Pricing Details - SSL Connections](http://azure.microsoft.com/zh-cn/pricing/details/web-sites/#ssl-connections)

-   有关 Azure 网站最佳实践（包括构建可扩展、有弹性的体系结构）的信息，请参阅[最佳实践：Windows Azure 网站 (WAWS)][最佳实践：Windows Azure 网站 (WAWS)]。

-   缩放 Azure 网站相关视频：

    [什么时候缩放 Azure 网站 - 和 Stefan Schackow 一起][什么时候缩放 Azure 网站 - 和 Stefan Schackow 一起]

    [基于 CPU 或计划自动缩放 Azure 网站 - 和 Stefan Schackow 一起][基于 CPU 或计划自动缩放 Azure 网站 - 和 Stefan Schackow 一起]

    [Azure 网站如何缩放 - 和 Stefan Schackow 一起][Azure 网站如何缩放 - 和 Stefan Schackow 一起]

<!-- LINKS --> <!-- IMAGES -->

  [什么是 Web 托管计划？]: http://azure.microsoft.com/zh-cn/documentation/articles/web-sites-web-hosting-plan-overview/
  [Azure 网站 Web 托管计划深入概述]: http://www.azure.microsoft.com/zh-cn/Documentation/Articles/azure-web-sites-web-hosting-plans-in-depth-overview/
  [网站定价详细信息]: http://azure.microsoft.com/zh-cn/pricing/details/web-sites/
  [Microsoft Azure 订阅]: http://go.microsoft.com/fwlink/?LinkID=235288
  [缩放到“共享”或“基本”计划模式]: #scalingsharedorbasic
  [缩放到“标准”计划模式]: #scalingstandard
  [缩放连接到您的网站的 SQL Server 数据库]: #ScalingSQLServer
  [开发人员功能]: #devfeatures
  [其他功能]: #OtherFeatures
  []: https://manage.windowsazure.cn/
  [选择网站]: ./media/web-sites-scale/01SelectWebSite.png
  [“缩放”选项卡]: ./media/web-sites-scale/02SelectScaleTab.png
  [选择 Web 托管计划]: ./media/web-sites-scale/03aChooseWHP.png
  [Microsoft Azure 虚拟机和云服务大小]: http://go.microsoft.com/fwlink/?LinkId=309169
  [“基本”模式的实例大小]: ./media/web-sites-scale/03bChooseBasicInstanceSize.png
  [“基本”模式的实例数]: ./media/web-sites-scale/04ChooseBasicInstanceCount.png
  [“保存”按钮]: ./media/web-sites-scale/05SaveButton.png
  [计划更改完成]: ./media/web-sites-scale/06BasicComplete.png
  [选择标准计划]: ./media/web-sites-scale/07ChooseStandard.png
  [“标准”模式下的“容量”部分]: ./media/web-sites-scale/08CapacitySectionStandard.png
  [选择实例大小]: ./media/web-sites-scale/09ChooseInstanceSize.png
  [设置计划时间]: ./media/web-sites-scale/10SetUpScheduleTimesButton.png
  [SetUpScheduleTimesDialog]: ./media/web-sites-scale/11SetUpScheduleTimesDialog.png
  [针对时间表编辑缩放设置]: ./media/web-sites-scale/12EditScaleSettingsForSchedule.png
  [按度量值缩放]: ./media/web-sites-scale/13ScaleByMetric.png
  [实例数]: ./media/web-sites-scale/14InstanceCount.png
  [目标 CPU]: ./media/web-sites-scale/15TargetCPU.png
  [链接的数据库]: ./media/web-sites-scale/16LinkedResources.png
  [缩放 SQL Server 数据库]: ./media/web-sites-scale/17ScaleDatabase.png
  [Microsoft Azure SQL Database 中的帐户和结算]: http://go.microsoft.com/fwlink/?LinkId=234930
  [如何监视网站]: http://www.windowsazure.cn/zh-cn/manage/services/web-sites/how-to-monitor-websites/
  [Microsoft Azure 免费试用版]: http://www.windowsazure.cn/zh-cn/pricing/free-trial/
  [数据传输定价详细信息]: http://www.windowsazure.cn/zh-cn/pricing/overview/#data_transfer
  [最佳实践：Windows Azure 网站 (WAWS)]: http://blogs.msdn.com/b/windowsazure/archive/2014/02/10/best-practices-windows-azure-websites-waws.aspx
  [什么时候缩放 Azure 网站 - 和 Stefan Schackow 一起]: http://www.windowsazure.com/zh-cn/documentation/videos/azure-web-sites-free-vs-standard-scaling/
  [基于 CPU 或计划自动缩放 Azure 网站 - 和 Stefan Schackow 一起]: http://www.windowsazure.com/zh-cn/documentation/videos/auto-scaling-azure-web-sites/
  [Azure 网站如何缩放 - 和 Stefan Schackow 一起]: http://www.windowsazure.com/zh-cn/documentation/videos/how-azure-web-sites-scale/
