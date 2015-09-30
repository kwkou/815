<properties
	pageTitle="使用流分析进行实时 Twitter 观点分析 | Windows Azure"
	description="了解如何使用流分析进行实时 Twitter 观点分析。在实时仪表板上提供从事件生成到数据的分步指南。"
	keywords="real-time twitter,sentiment analysis,social media analysis,social media analytics tools"
	services="stream-analytics"
	documentationCenter=""
	authors="jeffstokes72"
	manager="paulettm"
	editor="cgronlun"/>

<tags
	ms.service="stream-analytics"
	ms.date="08/19/2015"
	wacn.date="09/15/2015"/>


# 社交媒体分析：在 Azure 流分析中进行实时 Twitter 观点分析

在本教程中，你将学习如何构建一个观点分析解决方案：将实时 Twitter 事件引入事件中心，通过编写流分析查询来分析这些数据，然后将结果进行存储，或者使用仪表板来提供实时见解。

社交媒体分析工具可以利用在社交媒体中发布的大量文章进行分析，帮助组织了解各种趋势主题、有意义的主题以及大众的态度。观点分析（也称为“观点挖掘”）使用社交媒体分析工具来确定大众对产品、理念等的态度。

## 方案

新闻媒体网站感兴趣的是如何打造网站内容，使之贴近其读者，以便获得对竞争对手的竞争优势。他们对与读者相关的主题开展社交媒体分析的方式是对 Twitter 数据进行实时观点分析。具体说来，若要实时确定哪些主题在 Twitter 中很流行，他们需要对推文数量以及关键主题的观点进行实时分析。

## 先决条件
1.	学习本教程需要一个 Twitter 帐户。  
2.	本演练使用 GitHub 上的 Twitter 客户端应用程序。请在[此处](https://github.com/Azure/azure-stream-analytics/tree/master/DataGenerators/TwitterClient)下载，然后按照以下步骤设置你的解决方案。

## 创建事件中心输入和使用者组

示例应用程序将会生成事件并将其推送到事件中心实例（简称“事件中心”）。Service Bus 事件中心是适合流分析的首选事件引入方法。请参阅 [Service Bus 文档](/documentation/services/service-bus/)中的事件中心文档

请按照以下步骤来创建事件中心。

1.	在 Azure 门户中单击**“新建”**>**“应用程序服务”**> **SERVICE BUS** >**“事件中心”**>**“快速创建”**，然后提供创建新的事件中心所需的名称、区域以及新的或现有的命名空间。  
2.	最佳做法是让每个流分析作业都从单个事件中心使用者组进行读取。我们将在下面向你详细介绍创建使用者组的过程，你可以在此处了解有关使用者组的详细信息。若要创建使用者组，请导航到新创建的事件中心并单击**“使用者组”**选项卡，然后单击页面底部的**“创建”**，为使用者组提供一个名称。
3.	若要授予对事件中心的访问权限，需创建共享访问策略。单击事件中心的**“配置”**选项卡。
4.	在**“共享访问策略”**下，使用**管理**权限创建一个新策略。


  	![共享访问策略：你可以在其中使用管理权限创建策略。](./media/stream-analytics-twitter-sentiment-analysis-trends/stream-ananlytics-shared-access-policies.png)

5.	单击页面底部的“保存”。
6.	导航到**“仪表板”**，单击页面底部的**“连接信息”**，然后复制并保存连接信息。（使用显示在搜索图标下的复制图标。）

## 配置并启动 Twitter 客户端应用程序

我们提供了一个客户端应用程序，该程序可以通过 [Twitter 的流式处理 API](https://dev.twitter.com/streaming/overview) 来挖掘 Twitter 数据，收集有关参数化主题组的推文事件。将使用第三方开源工具 [Sentiment140](http://help.sentiment140.com/) 为每篇推文指定一个观点值（0：负面；2：中立；4：正面），然后将推文事件推送到事件中心。

请按以下步骤设置该应用程序：

1.	[下载 TwitterClient 解决方案](https://github.com/streamanalytics/samples/tree/master/TwitterClient)
2.	打开 App.config，然后将 oauth\_consumer\_key、oauth\_consumer\_secret、oauth\_token、oauth\_token\_secret 替换为使用你自己的值的 Twitter 令牌。  

	[生成 OAuth 访问令牌的步骤](https://dev.twitter.com/oauth/overview/application-owner-access-tokens)

	请注意，你需要使用空的应用程序来生成令牌。  
3.	将 App.config 中 EventHubConnectionString 和 EventHubName 的值替换为事件中心的连接字符串和名称。
4.	*可选：*调整要搜索的关键字。目前情况下，此应用程序会搜索“Azure,Skype,XBox,Microsoft,Seattle”。你可以根据需要调整 App.config 中 twitter\_keywords 的值。
5.	生成解决方案
6.	启动应用程序。你会看到已将 CreatedAt、Topic 和 SentimentScore 值发送到事件中心的推文事件：

	![观点分析：发送到事件中心的 SentimentScore 值。](./media/stream-analytics-twitter-sentiment-analysis-trends/stream-analytics-twitter-sentiment-output-to-event-hub.png)

## 创建流分析作业

现在，我们已获得 Twitter 提供的实时推文事件流，因此可以设置一个流分析作业来实时分析这些事件。

### 预配流分析作业

1.	在 [Azure 门户](https://manage.windowsazure.cn/)中，单击**“新建”**>**“数据服务”**>**“流分析”**>**“快速创建”**。
2.	指定以下值，然后单击**“创建流分析作业”**：

	* **作业名称**：输入作业名称。
	* **区域**：选择要在其中运行作业的区域。考虑将作业和事件中心放在同一区域，以确保获得更好的性能，并确保在不同区域之间传输数据时不需付费。
	* **存储帐户**：选择要使用的存储帐户，以便为所有在此区域运行的流分析作业存储监视数据。你可以选择现有存储帐户，也可以创建新的存储帐户。

3.	单击左窗格中的**“流分析”**，列出流分析作业。  
![流分析服务图标](./media/stream-analytics-twitter-sentiment-analysis-trends/stream-analytics-service-icon.png)

4.	新作业在显示时的状态为**“已创建”**。请注意，页面底部的**“启动”**按钮已禁用。你必须先配置作业输入、输出和查询，然后才能启动作业。


### 指定作业输入
1.	在流分析作业中，单击页面顶部的**“输入”**，然后单击**“添加输入”**。此时会打开一个对话框，引导你完成设置输入所需的一系列步骤。
2.	选择**“数据流”**，然后单击右侧的按钮。
3.	选择**“事件中心”**，然后单击右侧的按钮。
4.	在第三页中键入或选择以下值：

	* **输入别名**：输入此作业输入的友好名称，例如 TwitterStream。请注意，你需要在后面的查询中使用此名称。**事件中心**：如果你创建的事件中心与流分析作业属于同一订阅，请选择事件中心所在的命名空间。

		如果你的事件中心属于其他订阅，请选择**“使用其他订阅的事件中心”**，然后手动输入以下项目的相关信息：**Service Bus 命名空间**、**事件中心名称**、**事件中心策略名称**、**事件中心策略密钥**以及**事件中心分区计数**。

	* **事件中心名称**：选择事件中心的名称。
	* **事件中心策略名称**：选择此前在本教程中创建的事件中心策略。
	* **事件中心使用者组**：键入此前在本教程中创建的使用者组。
5.	单击右侧按钮。
6.	指定以下值：

	* **事件序列化程序格式**：JSON
	* **编码**：UTF8

7.	单击相应勾选按钮以添加此源，并确保流分析可以成功连接到事件中心。

### 指定作业查询

流分析支持简单的声明性查询模型，用于描述各种转换。若要了解有关语言的详细信息，请参阅 [Azure 流分析查询语言参考](https://msdn.microsoft.com/zh-cn/library/azure/dn834998.aspx)。本教程将帮助你创作和测试多个可通过 Twitter 数据完成的查询。

#### 示例数据输入

若要针对实际的作业数据来验证查询，可使用“示例数据”功能从流中提取事件，然后创建包含要测试事件的 .JSON 文件。

1.	选择事件中心输入，然后单击页面底部的**“示例数据”**。
2.	在出现的对话框中，指定开始收集数据的**“开始时间”**，以及使用额外数据的**“持续时间”**。
3.	单击**“详细信息”**按钮，然后单击**“单击此处”**链接以下载并保存所生成的 .JSON 文件。

#### 传递查询
开始时，我们将进行简单的传递查询来投射事件中的所有字段。

1.	单击流分析作业页顶部的**“查询”**。
2.	在代码编辑器中，用以下内容替换初始查询模板：

		SELECT * FROM TwitterStream

	请确保输入源的名称与你此前指定的输入的名称相匹配。

3.	单击查询编辑器下的**“测试”**
4.	浏览到示例 .JSON 文件
5.	单击勾选按钮，然后就会看到结果显示在查询定义下方。

	![显示在查询定义下方的结果](./media/stream-analytics-twitter-sentiment-analysis-trends/stream-analytics-sentiment-by-topic.png)

#### 按主题对推文计数：带聚合的翻转窗口

若要比较主题之间的提及数，我们将利用 [TumblingWindow](https://msdn.microsoft.com/zh-cn/library/azure/dn835055.aspx) 来获取按主题的提及数（每 5 秒）。

1.	在代码编辑器中将查询更改为：

		SELECT System.Timestamp as Time, Topic, COUNT(*)
		FROM TwitterStream TIMESTAMP BY CreatedAt
		GROUP BY TUMBLINGWINDOW(s, 5), Topic

	请注意，此查询使用 **TIMESTAMP BY** 关键字在要用于临时计算的负载中指定时间戳字段。如果未指定此字段，将根据每个事件到达事件中心的时间执行开窗操作。请参阅[流分析查询参考](https://msdn.microsoft.com/zh-cn/library/azure/dn834998.aspx)中的“到达时间与应用程序时间”以了解详细信息。

	该查询还通过 **System.Timestamp** 来访问每个窗口结束时的时间戳。

2.	单击查询编辑器下的**“重新运行”**以查看查询结果。

#### 确定趋势性主题：滑动窗口

若要确定趋势性主题，我们需要查找那些在给定时间内被提及的次数超过某个阈值的主题。就本教程来说，我们将使用 [SlidingWindow](https://msdn.microsoft.com/zh-cn/library/azure/dn835051.aspx) 来查找在过去 5 秒内被提及次数超过 20 次的主题。

1.	在代码编辑器中将查询更改为：

		SELECT System.Timestamp as Time, Topic, COUNT(*) as Mentions
		FROM TwitterStream TIMESTAMP BY CreatedAt
		GROUP BY SLIDINGWINDOW(s, 5), topic
		HAVING COUNT(*) > 20

2.	单击查询编辑器下的**“重新运行”**以查看查询结果。

	![滑动窗口查询输出](./media/stream-analytics-twitter-sentiment-analysis-trends/stream-analytics-query-output.png)

#### 提及次数和观点：带聚合的翻转窗口

我们将要测试的最后一个查询使用 TumblingWindow 每隔 5 秒来获取每个主题的提及数以及观点分数的平均值、最小值、最大值和标准偏差。

1.	在代码编辑器中将查询更改为：

		SELECT System.Timestamp as Time, Topic, COUNT(*), AVG(SentimentScore), MIN(SentimentScore),
    	Max(SentimentScore), STDEV(SentimentScore)
		FROM TwitterStream TIMESTAMP BY CreatedAt
		GROUP BY TUMBLINGWINDOW(s, 5), Topic

2.	单击查询编辑器下的**“重新运行”**以查看查询结果。
3.	这是将要用于仪表板的查询。单击页面底部的“保存”。


## 创建输出接收器

现在，我们已经定义了事件流、用于引入事件的事件中心输入，以及用于通过流执行转换的查询，最后一步是为作业定义输出接收器。我们需要将聚合的推文事件从作业查询写入 Azure Blob。你也可以将结果推送到 SQL 数据库、表存储或事件中心，具体取决于特定应用程序需要。

按照下面的步骤为 Blob 存储创建容器（如果还没有）：

1.	使用现有存储帐户，或者通过单击**“新建”**>**“数据服务”**>**“存储”**>**“快速创建”**并遵循屏幕上的说明进行操作来创建新的存储帐户。
2.	选择存储帐户，单击页面顶部的**“容器”**，然后单击**“添加”**。
3.	指定容器的**“名称”**，然后将其**“访问权限”**设置为“公共 Blob”。

## 指定作业输出

1.	在流分析作业中，单击页面顶部的**“输出”**，然后单击**“添加输出”**。此时会打开一个对话框，引导你完成设置输出所需的一系列步骤。
2.	选择**“Blob 存储”**，然后单击右侧的按钮。
3.	在第三页中键入或选择以下值：

	* **输出别名**：输入此作业输出的友好名称
	* **订阅**：如果你创建的 Blob 存储与流分析作业属于同一订阅，请选择**“使用当前订阅中的存储帐户”**。如果你的存储属于其他订阅，请选择**“使用其他订阅中的存储帐户”**，然后针对**“存储帐户”**、**“存储帐户密钥”**、**“容器”**手动输入相关信息。
	* **存储帐户**：选择存储帐户的名称
	* **容器**：选择容器的名称
	* **文件名前缀**：键入写入 blob 输出时要使用的文件前缀

4.	单击右侧按钮。
5.	指定以下值：
	* **事件序列化程序格式**：JSON
	* **编码**：UTF8
6.	单击相应勾选按钮以添加此源，并确保流分析可以成功连接到存储帐户。

## 启动作业

由于作业输入、查询和输出均已指定，因此我们可以启动流分析作业。

1.	在作业**“仪表板”**上，单击页面底部的**“启动”**。
2.	在出现的对话框中，选择**“作业开始时间”**，然后单击对话框底部的复选标记按钮。作业状态将更改为**“正在启动”**，然后很快会转变为**“正在运行”**。


## 查看观点分析的输出

当作业运行和处理实时 Twitter 流时，请选择你希望如何查看观点分析的输出。使用 [Azure 存储资源管理器](https://azurestorageexplorer.codeplex.com/)或 [Azure 资源管理器](http://www.cerebrata.com/products/azure-explorer/introduction)之类的工具实时查看作业输出。在这里，你可以扩展你的应用程序，使之包括基于输出的自定义仪表板，就像下图所示的使用 [Power BI](https://powerbi.com/) 的仪表板一样。

![社交媒体分析：Power BI 仪表板中的流分析观点分析（观点挖掘）输出。](./media/stream-analytics-twitter-sentiment-analysis-trends/stream-analytics-output-power-bi.png)

## 获取支持
如需进一步的帮助，请尝试我们的 [Azure 流分析论坛](https://social.msdn.microsoft.com/Forums/zh-CN/home?forum=AzureStreamAnalytics)。


## 后续步骤

- [Azure 流分析简介](/documentation/articles/stream-analytics-introduction)
- [Azure 流分析入门](/documentation/articles/stream-analytics-get-started)
- [缩放 Azure 流分析作业](/documentation/articles/stream-analytics-scale-jobs)
- [Azure 流分析查询语言参考](https://msdn.microsoft.com/zh-cn/library/azure/dn834998.aspx)
- [Azure 流分析管理 REST API 参考](https://msdn.microsoft.com/zh-cn/library/azure/dn835031.aspx)
 

<!---HONumber=69-->