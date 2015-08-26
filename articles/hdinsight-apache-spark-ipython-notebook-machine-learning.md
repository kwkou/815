<properties 
	pageTitle="在 HDInsight 上使用 Apache Spark 生成机器学习应用程序 | Azure" 
	description="逐步说明如何使用 Apache Spark 随附的笔记本生成计算机学习应用程序" 
	services="hdinsight" 
	documentationCenter="" 
	authors="nitinme" 
	manager="paulettm" 
	editor="cgronlun"/>

<tags 
	ms.service="hdinsight"
	ms.date="07/19/2015"
	wacn.date=""/>


# 在 Azure HDInsight 上使用 Apache Spark 生成机器学习应用程序

了解如何使用 HDInsight 中的 Apache Spark 群集生成机器学习应用程序。本文说明如何使用群集随附的 Jupyter 笔记本来生成和测试应用程序。应用程序使用所有群集默认提供的示例 HVAC.csv 数据。

**先决条件：**

必须满足以下条件：

- Azure 订阅。 
- Apache Spark 群集。有关说明，请参阅[在 Azure HDInsight 中设置 Apache Spark 群集](/documentation/articles/hdinsight-apache-spark-provision-clusters)。 

##<a name="data"></a>讲解数据

在开始生成应用程序之前，我们先来了解数据的结构，以及要对数据执行的分析类型。

在本文中，我们将使用所有 HDInsight 群集默认提供的示例 **HVAC.csv** 数据文件，其路径为 **\\HdiSamples\\SensorSampleData\\hvac**。下载并打开该 CSV 文件，以获取数据的快照。

![HVAC 数据快照](./media/hdinsight-apache-spark-ipython-notebook-machine-learning/HDI.Spark.ML.Show.Data.png "HVAC 数据的快照")

该数据显示装有 HVAC 系统的建筑物的目标温度和实际温度。我们假设“System”列代表系统 ID，“SystemAge”列代表建筑物安装 HVAC 系统的年数。

在指定系统 ID 和系统年数的情况下，我们可以使用这些数据来预测建筑物的温度比目标温度高还是低。

##<a name="app"></a>使用 Spark MLlib 编写机器学习应用程序

1. 启动 [Jupyter](https://jupyter.org) 笔记本。在 Azure 门户选择你的 Spark 群集，然后在底部的门户任务栏中单击“Jupyter 笔记本”。出现提示时，输入 Spark 群集的管理员凭据。

2. 创建新的笔记本。单击“新建”，然后单击“Python 2”。

	![创建新的 Jupyter 笔记本](./media/hdinsight-apache-spark-ipython-notebook-machine-learning/HDI.Spark.Note.Jupyter.CreateNotebook.png "创建新的 Jupyter 笔记本")

3. 新笔记本随即已创建，并以 Untitled.pynb 名称打开。在顶部单击笔记本名称，然后输入一个友好名称。

	![提供笔记本的名称](./media/hdinsight-apache-spark-ipython-notebook-machine-learning/HDI.Spark.Note.Jupyter.Notebook.Name.png "提供笔记本的名称")

3. 开始生成机器学习应用程序。在此应用程序中，我们使用 Spark ML 管道来执行文档分类。在管道中，我们将文档分割成单字、将单字转换成数字特征向量，最后使用特征向量和标签创建预测模型。

	若要开始生成应用程序，需要先导入所需的模块，并将资源分配给应用程序。将以下代码段粘贴到新笔记本中的空白单元格处，然后按 **SHIFT + ENTER**。


		from pyspark.ml import Pipeline
		from pyspark.ml.classification import LogisticRegression
		from pyspark.ml.feature import HashingTF, Tokenizer
		from pyspark.sql import Row, SQLContext
		
		import os
		import sys
		from pyspark import SparkConf
		from pyspark import SparkContext
		from pyspark.sql import SQLContext
		from pyspark.sql.types import *
		
		from pyspark.mllib.classification import LogisticRegressionWithSGD
		from pyspark.mllib.regression import LabeledPoint
		from numpy import array
		
		# Assign resources to the application
		conf = SparkConf()
		conf.setMaster('spark://headnodehost:7077')
		conf.setAppName('pysparkregression')
		conf.set("spark.cores.max", "4")
		conf.set("spark.executor.memory", "4g")
		
		sc = SparkContext(conf=conf)
		sqlContext = SQLContext(sc)

	每当你在 Jupyter 中运行作业时，Web 浏览器窗口标题中会显示“(繁忙)”状态以及笔记本标题。右上角 **Python 2** 文本的旁边还会出现一个实心圆。作业完成后，实心圆将变成空心圆。

	 ![Jupyter 笔记本作业的状态](./media/hdinsight-apache-spark-ipython-notebook-machine-learning/HDI.Spark.Jupyter.Job.Status.png "Jupyter 笔记本作业的状态")
 
4. 现在必须加载数据 (hvac.csv)，分析数据，并使用它来训练模型。为此，需要定义检查建筑物实际温度是否高于目标温度的函数。如果实际温度较高，则表示建筑物处于高温状态，我们 **1.0** 值表示。如果实际温度较低，则表示建筑物处于低温状态，我们以 **0.0** 值表示。

	将以下代码段粘贴到空白单元格中，然后按 **SHIFT + ENTER**。

		
		# List the structure of data for better understanding. Becuase the data will be
		# loaded as an array, this structure makes it easy to understand what each element
		# in the array corresponds to

		# 0 Date
		# 1 Time
		# 2 TargetTemp
		# 3 ActualTemp
		# 4 System
		# 5 SystemAge
		# 6 BuildingID

		LabeledDocument = Row("BuildingID", "SystemInfo", "label")

		# Define a function that parses the raw CSV file and returns an object of type LabeledDocument
		
		def parseDocument(line):
    		values = [str(x) for x in line.split(',')]
    		if (values[3] > values[2]):
        		hot = 1.0
    		else:
        		hot = 0.0        
    
    		textValue = str(values[4]) + " " + str(values[5])
    
    		return LabeledDocument((values[6]), textValue, hot)

		# Load the raw HVAC.csv file, parse it using the function
		data = sc.textFile("wasb:///HdiSamples/SensorSampleData/hvac/HVAC.csv")

		documents = data.filter(lambda s: "Date" not in s).map(parseDocument)
		training = documents.toDF()


5. 设置包括三个阶段的 Spark 机器学习管道：tokenizer、hashingTF 和 lr。有关管道介绍及其工作原理的详细信息，请参阅 <a href="http://spark.apache.org/docs/latest/ml-guide.html#how-it-works" target="_blank">Spark 机器学习管道</a>。

	将以下代码段粘贴到空白单元格中，然后按 **SHIFT + ENTER**。

		tokenizer = Tokenizer(inputCol="SystemInfo", outputCol="words")
		hashingTF = HashingTF(inputCol=tokenizer.getOutputCol(), outputCol="features")
		lr = LogisticRegression(maxIter=10, regParam=0.01)
		pipeline = Pipeline(stages=[tokenizer, hashingTF, lr])

6. 将管道拟合到培训文档中。将以下代码段粘贴到空白单元格中，然后按 **SHIFT + ENTER**。

		model = pipeline.fit(training)

7. 验证训练文档以根据应用程序的进度创建检查点。将以下代码段粘贴到空白单元格中，然后按 **SHIFT + ENTER**。

		training.show()

	此时应会出现与下面类似的输出：

		BuildingID SystemInfo label
		4          13 20      0.0  
		17         3 20       0.0  
		18         17 20      1.0  
		15         2 23       0.0  
		3          16 9       1.0  
		4          13 28      0.0  
		2          12 24      0.0  
		16         20 26      1.0  
		9          16 9       1.0  
		12         6 5        0.0  
		15         10 17      1.0  
		7          2 11       0.0  
		15         14 2       1.0  
		6          3 2        0.0  
		20         19 22      0.0  
		8          19 11      0.0  
		6          15 7       0.0  
		13         12 5       0.0  
		4          8 22       0.0  
		7          17 5       0.0

	返回并根据原始 CSV 文件验证输出。例如，CSV 文件中第一行包含此数据：

	![HVAC 数据快照](./media/hdinsight-apache-spark-ipython-notebook-machine-learning/HDI.Spark.ML.Show.Data.First.Row.png "HVAC 数据的快照")

	请注意，实际温度比目标温度低的情况表示建筑物处于低温状态。因此在训练输出中，第一个行的 **label** 值为 **0.0**，表示建筑物并非处于高温状态。

8.  准备要对其运行已训练模型的数据集。为此，我们传递了系统 ID 和系统年数（以训练输出中的 **SystemInfo** 表示），模型将预测该系统 ID 和系统年数所代表的建筑物的温度是较高（以 1.0 表示）还是较低（以 0.0 表示）。

	将以下代码段粘贴到空白单元格中，然后按 **SHIFT + ENTER**。
		
		# SystemInfo here is a combination of system ID followed by system age
		Document = Row("id", "SystemInfo")
		test = sc.parallelize([(1L, "20 25"),
                      (2L, "4 15"),
                      (3L, "16 9"),
                      (4L, "9 22"),
                      (5L, "17 10"),
                      (6L, "7 22")]) \
    		.map(lambda x: Document(*x)).toDF() 

9. 最后，对测试数据进行预测。将以下代码段粘贴到空白单元格中，然后按 **SHIFT + ENTER**。

		# Make predictions on test documents and print columns of interest
		prediction = model.transform(test)
		selected = prediction.select("SystemInfo", "prediction", "probability")
		for row in selected.collect():
		    print row

10. 你应该会看到与下面类似的输出：

		Row(SystemInfo=u'20 25', prediction=1.0, probability=DenseVector([0.4999, 0.5001]))
		Row(SystemInfo=u'4 15', prediction=0.0, probability=DenseVector([0.5016, 0.4984]))
		Row(SystemInfo=u'16 9', prediction=1.0, probability=DenseVector([0.4785, 0.5215]))
		Row(SystemInfo=u'9 22', prediction=1.0, probability=DenseVector([0.4549, 0.5451]))
		Row(SystemInfo=u'17 10', prediction=1.0, probability=DenseVector([0.4925, 0.5075]))
		Row(SystemInfo=u'7 22', prediction=0.0, probability=DenseVector([0.5015, 0.4985]))

	从预测中的第一行可以看出，对于 ID 为 20 且年数为 25 的 HVAC 系统，建筑物处于高温状态 (**prediction=1.0**)。DenseVector (0.49999) 的第一个值对应于预测 0.0，第二个值 (0.5001) 对应于预测 1.0。在输出中，即使第二个值只稍高一点，模型也仍旧显示 **prediction=1.0**。

11. 现在可以通过重新启动内核来退出笔记本。在顶部菜单栏中，依次单击“内核”、“重新启动”，然后在出现提示时再次单击“重新启动”。

	![重新启动 Jupyter 内核](./media/hdinsight-apache-spark-ipython-notebook-machine-learning/HDI.Spark.Jupyter.Restart.Kernel.png "重新启动 Jupyter 内核")
	  	   

##<a name="anaconda"></a>使用机器学习 Anaconda scikit-learn 库

HDInsight 上的 Apache Spark 群集包含 Anaconda 库，其中包括适用于机器学习的 **scikit-learn** 库。该库还包含用于直接从 Jupyter 笔记本生成示例应用程序的各种数据集。有关使用 scikit-learn 库的示例，请参阅 [http://scikit-learn.org/stable/auto_examples/index.html](http://scikit-learn.org/stable/auto_examples/index.html)。

##<a name="seealso"></a>另请参阅

* [概述：Azure HDInsight 上的 Apache Spark](/documentation/articles/hdinsight-apache-spark-overview)
* [在 HDInsight 群集上设置 Spark](/documentation/articles/hdinsight-apache-spark-provision-clusters)
* [使用 HDInsight 中的 Spark 和 BI 工具执行交互式数据分析](/documentation/articles/hdinsight-apache-spark-use-bi-tools)
* [使用 HDInsight 中的 Spark 生成实时流式处理应用程序](/documentation/articles/hdinsight-apache-spark-csharp-apache-zeppelin-eventhub-streaming)
* [管理 Azure HDInsight 中 Apache Spark 群集的资源](/documentation/articles/hdinsight-apache-spark-resource-manager)


[hdinsight-versions]: /documentation/articles/hdinsight-component-versioning

[hdinsight-upload-data]: /documentation/articles/hdinsight-upload-data

[hdinsight-storage]: /documentation/articles/hdinsight-use-blob-storage


[hdinsight-weblogs-sample]: /documentation/articles/hdinsight-hive-analyze-website-log
[hdinsight-sensor-data-sample]: /documentation/articles/hdinsight-hive-analyze-sensor-data

[azure-purchase-options]: http://www.windowsazure.cn/pricing/overview/
[azure-trial]: http://www.windowsazure.cn/pricing/1rmb-trial/
[azure-management-portal]: https://manage.windowsazure.cn/
[azure-create-storageaccount]: /documentation/articles/storage-create-storage-account

<!---HONumber=66-->