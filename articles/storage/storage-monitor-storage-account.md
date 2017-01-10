<properties
    pageTitle="如何监视存储帐户 | Azure"
    description="了解如何使用 Azure 门户在 Azure 中监视存储帐户。"
    services="storage"
    documentationcenter=""
    author="robinsh"
    manager="timlt"
    editor="tysonn" />
<tags
    ms.assetid="b83cba7b-4627-4ba7-b5d0-f1039fe30e78"
    ms.service="storage"
    ms.workload="storage"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="12/08/2016"
    wacn.date="01/06/2017"
    ms.author="robinsh" />

# 在 Azure 门户预览中监视存储帐户
## 概述

可在 [Azure 门户预览](https://portal.azure.cn)中监视存储帐户。配置存储帐户以便通过门户进行监视时，Azure 存储使用[存储分析](http://msdn.microsoft.com/zh-cn/library/azure/hh343270.aspx)跟踪帐户和日志请求数据的指标。

> [AZURE.NOTE] 在 [Azure 门户预览](https://portal.azure.cn)中查看监视数据会产生相关的额外费用。有关详细信息，请参阅<a href="http://msdn.microsoft.com/zh-cn/library/azure/hh360997.aspx">存储分析和计费</a>。<br />

> Azure 文件存储目前支持存储分析指标，但尚不支持日志记录。可以通过 [Azure 门户预览](https://portal.azure.cn)为 Azure 文件存储启用指标。

> 有关使用存储分析及其他工具来识别、诊断和排查 Azure 存储相关问题的深入指导，请参阅[监视、诊断和排查 Azure 存储问题](/documentation/articles/storage-monitoring-diagnosing-troubleshooting/)。


## 如何：为存储帐户配置监视

1. 在 [Azure 门户预览](https://portal.azure.cn/)中，单击“存储”，然后单击存储帐户名称以打开仪表板。

2. 单击“配置”，然后向下滚动到 Blob、表和队列服务的“监视”设置。

	![MonitoringOptions](./media/storage-monitor-storage-account/Storage_MonitoringOptions.png)

3. 在“监视”中，为每项服务设置监视级别和数据保留策略：

	-  要设置监视级别，请选择以下一项：

      **最低** - 收集经过汇总的有关 Blob、表和队列服务的入口/出口、可用性、延迟及成功百分比等指标。

      **详细** – 除最低监视指标外，在 Azure 存储服务 API 中为每项存储操作收集一组相同的指标。通过详细监视度量值可对应用程序运行期间出现的问题进行进一步分析。

      **关闭** - 关闭监视。现有监视数据将一直保留到保留期结束。

- 要设置数据保留策略，请在“保留期(天)”中，键入要保留数据的天数，范围介于 1 到 365 天之间。如果不需要设置保留策略，请输入零。如果没有保留策略，则由用户自行决定是否删除监视数据。建议根据要保留帐户的存储分析数据的时间来设置保留策略，以便由系统免费删除旧数据和未使用的分析数据。

4. 完成监视配置后，单击“保存”。

大约一小时后，应可开始在仪表板和“监视”页上看到监视数据。

为存储帐户配置监视之前，不会收集任何监视数据，仪表板和“监视”页上的指标图表为空。

设置监视级别和保留策略后，可选择要在 [Azure 门户预览](https://portal.azure.cn)中监视的可用指标，以及要在指标图表上显示的指标。将在每个监视级别显示一组默认指标。可以使用“添加指标”在指标列表中添加或删除指标。

指标存储在存储帐户中的以下 4 个表中：$MetricsTransactionsBlob、$MetricsTransactionsTable、$MetricsTransactionsQueue 和 $MetricsCapacityBlob。有关详细信息，请参阅[关于存储分析指标](http://msdn.microsoft.com/zh-cn/library/azure/hh343258.aspx)。


## 如何：针对监视功能自定义仪表板

在仪表板上，可以从 9 个可用指标中最多选择 6 个要在指标图表上显示的指标。对于每项服务（Blob、表和队列），“可用性”、“成功百分比”和“请求总数”均可供选择。对于最低监视或详细监视，仪表板上提供的指标是相同的。

1. 在 [Azure 门户预览](https://portal.azure.cn)中，单击“存储”，然后单击存储帐户名称以打开仪表板。

2. 要更改图表上显示的指标，请执行以下操作之一：

	- 要向图表添加新指标，请单击该图表下方表中指标标题旁的彩色复选框。

	- 要隐藏显示在图表上的某个指标，请清除该指标标题旁的彩色复选框。

		![Monitoring\_nmore](./media/storage-monitor-storage-account/storage_Monitoring_nmore.png)

3. 默认情况下，该图表显示趋势，以便仅显示每个指标的当前值（选择图表顶部的“相对”选项）。要显示 Y 轴以便能够看到绝对值，请选择“绝对”。

4. 要更改指标图表显示的时间范围，请在图表顶部选择 6 小时、24 小时或者 7 天。


## 如何：自定义“监视”页

在“监视”页上，可以查看存储帐户的一组完整指标。

- 如果存储帐户已配置最低监视，将汇总有关 Blob、表和队列服务的入口/出口、可用性、延迟及成功百分比等指标。

- 如果存储帐户已配置详细监视，那么除了服务级别的汇总之外，还将提供各项存储操的更精细解析。

使用以下过程选择要在指标图表中查看的存储指标和要在“监视”页上显示的表。这些设置不会影响存储帐户下监视数据的收集、汇总和存储。

##<a name="how-to-add-metrics-to-the-metrics-table"></a> 如何：向指标表中添加指标


1. 在 [Azure 门户预览](https://portal.azure.cn/)中，单击“存储”，然后单击存储帐户名称以打开仪表板。

2. 单击“监视”。

	此时将打开“监视”页。默认情况下，指标表显示可监视指标的子集。下图中显示的是为所有三项服务配置了详细监视的存储帐户的默认“监视”视图。可使用“添加指标”从所有可用指标中选择要监视的指标。

	![Monitoring\_VerboseDisplay](./media/storage-monitor-storage-account/Storage_Monitoring_VerboseDisplay.png)

	> [AZURE.NOTE] 选择指标时应考虑成本。在刷新监视视图时会产生相关的事务和数据传出费用。有关详细信息，请参阅[存储分析和计费](http://msdn.microsoft.com/zh-cn/library/azure/hh360997.aspx)。

3. 单击“添加指标”。

	最低监视中可用的汇总指标位于列表顶部。如果选中该复选框，则指标将显示在指标列表中。

	![AddMetricsInitialDisplay](./media/storage-monitor-storage-account/Storage_AddMetrics_InitialDisplay.png)

4. 将鼠标指针悬停于对话框右侧时会显示滚动条，可拖动它以将其他指标滚动到视野中。

	![AddMetricsScrollbar](./media/storage-monitor-storage-account/Storage_AddMetrics_Scrollbar.png)


5. 单击指标旁边的向下箭头可展开该指标范围内所含操作的列表。选择要在 [Azure 门户预览](https://portal.azure.cn)中的指标表中查看的每项操作。

	下图中展开了“授权错误百分比”指标。

	![ExpandCollapse](./media/storage-monitor-storage-account/Storage_AddMetrics_ExpandCollapse.png)


6. 为所有服务选择指标后，单击“确定”（复选标记）以更新监视配置。所选指标将添加到指标表中。

7. 要从指标表中删除某个指标，请单击以选中该指标，然后单击“删除指标”。

	![DeleteMetric](./media/storage-monitor-storage-account/Storage_DeleteMetric.png)

## 如何：在“监视”页上自定义指标图表

1. 在存储帐户的“监视”页上的指标表中，最多可选择 6 个要在指标图表上显示的指标。要选择指标，请单击其左侧的复选框。要从图表中删除指标，请清除复选框。

2. 要在相对值（仅显示最终值）和绝对值（显示 Y 轴）之间切换图表，请选择图表顶部的“相对”或“绝对”。

3.	要更改指标图表显示的时间范围，请在图表顶部选择“6 小时”、“24 小时”或者“7 天”。



## 如何：配置日志记录

对于存储帐户中提供的每项存储服务（Blob、表和队列），可保存“读取请求”、“写入请求”和/或“删除请求”的诊断日志，并可为其中每项服务设置数据保留策略。

1. 在 [Azure 门户预览](https://portal.azure.cn/)中，单击“存储”，然后单击存储帐户名称以打开仪表板。

2. 单击“配置”，然后使用键盘上的向下箭头向下滚动到“日志记录”。

	![Storagelogging](./media/storage-monitor-storage-account/Storage_LoggingOptions.png)


3. 为每项服务（Blob、表和队列）配置下列内容：

	- 要记录的请求类型：“读取请求”、“写入请求”和“删除请求”。

	- 保留记录数据的天数。如果不需要设置保留策略，请输入零。如果不设置保留策略，则由用户自行决定是否删除日志。

4. 单击“保存”。

诊断日志保存在存储帐户下名为 $logs 的 Blob 容器中。有关访问 $logs 容器的信息，请参阅[关于存储分析日志记录](http://msdn.microsoft.com/zh-cn/library/azure/hh343262.aspx)。

<!---HONumber=Mooncake_0103_2017-->