<properties
	pageTitle="云中的 Hadoop 是什么？ HDInsight 简介 | Azure"
	description="云中的 Hadoop 是什么，如何在 HDInsight 中管理它？ Hadoop 组件和大数据分析简介。"
	keywords="大数据分析,hadoop 简介,什么是 hadoop,云中的 hadoop"
	services="hdinsight"
	documentationCenter=""
	authors="cjgronlund"
	manager="paulettm"
	editor="cgronlun"/>

<tags
	ms.service="hdinsight"
	ms.date="03/29/2016"
	wacn.date="05/23/2016"/>


# 云中的 Hadoop 是什么？ 介绍如何使用 HDInsight 中的 Hadoop 组件进行大数据分析

获取有关 Azure HDInsight 中的 Hadoop、其生态系统和大数据的简介：什么是 HDInsight 中的 Hadoop？有哪些 Hadoop 组件、常见术语和用于大数据分析的方案？ 此外，了解有关使用 HDInsight 云中 Hadoop 的 Hadoop 教程、文档和资源。

## HDInsight 云中的 Hadoop 是什么？

Azure HDInsight 在云中部署并预配托管的 Apache Hadoop 群集，从而为旨在对大数据进行处理、分析和报告的软件框架提供高可靠性和可用性。HDInsight 使用 **Hortonworks 数据平台 (HDP)** Hadoop 分发版。Hadoop 通常是指组件的整个 Hadoop 生态系统，其中包括 Apache HBase 和 Apache Storm，以及 Hadoop 系列下的其他技术。有关详细信息，请参阅下面的 [HDInsight 中的 Hadoop 生态系统概述](#overview)。


## 什么是大数据？
大数据指的是具有以下特性的数据：收集的数量不断增长、收集速度日益加快、采用的非结构化格式的类型不断扩展且语义上下文灵活多变。

大数据描述各种大型数字信息，从 Twitter 源中的文本到工业设备的传感器信息再到客户浏览在线产品目录并购买的信息。大数据可以是历史数据（即已存储的数据），也可以是实时数据（即从数据源直接流式传输的数据）。

若要使大数据成为可付诸行动的情报或见解，不仅需要收集相关的数据和提出正确的问题，还需要确保这些数据可供访问、清理和分析，然后以有用的方式呈现。这就是有关 HDInsight 中的 Hadoop 的大数据分析的有用之处。


## <a name="overview"></a>HDInsight 中的 Hadoop 生态系统概述

HDInsight 是在 Azure 上快速扩展 Apache Hadoop 技术堆栈（作为大数据分析的首选解决方案）的云实现。它包括 Apache HBase、Storm、Pig、Hive、Sqoop、Oozie 等的实现。HDInsight 还可集成商业智能 (BI) 工具，例如 Power BI、Excel、SQL Server Analysis Services 和 SQL Server Reporting Services。

### Windows 群集

Azure HDInsight 使用 **Windows** 作为基础操作系统，在云中部署并设置 Hadoop 群集。

类别 | Windows 上的 Hadoop
---------|--------------------
**群集 OS** | Windows Server 2012 R2
**群集类型** | Hadoop、HBase、Storm
**部署** | Azure CLI、Azure PowerShell
**群集 UI** | 群集仪表板
**远程访问** | 远程桌面协议 (RDP)、REST API、ODBC、JDBC



### Hadoop、HBase、Storm 和自定义群集

HDInsight 为 Apache Hadoop、HBase 或 Storm 提供了群集配置。或者，你可以[使用脚本操作自定义群集](/documentation/articles/hdinsight-hadoop-customize-cluster-v1/)。

* **Hadoop**（“查询”工作负荷）：为可靠的数据存储提供了 [HDFS](#HDFS) 和一个简单的 [MapReduce](#mapreduce) 编程模型以并行地处理和分析数据。

* **<a target="_blank" href="http://hbase.apache.org/">HBase</a>**（“NoSQL”工作负荷）：构建于 Hadoop 上的 NoSQL 数据库，用于为大量非结构化和半结构化数据（可能为数十亿行乘以数百万列）提供随机访问和高度一致性。请参阅 [HDInsight 中的 HBase 概述](/documentation/articles/hdinsight-hbase-overview/)。

* **<a  target="_blank" href="https://storm.incubator.apache.org/">Apache Storm</a>**（“流”工作负荷）：一个分布式实时计算系统，用于快速处理大型数据流。Storm 以 HDInsight 中的托管群集形式提供。请参阅[使用 Storm 和 Hadoop 分析实时传感器数据](/documentation/articles/hdinsight-storm-sensor-data-analysis/)。

## 有哪些 Hadoop 组件和实用程序？

HDInsight 群集包含以下组件和实用程序。

* **[Avro](#avro)** (Microsoft .NET Library for Avro)：Microsoft .NET 环境的数据序列化。

* **[Hive 和 HCatalog](#hive)**：与结构化查询语言 (SQL) 类似的查询，以及表和存储管理层。

* **[Mahout](#mahout)**：机器学习。

* **[MapReduce](#mapreduce)**：用于 Hadoop 分布式处理和资源管理的旧框架。请参阅 [YARN](#yarn)，它是下一代资源框架。

* **[Oozie](#oozie)**：工作流管理。

* **[Phoenix](#phoenix)**：基于 HBase 的关系数据库层。

* **[Pig](#pig)**：更简单的 MapReduce 转换脚本。

* **[Sqoop](#sqoop)**：数据导入和导出。

* **[Tez](#tez)**：允许数据密集型进程大规模地高效运行。

* **[YARN](#yarn)**：Hadoop 核心库和下一代 MapReduce 软件框架的一部分。

* **[ZooKeeper](#zookeeper)**：协调分布式系统中的进程。

> [AZURE.NOTE] 有关特定组件的信息和版本信息，请参阅 [HDInsight 提供的 Hadoop 群集版本有哪些新功能？][component-versioning]

### <a name="avro"></a>Avro (Microsoft .NET Library for Avro)

Microsoft .NET Library for Avro 针对 Microsoft.NET 环境序列化实现了 Apache Avro 紧凑的二进制数据交换格式。它使用 <a target="_blank" href="http://www.json.org/">JavaScript 对象表示法 (JSON)</a> 定义与语言无关的架构，以支持语言互操作性，这意味着以一种语言序列化的数据可以用另一种语言读取。有关格式的详细信息可以在 <a target=_"blank" href="http://avro.apache.org/docs/current/spec.html">Apache Avro 规范</a>中找到。
Avro 文件格式支持分布式 MapReduce 编程模型。文件是“可拆分的”，意味着你可以在文件中搜寻任一点，然后即可从某一特定块开始读取。若要了解相关方法，请参阅[使用 Microsoft .NET Library for Avro 序列化数据](/documentation/articles/hdinsight-dotnet-avro-serialization/)。


### <a name="HDFS"></a>HDFS

Hadoop 分布式文件系统 (HDFS) 是一种分布式文件系统，采用 MapReduce 和 YARN，是 Hadoop 生态系统的核心。HDFS 是 HDInsight 上 Hadoop 群集的标准文件系统。

### <a name="hive"></a>Hive 和 HCatalog

<a target="_blank" href="http://hive.apache.org/">Apache Hive</a> 是构建于 Hadoop 上的一个数据仓库软件，允许使用类似于 SQL 的语言（称为 HiveQL）来查询和管理分布式存储中的大型数据集。Hive（类似于 Pig）是一种基于 MapReduce 的抽象。在运行时，Hive 会将查询转换为一系列 MapReduce 作业。Hive 比 Pig 在概念上更接近于关系数据库管理系统，因此适用于结构化程度更高的数据。对于非结构化数据，Pig 是更佳的选择。请参阅[将 Hive 与 HDInsight 中的 Hadoop 配合使用](/documentation/articles/hdinsight-use-hive/)。

<a target="_blank" href="https://cwiki.apache.org/confluence/display/Hive/HCatalog/">Apache HCatalog</a> 是 Hadoop 的表和存储管理层，为用户提供数据的关系视图。在 HCatalog 中，你可以读取和写入采用可以写入 Hive SerDe（序列化程序-反序列化程序）的任何格式的文件。

### <a name="mahout"></a>Mahout

<a target="_blank" href="https://mahout.apache.org/">Apache Mahout</a> 是在 Hadoop 上运行的一种可缩放的机器学习算法库。计算机学习应用程序采用统计学原理，使系统学习数据并使用以往的结果来确定将来的行为。请参阅[使用 Hadoop 上的 Mahout 生成电影推荐](/documentation/articles/hdinsight-mahout/)。

### <a name="mapreduce"></a>MapReduce
MapReduce 是一个旧软件框架，用于编写并行批量处理大数据集的应用程序。MapReduce 作业将分割大型数据集并将数据组织成键值对进行处理。

[YARN](#yarn) 是 Hadoop 下一代资源管理器和应用程序框架，也称为 MapReduce 2.0。MapReduce 作业将在 YARN 上运行。

有关 MapReduce 的详细信息，请参阅 Hadoop Wiki 中的 <a target="_blank" href="http://wiki.apache.org/hadoop/MapReduce">MapReduce</a>。

### <a name="oozie"></a>Oozie
<a target="_blank" href="http://oozie.apache.org/">Apache Oozie</a> 是一个管理 Hadoop 作业的工作流协调系统。它与 Hadoop 堆栈集成，支持 MapReduce、Pig、Hive 和 Sqoop 的 Hadoop 作业。它也能用于安排特定于某系统的作业，例如 Java 程序或 shell 脚本。请参阅[将基于时间的 Oozie 协调器与 Hadoop 配合使用](/documentation/articles/hdinsight-use-oozie-coordinator-time/)。

### <a name="phoenix"></a>Phoenix
<a  target="_blank" href="http://phoenix.apache.org/">Apache Phoenix</a> 是基于 HBase 的关系数据库层。Phoenix 提供 JDBC 驱动程序以允许用户直接查询和管理 SQL 表。Phoenix 将查询和其他语句转换为本机 NoSQL API 调用（而不是使用 MapReduce），因此可以在 NoSQL 存储之上实现更快的应用程序。请参阅[将 Apache Phoenix 和 SQuirreL 与 HBase 群集配合使用](/documentation/articles/hdinsight-hbase-phoenix-squirrel/)。


### <a name="pig"></a>Pig
<a  target="_blank" href="http://pig.apache.org/">Apache Pig</a> 是一个高级平台，允许你使用一种简单脚本语言（称为 Pig Latin）对超大型数据集执行复杂的 MapReduce 转换。Pig 转换 Pig Latin 脚本，使其可以在 Hadoop 内运行。可以创建用户定义的函数 (UDF) 来扩充 Pig Latin。请参阅[将 Pig 与 Hadoop 配合使用来分析 Apache 日志文件](/documentation/articles/hdinsight-use-pig/)。

### <a name="sqoop"></a>Sqoop
<a  target="_blank" href="http://sqoop.apache.org/">Apache Sqoop</a> 是一种用于在 Hadoop 和关系数据库（如 SQL）或其他结构化数据存储之间尽可能高效地传输批量数据的工具。请参阅[将 Sqoop 与 Hadoop 配合使用](/documentation/articles/hdinsight-use-sqoop/)。

### <a name="tez"></a>Tez
<a  target="_blank" href="http://tez.apache.org/">Apache Tez</a> 是一个构建于 Hadoop YARN 之上的应用程序框架，用于执行复杂的非循环常规图形数据处理。它是 MapReduce 框架的更灵活、功能更强大的后继，允许数据密集型进程（如 Hive）更高效地大规模运行。请参阅[“使用 Hive 和 HiveQL”中的“使用 Apache Tez 提高性能”](/documentation/articles/hdinsight-use-hive/#usetez)。

### <a name="yarn"></a>YARN
Apache YARN 是下一代 MapReduce（MapReduce 2.0 或 MRv2），它具有更大的可伸缩性和实时处理，支持超越 MapReduce 批处理的数据处理方案。YARN 提供资源管理和分布式应用程序框架。MapReduce 作业将在 YARN 上运行。

若要了解 YARN，请参阅 <a target="_blank" href="http://hadoop.apache.org/docs/current/hadoop-yarn/hadoop-yarn-site/YARN.html">Apache Hadoop 下一代 MapReduce (YARN)</a>。


### <a name="zookeeper"></a>ZooKeeper
<a  target="_blank" href="http://zookeeper.apache.org/">Apache ZooKeeper</a> 通过数据寄存器的共享层次结构命名空间 (znode) 协调大型分布式系统中的进程。Znode 包含协调流程所需的少量元数据信息：状态、位置、配置，等等。

## HDInsight 上的编程语言

HDInsight 群集（Hadoop、HBase 和 Storm 群集）支持多种编程语言，但某些编程语言默认情况下未安装。对于默认情况下未安装的库、模块或程序包，请使用脚本操作来安装组件。请参阅[使用 HDInsight 进行脚本操作开发](/documentation/articles/hdinsight-hadoop-script-actions/)。

### 默认编程语言支持

默认情况下，HDInsight 群集支持：

* Java

* Python

可以使用脚本操作安装其他语言：[使用 HDInsight 进行脚本操作开发](/documentation/articles/hdinsight-hadoop-script-actions/)。

### Java 虚拟机 (JVM) 语言

除了 Java 外，许多语言可以使用 Java 虚拟机 (JVM) 运行；但是，运行其中某些语言可能需要在群集上安装其他组件。

HDInsight 群集支持以下基于 JVM 的语言：

* Clojure

* Jython (Python for Java)

* Scala

### Hadoop 特定的语言

HDInsight 群集为以下特定于 Hadoop 生态系统的语言提供支持：

* 用于 Pig 作业的 Pig Latin

* 用于 Hive 作业的 HiveQL


## <a name="advantage"></a>Hadoop 在云中的优势

作为 Azure 云生态系统的一部分，HDInsight 中的 Hadoop 具有很多优势，其中包括：

* Hadoop 群集自动设置。创建 HDInsight 群集比手动配置 Hadoop 群集容易得多。有关详细信息，请参阅[在 HDInsight 中预配 Hadoop 群集](/documentation/articles/hdinsight-provision-clusters-v1/)。

* 最先进的 Hadoop 组件。有关详细信息，请参阅 [HDInsight 提供的 Hadoop 群集版本有哪些新功能？][component-versioning]。

* 群集具有高可用性和可靠性。已将第二个头节点添加到 HDInsight 所部署的 Hadoop 群集，以增加服务的可用性。Hadoop 群集的标准实现通常具有单个头节点。HDInsight 通过添加辅助头节点去除了此单点故障。切换到新 HA 群集配置不会更改群集的价格，除非客户不使用默认大节点而是使用超大头节点创建群集。
	有关详细信息，请参阅 [HDInsight 中的 Hadoop 群集的可用性和可靠性](/documentation/articles/hdinsight-high-availability/)。

* 使用 Azure Blob 存储（一种与 Hadoop 兼容的选项），数据存储高效又经济。有关详细信息，请参阅[将 Azure Blob 存储与 HDInsight 中的 Hadoop 配合使用](/documentation/articles/hdinsight-hadoop-use-blob-storage/)。

* 与其他 Azure 服务集成，包括 [Web Apps](/documentation/services/web-sites/) 和 [SQL 数据库](/documentation/services/sql-databases/)。
* 更多的 VM 大小。HDInsight 群集支持不同的 VM 类型和大小。HDInsight 群集现在可以利用 A2 到 A7 大小实现常规目的；搭载固态硬盘 (SSD) 和处理器速度提高 60% 的 D 系列节点。Azure HDInsight 上的 Apache HBase 客户可以受益于 D 系列的更高内存配置和性能。Azure HDInsight 上的 Apache Storm 客户还受益于更大的内存，因此可以加载更大的引用数据集，此外，更快的 CPU 可以提高吞吐量。
* 群集缩放。群集缩放使你能够更改正在运行的 HDInsight 群集的节点数，而无需删除或重新创建群集。
* 虚拟网络支持。HDInsight 群集可用于 Azure 虚拟网络，以支持隔离云资源或将云资源与数据中心资源相链接的混合方案。

* 进入成本低。开始[试用](/pricing/1rmb-trial/)或查阅 [HDInsight 定价详细信息](/home/features/hdinsight/pricing/)。


若要详细了解 HDInsight 中的 Hadoop 的优势，请参阅 [HDInsight 的 Azure 功能页][marketing-page]。



## <a id="resources"></a>用于详细了解大数据分析、Hadoop 和 HDInsight 的资源

使用以下资源在此介绍云中 Hadoop 和大数据分析的简介的基础上构建。


### HDInsight 的 Hadoop 文档

* [HDInsight 文档](/documentation/services/hdinsight/)：Azure HDInsight 的文档页，包含指向文章、视频及更多资源的链接。

### Apache Hadoop

* <a target="_blank" href="http://hadoop.apache.org/">Apache Hadoop</a>：了解有关 Apache Hadoop 软件库的详细信息，这是一个框架，允许你跨计算机群集分布式处理大型数据集。

* <a target="_blank" href="http://hadoop.apache.org/docs/r1.0.4/hdfs_design.html">HDFS</a>：详细了解 Hadoop 分布式文件系统的体系结构和设计，Hadoop 分布式文件系统是由 Hadoop 应用程序使用的主存储系统。

* <a target="_blank" href="http://hadoop.apache.org/docs/r1.2.1/mapred_tutorial.html">MapReduce 教程</a>：了解有关编程框架的详情，此编程框架用于编写 Hadoop 应用程序，以便并行地快速处理大型计算节点群集上的大量数据。

### Azure 上的 SQL 数据库

* [Azure SQL 数据库](/documentation/services/sql-databases/)：有关 SQL 数据库的文档、教程和视频。

* [Adventure Works for SQL 数据库](http://msftdbprodsamples.codeplex.com/releases/view/37304)：SQL 数据库示例数据库的下载页。

### Microsoft 商业智能（适用于 Windows 上的 HDInsight）

大家熟悉的商业智能 (BI) 工具（如 Excel、PowerPivot、SQL Server Analysis Services 和 SQL Server Reporting Services）使用 Power Query 外接程序或 Microsoft Hive ODBC 驱动程序来检索、分析和报告与 HDInsight 集成的数据。

这些 BI 工具可有助于大数据分析：

* [使用 Power Query 将 Excel 连接到 Hadoop](/documentation/articles/hdinsight-connect-excel-power-query/)：了解如何使用 Microsoft Power Query for Excel，将 Excel 连接到存储 HDInsight 群集关联数据的 Azure 存储帐户。

* [使用 Microsoft Hive ODBC 驱动程序将 Excel 连接到 Hadoop](/documentation/articles/hdinsight-connect-excel-hive-ODBC-driver/)：了解如何使用 Microsoft Hive ODBC 驱动程序从 HDInsight 导入数据。

* <a target="_blank" href="http://msdn.microsoft.com/zh-cn/library/hh231701.aspx">详细了解 SQL Server Analysis Services</a>。

* <a target="_blank" href="http://msdn.microsoft.com/zh-cn/library/ms159106.aspx">了解 SQL Server Reporting Services</a>。


### 尝试使用 Hadoop 解决方案来分析大数据（适用于 Windows 上的 HDInsight）

对组织数据使用大数据分析以洞察你的业务。下面是一些示例：

* [分析 HVAC 传感器数据](/documentation/articles/hdinsight-hive-analyze-sensor-data/)：了解如何通过将 Hive 与 HDInsight (Hadoop) 配合使用来分析传感器数据，然后在 Microsoft Excel 中实现数据的可视化。在此例中，你将使用 Hive 处理 HVAC 系统生成的历史数据，从而了解哪些系统无法可靠地维持设定的温度。

* [将 Hive 与 HDInsight 配合使用来分析网站日志](/documentation/articles/hdinsight-hive-analyze-website-log/)：了解如何使用 HDInsight 中的 HiveQL 来分析网站日志，从而了解来自外部网站的一天访问次数以及用户遇到的网站错误摘要。

* [使用 HDInsight (Hadoop) 中的 Storm 和 HBase 实时分析传感器数据](/documentation/articles/hdinsight-storm-sensor-data-analysis/)：了解如何构建一个解决方案，使用 HDInsight 中的 Storm 群集处理 Azure 事件中心的传感器数据，然后将处理后的传感器数据以近乎实时的信息显示在基于 Web 的仪表板上。


[marketing-page]: /home/features/hdinsight/
[component-versioning]: /documentation/articles/hdinsight-component-versioning-v1/
[zookeeper]: http://zookeeper.apache.org/

<!---HONumber=Mooncake_0516_2016-->