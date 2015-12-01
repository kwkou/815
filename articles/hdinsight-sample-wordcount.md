<properties
	pageTitle="HDInsight 中的 Hadoop MapReduce 单词计数示例 | Windows Azure"
	description="在 HDInsight 中的 Hadoop 群集上运行 MapReduce 单词计数示例。该程序用 Java 编写，将会统计单词在文本文件中出现的次数。"
	editor="cgronlun"
	manager="paulettm"
	services="hdinsight"
	documentationCenter=""
	tags="azure-portal"
	authors="mumian"/>

<tags
	ms.service="hdinsight"
	ms.date="10/15/2015"
	wacn.date="11/27/2015"/>

#在 HDInsight 中的 Hadoop 群集上运行 MapReduce 单词计数程序

了解如何使用 Azure PowerShell 在 HDInsight 中的 Hadoop 群集上运行 MapReduce 程序。该程序以 Java 编写，它将统计单词在文本文件中出现的次数，然后输出一个新的文本文件，其中包含每个单词及其出现的频率。

该程序安装在群集上。本教程中分析的文本文件是《莱奥纳多·达·芬奇笔记》(The Notebooks of Leonardo Da Vinci) 的 Project Gutenberg 电子书版本。

**其他相关文章：**

* [Azure HDInsight 入门][hdinsight-get-started]
* [为 HDInsight 中的 Hadoop 开发 Java MapReduce 程序](/documentation/articles/hdinsight-develop-deploy-java-mapreduce)
* [在 HDInsight 中提交 Hadoop 作业](/documentation/articles/hdinsight-submit-hadoop-jobs-programmatically)
* [示例：10GB GraySort][hdinsight-sample-10gb-graysort]
* [示例：Pi Estimator][hdinsight-sample-pi-estimator]
* [示例：C# 流式处理][hdinsight-sample-cs-streaming]

**先决条件**：

- **一个 HDInsight 群集**。有关可用于创建这种群集的各种不同方法的说明，请参阅 [Azure HDInsight 入门][hdinsight-get-started]或[预配 HDInsight 群集](/documentation/articles/hdinsight-provision-clusters)。
- **配备 Azure PowerShell 的工作站**。请参阅[安装和使用 Azure PowerShell](/documentation/articles/install-configure-powershell)。

<a id="run-sample"></a>
## 使用 Azure PowerShell 运行示例

**提交 MapReduce 作业**

1. 打开 **Windows PowerShell ISE**。有关说明，请参阅[安装和配置 Azure PowerShell][powershell-install-configure]。
2. 粘贴以下 PowerShell 脚本：

		$subscriptionName = "<Azure Subscription Name>"
		$resourceGroupName = "<Resource Group Name>"
		$clusterName = "<HDInsight cluster name>"             # HDInsight cluster name
		
		Select-AzureRmSubscription $subscriptionName
		
		# Define the MapReduce job
		$wordCountJobDefinition = New-AzureRmHDInsightMapReduceJobDefinition `
									-JarFile "wasb:///example/jars/hadoop-mapreduce-examples.jar" `
									-ClassName "wordcount" `
									-Arguments "wasb:///example/data/gutenberg/davinci.txt", "wasb:///example/data/WordCountOutput1"
		
		# Submit the job and wait for job completion
		$cred = Get-Credential -Message "Enter the HDInsight cluster HTTP user credential:" 
		$wordCountJob = Start-AzureRmHDInsightJob `
							-ResourceGroupName $resourceGroupName `
							-ClusterName $clusterName `
							-HttpCredential $cred `
							-JobDefinition $wordCountJobDefinition 
		
		Wait-AzureRmHDInsightJob `
			-ResourceGroupName $resourceGroupName `
			-ClusterName $clusterName `
			-HttpCredential $cred `
			-JobId $wordCountJob.JobId 
		
		# Get the job output
		$cluster = Get-AzureRmHDInsightCluster -ResourceGroupName $resourceGroupName -ClusterName $clusterName
		$defaultStorageAccount = $cluster.DefaultStorageAccount -replace '.blob.core.chinacloudapi.cn'
		$defaultStorageAccountKey = Get-AzureRmStorageAccountKey -ResourceGroupName $resourceGroupName -Name $defaultStorageAccount |  %{ $_.Key1 }
		$defaultStorageContainer = $cluster.DefaultStorageContainer
		
		Get-AzureRmHDInsightJobOutput `
			-ResourceGroupName $resourceGroupName `
			-ClusterName $clusterName `
			-HttpCredential $cred `
			-DefaultStorageAccountName $defaultStorageAccount `
			-DefaultStorageAccountKey $defaultStorageAccountKey `
			-DefaultContainer $defaultStorageContainer  `
			-JobId $wordCountJob.JobId `
			-DisplayOutputType StandardError

3. 设置前 3 个变量，然后运行脚本。
		
**检索 MapReduce 作业的结果**

1. 打开 **Windows PowerShell ISE**。有关说明，请参阅[安装和配置 Azure PowerShell][powershell-install-configure]。
2. 粘贴以下 PowerShell 脚本：

		$subscriptionName = "<Azure Subscription Name>"
		$resourceGroupName = "<Resource Group Name>"
		$clusterName = "<HDInsight cluster name>"             # HDInsight cluster name

		# Select the current subscription
		Select-AzureSubscription $subscriptionName
		
		# Get the cluster properties
		$cluster = Get-AzureRmHDInsightCluster -ResourceGroupName $resourceGroupName -ClusterName $clusterName
		$defaultStorageAccount = $cluster.DefaultStorageAccount -replace '.blob.core.chinacloudapi.cn'
		$defaultStorageAccountKey = Get-AzureRmStorageAccountKey -ResourceGroupName $resourceGroupName -Name $defaultStorageAccount |  %{ $_.Key1 }
		$defaultStorageContainer = $cluster.DefaultStorageContainer
		
		# Download the job output to the workstation
		$storageContext = New-AzureStorageContext -StorageAccountName $defaultStorageAccount -StorageAccountKey $defaultStorageAccountKey 
		Get-AzureStorageBlobContent -Container $defaultStorageContainer -Blob example/data/WordCountOutput/part-r-00000 -Context $storageContext -Force
		
		# Display the output file
		cat ./example/data/WordCountOutput/part-r-00000 | findstr "there"

	MapReduce 作业将生成一个名为 *part-r-00000* 的文件，其中包含单词和计数。该脚本使用 **findstr** 命令来列出包含*“there”*的所有单词。

WordCount 脚本的输出应出现在命令窗口中：

![在 HDInsight 中运行 Hadoop MapReduce 单词计数示例后，在 PowerShell 中生成的单词频率结果。][image-hdi-sample-wordcount-output]

请注意，MapReduce 作业的输出文件是不可变的。因此，如果你重新运行此示例，将需要更改输出文件的名称。

<a id="java-code"></a>
##Java 源代码

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

<a id="next-steps">
## 后续步骤

* [Azure HDInsight 入门][hdinsight-get-started]
* [为 HDInsight 中的 Hadoop 开发 Java MapReduce 程序](/documentation/articles/hdinsight-develop-deploy-java-mapreduce)
* [在 HDInsight 中提交 Hadoop 作业](/documentation/articles/hdinsight-submit-hadoop-jobs-programmatically)
* [示例：10GB GraySort][hdinsight-sample-10gb-graysort]
* [示例：Pi Estimator][hdinsight-sample-pi-estimator]
* [示例：C# 流式处理][hdinsight-sample-cs-streaming]

[hdinsight-sample-10gb-graysort]: /documentation/articles/hdinsight-sample-10gb-graysort
[hdinsight-sample-pi-estimator]: /documentation/articles/hdinsight-sample-pi-estimator
[hdinsight-sample-cs-streaming]: /documentation/articles/hdinsight-sample-csharp-streaming


[hdinsight-use-hive]: /documentation/articles/hdinsight-use-hive
[hdinsight-use-pig]: /documentation/articles/hdinsight-use-pig

[hdinsight-get-started]: /documentation/articles/hdinsight-get-started

[powershell-install-configure]: /documentation/articles/install-configure-powershell

[image-hdi-sample-wordcount-output]: ./media/hdinsight-sample-wordcount/HDI.Sample.WordCount.Output.png

<!---HONumber=82-->