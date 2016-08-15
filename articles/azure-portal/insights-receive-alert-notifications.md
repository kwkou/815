<properties 
	pageTitle="接收警报通知" 
	description="在满足警报规则条件时收到通知。" 
	authors="stepsic-microsoft-com" 
	manager="ronmart" 
	editor="" 
	services="azure-portal" 
	documentationCenter="na"/>

<tags 
	ms.service="azure-portal" 
	ms.date="09/08/2015" 
	wacn.date="05/09/2016"/>

# 接收警报通知

可以根据监视指标或事件接收 Azure 服务的警报。

对于指标值的警报规则，当指定指标的值超过分配的阈值时，警报规则会变为活动状态并发送一条通知。对于事件的警报规则，规则可以针对每个事件发送通知，或仅当发生一定数量的事件时发送通知。

在创建警报规则时，你可以选择向服务管理员和协同管理员或向你可以指定的另一位系统管理员发送电子邮件通知的选项。当规则变为活动状态以及当警报条件满足时发送通知电子邮件。

可以使用 [REST API](https://msdn.microsoft.com/zh-cn/library/azure/dn931945.aspx) 或 [.NET SDK](https://www.nuget.org/packages/Microsoft.Azure.Insights/) 来配置警报规则并以编程方式获取相关信息。

## 创建警报规则

1. 在[Azure 门户预览](https://portal.azure.cn/)中，请单击**浏览**，然后单击想要监视的资源。

2. 单击**操作**可重用功能区中的**警报规则**磁贴。

3. 单击**添加通知**命令。

    ![添加警报](./media/insights-receive-alert-notifications/Insights_AddAlert.png)

4. 可以命名警报规则，并选择显示在通知电子邮件中的说明。

5. 当选择**指标**时，将为该指标选择一个条件和阈值。这是 Azure 用于监视并显示警报活动的时间段。

    ![条件和阈值](./media/insights-receive-alert-notifications/Insights_ConditionAndThreshold.png)

6. 还可以选择**事件**，并在某个事件发生时获得通知。 

    ![事件](./media/insights-receive-alert-notifications/Insights_Events.png)

7. 最后，可以选择向责任管理员发送电子邮件通知。

单击**保存**后，每当选择的指标超出阈值时，你都可以在几分钟内获知。

## 管理你的警报规则

创建警报规则后，可以查看与前一天指标相比的警报阈值预览。

![事件](./media/insights-receive-alert-notifications/Insights_EditAlert.png)


当然可以编辑此警报规则，如果想要暂时停止接收有关通知，可以**禁用**或**启用**它。

## 后续步骤

* [为你的警报配置 webhook](/documentation/articles/insights-webhooks-alerts/) 可将通知路由到各个通道
* [监视服务指标](/documentation/articles/insights-how-to-customize-monitoring/)以确保你的服务可用且响应迅速。
* [启用监视和诊断](/documentation/articles/insights-how-to-use-diagnostics/)以收集有关服务的详细高频率指标。
* 在要确切了解代码在云中的执行情况时[监视应用程序性能](/documentation/articles/insights-perf-analytics/)。
* [查看事件并审核日志](/documentation/articles/insights-debugging-with-events/)以了解在服务中发生的所有事件。
* [跟踪服务运行状况](/documentation/articles/insights-service-health/)以在 Azure 遇到性能下降或服务中断时及时发现。
 
<!---HONumber=Mooncake_0503_2016-->