<properties linkid="manage-services-hdinsight-sample-wordcount" urlDisplayName="HDInsight Samples" pageTitle="The HDInsight WordCount sample | Azure" metaKeywords="hdinsight, hdinsight sample, hadoop, mapreduce" description="Learn how to run a simple MapReduce sample on HDInsight." umbracoNaviHide="0" disqusComments="1" editor="cgronlun" manager="paulettm" services="hdinsight" documentationCenter="" title="The HDInsight WordCount sample" authors="bradsev" />

# HDInsight WordCount 示例

此示例主题介绍如何使用 Azure PowerShell 在 HDInsight 中运行统计文本文件中单词出现次数的 Hadoop MapReduce 程序。该 WordCount MapReduce 程序是用 Java 编写的，在由 HDInsight 管理的 Hadoop 群集上运行。此示例中分析的文本文件是《莱奥纳多.达.芬奇笔记》(The Notebooks of Leonardo Da Vinci) 的 Project Gutenberg 电子书版本。

Hadoop MapReduce 程序读取该文本文件并统计每个单词出现的频率。输出是一个新的文本文件，其中包含若干行，每行包含一个单词以及计数（以制表符分隔的键/值对），而计数表明此单词出现在文档中的频率。此过程分为两个阶段。映射器将输入文本中的每行用作一个输入并将其拆分为多个单词。每当出现单词后跟一个 1 的情况时，映射器将发出一个键值对。随后，化简器会计算每个单词的各个计数的和并发出一个键值对（包含单词，后跟该单词的总出现次数）。

**你将了解到以下内容：**

-   如何使用 Azure PowerShell 在 HDInsight 群集上运行 MapReduce 程序。
-   如何在 Java 中编写 MapReduce 程序。

**先决条件**：

-   你必须具有 Azure 帐户。有关注册帐户的选项，请参阅[免费试用 Azure][] 页。

-   你必须已经设置了 HDInsight 群集。有关可用于创建这种群集的各种不同方法的说明，请参阅 [Azure HDInsight 入门][]或[设置 HDInsight 群集][]。

-   你必须已经安装了 Azure PowerShell，并且已将其配置为可用于你的帐户。有关如何进行此安装的说明，请参阅[安装和配置 Azure PowerShell][]

## 本文内容

本主题说明了如何运行该示例，展示了 MapReduce 程序的 Java 代码，总结了你已经学习到的内容，并概括了一些后续步骤。本文包括以下各节。

1.  [使用 Azure PowerShell 运行示例][]
2.  [WordCount MapReduce 程序的 Java 代码][]
3.  [摘要][]
4.  [后续步骤][]

## 使用 Azure PowerShell 运行示例

**提交 MapReduce 作业**

1.  打开 **Azure PowerShell**。有关打开 Azure PowerShell 控制台窗口的说明，请参阅[安装和配置 Azure PowerShell][]。

2.  在以下命令中设置两个变量，然后运行它们：

        $subscriptionName = "<SubscriptionName>"   # Azure 订阅名称
        $clusterName = "<ClusterName>"             # HDInsight 群集名称

3.  运行以下命令创建 MapReduce 作业定义：

        # 定义 MapReduce 作业
        $wordCountJobDefinition = New-AzureHDInsightMapReduceJobDefinition -JarFile "wasb:///example/jars/hadoop-examples.jar" -ClassName "wordcount" -Arguments "wasb:///example/data/gutenberg/davinci.txt", "wasb:///example/data/WordCountOutput" 

    > [WACOM.NOTE] *hadoop-examples.jar* 随版本 2.1 HDInsight 群集提供。该文件在版本 3.0 HDInsight 群集上已被重命名为 *hadoop-mapreduce.jar*。

    hadoop-examples.jar 文件随 HDInsight 群集提供。此 MapReduce 作业有两个参数。第一个参数是源文件名，第二个参数是输出文件路径。源文件随 HDInsight 群集提供，输出文件路径将会在运行时创建。

4.  运行以下命令来提交 MapReduce 作业：

        # 提交作业
        Select-AzureSubscription $subscriptionName
        $wordCountJob = Start-AzureHDInsightJob -Cluster $clusterName -JobDefinition $wordCountJobDefinition | Wait-AzureHDInsightJob -WaitTimeoutInSeconds 3600  

    除了 MapReduce 作业定义外，你还提供要运行 MapReduce 作业的 HDInsight 群集名称。

5.  运行以下命令来检查运行 MapReduce 作业时的错误：

        # 获取作业输出
        Get-AzureHDInsightJobOutput -Cluster $clusterName -JobId $wordCountJob.JobId -StandardError 

**检索 MapReduce 作业的结果**

1.  打开 **Azure PowerShell**。
2.  在以下命令中设置三个变量，然后运行它们：

        $subscriptionName = "<SubscriptionName>"   # Azure 订阅名称

        $storageAccountName = "<StorageAccountName>"   # Azure 存储帐户名
        $containerName = "<ContainerName>"             # Blob 存储容器名

    Azure 存储帐户是你在本教程前面部分创建的帐户。存储帐户用于承载作为默认 HDInsight 群集文件系统的 Blob 容器。Blob 存储容器通常与 HDInsight 群集共享相同的名称，除非你在设置群集时指定其他名称。

3.  运行以下命令以创建 Azure 存储上下文对象：

        # 选择当前订阅
        Select-AzureSubscription $subscriptionName

        # 创建存储帐户上下文对象
        $storageAccountKey = Get-AzureStorageKey $storageAccountName | %{ $_.Primary }
        $storageContext = New-AzureStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $storageAccountKey  

    *Select-AzureSubscription* 用于在你有多个订阅但默认订阅不是要使用的订阅时设置当前订阅。

4.  运行以下命令，将 MapReduce 作业输出从 Blob 容器下载到工作站：

        # 将作业输出下载到工作站
        Get-AzureStorageBlobContent -Container $ContainerName -Blob example/data/WordCountOutput/part-r-00000 -Context $storageContext -Force

    */example/data/WordCountOutput* 文件夹是你在运行 MapReduce 作业时指定的输出文件夹。*part-r-00000* 是 MapReduce 作业输出的默认文件名。此文件将下载到本地文件夹上的相同文件夹结构。例如，在以下屏幕快照中，当前文件是 C 根文件夹。此文件将下载到 *C:\\example\\data\\WordCountOutput* 文件夹。

5.  运行以下命令来打印 MapReduce 作业输出文件：

        cat ./example/data/WordCountOutput/part-r-00000 | findstr "there"

    MapReduce 作业将生成一个名为 part-r-00000** 的文件，其中包含单词和计数。此脚本使用 findstr 命令来列出包含“there”**的所有单词。

WordCount 脚本的输出应出现在 cmd 窗口中：

![HDI.Sample.WordCount.Output][]

请注意，MapReduce 作业的输出文件是不可变的。所以，如果你重新运行此示例，将需要更改输出文件的名称。

## WordCount MapReduce 程序的 Java 代码

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
    for (IntWritable val :values) {
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
    System.err.println("Usage:wordcount <in> <out>");
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

## 总结

在本主题中，你看到了如何运行一个 MapReduce 程序，该程序使用 Azure PowerShell 在 HDInsight 中计算单词在文本文件中的出现次数。

## 后续步骤

有关运行其他示例的教程，以及提供在 Azure HDInsight 上通过 Azure PowerShell 使用 Pig、Hive 和 MapReduce 作业的说明的教程，请参阅以下主题：

-   [Azure HDInsight 入门][]
-   [示例：10GB GraySort][]
-   [示例：Pi Estimator][]
-   [示例：C\# Steaming][]
-   [Pig 与 HDInsight 配合使用][]
-   [Hive 与 HDInsight 配合使用][]
-   [Azure HDInsight SDK 文档][]

  [免费试用 Azure]: http://www.windowsazure.cn/zh-cn/pricing/free-trial/
  [Azure HDInsight 入门]: /en-us/manage/services/hdinsight/get-started-hdinsight/
  [设置 HDInsight 群集]: /en-us/manage/services/hdinsight/provision-hdinsight-clusters/
  [安装和配置 Azure PowerShell]: /en-us/documentation/articles/install-configure-powershell/
  [使用 Azure PowerShell 运行示例]: #run-sample
  [WordCount MapReduce 程序的 Java 代码]: #java-code
  [摘要]: #summary
  [后续步骤]: #next-steps
  [HDI.Sample.WordCount.Output]: ./media/hdinsight-sample-wordcount/HDI.Sample.WordCount.Output.png
  [示例：10GB GraySort]: /en-us/manage/services/hdinsight/howto-run-samples/sample-10gb-graysort/
  [示例：Pi Estimator]: /en-us/manage/services/hdinsight/howto-run-samples/sample-pi-estimator/
  [示例：C\# Steaming]: /en-us/manage/services/hdinsight/howto-run-samples/sample-csharp-streaming/
  [Pig 与 HDInsight 配合使用]: /en-us/manage/services/hdinsight/using-pig-with-hdinsight/
  [Hive 与 HDInsight 配合使用]: /en-us/manage/services/hdinsight/using-hive-with-hdinsight/
  [Azure HDInsight SDK 文档]: http://msdn.microsoft.com/zh-cn/library/dn469975.aspx
