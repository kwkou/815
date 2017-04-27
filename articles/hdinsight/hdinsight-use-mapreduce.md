<properties
    pageTitle="将 MapReduce 与 Hadoop on HDInsight 配合使用 | Azure"
    description="学习如何在 HDInsight 群集中的 Hadoop 上运行 MapReduce 作业。用户将运行基本单词计数操作，将其作为 Java MapReduce 作业进行实现。"
    services="hdinsight"
    documentationcenter=""
    author="Blackmist"
    manager="jhubbard"
    editor="cgronlun"
    tags="azure-portal" />
<tags
    ms.assetid="7f321501-d62c-4ffc-b5d6-102ecba6dd76"
    ms.service="hdinsight"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="big-data"
    ms.date="01/12/2017"
    wacn.date="01/25/2017"
    ms.author="larryfr" />  


# 在 HDInsight 上的 Hadoop 中使用 MapReduce

[AZURE.INCLUDE [mapreduce-selector](../../includes/hdinsight-selector-use-mapreduce.md)]

本文介绍了如何在 HDInsight 群集中的 Hadoop 上运行 MapReduce 作业。我们将运行一个基本的单词计数操作，将其作为 Java MapReduce 作业进行实现。

## <a id="whatis"></a>什么是 MapReduce？

Hadoop MapReduce 是一个软件框架，用于编写处理海量数据的作业。输入数据已拆分成独立的区块，然后在群集中的节点之间并行处理。MapReduce 作业包括两个函数：

* **Mapper**：使用输入数据，（通常使用筛选器和排序操作）分析数据，然后发出元组（键值对）

* **Reducer**：使用 Mapper 发出的元组并执行汇总运算，根据 Mapper 数据生成更小的合并结果

下图演示了一个基本的单词计数 MapReduce 作业示例：

![HDI.WordCountDiagram][image-hdi-wordcountdiagram]

此作业的输出是每个单词在所分析的文本中出现的次数。

* mapper 将输入文本中的每一行作为一个输入并将其拆分为多个单词。每当一个单词出现时，mapper 发出一个键/值对，其中在该单词后跟一个 1。然后将输出排序，再发送到 reducer。
* 随后，reducer 将每个单词的计数求和，再发出一个键/值对，其中包含单词及其出现的总次数。

可以使用多种语言实现 MapReduce。Java 是最常见的实现，本文档中使用该语言进行演示。

### Hadoop 流式处理

与 Java 应用程序类似，基于 Java 和 Java 虚拟机（例如 Scalding 或 Cascading）的语言或框架可以作为 MapReduce 作业直接运行。其他语言（例如 C# 或 Python）或独立的可执行文件必须使用 Hadoop 流式处理。

Hadoop 流式处理通过 STDIN 和 STDOUT 与 mapper 和 reducer 通信 - mapper 和 reducer 每次从 STDIN 读取一行数据，并将输出写入到 STDOUT。mapper 和 reducer 读取或发出的每行数据必须采用制表符分隔的键/值对格式：

    [key]/t[value]

有关详细信息，请参阅 [Hadoop 流式处理](http://hadoop.apache.org/docs/r1.2.1/streaming.html)。

有关将 Hadoop 流式处理与 hdinsight 配合使用的示例，请参阅：

* [开发 Python MapReduce 作业](/documentation/articles/hdinsight-hadoop-streaming-python/)

## <a id="data"></a>关于示例数据

在本示例中，使用 Leonardo Da Vinci 的笔记本作为示例数据，它在 HDInsight 群集中以文本文档形式提供。

示例数据存储在 Azure Blob 存储中，HDInsight 可以将该存储用作 Hadoop 群集的默认文件系统。HDInsight 可以使用 **wasb** 前缀访问 Blob 存储中存储的文件。例如，若要访问 sample.log 文件，可使用以下语法：

    wasbs:///example/data/gutenberg/davinci.txt

由于 Azure Blob 存储是 HDInsight 的默认存储，因此也可以使用 **/example/data/gutenberg/davinci.txt** 访问该文件。

> [AZURE.NOTE]
在上述语法中，**wasbs:///** 用于访问 HDInsight 群集的默认存储容器中存储的文件。如果你在设置群集时指定了其他存储帐户，并想要访问这些帐户中存储的文件，可以通过指定容器名称和存储帐户地址来访问数据。例如，**wasbs://mycontainer@mystorage.blob.core.chinacloudapi.cn/example/data/gutenberg/davinci.txt**。

## <a id="job"></a>关于示例 MapReduce

本示例中使用的 MapReduce 作业位于 HDInsight 群集随附的 **wasbs://example/jars/hadoop-mapreduce-examples.jar** 中。其中包含一个对 **davinci.txt** 运行单词计数的示例。

> [AZURE.NOTE]
在 HDInsight 2.1 群集上，该文件位置为 **wasbs:///example/jars/hadoop-examples.jar**。

下面提供了用于单词计数 MapReduce 作业的 Java 代码，以供参考：

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

有关编写自己的 MapReduce 作业的说明，请参阅[为 HDInsight 开发 Java MapReduce 程序](/documentation/articles/hdinsight-develop-deploy-java-mapreduce-linux/)。

## <a id="run"></a>运行 MapReduce

HDInsight 可以使用各种方法运行 HiveQL 作业。使用下表来确定哪种方法最适合你，然后访问此链接进行演练。

| **使用此方法**... | **...实现此目的** | ...使用此**群集操作系统** | ...从此**客户端操作系统** |
|:--- |:--- |:--- |:--- |
| [SSH](/documentation/articles/hdinsight-hadoop-use-mapreduce-ssh/) |通过 **SSH** 使用 Hadoop 命令 |Linux |Linux、Unix、Mac OS X 或 Windows |
| [Curl](/documentation/articles/hdinsight-hadoop-use-mapreduce-curl/) |使用 **REST** 远程提交作业 |Linux 或 Windows |Linux、Unix、Mac OS X 或 Windows |
| [Windows PowerShell](/documentation/articles/hdinsight-hadoop-use-mapreduce-powershell/) |使用 **Windows PowerShell** 远程提交作业 |Linux 或 Windows |Windows |
| [远程桌面](/documentation/articles/hdinsight-hadoop-use-mapreduce-remote-desktop/) |通过**远程桌面**使用 Hadoop 命令 |Windows |Windows |

[AZURE.INCLUDE [hdinsight-linux-acn-version.md](../../includes/hdinsight-linux-acn-version.md)]

> [AZURE.IMPORTANT]
Linux 是在 HDInsight 3.4 或更高版本上使用的唯一操作系统。有关详细信息，请参阅 [HDInsight 在 Windows 上弃用](/documentation/articles/hdinsight-component-versioning/#hdi-version-32-and-33-nearing-deprecation-date)。

## <a id="nextsteps"></a>后续步骤

虽然 MapReduce 提供了强大的诊断功能，但掌握起来可能会比较困难。此处提供了多个基于 Java 的框架，使定义 MapReduce 应用程序更轻松，还提供了一些技术（例如 Pig 和 Hive），使得在 HDInsight 中处理数据更方便。若要了解更多信息，请参阅下列文章：

* [为 HDInsight 开发 Java MapReduce 程序](/documentation/articles/hdinsight-develop-deploy-java-mapreduce-linux/)
* [开发适用于 HDInsight 的 Python 流式处理 MapReduce 程序](/documentation/articles/hdinsight-hadoop-streaming-python/)
* [使用 Apache Hadoop on HDInsight 开发 Scalding MapReduce 作业](/documentation/articles/hdinsight-hadoop-mapreduce-scalding/)
* [将 Hive 与 HDInsight 配合使用][hdinsight-use-hive]
* [将 Pig 与 HDInsight 配合使用][hdinsight-use-pig]
* [运行 HDInsight 示例][hdinsight-samples]

[hdinsight-upload-data]: /documentation/articles/hdinsight-upload-data/
[hdinsight-get-started]: /documentation/articles/hdinsight-hadoop-linux-tutorial-get-started/
[hdinsight-develop-mapreduce-jobs]: /documentation/articles/hdinsight-develop-deploy-java-mapreduce-linux/
[hdinsight-use-hive]: /documentation/articles/hdinsight-use-hive/
[hdinsight-use-pig]: /documentation/articles/hdinsight-use-pig/
[hdinsight-samples]: /documentation/articles/hdinsight-run-samples/
[hdinsight-provision]: /documentation/articles/hdinsight-hadoop-provision-linux-clusters/

[powershell-install-configure]: https://docs.microsoft.com/powershell/azureps-cmdlets-docs

[image-hdi-wordcountdiagram]: ./media/hdinsight-use-mapreduce/HDI.WordCountDiagram.gif

<!---HONumber=Mooncake_0120_2017-->
<!--Update_Description: update from ASM to ARM-->