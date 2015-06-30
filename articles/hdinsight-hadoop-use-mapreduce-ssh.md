<properties
   pageTitle="将 MapReduce 与 HDInsight 中的 Hadoop 配合使用 | Azure"
   description="了解如何在 HDInsight 上的 Hadoop 中使用 SSH 运行 MapReduce 作业。"
   services="hdinsight"
   documentationCenter=""
   authors="Blackmist"
   manager="paulettm"
   editor="cgronlun"/>
<tags ms.service="hdinsight"
    ms.date="02/18/2015"
    wacn.date="04/15/2015"
    />


# 通过 SSH 将 Hive 与 HDInsight 上的 Hadoop 配合使用

[AZURE.INCLUDE [mapreduce-selector](../includes/hdinsight-selector-use-mapreduce.md)]

在本文中，你将了解如何使用 SSH 连接到 HDInsight 上的 Hadoop 群集，然后使用 Hadoop 命令提交 MapReduce 作业。

> [AZURE.NOTE] 如果你已熟悉如何使用基于 Linux 的 Hadoop 服务器，但刚接触 HDInsight，请参阅<a href="/documentation/articles/hdinsight-hadoop-linux-information/" target="_blank">基于 Linux 的 HDInsight 上的 Hadoop 须知信息</a>。

## <a id="prereq"></a>先决条件

若要完成本文中的步骤，你将需要：

* 基于 Linux 的 HDInsight（HDInsight 上的 Hadoop）群集

* SSH 客户端。SSH 客户端上应该装有 Linux、Unix 和 Mac OS。Windows 用户必须下载 <a href="http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html" target="_blank">Putty</a> 之类的客户端。

## <a id="ssh"></a>使用 SSH 连接

使用 SSH 命令连接到 HDInsight 群集的完全限定域名 (FQDN)。FQDN 是你为群集指定的名称后接 **.azurehdinsight.cn**。例如，以下命令将连接到名为 **myhdinsight** 的群集。

	ssh admin@myhdinsight-ssh.azurehdinsight.cn

**如果你在创建 HDInsight 群集时提供了 SSH 身份验证的证书密钥**，则可能需要指定客户端系统上的私钥位置。

	ssh admin@myhdinsight-ssh.azurehdinsight.cn -i ~/mykey.key

**如果你在创建 HDInsight 群集时提供了 SSH 身份验证的密码**，则需要根据提示提供该密码。

### Putty（Windows 客户端）

Windows 未提供内置的 SSH 客户端。建议使用 **Putty**，可以从 <a href="http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html" target="_blank">http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html</a> 下载。

有关使用 Putty 的详细信息，请参阅<a href="/documentation/articles/virtual-machines-linux-use-ssh-key/" target="_blank">如何在 Azure 上通过 Linux 使用 SSH</a>中的**使用 Putty 连接到 Linux 计算机**部分。

> [WACOM.NOTE] 如果你对 HDInsight 群集使用了 SSH 身份验证的证书，则还需要参阅<a href="/documentation/articles/virtual-machines-linux-use-ssh-key/" target="_blank">如何在 Azure 上通过 Linux 使用 SSH</a>中的**为 Putty 创建 PPK** 部分

## <a id="hadoop"></a>使用 Hadoop 命令

1. 连接到 HDInsight 群集后，请执行以下步骤，以使用 **Hadoop** 命令启动 MapReduce 作业。

		hadoop jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-mapreduce-examples.jar wordcount wasb:///example/data/gutenberg/davinci.txt wasb:///example/data/WordCountOutput

	这会启动包含在 **hadoop-mapreduce-examples.jar** 文件中的 **wordcount** 类。它使用 **wasb://example/data/gutenberg/davinci.txt** 文档作为输入，输出将存储到 **wasb:///example/data/WordCountOutput**。

	> [AZURE.NOTE] 有关此 MapReduce 作业和示例数据的详细信息，请参阅<a href="/documentation/articles/hdinsight-use-mapreduce/" target="_blank">在 HDInsight 上的 Hadoop 中使用 MapReduce</a>。

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