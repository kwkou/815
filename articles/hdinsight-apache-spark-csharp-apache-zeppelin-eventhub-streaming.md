<properties 
	pageTitle="使用 Azure 事件中心和 HDInsight 中的 Apache Spark 处理流数据 | Azure" 
	description="逐步说明如何向 Azure 事件中心发送数据流，然后使用 Zeppelin 笔记本在 Spark 接收这些事件" 
	services="hdinsight" 
	documentationCenter="" 
	authors="nitinme" 
	manager="paulettm" 
	editor="cgronlun"/>


<tags 
	ms.service="hdinsight"
	ms.date="07/10/2015" 
	wacn.date=""/>

# Spark Streaming：在 HDInsight 上使用 Apache Spark 处理来自 Azure 事件中心的事件

Spark Streaming 可以扩展核心 Spark API，以生成可缩放、高吞吐量、容错的流处理应用程序。可以从许多来源引入数据。在本文中，我们将使用事件中心来引入数据。事件中心是高度可缩放的引入系统，每秒可以引入数以百万计的事件。

在本教程中，你将学习如何创建 Azure 事件中心，使用以 C# 编写的控制台应用程序将消息引入事件中心，以及使用针对 HDInsight 中 Apache Spark 配置的 Zeppelin 笔记本同时检索这些消息。

**先决条件：**

必须满足以下条件：

- Azure 订阅。
- Apache Spark 群集。有关说明，请参阅[在 Azure HDInsight 中设置 Apache Spark 群集](/documentation/articles/hdinsight-apache-spark-provision-clusters)。
- 一个 [Azure 事件中心](/documentation/articles/service-bus-event-hubs-csharp-ephcs-getstarted)。
- 一个装有 Visual Studio 2013 的工作站。有关说明，请参阅[安装 Visual Studio](https://msdn.microsoft.com/zh-cn/library/e2h7fzkw.aspx)。

##<a name="createeventhub"></a>创建 Azure 事件中心

1. 在 [Azure 门户](https://manage.windowsazure.cn)中，选择“新建”>“服务总线”>“事件中心”>“自定义创建”。

2. 在“添加新事件中心”屏幕中，输入“事件中心名称”，选择要在其中创建中心的“区域”，然后创建新的命名空间或选择现有的命名空间。单击**箭头**继续。

	![向导页 1](./media/hdinsight-apache-spark-csharp-apache-zeppelin-eventhub-streaming/HDI.Spark.Streaming.Create.Event.Hub.png "创建 Azure 事件中心")

	> [AZURE.NOTE]你应该选择与 HDInsight 中 Apache Spark 群集相同的“位置”，以便降低延迟和成本。

3. 在“配置事件中心”屏幕中，输入“分区计数”和“消息保留期”值，然后单击复选标记。对于本示例，请使用分区计数 10，消息保留期 1。记下分区计数，因为稍后需要用到。

	![向导页 2](./media/hdinsight-apache-spark-csharp-apache-zeppelin-eventhub-streaming/HDI.Spark.Streaming.Create.Event.Hub2.png "为事件中心指定分区大小和保留天数")

4. 单击你创建的事件中心，单击“配置”，然后为事件中心创建两个访问策略。

	<table>
	<tr><th>Name</th><th>权限</th></tr>
	<tr><td>mysendpolicy</td><td>发送</td></tr>
	<tr><td>myreceivepolicy</td><td>侦听</td></tr>
	</table>

	创建权限后，在页面底部选择“保存”图标。这将会创建共享访问策略，用于对此事件中心进行发送 (**mysendpolicy**) 和侦听 (**myreceivepolicy**)。

	![策略](./media/hdinsight-apache-spark-csharp-apache-zeppelin-eventhub-streaming/HDI.Spark.Streaming.Event.Hub.Policies.png "创建事件中心策略")

	
5. 在同一页上，记下针对这两个策略生成的策略密钥。请保存这些密钥，因为稍后将要用到。

	![策略密钥](./media/hdinsight-apache-spark-csharp-apache-zeppelin-eventhub-streaming/HDI.Spark.Streaming.Event.Hub.Policy.Keys.png "保存策略密钥")

6. 在“仪表板”页上，单击底部的“连接信息”以使用两个策略来检索和保存事件中心的连接字符串。

	![策略密钥](./media/hdinsight-apache-spark-csharp-apache-zeppelin-eventhub-streaming/HDI.Spark.Streaming.Event.Hub.Policy.Connection.Strings.png "保存策略连接字符串")

[AZURE.INCLUDE [service-bus-event-hubs-get-started-send-csharp](../includes/service-bus-event-hubs-get-started-send-csharp.md)]

##<a name="receivezeppelin"></a>使用 Zeppelin 接收 HDInsight 上 Spark 中的消息

在本部分中，你创建一个 [Zeppelin](https://zeppelin.incubator.apache.org) 笔记本，以接收从事件中心发送到 HDInsight 上 Spark 群集中的消息。

1. 启动 Zeppelin 笔记本。在 Azure 门户选择你的 Spark 群集，然后在底部的门户任务栏中单击“Zeppelin 笔记本”。出现提示时，输入 Spark 群集的管理员凭据。遵循页面上显示的说明启动笔记本。

2. 创建新的笔记本。在标题窗格中单击“笔记本”，然后在下拉列表中单击“创建新笔记”。

	![创建新的 Zeppelin 笔记本](./media/hdinsight-apache-spark-csharp-apache-zeppelin-eventhub-streaming/HDI.Spark.CreateNewNote.png "创建新的 Zeppelin 笔记本")

	在同一页面上的“笔记本”标题下面，你应会看到名称以“笔记 XXXXXXXXX”开头的新笔记本。单击该新笔记本。

3. 在新笔记本的网页上单击标题，并根据需要更改笔记本的名称。按 ENTER 保存名称更改。此外，请确保笔记本标题在右上角显示“已连接”状态。

	![Zeppelin 笔记本状态](./media/hdinsight-apache-spark-csharp-apache-zeppelin-eventhub-streaming/HDI.Spark.NewNote.Connected.png "Zeppelin 笔记本状态")

4. 将以下代码段粘贴到新笔记本中默认创建的空白段落处，并替换占位符以使用你的事件中心配置。在此代码段中，你将接收来自事件中心的流，并将流注册到名为 **mytemptable** 的临时表。在下一部分，我们将启动发送器应用程序。然后，你可以直接从表读取数据。

		import org.apache.spark.streaming.{Seconds, StreamingContext}
		import org.apache.spark.streaming.eventhubs.EventHubsUtils
		import sqlContext.implicits._
		
		val ehParams = Map[String, String](
		  "eventhubs.policyname" -> "<name of policy with listen permissions>",
		  "eventhubs.policykey" -> "<key of policy with listen permissions>",
		  "eventhubs.namespace" -> "<service bus namespace>",
		  "eventhubs.name" -> "<event hub in the service bus namespace>",
		  "eventhubs.partition.count" -> "1",
		  "eventhubs.consumergroup" -> "$default",
		  "eventhubs.checkpoint.dir" -> "/<check point directory in your storage account>",
		  "eventhubs.checkpoint.interval" -> "10"
		)
		
		val ssc =  new StreamingContext(sc, Seconds(10))
		val stream = EventHubsUtils.createUnionStream(ssc, ehParams)
		
		case class Message(msg: String)
		stream.map(msg=>Message(new String(msg))).foreachRDD(rdd=>rdd.toDF().registerTempTable("mytemptable"))

		stream.print
		ssc.start


##<a name="runapps"></a>运行应用程序

1. 从 Zeppelin 笔记本运行包含代码段的段落。按 **SHIFT + ENTER** 或右上角的“播放”按钮。

	段落右上角的状态应从“就绪”逐渐变成“挂起”、“正在运行”和“已完成”。输出将显示在同一段落的底部。屏幕截图如下所示：

	![代码段的输出](./media/hdinsight-apache-spark-csharp-apache-zeppelin-eventhub-streaming/HDI.Spark.Streaming.Event.Hub.Zeppelin.Code.Output.png "代码段的输出")

2. 运行**发送器**项目，然后在控制台窗口中按 **Enter** 开始将消息发送到事件中心。

3. 在 Zeppelin 笔记本的新段落中，输入以下代码段以读取在 Spark 中收到的消息。

		%sql select * from mytemptable limit 10

	以下屏幕截图显示了 **mytemptable** 中收到的消息。

	![在 Zeppelin 中接收消息](./media/hdinsight-apache-spark-csharp-apache-zeppelin-eventhub-streaming/HDI.Spark.Streaming.Event.Hub.Zeppelin.Output.png "在 Zeppelin 笔记本中接收消息")

4. 重新启动 Spark SQL 解释程序以退出应用程序。单击顶部的“解释程序”选项卡，然后针对 Spark 解释程序单击“重新启动”。

	![重新启动 Zeppelin 解释程序](./media/hdinsight-apache-spark-csharp-apache-zeppelin-eventhub-streaming/HDI.Spark.Zeppelin.Restart.Interpreter.png "重新启动 Zeppelin 解释程序")

##<a name="sparkstreamingha"></a>以高可用性运行流式处理应用程序

使用 Zeppelin 在 HDInsight 上的 Spark 群集中接收流数据是塑造应用程序原型的好方法。不过，若要以高可用性和弹性在生产设置中运行流式处理应用程序，你需要执行以下操作：

1. 将依赖性 jar 复制到与群集关联的存储帐户。
2. 生成流式处理应用程序。
3. 通过 RDP 访问群集，并将应用程序 jar 复制到群集的头节点。
3. 通过 RDP 访问群集，并在群集节点上运行应用程序。

有关如何执行这些步骤的说明和示例流式处理应用程序，请从 GitHub 下载：[https://github.com/hdinsight/hdinsight-spark-examples](https://github.com/hdinsight/hdinsight-spark-examples)。


##<a name="seealso"></a>另请参阅


* [概述：Azure HDInsight 上的 Apache Spark](/documentation/articles/hdinsight-apache-spark-overview)
* [快速入门：在 HDInsight 上设置 Apache Spark 并使用 Spark SQL 运行交互式查询](/documentation/articles/hdinsight-apache-spark-zeppelin-notebook-jupyter-spark-sql)
* [使用 HDInsight 中的 Spark 生成机器学习应用程序](/documentation/articles/hdinsight-apache-spark-ipython-notebook-machine-learning)
* [使用 HDInsight 中的 Spark 和 BI 工具执行交互式数据分析](/documentation/articles/hdinsight-apache-spark-use-bi-tools)
* [管理 Azure HDInsight 中 Apache Spark 群集的资源](/documentation/articles/hdinsight-apache-spark-resource-manager)


[hdinsight-versions]: /documentation/articles/hdinsight-component-versioning
[hdinsight-upload-data]: /documentation/articles/hdinsight-upload-data
[hdinsight-storage]: /documentation/articles/hdinsight-use-blob-storage

[azure-purchase-options]: http://www.windowsazure.cn/pricing/overview/
[azure-trial]: http://www.windowsazure.cn/pricing/1rmb-trial/
[azure-management-portal]: https://manage.windowsazure.cn/
[azure-create-storageaccount]: /documentation/articles/storage-create-storage-account/

<!---HONumber=66-->