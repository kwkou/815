<properties
   pageTitle="将 MapReduce 与 HDInsight 中的 Hadoop 配合使用 | Azure"
   description="了解如何使用远程桌面连接到 HDInsight 上的 Hadoop 并运行 MapReduce 作业。"
   services="hdinsight"
   documentationCenter=""
   authors="Blackmist"
   manager="paulettm"
   editor="cgronlun"/>
<tags ms.service="hdinsight"
    ms.date="02/18/2015"
    wacn.date="04/15/2015"
    />


# 通过远程桌面在 HDInsight 上的 Hadoop 中使用 MapReduce

[AZURE.INCLUDE [mapreduce-selector](../includes/hdinsight-selector-use-mapreduce.md)]

在本文中，你将学习如何使用远程桌面连接到 HDInsight 上的 Hadoop 群集，然后使用 Hadoop 命令运行 MapReduce 作业。

## <a id="prereq"></a>先决条件

若要完成本文中的步骤，你将需要：

* 基于 Windows 的 HDInsight（HDInsight 上的 Hadoop）群集

* Windows 7、8 或 10 客户端

## <a id="connect"></a>使用远程桌面进行连接

为 HDInsight 群集启用远程桌面，然后根据<a href="/documentation/articles/hdinsight-administer-use-management-portal/#rdp" target="_blank">使用 RDP 连接到 HDInsight 群集</a>中的说明与该群集建立连接：

## <a id="hadoop"></a>使用 Hadoop 命令

连接到 HDInsight 群集的桌面之后，请使用以下步骤，利用 Hadoop 命令来运行 MapReduce 作业。

1. 从 HDInsight 桌面启动"Hadoop 命令行"。这会在 **c:\apps\dist\hadoop-&lt;版本号>** 目录中打开新的命令提示符。

	> [AZURE.NOTE] 版本号会随着 Hadoop 更新而更改。**HADOOP_HOME** 环境变量可用来查找路径。例如，`cd %HADOOP_HOME%` 会将目录更改为 Hadoop 目录，而你并不需要知道版本号。

2. 若要使用 **Hadoop** 命令运行示例 MapReduce 作业，请使用以下命令。

		hadoop jar hadoop-mapreduce-examples.jar wordcount wasb:///example/data/gutenberg/davinci.txt wasb:///example/data/WordCountOutput

	这会启动 **wordcount** 类（包含在当前目录中的 **hadoop-mapreduce-examples.jar** 文件内）。作为输入，它会使用 **wasb://example/data/gutenberg/davinci.txt** 文档，输出将存储到 **wasb:///example/data/WordCountOutput**。

	> [AZURE.NOTE] 有关此 MapReduce 作业和示例数据的详细信息，请参阅<a href="/documentation/articles/hdinsight-use-mapreduce/">在 HDInsight Hadoop 中使用 MapReduce</a>。

2. 作业会在处理时发出详细信息，最后在作业完成时返回与下面类似的信息。

		File Input Format Counters
        Bytes Read=1395666
		File Output Format Counters
        Bytes Written=337623

3. 完成之后，请使用以下命令行列出存储到 **wasb://example/data/WordCountOutput** 中的输出文件。

		hadoop fs -ls wasb:///example/data/WordCountOutput

	这应会显示两个文件：**_SUCCESS** 和 **part-r-00000**。**part-r-00000** 文件包含此作业的输出。

	> [WACOM.NOTE] 某些 MapReduce 作业可能会将结果拆分成多个 **part-r-#####** 文件。如果是这样，请使用 ##### 后缀指示文件的顺序。

4. 若要查看输出，请使用以下命令。

		hadoop fs -cat wasb:///example/data/WordCountOutput/part-r-00000

	这会显示 **wasb://example/data/gutenberg/davinci.txt** 文件中包含的单词列表，以及每个单词的出现次数。下面是要包含在文件中的数据示例。

		wreathed        3
		wreathing       1
		wreaths 		1
		wrecked 		3
		wrenching       1
		wretched        6
		wriggling       1

## <a id="summary"></a>摘要

如你所见，Hadoop 命令提供简单的方法让你在 HDInsight 群集上运行 MapReduce 作业，然后查看作业输出。

## <a id="nextsteps"></a>后续步骤

有关 HDInsight 中的 MapReduce 作业的一般信息。

* [在 HDInsight Hadoop 上使用 MapReduce](/documentation/articles/hdinsight-use-mapreduce)

有关 HDInsight 上的 Hadoop 的其他使用方法的信息。

* [将 Hive 与 HDInsight 上的 Hadoop 配合使用](/documentation/articles/hdinsight-use-hive)

* [将 Pig 与 HDInsight 上的 Hadoop 配合使用](/documentation/articles/hdinsight-use-pig)

<!--HONumber=50-->