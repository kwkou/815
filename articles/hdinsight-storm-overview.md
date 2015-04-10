<properties title="Storm in HDInsight overview" pageTitle="了解 HDInsight 中的 Apache Storm (hadoop)" description="Learn how you can use Apache Storm in HDInsight (Hadoop)" metaKeywords="Azure hdinsight storm, Azure hdinsight realtime, azure hadoop storm, azure hadoop realtime, azure hdinsight real-time, azure hadoop storm real-time, aure hadoop real-time" services="hdinsight" solutions="" documentationCenter="big-data" authors="larryfr" videoId="" scriptId="" manager="paulettm" editor="cgronlun"/>

<tags ms.service="hdinsight" ms.workload="big-data" ms.tgt_pltfrm="na" ms.devlang="na" ms.topic="article" ms.date="09/30/2014" ms.author="larryfr" />

# HDinsight Storm 概述

## 什么是 Storm？

[Apache Storm][apachestorm] 是分布式可容错开源计算系统，可用来实时处理数据。Storm 解决方案还提供有保障的数据处理功能，能够重新处理第一次未成功处理的数据。

## 什么是 Azure HDInsight Storm？

HDInsight Storm 以集成到 Azure 环境的管理群集形式提供，它在 Azure 环境中用作大型 Azure 解决方案的一部分。例如，Storm 可能使用来自 ServiceBus 队列或事件中心等服务的数据，并使用网站或云服务提供数据可视化。HDInsight Storm 群集还可以在 Azure 虚拟网络上配置，这可减少与同一虚拟网络上的其他资源进行通信的延迟，并且还可与专用数据中心的资源进行安全通信。

若要开始使用 Storm，请参阅 [HDInsight 中的 Storm 入门][gettingstarted]。

## 如何处理 HDInsight Storm 中的数据？

storm 群集处理**拓扑**而不是您所熟悉的 HDInsight 或 Hadoop 中的 MapReduce 作业。Storm 群集包含两种类型的节点，分别是运行 **Nimbus** 的头节点和运行 **Supervisor** 的工作节点。

* **Nimbus** - 类似于 Hadoop 中的 JobTracker，负责在整个群集中分发节点、将任务分配给计算机以及监视故障情况。HDInsight 提供两个 Nimbus 节点，因此 Storm 群集不会出现单点故障

* **Supervisor** - 每个工作节点的 supervisor 负责开始和停止该节点上的**工作进程**

* **工作进程** - 工作进程运行**拓扑**的一个子集。一个正在运行的拓扑分布于整个群集的很多工作进程上。

* **拓扑** - 定义处理数据**流**的计算图形。与 MapReduce 作业不同，拓扑运行到您停止它们为止

* **流** - 一个未绑定的**元组**集合。流由 **spout** 和 **bolt** 生成，并且由 **bolt** 使用

* **元组** - 动态类型值的一个命名列表

* **Spout** - 使用数据源的数据并发出一个或多个**流**

	> [WACOM.NOTE] 在许多情况下，从 Kafka、Azure ServiceBus 队列或事件中心等队列中读取数据。队列确保发生中断时数据持续不断。

* **Bolt** - 使用**流**，处理**元组**，并可以发出**流**。Bolt 还负责将数据编写到外部存储，比如队列、HDInsight HBase、blob 或其他数据存储

* **Thrift** - Apache Thrift 是用于可扩展式跨语言服务开发的软件框架。可用于构建在 C++、Java、Python、PHP、Ruby、Erlang、Perl、Haskell、C#、Cocoa、JavaScript、Node.js、Smalltalk 及其他语言间工作的服务。

	* **Nimbus** 是 Thrift 服务，**拓扑**是 Thrift 定义，因此可以使用各种编程语言来开发拓扑

有关 Storm 组件的详细信息，请参阅 apache.org 上的 [Storm 教程][apachetutorial]。

## 方案：Storm 有哪些使用案例？

以下是您可能使用 Apache storm 的一些常见方案。有关实际方案的信息，请参阅[公司如何使用 Storm][poweredby]。

### 实时分析

由于 storm 处理实时数据流，其最适合需要查找和响应到达时数据流中的特定事件或模式的数据分析。例如，Storm 拓扑可以监视传感器数据来确定系统运行状况，并在出现特定模式时生成 SMS 消息来提醒您。

### 提取转换加载 (ETL)

ETL 几乎可以认为是 Storm 处理的副作用。例如，如果您正在通过实时分析执行欺诈检测，那么您已经在引入和转换数据。您可能还想让 bolt 在 HBase、Hive 或其他数据存储中存储数据，以供将来分析使用。

### 分布式 RPC

分布式 RPC 是一种可以使用 Storm 创建的模式。请求被传递给 Storm，然后 Storm 会跨多个节点来分发计算，最后将结果流返回到等待的客户端。

有关 Storm 提供的分布式 RPC 和 DRPCClient 的详细信息，请参阅[分布式 RPC](https://storm.incubator.apache.org/documentation/Distributed-RPC.html)。

### 在线计算机学习

Storm 可用于之前已通过批处理培训的计算机学习解决方案，比如基于 Mahout 的解决方案。但其通用的分布式计算模型也适用于基于流的计算机学习解决方案。例如，[可扩展的高级大规模在线分析 (SAMOA) 项目][samoa]是使用流处理并可以与 Storm 配合使用的计算机学习库。

## 我可以使用哪些编程语言？

HDInsight Storm 群集对 .NET、Java 和 Python 提供开箱即用的支持。虽然 Storm [支持其他语言](https://storm.incubator.apache.org/about/multi-language.html)，但除了需要更改其他配置外，其中有许多还需要在 HDInsight 群集上安装其他编程语言。 

### .NET

SCP 是一种可让 .NET 开发人员设计和实现拓扑（包括 spout 和 bolt）的项目。HDInsight Storm 群集默认支持 SCP。

有关使用 SCP 进行开发的详细信息，请参阅[在 HDInsight 中的 Storm 上使用 SCP.NET 和 C# 开发流式数据处理应用程序](/zh-cn/documentation/articles/hdinsight-hadoop-storm-scpdotnet-csharp-develop-streaming-data-processing-application)。

### Java

您遇到的大多数 Java 示例都是无格式 Java 或 Trident。Trident 是一个高级别抽象，可更轻松地执行联接、汇总、分组和筛选等操作。但是，Trident 作用于批量元组，其中原始 Java 解决方案一次将处理一个元组流。

有关 Trident 的详细信息，请参阅 apache.org 上的 [Trident 教程](https://storm.incubator.apache.org/documentation/Trident-tutorial.html)。

有关原始 Java 和 Trident 拓扑的示例，请参阅 HDInsight Storm 群集上的 **%storm_home%\contrib\storm-starter** 目录。

## 常见的开发模式有哪些？

### 有保证的消息处理

Storm 可以提供不同级别的有保证的消息处理。例如，基本的 Storm 应用程序至少可以保证一次处理，而 Trident 仅可以保证一次处理。

有关详细信息，请参阅 apache.org 上的[数据处理保证](https://storm.apache.org/about/guarantees-data-processing.html)

### BasicBolt

读取输入元组，发出零个或多个元组，然后在执行方法结束时立即询问输入元组，这种模式非常普通，以至 Storm 提供 [IBasicBolt](https://storm.apache.org/apidocs/backtype/storm/topology/IBasicBolt.html) 接口来自动执行这种模式。

### 联接

在应用程序之间联接两个数据流的方式将有所不同。例如，您可以从多个流将每个元组联接到一个新流，也可以仅联接特定窗口的批量元组。两种方式的联接都可以使用 [fieldsGrouping](http://javadox.com/org.apache.storm/storm-core/0.9.1-incubating/backtype/storm/topology/InputDeclarer.html#fieldsGrouping%28java.lang.String,%20backtype.storm.tuple.Fields%29) 来实现，这是定义元组如何路由到 bolt 的方式。

在以下 Java 示例中，fieldsGrouping 用于将来自组件"1"、"2"和"3"的元组路由至 **MyJoiner** bolt。

	builder.setBolt("join", new MyJoiner(), parallelism) .fieldsGrouping("1", new Fields("joinfield1", "joinfield2")) .fieldsGrouping("2", new Fields("joinfield1", "joinfield2")) .fieldsGrouping("3", new Fields("joinfield1", "joinfield2")); 

### 批处理

批处理可以通过若干方式来实现。利用基本 Storm Java 拓扑，您可以在发出元组前使用简单计数器对 X 个元组进行批处理，或使用称为计时周期元组的内部计时机制每 X 秒发出一批元组。

有关使用计时周期的示例，请参阅[使用 Storm 和 HDInsight 分析传感器数据](/zh-cn/documentation/articles/hdinsight-storm-sensor-data-analysis.md)

如果您使用的是 Trident，则其基于批量处理元组。

### Caching

内存缓存通常用作加速处理的机制，因为它在内存中存储常用资产。由于拓扑分布于多个节点，并且每个节点中有多个进程，您应考虑使用 [fieldsGrouping](http://javadox.com/org.apache.storm/storm-core/0.9.1-incubating/backtype/storm/topology/InputDeclarer.html#fieldsGrouping%28java.lang.String,%20backtype.storm.tuple.Fields%29) 来确保用于缓存查询的包含字段的元组始终路由至同一进程。这可以避免在进程间重复缓存条目。

### 流式处理 top N

当拓扑依赖于计算"top"N 值（比如 Twitter 上的前 5 大趋势）时，您应并行计算 top N 值，然后将这些计算的输出合并到全局值中。这可以使用 [fieldsGrouping](http://javadox.com/org.apache.storm/storm-core/0.9.1-incubating/backtype/storm/topology/InputDeclarer.html#fieldsGrouping%28java.lang.String,%20backtype.storm.tuple.Fields%29) 按字段值路由至并行 bolt 来完成，其按字段值对数据进行划分，然后最终路由至从全局确定 top N 值的 bolt。

有关示例，请参阅 [RollingTopWords](https://github.com/nathanmarz/storm-starter/blob/master/src/jvm/storm/starter/RollingTopWords.java) 示例。

## 后续步骤

* [HDInsight 中的 Storm 入门][gettingstarted]

* [使用 Storm 和 HDInsight 分析传感器数据](/zh-cn/documentation/articles/hdinsight-storm-sensor-data-analysis)

* [在 HDInsight 中的 Storm 上使用 SCP.NET 和 C# 开发流式数据处理应用程序](/zh-cn/documentation/articles/hdinsight-hadoop-storm-scpdotnet-csharp-develop-streaming-data-processing-application)

[apachestorm]: https://storm.incubator.apache.org
[stormtrident]: https://storm.incubator.apache.org/documentation/Trident-API-Overview.html
[samoa]: http://yahooeng.tumblr.com/post/65453012905/introducing-samoa-an-open-source-platform-for-mining
[apachetutorial]: https://storm.incubator.apache.org/documentation/Tutorial.html
[poweredby]: https://storm.incubator.apache.org/documentation/Powered-By.html
[gettingstarted]: /zh-cn/documentation/articles/hdinsight-storm-getting-started

