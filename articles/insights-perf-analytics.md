<properties
	pageTitle="监视 Azure Web 应用性能"
	description="对负载和响应时间、依赖项信息绘制图表，并对性能设置警报。"
	services="azure-portal"
    documentationCenter="na"
	authors="alancameronwills" 
	manager="douge"/>

<tags 
	ms.service="azure-portal" 
	ms.date="09/23/2015"
	wacn.date="05/09/2016"/>

# 监视 Azure 网站性能

在 [Azure 门户预览](http://portal.azure.cn)中，可以设置监视以收集有关 Azure 网站或[虚拟机](/documentation/articles/virtual-machines-about/)中的应用程序依赖项的统计信息和详细信息。

Azure 通过利用扩展来支持应用程序性能监视（即 APM）。这些扩展会安装到应用程序中，收集数据并报告回给监视服务。

Application Insights 和 New Relic 是两个可用的性能监视扩展。若要使用它们，请在运行时安装代理。对于 Application Insights，还可以选择使用 SDK 生成代码。SDK 使你可以编写代码，以便更详细地监视应用的使用情况和性能。

## 添加扩展

1. 单击“浏览”并选择要检测的 Web 应用或虚拟机。

2. 添加 Application Insights 或 New Relic 扩展。

    如果要检测 Web 应用：

    ![设置，扩展，添加，Application Insights](./media/insights-perf-analytics/05-extend.png)

    或者，如果使用的是虚拟机：

    ![单击“分析”磁贴](./media/insights-perf-analytics/10-vm1.png)

### 对于 Application Insights 是可选的：使用 SDK 重新生成

Application Insights 可以通过将 SDK 安装到应用中来提供更详细的遥测。

在 Visual Studio 中，将 Application Insights SDK 添加到项目。

![右键单击 Web 项目，然后选择“添加 Application Insights”](./media/insights-perf-analytics/03-add.png)

当系统提示登录时，使用 Azure 帐户的凭据。

可以通过在开发计算机中运行应用来测试遥测，也可以只是继续进行并重新发布它。


## 浏览数据

使用应用程序一段时间以便生成一些遥测。

1. 然后，在 Web 应用或虚拟机边栏选项卡中会看到安装的扩展。
2. 单击表示应用程序的行以导航到该提供程序：

![单击刷新](./media/insights-perf-analytics/06-overview.png)

还可以使用“浏览”直接转到所使用的 Application Insights 组件或 New Relic 帐户。

进入边栏选项卡之后，例如对于 Application Insights，可以：

- 打开性能：

    ![在 Application Insights 概述边栏选项卡中，单击“性能”磁贴](./media/insights-perf-analytics/07-dependency.png)

- 钻取以查看单个请求：

    ![在网格中，单击依赖项以查看相关请求。](./media/insights-perf-analytics/08-requests.png)

- 下面是一个示例，它演示在某个 SQL 依赖项方面所花费的时间量，包括 SQL 调用数和相关统计信息（如平均持续时间和标准偏差）。 

    ![](./media/insights-perf-analytics/01-example.png)



## 后续步骤

* [监视服务运行状况指标](/documentation/articles/insights-how-to-customize-monitoring/)以确保你的服务可用且响应迅速。
* [启用监视和诊断](/documentation/articles/insights-how-to-use-diagnostics/)以收集有关服务的详细高频率指标。
* 每当操作事件发生或指标超过阈值时[接收警报通知](/documentation/articles/insights-receive-alert-notifications/)。


 
<!---HONumber=Mooncake_0503_2016-->