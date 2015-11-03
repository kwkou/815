<properties
	pageTitle="在 HDInsight 中运行 Hadoop 示例 | Azure"
	description="使用所提供的示例开始使用 Azure HDInsight 服务。在数据群集中使用运行 MapReduce 程序的 PowerShell 脚本。"
	services="hdinsight"
	documentationCenter=""
	tags="azure-portal"
	authors="mumian"
	manager="paulettm"
	editor="cgronlun"/>

<tags
	ms.service="hdinsight"
	ms.date="07/09/2015"
	wacn.date="10/03/2015"/>




#在 HDInsight 中运行 Hadoop 示例

[AZURE.INCLUDE [samples-selector](../includes/hdinsight-run-samples-selector.md)]

为帮助你开始使用 Azure HDInsight 在 Hadoop 群集上运行 MapReduce 作业，我们提供了一组示例。在你创建的每一个 HDInsight 托管群集上都可以使用这些示例。运行这些示例会让你熟悉使用 Azure PowerShell cmdlet 在 Hadoop 群集上运行作业。

也可以使用 Microsoft .NET API for HDInsight 从应用程序中以编程方式运行 MapReduce 程序。有关使用 HDInsight API 提交作业的详细信息，请参阅 [在 HDInsight 中提交 Hadoop 作业][hdinsight-submit-hadoop-jobs-programmatically]。

Web 上有许多介绍 Hadoop 相关技术（例如基于 Java 的 MapReduce 编程和流式处理）的其他文档，以及有关 Windows PowerShell 脚本中使用的 cmdlet 的文档。有关这些资源的详细信息，请参阅 [Azure HDInsight 简介][hdinsight-introduction]中最后的 **HDInsight 资源**部分。

**这些示例具有的特性**

<p>这些示例旨在让你以最快的速度了解如何部署 Hadoop 作业，并向你提供一个可扩展的测试平台来让你熟悉服务所使用的概念和脚本过程。它们向你提供了常见任务的示例，例如，创建和导入不同大小的数据集、按顺序运行和组合作业以及检查作业的结果。所使用的数据集大小可能不同，这样你就可以观察不同大小的数据集对作业性能的影响。</p>


**先决条件**：

- **一个 Azure 订阅**。请参阅 [azure-trial](/pricing/1rmb-trial/) 页。

- **一个 HDInsight 群集**。有关可用于创建此类群集的不同方法的说明，请参阅[使用自定义选项在 HDInsight 中预配 Hadoop 群集](/documentation/articles/hdinsight-provision-clusters)

- **配备 Azure PowerShell 的工作站**。请参阅[如何安装和配置 Azure PowerShell][powershell-install-configure]。

## 示例 ##

HDInsight 随附了以下示例：

- [**pi estimator Hadoop 示例**][hdinsight-sample-pi-estimator]：演示如何使用 HDInsight 运行一个 MapReduce 程序，该程序使用统计 (quasi-Monte Carlo) 方法估算 pi 的值。
- [**在 Hadoop 群集上运行 MapReduce 单词计数示例**][hdinsight-sample-wordcount]：演示如何使用 HDInsight 群集来运行用于计算单词在文本文件中的出现次数的 MapReduce 程序。
- [**10-GB Graysort Hadoop 示例**][hdinsight-sample-10gb-graysort]：演示如何使用 HDInsight 对 10 GB 文件运行常规用途的 GraySort。有三个作业要运行：Teragen 生成数据，Terasort 对数据排序，而 Teravalidate 确认数据已正确排序。
- [**Hadoop 中的 C# 流式处理单词计数 MapReduce 示例**][hdinsight-sample-csharp-streaming]：演示如何使用 C# 来编写使用 Hadoop 流式处理接口的 MapReduce 程序。


## 如何运行示例 ##

这些示例可使用 Azure PowerShell 来运行。我们针对上述每个示例提供了具体操作的说明。

##后续步骤 ##

从本文和每个示例的相关文章中，你了解到如何使用 Azure PowerShell 运行 HDInsight 群集附带的示例。有关 Pig、Hive 和 MapReduce 如何与 HDInsight 配合使用的教程，请参阅以下主题：

* [将 Hadoop 与 HDInsight 中的 Hive 配合使用以分析手机使用情况][hdinsight-get-started]
* [将 Pig 与 HDInsight 上的 Hadoop 配合使用][hdinsight-use-pig]
* [将 Hive 与 HDInsight 上的 Hadoop 配合使用][hdinsight-use-hive]
* [在 HDInsight 中提交 Hadoop 作业][hdinsight-submit-jobs]
* [Azure HDInsight SDK 文档][hdinsight-sdk-documentation]
* [在 HDInsight 中调试 Hadoop：错误消息][hdinsight-errors]


[hdinsight-errors]: /documentation/articles/hdinsight-debug-jobs/

[hdinsight-sdk-documentation]: http://msdn.microsoft.com/zh-cn/library/dn479185.aspx

[hdinsight-submit-jobs]: /documentation/articles/hdinsight-submit-hadoop-jobs-programmatically/
[hdinsight-introduction]: /documentation/articles/hdinsight-hadoop-introduction/


[powershell-install-configure]: /documentation/articles/install-configure-powershell
[hdinsight-get-started]: /documentation/articles/hdinsight-get-started
[hdinsight-samples]: /documentation/articles/hdinsight-run-samples
[hdinsight-sample-10gb-graysort]: /documentation/articles/hdinsight-sample-10gb-graysort
[hdinsight-sample-csharp-streaming]: /documentation/articles/hdinsight-sample-csharp-streaming
[hdinsight-sample-pi-estimator]: /documentation/articles/hdinsight-sample-pi-estimator
[hdinsight-sample-wordcount]: /documentation/articles/hdinsight-sample-wordcount
[hdinsight-use-hive]: /documentation/articles/hdinsight-use-hive
[hdinsight-use-pig]: /documentation/articles/hdinsight-use-pig
<!---HONumber=71-->