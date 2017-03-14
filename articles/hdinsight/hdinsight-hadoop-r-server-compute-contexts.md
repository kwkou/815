<properties
    pageTitle="适用于 HDInsight 上的 R Server 的计算上下文选项 | Azure"
    description="了解用户可用于 HDInsight 上的 R Server 的不同计算上下文选项"
    services="HDInsight"
    documentationcenter=""
    author="jeffstokes72"
    manager="jhubbard"
    editor="cgronlun" />
<tags 
    ms.assetid="0deb0b1c-4094-459b-94fc-ec9b774c1f8a"
    ms.service="HDInsight"
    ms.devlang="R"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="data-services"
    ms.date="01/09/2017"
    wacn.date="02/14/2017"
    ms.author="jeffstok" />

# Compute context options for R Server on HDInsight（适用于 HDInsight 上的 R Server 的计算上下文选项）
Azure HDInsight 上的 Microsoft R Server 提供最新的基于 R 的分析功能。它在 [Azure Blob](/documentation/articles/storage-introduction/ "Azure Blob 存储") 存储帐户或本地 Linux 文件系统中使用存储在容器的 HDFS 中的数据。由于 R Server 基于开放源代码的 R 构建，因此你构建的基于 R 的应用程序可以利用超过 8000 个任意开放源代码 R 包。这些应用程序还可以利用 [ScaleR](https://www.microsoft.com/zh-cn/cloud-platform/r-server "Revolution Analytics ScaleR")（R Server 附带的 Microsoft 的大数据分析包）中的例程。

群集的边缘节点为连接到群集和运行 R 脚本提供了便捷的位置。使用边缘节点，你可以选择跨边缘节点服务器的各个核心运行 ScaleR 的并行化分布式函数。你还可以选择通过使用 ScaleR 的 Hadoop Map Reduce 或 Spark 计算上下文跨群集的各个节点运行这些函数。

## 边缘节点的计算上下文
一般而言，在边缘节点上的 R Server 中运行的 R 脚本将在该节点上的 R 解释程序内运行。但是，调用 ScaleR 函数的步骤例外。ScaleR 调用将在计算环境中运行，而计算环境取决于你如何设置 ScaleR 计算上下文。从边缘节点运行 R 脚本时，计算上下文的可能值包括本地顺序（“local”）、本地并行（“localpar”）、Map Reduce 和 Spark。

“Local”和“localpar”选项的区别只体现在 rxExec 调用的执行方式。这两个选项都以并行方式跨所有可用核心执行其他 rx-function 调用，除非使用 ScaleR numCoresToUse 作了其他指定，例如，rxOptions(numCoresToUse=6)。下面汇总了各种计算上下文选项

| 计算上下文 | 设置方式 | 执行上下文 |
| ---------------- | ------------------------------- | ---------------------------------------- |
| 本地顺序 | rxSetComputeContext('local') | 跨边缘节点服务器的核心并行执行，但 rxExec 调用除外（这种调用是串行执行的） |
| 本地并行 | rxSetComputeContext('localpar') | 跨边缘节点服务器的核心并行执行 |
| Spark | RxSpark() | 通过 Spark 跨 HDI 群集的各个节点并行化分布式执行 |
| Map Reduce | RxHadoopMR() | 通过 Map Reduce 跨 HDI 群集的各个节点并行化分布式执行 |

假设你希望并行化执行以改进性能，有三个选项可供选择。要选择哪个选项取决于分析工作的性质，以及数据的大小和位置。

## 有关确定计算上下文的指导原则
目前，没有哪个公式可以告诉你要使用哪个计算上下文。但是，有些指导原则可帮助你做出正确的选择，或至少可以帮助你在运行基准测试之前缩小选择范围。这些指导原则包括：

1. 本地 Linux 文件系统比 HDFS 更快。
2. 如果数据在本地且采用 XDF，则重复分析的速度更快。
3. 最好是从文本数据源流式传输少量数据；如果数据量较大，可将其转换为 XDF 格式再进行分析。
4. 对于极大量的数据，可将数据复制或流式传输到边缘节点进行分析所造成的开销将变得难以控制。
5. 在 Hadoop 中进行分析时，Spark 比 Map Reduce 更快。

鉴于这些原则，有一些用于选择计算上下文的常规经验规则：

### Local
* 如果要分析的数据量较小，并且不需要重复的分析，则直接将其流式处理为分析例程，并使用“local”或“localpar”。
* 如果要分析的数据量较小或者大小适中并且需要重复分析，可将其复制到本地文件系统，导入到 XDF，然后通过“local”或“localpar”进行分析。

### Hadoop Spark
* 如果要分析的数据量较大，可使用 RxHiveData 或 RxParquetData 将它导入到 Spark DataFrame，或导入到 HDFS 中的 XDF（除非存储有问题），然后通过“Spark”进行分析。

### Hadoop Map Reduce
* 仅当使用 Spark 计算上下文遇到无法解决的问题时才使用这种方式，因为它的速度通常较慢。

## 关于 rxSetComputeContext 的内联帮助
有关详细信息和 ScaleR 计算上下文的示例，请参阅 R 中关于 rxSetComputeContext 方法的内联帮助，例如：

    > ?rxSetComputeContext

也可以参考 [R Server MSDN](https://msdn.microsoft.com/zh-cn/library/mt674634.aspx "MSDN 上的 R Server") 库中提供的 [ScaleR Distributed Computing Guide](https://msdn.microsoft.com/microsoft-r/scaler-distributed-computing)（ScaleR 分布式计算指南）。

## 后续步骤
在本文中，你已了解如何创建包含 R Server 的新 HDInsight 群集。此外，你还了解了有关从 SSH 会话使用 R 控制台的基本知识。接下来，你可以阅读以下文章，探索在 HDInsight 上使用 R Server 的其他方法：

* [Overview of R Server for Hadoop（Hadoop 的 R Server 概述）](/documentation/articles/hdinsight-hadoop-r-server-overview/)
* [Get started with R server for Hadoop](/documentation/articles/hdinsight-hadoop-r-server-get-started/)（Hadoop 的 R Server 入门）
* [将 RStudio Server 添加到 HDInsight（如果未在群集创建过程中添加）](/documentation/articles/hdinsight-hadoop-r-server-install-r-studio/)
* [Azure Storage options for R Server on HDInsight](/documentation/articles/hdinsight-hadoop-r-server-storage/)（适用于 HDInsight 上的 R Server 的 Azure 存储选项）

<!---HONumber=Mooncake_1205_2016-->