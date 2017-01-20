<properties
	pageTitle="缩放实例计数 | Azure"
	description="了解如何在 Azure 中缩放服务。"
	authors="rboucher"
	manager=""
	editor=""
	services="monitoring-and-diagnostics"
	documentationCenter="monitoring-and-diagnostics"/>

<tags
	ms.service="monitoring-and-diagnostics"
	ms.workload="na"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="09/08/2015"
	wacn.date="11/14/2016"
	ms.author="robb"/>

# 缩放实例计数

在 [Azure 门户预览](https://portal.azure.cn/)中，你可以手动设置服务的实例计数。这通常称为*扩大*或*缩小*。

基于实例计数进行缩放之前，应考虑的除了实例计数之外，缩放还会受**定价层**影响。不同定价层可以具有不同数量的核心和内存，因此它们对于相同数量的实例具有更佳性能（即*增加* 或*减少*）。本文专门介绍*缩小* 和*扩大*。

可以在门户中进行缩放，也可以使用 [REST API](https://msdn.microsoft.com/zh-cn/library/azure/dn931953.aspx) 或 [.NET SDK](https://www.nuget.org/packages/Microsoft.Azure.Insights/) 手动或自动调整规模。

> [AZURE.NOTE] 本文介绍了如何在位于 [http://portal.azure.cn](http://portal.azure.cn) 的门户预览中创建手动缩放设置，门户预览尚不支持创建自动缩放，需要创建自动缩放的用户要在 Azure 经典管理门户 ([http://manage.windowsazure.cn](http://manage.windowsazure.cn)) 中进行设置，但 Azure 经典管理门户及其基础后端具有限制。。

## 手动缩放

1. 在 [Azure 门户预览](https://portal.azure.cn/)中，单击“浏览”，然后导航到要缩放的资源（如“应用服务计划”）。

2. 点击 “应用服务计划”中的“扩大（应用服务计划）”磁贴来设置手动缩放实例计数。

    ![“缩放”分页](./media/insights-how-to-scale/Insights_ScaleBladeDayZero.png)
    
4. 可以使用滑块手动调整“实例”数。
5. 单击“保存”命令，几乎可以立即缩放到该实例数。


## 后续步骤

* [监视服务指标](/documentation/articles/insights-how-to-customize-monitoring/)以确保你的服务可用且响应迅速。
* [启用监视和诊断](/documentation/articles/insights-how-to-use-diagnostics/)以收集你的服务的详细高频指标。
* 每当操作事件发生或指标超过阈值时[接收警报通知](/documentation/articles/insights-receive-alert-notifications/)。
* 在要确切了解代码在云中的执行情况时[监视应用程序性能](/documentation/articles/insights-perf-analytics/)
* [查看事件和审核日志](/documentation/articles/insights-debugging-with-events/)以了解在服务中发生的所有事件。

 
<!---HONumber=Mooncake_1107_2016-->