<properties
   pageTitle="将 MapReduce 和 SSH 连接与 HDInsight 中的 Hadoop 配合使用 | Azure"
   description="了解如何在 HDInsight 上的 Hadoop 中使用 SSH 运行 MapReduce 作业。"
   services="hdinsight"
   documentationCenter=""
   authors="Blackmist"
   manager="paulettm"
   editor="cgronlun"
	tags="azure-portal"/>

<tags
   ms.service="hdinsight" 
   ms.date="07/06/2015"
   wacn.date="02/06/2017"/>

# 通过 SSH 将 MapReduce 与 HDInsight 上的 Hadoop 配合使用

[AZURE.INCLUDE [mapreduce-selector](../../includes/hdinsight-selector-use-mapreduce.md)]

在本文中，你将了解如何使用 Secure Shell (SSH) 连接到 HDInsight 群集上的 Hadoop，然后通过使用 Hadoop 命令提交 MapReduce 作业。

> [AZURE.NOTE]如果你已熟悉如何使用基于 Linux 的 Hadoop 服务器，但刚接触 HDInsight，请参阅[基于 Linux 的 HDInsight 提示](/documentation/articles/hdinsight-hadoop-linux-information/)。

##<a id="prereq"></a>先决条件

若要完成本文中的步骤，你将需要：

* 基于 Linux 的 HDInsight（HDInsight 上的 Hadoop）群集

* SSH 客户端。SSH 客户端上应该安装有 Linux、Unix 和 Mac 操作系统。Windows 用户必须下载 [PuTTY](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html) 之类的客户端。

##<a id="ssh"></a>使用 SSH 进行连接

使用 SSH 命令连接到 HDInsight 群集的完全限定域名 (FQDN)。FQDN 将是你为群集提供的名称，后跟 **.azurehdinsight.net**。例如，以下命令将连接到名为 **myhdinsight** 的群集：

	ssh admin@myhdinsight-ssh.azurehdinsight.cn

如果你在创建 HDInsight 群集时**提供了用于 SSH 身份验证的证书密钥**，则可能需要指定客户端系统上的私钥位置，例如：

	ssh admin@myhdinsight-ssh.azurehdinsight.cn -i ~/mykey.key

如果你在创建 HDInsight 群集时**提供了 SSH 身份验证的密码**，则需要根据提示提供该密码。

有关将 SSH 与 HDInsight 配合使用的详细信息，请参阅[在 Linux、OS X 和 Unix 中的 HDInsight 上将 SSH 与基于 Linux 的 Hadoop 配合使用](/documentation/articles/hdinsight-hadoop-linux-use-ssh-unix/)。

###PuTTY（Windows 客户端）

Windows 未提供内置的 SSH 客户端。建议使用可从 [http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html) 下载的 **PuTTY**。

有关使用 PuTTY 的详细信息，请参阅[从 Windows 在 HDInsight 上配合使用 SSH 与基于 Linux 的 Hadoop](/documentation/articles/hdinsight-hadoop-linux-use-ssh-windows/)。

##<a id="hadoop"></a>使用 Hadoop 命令

1. 连接到 HDInsight 群集后，使用以下 **Hadoop** 命令来启动 MapReduce 作业：

		hadoop jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-mapreduce-examples.jar wordcount wasbs:///example/data/gutenberg/davinci.txt wasbs:///example/data/WordCountOutput

	这会启动包含在 **hadoop-mapreduce-examples.jar** 文件中的 **wordcount** 类。它使用 ****wasbs://example/data/gutenberg/davinci.txt** 文档作为输入，输出将存储到 ****wasbs:///example/data/WordCountOutput**。

	> [AZURE.NOTE]有关此 MapReduce 作业和示例数据的详细信息，请参阅[在 HDInsight 上的 Hadoop 中使用 MapReduce](/documentation/articles/hdinsight-use-mapreduce/)。

2. 作业在处理时提供详细信息，并在完成时返回如下信息：

		File Input Format Counters
        Bytes Read=1395666
		File Output Format Counters
        Bytes Written=337623

3. 在作业完成时，使用以下命令列出存储在 ****wasbs://example/data/WordCountOutput** 上的输出文件：

		hadoop fs -ls wasbs:///example/data/WordCountOutput

	这应会显示两个文件：**\_SUCCESS** 和 **part-r-00000**。**part-r-00000** 文件包含此作业的输出。

	> [AZURE.NOTE]某些 MapReduce 作业可能会将结果拆分成多个 **part-r-#####** 文件。如果是这样，请使用 ##### 后缀指示文件的顺序。

4. 若要查看输出，请使用以下命令：

		hadoop fs -cat wasbs:///example/data/WordCountOutput/part-r-00000

	这会显示 ****wasbs://example/data/gutenberg/davinci.txt** 文件中包含的单词列表，以及每个单词的出现次数。下面是要包含在文件中的数据示例：

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

<!---HONumber=71-->