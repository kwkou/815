<properties
	pageTitle="故障排除： Web 应用性能下降"
	description="本文将会帮助你排查 Azure 中托管的 Web 应用存在的性能问题。"
	services="app-service\web"
	documentationCenter=""
	authors="cephalin"
	manager="wpickett"
	editor=""
	tags="top-support-issue"/>

<tags
	ms.service="app-service-web"
	ms.date="01/13/2016"
	wacn.date="05/24/2016"/>

# 故障排除： Web 应用性能下降

本文将会帮助你排查 [Azure Web 应用](/documentation/services/web-sites/)中托管的 Web 应用存在的性能问题。

如果你对本文中的任何观点存在疑问，可以联系 [MSDN Azure 和 CSDN Azure](/support/forums/) 上的 Azure 专家。或者，你也可以提出 Azure 支持事件。请转到 [Azure 支持站点](/support/contact/)并单击“获取支持”。

## 症状

浏览 Web 应用时，网页加载缓慢，并且有时还会超时。

## 原因

此问题通常是应用程序级别的问题造成的，例如：

-	请求耗费过长的时间
-	应用程序的内存/CPU 使用率过高
-	应用程序因异常而崩溃

## 疑难解答步骤

故障排除可划分为三种不同的任务，依次为：

1.	[观察和监视应用程序行为](#observe)
2.	[收集数据](#collect)
3.	[缓解问题](#mitigate)

[Azure Web 应用](/home/features/web-site/)为每个步骤提供了多种选项。

###<a name="observe"></a> 1\.观察和监视应用程序行为

#### 监视你的 Web 应用

此页面可让你找出应用程序是否存在任何问题。在经典管理门户中，单击你网页的“监视器”。

你可能想要在 Web 应用中监视的一些指标包括

-	平均内存工作集
-	平均响应时间
-	CPU 时间
-	内存工作集
-	请求

![](./media/app-service-web-troubleshoot-performance-degradation/1-monitor-metrics.png)

有关详细信息，请参阅：

-	[在 Azure 中监视 Web 应用](/documentation/articles/web-sites-monitor/)

#### 监视 Web 终结点状态

如果在**标准**定价层中运行 Web 应用， Web 应用可让你从 3 个地理位置监视 2 个终结点。

终结点监视可从测试 Web URL 的响应时间和运行时间的分布式地理位置配置 Web 测试。该测试可对 Web URL 执行 HTTP GET 操作，以从每个位置确定响应时间和运行时间。每个已配置位置每 5 分钟运行一次测试。

将使用 HTTP 响应代码监视运行时间，并且以毫秒为单位计算响应时间。如果 HTTP 响应代码大于或等于 400 或响应时间超过 30 秒，则监视测试失败。如果从所有指定的位置监视测试均成功，则终结点被视为可用。

若要设置此功能，请参阅[如何：监视 Web 终结点状态](/documentation/articles/web-sites-monitor/#webendpointstatus)。

###<a name="collect"></a> 2\.收集数据

####	使用 FTP 诊断日志

在你的网页的“仪表板”页面中，找到 FTP 诊断日志的链接，一般为 `ftp://********.ftp.chinacloudsites.chinacloudapi.cn/LogFiles`。点击进去，可以下载 eventlog.xml。

##### Web 服务器诊断

你可以启用或禁用以下种类的日志：

-	**详细错误日志记录** - 指示故障的 HTTP 状态代码（状态代码 400 或更大数字）的详细错误消息。其中可能包含有助于确定服务器返回错误代码的原因的信息。
-	**失败请求跟踪** - 有关失败请求的详细信息，包括对用于处理请求的 IIS 组件和每个组件所用的时间的跟踪。在尝试提高 Web 应用性能或查找导致特定 HTTP 错误的问题时，此信息很有用。
-	**Web 服务器日志记录** - 使用 W3C 扩展日志文件格式的 HTTP 事务信息。这在确定总体 Web 应用度量值（如处理的请求数量或来自特定 IP 地址的请求数）时非常有用。

##### 应用程序诊断

使用应用程序诊断可以捕获 Web 应用生成的信息。ASP.NET 应用程序可使用 `System.Diagnostics.Trace` 类将信息记录到应用程序诊断日志。

有关如何在应用程序中配置日志记录的详细说明，请参阅[在 Azure 中为 Web 应用启用诊断日志记录](/documentation/articles/web-sites-enable-diagnostic-log/)。

###<a name="mitigate"></a> 3\.缓解问题

####	缩放 Web 应用

在 Azure 中，为了提高性能和吞吐量，你可以调整运行应用程序的规模。向上缩放 Web 应用涉及到两个相关操作：将 App Service 计划更改为较高的定价层，以及在切换到较高的定价层后配置特定的设置。

有关缩放的详细信息，请参阅[在 Azure 中缩放 Web 应用](/documentation/articles/web-sites-scale/)。

此外，你可以选择在多个实例上运行应用程序。这不仅能提供更强大的处理能力，而且还能提供一定程度的容错。如果进程在某个实例上中断，其他实例仍将继续处理请求。

可以将缩放设置为手动或自动。

####	使用 AutoHeal

AutoHeal 会根据你选择的设置（例如配置更改、请求、基于内存的限制或执行请求所需的时间），回收应用程序的工作进程。在大多数情况下，回收进程是在出现问题后进行恢复的最快方式。尽管你始终可以从 Azure 经典管理门户直接重新启动 Web 应用，但 AutoHeal 可以自动为你执行此操作。你只需在 Web 应用的根 web.config 中添加一些触发器即可。请注意，即使你的应用程序并非 .Net 应用程序，这些设置的工作方式也仍然相同。


####	重新启动 Web 应用

这通常是在发生一次性问题后进行恢复的最简单方式。[Azure 经典管理门户](https://manage.windowsazure.cn/)上的 Web 应用边栏选项卡中提供了用于停止或重新启动应用程序的选项。

 ![](./media/app-service-web-troubleshoot-performance-degradation/2-restart.png)

你还可以使用 Azure Powershell 管理 Web 应用。有关详细信息，请参阅[将 Azure PowerShell 与 Azure 资源管理器配合使用](/documentation/articles/powershell-azure-resource-manager/)。

<!---HONumber=Mooncake_0104_2016-->