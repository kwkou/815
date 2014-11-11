<properties urlDisplayName="Run Hadoop Samples in HDInsight" pageTitle="在 HDInsight 中运行 Hadoop 示例 | Azure "metaKeywords="hdinsight, hdinsight 示例, hadoop, mapreduce" description="Get started using the Azure HDInsight service with the samples provided. Use PowerShell scripts that run MapReduce programs on data clusters." metaCanonical="" services="hdinsight" documentationCenter="" title="Run the Hadoop samples in HDInsight" authors="bradsev" solutions="" manager="paulettm" editor="cgronlun" />

<tags ms.service="hdinsight" ms.workload="big-data" ms.tgt_pltfrm="na" ms.devlang="na" ms.topic="article" ms.date="01/01/1900" ms.author="bradsev" />




# 在 HDInsight 中运行 Hadoop 示例

为帮助您开始使用 Azure HDInsight 在 Hadoop 群集上运行 MapReduce 作业，我们提供了一组示例。在你创建的每一个 HDInsight 托管群集上都可以使用这些示例。 运行这些示例会让您熟悉使用 Azure PowerShell HDInsight cmdlet 在 Hadoop 群集上运行作业。

还可以使用 Microsoft .NET API for HDInsight 从应用程序中以编程方式运行 MapReduce 程序。有关使用 HDInsight API 提交作业的更多信息，请参见[以编程方式提交 Hadoop 作业][hdinsight-submit-jobs]。

Web 上具有许多介绍 Hadoop 相关技术（例如基于 Java 的 MapReduce 编程和流式处理）的其他文档，以及介绍如何在 PowerShell 脚本中使用 cmdlet 的文档。 有关这些资源的更多信息，请参阅 [Azure HDInsight 简介][hdinsight-introduction]主题中最后的 **HDInsight 资源**部分。

**这些示例具有的特性**

<p>这些示例旨在让您以最快的速度了解如何部署 Hadoop 作业，并向您提供一个可扩展的测试平台来让您熟悉服务所使用的概念和脚本过程。它们向你提供了常见任务的示例，例如，创建和导入不同大小的数据集、按顺序运行作业和组合作业以及检查作业的结果。所使用的数据集大小可能不同，这样您就可以观察不同大小的数据集对作业性能的影响。</p>


**先决条件**：	

- 您必须具有 Azure 帐户。 有关注册帐户的选项，请参阅[免费试用 Azure](http://www.windowsazure.cn/zh-cn/pricing/free-trial/) 页。

- 您必须配置了 HDInsight 群集。有关可用于创建这种群集的各种不同方法的说明，请参阅[设置 HDInsight 群集](../hdinsight-provision-clusters/)。

- 您必须已经安装了 Azure PowerShell，并且已将其配置为可用于您的帐户。 有关如何进行此安装的说明，请参阅[安装和配置 Azure PowerShell][powershell-install-configure]。

## 示例 ##

HDInsight 随附了以下示例。

- [**Pi Estimator 示例**][hdinsight-sample-pi-estimator] 本教程演示如何使用 HDInsight 运行一个 MapReduce 程序，该程序使用统计方法（拟蒙特卡罗法）估算 Pi 的值。
- [**WordCount 示例**][hdinsight-sample-wordcount] 本教程演示如何使用 HDInsight 群集来运行用于计算单词在文本文件中的出现次数的 MapReduce 程序。
- [**10-GB Graysort 示例**][hdinsight-sample-10gb-graysort] 本教程演示如何使用 HDInsight 对 10 GB 文件运行通用 GraySort。有三个作业要运行：Teragen 生成数据，Terasort 对数据排序，而 Teravalidate 确认数据已正确排序。
- [**C# 流式处理示例**][hdinsight-sample-csharp-streaming] 本教程演示如何使用 C# 来编写使用 Hadoop 流接口的 MapReduce 程序。 


## 如何运行示例 ##

这些示例可使用 Azure PowerShell 来运行。在上面链接的页面上提供了针对每个示例如何实现的说明。

## 后续步骤 ##

从本文和每个示例的相关文章中，您了解到如何使用 Azure PowerShell 运行 HDInsight 群集附带的示例。有关使用 HDInsight 附带的 Pig、Hive 和 MapReduce 的教程，请参见以下主题：

* [Azure HDInsigth 服务入门][hdinsight-get-started]
* [将 Pig 与 HDInsight 配合使用][hdinsight-use-pig]
* [将 Hive 与 HDInsight 配合使用][hdinsight-use-hive]
* [以编程方式提交 Hadoop 作业][hdinsight-submit-jobs]
* [Azure HDInsight SDK 文档][hdinsight-sdk-documentation]
* [调试 HDInsight：错误消息][hdinsight-errors]


[hdinsight-errors]: ../hdinsight-debug-jobs/

[hdinsight-sdk-documentation]: http://msdn.microsoft.com/zh-cn/library/dn469975.aspx

[hdinsight-submit-jobs]: ../hdinsight-submit-hadoop-jobs-programmatically/
[hdinsight-introduction]: ../hdinsight-hadoop-introduction/



[Powershell-install-configure]: ../install-configure-powershell/

[hdinsight-get-started]: ../hdinsight-get-started/

[hdinsight-samples]: ../hdinsight-run-samples/
[hdinsight-sample-10gb-graysort]: ../hdinsight-sample-10gb-graysort/
[hdinsight-sample-csharp-streaming]: ../hdinsight-sample-csharp-streaming/
[hdinsight-sample-pi-estimator]: ../hdinsight-sample-pi-estimator/
[hdinsight-sample-wordcount]: ../hdinsight-sample-wordcount/

[hdinsight-use-hive]: ../hdinsight-use-hive/
[hdinsight-use-pig]: ../hdinsight-use-pig/

[submit-jobs-programmatically]: ../hdinsight-submit-hadoop-jobs-programmatically/
[hdinsight-resources]: ../hdinsight-introduction/

