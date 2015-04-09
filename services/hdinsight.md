<properties linkid="dev-net-HDInsight" urlDisplayName="Windows Azure HDInsight" pageTitle="HDInsight - Azure 微软云" metaKeywords="HDInsight,大数据,Big Data,Hadoop,Storm,HBase,学习路线图,Apache,群集, HDInsight Emulator,MapReduce,Maven,Pig,Python,Hive,Sqoop,Mahout,Power Query,Ambari API" description="了解如何使用云中的 Hadoop 从大数据挖掘有用信息。此处提供的文档和视频帮助你开始通过 HDInsight 在云中使用 Hadoop。按照教程可在几分钟内生成和运行 Hadoop 群集、处理大数据以及使用 Excel 分析结果。" metaCanonical="" services="HDInsight" documentationCenter="Services" title="Learn how to get insights from big data using Hadoop in the cloud" authors="" solutions="" manager="" editor="Haifeng Liu" />


#HDInsight (Hadoop) 文档

####了解如何使用云中的 Hadoop 从大数据挖掘有用信息

此处提供的文档帮助你开始通过 HDInsight 在云中使用 Hadoop。按照教程可在几分钟内生成和运行 Hadoop 群集、处理大数据以及使用 Excel 分析结果。

####快速链接

-   [服务概述](/home/features/hdinsight/)
-   [可交付的解决方案](/solutions/data-management/)
-   [定价详细信息](/pricing/details/hdinsight/)

####特色

-   [从这里开始：遵循 HDInsight 学习路线图](/zh-cn/documentation/articles/hdinsight-learn-map/)
-   [Storm 上的单词计数拓扑入门](/zh-cn/documentation/articles/hdinsight-storm-getting-started/)

##教程和指南

###探究	

####[HDInsight 提供的 Hadoop 群集版本有哪些新增功能？](/zh-cn/documentation/articles/hdinsight-component-versioning/)

了解 HDInsight 中包含哪些 Hadoop 组件和版本。

####[HDInsight 学习路线图](/zh-cn/documentation/articles/hdinsight-learn-map/)

此页提供有关 Azure HDInsight 学习资源的快速概览。请在示意图的指导下选择最有效的学习路线。

####[HDInsight 中的 Hadoop 简介](/zh-cn/documentation/articles/hdinsight-hadoop-introduction/)

大致了解 HDInsight 组件、常见术语和方案。HDInsight 在云中部署和设置 Apache Hadoop 集群，同时提供设计用来管理、分析和报告大数据的软件框架。

####[开始使用 HDInsight 中的 Hadoop](/zh-cn/documentation/articles/hdinsight-get-started/)

了解如何设置集群、用 Hive 查询数据并输出到 Excel 进行分析。

####[HDInsight 中的 HBase 概述](/zh-cn/documentation/articles/hdinsight-hbase-overview/)

了解 HBase，这是基于 Hadoop 构建的 NoSQL 数据库，并设计用于大量数据。HDInsight 上的 HBase 群集配置为直接在 Azure Blob 存储中存储数据，在性能对成本方面可以降低延迟并增加弹性。

####[在 HDInsight 中将 HBase 与 Hadoop 配合使用入门](/zh-cn/documentation/articles/hdinsight-hbase-get-started/)

Apache HBase 是开源分布式大规模数据存储，可实现随机读写的低延迟。在本教程中，你将了解如何利用 HDInsight 创建和查询 HBase 表。

####[HDInsight 中的 Storm 概述](/zh-cn/documentation/articles/hdinsight-storm-overview/)

HDInsight 中的 Storm 让您能实时处理流数据。作为一个集成到 Azure 生态系统中的托管集群，Storm 可以配置为与其他 Azure 服务一起工作，成为一个完整的实时数据处理和分析解决方案。

####[在 HDInsight 的 Storm 上的单词计数拓扑入门](/zh-cn/documentation/articles/hdinsight-storm-getting-started/)

了解如何在 HDInsight 中的 Storm 上设置并运行一个单词计数拓扑。本教程将指导您设置一个 Storm 集群，然后运行、监视并停止 Storm 拓扑。

####[HDInsight Emulator 入门](/zh-cn/documentation/articles/hdinsight-get-started-emulator/)

了解如何使用 Microsoft HDInsight Emulator for Azure，它提供本地开发环境。

###开发

####[Azure HDInsight 发行说明](/zh-cn/documentation/articles/hdinsight-release-notes/)

了解每个 HDInsight 版本中的最新改进和升级。了解你可能需要对你的 HDInsight 配置和作业等进行的更改。

####[运行 HDInsight 中的 Hadoop 示例](/zh-cn/documentation/articles/hdinsight-run-samples/)

包含的四个示例旨在让你快速开始使用并为你提供一个用于演练概念的可扩展测试平台。创建数据集，并观察数据大小对作业产生的影响。

####[开发适用于 HDInsight 中的 Hadoop 的 Java MapReduce 程序](/zh-cn/documentation/articles/hdinsight-develop-deploy-java-mapreduce/)

通过本端到端方案了解如何在 HDInsight Emulator 上开发和测试单词计数 MapReduce 作业，然后在 HDInsight 上部署和运行该作业。

####[开发适用于 HDInsight 的 C# Hadoop Streaming 计划](/zh-cn/documentation/articles/hdinsight-hadoop-develop-deploy-streaming-jobs/)

了解如何在 HDInsight Emulator 上开发和测试 Hadoop Streaming MapReduce 程序，并随后使用 PowerShell 脚本在 HDInsight 上运行该程序。

####[使用 Maven 构建使用带 Hadoop HBase 的 Java 应用程序](/zh-cn/documentation/articles/hdinsight-hbase-build-java-maven/)

了解如何使用 Apache Maven 在 Java 中构建 Apache HBase 应用程序。然后，通过 HDInsight 使用该应用程序 (Hadoop)。

####[提交 HDInsight 中的 Hadoop 作业](/zh-cn/documentation/articles/hdinsight-submit-hadoop-jobs-programmatically/)

了解如何使用 PowerShell 和 HDInsight .NET SDK 提交 MapReduce 和 Hive 作业。

####[使用 HDInsight 中的 Hadoop MapReduce](/zh-cn/documentation/articles/hdinsight-use-mapreduce/)

了解如何使用你的工作站中的 Azure PowerShell 来将计算文本中单词出现次数的 MapReduce 程序提交到 HDInsight 群集。

####[在 HDInsight 中将 Hive 与 Hadoop 配合使用](/zh-cn/documentation/articles/hdinsight-use-hive/)

使用 HiveQL 查询 Apache log4j 日志文件中的数据，并报告基本统计信息。

####[在 HDInsight 中将 Pig 与 Hadoop 配合使用](/zh-cn/documentation/articles/hdinsight-use-pig/)

编写 Pig Latin 语句以分析 Apache log4j 日志文件，并对数据运行各种查询以生成输出。

####[在 HDInsight 中将 Python 与 Hive 和 Pig 配合使用](/zh-cn/documentation/articles/hdinsight-python/)

Hive 和 Pig 都可让你使用多种编程语言来创建用户自定义函数 (UDF)。了解如何使用 Hive 和 Pig 创建 Python UDF。

####[使用 Sqoop 在 Hadoop 群集与数据库之间移动数据](/zh-cn/documentation/articles/hdinsight-use-sqoop/)

了解如何使用工作站中的 Azure PowerShell 在 HDInsight 群集与 Azure SQL 数据库之间运行 Sqoop 导入和导出。

####[在 HDInsight 中将 Oozie 与 Hadoop 配合使用](/zh-cn/documentation/articles/hdinsight-use-oozie/)

了解如何运行 Apache Oozie 工作流来处理 log4j 日志文件，以统计每个日志级别类型的出现次数。然后，将结果导出到 Azure SQL 数据库表。

####[使用 Microsoft .NET Library for Avro 序列化数据](/zh-cn/documentation/articles/hdinsight-dotnet-avro-serialization/)

了解如何使用 Microsoft .NET Library for Avro 来将对象和其它数据结构序列化为字节流，以便将它们保留在内存、数据库或文件中。

####[HDInsight SDK 参考文档](http://msdn.microsoft.com/zh-cn/library/azure/dn479185.aspx)

使用 Azure HDInsight PowerShell 在 HDInsight 群集上配置和运行作业。开发使用 Azure HDInsight .NET SDK 管理 HDInsight 作业的应用程序。

####[调试 HDInsight 中的·Hadoop：解释错误消息](/zh-cn/documentation/articles/hdinsight-debug-jobs/)

了解在使用 PowerShell 管理 HDInsight 时可能会发生的错误以及从这些错误中恢复的步骤。

####[在 Storm 上用 SCP.NET 和 C# 开发流数据处理应用程序](/zh-cn/documentation/articles/hdinsight-hadoop-storm-scpdotnet-csharp-develop-streaming-data-processing-application/)

了解如何在 HDInsight 中的 Storm 上用 SCP.NET 和 C# 开发流数据处理应用程序。

###分析

####[使用 Storm 和 Hadoop 分析实时传感器数据](/zh-cn/documentation/articles/hdinsight-storm-sensor-data-analysis/)

了解如何生成一个在 HDInsight 中使用 Storm 集群来处理来自 Azure 事件中心的传感器数据的解决方案，然后将处理过的传感器数据作为近实时信息显示在一个基于 Web 的仪表板上。

####[使用 Mahout 生成影片建议](/zh-cn/documentation/articles/hdinsight-mahout/)

了解如何基于用户偏好使用 Mahout 推荐引擎来获取影片建议。Mahout 是 Apache Hadoop 的机器学习库，包含用于处理数据（例如筛选、分类和群集）的算法。

####[使用 Hadoop 通过 Giraph 执行图形分析](/zh-cn/documentation/articles/hdinsight-giraph/)

了解如何通过 Apache Giraph 查找 HDInsight 上使用 Hadoop 的对象之间的最短路径。使用 Giraph，你可以深入了解各种关系，例如社交网络（也称为“社交图”）上好友之间的关系，或者 Internet 等大型网络上路由器之间的关系。

####[使用 HDInsight 中的 Hadoop 分析航班延误数据](/zh-cn/documentation/articles/hdinsight-analyze-flight-delay-data/)

了解如何使用 Hive 计算各机场的航班平均延误时间，以及如何使用 Sqoop 将结果导入 SQL 数据库。

####[使用 Microsoft Hive ODBC Driver 将 Excel 连接到 Hadoop](/zh-cn/documentation/articles/hdinsight-connect-excel-hive-ODBC-driver/)

使用 Microsoft Hive ODBC Driver 将数据从 Azure HDInsight 导入到 Excel 中。

####[利用 Power Query 将 Excel 连接到 Hadoop](/zh-cn/documentation/articles/hdinsight-connect-excel-power-query/)

Microsoft 的大数据解决方案的一个关键功能是将 Apache Hadoop 与 Microsoft 商业智能 (BI) 组件可靠集成。了解如何使用 Power Query 将 HDInsight 数据导入 Excel 中。

###管理

####[HDInsight 中的 Hadoop 群集的可用性和可靠性](/zh-cn/documentation/articles/hdinsight-high-availability/)

HDInsight 群集已得到增强，可提供管理企业工作负载所需的可靠性和可用性。

####[使用管理门户管理 HDInsight 中的 Hadoop 群集](/zh-cn/documentation/articles/hdinsight-administer-use-management-portal/)

了解如何使用 Azure 管理门户创建 HDInsight 群集，以及如何打开管理工具。

####[使用 PowerShell 管理 HDInsight 中的 Hadoop 群集](/zh-cn/documentation/articles/hdinsight-administer-use-powershell/)

了解如何使用本地 Azure PowerShell 控制台管理 HDInsight 群集。

####[使用命令行接口管理 HDInsight 中的 Hadoop 群集](/zh-cn/documentation/articles/hdinsight-administer-use-command-line/)

了解如何使用跨平台命令行接口管理 HDInsight 群集。

####[使用 Ambari API 监视 HDInsight 中的 Hadoop 群集](/zh-cn/documentation/articles/hdinsight-monitor-use-ambari-api/)

使用 Apache Ambari API 设置、管理和监控 Hadoop 群集。Ambari 具有直观的操作员工具和强大的 API，可隐藏 Hadoop 的复杂性。

####[在 HDInsight 中将基于时间的 Oozie 协调器与 Hadoop 配合使用](/zh-cn/documentation/articles/hdinsight-use-oozie-coordinator-time/)

了解如何定义工作流和协调器，以及如何基于时间触发 Hadoop 作业。

####[配置 HDInsight 中的 Hadoop 群集](/zh-cn/documentation/articles/hdinsight-provision-clusters/)

了解如何使用 Azure 管理门户、PowerShell、命令行接口和 HDInsight .NET SDK 设置 HDInsight 群集。

####[配置 Azure Virtual Network 上的 HBase 群集](/zh-cn/documentation/articles/hdinsight-hbase-provision-vnet/)

在 Azure 虚拟网络上的 HDInsight 中创建 HBase 群集。虚拟网络集成允许应用程序直接与 HBase 通信，可提高性能和安全性。

####[为 HDInsight 中的 Hadoop 作业上载数据](/zh-cn/documentation/articles/hdinsight-upload-data/)

了解如何使用 Azure Storage Explorer、Azure PowerShell、Hadoop 命令行或 Sqoop 在 HDInsight 中上载和访问数据。

####[在 HDInsight 中将 Azure Blob 存储与 Hadoop 配合使用](/zh-cn/documentation/articles/hdinsight-use-blob-storage/)

了解 HDInsight 如何使用存储在 Azure Blob 存储中的数据、何时在 HDFS 中存储数据以及何时在 Blob 存储中存储数据。     

