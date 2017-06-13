<properties
    pageTitle="将 MapReduce 和 SSH 连接与 HDInsight 中的 Hadoop 配合使用 | Azure"
    description="了解如何在 HDInsight 上的 Hadoop 中使用 SSH 运行 MapReduce 作业。"
    services="hdinsight"
    documentationcenter=""
    author="Blackmist"
    manager="jhubbard"
    editor="cgronlun"
    tags="azure-portal" />
<tags
    ms.assetid="844678ba-1e1f-4fda-b9ef-34df4035d547"
    ms.service="hdinsight"
    ms.custom="hdinsightactive"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="big-data"
    ms.date="05/03/2017"
    wacn.date="06/05/2017"
    ms.author="v-dazen"
    ms.translationtype="Human Translation"
    ms.sourcegitcommit="08618ee31568db24eba7a7d9a5fc3b079cf34577"
    ms.openlocfilehash="40b0865448f3e6b585305bf2f7eb442abd77ffd5"
    ms.contentlocale="zh-cn"
    ms.lasthandoff="05/26/2017" />

# <a name="use-mapreduce-with-hadoop-on-hdinsight-with-ssh"></a>通过 SSH 将 MapReduce 与 HDInsight 上的 Hadoop 配合使用

[AZURE.INCLUDE [mapreduce-selector](../../includes/hdinsight-selector-use-mapreduce.md)]

了解如何从安全外壳 (SSH) 将 MapReduce 作业提交到 HDInsight。

> [AZURE.NOTE]
> 如果已熟悉如何使用基于 Linux 的 Hadoop 服务器，但刚接触 HDInsight，请参阅[基于 Linux 的 HDInsight 提示](/documentation/articles/hdinsight-hadoop-linux-information/)。

## <a id="prereq"></a>先决条件

* 基于 Linux 的 HDInsight（HDInsight 上的 Hadoop）群集

    > [AZURE.IMPORTANT]
    > Linux 是在 HDInsight 3.4 版或更高版本上使用的唯一操作系统。 有关详细信息，请参阅 [HDInsight 组件版本控制](/documentation/articles/hdinsight-component-versioning/#hdi-version-33-nearing-deprecation-date)。

* SSH 客户端。 有关详细信息，请参阅[将 SSH 与 HDInsight 配合使用](/documentation/articles/hdinsight-hadoop-linux-use-ssh-unix/)

## <a id="ssh"></a>使用 SSH 进行连接

使用 SSH 连接到群集。 例如，以下命令将连接到名为 **myhdinsight** 的群集：

    ssh admin@myhdinsight-ssh.azurehdinsight.cn

**如果使用用于 SSH 身份验证的证书密钥**，则可能需要指定客户端系统上的私钥位置，例如：

    ssh -i ~/mykey.key admin@myhdinsight-ssh.azurehdinsight.cn

**如果使用用于 SSH 身份验证的密码**，则需要根据提示提供密码。

有关将 SSH 与 HDInsight 配合使用的详细信息，请参阅[将 SSH 与 HDInsight 配合使用](/documentation/articles/hdinsight-hadoop-linux-use-ssh-unix/)。

## <a id="hadoop"></a>使用 Hadoop 命令

1. 连接到 HDInsight 群集后，使用以下命令启动 MapReduce 作业：

        yarn jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-mapreduce-examples.jar wordcount /example/data/gutenberg/davinci.txt /example/data/WordCountOutput

    此命令启动 `hadoop-mapreduce-examples.jar` 文件中包含的 `wordcount` 类。 它使用 `/example/data/gutenberg/davinci.txt` 文档作为输入，并将输出存储在 `/example/data/WordCountOutput` 中。

    > [AZURE.NOTE]
    > 有关此 MapReduce 作业和示例数据的详细信息，请参阅[在 HDInsight 上的 Hadoop 中使用 MapReduce](/documentation/articles/hdinsight-use-mapreduce/)。

2. 作业在处理时提供详细信息，并在完成时返回类似于以下文本的信息：

        File Input Format Counters
        Bytes Read=1395666
        File Output Format Counters
        Bytes Written=337623

3. 作业完成后，使用以下命令列出输出文件：

        hdfs dfs -ls /example/data/WordCountOutput

    此命令显示两个文件（`_SUCCESS` 和 `part-r-00000`）。 `part-r-00000` 文件包含此作业的输出。

    > [AZURE.NOTE]
    > 某些 MapReduce 作业可能会将结果拆分成多个 **part-r-#####** 文件。 如果是这样，请使用 ##### 后缀指示文件的顺序。

4. 若要查看输出，请使用以下命令：

        hdfs dfs -cat /example/data/WordCountOutput/part-r-00000

    此命令会显示 **wasbs://example/data/gutenberg/davinci.txt** 文件中包含的单词以及每个单词出现的次数的列表。 以下文本是文件中所含数据的示例：

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

<!--Update_Description: wording update-->