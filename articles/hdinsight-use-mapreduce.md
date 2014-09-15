<properties linkid="manage-services-hdinsight-howto-mapreduce" urlDisplayName="MapReduce with HDInsight " pageTitle="Use MapReduce with HDInsight | Azure" metaKeywords="" description="Learn how to use HDInsight to execute a simple Hadoop MapReduce job." metaCanonical="" services="hdinsight" documentationCenter="" title="Use MapReduce with HDInsight" authors="jgao" solutions="" manager="paulettm" editor="cgronlun" />

# MapReduce 与 HDInsight 配合使用

Hadoop MapReduce 是一个软件框架，用于编写处理海量数据的应用程序。在本教程中，你将从工作站上使用 Azure PowerShell 来提交一个 MapReduce 程序，该程序将文本中的单词出现次数统计到 HDInsight 群集。该单词统计程序用 Java 和随 HDInsight 群集提供的程序编写。

**先决条件：**

在开始阅读本教程前，你必须具有：

-   HDInsight 群集。有关可用于创建这种群集的各种不同方法的说明，请参阅[设置 HDInsight 群集](/en-us/manage/services/hdinsight/provision-hdinsight-clusters/)。

-   已安装并已配置 Azure PowerShell 的工作站。有关说明，请参阅[安装和配置 Azure PowerShell][powershell-install-configure]。

**估计完成时间：** 30 分钟

## 在本教程中

1.  [了解方案](#scenario)
2.  [使用 Azure PowerShell 运行示例](#run-sample)
3.  [字数统计 MapReduce 程序的 Java 代码](#java-code)
4.  [后续步骤](#next-steps)

## 了解方案

下图解释了 MapReduce 怎样用于字数统计方案：

![HDI.WordCountDiagram][image-hdi-wordcountdiagram]

MapReduce 作业的输出结果是一组键值对。键是一个用于指定单词的字符串，值是一个用于指定单词在文本中的总出现次数的整数。这是通过两个阶段完成的：

-   映射器将输入文本中的每行用作一个输入并将其拆分为多个单词。它在每次出现后面跟有 1 的单词时发出一个键值对。输出结果在发送到化简器之前会先存储起来。

-   随后，化简器会计算每个单词的计数的和并发出一个键值对（包含单词，后跟该单词的总出现次数）。

运行 MapReduce 作业要求以下元素：

-   MapReduce 程序。在本教程中，你将使用 HDInsight 群集附带的字数统计示例，因此，你无需编写自己的 MapReduce 程序。该示例位于 */example/jars/hadoop-examples.jar*。版本 3.0 的 HDInsight 群集上的文件名为 *hadoop-mapreduce-examples.jar*。有关编写自己的 MapReduce 作业的说明，请参阅[为 HDInsight 开发 Java MapReduce 程序][hdinsight-develop-MapReduce]。
-   一个输入文件。你将使用 */example/data/gutenberg/davinci.txt* 作为输入文件。有关上传文件的信息，请参见[将数据上传到 HDInsight][hdinsight-upload-data]。
-   输出文件文件夹。你将使用 */example/data/WordCountOutput* 作为输出文件文件夹。如果该文件夹不存在，系统将会创建。如果该文件夹存在，MapReduce 作业将会失败。如果你想第二次运行 MapReduce 作业，一定要删除该输出文件夹或指定另一输出文件夹。

## 使用 Azure PowerShell 运行示例

1.  打开 **Azure PowerShell**。有关打开 Azure PowerShell 控制台窗口的说明，请参阅[安装和配置 Azure PowerShell][powershell-install-configure]。

2.  在以下命令中设置两个变量，然后运行它们：

        $subscriptionName = "<SubscriptionName>"   # Azure 订阅名称
        $clusterName = "<ClusterName>"             # HDInsight 群集名称

3.  运行以下命令并提供你的 Azure 帐户信息：

        Add-AzureAccount

4.  运行以下命令创建 MapReduce 作业定义：

        # 定义 MapReduce 作业
        $wordCountJobDefinition = New-AzureHDInsightMapReduceJobDefinition -JarFile "wasb:///example/jars/hadoop-examples.jar" -ClassName "wordcount" -Arguments "wasb:///example/data/gutenberg/davinci.txt", "wasb:///example/data/WordCountOutput" 

    > [WACN.NOTE] *hadoop-examples.jar* 随版本 2.1 HDInsight 群集提供。该文件在版本 3.0 HDInsight 群集上已被重命名为 *hadoop-mapreduce.jar*。

    hadoop-examples.jar 文件随 HDInsight 群集分发提供。此 MapReduce 作业有两个参数。第一个参数是源文件名，第二个参数是输出文件路径。源文件随 HDInsight 群集分发提供，输出文件路径将会在运行时创建。

5.  运行以下命令来提交 MapReduce 作业：

        # 提交作业
        Select-AzureSubscription $subscriptionName
        $wordCountJob = Start-AzureHDInsightJob -Cluster $clusterName -JobDefinition $wordCountJobDefinition | Wait-AzureHDInsightJob -WaitTimeoutInSeconds 3600  

    除了 MapReduce 作业定义外，你还要提供需运行 MapReduce 作业的 HDInsight 群集名称，以及凭据。Start-AzureHDInsightJob 是异步调用。若要检查作业是否完成，请使用 *Wait-AzureHDInsightJob* cmdlet。

6.  运行以下命令来检查 MapReduce 作业的完成：

        Wait-AzureHDInsightJob -Job $wordCountJob -WaitTimeoutInSeconds 3600 

7.  运行以下命令来检查运行 MapReduce 作业时的错误：

        # 获取作业输出
        Get-AzureHDInsightJobOutput -Cluster $clusterName -JobId $wordCountJob.JobId -StandardError 

**检索 MapReduce 作业的结果**

1.  打开 **Azure PowerShell**。
2.  运行以下命令将目录更改到 c:\\ 根目录：

        cd \

    默认 Azure Powershell 目录是 *C:\\Windows\\System32\\WindowsPowerShell\\v1.0*。默认情况下，你对此文件夹没有写入权限。你必须将目录更改到 C:\\ 根目录或你有写权限的文件夹。

3.  在以下命令中设置三个变量，然后运行它们：

        $subscriptionName = "<SubscriptionName>"   # Azure 订阅名称

        $storageAccountName = "<StorageAccountName>"   # Azure 存储帐户名
        $containerName = "<ContainerName>"             # Blob 存储容器名

        Azure 存储帐户是你在本教程前面部分创建的帐户。存储帐户用于承载作为默认 HDInsight 群集文件系统的 Blob 容器。Blob 存储容器名称通常与 HDInsight 群集共享相同的名称，除非在你设置群集时指定其他名称。

4.  运行以下命令以创建 Azure 存储上下文对象：

        # 选择当前订阅
        Select-AzureSubscription $subscriptionName

        # 创建存储帐户上下文对象
        $storageAccountKey = Get-AzureStorageKey $storageAccountName | %{ $_.Primary }
        $storageContext = New-AzureStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $storageAccountKey  

    *Select-AzureSubscription* 用于在你有多个订阅但默认订阅不是要使用的订阅时设置当前订阅。

5.  运行以下命令，将 MapReduce 作业输出从 Blob 容器下载到工作站：

        # 将作业输出下载到工作站
        Get-AzureStorageBlobContent -Container $ContainerName -Blob example/data/WordCountOutput/part-r-00000 -Context $storageContext -Force

    */example/data/WordCountOutput* 文件夹是你在运行 MapReduce 作业时指定的输出文件夹。*part-r-00000* 是 MapReduce 作业输出的默认文件名。此文件将下载到本地文件夹上的相同文件夹结构。例如，在以下屏幕快照中，当前文件是 C 根文件夹。此文件将下载到 \*C:\\example\\data\\WordCountOutput\* 文件夹。

6.  运行以下命令来打印 MapReduce 作业输出文件：

        cat ./example/data/WordCountOutput/part-r-00000 | findstr "there"

    MapReduce 作业将生成一个名为 part-r-00000** 的文件，其中包含单词和计数。此脚本使用 findstr 命令来列出包含“there”**的所有单词。

请注意，MapReduce 作业的输出文件是不可变的。所以，如果你重新运行此示例，将需要更改输出文件的名称。

## 字数统计 MapReduce 程序的 Java 代码

下面是字数统计 Java MapReduce 程序的源代码：

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

## 后续步骤

虽然 MapReduce 提供了强大的诊断功能，但掌握起来可能会比较困难。利用其他语言（如 Pig 和 Hive），可以更轻松地处理存储在 HDInsight 中的数据。若要了解更多信息，请参阅下列文章：

-   [Azure HDInsight 入门][hdinsight-getting-started]
-   [为 HDInsight 开发 Java MapReduce 程序][hdinsight-develop-MapReduce]
-   [为 HDInsight 开发 C# Hadoop 流 MapReduce 程序][hdinsight-develop-streaming]
-   [Hive 与 HDInsight 配合使用][hdinsight-hive]
-   [Pig 与 HDInsight 配合使用][hdinsight-pig]
-   [运行 HDInsight 示例][hdinsight-samples]

[hdinsight-upload-data]: /en-us/documentation/articles/hdinsight-upload-data/

[hdinsight-getting-started]: /en-us/manage/services/hdinsight/get-started-hdinsight/
[hdinsight-develop-mapreduce]: /en-us/documentation/articles/hdinsight-develop-deploy-java-mapreduce/
[hdinsight-develop-streaming]: /en-us/documentation/articles/hdinsight-hadoop-develop-deploy-streaming-jobs/
[hdinsight-hive]: /en-us/documentation/articles/hdinsight-use-hive/
[hdinsight-pig]: /en-us/documentation/articles/hdinsight-use-pig/
[hdinsight-samples]: /en-us/documentation/articles/hdinsight-run-samples/

[powershell-install-configure]: /en-us/manage/install-and-configure-windows-powershell/

[image-hdi-wordcountdiagram]: ./media/hdinsight-get-started/HDI.WordCountDiagram.gif
