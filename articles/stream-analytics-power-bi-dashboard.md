<properties 
	pageTitle="流分析的上的 Power BI 仪表板 | Windows Azure" 
	description="使用实时流式处理 Power BI 仪表板来采集商业智能信息，并分析流分析作业中的大量数据。" 
	keywords="business intelligence tools,power bi,streaming data,power bi dashboard"	
	services="stream-analytics" 
	documentationCenter="" 
	authors="jeffstokes72" 
	manager="paulettm" 
	editor="cgronlun"/>

<tags 
	ms.service="stream-analytics" 
	ms.date="08/19/2015" 
	wacn.date="09/15/2015"/>
	
# Azure 流分析和 Power BI：用于对流式数据进行实时分析的实时仪表板

Azure 流分析允许你利用领先的商业智能工具 Microsoft Power BI。了解如何使用 Azure 流分析来分析大量流式数据，以便在实时 Power BI 仪表板中获得相关见解。

使用 [Microsoft Power BI](https://powerbi.com/) 快速构建实时仪表板。[观看演示本方案的视频](https://www.youtube.com/watch?v=SGUpT-a99MA)。

在本文中，你将了解如何使用 Power BI 作为 Azure 流分析作业的输出，以便创建你自己的自定义商业智能工具。

> [AZURE.NOTE]Power BI 输出是 Azure 流分析的预览功能。

## 先决条件 ##

* Microsoft Azure 帐户
* 流分析作业的输入，提供可用的流式数据。流分析接受 Azure 事件中心或 Azure Blob 存储提供的输入。  
* 用于 Power BI 的工作或学校帐户

## 创建 Azure 流分析作业 ##

在 [Azure 门户](https://manage.windowsazure.cn)中，单击**“新建 > 数据服务 > 流分析 > 快速创建”**。

指定以下值，然后单击**“创建流分析作业”**：

* **作业名称** - 输入作业名称。例如，**DeviceTemperatures**。
* **区域** - 选择要在其中放置作业的区域。考虑将作业和事件中心放在同一区域，以确保获得理想的性能，避免在不同区域之间传输数据引发的费用。
* **存储帐户** - 选择要使用的存储帐户，以便为所有在此区域运行的流分析作业存储监视数据。你可以选择现有存储帐户，也可以创建新的存储帐户。

单击左窗格中的**“流分析”**，列出流分析作业。

![graphic1][graphic1]

> [AZURE.TIP]新作业在列出时的状态将为**“未启动”**。请注意，页面底部的**“启动”**按钮已禁用。这是正常现象，因为你必须先配置作业的输入、输出、查询等，然后才能启动作业。

## 指定作业输入 ##

就本教程来说，我们假定你使用事件中心作为输入，并使用 JSON 序列化和 UTF-8 编码。

* 单击作业名称。
* 单击页面顶部的**“输入”**，然后单击**“添加输入”**。此时会打开一个对话框，引导你完成设置输入所需的一系列步骤。
*	选择**“数据流”**，然后单击右侧的按钮。
*	选择**“事件中心”**，然后单击右侧的按钮。
*	在第三页中键入或选择以下值：
  *	**输入别名** - 输入此作业输入的友好名称。请注意，你需要在后面的查询中使用此名称。
  * **事件中心** - 如果你创建的事件中心与流分析作业属于同一订阅，请选择事件中心所在的命名空间。
*	如果你的事件中心属于其他订阅，请选择**“使用其他订阅的事件中心”**，然后手动输入以下项目的相关信息：**Service Bus 命名空间**、**事件中心名称**、**事件中心策略名称**、**事件中心策略密钥**以及**事件中心分区计数**。

> [AZURE.NOTE]此示例使用默认数量的分区，即 16 个分区。

* **事件中心名称** - 选择你的 Azure 事件中心的名称。
* **事件中心策略名称** - 选择你所使用的事件中心的事件中心策略。确保此策略具有管理权限。
*	**事件中心使用者组** – 你可以将此字段留空，也可以指定你在事件中心的使用者组。请注意，每个事件中心的使用者组一次只能有 5 个读取器。因此，请根据你的作业来相应地确定适当的使用者组。如果将该字段留空，它将使用默认使用者组。

*	单击右侧按钮。
*	指定以下值：
  *	**事件序列化程序格式** - JSON
  *	**编码** - UTF8
*	单击相应勾选按钮以添加此源，并确保流分析可以成功连接到事件中心。

## 添加 Power BI 输出 ##

1.  单击页面顶部的**“输出”**，然后单击**“添加输出”**。你将看到 Power BI 被列为输出选项。

    ![graphic2][graphic2]

2.  选择 **Power BI**，然后单击右侧的按钮。
3.  你会看到如下所示的屏幕：

    ![graphic3][graphic3]

4.  在此步骤中，请提供流分析作业输出的工作或学校帐户。如果你已经有一个 Power BI 帐户，请选择**“立即授权”**。如果没有，则选择**“立即注册”**。[下面是一个很好的博客，为你介绍了 Power BI 注册的详细信息](http://blogs.technet.com/b/powerbisupport/archive/2015/02/06/power-bi-sign-up-walkthrough.aspx)。

    ![graphic11][graphic11]

5.  接下来，你会看到如下所示的屏幕：

    ![graphic4][graphic4]

提供如下所示的值：

* **输出别名** – 你可以使用任何便于引用的输出别名。如果决定为你的作业设置多个输出，则此输出别名特别有用。在这种情况下，你必须在查询中引用此输出。例如，可以使用输出别名值“OutPbi”。
* **数据集名称** - 提供你希望 Power BI 输出使用的数据集名称。例如，可以使用“pbidemo”。
*	**表名** - 在 Power BI 输出数据集下提供表名。该表名可以是“pbidemo”。目前，流分析作业的 Power BI 输出只能在数据集中设置一个表。

>	[AZURE.NOTE] You should not explicitly create this dataset and table in your Power BI account. They will be automatically created when you start your Stream Analytics job and the job starts pumping output into Power BI. If your job query doesn’t return any results, the dataset and table will not be created.

*	单击**“确定”**和**“测试连接”**。现在，你的输出配置已完成。

>	[AZURE.WARNING] Also be aware that if Power BI already had a dataset and table with the same name as the one you provided in this Stream Analytics job, the existing data will be overwritten.


## 编写查询 ##

转到作业的**“查询”**选项卡。编写查询，你希望其输出位于 Power BI 中。例如，该查询可以类似于以下 SQL 查询：

    SELECT
    	MAX(hmdt) AS hmdt,
    	MAX(temp) AS temp,
    	System.TimeStamp AS time,
    	dspl
    INTO
        OutPBI
    FROM
    	Input TIMESTAMP BY time
    GROUP BY
    	TUMBLINGWINDOW(ss,1),
    	dspl

    
    
启动你的作业。验证事件中心是否正在接收事件，以及你的查询是否生成预期的结果。如果你的查询输出的行数为 0，则不会自动创建 Power BI 数据集和表。

## 在 Power BI 中创建仪表板 ##

转到 [Powerbi.com](https://powerbi.com)，用你的工作或学校帐户登录。如果流分析作业查询输出了结果，你会看到你的数据集已经创建：

![graphic5][graphic5]

若要创建仪表板，请转到“仪表板”选项，然后创建新的仪表板。

![graphic6][graphic6]

在此示例中，我们将它标记为“演示仪表板”。

现在，请单击流分析作业创建的数据集（当前示例中的 pbidemo）。你会转到一个页面，然后便可根据此数据集创建图表。下面仅仅是你可以创建的报告的一个示例：

选择 Σ temp 和时间字段。然后会自动转到图表的“值”和“轴”：

![graphic7][graphic7]

然后，你会自动获得如下所示的图表：

![graphic8][graphic8]

在值部分，单击 temp 的下拉列表，针对温度选择**“平均”**，然后在图表中单击**“可视化”**并选择**“折线图”**：

![graphic9][graphic9]

现在，你将获得平均值随时间变化的折线图。使用如下所示的固定选项，你可以将此项固定到此前创建的仪表板：

![graphic10][graphic10]

现在，当你查看包含这个已固定报告的仪表板时，你会看到报告的实时更新。尝试更改事件中的数据，例如添加一个峰值温度或类似的变化，你就会看到反映在仪表板中的实时效果。

请注意，本教程演示了如何为数据集创建一种类型的图表。Power BI 可帮助你为组织创建其他客户商业智能工具。如需 Power BI 仪表板的其他示例，请观看 [Power BI 入门](https://youtu.be/L-Z_6P56aas?t=1m58s)视频。

若要详细了解如何通过 Power BI 来创建仪表板，请参阅另一有用的资源：[Power BI 预览版中的仪表板](http://support.powerbi.com/knowledgebase/articles/424868-dashboards-in-power-bi-preview)。

## 限制和最佳实践 ##
Power BI 同时部署并行性和吞吐量约束，如下所述：[https://powerbi.microsoft.com/pricing](https://powerbi.microsoft.com/pricing "Power BI 定价")

由于这些因素，Power BI 最适合可通过 Azure 流分析来大幅减少数据加载的案例。我们建议你使用 TumblingWindow 或 HoppingWindow 来确保数据推送速率最多为 1 次推送/秒，并且你的查询满足吞吐量要求 – 你可以使用以下公式来计算提供你的窗口所需的值（以秒为单位）：![equation1](./media/stream-analytics-power-bi-dashboard/equation1.png)。

例如，如果你每秒有 1,000 个设备在发送数据，并且你使用的 Power BI Pro SKU 支持 1,000,000 行/小时，而你需要获取每个设备在 Power BI 上的平均数据，则每个设备最多可以每 4 秒进行一次推送（如下所示）：![equation2](./media/stream-analytics-power-bi-dashboard/equation2.png)

这意味着，我们需要将原始查询更改为：

    SELECT
    	MAX(hmdt) AS hmdt,
    	MAX(temp) AS temp,
    	System.TimeStamp AS time,
    	dspl
    INTO
    	OutPBI
    FROM
    	Input TIMESTAMP BY time
    GROUP BY
    	TUMBLINGWINDOW(ss,4),
    	dspl



## 获取帮助 ##
如需进一步的帮助，请尝试我们的 [Azure 流分析论坛](https://social.msdn.microsoft.com/Forums/zh-CN/home?forum=AzureStreamAnalytics)

## 后续步骤 ##

- [Azure 流分析简介](/documentation/articles/stream-analytics-introduction)
- [Azure 流分析入门](/documentation/articles/stream-analytics-get-started)
- [缩放 Azure 流分析作业](/documentation/articles/stream-analytics-scale-jobs)
- [Azure 流分析查询语言参考](https://msdn.microsoft.com/zh-cn/library/azure/dn834998.aspx)
- [Azure 流分析管理 REST API 参考](https://msdn.microsoft.com/zh-cn/library/azure/dn835031.aspx)


[graphic1]: ./media/stream-analytics-power-bi-dashboard/1-stream-analytics-power-bi-dashboard.png
[graphic2]: ./media/stream-analytics-power-bi-dashboard/2-stream-analytics-power-bi-dashboard.png
[graphic3]: ./media/stream-analytics-power-bi-dashboard/3-stream-analytics-power-bi-dashboard.png
[graphic4]: ./media/stream-analytics-power-bi-dashboard/4-stream-analytics-power-bi-dashboard.png
[graphic5]: ./media/stream-analytics-power-bi-dashboard/5-stream-analytics-power-bi-dashboard.png
[graphic6]: ./media/stream-analytics-power-bi-dashboard/6-stream-analytics-power-bi-dashboard.png
[graphic7]: ./media/stream-analytics-power-bi-dashboard/7-stream-analytics-power-bi-dashboard.png
[graphic8]: ./media/stream-analytics-power-bi-dashboard/8-stream-analytics-power-bi-dashboard.png
[graphic9]: ./media/stream-analytics-power-bi-dashboard/9-stream-analytics-power-bi-dashboard.png
[graphic10]: ./media/stream-analytics-power-bi-dashboard/10-stream-analytics-power-bi-dashboard.png
[graphic11]: ./media/stream-analytics-power-bi-dashboard/11-stream-analytics-power-bi-dashboard.png

<!---HONumber=69-->