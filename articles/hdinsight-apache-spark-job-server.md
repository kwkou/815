<properties 
	pageTitle="HDInsight 上的 Apache Spark 作业服务器 | Azure" 
	description="了解如何使用 Spark 作业服务器在 Spark 群集上远程提交和管理作业。" 
	services="hdinsight" 
	documentationCenter="" 
	authors="nitinme" 
	manager="paulettm" 
	editor="cgronlun"/>

<tags 
	ms.service="hdinsight" 
	ms.date="07/10/2015" 
	wacn.date=""/>


# Azure HDInsight 群集上的 Spark 作业服务器

Azure HDInight 上的 Apache Spark 群集可将 Spark 作业服务器打包为群集部署的一部分。Spark 作业服务器提供用于创建 Spark 上下文、将 Spark 应用程序提交到上下文、检查作业状态、终止上下文等操作的 REST API。本文提供有关如何使用 Curl 在使用作业服务器的 Spark 群集上执行几个常见任务的一些示例。

>[AZURE.NOTE]有关 Spark 作业服务器的完整文档，请参阅 [https://github.com/spark-jobserver/spark-jobserver](https://github.com/spark-jobserver/spark-jobserver)。

## <a name="uploadjar"></a>将 jar 上载到 Spark 群集

	curl.exe -k -u "<hdinsight user>:<user password>" --data-binary @<location of jar on the computer> https://<cluster name>.azurehdinsight.cn/sparkjobserver/jars/<application name>

示例：
	
	curl.exe -k -u "myuser:myPass@word1" --data-binary @C:\mylocation\eventhubs-examples\target\spark-streaming-eventhubs-example-0.1.0-jar-with-dependencies.jar https://mysparkcluster.azurehdinsight.cn/sparkjobserver/jars/streamingjar


##<a name="createcontext"></a>在作业服务器中创建新的持久性上下文

	curl.exe -k -u "<hdinsight user>:<user password>" -d "" "https://<cluster name>.azurehdinsight.cn/sparkjobserver/contexts/<context name>?num-cpu-cores=<value>&memory-per-node=<value>"

示例：

	curl.exe -k -u "myuser:myPass@word1" -d "" "https://mysparkcluster.azurehdinsight.cn/sparkjobserver/contexts/mystreaming?num-cpu-cores=4&memory-per-node=1024m"


##<a name="submitapp"></a>将应用程序提交到群集

	curl.exe -k -u "<hdinsight user>:<user password>" -d @<input file name> "https://<cluster name>.azurehdinsight.cn/sparkjobserver/jobs?appName=<app name>&classPath=<class path>&context=<context>"

示例：

	curl.exe -k -u "myuser:myPass@word1" -d @mypostdata.txt "https://mysparkcluster.azurehdinsight.cn/sparkjobserver/jobs?appName=streamingjar&classPath=org.apache.spark.streaming.eventhubs.example.EventCountJobServer&context=mystreaming"

其中的 mypostdata.txt 定义你的应用程序。


##<a name="submitapp"></a>删除作业

	curl.exe -X DELETE -k -u "<hdinsight user>:<user password>" "https://<cluster name>.azurehdinsight.cn/sparkjobserver/contexts/<context>"

示例：

	curl.exe -X DELETE -k -u "myuser:myPass@word1" "https://mysparkcluster.azurehdinsight.cn/sparkjobserver/contexts/mystreaming"


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