<properties
   pageTitle="将 MapReduce 与 HDinsight 中的 Hadoop 配合使用 | Azure"
   description="了解如何使用 PowerShell 在 HDInsight 上的 Hadoop 上远程运行 MapReduce 作业。"
   services="hdinsight"
   documentationCenter=""
   authors="Blackmist"
   manager="paulettm"
   editor="cgronlun"/>
<tags ms.service="hdinsight"
    ms.date="02/18/2015"
    wacn.date="04/15/2015"
    />

# 使用 PowerShell 对 HDInsight 上的 Hadoop 运行 Hive 查询

[AZURE.INCLUDE [mapreduce-selector](../includes/hdinsight-selector-use-mapreduce.md)]

本文档提供使用 PowerShell 在 HDInsight 上的 Hadoop 群集中运行 MapReduce 作业的示例。

## <a id="prereq"></a>先决条件

若要完成本文中的步骤，你将需要：

* Azure HDInsight（HDInsight 上的 Hadoop）群集（基于 Windows 或 Linux）

* <a href="/documentation/articles/install-configure-powershell/" target="_blank">Azure PowerShell</a>

## <a id="powershell"></a>使用 PowerShell 运行 MapReduce 作业

Azure PowerShell 提供  *cmdlet*，可让你在 HDInsight 上远程运行 MapReduce 作业。从内部来讲，完成该操作的方法是使用 REST 调用 HDInsight 群集上运行的 <a href="https://cwiki.apache.org/confluence/display/Hive/WebHCat" target="_blank">WebHCat</a>（前称 Templeton）。

在远程 HDInsight 群集上运行 MapReduce 作业时，将使用以下 Cmdlet。

* **Add-AzureAccount** - 在 Azure 订阅中进行 PowerShell 身份验证

* **New-AzureHDInsightMapReduceJobDefinition** - 使用指定的 MapReduce 信息创建新的*作业定义*

* **Start-AzureHDInsightJob** - 将作业定义发送到 HDInsight、启动作业，并返回可用来检查作业状态的*作业*对象

* **Wait-AzureHDInsightJob** - 使用作业对象来检查作业的状态。它将等到作业完成，或已超过等待时间

* **Get-AzureHDInsightJobOutput** - 用于检索作业的输出

以下步骤演示了如何使用这些 Cmdlet 在 HDInsight 群集上运行作业。

1. 使用编辑器将以下代码保存为 **mapreducejob.ps1**。必须将 **CLUSTERNAME** 替换为 HDInsight 群集的名称。

		#Login to your Azure subscription
		# Is there an active Azure subscription?
		$sub = Get-AzureSubscription -ErrorAction SilentlyContinue
		if(-not($sub))
		{
		    Add-AzureAccount
		}

		#Specify the cluster name
		$clusterName = "CLUSTERNAME"

		#Define the MapReduce job
		# -JarFile = the JAR containing the MapReduce application
		# -ClassName = the class of the application
		# -Arguments = The input file, and the output directory
		$wordCountJobDefinition = New-AzureHDInsightMapReduceJobDefinition -JarFile "wasb:///example/jars
		/hadoop-mapreduce-examples.jar" `
		                          -ClassName "wordcount" `
		                          -Arguments "wasb:///example/data/gutenberg/davinci.txt", "wasb:///example/data
		                          /WordCountOutput"

		#Submit the job to the cluster
		Write-Host "Start the MapReduce job..." -ForegroundColor Green
		$wordCountJob = Start-AzureHDInsightJob -Cluster $clusterName -JobDefinition $wordCountJobDefinition

		#Wait for the job to complete
		Write-Host "Wait for the job to complete..." -ForegroundColor Green
		Wait-AzureHDInsightJob -Job $wordCountJob -WaitTimeoutInSeconds 3600

		# Print the output
		Write-Host "Display the standard output..." -ForegroundColor Green
		Get-AzureHDInsightJobOutput -Cluster $clusterName -JobId $wordCountJob.JobId -StandardOutput

2. 打开新的 **Microsoft Azure PowerShell** 提示符。将目录切换到 **mapreducejob.ps1** 文件所在位置，然后使用以下命令来运行脚本。

		.\mapreducejob.ps1

3. 作业完成之后，你应该会收到与下面类似的输出。

		Cluster         : CLUSTERNAME
		ExitCode        : 0
		Name            : wordcount
		PercentComplete : map 100% reduce 100%
		Query           :
		State           : Completed
		StatusDirectory : f1ed2028-afe8-402f-a24b-13cc17858097
		SubmissionTime  : 12/5/2014 8:34:09 PM
		JobId           : job_1415949758166_0071

	> [AZURE.NOTE] 如果 **ExitCode** 的值不是 0，请参阅[故障排除](#troubleshooting)。

	这表示作业已成功完成。

## <a id="results"></a>查看作业输出

MapReduce 作业已将操作结果存储到 Azure Blob 存储（位于指定为作业参数的 **wasb:///example/data/WordCountOutput** 路径中）。可以通过 Azure PowerShell 访问Azure Blob 存储，但你必须知道存储帐户名称、密钥，以及 HDInsight 群集用来直接访问文件的容器。

幸运的是，你可以使用以下 Azure PowerShell Cmdlet 获取此信息：

* **Get-AzureHDInsightCluster** - 返回有关 HDInsight 群集的信息（包括任何关联的存储帐户）。始终会有与群集关联的默认存储帐户。
* **New-AzureStorageContext** - 如果指定使用 **Get-AzureHDInsightCluster** 检索的存储帐户名称和密钥，则会返回可用来访问存储帐户的内容对象。
* **Get-AzureStorageBlob** - 如果指定内容对象和容器名称，则会返回容器内的 Blob 列表。
* **Get-AzureStorageBlobContent** - 如果指定内容对象、文件路径和名称以及容器名称（从 **Get-AzureHDinsightCluster** 返回），则会从 Azure Blob 存储下载文件。

以下示例将检索存储信息，然后从 **wasb:///example/data/WordCountOutput** 下载输出。将 **CLUSTERNAME** 替换为 HDInsight 群集的名称。

		#Login to your Azure subscription
		# Is there an active Azure subscription?
		$sub = Get-AzureSubscription -ErrorAction SilentlyContinue
		if(-not($sub))
		{
		    Add-AzureAccount
		}

		#Specify the cluster name
		$clusterName = "CLUSTERNAME"

		#Retrieve the cluster information
		$clusterInfo = Get-AzureHDInsightCluster -ClusterName $clusterName

		#Get the storage account information
		$storageAccountName = $clusterInfo.DefaultStorageAccount.StorageAccountName
		$storageAccountKey = $clusterInfo.DefaultStorageAccount.StorageAccountKey
		$storageContainer = $clusterInfo.DefaultStorageAccount.StorageContainerName

		#Create the context object
		$context = New-AzureStorageContext -StorageAccountName $storageAccountName -StorageAccountKey 
		$storageAccountKey

		#Download the files from wasb:///example/data/WordCountOutput
		#Use the -blob switch to filter only blobs contained in example/data/WordCountOutput
		Get-AzureStorageBlob -Container $storageContainer -Blob example/data/WordCountOutput/* -Context $context 
		| Get-AzureStorageBlobContent -Context $context

> [AZURE.NOTE] 此示例会将下载的文件存储到你从中运行脚本的目录中的 **example/data/WordCountOutput** 文件夹内。

MapReduce 作业的输出会存储在名称为 *part-r-#####* 的文件中。使用文本编辑器打开 **example/data/WordCountOutput/part-r-00000** 文件，以查看作业生成的单词和计数。

> [AZURE.NOTE] MapReduce 作业的输出文件是固定不变的。所以，如果你重新运行此示例，将需要更改输出文件的名称。

## <a id="troubleshooting"></a>故障排除

如果在作业完成时未返回任何信息，则可能表示处理期间发生错误。若要查看此作业的错误信息，请将以下内容添加到 **mapreducejob.ps1** 文件的末尾，保存，然后重新运行该文件。

	# Print the output of the WordCount job.
	Write-Host "Display the standard output ..." -ForegroundColor Green
	Get-AzureHDInsightJobOutput -Cluster $clusterName -JobId $wordCountJob.JobId -StandardError

这样就会返回运行作业时写入到服务器上的 STDERR 的信息，而且可能有助于判断作业的失败原因。

## <a id="summary"></a>摘要

如你所见，Azure PowerShell 提供了简单的方法让你在 HDInsight 群集上运行 MapReduce 作业、监视作业状态，以及检索输出。

## <a id="nextsteps"></a>后续步骤

有关 HDInsight 中的 MapReduce 作业的一般信息。

* [在 HDInsight Hadoop 上使用 MapReduce](/documentation/articles/hdinsight-use-mapreduce)

有关 HDInsight 上的 Hadoop 的其他使用方法的信息。

* [将 Hive 与 HDInsight 上的 Hadoop 配合使用](/documentation/articles/hdinsight-use-hive)

* [将 Pig 与 HDInsight 上的 Hadoop 配合使用](/documentation/articles/hdinsight-use-pig)

<!--HONumber=50-->