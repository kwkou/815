<properties
    pageTitle="Azure Monitor 概述 | Azure"
    description="Azure Monitor 收集在警报、webhook、自动缩放和自动化中使用的统计信息。 本文还列出了其他 Microsoft 监视选项。"
    author="rboucher"
    manager="carmonm"
    editor=""
    services="monitoring-and-diagnostics"
    documentationcenter="monitoring-and-diagnostics"
    translationtype="Human Translation" />
<tags
    ms.assetid="1b962c74-8d36-4778-b816-a893f738f92d"
    ms.service="monitoring-and-diagnostics"
    ms.workload="na"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="03/31/2017"
    wacn.date="05/02/2017"
    ms.author="robb"
    ms.sourcegitcommit="78da854d58905bc82228bcbff1de0fcfbc12d5ac"
    ms.openlocfilehash="e72d762ec776b08deb6ca4f9845287192096d62a"
    ms.lasthandoff="04/22/2017" />

# <a name="overview-of-azure-monitor"></a>Azure Monitor 概述
本文提供有关监视 Azure 资源的概念性概述， 并提供有关特定类型资源的信息的导航。如需要了解从非 Azure 角度监视应用程序的信息，请参阅[监视和诊断指南](/documentation/articles/best-practices-monitoring/)。


云应用程序很复杂，包含很多移动部件。 监视可以为用户提供数据，确保应用程序始终处于健康运行状态。 监视还有助于避免潜在问题，或者解决过去的问题。 此外，还可以利用监视数据深入了解应用程序的情况。 这些解析可帮助提升应用程序性能或维护性，或者将某些原本需要手动介入的操作自动化。

下图从概念方面介绍了 Azure 监视，包括可以收集的日志类型，以及可以对该数据执行的操作。   

![非计算资源的监视和诊断模型](./media/monitoring-overview/Monitoring_Azure_Resources-compute_v4.png)

<br/>

![计算资源的监视和诊断模型](./media/monitoring-overview/Monitoring_Azure_Resources-non-compute_v4.png)


## <a name="monitoring-sources"></a>监视源
### <a name="activity-logs"></a>活动日志
可以搜索活动日志（以前称为操作日志或审核日志）中是否存在通过 Azure 基础结构查看的资源的相关信息。 日志包含多种信息，例如创建或销毁资源的时间。  

### <a name="host-vm"></a>主 VM
**仅计算**

某些计算资源（例如云服务、虚拟机和 Service Fabric）有一个与之交互的专用主 VM。 主 VM 等同于 Hyper-V 虚拟机监控程序模型中的根 VM。 在这种情况下，除了来宾 OS 以外，可以只收集主机 VM 上的指标。  

对于其他 Azure 服务，在用户的资源和特定主 VM 之间没有必然的 1:1 映射关系，因此不提供主机 VM 指标。

### <a name="resource---metrics-and-diagnostics-logs"></a>资源 - 指标和诊断日志
可收集的指标取决于资源类型。 例如，虚拟机提供有关磁盘 IO 和 CPU 百分比的统计信息。 但这些统计信息对于服务总线队列来说并不存在，后者提供的是队列大小和消息吞吐量之类的指标。

对于计算资源来说，用户可以获取有关来宾 OS 和诊断模块（例如 Azure 诊断）的指标。 Azure 诊断有助于收集诊断数据，并可将诊断数据路由到其他位置（包括 Azure 存储）。


### <a name="application---diagnostics-logs-application-logs-and-metrics"></a>应用程序 - 诊断日志、应用程序日志和指标
**仅计算**

在计算模型中，应用程序可以基于来宾 OS 运行。 应用程序会发出自己的日志和指标集。

指标类型包括

* 性能计数器
* 应用程序日志
* Windows 事件日志
* .NET 事件源
* IIS Logs
* 基于清单的 ETW
* 故障转储
* 客户错误日志

## <a name="uses-for-monitoring-data"></a>用于监视数据
### <a name="visualize"></a>可视化
以图形和表格的形式将监视数据可视化可以更快地找出其中的趋势，其速度远非单纯查看数据可比。  

可视化方法包括：

* 使用 Azure 门户预览
* 将数据路由到 Microsoft PowerBI
* 将数据路由到第三方可视化工具，可以使用实时传送视频流，也可以让工具从 Azure 存储中的存档读取。

### <a name="archive"></a>存档
监视数据通常写入 Azure 存储并保存在那里，直至用户将其删除。

可以通过多种方式使用该数据：

* 写入以后，即可让 Azure 内外的其他工具读取和处理该数据。
* 可以将该数据下载到本地进行本地存档，也可以更改云中的保留策略，让数据保留更长的时间。  
* <a name="you-leave-the-data-in-azure-storage-indefinitely-though-you-have-to-pay-for-azure-storage-based-on-the-amount-of-data-you-keep"></a>可以无限期地将数据保留在 Azure 存储中，不过，这需要根据保留的数据量支付 Azure 存储费用。

### <a name="query"></a>查询
可以使用 Azure 监视器 REST API、跨平台命令行接口 (CLI) 命令、PowerShell cmdlet 或 .NET SDK 访问系统或 Azure 存储中的数据

示例包括：

* 获取所编写的自定义监视应用程序的数据
* 创建自定义查询，将该数据发送到第三方应用程序。

### <a name="route"></a>路由
可以将监视数据实时流式传输到其他位置。

示例包括：

* 发送到事件中心，以便将其路由到第三方工具执行实时分析。

### <a name="automate"></a>自动化
可以使用监视数据触发警报或甚至整个过程。
示例包括：

* 使用数据根据应用程序负载自动缩放计算实例数。
* 当某个指标超出预定阈值时发送电子邮件。
* 调用 Web URL (webhook)，在 Azure 外部系统中执行操作。
* 在 Azure 自动化中启动 Runbook，执行各种任务

## <a name="methods-of-use"></a>使用方法
一般情况下，可以使用下述方法之一操作数据的跟踪、路由和检索。 并非所有方法都适用于所有操作或数据类型。

* [Azure 门户预览](https://portal.azure.cn)
* [PowerShell](/documentation/articles/insights-powershell-samples/)  
* [跨平台的命令行接口 (CLI)](/documentation/articles/insights-cli-samples/)
* [REST API](https://msdn.microsoft.com/zh-cn/library/dn931943.aspx)
* [.NET SDK](https://msdn.microsoft.com/zh-cn/library/dn802153.aspx)

## <a name="azures-monitoring-offerings"></a>Azure 的监视产品/服务
Azure 的产品/服务可用于监视从裸机基础结构到应用程序遥测在内的各种服务。 最佳监视策略综合使用了所有这三种方式，可以对服务的运行状况进行全面、细致的了解。

* [Azure Monitor](http://aka.ms/azmondocs) — 对来自 Azure 基础结构（活动日志）和每个单独的 Azure 资源（诊断日志）的数据提供可视化、查询、路由、警报、自动调整规模和自动化功能。 本文是 Azure Monitor 文档的一部分。 “Azure 监视器”名称在 2016 年 9 月 25 日于 Ignite 发布。  以前的名称为“Azure Insights”。  

## <a name="next-steps"></a>后续步骤
详细了解以下内容

* [Ignite 2016 的 Azure Monitor 视频](https://myignite.microsoft.com/videos/4977)
* [Azure 存储故障诊断](/documentation/articles/storage-e2e-troubleshooting/) 在使用存储 Blob、表或队列的情况下。