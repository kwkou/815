<properties 
	pageTitle="将 Hive 用于 Hadoop 以进行网站日志分析 | Azure" 
	description="了解如何通过将 Hive 与 HDInsight 配合使用来分析网站日志。我们将使用日志文件作为 HDInsight 表的输入，并使用 HiveQL 来查询数据。" 
	services="hdinsight" 
	documentationCenter="" 
	authors="nitinme" 
	manager="paulettm" 
	editor="cgronlun"
	tags="azure-portal"/>

<tags
	ms.service="hdinsight"
	ms.date="02/25/2016"
	wacn.date="04/18/2016"/>

# 将 Hive 与 HDInsight 配合使用来分析来自网站的日志

了解如何通过将 HiveQL 与 HDInsight 配合使用来分析来自网站的日志。网站日志分析可用于根据类似活动分类受众，按人口统计分类站点访问者，以及了解他们查看的内容和这些内容来自的网站等。

在此示例中，你将使用 HDInsight 群集来分析网站日志文件，以深入了解每天从外部网站访问网站的频率。你还将生成用户遇到的网站错误的摘要。你将了解如何执行以下操作：

- 连接到 Azure Blob 存储，其中包含网站日志文件。
- 创建 HIVE 表以查询这些日志。
- 创建 HIVE 查询以分析数据。
- 使用 Microsoft Excel 连接到 HDInsight（通过使用开放的数据库连接 (ODBC)），以检索分析的数据。

![HDI.Samples.Website.Log.Analysis][img-hdi-weblogs-sample]

##先决条件

- 必须已在 Azure HDInsight 上预配了一个 Hadoop 群集。有关说明，请参阅[预配 HDInsight 群集][hdinsight-provision]。 
- 你必须已安装 Microsoft Excel 2013 或 Excel 2010。
- 你必须拥有 [Microsoft Hive ODBC 驱动程序](http://www.microsoft.com/download/details.aspx?id=40886)，才能将数据从 Hive 导入 Excel。


##运行示例

1. 在 Azure 经典管理门户中，单击要在其上运行示例的群集，然后单击底部的“查询控制台”。或者，你也可以通过使用以下 URL 直接打开查询控制台：

	 	https://<clustername>.azurehdinsight.cn
	
	出现提示时，通过使用设置群集时所用的管理员用户名和密码进行身份验证。
  
2. 在打开的网页中，单击“入门库”选项卡，然后在“使用示例数据的解决方案”类别下方，单击“网站日志分析”示例。

3. 按照网页上提供的说明完成该示例。

##后续步骤
尝试以下示例：[通过将 Hive 与 HDInsight 配合使用分析传感器数据](/documentation/articles/hdinsight-hive-analyze-sensor-data/)。


[hdinsight-provision]: /documentation/articles/hdinsight-provision-clusters-v1/
[img-hdi-weblogs-sample]: ./media/hdinsight-hive-analyze-website-log/hdinsight-weblogs-sample.png

<!---HONumber=Mooncake_0411_2016-->