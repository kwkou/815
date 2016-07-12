<properties 
	pageTitle="查看事件和审核日志" 
	description="了解如何查看在你的 Azure 订阅中发生的所有事件。" 
	authors="HaniKN-MSFT" 
	manager="kamrani" 
	editor="" 
	services="azure-portal" 
	documentationCenter="na"/>

<tags 
	ms.service="azure-portal" 
	ms.date="04/28/2015" 
	wacn.date="05/09/2016"/>

# 查看事件和审核日志

对 Azure 资源执行的所有操作都会由 Azure Resource Manager 进行完全审核（从创建和删除到授予或撤消访问权限）。可以在 Azure 门户预览中浏览这些日志，也可以使用 [REST API](https://msdn.microsoft.com/zh-cn/library/azure/dn931927.aspx) 或 [.NET SDK](https://www.nuget.org/packages/Microsoft.Azure.Insights/) 以编程方式访问完整的事件集。

## 浏览影响 Azure 订阅的事件

1. 登录到 [Azure 门户预览](https://portal.azure.cn/)。
2. 单击“浏览”，然后选择“审核日志”。  

    ![浏览中心](./media/insights-debugging-with-events/Insights_Browse.png)
    
3. 这时，将打开一个边栏选项卡，其中显示过去 7 天影响你的任何订阅的所有事件。顶部是一个按级别显示数据的图表，下部是日志的完整列表：

    ![所有事件](./media/insights-debugging-with-events/Insights_AllEvents.png)

    >[AZURE.NOTE] 在 Azure 门户预览中只能查看给定订阅的 500 个最新事件。

4. 可以单击任何日志条目来查看组成它的事件。例如，向资源组部署某种内容时，可能会创建或修改许多不同的资源。对于每个条目，可以查看：
    * 事件的“级别”— 例如，可能只是要跟踪的某种内容（“信息”），或是当出现了需要了解的错误内容时（“错误”）。 
    * “状态”— 最终状态通常是“成功”或“失败”，但对于长时间运行的操作也可能是“已接受”。
    * 事件发生的“时间”。
    * 执行操作的“人员”（如果有人）。并非所有操作都由用户执行，某些操作由后端服务执行，因此它们没有“调用方”。
    * 事件的“相关 ID”— 这是此操作集的唯一标识符。

5. 从这里可以转到详细信息边栏选项卡，以查看该事件的具体细节。
   
    ![资源组](./media/insights-debugging-with-events/Insights_EventDetails.png)

    对于“失败”事件，此页通常会显示“子状态”和“属性”部分，其中包含了可用于调试的详细信息。

## 筛选到特定日志

若要查看适用于特定实体或属于特定类型的事件，可以通过单击“筛选器”命令来筛选审核日志边栏选项卡。还可使用筛选器边栏选项卡更改审核记录边栏选项卡的“时间跨度”。

单击此命令之后，会打开一个新边栏选项卡：

![筛选器](./media/insights-debugging-with-events/Insights_EventFilter.png)

有四种类型的筛选器：

1. 按订阅
2. 按“资源组”
3. 按“资源类型”
4. 按特定“资源”— 对于此类型，必须传入感兴趣的资源的完整资源 ID

此外，还可以按执行事件的人员或者按事件级别来筛选事件。

选择完要查看的内容之后，单击边栏选项卡底部的“更新”按钮。

## 监视影响特定资源的事件

1. 单击“浏览”以查找感兴趣的资源。还可以查看整个“资源组”的所有日志。
2. 在资源的边栏选项卡上，向下滚动，直到找到“事件”磁贴。  

    ![“事件”磁贴](./media/insights-debugging-with-events/Insights_EventsTile.png)
    
3. 单击该磁贴以查看仅筛选到所选资源的事件。可以使用“筛选器”命令更改时间范围或应用更多特定筛选器。

## 后续步骤

* 每当发生事件时[接收警报通知](/documentation/articles/insights-receive-alert-notifications/)。
* [监视服务指标](/documentation/articles/insights-how-to-customize-monitoring/)以确保你的服务可用且响应迅速。
* [跟踪服务运行状况](/documentation/articles/insights-service-health/)以在 Azure 遇到性能下降或服务中断时及时发现。  


<!---HONumber=Mooncake_0503_2016-->