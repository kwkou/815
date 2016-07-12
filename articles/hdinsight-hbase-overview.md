<properties
	pageTitle="HDInsight 中的 HBase 是什么？| Azure"
	description="介绍 HDInsight 中的 Apache HBase - 在 Hadoop 上构建的 NoSQL 数据库。了解相关用例并将 HBase 与其他 Hadoop 群集进行比较。"
	keywords="bigtable,nosql,what is hbase"
	services="hdinsight"
	documentationCenter=""
	tags="azure-portal"
	authors="mumian" 
	manager="paulettm" 
	editor="cgronlun"/>

<tags 
	ms.service="hdinsight" 
	ms.date="04/05/2016"
	wacn.date="05/24/2016"/>



# HDInsight 中的 HBase 是什么：为 Hadoop 提供类似于 BigTable 的功能的 NoSQL 数据库

Apache HBase 是一种开源 NoSQL 数据库，它构建于 Hadoop 基础之上，并基于 Google BigTable 模型化。HBase 针对按列系列组织的无架构数据库中的大量非结构化和结构化数据提供随机访问和强一致性。

数据存储在表的各行中，行中的数据按列系列分组。HBase 是无架构数据库，也就是说，在使用其数据前，不必定义列以及列中存储的数据类型。开放源代码可进行线性伸缩，以处理上千节点上数 PB 的数据。开放源代码可依赖数据冗余、批处理以及 Hadoop 生态系统中的分布式应用程序提供的其他功能。

## 如何在 Azure HDInsight 中实施 HBase？

HDInsight HBase 以集成到 Azure 环境中的托管群集形式提供。这些群集配置为在 Azure Blob 存储中直接存储数据，这样就减少了延迟，并提高了选择性能和价格的灵活性。这样，客户便可构建用于处理大型数据集的交互式 Web 应用，构建用于存储数百万个终结点的传感器数据与遥测数据的服务，并通过 Hadoop 作业来分析这些数据。HBase 和 Hadoop 是在 Azure 中构建大数据项目的良好起点，特别是可以支持实时应用程序来处理大数据集。

HDInsight 实施利用 HBase 的横向扩展架构来提供表自动分片、使读写操作保持高度的一致性，以及支持自动故障转移。性能可通过对读取使用内存中缓存并对写入使用高吞吐量流式处理来提高。可视化网络设置也适用于 HDInsight HBase。有关详细信息，请参阅[在 Azure 虚拟网络上预配 HDInsight 群集][hbase-provision-vnet]。

## 如何在 HDInsight HBase 中管理数据？
数据可以在 HBase 中通过使用 HBase shell 中的 **create**、**get**、**put** 和 **scan** 命令来管理。数据通过使用 **put** 写入到数据库，并通过使用 **get** 读取。**scan** 命令用于从表中的多行获得数据。Data 也可以使用 HBase C# API 进行管理，该 API 在 HBase REST API 顶部提供客户端库。HBase 数据库还可以通过使用 Hive 进行查询。有关这些编程模型的简介，请参阅[开始在 HDInsight 中将 HBase 与 Hadoop 配合使用][hbase-get-started]。共同的处理器也适用，这样，便可在托管数据库的节点中处理数据。
 

## 方案：HBase 用例
创建 BigTable 以及延伸开来的 HBase 的典型用例就是 Web 搜索。搜索引擎构建索引，用于将词语映射到包含这些词语的 web 页面。然而，还有 HBase 适用的许多其他用例，本部分中列出了其中几个。

- 键值存储：HBase 可用作键值存储，适用于管理消息系统。Facebook 使用 HBase 作为消息系统，它最适用于存储和管理 Internet 通信。WebTable 使用 HBase 搜索和管理从网页中提取的表。

- 传感器数据：HBase 用于捕获从各种源逐步收集的数据。这包括社交分析、时间序列、使交互式仪表板与趋势和计数器保持同步，以及管理审计日志系统。具体示例包括：Bloomberg 交易终端以及开放时间序列数据库 (Open Time Series Database, OpenTSDB)，后者用于存储所收集的服务器系统运行状况指标并对其进行访问。

- 实时查询：[Phoenix](http://phoenix.apache.org/) 是 Apache HBase 的 SQL 查询引擎。该引擎以 JDBC 驱动程序的形式供用户访问，并且支持使用 SQL 来查询和管理 HBase 表。

- HBase 即平台：应用程序可以将 HBase 作为数据存储库而在其上运行。具体示例包括 Phoenix、OpenTSDB、Kiji 和 Titan。应用程序也可以与 HBase 集成。具体示例包括：Hive、Pig、Solr、Storm、Flume、Impala、Ganglia 和 Drill。


##<a name="next-steps"></a>后续步骤

- [开始在 HDInsight 中将 HBase 与 Hadoop 配合使用][hbase-get-started]
- [在 Azure 虚拟网络上设置 HDInsight 群集][hbase-provision-vnet]
- [在 HDInsight 中配置 HBase 复制](/documentation/articles/hdinsight-hbase-geo-replication/) 
- [借助 Maven 生成可将 HBase 与 HDInsight (Hadoop) 配合使用的 Java 应用程序][hbase-build-java-maven]

##<a name="see-also"></a>另请参阅

- [Apache HBase](https://hbase.apache.org/)




[hbase-provision-vnet]: /documentation/articles/hdinsight-hbase-provision-vnet-v1/
[hbase-build-java-maven]: /documentation/articles/hdinsight-hbase-build-java-maven/
[hdinsight-use-hive]: /documentation/articles/hdinsight-use-hive/
[hdinsight-storage]: /documentation/articles/hdinsight-use-blob-storage/
[hbase-get-started]: /documentation/articles/hdinsight-hbase-tutorial-get-started-v1/

[azure-purchase-options]: /pricing/overview/

[azure-trial]: /pricing/1rmb-trial/
[azure-management-portal]: https://manage.windowsazure.cn/
[azure-create-storageaccount]: /documentation/articles/storage-create-storage-account/
[apache-hadoop]: http://hadoop.apache.org/

<!---HONumber=76-->