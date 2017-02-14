<properties
    pageTitle="什么是 Hadoop？ Azure HDInsight 简介 | Azure"
    description="获取 HDInsight 上的 Hadoop 生态系统和组件的简介。HDInsight 包括 Hadoop、Spark、HBase 等，用于大数据处理和分析。"
    keywords="大数据分析,hadoop 简介,什么是 hadoop,hadoop 技术堆栈,hadoop 生态系统"
    services="hdinsight"
    documentationcenter=""
    author="cjgronlund"
    manager="jhubbard"
    editor="cgronlun" />
<tags
    ms.assetid="e56a396a-1b39-43e0-b543-f2dee5b8dd3a"
    ms.service="hdinsight"
    ms.devlang="na"
    ms.topic="get-started-article"
    ms.tgt_pltfrm="na"
    ms.workload="big-data"
    ms.date="12/14/2016"
    wacn.date="01/25/2017"
    ms.author="cgronlun" />  


# Azure HDInsight 上的 Hadoop 生态系统简介
 本文简单介绍了 Azure HDInsight 上的 Hadoop 及其生态系统和大数据。了解 Hadoop 组件、常用术语以及大数据分析方案。

## HDInsight 上的 Hadoop 是什么？
Hadoop 是指一个由开源软件组成的生态系统，其所构成的框架适用于对计算机群集上的大数据集进行分布式处理、存储和分析。Azure HDInsight 使 **Hortonworks 数据平台 (HDP)** 分发中的 Hadoop 组件能够在云中使用，可部署具有高可靠性和可用性的托管群集，并使用 Active Directory 提供企业级安全性和监管。

Apache Hadoop 是进行大数据处理的原始开源项目。下面是被视为 Hadoop 技术堆栈一部分的相关软件和实用程序的开发情况，包括 Apache Hive、Apache HBase、Apache Spark 等。有关详细信息，请参阅 [HDInsight 中的 Hadoop 生态系统概述](#overview)。

## 什么是大数据？
大数据描述各种大型数字信息，从 Twitter 源中的文本到工业设备的传感器信息，再到客户在网站上进行浏览和购买的信息。大数据可以是历史数据（即已存储的数据），也可以是实时数据（即从数据源直接流式传输的数据）。大数据的收集量日益增加，收集速度越来越快，收集格式也越来越多。

若要通过分析大数据来提供可行的情报或见解，必须收集相关数据并提出适当的问题。此外还需确保数据可供访问，已进行过清理和分析，并以可用方式提供。这就是有关 HDInsight 中的 Hadoop 的大数据分析的有用之处。

## <a name="overview"></a>HDInsight 中的 Hadoop 生态系统概述
HDInsight 是快速扩展的适用于大数据分析的 Apache Hadoop 技术堆栈在 Azure 上的云分发。它包括 Apache Spark、HBase、Storm、Pig、Hive、Interactive Hive、Sqoop、Oozie、Ambari 等的实现。HDInsight 还可集成商业智能 (BI) 工具，例如 Power BI、Excel、SQL Server Analysis Services 和 SQL Server Reporting Services。

### Hadoop、HBase、Spark、Interactive Hive、Storm、自定义群集和其他群集
HDInsight 提供以下群集类型：

* **[Apache Hadoop](https://wiki.apache.org/hadoop)**：为可靠的数据存储提供了 [HDFS](#hdfs) 和一个简单的 [MapReduce](#mapreduce) 编程模型，以并行地处理和分析数据。
* **[Apache Spark](http://spark.apache.org/)**：一种并行处理框架，支持内存中处理，以提升大数据分析应用程序的性能；Spark 适用于 SQL、流式数据处理和机器学习。请参阅[概述：什么是 HDInsight 中的 Apache Spark？](/documentation/articles/hdinsight-apache-spark-overview/)
* **[Apache HBase](http://hbase.apache.org/)**：构建于 Hadoop 上的 NoSQL 数据库，用于为大量非结构化和半结构化数据（可能为数十亿行乘以数百万列）提供随机访问和高度一致性。请参阅 [HDInsight 中的 HBase 概述](/documentation/articles/hdinsight-hbase-overview/)。
* **[Microsoft R Server](https://msdn.microsoft.com/microsoft-r/rserver)**：用于托管和管理并行分布式 R 进程的企业级服务器。它可让数据科研人员、统计人员和 R 程序员根据需要访问 HDInsight 上可缩放的分布式分析方法。请参阅 [HDInsight 上的 R Server 概述](https://docs.microsoft.com/azure/hdinsight/hdinsight-hadoop-r-server-overview)。
* **[Apache Storm](https://storm.incubator.apache.org/)**：一个分布式实时计算系统，用于快速处理大型数据流。Storm 以 HDInsight 中的托管群集形式提供。请参阅[使用 Storm 和 Hadoop 分析实时传感器数据](/documentation/articles/hdinsight-storm-sensor-data-analysis/)。
* **[Apache Interactive Hive 预览版（AKA：Live Long and Process）](https://cwiki.apache.org/confluence/display/Hive/LLAP)**：内存中缓存用于实现交互式且更快的 Hive 查询。请参阅[在 HDInsight 中使用 Interactive Hive](https://docs.microsoft.com/azure/hdinsight/hdinsight-hadoop-use-interactive-hive)。
* **[已加入域的群集预览版](https://docs.microsoft.com/azure/hdinsight/hdinsight-domain-joined-introduction)**：已加入到 Active Directory 域，以便用户可以控制访问权限并提供数据管理的群集。
* **[使用脚本操作的自定义群集](/documentation/articles/hdinsight-hadoop-customize-cluster-linux/)**：使用在预配期间运行的脚本安装其他组件的群集。

### 自定义脚本示例
脚本操作是在群集预配期间运行的 Linux 上的 Bash 脚本，可用于在群集上安装其他组件。

以下脚本示例由 HDInsight 团队提供：

* [Hue](/documentation/articles/hdinsight-hadoop-hue-linux/)：一组 Web 应用程序，用来与群集交互。仅 Linux 群集。
* [Giraph](/documentation/articles/hdinsight-hadoop-giraph-install-linux/)：通过图形处理对事物或人员之间的关系建模。
* [R](/documentation/articles/hdinsight-hadoop-r-scripts/)：机器学习中用于统计计算的开源语言和环境。
* [Solr](/documentation/articles/hdinsight-hadoop-solr-install-linux/)：允许对数据进行全文搜索的企业级搜索平台。

有关开发你自己的脚本操作的信息，请参阅[使用 HDInsight 进行脚本操作开发](/documentation/articles/hdinsight-hadoop-script-actions-linux/)。

## 有哪些 Hadoop 组件和实用程序？
HDInsight 群集包含以下组件和实用程序。

* **[Ambari](#ambari)**：群集预配、管理、监视和实用程序。
* **[Avro](#avro)** (Microsoft .NET Library for Avro)：Microsoft .NET 环境的数据序列化。
* **[Hive 和 HCatalog](#hive)**：与结构化查询语言 (SQL) 类似的查询，以及表和存储管理层。
* **[Mahout](#mahout)**：适用于可缩放的机器学习应用程序。
* **[MapReduce](#mapreduce)**：Hadoop 分布式处理和资源管理的旧框架。参见下一代资源框架 [YARN](#yarn)。
* **[Oozie](#oozie)**：工作流管理。
* **[Phoenix](#phoenix)**：基于 HBase 的关系数据库层。
* **[Pig](#pig)**：更简单的 MapReduce 转换脚本。
* **[Sqoop](#sqoop)**：数据导入和导出。
* **[Tez](#tez)**：让数据密集型进程能够大规模高效运行。
* **[YARN](#yarn)**：Hadoop 核心库和下一代 MapReduce 软件框架的一部分。
* **[ZooKeeper](#zookeeper)**：协调分布式系统中的进程。

> [AZURE.NOTE]
有关特定组件的信息和版本信息，请参阅 [HDInsight 中的 Hadoop 组件、版本和服务产品][component-versioning]
>
>

### <a name="ambari"></a>Ambari
Apache Ambari 用于设置、管理和监视 Apache Hadoop 群集。它包括一系列直观的操作员工具和一组免除 Hadoop 复杂性的可靠 API，可简化群集操作。Linux 上的 HDInsight 群集提供 Ambari Web UI 和 Ambari REST API，而 Windows 上的群集提供 REST API 的子集。HDInsight 群集上的 Ambari 视图允许 UI 插件功能。

请参阅[使用 Ambari 管理 HDInsight 群集](/documentation/articles/hdinsight-hadoop-manage-ambari/)（仅限 Linux）、[使用 Ambari API 监视 HDInsight 中的 Hadoop 群集](/documentation/articles/hdinsight-monitor-use-ambari-api/)和 <a target="_blank" href="https://github.com/apache/ambari/blob/trunk/ambari-server/docs/api/v1/index.md">Apache Ambari API 参考</a>。

### <a name="avro"></a>Avro (Microsoft .NET Library for Avro)
Microsoft .NET Library for Avro 针对 Microsoft.NET 环境序列化实现了 Apache Avro 紧凑的二进制数据交换格式。它使用 <a target="_blank" href="http://www.json.org/">JavaScript 对象表示法 (JSON)</a> 定义与语言无关的架构，以支持语言互操作性，这意味着可以用一种语言读取以另一种语言序列化的数据。<a target=_"blank" href="http://avro.apache.org/docs/current/spec.html">Apache Avro 规范</a>中可找到有关格式的详细信息。Avro 文件格式支持分布式 MapReduce 编程模型。文件是“可拆分的”，这意味着可搜寻文件中的任意点，并可从某特定块开始读取。若要了解相关方法，请参阅[使用 Microsoft .NET Library for Avro 序列化数据](/documentation/articles/hdinsight-dotnet-avro-serialization/)。

### <a name="hdfs" id="HDFS"></a>HDFS
Hadoop 分布式文件系统 (HDFS) 是一种分布式文件系统，采用 MapReduce 和 YARN，是 Hadoop 生态系统的核心。HDFS 是 HDInsight 上 Hadoop 群集的标准文件系统。

### <a name="hive"></a>Hive 和 HCatalog
<a target="_blank" href="http://hive.apache.org/">Apache Hive</a> 是构建于 Hadoop 上的一个数据仓库软件，允许使用类似于 SQL 的语言（称为 HiveQL）来查询和管理分布式存储中的大型数据集。Hive（类似于 Pig）是一种基于 MapReduce 的抽象。在运行时，Hive 会将查询转换为一系列 MapReduce 作业。Hive 比 Pig 在概念上更接近于关系数据库管理系统，因此适用于结构化程度更高的数据。对于非结构化数据，Pig 是更佳的选择。请参阅[将 Hive 与 HDInsight 中的 Hadoop 配合使用](/documentation/articles/hdinsight-use-hive/)。

<a target="_blank" href="https://cwiki.apache.org/confluence/display/Hive/HCatalog/">Apache HCatalog</a> 是 Hadoop 的表和存储管理层，为用户提供数据的关系视图。在 HCatalog 中，你可以读取和写入采用可以写入 Hive SerDe（序列化程序-反序列化程序）的任何格式的文件。

### <a name="mahout"></a>Mahout
<a target="_blank" href="https://mahout.apache.org/">Apache Mahout</a> 是在 Hadoop 上运行的可缩放机器学习算法库。计算机学习应用程序采用统计学原理，使系统学习数据并使用以往的结果来确定将来的行为。请参阅[使用 Hadoop 上的 Mahout 生成电影推荐](/documentation/articles/hdinsight-mahout/)。

### <a name="mapreduce"></a>MapReduce
MapReduce 是一个旧软件框架，用于编写并行批量处理大数据集的应用程序。MapReduce 作业将分割大型数据集并将数据组织成键值对进行处理。

[YARN](#yarn) 是 Hadoop 下一代资源管理器和应用程序框架，也称为 MapReduce 2.0。MapReduce 作业在 YARN 上运行。

有关 MapReduce 的详细信息，请参阅 Hadoop Wiki 中的 <a target="_blank" href="http://wiki.apache.org/hadoop/MapReduce">MapReduce</a>。

### <a name="oozie"></a>Oozie
<a target="_blank" href="http://oozie.apache.org/">Apache Oozie</a> 是一个管理 Hadoop 作业的工作流协调系统。它与 Hadoop 堆栈集成，支持 MapReduce、Pig、Hive 和 Sqoop 的 Hadoop 作业。它也能用于安排特定于某系统的作业，例如 Java 程序或 shell 脚本。请参阅[将 Oozie 与 Hadoop 配合使用](/documentation/articles/hdinsight-use-oozie/)。

### <a name="phoenix"></a>Phoenix
<a  target="_blank" href="http://phoenix.apache.org/">Apache Phoenix</a> 是基于 HBase 的关系数据库层。Phoenix 提供 JDBC 驱动程序以允许用户直接查询和管理 SQL 表。Phoenix 将查询和其他语句转换为本机 NoSQL API 调用（而不是使用 MapReduce），因此可以在 NoSQL 存储之上实现更快的应用程序。请参阅[将 Apache Phoenix 和 SQuirreL 与 HBase 群集配合使用](/documentation/articles/hdinsight-hbase-phoenix-squirrel/)。

### <a name="pig"></a>Pig
<a  target="_blank" href="http://pig.apache.org/">Apache Pig</a> 是一个高级平台，允许使用名为 Pig Latin 的简单脚本语言对超大型数据集执行复杂的 MapReduce 转换。Pig 转换 Pig Latin 脚本，使其可以在 Hadoop 内运行。可以创建用户定义的函数 (UDF) 来扩充 Pig Latin。请参阅[将 Pig 与 Hadoop 配合使用](/documentation/articles/hdinsight-use-pig/)。

### <a name="sqoop"></a>Sqoop
<a  target="_blank" href="http://sqoop.apache.org/">Apache Sqoop</a> 工具可尽量高效地在 Hadoop 和关系数据库（如 SQL）或其他结构化数据存储之间传输批量数据。请参阅[将 Sqoop 与 Hadoop 配合使用](/documentation/articles/hdinsight-use-sqoop/)。

### <a name="tez"></a>Tez
<a  target="_blank" href="http://tez.apache.org/">Apache Tez</a> 是构建于 Hadoop YARN 之上的应用程序框架，用于执行复杂的非循环常规图形数据处理。它是 MapReduce 框架的更灵活、功能更强大的后继，允许数据密集型进程（如 Hive）更高效地大规模运行。请参阅[“使用 Hive 和 HiveQL”中的“使用 Apache Tez 提高性能”](/documentation/articles/hdinsight-use-hive/#usetez)。

### <a name="yarn"></a>YARN
Apache YARN 是下一代 MapReduce（MapReduce 2.0 或 MRv2），它具有更大的可伸缩性和实时处理，支持超越 MapReduce 批处理的数据处理方案。YARN 提供资源管理和分布式应用程序框架。MapReduce 作业在 YARN 上运行。

若要了解 YARN，请参阅 <a target="_blank" href="http://hadoop.apache.org/docs/current/hadoop-yarn/hadoop-yarn-site/YARN.html">Apache Hadoop 下一代 MapReduce (YARN)</a>。

### <a name="zookeeper"></a>ZooKeeper
<a  target="_blank" href="http://zookeeper.apache.org/">Apache ZooKeeper</a> 通过数据寄存器的共享层次结构命名空间 (znode) 协调大型分布式系统中的进程。Znode 包含协调流程所需的少量元数据信息：状态、位置、配置，等等。

## HDInsight 上的编程语言
HDInsight 群集（Hadoop、HBase、Storm 和 Spark 群集）支持多种编程语言，但某些编程语言默认情况下未安装。对于默认情况下未安装的库、模块或程序包，请使用脚本操作来安装组件。请参阅[使用 HDInsight 进行脚本操作开发](/documentation/articles/hdinsight-hadoop-script-actions-linux/)。

### 默认编程语言支持
默认情况下，HDInsight 群集支持：

* Java
* Python

可以使用脚本操作安装其他语言：[使用 HDInsight 进行脚本操作开发](/documentation/articles/hdinsight-hadoop-script-actions-linux/)。

### Java 虚拟机 (JVM) 语言
除了 Java 外，许多语言可以使用 Java 虚拟机 (JVM) 运行；但是，运行其中某些语言可能需要在群集上安装其他组件。

HDInsight 群集支持以下基于 JVM 的语言：

* Clojure
* Jython (Python for Java)
* Scala

### Hadoop 特定的语言
HDInsight 群集支持以下特定于 Hadoop 生态系统的语言：

* 用于 Pig 作业的 Pig Latin
* 用于 Hive 作业的 HiveQL 和 SparkSQL

## <a name="advantage"></a>Hadoop 在云中的优势
作为 Azure 云生态系统的一部分，HDInsight 中的 Hadoop 具有很多优势，其中包括：

* Hadoop 群集自动设置。创建 HDInsight 群集比手动配置 Hadoop 群集容易得多。有关详细信息，请参阅[在 HDInsight 中预配 Hadoop 群集](/documentation/articles/hdinsight-hadoop-provision-linux-clusters/)。
* 最先进的 Hadoop 组件。有关详细信息，请参阅 [HDInsight 中的 Hadoop 组件、版本和服务产品][component-versioning]。
* 群集具有高可用性和可靠性。有关详细信息，请参阅 [HDInsight 中的 Hadoop 群集的可用性和可靠性](/documentation/articles/hdinsight-high-availability-linux/)。
* 使用 Azure Blob 存储或 Azure Data Lake Store（这两种都是与 Hadoop 兼容的存储选项），数据存储高效又经济。有关详细信息，请参阅[在 HDInsight 中将 Azure Blob 存储与 Hadoop 配合使用](/documentation/articles/hdinsight-hadoop-use-blob-storage/)或[将 Data Lake Store 与 HDInsight 群集配合使用](https://docs.microsoft.com/azure/data-lake-store/data-lake-store-hdinsight-hadoop-use-portal)。
* 与其他 Azure 服务集成，包括 [Web 应用](/documentation/services/app-service/web/)和 [SQL 数据库](/documentation/services/sql-databases/)。
* 用于运行 HDInsight 群集的其他 VM 大小和类型。有关详细信息，请参阅 [HDInsight 中的 Hadoop 组件、版本和服务产品][component-versioning]。
* 群集缩放。群集缩放使你能够更改正在运行的 HDInsight 群集的节点数，而无需删除或重新创建群集。
* 虚拟网络支持。HDInsight 群集可用于 Azure 虚拟网络，以支持隔离云资源或将云资源与数据中心资源相链接的混合方案。
* 启用成本低。开始[试用](/pricing/1rmb-trial/)，或查阅 [HDInsight 定价详细信息](/pricing/details/hdinsight/)。

若要详细了解 HDInsight 中的 Hadoop 的优势，请参阅 [HDInsight 的 Azure 功能页][marketing-page]。

## <a id="resources"></a>用于详细了解大数据分析、Hadoop 和 HDInsight 的资源
使用以下资源在此介绍云中 Hadoop 和大数据分析的简介的基础上构建。

### HDInsight 的 Hadoop 文档
* [HDInsight 文档](/documentation/services/hdinsight/)：Azure HDInsight 上的 Hadoop 文档页，包含文章、视频及其他资源的链接。
* [HDInsight 中的 Hadoop 入门](/documentation/articles/hdinsight-hadoop-linux-tutorial-get-started/)：有关预配 HDInsight Hadoop 群集和运行示例 Hive 查询的快速入门教程。
* [HDInsight 中的 Spark 入门](/documentation/articles/hdinsight-apache-spark-jupyter-spark-sql/)：介绍如何创建 Spark 群集和运行交互式 Spark SQL 查询的快速入门教程。
* [在 HDInsight 上使用 R Server](/documentation/articles/hdinsight-hadoop-r-server-get-started/)：开始在 HDInsight 中使用 R Server。
* [预配 HDInsight 群集](/documentation/articles/hdinsight-hadoop-provision-linux-clusters/)：了解如何通过 Azure 门户预览、Azure CLI 或 Azure PowerShell 预配 HDInsight Hadoop 群集。

### Apache Hadoop
* <a target="_blank" href="http://hadoop.apache.org/">Apache Hadoop</a>：了解有关 Apache Hadoop 软件库的详细信息，它是可用于跨计算机群集分布式处理大型数据集的框架。
* <a target="_blank" href="http://hadoop.apache.org/docs/r1.0.4/hdfs_design.html">HDFS</a>：详细了解 Hadoop 分布式文件系统的体系结构和设计，它是 Hadoop 应用程序使用的主存储系统。
* <a target="_blank" href="http://hadoop.apache.org/docs/r1.2.1/mapred_tutorial.html">MapReduce 教程</a>：了解有关编程框架的详情，此编程框架用于编写 Hadoop 应用程序，以便并行地快速处理大型计算节点群集上的大量数据。

### Microsoft 商业智能
大家熟悉的商业智能 (BI) 工具（如 Excel、PowerPivot、SQL Server Analysis Services 和 SQL Server Reporting Services）使用 Power Query 外接程序或 Microsoft Hive ODBC 驱动程序来检索、分析和报告与 HDInsight 集成的数据。

这些 BI 工具可有助于大数据分析：

* [使用 Power Query 将 Excel 连接到 Hadoop](/documentation/articles/hdinsight-connect-excel-power-query/)：了解如何使用 Microsoft Power Query for Excel，将 Excel 连接到存储 HDInsight 群集关联数据的 Azure 存储帐户。所需的 Windows 工作站。适用于 Linux 或 Windows 上的群集。
* [使用 Microsoft Hive ODBC 驱动程序将 Excel 连接到 Hadoop](/documentation/articles/hdinsight-connect-excel-hive-ODBC-driver/)：了解如何使用 Microsoft Hive ODBC 驱动程序从 HDInsight 导入数据。所需的 Windows 工作站。适用于 Linux 或 Windows 上的群集。
* [Microsoft 云平台](http://www.microsoft.com/server-cloud/solutions/business-intelligence/default.aspx)：了解有关 Power BI for Office 365、下载 SQL Server 试用版，以及设置 SharePoint Server 2013 和 SQL Server BI 的信息。
* [SQL Server Analysis Services](http://msdn.microsoft.com/zh-cn/library/hh231701.aspx)。
* [SQL Server Reporting Services](http://msdn.microsoft.com/zh-cn/library/ms159106.aspx)。

[marketing-page]: /home/features/hdinsight/
[component-versioning]: /documentation/articles/hdinsight-component-versioning/
[zookeeper]: http://zookeeper.apache.org/

<!---HONumber=Mooncake_0120_2017-->
<!--Update_Description: update from ASM to ARM-->