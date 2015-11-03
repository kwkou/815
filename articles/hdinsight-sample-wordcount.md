<properties
	pageTitle="在 HDInsight 中运行 Hadoop MapReduce 单词计数示例 | Azure"
	description="在 HDInsight 中的 Hadoop 群集上运行 MapReduce 单词计数示例。该程序用 Java 编写，将会统计单词在文本文件中出现的次数。"
	editor="cgronlun"
	manager="paulettm"
	services="hdinsight"
	documentationCenter=""
	authors="bradsev"/>

<tags
	ms.service="hdinsight"
	ms.date="07/09/2015"
	wacn.date="10/03/2015"/>

#在 HDInsight 中的 Hadoop 群集上运行以 Java 编写的 MapReduce 单词计数示例

本教程说明如何在 HDInsight 中的 Hadoop 群集上运行 MapReduce 单词计数示例。该程序是用 Java 编写的。它将统计单词在文本文件中出现的次数，然后输出一个新的文本文件，其中包含每个单词及其出现的频率。此示例中分析的文本文件是《莱奥纳多.达.芬奇笔记》(The Notebooks of Leonardo Da Vinci) 的 Project Gutenberg 电子书版本。

**你将学习以下内容：**
		
* 如何使用 Azure PowerShell 在 HDInsight 群集上运行 MapReduce 程序。
* 如何在 Java 中编写 MapReduce 程序。


**先决条件**：

- **一个 Azure 订阅**。请参阅 [azure-trial](/pricing/1rmb-trial/) 页。

- **一个 HDInsight 群集**。有关可用于创建这种群集的各种不同方法的说明，请参阅 [Azure HDInsight 入门][hdinsight-get-started]或[预配 HDInsight 群集](/documentation/articles/hdinsight-provision-clusters)。

- **配备 Azure PowerShell 的工作站**。请参阅[安装和配置 Azure PowerShell][powershell-install-configure]。

## <a id="run-sample"></a>使用 Azure PowerShell 运行示例</h2>

**提交 MapReduce 作业**

1.	打开 **Azure PowerShell** 控制台。有关说明，请参阅[安装和配置 Azure PowerShell][powershell-install-configure]。

3. 在以下命令中设置两个变量，然后运行它们：
		
		$subscriptionName = "<SubscriptionName>"   # Azure subscription name
		$clusterName = "<ClusterName>"             # HDInsight cluster name
		
5. 运行以下命令创建 MapReduce 作业定义：

		# Define the MapReduce job
		$wordCountJobDefinition = New-AzureHDInsightMapReduceJobDefinition -JarFile "wasb:///example/jars/hadoop-mapreduce-examples.jar" -ClassName "wordcount" -Arguments "wasb:///example/data/gutenberg/davinci.txt", "wasb:///example/data/WordCountOutput"

	> [AZURE.NOTE]HDInsight 版本 2.1 群集随附了 *hadoop-examples.jar*。在 HDInsight 版本 3.0 群集上，该文件已重命名为 *hadoop-mapreduce.jar*。
	
	HDInsight 群集随附了 hadoop-mapreduce-examples.jar 文件。此 MapReduce 作业有两个参数。第一个参数是源文件名，第二个参数是输出文件路径。源文件随 HDInsight 群集提供，输出文件路径将会在运行时创建。

6. 运行以下命令来提交 MapReduce 作业：

		# Submit the job
		Select-AzureSubscription $subscriptionName
		$wordCountJob = Start-AzureHDInsightJob -Cluster $clusterName -JobDefinition $wordCountJobDefinition | Wait-AzureHDInsightJob -WaitTimeoutInSeconds 3600  

	除了 MapReduce 作业定义外，你还提供要运行 MapReduce 作业的 HDInsight 群集名称。

8. 运行以下命令来检查运行 MapReduce 作业时的错误：
	
		# Get the job output
		Get-AzureHDInsightJobOutput -Cluster $clusterName -JobId $wordCountJob.JobId -StandardError 
		
**检索 MapReduce 作业的结果**

1. 打开 **Azure PowerShell** 控制台。
2. 在以下命令中设置三个变量，然后运行它们：

		$subscriptionName = "<SubscriptionName>"       # Azure subscription name
		$storageAccountName = "<StorageAccountName>"   # Azure storage account name
		$containerName = "<ContainerName>"			   # Blob storage container name

	Azure 存储帐户是你在本教程前面部分创建的帐户。存储帐户用于承载作为默认 HDInsight 群集文件系统的 Blob。Azure Blob 存储容器通常与 HDInsight 群集共享相同的名称，除非你在设置群集时指定其他名称。

3. 运行以下命令以创建 Azure 存储上下文对象：
		
		# Select the current subscription
		Select-AzureSubscription $subscriptionName

		# Create the storage account context object
		$storageAccountKey = Get-AzureStorageKey $storageAccountName | %{ $_.Primary }
		$storageContext = New-AzureStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $storageAccountKey  

	**Select-AzureSubscription** 用于在你具有多个订阅但默认订阅不是要使用的订阅时设置当前订阅。

4. 运行以下命令，将 MapReduce 作业输出从 Blob 下载到工作站：

		# Download the job output to the workstation
		Get-AzureStorageBlobContent -Container $ContainerName -Blob example/data/WordCountOutput/part-r-00000 -Context $storageContext -Force

	*example/data/WordCountOutput* 文件夹是你在运行 MapReduce 作业时指定的输出文件夹。*part-r-00000* 是 MapReduce 作业输出的默认文件名。此文件将下载到本地文件夹中的相同文件夹结构。例如，在以下屏幕快照中，当前文件是 C 根文件夹。此文件将下载到 *C:\\example\\data\\WordCountOutput* 文件夹。

5. 运行以下命令来打印 MapReduce 作业输出文件：

		cat ./example/data/WordCountOutput/part-r-00000 | findstr "there"


	MapReduce 作业将生成一个名为 *part-r-00000* 的文件，其中包含单词和计数。该脚本使用 **findstr** 命令来列出包含*“there”*的所有单词。

WordCount 脚本的输出应出现在命令窗口中：

![在 HDInsight 中运行 Hadoop MapReduce 单词计数示例后，在 PowerShell 中生成的单词频率结果。][image-hdi-sample-wordcount-output]

请注意，MapReduce 作业的输出文件是不可变的。因此，如果你重新运行此示例，将需要更改输出文件的名称。

## <a id="java-code"></a>WordCount MapReduce 程序的 Java 代码</h2>



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



在本主题中，你看到了如何运行一个 MapReduce 程序，该程序使用 Azure PowerShell 在 HDInsight 中计算单词在文本文件中的出现次数。

## <a id="next-steps"></a>后续步骤</h2>

有关运行其他示例的教程，以及提供在 Azure HDInsight 中通过 Azure PowerShell 使用 Pig、Hive 和 MapReduce 作业的说明的教程，请参阅：

* [Azure HDInsight 入门][hdinsight-get-started]
* [示例：10GB GraySort][hdinsight-sample-10gb-graysort]
* [示例：Pi Estimator][hdinsight-sample-pi-estimator]
* [示例：C# 流式处理][hdinsight-sample-cs-streaming]
* [将 Pig 与 HDInsight 配合使用][hdinsight-use-pig]
* [将 Hive 与 HDInsight 配合使用][hdinsight-use-hive]
* [Azure HDInsight SDK 文档][hdinsight-sdk-documentation]

[hdinsight-sdk-documentation]: https://msdn.microsoft.com/zh-cn/library/dn479185.aspx

[hdinsight-sample-10gb-graysort]: /documentation/articles/hdinsight-sample-10gb-graysort/
[hdinsight-sample-pi-estimator]: /documentation/articles/hdinsight-sample-pi-estimator/
[hdinsight-sample-cs-streaming]: /documentation/articles/hdinsight-sample-csharp-streaming/


[hdinsight-use-hive]: /documentation/articles/hdinsight-use-hive/
[hdinsight-use-pig]: /documentation/articles/hdinsight-use-pig/
 
[hdinsight-get-started]: /documentation/articles/hdinsight-get-started/

[Powershell-install-configure]: /documentation/articles/install-configure-powershell/

[image-hdi-sample-wordcount-output]: ./media/hdinsight-sample-wordcount/HDI.Sample.WordCount.Output.png

<!---HONumber=71-->