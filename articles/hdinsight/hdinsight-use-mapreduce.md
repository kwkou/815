<properties
    pageTitle="将 MapReduce 与 Hadoop on HDInsight 配合使用 | Azure"
    description="学习如何在 HDInsight 群集中的 Hadoop 上运行 MapReduce 作业。"
    services="hdinsight"
    documentationcenter=""
    author="Blackmist"
    manager="jhubbard"
    editor="cgronlun"
    tags="azure-portal"
    translationtype="Human Translation" />
<tags
    ms.assetid="7f321501-d62c-4ffc-b5d6-102ecba6dd76"
    ms.service="hdinsight"
    ms.custom="hdinsightactive"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="big-data"
    ms.date="04/03/2017"
    wacn.date="05/08/2017"
    ms.author="larryfr"
    ms.sourcegitcommit="2c4ee90387d280f15b2f2ed656f7d4862ad80901"
    ms.openlocfilehash="58d7a353faaed4ef86304da21933952888154d7c"
    ms.lasthandoff="04/28/2017" />

# <a name="use-mapreduce-in-hadoop-on-hdinsight"></a>在 Hadoop on HDInsight 中使用 MapReduce

了解如何在 HDInsight 群集上运行 MapReduce 作业。 使用下表找到可将 MapReduce 与 HDInsight 配合使用的各种方法：

| **请使用以下方法**... | **...实现此目的** | ...使用此 **群集操作系统** | ...从此 **客户端操作系统** |
|:--- |:--- |:--- |:--- |
| [SSH](/documentation/articles/hdinsight-hadoop-use-mapreduce-ssh/) |通过 **SSH** |Linux |Linux、Unix、Mac OS X 或 Windows |
| [REST](/documentation/articles/hdinsight-hadoop-use-mapreduce-curl/) |使用 **REST**（示例使用 cURL）远程提交作业 |Linux 或 Windows |Linux、Unix、Mac OS X 或 Windows |
| [Windows PowerShell](/documentation/articles/hdinsight-hadoop-use-mapreduce-powershell/) |使用 **Windows PowerShell** |Linux 或 Windows |Windows |
| [远程桌面](/documentation/articles/hdinsight-hadoop-use-mapreduce-remote-desktop/)（HDInsight 3.2 和 3.3） |通过**远程桌面**使用 Hadoop 命令 |Windows |Windows |

> [AZURE.IMPORTANT]
> Linux 是 HDInsight 3.4 或更高版本上使用的唯一操作系统。 有关详细信息，请参阅[弃用 HDInsight 3.2 和 3.3](/documentation/articles/hdinsight-component-versioning/#hdi-version-33-nearing-deprecation-date)。

## <a id="whatis"></a>什么是 MapReduce

Hadoop MapReduce 是一个软件框架，用于编写处理海量数据的作业。 输入数据已拆分成独立的区块，这些区块将在群集中的节点之间并行处理。 MapReduce 作业包括两个函数：

* **映射器**：使用输入数据，对数据进行分析（通常使用筛选器和排序操作），然后发出元组（键/值对）

* **化简器**：使用映射器发出的元组并执行汇总运算，以基于映射器数据创建更小的合并结果

下图演示了一个基本的单词计数 MapReduce 作业示例：

![HDI.WordCountDiagram][image-hdi-wordcountdiagram]

此作业的输出是每个单词在所分析的文本中出现的次数。

* mapper 将输入文本中的每一行作为一个输入并将其拆分为多个单词。 每当一个单词出现时，mapper 发出一个键/值对，其中在该单词后跟一个 1。 然后将输出排序，再发送到 reducer。
* 随后，化简器会计算每个单词的计数的和并发出一个键/值对（包含单词，后跟该单词的总出现次数）。

MapReduce 可使用多种语言实现。 Java 是最常见的实现，本文档中使用该语言进行演示。

## <a name="development-languages"></a>开发语言

基于 Java 和 Java 虚拟机的语言或框架可作为 MapReduce 作业直接运行。 在本文档中使用的示例是 Java MapReduce 应用程序。 诸如 C#、Python 的非 Java 语言或独立可执行文件必须使用 Hadoop 流式处理。

Hadoop 流式处理通过 STDIN 和 STDOUT 与映射器和化简器通信。 映射器和化简器从 STDIN 中一次读取一行数据，并将输出写入 STDOUT。 映射器和化简器读取或发出的每行必须采用制表符分隔的键/值对格式：

    [key]/t[value]

有关详细信息，请参阅 [Hadoop Streaming](http://hadoop.apache.org/docs/r1.2.1/streaming.html)（Hadoop 流式处理）。

有关将 Hadoop 流式处理与 HDInsight 配合使用的示例，请参阅以下文档：

* [开发 C# MapReduce 作业](/documentation/articles/hdinsight-hadoop-dotnet-csharp-mapreduce-streaming/)

* [开发 Python MapReduce 作业](/documentation/articles/hdinsight-hadoop-streaming-python/)

## <a id="data"></a>示例数据

HDInsight 提供存储在 `/example/data` 和 `/HdiSamples` 目录中的各种示例数据集。 这些目录位于群集的默认存储中。 在本文档中，我们使用 `/example/data/gutenberg/davinci.txt` 文件。 此文件包含 Leonardo Da Vinci 的笔记本。

## <a id="job"></a>MapReduce 示例

MapReduce 单词计数应用程序示例包含在 HDInsight 群集中。 此示例位于群集默认存储的 `/example/jars/hadoop-mapreduce-examples.jar` 中。

以下 Java 代码是包含在 `hadoop-mapreduce-examples.jar` 文件中的 MapReduce 应用程序的源代码：

    package org.apache.hadoop.examples;

    import java.io.IOException;
    import java.util.StringTokenizer;

    import org.apache.hadoop.conf.Configuration;
    import org.apache.hadoop.fs.Path;
    import org.apache.hadoop.io.IntWritable;
    import org.apache.hadoop.io.Text;
    import org.apache.hadoop.mapreduce.Job;
    import org.apache.hadoop.mapreduce.Mapper;
    import org.apache.hadoop.mapreduce.Reducer;
    import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
    import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
    import org.apache.hadoop.util.GenericOptionsParser;

    public class WordCount {

        public static class TokenizerMapper
            extends Mapper<Object, Text, Text, IntWritable>{

        private final static IntWritable one = new IntWritable(1);
        private Text word = new Text();

        public void map(Object key, Text value, Context context
                        ) throws IOException, InterruptedException {
            StringTokenizer itr = new StringTokenizer(value.toString());
            while (itr.hasMoreTokens()) {
            word.set(itr.nextToken());
            context.write(word, one);
            }
        }
        }

        public static class IntSumReducer
            extends Reducer<Text,IntWritable,Text,IntWritable> {
        private IntWritable result = new IntWritable();

        public void reduce(Text key, Iterable<IntWritable> values,
                            Context context
                            ) throws IOException, InterruptedException {
            int sum = 0;
            for (IntWritable val : values) {
            sum += val.get();
            }
            result.set(sum);
            context.write(key, result);
        }
        }

        public static void main(String[] args) throws Exception {
        Configuration conf = new Configuration();
        String[] otherArgs = new GenericOptionsParser(conf, args).getRemainingArgs();
        if (otherArgs.length != 2) {
            System.err.println("Usage: wordcount <in> <out>");
            System.exit(2);
        }
        Job job = new Job(conf, "word count");
        job.setJarByClass(WordCount.class);
        job.setMapperClass(TokenizerMapper.class);
        job.setCombinerClass(IntSumReducer.class);
        job.setReducerClass(IntSumReducer.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);
        FileInputFormat.addInputPath(job, new Path(otherArgs[0]));
        FileOutputFormat.setOutputPath(job, new Path(otherArgs[1]));
        System.exit(job.waitForCompletion(true) ? 0 : 1);
        }
    }

有关编写自己的 MapReduce 应用程序的说明，请参阅以下文档：

* [为 HDInsight 开发 Java MapReduce 应用程序](/documentation/articles/hdinsight-develop-deploy-java-mapreduce-linux/)

* [为 HDInsight 开发 Python MapReduce 应用程序](/documentation/articles/hdinsight-hadoop-streaming-python/)

## <a id="run"></a>运行 MapReduce

HDInsight 可以使用各种方法运行 HiveQL 作业。 使用下表来确定哪种方法最适合你，然后按链接进行演练。

| **使用此方法**... | **...实现此目的** | ...使用此 **群集操作系统** | ...从此 **客户端操作系统** |
|:--- |:--- |:--- |:--- |
| [SSH](/documentation/articles/hdinsight-hadoop-use-mapreduce-ssh/) |通过 **SSH** |Linux |Linux、Unix、Mac OS X 或 Windows |
| [Curl](/documentation/articles/hdinsight-hadoop-use-mapreduce-curl/) |使用 **REST** |Linux 或 Windows |Linux、Unix、Mac OS X 或 Windows |
| [Windows PowerShell](/documentation/articles/hdinsight-hadoop-use-mapreduce-powershell/) |使用 **Windows PowerShell** |Linux 或 Windows |Windows |
| [远程桌面](/documentation/articles/hdinsight-hadoop-use-mapreduce-remote-desktop/)（HDInsight 3.2 和 3.3） |通过**远程桌面**使用 Hadoop 命令 |Windows |Windows |

[AZURE.INCLUDE [hdinsight-linux-acn-version.md](../../includes/hdinsight-linux-acn-version.md)]

> [AZURE.IMPORTANT]
> Linux 是 HDInsight 3.4 或更高版本上使用的唯一操作系统。 有关详细信息，请参阅[弃用 HDInsight 3.2 和 3.3](/documentation/articles/hdinsight-component-versioning/#hdi-version-33-nearing-deprecation-date)。

## <a id="nextsteps"></a>后续步骤

若要了解如何使用 HDInsight 中的数据的详细信息，请参阅以下文档：

* [为 HDInsight 开发 Java MapReduce 程序](/documentation/articles/hdinsight-develop-deploy-java-mapreduce-linux/)

* [为 HDInsight 开发 Python 流式处理 MapReduce 程序](/documentation/articles/hdinsight-hadoop-streaming-python/)

* [使用 Apache Hadoop on HDInsight 开发 Scalding MapReduce 作业](/documentation/articles/hdinsight-use-mapreduce/)

* [将 Hive 与 HDInsight 配合使用][hdinsight-use-hive]

* [将 Pig 与 HDInsight 配合使用][hdinsight-use-pig]

[hdinsight-upload-data]: /documentation/articles/hdinsight-upload-data/
[hdinsight-get-started]: /documentation/articles/hdinsight-hadoop-linux-tutorial-get-started/
[hdinsight-develop-mapreduce-jobs]: /documentation/articles/hdinsight-develop-deploy-java-mapreduce-linux/
[hdinsight-use-hive]: /documentation/articles/hdinsight-use-hive/
[hdinsight-use-pig]: /documentation/articles/hdinsight-use-pig/

[powershell-install-configure]: https://docs.microsoft.com/zh-cn/powershell/azureps-cmdlets-docs

[image-hdi-wordcountdiagram]: ./media/hdinsight-use-mapreduce/HDI.WordCountDiagram.gif

<!--Update_Description: wording update-->