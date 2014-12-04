<properties title="How to use end user analytics" pageTitle="How to use end user analytics" description="Learn about end user analytics in Azure." authors="vladj"  />

# 网站分析

想要知道有多少用户访问过你的网站吗？想要知道页面的平均加载时间，或者用户正在使用哪款浏览器访问你的网站吗？只需在网页中插入几行脚本，你就能收集有关客户如何使用你的网站的数据。

![最终用户分析][最终用户分析]

## 如何设置最终用户分析

1.  在“网站”分页上，单击标签为“最终用户分析”的部件。
2.  在“配置”分页上，选择并复制整个检测脚本。

   ![配置][配置]

1.  <p>将该脚本粘贴到网页中结束标记的前面并紧靠该标记。建议你将该脚本插入到你的所有网页中。如果使用的是 ASP.NET，则可以将该脚本插入到应用程序的母版页中。</p>
2.  部署并使用你的 Web 应用程序。使用情况分析信息大约会在 5-10 分钟后出现。

## 浏览数据

使用“浏览器”会话部件可以深入查看不同的浏览器以及浏览器版本。

![浏览器][浏览器]

“分析”部件显示：

-   不同设备类型（包括“台式机”和“移动设备”）的明细。
-   在过去一周最常访问的 5 个页面，以及页面加载时间的图表。另外，还会显示会话和视图的数目

   ![最常访问的页面][最常访问的页面]

-   在过去一周加载速度最慢的页面；你可以进行改善以实现业务目标

  [最终用户分析]: ./media/insights-usage-analytics/Insights_ConfiguredExperience.png
  [配置]: ./media/insights-usage-analytics/Insights_CopyCode.png
  [浏览器]: ./media/insights-usage-analytics/Insights_Browsers.png
  [最常访问的页面]: ./media/insights-usage-analytics/Insights_TopPages.png
