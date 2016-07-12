<properties
	pageTitle="监视 DocumentDB 请求和存储 | Azure"
	description="了解如何监视你的 DocumentDB 帐户的性能指标（如请求和服务器错误）以及使用情况指标（如存储消耗）。"
	services="documentdb"
	documentationCenter=""
	authors="mimig1"
	manager="jhubbard"
	editor="cgronlun"/>

<tags
	ms.service="documentdb"
	ms.date="06/10/2016"
	wacn.date="07/04/2016"/>

# 在 Azure 门户预览中监视 DocumentDB 帐户使用情况

你可以在 [Azure 门户预览](https://portal.azure.cn/)中监视 DocumentDB 帐户。对于每个 DocumentDB 帐户，性能指标（如请求和服务器错误）和使用情况指标（如存储消耗）都可用。

## 查看 DocumentDB 帐户的性能指标
1.	在 [Azure 门户预览](https://portal.azure.cn/)中，单击“浏览”、“DocumentDB 帐户”，然后单击要查看其性能指标的 DocumentDB 帐户的名称。
2.	默认情况下，在“监视”可重用功能区中可以看到如下内容：
	*	当天的请求总数。
	*	该帐户的存储使用情况。

	如果你的表显示**无可用数据**，请参阅[疑难解答](#troubleshooting)部分。

	![“监视”可重用功能区的屏幕截图，它显示当天的请求总数和存储使用情况](./media/documentdb-monitor-accounts/documentdb-total-requests-and-usage.png)


3.	单击“请求总数”或“存储”磁贴将打开一个详细的“指标”边栏选项卡。
4.	“指标”边栏选项卡显示有关所选指标的详细信息。边栏选项卡的顶部有一个图形，该图形的下方有一个表，其中显示了所选指标的聚合值，例如平均值、最小值、最大值和总数。指标边栏选项卡还显示警报列表，这些警报已经定义，且根据当前指标边栏选项卡上显示的指标进行了筛选（如此，如果你的警报数量较多，将只能在此处看到相关的警报）。   

	![“指标”边栏选项卡的屏幕截图](./media/documentdb-monitor-accounts/documentdb-metric-blade.png)


## 自定义 DocumentDB 帐户的性能指标视图

1.	若要自定义显示在特定部件中的指标，右键单击该指标图表，然后选择“编辑图表”。  
	![“编辑图表”按钮突出显示的请求总数图表的屏幕截图](./media/documentdb-monitor-accounts/madocdb3.png)

2.	在“编辑图表”边栏选项卡中，有选项可用于修改显示在该部件中的指标，以及它们的时间范围。  
	![“编辑图表”边栏选项卡的屏幕截图](./media/documentdb-monitor-accounts/madocdb4.png)

3.	若要更改显示在该部件中的指标，只需选择或清除可用的性能指标，然后单击边栏选项卡底部的“保存”。
4.	若要更改时间范围，请选择一个不同的范围（例如，“自定义”），然后单击边栏选项卡底部的“保存”。  

	![“编辑图表”边栏选项卡的“时间范围”部件的屏幕截图，显示如何输入自定义时间范围](./media/documentdb-monitor-accounts/madocdb5.png)


## 创建并排性能指标图表
Azure 门户预览使你能够创建并排的指标图表。

1.	首先，请右键单击你想要克隆和修改的图表，然后选择“自定义”。

	![“自定义”选项突出显示的请求总数图表的屏幕截图](./media/documentdb-monitor-accounts/madocdb6.png)

2.	请单击菜单上的“克隆”以复制部件，然后单击“完成自定义”。

	![“克隆”和“完成自定义”选项突出显示的请求总数图表的屏幕截图](./media/documentdb-monitor-accounts/madocdb7.png)


你现在可能将此部件视为其他任何指标部件，同时自定义显示在部件中的指标和时间范围。通过执行此操作，你可以同时看到两个并排的不同的指标图表。  
	![请求总数图表和过去一小时新的请求总数的屏幕截图](./media/documentdb-monitor-accounts/madocdb8.png)

## 设置 DocumentDB 帐户性能指标警报
1.	在 [Azure 门户预览](https://portal.azure.cn/)中，单击“浏览全部”、“DocumentDB 帐户”，然后单击要为其设置性能指标警报的 DocumentDB 帐户的名称。

2.	如果“所有设置”边栏选项卡未打开，请单击左侧顶部的“设置”命令以打开边栏选项卡。
	![“数据库帐户”边栏选项卡上“设置”命令的屏幕截图](./media/documentdb-monitor-accounts/madocdb10.png)

3.	在“所有设置”边栏选项卡中，单击“警报规则”以打开警报规则边栏选项卡。  
	![所选的警报规则部件的屏幕截图](./media/documentdb-monitor-accounts/madocdb10.5.png)

4.	在警报规则边栏选项卡中，单击“添加警报”。  
	![“添加警报”按钮突出显示的“警报规则”边栏选项卡的屏幕截图](./media/documentdb-monitor-accounts/madocdb11.png)

5.	在“添加警报规则”边栏选项卡中，请指定：
	*	你正在设置的警报规则的名称。
	*	新的警报规则的说明。
	*	警报规则指标。
	*	确定何时激活警报的条件、阈值和时间段。例如，在过去的 15 分钟服务器错误计数大于 5。
	*	当警报被触发时，服务管理员和协同管理员是否将通过电子邮件得到通知。
	*	警报通知的其他电子邮件地址。  
	![“添加警报规则”边栏选项卡的屏幕截图](./media/documentdb-monitor-accounts/madocdb12.png)

## 监视 DocumentDB 的其他方法
门户预览中可用的帐户级别指标（如帐户存储使用情况和请求总数）不可通过 DocumentDB API 使用。但是，你可以以集合级别检索使用情况数据。若要检索集合级别的数据，请执行以下操作：

- 若要使用 REST API，请[在集合上执行 GET](https://msdn.microsoft.com/library/mt489073.aspx)。集合的配额和使用情况信息将返回到响应中的 x-ms-resource-quota 和 x-ms-resource-usage 标头中。
- 若要使用 .NET SDK，请使用 [DocumentClient.ReadDocumentCollectionAsync](https://msdn.microsoft.com/library/microsoft.azure.documents.client.documentclient.readdocumentcollectionasync.aspx) 方法，它将返回 [ResourceResponse](https://msdn.microsoft.com/library/dn799209.aspx)，其包含大量使用情况属性，例如 **CollectionSizeUsage**、**DatabaseUsage**、**DocumentUsage** 等。

若要访问其他指标，请使用 [Azure Insights SDK](https://www.nuget.org/packages/Microsoft.Azure.Insights)。可以通过调用以下命令来检索可用的指标定义：

    https://management.azure.com/subscriptions/{SubscriptionId}/resourceGroups/{ResourceGroup}/providers/Microsoft.DocumentDb/databaseAccounts/{DocumentDBAccountName}/metricDefinitions?api-version=2015-04-08 

用于检索各个指标的查询使用以下格式：

    https://management.azure.com/subscriptions/{SubecriptionId}/resourceGroups/{ResourceGroup}/providers/Microsoft.DocumentDb/databaseAccounts/{DocumentDBAccountName}/metrics?api-version=2015-04-08&$filter=%28name.value%20eq%20%27Total%20Requests%27%29%20and%20timeGrain%20eq%20duration%27PT5M%27%20and%20startTime%20eq%202016-06-03T03%3A26%3A00.0000000Z%20and%20endTime%20eq%202016-06-10T03%3A26%3A00.0000000Z

有关详细信息，请参阅 [Retrieving Resource Metrics via the Azure Insights API（通过 Azure Insights API 检索资源指标）](https://blogs.msdn.microsoft.com/cloud_solution_architect/2016/02/23/retrieving-resource-metrics-via-the-azure-insights-api/)。

## 故障排除
如果你的监视磁贴显示**无可用数据**消息，并且你最近向数据库提出过请求或添加过数据，则可以编辑该磁贴以反映最新的使用情况。

### 编辑磁贴以刷新当前数据
1.	若要自定义显示在特定部件中的指标，右键单击该指标图表，然后选择“编辑图表”。  
    ![表明无数据可用的“请求总数”磁贴的屏幕截图](./media/documentdb-monitor-accounts/documentdb-no-available-data.png)

2.	在“编辑图表”边栏选项卡上，在“时间范围”部分中，单击“过去一小时”，然后单击“保存”。  
	![选择了过去一个小时的“编辑图表”边栏选项卡的屏幕截图](./media/documentdb-monitor-accounts/documentdb-no-available-data-past-hour.png)


3.	现在你的磁贴应该刷新以向你显示当前数据和使用情况。  
	![更新后的过去一小时请求总数磁贴的屏幕截图](./media/documentdb-monitor-accounts/documentdb-no-available-data-fixed.png)

## 后续步骤
若要了解有关 DocumentDB 容量的详细信息，请参阅[管理 DocumentDB 容量](/documentation/articles/documentdb-manage/)。

<!---HONumber=Mooncake_0627_2016-->