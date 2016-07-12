<properties 
	pageTitle="跟踪服务运行状况" 
	description="在 Azure 遇到性能下降或服务中断及时发现。" 
	authors="stepsic-microsoft-com" 
	manager="kamrani" 
	editor="" 
	services="azure-portal" 
	documentationCenter="na"/>

<tags 
	ms.service="azure-portal" 
	ms.date="09/08/2015"
	wacn.date="05/09/2016"/>

# 跟踪服务运行状况

每次发生服务中断或性能下降时 Azure 会进行宣传。可以在 Azure 门户中浏览这些事件，也可以使用 [REST API](https://msdn.microsoft.com/zh-cn/library/azure/dn931927.aspx) 或 [.NET SDK](https://www.nuget.org/packages/Microsoft.Azure.Insights/) 以编程方式访问完整的事件集。

## 按区域查看服务运行状况

1. 登录到 [Azure 门户预览](https://portal.azure.cn/)。

2. 在**主页**中，你会看到名为**服务运行状况**的磁贴

    ![主页](./media/insights-service-health/Insights_Home.png)

3. 单击该磁贴会在 Azure 中显示所有区域的列表。可以单击任何区域以查看该区域的服务运行状况历史记录。

    ![主页](./media/insights-service-health/Insights_Regions.png)

4. 还可以通过单击表中的事件查看各个事件的详细信息。

## 浏览完整的服务运行状况日志

2. 单击“浏览”，然后选择“审核日志”。  

    ![浏览中心](./media/insights-service-health/Insights_Browse.png)

3. 这时，将打开一个边栏选项卡，其中显示过去 7 天影响你的任何订阅的所有事件。此列表中将显示服务运行状况条目，但可能很难找到它们，因为列表可能非常大。

4. 单击**筛选器**命令。

5. 选择**事件类别**，然后选择**服务运行状况**：

    ![所有事件](./media/insights-service-health/Insights_Filter.png)

6. 单击**更新**。

7. 现在，可以看到影响你的订阅的所有服务运行状况事件︰

    ![资源组](./media/insights-service-health/Insights_HealthEvent.png)

8. 从这里可以转到详细信息边栏选项卡，以查看该事件的具体细节。
   
## 后续步骤

* 每当发生事件时[接收警报通知](/documentation/articles/insights-receive-alert-notifications/)。
* [监视服务指标](/documentation/articles/insights-how-to-customize-monitoring/)以确保你的服务可用且响应迅速。
 
<!---HONumber=Mooncake_0503_2016-->