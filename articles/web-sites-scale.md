<properties 
	pageTitle="在 Azure 中缩放 Web 应用" 
	description="了解如何向上扩展和横向扩展 Azure 中的 Web 应用（包括自动缩放）。" 
	services="app-service\web" 
	documentationCenter="" 
	authors="cephalin" 
	manager="wpickett" 
	editor="mollybos"/>

<tags 
	ms.service="app-service"
	ms.date="02/25/2016" 
	wacn.date="04/26/2016"/>

# 在 Azure 中缩放 Web 应用#

为提高你的 Web 应用在 Azure 上的性能和吞吐量，你可以使用 Azure 经典管理门户将 Web 托管计划模式从“免费”缩放为“共享”、“基本”或“标准”。

在 Azure Web 应用上放大包括两个相关操作：将您的 Web 托管计划模式更改为更高服务级别，以及切换到更高服务级别后配置某些设置。本文将介绍这两个主题。更高的服务层（如“标准”模式）在确定如何使用 Azure 上的资源方面可提供更高的可靠性和灵活性。

可在经典管理门户的“缩放”选项卡中轻松地更改模式并进行配置。您可以根据需要向上或向下缩放。这些更改只需数秒即可生效，可影响 Web 托管计划中的所有 Web 应用。无需更改代码或重新部署应用程序。

有关各个 Web 托管计划的定价和功能的信息，请参阅[ Web 应用定价详细信息](/home/features/web-site/pricing/)。

<a name="scalingsharedorbasic"></a>
<!-- ===================================== -->
## 缩放到“共享”或“基本”模式
<!-- ===================================== -->

1. 在浏览器中，打开[经典管理门户][portal]。
	
2. 在“ Web 应用”选项卡上，选择你的 Web 应用。
	
	![选择 Web 应用][Select Website]
	
3. 单击“缩放”选项卡。
	
	![“缩放”选项卡][SelectScaleTab]
	
4. 在“Web 托管计划模式”部分，选择“共享”或“基本”。映像中的示例选择的是“基本”。
	
	![选择 Web 托管计划][ChooseWHP]
	
	“Web 托管计划站点”部分将介绍当前计划中的简短站点列表。当前计划中的所有站点都将更改为您选择的 Web 托管计划模式。
	
5. 在“容量”部分，选择“实例大小”。可用的选项包括“小”、“中”或“大”。在“共享”模式下，实例大小选项不可用。有关这些实例大小的详细信息，请参阅 Azure [虚拟机](/documentation/articles/virtual-machines-windows-sizes/)和[云服务大小](/documentation/articles/cloud-services-sizes-specs/)。
	
	![“基本”模式的实例大小][ChooseBasicInstanceSize]
	
6. 使用滑块来选择所需的“实例计数”。
	
	![“基本”模式的实例数][ChooseBasicInstanceCount]
	
7. 在命令栏中，选择“保存”。
	
	![“保存”按钮][SaveButton]
 	
	> [AZURE.NOTE] 你可以根据需要，分别配置和保存“Web 托管计划”、“实例大小”和“实例计数”设置。
	
8. 将显示一条确认消息，提醒您当前 Web 应用所在的 Web 托管计划中的其他站点也将更改为新模式。选择“是”以完成更改。
	
	在示例中，计划模式已更改为“基本”：
	
	![计划更改完成][BasicComplete]
	
<a name="scalingstandard"></a>
<!-- ================================= -->
## 缩放到“标准”计划模式
<!-- ================================= -->

> [AZURE.NOTE] 在将 Web 托管计划切换到“标准”模式之前，必须先取消 Azure Web 应用订阅已有的支出上限。否则，如果在计费期结束之前达到上限，您的 Web 应用将有不可用的风险。若要查看或更改针对你的 Azure Web 应用订阅的选项，请参阅 [Azure 订阅][azuresubscriptions]。

1. 若要缩放到“标准”模式，请按照缩放到“共享”或“基本”模式时采用的初始步骤进行操作，然后为 Web 托管计划模式选择“标准”。 
	
	![选择标准计划][ChooseStandard]
	
	和之前一样，“Web 托管计划站点”部分将介绍当前计划中的简短站点列表。在此情况下，当前计划中的所有站点都会更改为“标准”模式。
	
2. 选择“标准”并展开“容量”部分，以显示“实例大小”和“实例计数”选项，这两个选项在“基本”模式下也可用。“针对时间表编辑缩放设置”和“按度量值缩放”选项仅在“标准”模式下可用。
	
	![“标准”模式下的“容量”部分][CapacitySectionStandard]
	
3. 配置“实例大小”。可用的选项包括“小”、“中”或“大”。
	
	![选择实例大小][ChooseInstanceSize]
	
	有关这些实例大小的详细信息，请参阅 [Azure 虚拟机和云服务大小][vmsizes]。
	
4. 如果要根据白天与夜间、工作日与周末和/或特定日期与时间对资源进行自动缩放，请在“针对时间表编辑缩放设置”选项下选择“设置计划时间”。
	
	![设置计划时间][SetUpScheduleTimesButton]
	
5. “设置计划时间”对话框提供了大量有用的配置选择。
	
	![SetUpScheduleTimesDialog][SetUpScheduleTimesDialog]
	
6. 在“定期计划”下，选择“白天和夜间之间的缩放级别不同”和/或“工作日和周末之间的缩放级别不同”，根据单独的白天和夜间计划和/或单独的工作日和周末计划来对资源进行缩放。
	
	> [AZURE.NOTE] 出于此功能的目的，周末在星期五结束时开始（默认为晚上 8:00），在星期一白天开始时结束（默认为上午 8:00）。周末配置文件使用你在“时间”设置中定义的一天的开始和结束时间。
	
7. 在“时间”下，选择一天的开始和结束时间（以半小时为增量）以及时区。默认情况下，一天在上午 8:00 开始，在晚上 8:00 结束。是否采用夏令时取决于您选择的时区。
	
8. 在“特定日期”下，你可以创建一个或多个要缩放资源的命名的时间范围。例如，您可能要为 Web 流量可能会出现较大峰值的销售活动或产品发布会提供额外资源。
	
9. 选择好之后，单击“确定”以关闭“计划时间”对话框。
	
10.   “针对时间表编辑缩放设置”框现在根据你所做的更改显示可配置的计划或事件。选择一个定期计划或特定日期并进行配置。
	
	![针对时间表编辑缩放设置][EditScaleSettingsForSchedule]
	
	你现在可以使用“按度量值缩放”和“实例计数”功能，为你选择的每个计划微调资源缩放。
	
11.  要根据 Web 应用负载的变动来动态调整 Web 应用使用的实例数，请通过选择“CPU”来启用“按度量值缩放”选项。
	
	![按度量值缩放][ScaleByMetric]
	
	该图表显示了过去一周所用的实例数。您可以使用该图表来监视缩放活动。
	
12. “按度量值缩放”修改“实例计数”功能，以便你设置自动缩放所用的虚拟机数量的最大值和最小值。Azure 永远都不会超出您设置的上限或下限。
	
	![实例数][InstanceCount]
	
13. “按度量值缩放”还会启用“目标 CPU”选项，以便你指定 CPU 使用率的目标范围。此范围表示 Web 应用的平均 CPU 使用率。Azure 将添加或删除标准实例以使 Web 应用保持在这个范围内。
	
	![目标 CPU][TargetCPU]
	
	**注意**：在启用了“按度量值缩放”后，Azure 每隔 5 分钟就会检查你的 Web 应用的 CPU，并且在该时间点根据需要添加实例。如果 CPU 使用率较低，Azure 将每两小时删除一次实例，以便确保你的 Web 应用保持高性能。通常，将最小实例计数设为 1 是适合的。但是，如果您的 Web 应用使用率突然增加，则必须确保您具有足够的最小实例数目来处理负载。例如，如果在 Azure 检查您的 CPU 使用率之前的 5 分钟时间间隔中遇到了流量突增，则您的 Web 应用在该时间段可能无法响应。如果您遇到突增的大量流量，请将该最小实例数设为更高的值以便应对这些突增。
	
14. 在你完成了对“针对时间表编辑缩放设置”列表中项目的更改之后，单击页面底部命令栏中的“保存”图标可同时保存所有计划设置（无需单独保存每个计划）。

<a name="ScalingSQLServer"></a>
##缩放连接到您的 Web 应用的 SQL Server 数据库	
如果有一个或多个 SQL Server 数据库链接到你的 Web 应用（不考虑其 Web 托管计划模式），这些数据库将列在“缩放”页面底部的“链接的资源”部分。

1. 若要缩放其中一个数据库，请在“链接的资源”部分单击该数据库名称旁边的“管理此数据库的缩放”链接。
	
	![链接的数据库][LinkedResources]
	
2. 该链接指向 Azure 经典管理门户的 SQL Server 选项卡，可在其中配置数据库的“版本”和“最大大小”：
	
	![缩放 SQL Server 数据库][ScaleDatabase]
	
	对于“版本”，可选择“基本”、“标准”或“高级”，具体取决于你需要的存储容量。对于“Web”和“企业”版本的未来情况，请参阅 [Web 和企业版停用常见问题](/documentation/articles/sql-database-web-business-sunset-faq/)。
	
	你为“最大大小”选择的值将指定数据库的上限。数据库费用取决于你实际存储的数据量，因此更改“最大大小”属性本身不会影响你的数据库费用。有关详细信息，请参阅 [Azure SQL 数据库中的帐户和结算][SQLaccountsbilling]。

<a name="devfeatures"></a>
## 开发人员功能
提供了以下面向开发人员的功能，具体取决于 Web 应用的模式：

### 位数 ###

- “基本”和“标准”模式支持 64 位和 32 位应用程序。
- “免费”和“共享”计划模式仅支持 32 位应用程序。

### 调试器支持 ###

- “免费”、“共享”和“基本”Web 托管计划模式提供每个应用程序 1 个并发连接的调试器支持。
- “标准”Web 托管计划模式提供每个应用程序 5 个并发连接的调试器支持。

<a name="OtherFeatures"></a>
## 其他功能

### Web 端点监视 ###

- “基本”和“标准”Web 托管计划模式提供 Web 终结点监视功能。有关 Wb 终结点监视的详细信息，请参阅[如何监视 Web 应用](/documentation/articles/web-sites-monitor/)。

- 有关 Web 托管计划所有其他功能（包括定价和所有用户（包括开发人员）感兴趣的功能）的详细信息，请参阅[ Web 应用定价详细信息](/home/features/web-site/pricing/)。

<a name="Next Steps"></a>
## 后续步骤

- 若要开始使用 Azure，请参阅 [Azure 免费试用版](/zh-cn/pricing/1rmb-trial/)。

- 有关定价、支持和 SLA 的信息，请访问以下链接。
	
	[数据传输定价详细信息](/pricing/details/data-transfer/)
	
	[服务级别协议](/support/legal/sla/)
	
    [SQL 数据库定价详细信息](/home/features/sql-database/pricing/)
	
	Azure 的[虚拟机](/documentation/articles/virtual-machines-windows-sizes/)和[云服务](/documentation/articles/cloud-services-sizes-specs/)大小
	
	[ Web 应用定价详细信息](/home/features/web-site/pricing/)

<!-- LINKS -->
[vmsizes]: http://msdn.microsoft.com/zh-cn/library/azure/dn197896.aspx
[SQLaccountsbilling]: http://msdn.microsoft.com/zh-cn/library/azure/ee621788.aspx
[azuresubscriptions]: https://manage.windowsazure.cn
[portal]: https://manage.windowsazure.cn/

<!-- IMAGES -->
[Select Website]: ./media/web-sites-scale/01SelectWebSite.png
[SelectScaleTab]: ./media/web-sites-scale/02SelectScaleTab.png

[ChooseWHP]: ./media/web-sites-scale/03aChooseWHP.png
[ChooseBasicInstanceSize]: ./media/web-sites-scale/03bChooseBasicInstanceSize.png
[ChooseBasicInstanceCount]: ./media/web-sites-scale/04ChooseBasicInstanceCount.png
[SaveButton]: ./media/web-sites-scale/05SaveButton.png
[BasicComplete]: ./media/web-sites-scale/06BasicComplete.png
[ChooseStandard]: ./media/web-sites-scale/07ChooseStandard.png
[CapacitySectionStandard]: ./media/web-sites-scale/08CapacitySectionStandard.png
[ChooseInstanceSize]: ./media/web-sites-scale/09ChooseInstanceSize.png
[SetUpScheduleTimesButton]: ./media/web-sites-scale/10SetUpScheduleTimesButton.png
[SetUpScheduleTimesDialog]: ./media/web-sites-scale/11SetUpScheduleTimesDialog.png
[EditScaleSettingsForSchedule]: ./media/web-sites-scale/12EditScaleSettingsForSchedule.png
[ScaleByMetric]: ./media/web-sites-scale/13ScaleByMetric.png
[InstanceCount]: ./media/web-sites-scale/14InstanceCount.png
[TargetCPU]: ./media/web-sites-scale/15TargetCPU.png
[LinkedResources]: ./media/web-sites-scale/16LinkedResources.png
[ScaleDatabase]: ./media/web-sites-scale/17ScaleDatabase.png

<!---HONumber=76-->