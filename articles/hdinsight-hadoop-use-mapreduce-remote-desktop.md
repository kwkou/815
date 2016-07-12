<properties
   pageTitle="将 MapReduce 和远程桌面与 HDInsight 中的 Hadoop 配合使用 | Azure"
   description="了解如何使用远程桌面连接到 HDInsight 上的 Hadoop 并运行 MapReduce 作业。"
   services="hdinsight"
   documentationCenter=""
   authors="Blackmist"
   manager="paulettm"
   editor="cgronlun"
   tags="azure-portal"/>

<tags
	ms.service="hdinsight"
	ms.date="04/22/2016"
	wacn.date="06/29/2016"/>

# 通过远程桌面在 HDInsight 上的 Hadoop 中使用 MapReduce

[AZURE.INCLUDE [mapreduce-selector](../includes/hdinsight-selector-use-mapreduce.md)]

在本文中，你将学习如何通过使用远程桌面连接到 HDInsight 群集上的 Hadoop，然后通过使用 Hadoop 命令运行 MapReduce 作业。

##<a id="prereq"></a>先决条件

若要完成本文中的步骤，你将需要：

* 基于 Windows 的 HDInsight（HDInsight 上的 Hadoop）群集

* 运行 Windows 10、Windows 8 或 Windows 7 的客户端计算机

##<a id="connect"></a>使用远程桌面进行连接

为 HDInsight 群集启用远程桌面，然后根据[使用 RDP 连接到 HDInsight 群集](/documentation/articles/hdinsight-administer-use-management-portal-v1/#rdp)中的说明连接到该群集。

##<a id="hadoop"></a>使用 Hadoop 命令

连接到 HDInsight 群集的桌面之后，请使用以下步骤，以通过 Hadoop 命令来运行 MapReduce 作业：

1. 从 HDInsight 桌面启动“Hadoop 命令行”。这将在 **c:\\apps\\dist\\hadoop-&lt;version number>** 目录中打开新的命令提示符。

	> [AZURE.NOTE]版本号会随着 Hadoop 更新而更改。**HADOOP\_HOME** 环境变量可用来查找路径。例如，`cd %HADOOP_HOME%` 会将目录更改为 Hadoop 目录，而不需要你知道版本号。

2. 若要使用 **Hadoop** 命令运行示例 MapReduce 作业，请使用以下命令：

		hadoop jar hadoop-mapreduce-examples.jar wordcount wasb:///example/data/gutenberg/davinci.txt wasb:///example/data/WordCountOutput

	这将启动 **wordcount** 类（包含在当前目录中的 **hadoop-mapreduce-examples.jar** 文件内）。它使用 **wasb://example/data/gutenberg/davinci.txt** 文档作为输入，输出将存储到 **wasb:///example/data/WordCountOutput**。

	> [AZURE.NOTE]有关此 MapReduce 作业和示例数据的详细信息，请参阅<a href="/documentation/articles/hdinsight-use-mapreduce/\">在 HDInsight Hadoop 中使用 MapReduce</a>。

2. 作业在处理时提供详细信息，并在完成时返回如下信息：

		File Input Format Counters
        Bytes Read=1395666
		File Output Format Counters
        Bytes Written=337623

3. 在作业完成时，使用以下命令以列出存储在 **wasb://example/data/WordCountOutput** 上的输出文件：

		hadoop fs -ls wasb:///example/data/WordCountOutput

	这应会显示两个文件：**\_SUCCESS** 和 **part-r-00000**。**part-r-00000** 文件包含此作业的输出。

	> [AZURE.NOTE]某些 MapReduce 作业可能会将结果拆分成多个 **part-r-#####** 文件。如果是这样，请使用 ##### 后缀指示文件的顺序。

4. 若要查看输出，请使用以下命令：

		hadoop fs -cat wasb:///example/data/WordCountOutput/part-r-00000

	这会显示 **wasb://example/data/gutenberg/davinci.txt** 文件中包含的单词列表，以及每个单词的出现次数。下面是要包含在文件中的数据示例：

		wreathed        3
		wreathing       1
		wreaths 		1
		wrecked 		3
		wrenching       1
		wretched        6
		wriggling       1

##<a id="summary"></a>摘要

如你所见，Hadoop 命令提供简单的方法让你在 HDInsight 群集上运行 MapReduce 作业，然后查看作业输出。

##<a id="nextsteps"></a>后续步骤

有关 HDInsight 中的 MapReduce 作业的一般信息：

* [在 HDInsight Hadoop 上使用 MapReduce](/documentation/articles/hdinsight-use-mapreduce/)

有关 HDInsight 上的 Hadoop 的其他使用方法的信息：

* [将 Hive 与 HDInsight 上的 Hadoop 配合使用](/documentation/articles/hdinsight-use-hive/)

* [将 Pig 与 HDInsight 上的 Hadoop 配合使用](/documentation/articles/hdinsight-use-pig/)

<!---HONumber=79-->