<properties 
	pageTitle="在 HDInsight 上设置 Spark 群集并从 Zeppelin 和 Jupyter 使用 Spark SQL 执行交互式分析 | Azure" 
	description="逐步说明如何在 HDInsight 群集中快速设置 Apache Spark 群集，然后从 Zeppelin 和 Jupyter 笔记本使用 Spark SQL 来运行交互式查询。" 
	services="hdinsight" 
	documentationCenter="" 
	authors="nitinme" 
	manager="paulettm" 
	editor="cgronlun"/>

<tags 
	ms.service="hdinsight"
	ms.date="07/19/2015" 
	wacn.date=""/>


# 快速入门：在 HDInsight 上设置 Apache Spark 并使用 Spark SQL 运行交互式查询

了解如何使用“快速创建”选项在 HDInsight 中设置 Apache Spark 群集，然后使用基于 Web 的 [Zeppelin](https://zeppelin.incubator.apache.org) 和 [Jupyter](https://jupyter.org) 笔记本在 Spark 群集上运行 Spark SQL 交互式查询。


   ![开始使用 HDInsight 中的 Apache Spark](./media/hdinsight-apache-spark-zeppelin-notebook-jupyter-spark-sql/HDI.GetStartedFlow.Spark.png "HDInsight 中的 Apache Spark 入门教程。演示的步骤：创建存储帐户；设置群集；运行Spark SQL 语句")

**先决条件：**

在开始学习本教程之前，你必须有一个 Azure 订阅。

##<a name="storage"></a>创建 Azure 存储帐户

在 HDInsight 中设置 HDInsight 群集时，将要指定 Azure 存储帐户。将该帐户的一个特定 Blob 存储容器指定为默认文件系统。默认情况下，HDInsight 群集与你指定的存储帐户设置在同一数据中心内。有关详细信息，请参阅[将 Azure Blob 存储与 HDInsight 配合使用][hdinsight-storage]。


**创建 Azure 存储帐户**

1. 登录到 [Azure 门户][azure-management-portal]。
2. 单击左下角的“新建”，然后如图所示输入值。

	![Azure 门户，你可以在其中使用“快速创建”设置新的存储帐户](./media/hdinsight-apache-spark-zeppelin-notebook-jupyter-spark-sql/HDI.StorageAccount.QuickCreate.png "Azure 门户，你可以在其中使用“快速创建”设置新的存储帐户")

>[AZURE.NOTE]确保在群集支持的位置中创建存储帐户。

从列表中选择新存储帐户，然后单击页面底部的“管理访问密钥”。记下“主访问密钥”（或“辅助访问密钥”- 任一密钥都有效）。本教程后面的步骤中将会用到此密钥。有关详细信息，请参阅[如何创建存储帐户][azure-create-storageaccount]。
	
##<a name="provision"></a>设置 HDInsight Spark 群集

在本部分中，你将基于 Spark 版本 1.3.1 设置一个 HDInsight 版本 3.2 群集。有关 HDInsight 版本及其 SLA 的信息，请参阅 [HDInsight 组件版本](/documentation/articles/hdinsight-component-versioning)。

>[AZURE.NOTE]本文中的步骤将使用基本配置设置在 HDInsight 中创建一个 Spark 群集。有关其他群集配置设置（例如，使用其他存储、Azure 虚拟网络或 Hive 元存储）的信息，请参阅[使用自定义选项设置 HDInsight 群集](/documentation/articles/hdinsight-apache-spark-provision-clusters)。


**设置 Spark 群集**

1. 登录到 [Azure 门户][azure-management-portal]。 

2. 单击左下角的“新建”，然后如图所示输入值。

	![在 HDInsight 中创建 Spark 群集](./media/hdinsight-apache-spark-zeppelin-notebook-jupyter-spark-sql/HDI.QuickCreateCluster.png "在 HDInsight 中创建 Spark 群集")


##<a name="zeppelin"></a>使用 Zeppelin 笔记本运行交互式 Spark SQL 查询

设置群集后，可以使用基于 Web 的 Zeppelin 笔记本针对 Spark HDInsight 群集运行 Spark SQL 交互式查询。在本部分中，我们将使用群集默认提供的示例数据文件 (hvac.csv) 来运行一些交互式 Spark SQL 查询。

>[AZURE.NOTE]群集上也会按默认提供你遵循以下说明创建的笔记本。启动 Zeppelin 后，可以根据 **Zeppelin HVAC 教程**名称来找到此笔记本。

1. 启动 Zeppelin 笔记本。在 Azure 门户中选择新建的 Spark 群集，然后在底部的门户任务栏中单击“Zeppelin 笔记本”。出现提示时，输入群集的管理员凭据。遵循页面上显示的说明启动笔记本。

2. 创建新的笔记本。在标题窗格中单击“笔记本”，然后单击“创建新笔记”。

	![创建新的 Zeppelin 笔记本](./media/hdinsight-apache-spark-zeppelin-notebook-jupyter-spark-sql/HDI.Spark.CreateNewNote.png "创建新的 Zeppelin 笔记本")

	在同一页面上的“笔记本”标题下面，你应会看到名称以“笔记 XXXXXXXXX”开头的新笔记本。单击该新笔记本。

3. 在新笔记本的网页上单击标题，并根据需要更改笔记本的名称。按 ENTER 保存名称更改。此外，请确保笔记本标题在右上角显示“已连接”状态。

	![Zeppelin 笔记本状态](./media/hdinsight-apache-spark-zeppelin-notebook-jupyter-spark-sql/HDI.Spark.NewNote.Connected.png "Zeppelin 笔记本状态")

4. 将示例数据载入临时表。当你在 HDInsight 中设置 Spark 群集时，系统会将示例数据文件 **hvac.csv** 复制到 **\\HdiSamples\\SensorSampleData\\hvac** 下的关联存储帐户。

	将以下代码段粘贴到新笔记本中默认创建的空白段落处。

		// Create an RDD using the default Spark context, sc
		val hvacText = sc.textFile("wasb:///HdiSamples/SensorSampleData/hvac/HVAC.csv")
		
		// Define a schema
		case class Hvac(date: String, time: String, targettemp: Integer, actualtemp: Integer, buildingID: String)
		
		// Map the values in the .csv file to the schema
		val hvac = hvacText.map(s => s.split(",")).filter(s => s(0) != "Date").map(
    		s => Hvac(s(0), 
            		s(1),
            		s(2).toInt,
            		s(3).toInt,
            		s(6)
        	)
		).toDF()
		
		// Register as a temporary table called "hvac"
		hvac.registerTempTable("hvac")
		
	按 **SHIFT + ENTER** 或单击“播放”按钮，使段落运行代码段。段落右上角的状态应从“就绪”逐渐变成“挂起”、“正在运行”和“已完成”。输出将显示在同一段落的底部。屏幕截图如下所示：

	![基于原始数据创建临时表](./media/hdinsight-apache-spark-zeppelin-notebook-jupyter-spark-sql/HDI.Spark.Note.LoadDataIntoTable.png "基于原始数据创建临时表")

	你也可以为每个段落提供标题。在右下角单击“设置”图标，然后单击“显示标题”。

5. 现在可以针对 **hvac** 表运行 Spark SQL 语句。将以下查询粘贴到新段落中。该查询将检索建筑物 ID，以及在给定日期当天每栋建筑物的目标温度与实际温度之间的差异。按 **SHIFT + ENTER**。

		%sql
		select buildingID, (targettemp - actualtemp) as temp_diff, date 
		from hvac
		where date = "6/1/13" 

	开头的 **%Sql** 语句告诉笔记本要使用Spark SQL 解释程序。你可以从笔记本标题中的“解释程序”选项卡查看已定义的解释程序。

	以下屏幕快照显示了输出。

	![使用笔记本运行 Spark SQL 语句](./media/hdinsight-apache-spark-zeppelin-notebook-jupyter-spark-sql/HDI.Spark.Note.SparkSQLQuery1.png "使用笔记本运行 Spark SQL 语句")

	 单击显示选项（以矩形突出显示）以针对相同输出切换不同的表示形式。单击“设置”以选择构成输出中的密钥和值的项。以上屏幕截图使用 **buildingID** 作为密钥，使用 **temp_diff** 平均值作为值。

	
6. 你还可以在查询中使用变量来运行 Spark SQL 语句。下一个代码段演示如何在查询中使用你可以用来查询的值定义 **Temp** 变量。当你首次运行查询时，下拉列表中会自动填充你指定的变量值。

		%sql
		select buildingID, date, targettemp, (targettemp - actualtemp) as temp_diff
		from hvac
		where targettemp > "${Temp = 65,65|75|85}" 

	将此代码段粘贴到新段落，然后按 **SHIFT + ENTER**。以下屏幕快照显示了输出。

	![使用笔记本运行 Spark SQL 语句](./media/hdinsight-apache-spark-zeppelin-notebook-jupyter-spark-sql/HDI.Spark.Note.SparkSQLQuery2.png "使用笔记本运行 Spark SQL 语句")

	对于后续查询，可以从下拉列表中选择新的值，然后再次运行查询。单击“设置”以选择构成输出中的密钥和值的项。以上屏幕截图使用 **buildingID** 作为密钥，使用 **temp_diff** 平均值作为值，使用 **targettemp** 作为组。

7. 重新启动 Spark SQL 解释程序以退出应用程序。单击顶部的“解释程序”选项卡，然后针对 Spark 解释程序单击“重新启动”。

	![重新启动 Zeppelin 解释程序](./media/hdinsight-apache-spark-zeppelin-notebook-jupyter-spark-sql/HDI.Spark.Zeppelin.Restart.Interpreter.png "重新启动 Zeppelin 解释程序")

##<a name="jupyter"></a>使用 Jupyter 笔记本运行 Spark SQL 查询

在本部分中，你将使用 Jupyter 笔记本针对 Spark 群集运行 Spark SQL 查询。

>[AZURE.NOTE]群集上也会按默认提供你遵循以下说明创建的笔记本。启动 Jupyter 后，可以根据 **HVACTutorial.ipynb** 名称来找到此笔记本。

1. 启动 Jupyter 笔记本。在 Azure 门户选择你的 Spark 群集，然后在底部的门户任务栏中单击“Jupyter 笔记本”。出现提示时，输入 Spark 群集的管理员凭据。
2. 创建新的笔记本。单击“新建”，然后单击“Python2”。

	![创建新的 Jupyter 笔记本](./media/hdinsight-apache-spark-zeppelin-notebook-jupyter-spark-sql/HDI.Spark.Note.Jupyter.CreateNotebook.png "创建新的 Jupyter 笔记本")

3. 新笔记本随即已创建，并以 Untitled.pynb 名称打开。在顶部单击笔记本名称，然后输入一个友好名称。

	![提供笔记本的名称](./media/hdinsight-apache-spark-zeppelin-notebook-jupyter-spark-sql/HDI.Spark.Note.Jupyter.Notebook.Name.png "提供笔记本的名称")

4. 导入所需的模块，然后创建 Spark 和 SQL 上下文。将以下代码段粘贴到空白单元格中，然后按 **SHIFT + ENTER**。

		from pyspark import SparkContext
		from pyspark.sql import SQLContext
		from pyspark.sql.types import *

		# Create Spark and SQL contexts
		sc = SparkContext('spark://headnodehost:7077', 'pyspark')
		sqlContext = SQLContext(sc)

	每当你在 Jupyter 中运行作业时，Web 浏览器窗口标题中会显示“(繁忙)”状态以及笔记本标题。右上角 **Python 2** 文本的旁边还会出现一个实心圆。作业完成后，实心圆将变成空心圆。

	 ![Jupyter 笔记本作业的状态](./media/hdinsight-apache-spark-zeppelin-notebook-jupyter-spark-sql/HDI.Spark.Jupyter.Job.Status.png "Jupyter 笔记本作业的状态")

4. 将示例数据载入临时表。当你在 HDInsight 中设置 Spark 群集时，系统会将示例数据文件 **hvac.csv** 复制到 **\\HdiSamples\\SensorSampleData\\hvac** 下的关联存储帐户。

	将以下代码段粘贴到空白单元格中，然后按 **SHIFT + ENTER**。此代码段会将数据注册到名为 **hvac** 的临时表。


		# Load the data
		hvacText = sc.textFile("wasb:///HdiSamples/SensorSampleData/hvac/HVAC.csv")
		
		# Create the schema
		hvacSchema = StructType([StructField("date", StringType(), False),StructField("time", StringType(), False),StructField("targettemp", IntegerType(), False),StructField("actualtemp", IntegerType(), False),StructField("buildingID", StringType(), False)])
		
		# Parse the data in hvacText
		hvac = hvacText.map(lambda s: s.split(",")).filter(lambda s: s[0] != "Date").map(lambda s:(str(s[0]), str(s[1]), int(s[2]), int(s[3]), str(s[6]) ))
		
		# Create a data frame
		hvacdf = sqlContext.createDataFrame(hvac,hvacSchema)
		
		# Register the data fram as a table to run queries against
		hvacdf.registerAsTable("hvac")
		
		# Run queries against the table and display the data
		data = sqlContext.sql("select buildingID, (targettemp - actualtemp) as temp_diff, date from hvac where date = \"6/1/13\"")
		data.show()

5. 作业成功完成后，将显示以下输出。

		buildingID temp_diff date  
		4          8         6/1/13
		3          2         6/1/13
		7          -10       6/1/13
		12         3         6/1/13
		7          9         6/1/13
		7          5         6/1/13
		3          11        6/1/13
		8          -7        6/1/13
		17         14        6/1/13
		16         -3        6/1/13
		8          -8        6/1/13
		1          -1        6/1/13
		12         11        6/1/13
		3          14        6/1/13
		6          -4        6/1/13
		1          4         6/1/13
		19         4         6/1/13
		19         12        6/1/13
		9          -9        6/1/13
		15         -10       6/1/13

6. 重新启动内核以退出应用程序。在顶部菜单栏中，依次单击“内核”、“重新启动”，然后在出现提示时再次单击“重新启动”。

	![重新启动 Jupyter 内核](./media/hdinsight-apache-spark-zeppelin-notebook-jupyter-spark-sql/HDI.Spark.Jupyter.Restart.Kernel.png "重新启动 Jupyter 内核")


##<a name="seealso"></a>另请参阅


* [概述：Azure HDInsight 上的 Apache Spark](/documentation/articles/hdinsight-apache-spark-overview)
* [在 HDInsight 群集上设置 Spark](/documentation/articles/hdinsight-apache-spark-provision-clusters)
* [使用 HDInsight 中的 Spark 和 BI 工具执行交互式数据分析](/documentation/articles/hdinsight-apache-spark-use-bi-tools)
* [使用 HDInsight 中的 Spark 生成机器学习应用程序](/documentation/articles/hdinsight-apache-spark-ipython-notebook-machine-learning)
* [使用 HDInsight 中的 Spark 生成实时流式处理应用程序](/documentation/articles/hdinsight-apache-spark-csharp-apache-zeppelin-eventhub-streaming)
* [管理 Azure HDInsight 中 Apache Spark 群集的资源](/documentation/articles/hdinsight-apache-spark-resource-manager)


[hdinsight-versions]: /documentation/articles/hdinsight-component-versioning
[hdinsight-upload-data]: /documentation/articles/hdinsight-upload-data
[hdinsight-storage]: /documentation/articles/hdinsight-use-blob-storage

[azure-purchase-options]: http://www.windowsazure.cn/pricing/overview/
[azure-trial]: http://www.windowsazure.cn/pricing/1rmb-trial/
[azure-management-portal]: https://manage.windowsazure.cn/
[azure-create-storageaccount]: /documentation/articles/storage-create-storage-account

<!---HONumber=66-->