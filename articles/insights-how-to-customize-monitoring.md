<properties title="How to customize monitoring" pageTitle="How to customize monitoring" description="Learn how to customize monitoring charts in Azure." authors="stepsic"  />

# 自定义监视

在 Azure 门户预览版中，你现在可以使用比以往更多的方法查看监视数据，包括自定义时间范围及选择更多的度量值。

1.  在 [Azure 门户预览版][Azure 门户预览版]中，依次单击“浏览”和“网站”，然后单击网站的名称以打开分页。

2.  在“监视”小视窗中，你可以查看请求、错误、[Web 测试][Web 测试]和[分析][分析]。单击“今天的请求和错误”部件会显示“度量值”分页。

   ![“监视”小视窗][“监视”小视窗]

1.  “度量值”分页显示有关所选度量值的详细信息。分页的顶部有一个图形，该图形的下方有一个表，其中显示了这些度量值的聚合信息，例如平均值、最小值和最大值。该表的下方有一个列表，其中列出了你定义的警报，这些警报已根据分页上显示的度量值进行筛选。这样，即使发生了许多警报，你也只会在这里看到相关的警报。不过，你仍可以查看网站的所有警报，只需单击“网站”分页上的“警报规则”部件即可。

   ![“度量值”分页][“度量值”分页]

1.  若要自定义显示的度量值，请右键单击图表，然后选择“编辑查询”：

   ![编辑查询][编辑查询]

1.  在“编辑查询”分页上，你可以执行两项操作：更改时间范围，以及选择不同的度量值。

   ![编辑查询][1]

1.  更改时间范围十分轻松，只需选择不同的范围（例如“过去的时间”)，然后单击分页底部的“保存”即可。你也可以选择“自定义”，这是门户预览版的新功能：

   ![自定义时间范围][自定义时间范围]

1.  “自定义”可让你选择过去 2 周的任何一个时间段，例如，你可以查看整整两周的数据，或者只查看昨天某个小时的数据，在文本框中键入一个不同的小时即可。

2.  在时间范围的下面，可以选择要在图表上显示的任意数目的度量值。你可以查看一些新的度量值：“内存工作集”和“平均内存工作集”。

3.  如果单击“保存”，则你的更改将会保留到你退出“网站”分页为止。当你稍后返回时，你仍会看到原始的度量值和时间范围。

## Web 托管计划监视

你可以监视有关运行网站的实例的性能度量值，这是 Azure 门户预览版提供的另一项新功能。若要访问这些度量值，请单击“摘要”小视窗中的“Web 托管计划”图标。

![Web 托管计划][Web 托管计划]

“监视”小视窗中显示了一个图表，其行为与“网站”分页中的图表类似，不过，其中还提供了一些新的度量值：

-   CPU 百分比
-   内存百分比
-   HTTP 队列深度
-   磁盘队列深度

## 创建并排图表

通过 Azure 门户预览版中强大的用户自定义功能，你可以创建并排图表，以供自定义时使用。

1.  首先，请右键单击最先要处理的图表，然后选择“自定义”。

   ![自定义图表][自定义图表]

1.  接下来，请单击“...”菜单上的“克隆”以复制部件

   ![克隆部件][克隆部件]

1.  最后，请在屏幕顶部的工具栏上单击“完成”。现在，你可以将此部件视为普通的度量值部件。如果你右键单击并更改了显示的度量值，则会看到有两个不同的度量值同时并排显示：

   ![并排显示的两个度量值][并排显示的两个度量值]

请注意，当你退出门户时，图表的时间范围和选择的度量值将会重置。

  [Azure 门户预览版]: https://portal.azure.com/
  [Web 测试]: http://go.microsoft.com/fwlink/?LinkID=394528&clcid=0x409
  [分析]: http://go.microsoft.com/fwlink/?LinkID=394529&clcid=0x409
  [“监视”小视窗]: ./media/insights-how-to-customize-monitoring/Insights_MonitoringChart.png
  [“度量值”分页]: ./media/insights-how-to-customize-monitoring/Insights_MetricBlade.png
  [编辑查询]: ./media/insights-how-to-customize-monitoring/Insights_MetricMenu.png
  [1]: ./media/insights-how-to-customize-monitoring/Insights_EditQuery.png
  [自定义时间范围]: ./media/insights-how-to-customize-monitoring/Insights_CustomTime.png
  [Web 托管计划]: ./media/insights-how-to-customize-monitoring/Insights_WHPSelect.png
  [自定义图表]: ./media/insights-how-to-customize-monitoring/Insights_Customize.png
  [克隆部件]: ./media/insights-how-to-customize-monitoring/Insights_ClonePart.png
  [并排显示的两个度量值]: ./media/insights-how-to-customize-monitoring/Insights_SideBySide.png
