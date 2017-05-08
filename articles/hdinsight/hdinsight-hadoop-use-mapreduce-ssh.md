<properties
    pageTitle="将 MapReduce 和 SSH 连接与 HDInsight 中的 Hadoop 配合使用 | Azure"
    description="了解如何在 HDInsight 上的 Hadoop 中使用 SSH 运行 MapReduce 作业。"
    services="hdinsight"
    documentationcenter=""
    author="Blackmist"
    manager="jhubbard"
    editor="cgronlun"
    tags="azure-portal"
    translationtype="Human Translation" />
<tags
    ms.assetid="844678ba-1e1f-4fda-b9ef-34df4035d547"
    ms.service="hdinsight"
    ms.custom="hdinsightactive"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="big-data"
    ms.date="02/08/2017"
    wacn.date="05/08/2017"
    ms.author="larryfr"
    ms.sourcegitcommit="2c4ee90387d280f15b2f2ed656f7d4862ad80901"
    ms.openlocfilehash="b89745a56554640346a908a0d8942a141a69989a"
    ms.lasthandoff="04/28/2017" />

# <a name="use-mapreduce-with-hadoop-on-hdinsight-with-ssh"></a>通过 SSH 将 MapReduce 与 HDInsight 上的 Hadoop 配合使用

[AZURE.INCLUDE [mapreduce-selector](../../includes/hdinsight-selector-use-mapreduce.md)]

本文将介绍如何使用安全外壳 (SSH) 连接到 HDInsight 群集上的 Hadoop，然后如何通过使用 Hadoop 命令提交 MapReduce 作业。

> [AZURE.NOTE]
> 如果已熟悉如何使用基于 Linux 的 Hadoop 服务器，但刚接触 HDInsight，请参阅[基于 Linux 的 HDInsight 提示](/documentation/articles/hdinsight-hadoop-linux-information/)。

## <a id="prereq"></a>先决条件

若要完成本文中的步骤，需要做好以下准备：

* 基于 Linux 的 HDInsight（HDInsight 上的 Hadoop）群集

    > [AZURE.IMPORTANT]
    > Linux 是 HDInsight 3.4 或更高版本上使用的唯一操作系统。 有关详细信息，请参阅 [HDInsight 在 Windows 上即将弃用](/documentation/articles/hdinsight-component-versioning/#hdi-version-33-nearing-deprecation-date)。

* SSH 客户端。 SSH 客户端上应该安装有 Linux、Unix 和 Mac 操作系统。 Windows 用户必须下载 [PuTTY](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html)之类的客户端。

## <a id="ssh"></a>使用 SSH 进行连接

使用 SSH 命令连接到 HDInsight 群集的完全限定域名 (FQDN)。 FQDN 是你为群集指定的名称，后跟 **.azurehdinsight.cn**。 例如，以下命令将连接到名为 **myhdinsight**的群集：

    ssh admin@myhdinsight-ssh.azurehdinsight.cn

**提供了用于 SSH 身份验证的证书密钥** ，则可能需要指定客户端系统上的私钥位置，例如：

    ssh -i ~/mykey.key admin@myhdinsight-ssh.azurehdinsight.cn

如果创建 HDInsight 群集时**提供了 SSH 身份验证密码**，则需要在系统提示时提供该密码。

有关将 SSH 与 HDInsight 配合使用的详细信息，请参阅[将 SSH 与 HDInsight 配合使用](/documentation/articles/hdinsight-hadoop-linux-use-ssh-unix/)。

## <a id="hadoop"></a>使用 Hadoop 命令

1. 连接到 HDInsight 群集后，使用以下 **Hadoop** 命令来启动 MapReduce 作业：

        yarn jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-mapreduce-examples.jar wordcount /example/data/gutenberg/davinci.txt /example/data/WordCountOutput

    这会启动包含在 **hadoop-mapreduce-examples.jar** 文件中的 **wordcount** 类。 作为输入，它会使用 **/example/data/gutenberg/davinci.txt** 文档，输出将存储在 **/example/data/WordCountOutput**。

    > [AZURE.NOTE]
    > 有关此 MapReduce 作业和示例数据的详细信息，请参阅[在 HDInsight 上的 Hadoop 中使用 MapReduce](/documentation/articles/hdinsight-use-mapreduce/)。

2. 作业在处理时提供详细信息，并在完成时返回如下信息：

        File Input Format Counters
        Bytes Read=1395666
        File Output Format Counters
        Bytes Written=337623

3. 作业完成之后，请使用以下命令行列出存储在 **wasbs://example/data/WordCountOutput** 中的输出文件：

        hdfs dfs -ls /example/data/WordCountOutput

    这应会显示两个文件：**_SUCCESS** 和 **part-r-00000**。 **part-r-00000** 文件包含此作业的输出。

    > [AZURE.NOTE]
    > 某些 MapReduce 作业可能会将结果拆分成多个 **part-r-#####** 文件。 如果是这样，请使用 ##### 后缀指示文件的顺序。

4. 若要查看输出，请使用以下命令：

        hdfs dfs -cat /example/data/WordCountOutput/part-r-00000

    这会显示 **wasbs://example/data/gutenberg/davinci.txt** 文件中包含的单词列表，以及每个单词的出现次数。 下面是文件中所含数据的示例：

        wreathed        3
        wreathing       1
        wreaths         1
        wrecked         3
        wrenching       1
        wretched        6
        wriggling       1

## <a id="summary"></a>摘要

如你所见，Hadoop 命令提供简单的方法让你在 HDInsight 群集上运行 MapReduce 作业，然后查看作业输出。

## <a id="nextsteps"></a>后续步骤

有关 HDInsight 中的 MapReduce 作业的一般信息：

* [在 HDInsight Hadoop 上使用 MapReduce](/documentation/articles/hdinsight-use-mapreduce/)

有关 HDInsight 上 Hadoop 的其他使用方法的信息：

* [将 Hive 与 Hadoop on HDInsight 配合使用](/documentation/articles/hdinsight-use-hive/)
* [将 Pig 与 Hadoop on HDInsight 配合使用](/documentation/articles/hdinsight-use-pig/)