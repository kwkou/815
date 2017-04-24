<properties
    pageTitle="应用服务中 Web 应用性能缓慢 | Azure"
    description="本文将帮助你排查 Azure App Service 中 Web 应用性能缓慢的问题。"
    services="app-service\web"
    documentationcenter=""
    author="cephalin"
    manager="erikre"
    editor=""
    tags="top-support-issue"
    keywords="Web 应用性能，缓慢应用，应用缓慢"
    translationtype="Human Translation" />
<tags
    ms.assetid="b8783c10-3a4a-4dd6-af8c-856baafbdde5"
    ms.service="app-service-web"
    ms.workload="web"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="07/06/2016"
    wacn.date="04/24/2017"
    ms.author="cephalin"
    ms.sourcegitcommit="a114d832e9c5320e9a109c9020fcaa2f2fdd43a9"
    ms.openlocfilehash="7b3dea474a2d5a81fc32d1a388b5affda23fa484"
    ms.lasthandoff="04/14/2017" />

# <a name="troubleshoot-slow-web-app-performance-issues-in-azure-app-service"></a>排查 Azure App Service 中 Web 应用性能缓慢的问题
本文将帮助排查 [Azure 应用服务](/documentation/articles/app-service-changes-existing-services/)中 Web 应用性能缓慢的问题。

如果你对本文中的任何观点存在疑问，可以联系 [MSDN Azure 和 CSDN Azure](/support/forums/) 上的 Azure 专家。 或者，你也可以提出 Azure 支持事件。 请转到 [Azure 支持站点](/support/contact/)，并单击“**获取支持**”。

## <a name="symptom"></a>症状
浏览 Web 应用时，网页加载缓慢，并且有时还会超时。

## <a name="cause"></a>原因
此问题通常是应用程序级别的问题造成的，例如：

* 请求耗费过长的时间
* 应用程序的内存/CPU 使用率过高
* 应用程序因异常而崩溃

## <a name="troubleshooting-steps"></a>疑难解答步骤
故障排除可划分为三种不同的任务，依次为：

1. [观察和监视应用程序行为](#observe)
2. [收集数据](#collect)
3. [缓解问题](#mitigate)

[应用服务 Web 应用](/home/features/app-service/web-apps/)为每个步骤提供了多种选项。

### <a name="observe"></a> 1.观察和监视应用程序行为
#### <a name="track-service-health"></a>跟踪服务运行状况
每次发生服务中断或性能下降时 Azure 会进行宣传。 可以在 [Azure 门户预览](https://portal.azure.cn/)中跟踪服务的运行状况。 有关详细信息，请参阅[跟踪服务的运行状况](/documentation/articles/insights-service-health/)。

#### <a name="monitor-your-web-app"></a>监视你的 Web 应用
此选项可让你找出应用程序是否存在任何问题。 在 Web 应用的边栏选项卡中，单击“请求和错误”磁贴。 “指标”边栏选项卡将显示所有可以添加的指标。

你可能想要在 Web 应用中监视的一些指标包括

* 平均内存工作集
* 平均响应时间
* CPU 时间
* 内存工作集
* 请求

![监视 Web 应用性能](./media/app-service-web-troubleshoot-performance-degradation/1-monitor-metrics.png)

有关详细信息，请参阅：

* [监视 Azure 应用服务中的 Web 应用](/documentation/articles/web-sites-monitor/)
* [接收警报通知](/documentation/articles/insights-receive-alert-notifications/)

#### <a name="monitor-web-endpoint-status"></a>监视 Web 终结点状态
如果在 **标准** 定价层中运行 Web 应用，Web Apps 可让你从 3 个地理位置监视 2 个终结点。

终结点监视可从测试 Web URL 的响应时间和运行时间的分布式地理位置配置 Web 测试。 该测试可对 Web URL 执行 HTTP GET 操作，以从每个位置确定响应时间和运行时间。 每个已配置位置每 5 分钟运行一次测试。

将使用 HTTP 响应代码监视运行时间，并且以毫秒为单位计算响应时间。 如果 HTTP 响应代码大于或等于 400 或响应时间超过 30 秒，则监视测试失败。 如果从所有指定的位置监视测试均成功，则终结点被视为可用。

有关详细信息，请参阅[在 Azure App Service 中监视应用](/documentation/articles/web-sites-monitor/)

另外，有关终结点监视的视频，请参阅[保持 Azure 网站运行以及终结点监视 - Stefan Schackow](https://channel9.msdn.com/Shows/Azure-Friday/Keeping-Azure-Web-Sites-up-plus-Endpoint-Monitoring-with-Stefan-Schackow)。

#### <a name="application-performance-monitoring-using-extensions"></a>使用扩展的应用程序性能监视
还可以利用*站点扩展*监视应用程序的性能。

每个应用服务 Web 应用提供了一个可扩展的管理终结点，通过此终结点可利用一组作为站点扩展部署的功能强大的工具。 这些工具涵盖从源代码编辑器（例如 [Visual Studio Team Services](https://www.visualstudio.com/products/what-is-visual-studio-online-vs.aspx) ）到用于已连接资源（例如连接到 Web 应用的 MySQL 数据库）的管理工具。

### <a name="collect"></a> 2.收集数据
#### <a name="enable-diagnostics-logging-for-your-web-app"></a>为 Web 应用启用诊断日志记录
Web 应用环境为 Web 服务器和 Web 应用中的日志记录信息提供了诊断功能。 这些诊断功能按逻辑分为 Web 服务器诊断和应用程序诊断。

##### <a name="web-server-diagnostics"></a>Web 服务器诊断
你可以启用或禁用以下种类的日志：

* **详细错误日志记录** - 指示故障的 HTTP 状态代码（状态代码 400 或更大数字）的详细错误消息。 其中可能包含有助于确定服务器返回错误代码的原因的信息。
* **失败请求跟踪** - 有关失败请求的详细信息，包括对用于处理请求的 IIS 组件和每个组件所用的时间的跟踪。 在尝试提高 Web 应用性能或查找导致特定 HTTP 错误的问题时，此信息很有用。
* **Web 服务器日志记录** - 使用 W3C 扩展日志文件格式的 HTTP 事务信息。 这在确定总体 Web 应用指标（如处理的请求数量或来自特定 IP 地址的请求数）时非常有用。

##### <a name="application-diagnostics"></a>应用程序诊断
使用应用程序诊断可以捕获 Web 应用程序生成的信息。 ASP.NET 应用程序可使用 `System.Diagnostics.Trace` 类将信息记录到应用程序诊断日志。

有关如何在应用程序中配置日志记录的详细说明，请参阅[在 Azure 应用服务中启用 Web 应用的诊断日志记录](/documentation/articles/web-sites-enable-diagnostic-log/)。

#### <a name="use-remote-profiling"></a>使用远程分析
在 Azure App Service 中，可以远程分析 Web 应用、API 应用和 WebJob。 如果进程运行速度比预期缓慢，或者 HTTP 请求的延迟高于平时并且进程的 CPU 使用率偏高，则你可以远程分析进程并获取 CPU 采样调用堆栈，以分析进程活动和代码繁忙的路径。

有关详细信息，请参阅 [Azure 应用服务中的远程分析支持](https://azure.microsoft.com/zh-cn/blog/remote-profiling-support-in-azure-app-service)。

#### <a name="use-the-azure-app-service-support-portal"></a>使用 Azure App Service 支持门户
在 Web 应用中，可通过查看 HTTP 日志、事件日志、进程转储等信息来排查与 Web 应用相关的问题。 可以使用支持门户 (**http://&lt;your app name>.scm.chinacloudsites.cn/Support**) 访问所有这些信息

Azure App Service 支持门户具有三个不同的选项卡，用于支持常见故障排除方案的三个步骤：

1. 观察当前行为
2. 通过收集诊断信息和运行内置分析器进行分析
3. 缓解

如果目前正在发生问题，请单击“分析” > “诊断” > “立即诊断”创建诊断会话，该会话将收集 HTTP 日志、事件查看器日志、内存转储、PHP 错误日志和 PHP 进程报告。

完成数据收集后，该会话将对数据运行分析并提供一份 HTML 报告。

如果你想要下载数据，数据默认情况下会存储在 D:\home\data\DaaS 文件夹中。

有关 Azure 应用服务支持门户的详细信息，请参阅[用于支持 Azure 网站的站点扩展的最新更新](https://azure.microsoft.com/zh-cn/blog/new-updates-to-support-site-extension-for-azure-websites)。

#### <a name="use-the-kudu-debug-console"></a>使用 Kudu 调试控制台
Web Apps 随附可用于调试、浏览和上载文件的调试控制台，以及用于获取环境相关信息的 JSON 终结点。 此控制台称为 Web 应用的 *Kudu 控制台*或 *SCM 仪表板*。

转到链接 **https://&lt;Your app name>.scm.chinacloudsites.cn/** 即可访问此仪表板。

Kudu 提供的一些信息和功能包括：

* 应用程序的环境设置
* 日志流
* 诊断转储
* 调试控制台，你可以在其中运行 Powershell cmdlet 和基本 DOS 命令。

Kudu 的另一项有用功能是，如果应用程序引发第一次异常，你可以使用 Kudu 和 SysInternals 工具 Procdump 创建内存转储。 这些内存转储是进程的快照，通常可以帮助你排查较复杂的 Web 应用问题。

有关 Kudu 提供的功能的详细信息，请参阅 [Azure Websites Team Services tools you should know about](https://azure.microsoft.com/zh-cn/blog/windows-azure-websites-online-tools-you-should-know-about/)（你应该了解的 Azure Websites Team Services 工具）。

### <a name="mitigate"></a> 3.缓解问题
#### <a name="scale-the-web-app"></a>缩放 Web 应用
在 Azure 应用服务中，为了提高性能和吞吐量，可以调整运行应用程序的规模。 向上缩放 Web 应用涉及到两个相关操作：将 App Service 计划更改为较高的定价层，以及在切换到较高的定价层后配置特定的设置。

有关缩放的详细信息，请参阅[缩放 Azure 应用服务中的 Web 应用](/documentation/articles/web-sites-scale/)。

此外，你可以选择在多个实例上运行应用程序。 这不仅能提供更强大的处理能力，而且还能提供一定程度的容错。 如果进程在某个实例上中断，其他实例仍将继续处理请求。

可以将缩放设置为手动或自动。

#### <a name="use-autoheal"></a>使用 AutoHeal
AutoHeal 会根据你选择的设置（例如配置更改、请求、基于内存的限制或执行请求所需的时间），回收应用程序的工作进程。 在大多数情况下，回收进程是在出现问题后进行恢复的最快方式。 尽管始终可以从 Azure 门户预览直接重新启动 Web 应用，但 AutoHeal 可以自动为你执行此操作。 你只需在 Web 应用的根 web.config 中添加一些触发器即可。 请注意，即使你的应用程序并非 .Net 应用程序，这些设置的工作方式也仍然相同。

有关详细信息，请参阅 [自动修复 Azure 网站](https://azure.microsoft.com/zh-cn/blog/auto-healing-windows-azure-web-sites/)。

#### <a name="restart-the-web-app"></a>重新启动 Web 应用
这通常是在发生一次性问题后进行恢复的最简单方式。 [Azure 门户预览](https://portal.azure.cn/)上的 Web 应用边栏选项卡中提供了用于停止或重新启动应用的选项。

 ![重新启动 Web 应用以解决性能问题](./media/app-service-web-troubleshoot-performance-degradation/2-restart.png)

你还可以使用 Azure Powershell 管理 Web 应用。 有关详细信息，请参阅[将 Azure PowerShell 与 Azure Resource Manager 配合使用](/documentation/articles/powershell-azure-resource-manager/)。
<!--Update_Description: update to Azure portal preview-->