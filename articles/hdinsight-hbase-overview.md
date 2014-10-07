<properties linkid="manage-services-hdinsight-hbase-overview" urlDisplayName="HDInsight HBase overview" pageTitle="An overview of HBase in HDInsight | Azure" metaKeywords="" description="An introduction to HBase in HDInsight, use-cases and a comparison with other database solutions ." metaCanonical="" services="hdinsight" documentationCenter="" title="HDInsight HBase overview" authors="bradsev" solutions="big-data" manager="paulettm" editor="cgronlun" />

<tags ms.service="hdinsight" ms.workload="big-data" ms.tgt_pltfrm="na" ms.devlang="na" ms.topic="article" ms.date="08/21/2014" ms.author="bradsev"></tags>

# HDInsight HBase 概述

## 什么是 HBase？

HBase 是构建于 Hadoop 上的 Apache 开源 NoSQL 数据库，用于为大量非结构化和半结构化数据提供随机访问和高度一致性。它在 Google 的 BigTable 上建模，是面向列系列的数据库。数据存储在表的各行中，行中的数据按列系列分组。HBase 是无模式数据库，也就是说，在使用其数据前，不必定义列以及列中存储的数据类型。开源代码最初由 Mike Cafarella 在 2007 年发布，可线性扩展到处理几千个节点上的千万亿字节数据。开源代码可依赖数据冗余、批处理以及 Hadoop 生态系统中的分布式应用程序提供的其他功能。

## 什么是 Azure HDInsight HBase？

HDInsight HBase 以集成到 Azure 环境中的托管群集形式提供。这些群集配置为在 Azure Blob 存储中直接存储数据，这样就降低了延迟，使客户在性能与价格方面做出选择时拥有更大的弹性。客户可以生成用于处理大型数据集的交互式网站，生成用于存储数百万个终结点的传感器数据与遥测数据的服务，并通过 Hadoop 作业来分析这些数据。HBase 和 Hadoop 是在 Azure 中构建大数据项目的良好起点，特别是可以支持实时应用程序来处理大数据集。

HDInsight 实施利用 HBase 的横向扩展架构来提供表自动分片、使读写操作保持高度的一致性，以及支持自动故障转移。性能的提高得益于对读操作和高吞吐量流式写操作进行内存中缓存。

## 如何管理 HDInsight HBase 中的数据？

HBase 中的数据可以使用 HBase shell 中的 `create` `get`、`put` 和 `scan` 命令进行管理。数据使用 `put` 命令写入数据库，而使用 `get` 命令进行读取。`scan` 命令用于从表中的多个行获取数据。Data 也可以使用 HBase C# API 进行管理，该 API 在 HBase REST API 顶部提供客户端库。HBase 数据库也可以使用 Hive 进行查询。有关这些编程模型的简介，请参阅[开始在 HDInsight 中将 HBase 与 Hadoop 配合使用][]。另外，也可以使用协同处理器来处理托管数据库的节点中的数据。

## 方案：HBase 有哪些使用案例？

创建 BigTable 以及延伸开来的 HBase 的典型使用案例就是 web 搜索。搜索引擎构建索引，用于将词语映射到包含这些词语的 web 页面。然而，还有 HBase 适用的许多其他使用案例，本节中列出了其中几个。

### 第 1 种使用案例：键-值存储

HBase 可用作键-值存储，适用于管理消息系统。Facebook 使用 HBase 来管理其消息系统，HBase 最适合用于存储和管理 Internet 通信。WebTable 使用 HBase 来搜索和管理从 web 页面中提取的表。

### 第 2 种使用案例：传感器数据

Hase 可用于捕获从各个来源中逐步收集的数据。这包括社交分析，时间序列，使交互式仪表板与趋势和计数器保持同步，以及管理审计日志系统。具体示例包括：Bloomberg 交易终端
以及开放时间序列数据库 (Open Time Series Database, OpenTSDB)，后者用于存储所收集的服务器系统运行状况指标并对其进行访问。

### 第 3 种使用案例：实时查询

[Phoenix][] 是 Apache HBase 的 SQL 查询引擎。该引擎以 JDBC 驱动程序的形式供用户访问，并且支持使用 SQL 来查询和管理 HBase 表。

### 第 4 种使用案例：HBase 即平台

应用程序可以将 HBase 作为数据存储库而在其上运行。具体示例包括 Phoenix、OpenTSDB、Kiji 和 Titan。应用程序也可以与 HBase 集成。具体示例包括：Hive、Pig、Solr、Storm、Flume、Impala、Spark、Ganglia 和 Drill。

## <a name="next-steps"></a>后续步骤

[开始在 HDInsight 中将 HBase 与 Hadoop 配合使用][]

[在 Azure 虚拟网络上设置 HDInsight 群集][]
<!--
[在 HDInsight 中使用 HBase 分析 Twitter 传感器数据][]
-->
[借助 Maven 生成可将 HBase 与 HDInsight (Hadoop) 配合使用的 Java 应用程序][]

[C# HBase SDK][]

## <a name="see-also"></a>另请参阅

[Apache HBase][]

<!---
[Bigtable：针对结构化数据的分布式存储系统][]
--->
  [开始在 HDInsight 中将 HBase 与 Hadoop 配合使用]: ../hdinsight-hbase-get-started/
  [Phoenix]: http://phoenix.apache.org/
  [在 Azure 虚拟网络上设置 HDInsight 群集]: ../hdinsight-hbase-provision-vnet/
  [在 HDInsight 中使用 HBase 分析 Twitter 传感器数据]: ../hdinsight-hbase-analyze-twitter-sentiment/
  [借助 Maven 生成可将 HBase 与 HDInsight (Hadoop) 配合使用的 Java 应用程序]: ../hdinsight-hbase-build-java-maven/
  [C# HBase SDK]: https://github.com/hdinsight/hbase-sdk-for-net
  [Apache HBase]: https://hbase.apache.org/
  [Bigtable：针对结构化数据的分布式存储系统]: http://research.google.com/archive/bigtable.html
