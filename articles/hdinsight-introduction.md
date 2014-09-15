<properties linkid="manage-services-hdinsight-introduction-hdinsight" urlDisplayName="HDInsight Introduction" pageTitle="Introduction to Azure HDInsight | Azure" metaKeywords="" description="Learn how Azure HDInsight uses Apache Hadoop clusters in the cloud, to provide a software framework to manage, analyze, and report on big data." metaCanonical="" services="hdinsight" documentationCenter="" title="Introduction to Azure HDInsight" authors="bradsev" solutions="" manager="paulettm" editor="cgronlun" />

# Azure HDInsight 简介

## 概述

Azure HDInsight 是一种服务，它以云方式部署并设置 Apache™ Hadoop® 群集，从而提供旨在对大数据进行管理、分析和报告的软件框架。

### 大数据

数据被称为“大数据”，是为了表明它的数量会随着收集过程以不断提高的速率逐渐增大，而且是针对于种类范围不断拓宽的各种非结构化格式和可变语义上下文。大型数据收集本身不会为企业创造价值。若要使大数据发挥价值，成为可付诸行动的情报或见解，不仅必须提出正确的问题，还必须收集与这些问题相关的数据，这些数据必须可供访问、清理和分析，然后以有用的方式呈现，通常这些数据会与来自各种源的数据结合使用，最终打造视角和上下文（我们现在称之为混合）。

### Apache Hadoop

Apache Hadoop 是一个辅助大数据管理和分析的软件框架。Apache Hadoop 内核提供了带 Hadoop 分布式文件系统 (HDFS) 的可靠数据存储和一个简单的 MapReduce 编程模型，该模型用于并行处理和分析存储在此分布式系统中的数据。HDFS 使用数据复制来处理在部署此高度分布式系统时出现的硬件故障问题。

### MapReduce

为了降低分析来自各种源的非结构化数据的复杂性，MapReduce 编程模型支持用于闭包映射和精简操作的核心抽象。MapReduce 编程模型会将其所有作业视为对包含密钥值对的数据集进行的计算。因此，输入和输出文件都必须包含仅包括密钥值对的数据集。这种约束的主要收获是导致 MapReduce 作业可以组合。

其他与 Hadoop 相关的项目（如 Pig 和 Hive）建立在 HDFS 和 MapReduce 框架基础之上。与直接使用 MapReduce 程序相比，此类项目管理群集会轻松得多。例如，Pig 使你能够使用称为 Pig Latin 的过程语言来编写程序，这些程序将在群集上编译为 MapReduce 程序。它还使你能够流畅地控制对数据流的管理。Hive 是一个数据仓库基础结构，它为存储在群集中的文件数据提供表抽象，然后可以使用以声明语言（称为 HiveQL）编写的类似于 SQL 的语句对数据进行查询。

### HDInsight

Azure HDInsight 使 Apache Hadoop 可在云中作为服务使用，它使 HDFS/MapReduce 软件框架和相关项目（如 Pig、Hive 和 Oozie）可用于更简单、缩放性更高且经济实用的环境。

已将第二个头节点添加到 HDInsight 所部署的 Hadoop 群集，以增加服务的可用性。Hadoop 群集的标准实现通常具有单个头节点。HDInsight 通过添加辅助头节点去除了此单点故障。切换到新 HA 群集配置不会更改群集的价格，除非客户使用超大头节点设置群集。

HDInsight 带来的主要效率之一与其管理和存储数据的方式有关。HDInsight 使用 Azure Blob 存储作为默认文件系统。Blob 存储和 HDFS 是独立的文件系统，并且已分别针对数据的存储和计算进行了优化。

-   对于将使用 HDInsight 处理的数据而言，Azure Blob 存储是一个可高度缩放、高度可用、成本低、可长期使用且可共享的存储选项。
-   HDFS 上 HDInsight 所部署的 Hadoop 群集已为对数据运行 MapReduce 计算任务进行了优化。

HDInsight 群集部署在 Azure 中的计算节点上，用于执行 MapReduce 任务，一旦任务完成，即可由用户删除。在完成计算后将数据保存在 HDFS 群集中是一种成本很高的数据存储方法。Blob 存储是一个可靠的通用 Azure 存储解决方案。因此，通过在 Blob 存储中存储数据，可以安全地删除用于计算的群集而不会丢失用户数据。不过，Blob 存储不仅仅是一个低成本的解决方案：它提供了功能完备的 HDFS 文件系统接口，该接口通过使 Hadoop 生态系统中全套组件（默认）都能直接对它管理的数据操作，为顾客提供顺畅的体验。

HDInsight 使用 Azure PowerShell 配置、运行 Hadoop 作业，并对其进行后期处理。HDInsight 还提供了一个 Sqoop 连接器，该连接器可用于将数据从 Azure SQL 数据库导入到 HDFS，或将数据从 HDFS 导出到 Azure SQL 数据库。

Microsoft Power Query for Excel 可用于将数据从 Azure HDInsight 或任何 HDFS 导入到 Excel 中。这一外接程序通过简化数据发现和对各种数据源的访问，增强了 Excel 中的自助式 BI 体验。除了 Power Query 之外，还可使用 Microsoft Hive ODBC 驱动程序集成 Excel、SQL Server Analysis Services 和 Reporting Services 等商业智能 (BI) 工具，推动和简化端到端数据分析。

### 概要

本主题描述了 HDInsight 所支持的 Hadoop 生态系统、HDInsight 的主要使用方案以及其他资源的指南。本主题包含以下部分：

-   [HDInsight 上的 Hadoop 生态系统][]：HDInsight 提供了对 Pig、Hive 和 Sqoop 的实现，并通过 Power Query 或 Microsoft Hive ODBC 驱动程序支持与 Blob 存储/HDFS 和 MapReduce 框架集成的其他 BI 工具（如 Excel、SQL Server Analysis Services 和 Reporting Services）。此部分描述 Hadoop 生态系统中的这些程序将用于处理哪些作业。

-   [适用于 HDInsight 的大型数据方案][]：此部分解答了以下问题：HDInsight 对于哪些类型的作业而言是一种适用的技术？

-   [HDInsight 的资源][]：此部分指明可从何处查找相关资源以了解其他信息。

## Azure 上的 Hadoop 生态系统

### 介绍

HDInsight 提供一个实现 Microsoft 的基于云的解决方案以处理大型数据的框架。此联合生态系统管理和分析大型数据的数量，并使用 MapReduce 编程模型的并行处理功能。本节详细列举并简述了可与 HDInsight 配合使用的与 Apache 兼容的 Hadoop 技术。

HDInsight 提供了对 Hive 和 Pig 的实现以集成数据处理和仓库功能。Microsoft 的大数据解决方案与 Microsoft 的 BI 工具（如 SQL Server Analysis Services、Reporting Services、PowerPivot 和 Excel）集成。这使你能够在 Blob 存储中对由 HDInsight 存储和管理的数据执行直接 BI 处理。

也可下载作为 Hadoop 生态系统的一部分并已构建为基于 Hadoop 群集运行的其他与 Apache 兼容的技术和相关技术，并将其与 HDInsight 配合使用。这包括开放源代码技术（如 Sqoop），这些技术将 HDFS 与关系数据存储相集成。

### Pig

Pig 是一个高级别平台，用于在 Hadoop 群集上处理大数据。Pig 包含一种称作 Pig Latin 的数据流语言（该语言支持对大型数据集编写查询）和一个从控制台运行程序的执行环境。Pig Latin 程序包含在后台转换为 MapReduce 程序系列的数据集转换系列。Pig Latin 抽象提供了比 MapReduce 更丰富的数据结构，并为 Hadoop 执行 SQL 对关系数据库管理系统 (RDBMS) 执行的操作。Pig Latin 可完全扩展。在整理分析时，可调用用 Java、Python、Ruby、C\# 或 JavaScript 编写的用户定义的函数 (UDF) 来自定义每个处理路径阶段。有关其他信息，请参阅[欢迎使用 Apache Pig！（可能为英文页面）][]

### Hive

Hive 是一个分布式数据仓库，管理 HDFS 中存储的数据。它是 Hadoop 查询引擎。Hive 提供了类似 SQL 的接口和相关数据模型，使分析人员能够使用强大的 SQL 技能。Hive 使用一种称作 HiveQL 的语言（一种 SQL 行话）。Hive（类似于 Pig）是一种基于 MapReduce 的抽象，它在运行时会将查询转换为一系列 MapReduce 作业。适用于 Hive 的方案在概念上与适用于 RDBMS 的方案更相近，因此适用于结构化程度更高的数据。对于非结构化数据，Pig 是更佳的选择。有关其他信息，请参阅[欢迎使用 Apache Hive！（可能为英文页面）][]

### Sqoop

Sqoop 是一种用于在 Hadoop 和关系数据库（如 SQL）或其他结构化数据存储之间尽可能高效地传输批量数据的工具。使用 Sqoop 可将数据从外部结构化数据存储导入到 HDFS 或相关的系统（如 Hive）。Sqoop 也可从 Hadoop 中提取数据并将提取到的数据导出到外部关系数据库、企业数据仓库或任何其他结构化数据存储类型。有关其他信息，请参阅 [Apache Sqoop][] 网站。

### Microsoft Avro Library

Microsoft Avro Library 针对 Microsoft.NET 环境实现了 Apache Avro 数据序列化系统。Apache Avro 为序列化提供了一种紧凑的二进制数据交换格式。它使用 [JSON][] 定义与语言无关的架构，以支持语言互操作性。以一种语言序列化的数据可以用另一种语言读取。目前支持 C、C++、C\#、Java、PHP、Python 和 Ruby。有关格式的详细信息可以在 [Apache Avro 规范][]中找到。请注意，Microsoft Avro Library 的当前版本不支持此规范的远程过程调用 (RPC) 部分。

Apache Avro 序列化格式广泛应用于 Azure HDInsight 及其他 Apache Hadoop 环境中。Avro 提供了简便的方法来表示 Hadoop MapReduce 作业内的复杂数据结构。Avro 文件格式已设计为支持分布式 MapReduce 编程模型。实现分布的关键是文件必须是“可拆分的”，也就是说，用户可以在文件中随机设置一个点，然后即可从某一特定块开始读取。有关其他信息，请参阅[使用 Microsoft Avro Library 序列化数据][]。

### 商业智能工具和连接器

大家熟悉的商业智能 (BI) 工具（如 Excel、PowerPivot、SQL Server Analysis Services 和 Reporting Services）使用 Power Query 外接程序或 Microsoft Hive ODBC 驱动程序来检索、分析和报告与 HDInsight 集成的数据。

-   从 [Microsoft 下载中心][]可下载 Microsoft Power Query for Excel。

-   从这个[下载网站][]可下载 Microsoft Hive ODBC 驱动程序。

-   有关 Analysis Services 的信息，请参阅 [SQL Server 2012 Analysis Services][]。

-   有关 Reporting Services 的信息，请参阅 [SQL Server 2012 Reporting][]。

## 适用于 HDInsight 的大型数据方案

提供 HDInsight 使用案例的一种典型情形是：以批处理的方式，对 Azure 节点上存储的整个非结构化数据集（不需要频繁更新）进行即席分析。

这些情况适用于商业、科学和监管方面的各种活动。例如，这可能包括监控零售业的供应链、金融业的可疑交易模式、公用事业和服务的需求模式、一系列环境传感器中的空气和水质量或大城市区域内的犯罪模式。

HDInsight（通常还有 Hadoop 技术）最适合于处理大量已记录或存档的数据，这些数据在写入后不需要进行频繁更新，并且通常会读取这些数据来进行完整分析。此方案对更适合由 RDBMS 处理的数据加以补充，RDBMS 要求数据容量较小（GB 而非 PB），并且必须持续更新它，或查询它以获得完整数据集内的特定数据点。RDBMS 最适用于根据固定架构组织和存储的结构化数据。MapReduce 可以很好地处理没有预定义架构的非结构化数据，因为它能够在那些数据被处理时对它们进行转译。

## 后续步骤：HDInsight 的资源

**Microsoft：HDInsight**

-   [HDInsight 文档][]：Azure HDInsight 的文档页，带有文章、视频及其他资源的链接。

-   [Azure HDInsight 入门][]：提供 HDInsight 快速入门知识的教程。

-   [运行 HDInsight 示例][]：有关如何运行 HDInsight 随附示例的教程。

-   [大数据和 Azure][]：探索你可以使用 Azure 生成什么内容的大数据方案。

-   [Azure HDInsight SDK][]：HDinsight SDK 的参考文档。

**Microsoft：Windows 和 SQL Database**

-   [Azure 主页][]：开始构建应用程序所需的方案、免费试用版注册、开发工具和文档。

-   [Azure SQL Database][]：有关 SQL Database 的 MSDN 文档。

-   [SQL Database 的管理门户][]：一种轻量版易用型数据库管理工具，用于以云方式管理 SQL Database。

-   [Adventure Works for SQL Database][]：SQL Database 示例数据库的下载页。

**Microsoft：商业智能**

-   [利用 Power Query 将 Excel 连接到 HDInsight][]：了解如何使用 Microsoft Power Query for Excel，将 Excel 连接到存储 HDInsight 群集关联数据的 Azure 存储帐户。

-   [使用 Microsoft Hive ODBC 驱动程序将 Excel 连接到 HDInsight][]：了解如何使用 Microsoft Hive ODBC 驱动程序从 Azure HDInsight 导入数据。

-   [Microsoft BI PowerPivot][]：下载并获取有关功能强大的数据混合 Web 应用程序和探索工具的信息。

-   [SQL Server 2012 Analysis Services][]：下载 SQL Server 2012 评估版，了解如何生成可带来能够付诸实施的见解的企业级综合分析解决方案。

-   [SQL Server 2012 Reporting][]：下载 SQL Server 2012 评估版，了解如何创建支持企业间实时决策的可高度伸缩的全面解决方案。

**Apache Hadoop**：

-   [Apache Hadoop][]：了解有关 Apache Hadoop 软件库的详细信息，这是一个框架，允许你跨计算机群集分布式处理大型数据集。

-   [HDFS][]：了解有关 Hadoop 分布式文件系统 (HDFS) 的体系结构和设计，Hadoop 分布式文件系统 (HDFS) 是由 Hadoop 应用程序使用的主存储系统。

-   [MapReduce][]：了解有关编程框架的详情，此编程框架用于编写 Hadoop 应用程序，以便以并行方式快速处理大型计算节点群集上的大量数据。

  [HDInsight 上的 Hadoop 生态系统]: #Ecosystem
  [适用于 HDInsight 的大型数据方案]: #Scenarios
  [HDInsight 的资源]: #Resources
  [欢迎使用 Apache Pig！（可能为英文页面）]: http://pig.apache.org/
  [欢迎使用 Apache Hive！（可能为英文页面）]: http://hive.apache.org/
  [Apache Sqoop]: http://sqoop.apache.org/
  [JSON]: http://www.json.org
  [Apache Avro 规范]: http://avro.apache.org/docs/current/spec.html
  [使用 Microsoft Avro Library 序列化数据]: http://azure.microsoft.com/zh-cn/documentation/articles/hdinsight-dotnet-avro-serialization/
  [Microsoft 下载中心]: http://go.microsoft.com/fwlink/?LinkID=286689
  [下载网站]: http://go.microsoft.com/fwlink/?LinkID=286698
  [SQL Server 2012 Analysis Services]: http://www.microsoft.com/zh-cn/server-cloud/solutions/business-intelligence/analysis.aspx#fbid=9ZH5wGSDgf0
  [SQL Server 2012 Reporting]: http://www.microsoft.com/zh-cn/server-cloud/solutions/business-intelligence/dashboards-reports.aspx#fbid=9ZH5wGSDgf0
  [HDInsight 文档]: http://azure.microsoft.com/zh-cn/documentation/services/hdinsight/
  [Azure HDInsight 入门]: /en-us/manage/services/hdinsight/get-started-hdinsight/
  [运行 HDInsight 示例]: /en-us/manage/services/hdinsight/howto-run-samples/
  [大数据和 Azure]: http://azure.microsoft.com/zh-cn/solutions/big-data/
  [Azure HDInsight SDK]: http://msdn.microsoft.com/zh-cn/library/dn469975.aspx
  [Azure 主页]: https://www.windowsazure.cn
  [Azure SQL Database]: http://msdn.microsoft.com/zh-cn/library/azure/ee336279.aspx
  [SQL Database 的管理门户]: http://msdn.microsoft.com/zh-cn/library/azure/gg442309.aspx
  [Adventure Works for SQL Database]: http://msftdbprodsamples.codeplex.com/releases/view/37304
  [利用 Power Query 将 Excel 连接到 HDInsight]: /en-us/manage/services/hdinsight/connect-excel-with-power-query/
  [使用 Microsoft Hive ODBC 驱动程序将 Excel 连接到 HDInsight]: /en-us/manage/services/hdinsight/connect-excel-with-hive-ODBC/
  [Microsoft BI PowerPivot]: http://office.microsoft.com/zh-cn/excel/HA101959985.aspx
  [Apache Hadoop]: http://hadoop.apache.org/
  [HDFS]: http://hadoop.apache.org/docs/r0.18.1/hdfs_design.html
  [MapReduce]: http://mapreduce.org/
