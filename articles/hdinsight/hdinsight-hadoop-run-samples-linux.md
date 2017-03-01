<properties
    pageTitle="在 HDInsight 上运行 Hadoop MapReduce 示例 | Azure"
    description="开始将 MapReduce 示例与 HDInsight 配合使用。使用 SSH 连接到群集，然后使用 Hadoop 命令来运行示例作业。"
    services="hdinsight"
    documentationcenter=""
    author="Blackmist"
    manager="jhubbard"
    editor="cgronlun"
    tags="azure-portal" />
<tags 
    ms.assetid="e1d2a0b9-1659-4fab-921e-4a8990cbb30a"
    ms.service="hdinsight"
    ms.workload="big-data"
    ms.tgt_pltfrm="na"
    ms.devlang="na"
    ms.topic="article"
    ms.date="01/12/2017"
    wacn.date="01/25/2017"
    ms.author="larryfr" />

# 在 HDInsight 中运行 Hadoop 示例
[AZURE.INCLUDE [samples-selector](../../includes/hdinsight-run-samples-selector.md)]

HDInsight 群集提供了一组 MapReduce 示例，可以通过这些示例来熟悉如何运行 Hadoop MapReduce 作业。在本文档中，你将学习提供的示例并了解如何运行部分示例。

## 先决条件

* **基于 Linux 的 HDInsight 群集**：请参阅[在 Linux 上的 HDInsight 中开始将 Hadoop 与 Hive 配合使用](/documentation/articles/hdinsight-hadoop-linux-tutorial-get-started/)

    [AZURE.INCLUDE [hdinsight-linux-acn-version.md](../../includes/hdinsight-linux-acn-version.md)]

    > [AZURE.IMPORTANT]
    Linux 是在 HDInsight 3.4 版或更高版本上使用的唯一操作系统。有关详细信息，请参阅 [HDInsight 在 Windows 上弃用](/documentation/articles/hdinsight-component-versioning/#hdi-version-32-and-33-nearing-deprecation-date)。

* **SSH 客户端**：有关如何将 SSH 与 HDInsight 配合使用的信息，请参阅以下文章：
  
    * [在 Linux、Unix 或 OS X 中的 HDInsight 上将 SSH 与基于 Linux 的 Hadoop 配合使用](/documentation/articles/hdinsight-hadoop-linux-use-ssh-unix/)
    * [在 Windows 中的 HDInsight 上将 SSH 与基于 Linux 的 Hadoop 配合使用](/documentation/articles/hdinsight-hadoop-linux-use-ssh-windows/)

## 示例
**位置**：这些示例位于 HDInsight 群集的 **/usr/hdp/current/hadoop-mapreduce-client/hadoop-mapreduce-examples.jar** 中

**内容**：下面的示例都包含在该存档文件中：

* **aggregatewordcount**：基于聚合的映射/化简程序，用于计算输入文件中单词的数目
* **aggregatewordhist**：基于聚合的映射/化简程序，用于计算输入文件中单词的直方图
* **bbp**：一种映射/化简程序，使用 Bailey-Borwein-Plouffe 来计算 Pi 的精确位数
* **dbcount**：一个示例作业，用于对数据库中存储的 pageview 日志进行计数
* **distbbp**：一种映射/化简程序，使用 BBP 类型的公式来计算 Pi 的精确位数
* **grep**：一种映射/化简程序，用于计算输入中某个正则表达式的匹配数
* **联接**：一种作业，可以将经过排序且等分的数据集联接起来
* **multifilewc**：一种作业，可对多个文件中的单词计数
* **pentomino**：一种映射/化简平铺排列程序，用于查找五格拼板问题的解决方案
* **pi**：一种映射/化简程序，可使用拟蒙特卡罗法估算 Pi 值
* **randomtextwriter**：一种映射/化简程序，可以为每个节点写入 10GB 的随机文本数据
* **randomwriter**：一种映射/化简程序，可以为每个节点写入 10GB 的随机数据
* **secondarysort**：一个示例，用于定义化简时的次级排序
* **排序**：一种映射/化简程序，用于对随机写入器所写入的数据进行排序
* **sudoku**：数独解算器
* **teragen**：为 terasort 生成数据
* **terasort**：运行 terasort
* **teravalidate**：检查 terasort 的结果
* **wordcount**：一种映射/化简程序，用于计算输入文件中单词的数目
* **wordmean**：一种映射/化简程序，用于计算输入文件中单词的平均长度
* **wordmedian**：一种映射/化简程序，用于计算输入文件中单词的中间长度
* **wordstandarddeviation**：一种映射/化简程序，用于计算输入文件中单词长度的标准差

**源代码**：这些示例的源代码提供在 HDInsight 群集的 **/usr/hdp/2.2.4.9-1/hadoop/src/hadoop-mapreduce-project/hadoop-mapreduce-examples** 中

> [AZURE.NOTE]
路径中的 `2.2.4.9-1` 是 HDInsight 群集的 Hortonworks 数据平台的版本，在 HDInsight 更新时可能会变化。
> 
> 

## 如何运行示例
1. 使用 SSH 连接到 HDInsight，如以下文章所述：
   
    * [在 Linux、Unix 或 OS X 中的 HDInsight 上将 SSH 与基于 Linux 的 Hadoop 配合使用](/documentation/articles/hdinsight-hadoop-linux-use-ssh-unix/)
    * [在 Windows 中的 HDInsight 上将 SSH 与基于 Linux 的 Hadoop 配合使用](/documentation/articles/hdinsight-hadoop-linux-use-ssh-windows/)
2. 在 `username@#######:~$` 提示符下，使用以下命令列出示例：
   
        yarn jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-mapreduce-examples.jar
   
    这将生成本文档前一部分中的示例列表。
3. 使用以下命令来获取有关特定示例的帮助。在本例中为 **wordcount** 示例：
   
        yarn jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-mapreduce-examples.jar wordcount
   
    你应该会收到以下消息：
   
        Usage: wordcount <in> [<in>...] <out>
   
    这指示可以为源文档提供多个输入路径。最终路径是输出（源文档中单词的计数）的存储位置。
4. 使用以下语句来计算《达芬奇笔记》（已作为示例数据提供给你的群集）中所有单词的数目：
   
        yarn jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-mapreduce-examples.jar wordcount /example/data/gutenberg/davinci.txt /example/data/davinciwordcount
   
    此作业的输入从 **wasbs:///example/data/gutenberg/davinci.txt** 读取。
   
    此示例的输出将存储在 **wasbs:///example/data/davinciwordcount** 中。
   
    > [AZURE.NOTE]
    如字数统计示例帮助中所述，你还可以指定多个输入文件。例如，`hadoop jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-mapreduce-examples.jar wordcount /example/data/gutenberg/davinci.txt /example/data/gutenberg/ulysses.txt /example/data/twowordcount` 会计算 davinci.txt 和 ulysses.txt 中单词的数目。
    > 
    > 
5. 作业完成后，使用以下命令查看输出：
   
        hdfs dfs -cat /example/data/davinciwordcount/*
   
    这将连接通过该作业生成的所有输出文件并显示它们。此基本示例只有一个文件，但如果有更多文件，此命令将循环访问所有这些文件。
   
    输出与下面类似：
   
        zum     1
        zur     1
        zwanzig 1
        zweite  1
   
    每一行表示一个单词，并显示它在输入数据中的出现次数。

## 数独
数独示例的以下使用说明似乎帮助不大：“在命令行中包括一个拼图”。

[数独](https://en.wikipedia.org/wiki/Sudoku)是由九个 3x3 网格组成的逻辑拼图。网格中的某些单元格包含数字，其他单元格为空白，游戏目的是找出空白单元格。上面的链接包含更多有关此拼图的信息，不过此示例的目的是找出空白单元格。因此，我们的输入应该是采用以下格式的文件：

* 九行加九列
* 每一列可能包含数字或 `?`（表示空白单元格）
* 用空格来分隔单元格

可以通过特定方式来构造数独游戏，只需确保每一列或行的数字不重复即可。HDInsight 群集上已有一个正确构造的示例。该示例位于 **/usr/hdp/2.2.4.9-1/hadoop/src/hadoop-mapreduce-project/hadoop-mapreduce-examples/src/main/java/org/apache/hadoop/examples/dancing/puzzle1.dta** 上，其中包含以下内容：

    8 5 ? 3 9 ? ? ? ?
    ? ? 2 ? ? ? ? ? ?
    ? ? 6 ? 1 ? ? ? 2
    ? ? 4 ? ? 3 ? 5 9
    ? ? 8 9 ? 1 4 ? ?
    3 2 ? 4 ? ? 8 ? ?
    9 ? ? ? 8 ? 5 ? ?
    ? ? ? ? ? ? 2 ? ?
    ? ? ? ? 4 5 ? 7 8

> [AZURE.NOTE]
对 HDInsight 群集进行更新时，路径的 `2.2.4.9-1` 部分可能会变化。
> 
> 

若要通过数独示例来运行此内容，请使用以下命令：

    yarn jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-mapreduce-examples.jar sudoku /usr/hdp/2.2.9.1-1/hadoop/src/hadoop-mapreduce-project/hadoop-mapreduce-examples/src/main/java/org/apache/hadoop/examples/dancing/puzzle1.dta

结果应如下所示：

    8 5 1 3 9 2 6 4 7
    4 3 2 6 7 8 1 9 5
    7 9 6 5 1 4 3 8 2
    6 1 4 8 2 3 7 5 9
    5 7 8 9 6 1 4 2 3
    3 2 9 4 5 7 8 1 6
    9 4 7 2 8 6 5 3 1
    1 8 5 7 3 9 2 6 4
    2 6 3 1 4 5 9 7 8

## Pi (π)
该示例使用统计学方法（拟蒙特卡罗法）来估算 pi 值。如果单位正方形有一个内切圆，则正方面内部随机分布的点位于该圆中的概率等于圆圈面积与正方形面积之比，即 pi/4。pi 的值可以从 4R 的值估算出来，其中 R 是圆圈内的点数与正方形内的总点数之比。点数越多，估算值越准确。

此示例的映射器会在单位平方形内部随机生成多个点，然后计算位于圆圈内部的那些点的数目。

然后，化简器累计由映射器统计的点数，并根据公式 4R 估算 pi 的值，其中 R 是圆圈内统计的点数与方形内总点数的比率。

请使用以下命令运行此示例。该命令使用 16 个映射（每个映射 10,000,000 个示例）来估算 pi 值：

    yarn jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-mapreduce-examples.jar pi 16 10000000

此命令返回的值应类似于 **3.14159155000000000000**。例如，pi 采用前 10 位小数时为 3.1415926535。

## 10GB Greysort
GraySort 是一个基准排序，其指标是在对巨量数据（通常至少 100TB）排序时达到的排序速率（TB/分钟）。

此示例使用适度的 10GB 数据，这样运行速度相对更快。它使用由 Owen O'Malley 和 Arun Murthy 开发的 MapReduce 应用程序，此应用程序以 0.578TB/分钟（100TB 用时 173 分钟）的速率荣获 2009 年年度常用（“daytona”）TB 级排序基准。有关此排序基准和其他排序基准的详细信息，请参阅 [Sortbenchmark](http://sortbenchmark.org/) 网站。

本示例使用三组 MapReduce 程序：

* **TeraGen**：一种 MapReduce 程序，用于生成要排序的数据行
* **TeraSort**：以输入数据为例，使用 MapReduce 将数据排序到总数订单中
  
    TeraSort 是 MapReduce 函数的一种标准排序，但自定义的分区程序除外，此分区程序使用 N-1 个抽样键（用于定义每次简化的键范围）的已排序列表。尤其是，会将 sample[i-1] <= key < sample[i] 的所有键发送到化简变量 i。这样可确保化简变量 i 的输出均小于化简变量 i+1 的输出。
* **TeraValidate**：一个 MapReduce 程序，用于验证输出是否已全局排序
  
    它在输出目录中对于每个文件创建一个映射，每个映射都确保每个键均小于或等于前一个键。映射函数也会生成每个文件的第一个和最后一个键的记录，并且化简函数可确保文件 i 的第一个键大于文件 i-1 的最后一个键。任何问题都会报告为包含无次序键的化简的输出结果。

使用以下步骤来生成数据，排序，然后对输出进行验证：

1. 生成 10GB 数据，这些数据将存储到 HDInsight 群集的默认存储位置的 **wasbs:///example/data/10GB-sort-input** 中：
   
        yarn jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-mapreduce-examples.jar teragen -Dmapred.map.tasks=50 100000000 /example/data/10GB-sort-input
   
    `-Dmapred.map.tasks` 告诉 Hadoop 多少个映射任务将用于此作业。最后两个参数指示作业创建 10GB 大小的数据并将其存储在 **wasbs:///example/data/10GB-sort-input** 中。
2. 使用以下命令对数据排序：
   
        yarn jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-mapreduce-examples.jar terasort -Dmapred.map.tasks=50 -Dmapred.reduce.tasks=25 /example/data/10GB-sort-input /example/data/10GB-sort-output
   
    `-Dmapred.reduce.tasks` 告诉 Hadoop 多少个化简任务将用于此作业。最后两个参数只是数据的输入和输出位置。
3. 使用以下方法来验证排序所生成的数据：
   
        yarn jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-mapreduce-examples.jar teravalidate -Dmapred.map.tasks=50 -Dmapred.reduce.tasks=25 /example/data/10GB-sort-output /example/data/10GB-sort-validate

## 后续步骤
在本文中，你学习了如何运行基于 Linux 的 HDInsight 群集附带的示例。有关将 Pig、Hive 和 MapReduce 用于 HDInsight 的教程，请参阅以下主题：

* [将 Pig 与 HDInsight 上的 Hadoop 配合使用][hdinsight-use-pig]
* [将 Hive 与 HDInsight 上的 Hadoop 配合使用][hdinsight-use-hive]
* [将 MapReduce 与 HDInsight 上的 Hadoop 配合使用][hdinsight-use-mapreduce]

[hdinsight-errors]: /documentation/articles/hdinsight-debug-jobs/
[hdinsight-use-mapreduce]: /documentation/articles/hdinsight-use-mapreduce/
[hdinsight-sdk-documentation]: https://msdn.microsoft.com/zh-cn/library/azure/dn479185.aspx

[hdinsight-submit-jobs]: /documentation/articles/hdinsight-submit-hadoop-jobs-programmatically/
[hdinsight-introduction]: /documentation/articles/hdinsight-hadoop-introduction/


[hdinsight-samples]: /documentation/articles/hdinsight-run-samples/

[hdinsight-use-hive]: /documentation/articles/hdinsight-use-hive/
[hdinsight-use-pig]: /documentation/articles/hdinsight-use-pig/

<!---HONumber=Mooncake_0120_2017-->