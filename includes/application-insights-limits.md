每个应用程序（即每个检测密钥）的度量值和的事件数都具有一定限制。

限制取决于你选择的[定价层](/pricing/details/application-insights/)。

**资源** | **默认限制** | **最大限制**
-------- | ------------- | -------------
每月的会话数据点数<sup>1，2</sup> | 不受限制 | 
每月的请求、事件、依赖项、跟踪、异常和页面视图的总数据点数 | 500 万 | 5000 万<sup>3</sup>
[跟踪和日志](/documentation/articles/app-insights-search-diagnostic-logs)数据速率 | 200 dp/s | 500 dp/s
[异常](/documentation/articles/app-insights-asp-net-exceptions)数据速率 | 50 dp/s | 50 dp/s
请求、事件、依赖项和页面视图遥测的总数据速率 | 200 dp/s | 500 dp/s
[原始数据](/documentation/articles/app-insights-diagnostic-search)保留期 | 7 天
[聚合数据](/documentation/articles/app-insights-metrics-explorer)保留期 | 90 天
[属性](/documentation/articles/app-insights-api-custom-events-metrics#properties)名称计数 | 100 |
属性名称长度 | 150 | 
属性值长度 | 8192 | 
跟踪和异常消息长度 | 10000 |
[度量值](/documentation/articles/app-insights-api-custom-events-metrics#properties/)名称计数 | 100 |
度量值名称长度 | 150 | 
[可用性测试](/documentation/articles/app-insights-monitor-web-app-availability/) | 10 | 

<sup>1</sup> 数据点是单个度量值或事件，具有附加的属性和度量。

<sup>2</sup> 会话数据点用于记录会话的开始或结束，以及用户标识。

<sup>3</sup> 你可以购买更多的容量以扩展到 5000 万个以上。
 
[关于 Application Insights 中的定价和配额](/documentation/articles/app-insights-pricing)

<!---HONumber=Mooncake_0905_2016-->