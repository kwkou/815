<properties 
	pageTitle="使用资源管理器向 HDInsight 中的 Apache Spark 群集分配资源 | Azure" 
	description="了解如何对 HDInsight 上的 Apache Spark 群集使用资源管理器，以提高性能。" 
	services="hdinsight" 
	documentationCenter="" 
	authors="nitinme" 
	manager="paulettm" 
	editor="cgronlun"/>

<tags 
	ms.service="hdinsight" 
	ms.date="07/19/2015" 
	wacn.date=""/>


# 管理 Azure HDInsight 中 Apache Spark 群集的资源

资源管理器是 Spark 群集仪表板的组件，可让你管理群集上运行的每个应用程序所用的核心和 RAM 等资源。

## <a name="launchrm"></a>如何启动资源管理器？

1. 在 Azure 门户选择你的 Spark 群集，然后在底部的门户任务栏中单击“Spark 仪表板”。

2. 在仪表板的顶部窗格中，单击“资源管理器”选项卡。

##<a name="scenariosrm"></a>如何使用资源管理器修复这些问题？

下面介绍了当你使用 Spark 群集时可能会遇到的一些常见问题，并说明了如何使用资源管理器解决这些问题。

### HDInsight 上的 Spark 群集速度很慢

HDInsight 中的 Apache Spark 群集专门为多租户而设计，因此资源已拆分成多个组件（笔记本、作业服务器等）。这样，你便可以同时使用所有 Spark 组件，而无需担心有组件无法运行资源，但由于资源已分段，因此每个组件都变慢。你可以根据需要进行调整。


### 我只需在 Spark 群集上使用 Jupyter 笔记本。如何将所有资源分配给群集？

1. 在“Spark 仪表板”中，单击“Spark UI”选项卡以找出可分配给应用程序的核心数目和 RAM 上限。

	![资源分配](./media/hdinsight-apache-spark-resource-manager/HDI.Spark.UI.Resource.png "查找已分配给 Spark 群集的资源")

	根据上面的屏幕截图，可以分配的核心数目上限为 7 个（总共 8 个，其中有 1 正在使用），可分配的 RAM 上限为 9 GB（总共 12 GB RAM，其中 2 GB 必须保留给系统使用，有 1 GB 正在供给其他应用程序使用）。

	你还应该考虑到所有正在运行的应用程序。可以从“Spark UI”选项卡查看运行中的应用程序。

	![正在运行的应用程序](./media/hdinsight-apache-spark-resource-manager/HDI.Spark.UI.Running.Apps.png "在群集上运行的应用程序")

	
2. 在 HDInsight Spark 仪表板中单击“资源管理器”选项卡，然后指定“默认应用程序核心计数”和“每个工作线程节点的默认执行器内存”的值。将其他属性设置为 0。

	![资源分配](./media/hdinsight-apache-spark-resource-manager/HDI.Spark.UI.Allocate.Resources.png "向应用程序分配资源")

### 我不需要将 BI 工具与 Spark 群集配合使用。该如何回收资源？ 

将 Thrift 服务器核心计数和 Thrift 服务器执行器内存指定为 0。在未分配核心或内存的情况下，Thrift 服务器将进入“等候”状态。

![资源分配](./media/hdinsight-apache-spark-resource-manager/HDI.Spark.UI.No.Thrift.png "未向 thrift 服务器分配资源")

##<a name="seealso"></a>另请参阅

* [概述：Azure HDInsight 上的 Apache Spark](/documentation/articles/hdinsight-apache-spark-overview)
* [在 HDInsight 群集上设置 Spark](/documentation/articles/hdinsight-apache-spark-provision-clusters)
* [使用 HDInsight 中的 Spark 和 BI 工具执行交互式数据分析](/documentation/articles/hdinsight-apache-spark-use-bi-tools)
* [使用 HDInsight 中的 Spark 生成机器学习应用程序](/documentation/articles/hdinsight-apache-spark-ipython-notebook-machine-learning)
* [使用 HDInsight 中的 Spark 生成实时流式处理应用程序](/documentation/articles/hdinsight-apache-spark-csharp-apache-zeppelin-eventhub-streaming)


[hdinsight-versions]: /documentation/articles/hdinsight-component-versioning
[hdinsight-upload-data]: /documentation/articles/hdinsight-upload-data
[hdinsight-storage]: /documentation/articles/hdinsight-use-blob-storage

[azure-purchase-options]: http://www.windowsazure.cn/pricing/overview/
[azure-trial]: http://www.windowsazure.cn/pricing/1rmb-trial/
[azure-management-portal]: https://manage.windowsazure.cn/
[azure-create-storageaccount]: /documentation/articles/storage-create-storage-account

<!---HONumber=66-->