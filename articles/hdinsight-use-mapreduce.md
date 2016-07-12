<properties
   pageTitle="将 MapReduce 与 HDInsight 上的 Hadoop 配合使用 | Azure"
   description="学习如何在 HDInsight 上的 Hadoop 群集中运行 MapReduce 作业。你将运行一个实现为 Java MapReduce 作业的基本单词计数操作。"
   services="hdinsight"
   documentationCenter=""
   authors="Blackmist"
   manager="paulettm"
   editor="cgronlun"
	tags="azure-portal"/>

<tags
	ms.service="hdinsight"
	ms.date="03/18/2016"
	wacn.date="04/26/2016"/>

# 在 HDInsight 上的 Hadoop 中使用 MapReduce

[AZURE.INCLUDE [mapreduce-selector](../includes/hdinsight-selector-use-mapreduce.md)]

在本文中，你将学习如何在 HDInsight 上的 Hadoop 群集中运行 MapReduce 作业。我们将运行一个实现为 Java MapReduce 作业的基本单词计数操作。

##<a id="whatis"></a>什么是 MapReduce？

Hadoop MapReduce 是一个软件框架，用于编写处理海量数据的作业。输入数据已拆分成独立的区块，这些区块将在群集中的节点之间并行处理。MapReduce 作业包括两个函数：

* **映射器**：使用输入数据，对数据进行分析（通常使用筛选器和排序操作），然后发出 Tuple（键/值对）
* **化简器**：使用映射器发出的 Tuple，并执行汇总运算，以基于映射器数据创建更小的合并结果

下图演示了一个基本的单词计数 MapReduce 作业示例：

![HDI.WordCountDiagram][image-hdi-wordcountdiagram]

此作业的输出是某个单词在所分析的文本中出现的次数。

* 映射器将输入文本中的每行用作一个输入并将其拆分为多个单词。每当文本中的单词后跟一个 1 时，映射器将发出一个键/值对。输出在发送到化简器之前经过排序。

* 随后，化简器会计算每个单词的计数的和并发出一个键/值对（包含单词，后跟该单词的总出现次数）。

可以使用多种语言实现 MapReduce。Java 是最常见的实现，本文档中使用该语言进行演示。

### Hadoop 流式处理

与 Java 应用程序一样，基于 Java 和 Java 虚拟机（例如 Scalding 或 Cascading）的语言或框架可以直接作为 MapReduce 作业运行。其他实体（例如 C# 或 Python）或独立的可执行文件必须使用 Hadoop 流式处理。

Hadoop 流式处理通过 STDIN 和 STDOUT 与映射器和化简器通信 - 映射器和化简器每次从 STDIN 读取一行数据，并将输出写入到 STDOUT。读取的每行或者由映射器和化简器发出的每行必须采用制表符分隔的键/值对格式：

    [key]/t[value]

有关详细信息，请参阅 [Hadoop 流式处理](http://hadoop.apache.org/docs/r1.2.1/streaming.html)。

有关将 Hadoop 流式处理与 hdinsight 配合使用的示例，请参阅：

* [开发 C# Hadoop 流式处理程序](/documentation/articles/hdinsight-hadoop-develop-deploy-streaming-jobs/)

##<a id="data"></a>关于示例数据

对于本示例中的示例数据，你将使用 HDInsight 群集中作为文本文档提供的 Leonardo Da Vinci 笔记本。

示例数据存储在 Azure Blob 存储中，HDInsight 可以将该存储用作 Hadoop 群集的默认文件系统。HDInsight 可以使用 **wasb** 前缀访问 Blob 存储中存储的文件。例如，若要访问 sample.log 文件，可使用以下语法：

	wasb:///example/data/gutenberg/davinci.txt

由于 Azure Blob 存储是 HDInsight 的默认存储，因此你也可以使用 **/example/data/gutenberg/davinci.txt** 访问该文件。

> [AZURE.NOTE]在上述语法中，****wasb:///** 用于访问 HDInsight 群集的默认存储容器中存储的文件。如果你在设置群集时指定了其他存储帐户，并想要访问这些帐户中存储的文件，你可以指定容器名称和存储帐户地址来访问数据。例如 ****wasb://mycontainer@mystorage.blob.core.chinacloudapi.cn/example/data/gutenberg/davinci.txt**。

##<a id="job"></a>关于示例 MapReduce

本示例中使用的 MapReduce 作业位于 HDInsight 群集随附的 ****wasb://example/jars/hadoop-mapreduce-examples.jar** 中。其中包含一个你要针对 **davinci.txt** 运行的单词计数示例。

> [AZURE.NOTE]在 HDInsight 2.1 群集上，该文件位置为 ****wasb:///example/jars/hadoop-examples.jar**。

下面提供了单词计数 MapReduce 作业的 Java 代码供你参考：

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

有关编写自己的 MapReduce 作业的说明，请参阅[为 HDInsight 开发 Java MapReduce 程序](/documentation/articles/hdinsight-develop-deploy-java-mapreduce/)。

##<a id="run"></a>运行 MapReduce

HDInsight 可以使用各种方法运行 HiveQL 作业。使用下表来确定哪种方法最适合你，然后按链接进行演练。

| **使用此方法**... | **...实现此目的** | ...使用此**群集操作系统** | ...从此**客户端操作系统** |
|:-------------------------------------------------------------------|:--------------------------------------------------------|:------------------------------------------|:-----------------------------------------|
| [Curl](/documentation/articles/hdinsight-hadoop-use-mapreduce-curl/) | 使用 **REST** 远程提交作业 | Windows | Windows |
| [Windows PowerShell](/documentation/articles/hdinsight-hadoop-use-mapreduce-powershell/) | 使用 **Windows PowerShell** 远程提交作业 | Windows | Windows |
| [远程桌面](/documentation/articles/hdinsight-hadoop-use-mapreduce-remote-desktop/) | 通过**远程桌面**使用 Hadoop 命令 | Windows | Windows |

##<a id="nextsteps"></a>后续步骤

虽然 MapReduce 提供了强大的诊断功能，但掌握起来可能会比较困难。有多个基于 Java 的框架可让你更轻松地定义 MapReduce 应用程序，还有一些技术（例如 Pig 和 Hive）可让你更方便地在 HDInsight 中处理数据。若要了解更多信息，请参阅下列文章：

* [为 HDInsight 开发 Java MapReduce 程序](/documentation/articles/hdinsight-develop-deploy-java-mapreduce/)

* [将 Hive 与 HDInsight 配合使用][hdinsight-use-hive]

* [将 Pig 与 HDInsight 配合使用][hdinsight-use-pig]

* [运行 HDInsight 示例][hdinsight-samples]


[hdinsight-upload-data]: /documentation/articles/hdinsight-upload-data/
[hdinsight-get-started]: /documentation/articles/hdinsight-hadoop-tutorial-get-started-windows-v1/
[hdinsight-develop-mapreduce-jobs]: /documentation/articles/hdinsight-develop-deploy-java-mapreduce/
[hdinsight-develop-streaming]: /documentation/articles/hdinsight-hadoop-develop-deploy-streaming-jobs/
[hdinsight-use-hive]: /documentation/articles/hdinsight-use-hive/
[hdinsight-use-pig]: /documentation/articles/hdinsight-use-pig/
[hdinsight-samples]: /documentation/articles/hdinsight-run-samples/
[hdinsight-provision]: /documentation/articles/hdinsight-provision-clusters-v1/

[powershell-install-configure]: /documentation/articles/powershell-install-configure/

[image-hdi-wordcountdiagram]: ./media/hdinsight-use-mapreduce/HDI.WordCountDiagram.gif

<!---HONumber=Mooncake_1207_2015-->