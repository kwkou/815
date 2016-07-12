<properties 
	pageTitle="监视服务指标" 
	description="了解如何在 Azure 中自定义监视图表。" 
	authors="stepsic-microsoft-com" 
	manager="ronmart" 
	editor="" 
	services="azure-portal"
documentationCenter=""/>

<tags 
	ms.service="azure-portal" 
	ms.date="09/08/2015"
	wacn.date="05/09/2016"/>

# 监视服务指标

所有 Azure 服务都会跟踪使你可以监视你服务的运行状况、性能、可用性和使用情况的关键指标。可以在 Azure 门户预览中查看这些指标，也可以使用 [REST API](https://msdn.microsoft.com/zh-cn/library/azure/dn931930.aspx) 或 [.NET SDK](https://www.nuget.org/packages/Microsoft.Azure.Insights/) 以编程方式访问完整的指标集。

对于某些服务，可能需要打开诊断以便查看任何指标。对于其他服务（如虚拟机），你会获得基本指标集，但需要启用完整高频率指标集。请参阅[启用监视和诊断](/documentation/articles/insights-how-to-use-diagnostics/)以了解详细信息。

## 使用监视图表 

可以在所选的任何时间段内绘制任何指标的图表。

1. 在 [Azure 门户预览](https://portal.azure.cn/)中，单击“浏览”，然后单击要监视的资源。

2. “监视”部分包含每个 Azure 资源的最重要指标。例如，Web 应用具有“请求和错误”，而虚拟机具有“CPU 百分比”和“磁盘读写”：

    ![“监视”小视窗](./media/insights-how-to-customize-monitoring/Insights_MonitoringChart.png)

3. 单击任何图表都会显示“指标”边栏选项卡。在该边栏选项卡中，除了该图之外还有一个表，其中显示指标（例如在所选时间范围内的平均值、最小值和最大值）的聚合。下面是资源的警报规则。

    ![“度量值”分页](./media/insights-how-to-customize-monitoring/Insights_MetricBlade.png)

4. 若要自定义显示的折线图，请单击图表上的“编辑”按钮，或指标边栏选项卡上的“编辑图表”命令。

5. 在“编辑查询”边栏选项卡上，你可以执行三个操作：
    - 更改时间范围
    - 在条形图与折线图之间切换外观
    - 选择不同指标

        ![编辑查询](./media/insights-how-to-customize-monitoring/Insights_EditQuery.png)

6. 更改时间范围十分轻松，只需选择不同的范围（例如“前一个小时”），然后单击边栏选项卡底部的“保存”即可。还可以选择“自定义”，这使你可以选择过去 2 周内的任何时间段。例如，你可以查看整个两周，或仅仅是昨天的 1 小时。在文本框中输入一个不同的小时即可。

    ![自定义时间范围](./media/insights-how-to-customize-monitoring/Insights_CustomTime.png)

7. 在时间范围的下面，可以选择要在图表上显示的任意数目的度量值。

8. 单击“保存”时，会针对该特定资源保存更改。例如，如果你有两个虚拟机，并且对其中一个更改了图表，则不会影响另一个。

## 创建并排图表

借助门户预览中功能强大的自定义，可以添加所需任何数量的图表。

1. 在边栏选项卡顶部的“...”菜单中，单击“添加磁贴”：

    ![添加菜单](./media/insights-how-to-customize-monitoring/Insights_AddMenu.png)
    
2. 然后可以从屏幕右侧的“库”中选择图表：

    ![库](./media/insights-how-to-customize-monitoring/Insights_Gallery.png)
    
3. 如果看不到所需指标，则始终可以添加一个预设指标，然后“编辑”图表以显示所需指标。 

## 监视使用配额

大多数指标都会显示随时间推移的趋势，但某些数据（如使用配额）是具有阈值的时间点信息。

还可以在具有配额的资源的边栏选项卡上查看使用配额：

   ![使用情况](./media/insights-how-to-customize-monitoring/Insights_UsageChart.png)

与指标一样，可以使用 [REST API](https://msdn.microsoft.com/zh-cn/library/azure/dn931963.aspx) 或 [.NET SDK](https://www.nuget.org/packages/Microsoft.Azure.Insights/) 以编程方式访问完整使用配额集。

## 后续步骤

* 每当指标超过阈值时[接收警报通知](/documentation/articles/insights-receive-alert-notifications/)。
* [启用监视和诊断](/documentation/articles/insights-how-to-use-diagnostics/)以收集有关服务的详细高频率指标。
* [自动缩放实例计数](/documentation/articles/insights-how-to-scale/)以确保服务可用且响应迅速。
* 在要确切了解代码在云中的执行情况时[监视应用程序性能](/documentation/articles/insights-perf-analytics/)。


 
<!---HONumber=Mooncake_0503_2016-->