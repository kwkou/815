<properties
	pageTitle="C# 流式处理单词计数 Hadoop 示例 | Azure"
	description="如何用 C# 编写使用 Hadoop 流式处理接口的 MapReduce 程序，以及如何使用 PowerShell cmdlet 运行这些程序。"
	editor="cgronlun"
	manager="paulettm"
	services="hdinsight"
	documentationCenter=""
	authors="bradsev"/>

<tags
	ms.service="hdinsight"
	ms.date="07/09/2015"
	wacn.date="10/03/2015"/>
 
# HDInsight 上 Hadoop 中的 C# 流式处理单词计数 Hadoop 示例
 
Hadoop 向 MapReduce 提供了一个流式处理 API，利用它，你可以采用 Java 之外的其他语言来编写映射函数和化简函数。本教程介绍如何使用 Hadoop 流式处理接口以 C# 编写 MapReduce 程序，以及如何使用 Azure PowerShell cmdlet 在 Azure HDInsight 中运行这些程序。

在示例中，映射器和化简器都是可执行的，它们从 [stdin][stdin-stdout-stderr] 读取输入（逐行）并将输出结果发送到 [stdout][stdin-stdout-stderr]。程序计算文本中所有单词的数量。

如果为**映射器**指定可执行文件，则当初始化映射器时，每个映射器任务都将启动此可执行文件作为一个单独的进程。当映射器任务运行时，它将其输入转换为行，并将这些行馈送到进程的 [stdin][stdin-stdout-stderr]。

同时，映射器从进程的 stdout 中收集面向行的输出。然后将每行转换为一个键/值对（作为映射器的输出收集）。默认情况下，一行的前缀直至第一个制表符是键，而该行的剩余部分（不包括制表符）是值。如果行中没有制表符，则整行被视为键，而值为 Null。

如果为**化简器**指定可执行文件，则当初始化化简器时，每个化简器任务都将启动此可执行文件作为一个单独的进程。当化简器任务运行时，它将其输入键/值对转换为行，并将这些行馈送到进程的 [stdin][stdin-stdout-stderr]。

同时，化简器从进程的 [stdout][stdin-stdout-stderr] 中收集面向行的输出。然后将每行转换为一个键/值对（作为化简器的输出收集）。默认情况下，一行的前缀直至第一个制表符是键，而该行的剩余部分（不包括制表符）是值。

有关 Hadoop 流接口的详细信息，请参阅 [Hadoop 流][hadoop-streaming]。
 
**在本教程中，你将了解如何：**
	
* 使用 Azure PowerShell 在 HDInsight 中运行 C# 流式处理程序以分析文件中包含的数据。
* 编写使用 Hadoop 流式处理接口的 C# 代码。


**先决条件**：

在开始之前，你必须满足以下条件：

- **一个 Azure 订阅**。请参阅 [azure-trial](/pricing/1rmb-trial/) 页。

- **一个 HDInsight 群集**。有关可用于创建这类群集的不同方法的说明，请参阅[预配 HDInsight 群集](/documentation/articles/hdinsight-provision-clusters)。

- **配备 Azure PowerShell 的工作站**。请参阅[安装和配置 Azure PowerShell][powershell-install-configure]。


## <a id="run-sample"></a>使用 Azure PowerShell 运行示例

**运行 MapReduce 作业**

1.	打开 **Azure PowerShell**。有关如何打开 Azure PowerShell 控制台窗口的说明，请参阅[安装和配置 Azure PowerShell][powershell-install-configure]。

3. 在以下命令中设置两个变量，然后运行它们：
		
		$subscriptionName = "<SubscriptionName>"   # Azure subscription name
		$clusterName = "<ClusterName>"             # HDInsight cluster name


2. 运行以下命令来定义 MapReduce 作业：
 
		# Create a MapReduce job definition for the streaming job.
		$streamingWC = New-AzureHDInsightStreamingMapReduceJobDefinition -Files "/example/apps/wc.exe", "/example/apps/cat.exe" -InputPath "/example/data/gutenberg/davinci.txt" -OutputPath "/example/data/StreamingOutput/wc.txt" -Mapper "cat.exe" -Reducer "wc.exe" 

	这些参数指定映射器和化简器函数以及输入文件和输出文件。
                 
5. 运行以下命令以运行 MapReduce 作业，等待作业完成，然后打印标准错误消息：

		# Run the C# Streaming MapReduce job.
		# Wait for the job to complete.
		# Print output and standard error file of the MapReduce job
		Select-AzureSubscription $subscriptionName
		$streamingWC | Start-AzureHDInsightJob -Cluster $clustername | Wait-AzureHDInsightJob -WaitTimeoutInSeconds 3600 | Get-AzureHDInsightJobOutput -Cluster $clustername -StandardError 

6. 运行下列命令以显示单词计数的结果：

		$subscriptionName = "<SubscriptionName>"   
		$storageAccountName = "<StorageAccountName>" 
		$containerName = "<ContainerName>"

		# Select the current subscription
		Select-AzureSubscription $subscriptionName
              
		# Blob storage container and account name
      $storageAccountKey = Get-AzureStorageKey -StorageAccountName $storageAccountName | %{ $_.Primary } $storageContext = New-AzureStorageContext -Environment AzureChinaCloud -StorageAccountName $storageAccountName -StorageAccountKey $storageAccountKey
 
		# Retrieve the output
		Get-AzureStorageBlobContent -Container $containerName -Blob "example/data/StreamingOutput/wc.txt/part-00000" -Context $storageContext -Force 

		# The number of words in the text is:
		cat ./example/data/StreamingOutput/wc.txt/part-00000

	请注意，MapReduce 作业的输出文件是不可变的。因此，如果你重新运行此示例，将需要更改输出文件的名称。


## <a id="java-code"></a>Hadoop 流式处理的 C# 代码

MapReduce 程序使用 cat.exe 应用程序作为映射接口将文本流式传输到控制台，并使用 wc.exe 应用程序作为化简接口来统计从文档中流式传输的字数。映射器和化简器都从标准输入流 (stdin) 逐行读取字符，并写入到标准输出流 (stdout)。



	// The source code for the cat.exe (Mapper). 
	 
	using System;
	using System.IO;
	
	namespace cat
	{
	    class cat
	    {
	        static void Main(string[] args)
	        {
	            if (args.Length > 0)
	            {
	                Console.SetIn(new StreamReader(args[0])); 
	            }
	
	            string line;
	            while ((line = Console.ReadLine()) != null) 
	            {
	                Console.WriteLine(line);
	            }
	        }
	    }
	}

 

cat.cs 文件中的映射器代码使用 [StreamReader][streamreader] 对象将传入流的字符读入到控制台，然后控制台使用静态 [Console.Writeline][console-writeline] 方法将流写入标准输出流。


	// The source code for wc.exe (Reducer) is:
	
	using System;
	using System.IO;
	using System.Linq;
	
	namespace wc
	{
	    class wc
	    {
	        static void Main(string[] args)
	        {
	            string line;
	            var count = 0;
	
	            if (args.Length > 0){
	                Console.SetIn(new StreamReader(args[0]));
	            }
	
	            while ((line = Console.ReadLine()) != null) {
	                count += line.Count(cr => (cr == ' ' || cr == '\n'));
	            }
	            Console.WriteLine(count);
	        }
	    }
	}


wc.cs 文件中的化简器代码使用 [StreamReader][streamreader] 对象从 cat.exe 映射器输出的标准输入流读取字符。当它使用 [Console.Writeline][console-writeline] 方法读取字符时，它将通过统计位于每个单词末尾的空格和行结束字符的数目来计算单词数量。然后使用 [Console.Writeline][console-writeline] 方法将总数写入标准输出流中。


## <a id="summary"></a>摘要

在本教程中，你已学习如何使用 Hadoop 流式处理在 HDInsight 中部署 MapReduce 作业。

## <a id="next-steps"></a>后续步骤


有关运行其他示例的教程，以及提供在 Azure HDInsight 中通过 Azure PowerShell 运行 Pig、Hive 和 MapReduce 作业的说明的教程，请参阅以下文章：

* [Azure HDInsight 入门][hdinsight-get-started]
* [示例：Pi Estimator][hdinsight-sample-pi-estimator]
* [示例：Wordcount][hdinsight-sample-wordcount]
* [示例：10GB GraySort][hdinsight-sample-10gb-graysort]
* [将 Pig 与 HDInsight 配合使用][hdinsight-use-pig]
* [将 Hive 与 HDInsight 配合使用][hdinsight-use-hive]
* [Azure HDInsight SDK 文档][hdinsight-sdk-documentation]

[hdinsight-sdk-documentation]: https://msdn.microsoft.com/zh-cn/library/dn479185.aspx

[hadoop-streaming]: http://wiki.apache.org/hadoop/HadoopStreaming
[streamreader]: http://msdn.microsoft.com/zh-cn/library/system.io.streamreader.aspx
[console-writeline]: http://msdn.microsoft.com/zh-cn/library/system.console.writeline
[stdin-stdout-stderr]: http://msdn.microsoft.com/zh-cn/library/3x292kth(v=vs.110).aspx

[Powershell-install-configure]: /documentation/articles/install-configure-powershell/

[hdinsight-get-started]: /documentation/articles/hdinsight-get-started/

[hdinsight-samples]: /documentation/articles/hdinsight-run-samples/
[hdinsight-sample-10gb-graysort]: /documentation/articles/hdinsight-sample-10gb-graysort/
[hdinsight-sample-csharp-streaming]: /documentation/articles/hdinsight-sample-csharp-streaming/
[hdinsight-sample-pi-estimator]: /documentation/articles/hdinsight-sample-pi-estimator/
[hdinsight-sample-wordcount]: /documentation/articles/hdinsight-sample-wordcount/

[hdinsight-use-hive]: /documentation/articles/hdinsight-use-hive/
[hdinsight-use-pig]: /documentation/articles/hdinsight-use-pig/

<!---HONumber=71-->