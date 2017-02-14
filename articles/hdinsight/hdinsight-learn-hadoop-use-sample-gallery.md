<!-- not suitable for Mooncake -->

<properties
   pageTitle="使用示例库了解 HDInsight 中的 Hadoop | Azure"
   description="通过从 HDInsight 入门库运行示例应用程序快速了解 Hadoop。使用示例数据，或提供自己的数据。"
   services="hdinsight"
   documentationCenter=""
   tags="azure-portal"
   authors="mumian"
   manager="paulettm"
   editor="cgronlun"/>

<tags
	ms.service="hdinsight"
	ms.date="10/21/2016"
	wacn.date="02/14/2017"/>

# 使用 Azure HDInsight 入门库了解 Hadoop

借助 HDInsight 入门库，可以通过在 HDInsight 中运行示例应用程序快速轻松地了解 Hadoop。某些示例随附了示例数据。可以为其余示例提供自己的数据。目前，有下面六个示例（还会有更多）：

- 包含 Azure 数据的解决方案
	- Azure 网站日志分析
	- Azure 存储空间分析
- 包含示例数据的解决方案
	- 传感器数据分析
	- Twitter 趋势分析
	- 网站日志分析
	- Mahout 电影推荐

![包括示例数据的 HDInsight Hadoop、Storm 和 HBase 入门库解决方案。][hdinsight.sample.gallery]

可以通过浏览至 http://\<YourHDInsightClusterName\>.azurehdinsight.cn/ 或从 Azure 经典管理门户访问仪表板。

**从入门库运行示例**

1. 登录 [Azure 经典管理门户][azure.portal]。
2. 从左侧菜单中依次单击**浏览**、**HDInsight 群集**和群集名称。
3. 在顶部菜单中单击**仪表板**。
4. 为 HTTP 用户（也称为群集用户）输入用户名和密码。
6. 单击该页顶部的**入门库**。
7. 单击其中一个示例。每个示例都提供了运行它的详细步骤。下图显示了 Twitter 趋势分析示例：

	![HDInsight Twitter 趋势分析示例][hdinsight.twitter.sample]

## 后续步骤
了解 HDInsight 的其他途径包括：

- [HDInsight 信息图][hdinsight.infographic]

<!--Image references-->
[hdinsight.sample.gallery]: ./media/hdinsight-learn-hadoop-use-sample-gallery/HDInsight-Getting-Started-Gallery.png
[hdinsight.twitter.sample]: ./media/hdinsight-learn-hadoop-use-sample-gallery/HDInsight-Twitter-Trend-Analysis-sample.png

<!--Link references-->
[hdinsight.learn.map]: /documentation/articles/hdinsight-learn-map
[hdinsight.infographic]: http://go.microsoft.com/fwlink/?linkid=523960
[azure.portal]: https://manage.windowsazure.cn

<!---HONumber=Mooncake_Quality_Review_1202_2016-->