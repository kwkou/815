<!-- not suitable for Mooncake -->

<properties
    pageTitle="Apache Kafka on HDInsight 简介 | Azure"
    description="了解 Apache Kafka on HDInsightKafka 的简介与功能，以及可在何处找到示例和入门信息。"
    services="hdinsight"
    documentationcenter=""
    author="Blackmist"
    manager="jhubbard"
    editor="cgronlun" />
<tags 
    ms.assetid="f284b6e3-5f3b-4a50-b455-917e77588069"
    ms.service="hdinsight"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="big-data"
    ms.date="11/10/2016"
    wacn.date="02/06/2017"
    ms.author="larryfr" />

# Apache Kafka on HDInsight（预览版）简介

[Apache Kafka](https://kafka.apache.org) 是开源分布式流式处理平台，可用于构建实时流数据管道和应用程序。Kafka 还提供类似于消息队列的消息中转站功能，可在其中向命名的数据流发布和订阅信息。Kafka on HDInsight 在 Azure 云中提供托管的、高度可缩放且高度可用的服务。

## 为何使用 Kafka on HDInsight？

Kafka 提供以下功能：

* 发布-订阅消息模式：Kafka 提供生成者 API 用于向 Kafka 主题发布记录。订阅某个主题时，将会用到使用者 API。

* 流式处理：执行实时流处理时，往往会将 Kafka 与 Apache Storm 或 Spark 配合使用。Kafka 0.10.0.0（HDInsight 版本 3.5）中引入了流式处理 API，可用于构建流式处理解决方案，而无需使用 Storm 或 Spark。

* 横向缩放：Kafka 可在 HDInsight 群集中的节点之间将流分区。使用者进程可与单个分区相关联，在使用记录时提供负载均衡。

* 有序传送：在每个分区内，记录将按接收顺序存储在流中。通过在使用者进程与分区之间建立一对一的关联，可以保证记录按顺序处理。

* 容错：可以在节点之间复制分区以提供容错能力。

## 用例

* **消息传送**：由于支持发布-订阅消息模式，Kafka 通常用作消息中转站。

* **活动跟踪**：由于 Kafka 提供有序的日志记录，因此可用于跟踪和重建活动，例如，网站上或应用程序中的用户操作。

* **聚合**：使用流处理可以从不同的流聚合信息，然后将信息组合并集中化为操作数据。

* **转换**：使用流处理可将多个输入主题的数据组合并扩充为一个或多个输出主题。

## 从何入手？

请参阅 [Get started with Kafka on HDInsight](/documentation/articles/hdinsight-apache-kafka-get-started/)（Kafka on HDInsight 入门），了解有关创建 Kafka 群集和使用 Kafka 的步骤，包括演示如何使用生成者、使用者和流式处理 API 的基于 Java 的示例

## 后续步骤

单击以下链接了解如何使用 Apache Kafka on HDInsight：

* [Get started with Kafka on HDInsight](/documentation/articles/hdinsight-apache-kafka-get-started/)（Kafka on HDInsight 入门）

* [Use MirrorMaker to create a replica of Kafka on HDInsight](/documentation/articles/hdinsight-apache-kafka-mirroring/)（使用 MirrorMaker 在 HDInsight 上创建 Kafka 的副本）

* [Use Apache Storm with Kafka on HDInsight](/documentation/articles/hdinsight-apache-storm-with-kafka/)（将 Apache Storm 与 Kafka on HDInsight 配合使用）

* [Use Apache Spark with Kafka on HDInsight](/documentation/articles/hdinsight-apache-spark-with-kafka/)（将 Apache Spark 与 Kafka on HDInsight 配合使用）

<!---HONumber=Mooncake_1219_2016-->