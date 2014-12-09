<properties urlDisplayName="HDInsight Introduction" pageTitle="HDInsight 中的 Hadoop 简介：云中的大数据分析 | Azure" metaKeywords="" description="Learn how Azure HDInsight uses Apache Hadoop clusters in the cloud to provide a software framework to manage, analyze, and report on big data." metaCanonical="" services="hdinsight" documentationCenter="" title="Introduction to Hadoop in HDInsight: Big data processing and analysis in the cloud" authors="cgronlun" solutions="" manager="paulettm" editor="cgronlun" />

<tags ms.service="hdinsight" ms.workload="big-data" ms.tgt_pltfrm="na" ms.devlang="na" ms.topic="article" ms.date="10/23/2014" ms.author="cgronlun" />



# HDInsight 中的 Hadoop 简介：云中的大数据处理和分析

大致了解 HDInsight 组件、常见术语和方案，并查看适用于使用 HDInsight 中的 Hadoop 的资源。

Azure HDInsight 以云方式部署并设置 Apache Hadoop 群集，从而提供旨在对大数据进行管理、分析和报告的软件框架。Hadoop 内核提供了带 Hadoop 分布式文件系统 (HDFS) 的可靠数据存储和一个简单的 MapReduce 编程模型，该模型用于并行处理和分析存储在此分布式系统中的数据。 

### 什么是大数据？
大数据指的是具有以下特性的数据：收集的数量不断增长、收集速度日益加快、采用的非结构化格式的类型不断扩展且语义上下文灵活多变。 

大数据描述各种大型数字信息，从 Twitter 源中的文本到工业设备的传感器信息再到客户浏览在线产品目录并购买的信息。大数据可以是历史数据（即已存储的数据），也可以是实时数据（即从数据源直接流式传输的数据）。 

若要使大数据成为可付诸行动的情报或见解，不仅必须提出正确的问题，还必须收集与这些问题相关的数据，这些数据必须可供访问、清理和分析，然后以有用的方式呈现。这正是 HDInsight 中的 Hadoop 用武之地。


## 本文内容

本文将概述 HDInsight 上的 Hadoop，包括：

* **[HDInsight 上的 Hadoop 生态系统概述：](#overview)**  HDInsight 是 Azure 的 Hadoop 解决方案，并提供对 Storm、HBase、Pig、Hive、Sqoop、Oozie、Ambari 等的实现。HDInsight 还可集成商业智能 (BI) 工具，例如 Excel、SQL Server Analysis Services 和 SQL Server Reporting Services。
* **[Hadoop 在云中的优势：](#advantage)** 您应考虑 HDInsight 在云中实现 Hadoop 的原因。
* **[HDInsight 大数据分析解决方案：](#solutions)** 可以使用 HDInsight 为您的组织解决问题的一些实用方法，从分析 Twitter 观点数据到分析 HVAC 系统效率。
* **[用于详细了解大数据分析、Hadoop 和 HDInsight 的资源：](#resources)** 指向其他内容的链接。


## <a name="overview"></a>HDInsight 上的 Hadoop 生态系统概述

Hadoop 是一种迅速扩展的技术堆栈，是大数据分析的首选解决方案。HDInsight 是 Microsoft Azure 在云中实现 Hadoop 的框架。
 
HDInsight 中的 Hadoop 群集使用 Hortonworks 数据平台 (HDP) 分发版本和该分发内包含的 Hadoop 组件套件。有关组件及其在 HDInsight 群集中的版本的信息，请参阅 [HDInsight 提供的群集版本有哪些新功能？][组件版本]

Apache Hadoop 是一个用于大数据管理和分析的软件框架。Apache Hadoop 内核提供了带 Hadoop 分布式文件系统 (HDFS) 的可靠数据存储和一个简单的 MapReduce 编程模型，该模型用于并行处理和分析存储在此分布式系统中的数据。HDFS 使用数据复制来处理在部署此高度分布式系统时出现的硬件故障问题。

以下是 HDInsight 中的 Hadoop 技术：

* **[Ambari](#ambari)：**群集设置、管理和监视
* **[Avro](#avro)** (Microsoft .NET Library for Avro)：Microsoft .NET 环境的数据序列化
* **[HBase](#hbase)：**超大型表格的非关系型数据库
* **[HDFS](#hdfs)：** Hadoop 分布式文件系统
* **[Hive](#hive)：**类似 SQL 的查询
* **[Mahout](#mahout)：**计算机学习
* **[MapReduce 和 YARN](#mapreduce):**分布式处理和资源管理
* **[Oozie](#oozie)：**工作流管理
* **[Pig](#pig)：**更简单的 MapReduce 转换脚本
* **[Sqoop](#sqoop)：**数据导入和导出
* **[Storm](#storm)：**快速、大型数据流的实时处理
* **[Zookeeper](#zookeeper)：**协调分布式系统中的流程
 
此外，请了解可以与 HDInsight 组合使用的**[商业智能工具](#bi)**。

### <a name="ambari"></a>Ambari

Apache Ambari 用于设置、管理和监视 Apache Hadoop 群集。它包括一系列直观的操作员工具和一组隐藏 Hadoop 复杂性的可靠 API，可简化群集操作。请参阅[使用 Ambari API 监视 HDInsight 中的 Hadoop 群集](../hdinsight-monitor-use-ambari-api/)和 <a target="_blank" href="https://github.com/apache/ambari/blob/trunk/ambari-server/docs/api/v1/index.md">Apache Ambari API 参考</a>。

### <a name="avro"></a>Avro (Microsoft .NET Library for Avro)

Microsoft .NET Library for Avro 针对 Microsoft.NET 环境序列化实现了 Apache Avro 紧凑的二进制数据交换格式。它使用 <a target="_blank" href="http://www.json.org/">JSON</a> 定义与语言无关的架构，以支持语言互操作性，这意味着以一种语言序列化的数据可以用另一种语言读取。有关格式的详细信息可以在 <a target=_"blank" href="http://avro.apache.org/docs/current/spec.html">Apache Avro 规范</a>中找到。 
Avro 文件格式支持分布式 MapReduce 编程模型。文件是"可拆分的"，也就是说，您可以在文件中任意设置一个点，然后即可从某一特定块开始读取。若要了解相关方法，请参阅[使用 Microsoft .NET Library for Avro 将数据序列化](../hdinsight-dotnet-avro-serialization/)。

### <a name="hbase"></a>HBase

<a target="_blank" href="http://hbase.apache.org/">Apache HBase</a> 是构建于 Hadoop 上的一个非关系数据库，专为海量非结构化和半结构化数据而设计，这类数据可能是几十亿行乘几百万列。HDInsight 上的 HBase 群集已配置为将数据直接存储在 Azure Blob 存储中，既减少了延迟又提高了灵活性。请参阅 [HDInsight 上的 HBase 概述](../hdinsight-hbase-overview/)。

### <a name="hdfs"></a>HDFS

Hadoop 分布式文件系统 (HDFS) 是一种分布式文件系统，采用 MapReduce 和 YARN，是 Hadoop 生态系统的核心。HDFS 是 HDInsight 上 Hadoop 群集的标准文件系统。

### <a name="hive"></a>Hive

<a target="_blank" href="http://hive.apache.org/">Apache Hive</a> 是构建于 Hadoop 上的一个数据仓库软件，允许使用类似于 SQL 的语言调用 HiveQL 来查询和管理分布式存储中的大型数据集。Hive（类似于 Pig）是一种基于 MapReduce 的抽象，它在运行时会将查询转换为一系列 MapReduce 作业。Hive 比 Pig 在概念上更接近于关系数据库管理系统，因此适用于结构化程度更高的数据。对于非结构化数据，Pig 是更佳的选择。请参阅[将 Hive 与 HDInsight 中的 Hadoop 配合使用](../hdinsight-use-hive/)

### <a name="mahout"></a>Mahout

<a target="_blank" href="https://mahout.apache.org/">Apache Mahout</a> 是在 Hadoop 上运行的一种可扩展的计算机学习算法库。计算机学习应用程序采用统计学原理，使系统学习数据并使用以往的结果来确定将来的行为。请参阅[使用 Hadoop 上的 Mahout 生成电影推荐](../hdinsight-mahout/)。

### <a name="mapreduce"></a>MapReduce 和 YARN
Hadoop MapReduce 是一个用于编写并行处理大数据集的应用程序的软件框架。MapReduce 作业将分割大型数据集并将数据组织成键值对进行处理。 

Apache YARN 是下一代 MapReduce（MapReduce 2.0 或 MRv2），用于将 JobTracker 的两个主要任务（资源管理和作业计划/监视）分割成单独的实体。

有关 MapReduce 的更多信息，请参阅 Hadoop Wiki 中的 <a target="_blank" href="http://wiki.apache.org/hadoop/MapReduce">MapReduce</a>。若要了解 YARN，请参阅 <a target="_blank" href="http://hadoop.apache.org/docs/current/hadoop-yarn/hadoop-yarn-site/YARN.html">Apache Hadoop 下一代 MapReduce (YARN)</a>。

### <a name="oozie"></a>Oozie
<a target="_blank" href="http://oozie.apache.org/">Apache Oozie</a> 是一个管理 Hadoop 作业的工作流协调系统。它与 Hadoop 堆栈集成，支持 MapReduce、Pig、Hive 和 Sqoop 的 Hadoop 作业。它也能用于安排特定于某系统的作业，例如 Java 程序或 shell 脚本。请参阅[将基于时间的 Oozie 协调器与 HDInsight 中的 Hadoop 配合使用](../hdinsight-use-oozie-coordinator-time/)。

### <a name="pig"></a>Pig

<a  target="_blank" href="http://pig.apache.org/">Apache Pig</a> 是一个高级平台，可以使用一种简单脚本语言 Pig Latin 对超大型数据集执行复杂的 MapReduce 转换。Pig 转换 Pig Latin 脚本，使其可以在 Hadoop 内运行。可以创建用户定义的函数 (UDF) 来扩展 Pig Latin。请参阅[将 Pig 与 Hadoop 配合使用来分析 Apache 日志文件](../hdinsight-use-pig/)。

### <a name="sqoop"></a>Sqoop
<a  target="_blank" href="http://sqoop.apache.org/">Apache Sqoop</a>是一种用于在 Hadoop 和关系数据库（如 SQL）或其他结构化数据存储之间尽可能高效地传输批量数据的工具。请参阅[将 Sqoop 与 Hadoop 配合使用](../hdinsight-use-sqoop/)。

### <a name="storm"></a>Storm
<a  target="_blank" href="https://storm.incubator.apache.org/">Apache Storm</a> 是一个分布式实时计算系统，用于快速处理大型数据流。Storm 以 HDInsight 中的托管群集形式提供。请参阅[使用 Storm 和 Hadoop 分析实时传感器数据](../hdinsight-storm-sensor-data-analysis/)。

### <a name="zookeeper"></a>Zookeeper
<a  target="_blank" href="http://zookeeper.apache.org/">Apache Zookeeper</a> 通过数据寄存器的共享层次结构命名空间 (znode) 协调大型分布式系统中的流程。Znode 包含协调流程所需的少量元数据信息：状态、位置、配置等等。 

### <a name="bi"></a>商业智能工具 
大家熟悉的商业智能 (BI) 工具（如 Excel、PowerPivot、SQL Server Analysis Services 和 Reporting Services）使用 Power Query 外接程序或 Microsoft Hive ODBC 驱动程序来检索、分析和报告与 HDInsight 集成的数据。 

这些 BI 工具可有助于大数据分析：

* <a target="_blank" href="http://www.microsoft.com/zh-cn/download/details.aspx?id=39379">下载 Microsoft Power Query for Excel</a> 
* <a target="_blank" href="http://www.microsoft.com/zh-cn/download/details.aspx?id=40886">下载 Microsoft Hive ODBC Driver</a>
* <a target="_blank" href="http://www.microsoft.com/zh-cn/server-cloud/solutions/business-intelligence/analysis.aspx">了解有关 SQL Server Analysis Services 的更多信息</a>
* <a target="_blank" href="http://msdn.microsoft.com/zh-cn/library/ms159106.aspx">了解 SQL Server Reporting Services</a>

## <a name="advantage"></a>Hadoop 在云中的优势

作为 Azure 云生态系统的一部分，HDInsight 中的 Hadoop 具有很多优势，其中包括：

* 最先进的 Hadoop 组件。有关详细信息，请参阅 [HDInsight 提供的 Hadoop 群集版本有哪些新功能？][组件版本]
* 群集具有高可用性和可靠性。有关详细信息，请参阅[ HDInsight 中的 Hadoop 群集的可用性和可靠性](../hdinsight-high-availability/)。
* 使用 Azure Blob 存储（一种与 Hadoop 兼容的选项），数据存储高效又经济。有关详细信息，请参阅[将 Azure Blob 存储与 HDInsight 中的 Hadoop 配合使用](../hdinsight-use-blob-storage/)。
* 与其他 Azure 服务集成，包括[网站](/zh-cn/manage/services/websites/) 和 [SQL Database](/zh-cn/manage/services/sql-database/)。
* 进入成本低。开始[免费试用](/zh-cn/pricing/free-trial/) 或咨询 [HDInsight 定价详细信息](../pricing/details/hdinsight/)。

若要详细了解 HDInsight 中的 Hadoop 的优势，请参阅 [HDInsight 的 Azure 功能页面][营销页面]。

## <a name="solutions"></a>试用 HDInsight 大数据分析解决方案

分析组织的数据，获得业务的深度见解。下面是一些示例： 

* [分析 HVAC 传感器数据](../hdinsight-hive-analyze-sensor-data/)：了解如何通过将 Hive 与 HDInsight (Hadoop) 配合使用来分析传感器数据，然后在 Microsoft Excel 中实现数据的可视化。在此例中，您将使用 Hive 处理 HVAC 系统生成的历史数据，从而了解哪些系统无法可靠地维持设定的温度。
* [将 Hive 与 HDInsight 配合使用来分析网站日志](../hdinsight-hive-analyze-website-log/)：了解如何使用 HDInsight 中的 HiveQL 来分析网站日志，从而了解来自外部网站的一天访问次数以及用户遇到的网站错误汇总。
* [使用 HDInsight (Hadoop) 中的 Storm 和 HBase 分析实时传感器数据](../hdinsight-storm-sensor-data-analysis/)：了解如何构建一个解决方案，使用 HDInsight 中的 Storm 群集处理 Azure 事件中心的传感器数据，然后将处理后的传感器数据以近乎实时的信息显示在基于 Web 的仪表板上。

若要试用 HDInsight 上的 Hadoop，请参阅 [HDInsight 文档页](/zh-cn/manage/services/hdinsight/) 上"探究"部分的"入门"文章。若要尝试更高级的示例，请向下滚动到"分析"部分。



## <a name="resources"></a>用于详细了解大数据分析、Hadoop 和 HDInsight 的资源 

### HDInsight 中的 Hadoop	

* [HDInsight 文档](/zh-cn/manage/services/hdinsight/)：Azure HDInsight 的文档页，带有文章、视频及其他资源的链接。

* [HDInsight 中的 Hadoop 的学习计划图](../hdinsight-learn-map)：HDInsight 的 Hadoop 文档导航。

* [Azure HDInsight 入门](../hdinsight-get-started/)：关于使用 HDInsight 中的 Hadoop 的快速入门教程。

* [运行 HDInsight 示例](../hdinsight-run-samples/)：有关如何运行随 HDInsight 提供的示例的教程。
	
* [Azure HDInsight SDK](http://msdn.microsoft.com/zh-cn/library/dn479185.aspx)：HDinsight SDK 的参考文档。


### Azure 上的 SQL Database	
		
* [Azure SQL Database](http://msdn.microsoft.com/zh-cn/library/windowsazure/ee336279.aspx)：有关 SQL Database 的 MSDN 文档。
	
* [SQL Database 的管理门户](http://msdn.microsoft.com/zh-cn/library/windowsazure/gg442309.aspx)：一个易用的轻型数据库管理工具，用于在云中管理 SQL Database。

* [Adventure Works for SQL Database](http://msftdbprodsamples.codeplex.com/releases/view/37304)：SQL Database 示例数据库的下载页。	

### Microsoft 商业智能		

* [利用 Power Query 将 Excel 连接到 Hadoop](../hdinsight-connect-excel-power-query/)：了解如何使用 Microsoft Power Query for Excel，将 Excel 连接到存储 HDInsight 群集关联数据的 Azure 存储帐户。 

* [使用 Microsoft Hive ODBC 驱动程序将 Excel 连接到 Hadoop](../hdinsight-connect-excel-hive-ODBC-driver/)：了解如何使用 Microsoft Hive ODBC 驱动程序从 HDInsight 导入数据。

* [Microsoft 商业智能 (BI)](http://www.microsoft.com/zh-cn/server-cloud/solutions/business-intelligence/default.aspx)：了解适用于 Office 365 的 Power BI，下载 SQL Server 试用版，以及设置 SharePoint Server 2013 和 SQL Server BI。
			

### Apache Hadoop			

* <a target="_blank" href="http://hadoop.apache.org/">Apache Hadoop</a>：了解有关 Apache Hadoop 软件库的详细信息，这是一个框架，允许你跨计算机群集分布式处理大型数据集。	

*  <a target="_blank" href="http://hadoop.apache.org/docs/r0.18.1/hdfs_design.html">HDFS</a>：了解有关 Hadoop 分布式文件系统 (HDFS) 的体系结构和设计，Hadoop 分布式文件系统 (HDFS) 是由 Hadoop 应用程序使用的主存储系统。		

* <a target="_blank" href="http://mapreduce.org/">MapReduce</a>：了解有关编程框架的详情，此编程框架用于编写 Hadoop 应用程序，以便以并行方式快速处理大型计算节点群集上的大量数据。	


[marketing-page]: /zh-cn/manage/services/hdinsight/
[component-versioning]: ../hdinsight-component-versioning/

[zookeeper]: http://zookeeper.apache.org/ 
