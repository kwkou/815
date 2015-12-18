<properties
   pageTitle="将 MapReduce 和 PowerShell 与 Hadoop 配合使用 | Windows Azure"
   description="了解如何使用 PowerShell 在 HDInsight 上的 Hadoop 上远程运行 MapReduce 作业。"
   services="hdinsight"
   documentationCenter=""
   authors="Blackmist"
   manager="paulettm"
   editor="cgronlun"
	tags="azure-portal"/>

<tags
	ms.service="hdinsight"
	ms.date="10/16/2015"
	wacn.date=""/>

#使用 PowerShell 对 HDInsight 上的 Hadoop 运行 Hive 查询

[AZURE.INCLUDE [mapreduce-selector](../includes/hdinsight-selector-use-mapreduce.md)]

本文档提供使用 Azure PowerShell 在 HDInsight 上的 Hadoop 群集中运行 MapReduce 作业的示例。

##<a id="prereq"></a>先决条件

若要完成本文中的步骤，你将需要：

- **Azure HDInsight（HDInsight 上的 Hadoop）群集（基于 Windows）** 

- **配备 Azure PowerShell 的工作站**。请参阅[安装和配置 Azure PowerShell](/documentation/articles/powershell-install-configure)

##<a id="powershell"></a>使用 Azure PowerShell 运行 MapReduce 作业

Azure PowerShell 提供 *cmdlet*，可让你在 HDInsight 上远程运行 MapReduce 作业。从内部来讲，完成该操作的方法是使用 REST 调用 HDInsight 群集上运行的 [WebHCat](https://cwiki.apache.org/confluence/display/Hive/WebHCat)（前称 Templeton）。

在远程 HDInsight 群集上运行 MapReduce 作业时，将使用以下 Cmdlet。

* **Login-AzureRmAccount**：向 Azure 订阅进行 Azure PowerShell 身份验证

[AZURE.INCLUDE [azurerm-azurechinacloud-environment-parameter](../includes/azurerm-azurechinacloud-environment-parameter.md)]

* **New-AzureRmHDInsightMapReduceJobDefinition**：使用指定的 MapReduce 信息创建新的*作业定义*

* **Start-AzureRmHDInsightJob**：将作业定义发送到 HDInsight，启动作业，然后返回可用来检查作业状态的*作业*对象

* **Wait-AzureRmHDInsightJob**：使用作业对象来检查作业的状态。它等到作业完成或超出等待时间。

* **Get-AzureRmHDInsightJobOutput**：用于检索作业的输出

以下步骤演示了如何使用这些 Cmdlet 在 HDInsight 群集上运行作业。

1. 使用编辑器将以下代码保存为 **mapreducejob.ps1**。必须将 **CLUSTERNAME** 替换为 HDInsight 群集的名称。

		#Specify the values
		$clusterName = "CLUSTERNAME"
		$creds=Get-Credential
        		
		# Login to your Azure subscription
		# Is there an active Azure subscription?
		$sub = Get-AzureRmSubscription -ErrorAction SilentlyContinue
		if(-not($sub))
		{
		    $china = Get-AzureRmEnvironment -Name AzureChinaCloud; Login-AzureRmAccount -Environment $china
		}
        
        #Get HTTPS/Admin credentials for submitting the job later
        $creds = Get-Credential
        #Get the cluster info so we can get the resource group, storage, etc.
        $clusterInfo = Get-AzureRmHDInsightCluster -ClusterName $clusterName
        $resourceGroup = $clusterInfo.ResourceGroup
        $storageAccountName=$clusterInfo.DefaultStorageAccount.split('.')[0]
        $container=$clusterInfo.DefaultStorageContainer
        $storageAccountKey=Get-AzureRmStorageAccountKey `
            -Name $storageAccountName `
            -ResourceGroupName $resourceGroup `
            | %{ $_.Key1 }

        #Create a storage content and upload the file
        $context = New-AzureStorageContext `
            -StorageAccountName $storageAccountName `
            -StorageAccountKey $storageAccountKey
            
		#Define the MapReduce job
		#NOTE: If using an HDInsight 2.0 cluster, use hadoop-examples.jar instead.
		# -JarFile = the JAR containing the MapReduce application
		# -ClassName = the class of the application
		# -Arguments = The input file, and the output directory
		$wordCountJobDefinition = New-AzureRmHDInsightMapReduceJobDefinition `
            -JarFile "wasb:///example/jars/hadoop-mapreduce-examples.jar" `
            -ClassName "wordcount" `
            -Arguments `
                "wasb:///example/data/gutenberg/davinci.txt", `
                "wasb:///example/data/WordCountOutput"

		#Submit the job to the cluster
		Write-Host "Start the MapReduce job..." -ForegroundColor Green
		$wordCountJob = Start-AzureRmHDInsightJob `
            -ClusterName $clusterName `
            -JobDefinition $wordCountJobDefinition `
            -HttpCredential $creds

		#Wait for the job to complete
		Write-Host "Wait for the job to complete..." -ForegroundColor Green
		Wait-AzureRmHDInsightJob `
            -ClusterName $clusterName `
            -JobId $wordCountJob.JobId `
            -HttpCredential $creds
        # Download the output
        Get-AzureStorageBlobContent `
            -Blob example/data/WordCountOutput/* `
            -Container $container `
            -Destination output.txt `
            -Context $context
		# Print the output
		Get-AzureRmHDInsightJobOutput `
            -Clustername $clusterName `
            -JobId $wordCountJob.JobId `
            -DefaultContainer $container `
            -DefaultStorageAccountName $storageAccountName `
            -DefaultStorageAccountKey $storageAccountKey `
            -HttpCredential $creds
            
2. 打开一个新的 **Azure PowerShell** 命令提示符。将目录更改为 **mapreducejob.ps1** 文件所在位置，然后使用以下命令来运行脚本：

		.\mapreducejob.ps1
    
    运行该脚本时，系统可能会提示你向 Azure 订阅进行身份验证。还会要求你提供 HDInsight 群集的 HTTPS/Admin 帐户名称和密码。

3. 在作业完成后，你应该会收到如下输出：

		Cluster         : CLUSTERNAME
		ExitCode        : 0
		Name            : wordcount
		PercentComplete : map 100% reduce 100%
		Query           :
		State           : Completed
		StatusDirectory : f1ed2028-afe8-402f-a24b-13cc17858097
		SubmissionTime  : 12/5/2014 8:34:09 PM
		JobId           : job_1415949758166_0071

	此输出指示作业已成功完成。

	> [AZURE.NOTE]如果 **ExitCode** 的值不是 0，请参阅[故障排除](#troubleshooting)。

    此示例还会将下载的文件存储到你从中运行脚本的目录中的 **example/data/WordCountOutput** 文件夹。

##查看输出

MapReduce 作业的输出会存储在名称为 *part-r-#####* 的文件中。使用文本编辑器打开 **example/data/WordCountOutput/part-r-00000** 文件，以查看作业生成的单词和计数。

> [AZURE.NOTE]MapReduce 作业的输出文件是固定不变的。因此，如果你重新运行此示例，将需要更改输出文件的名称。

##<a id="troubleshooting"></a>故障排除

如果在作业完成时未返回任何信息，则可能表示处理期间发生错误。若要查看此作业的错误信息，请将以下命令添加到 **mapreducejob.ps1** 文件的末尾，保存，然后重新运行该文件。

	# Print the output of the WordCount job.
	Write-Host "Display the standard output ..." -ForegroundColor Green
	Get-AzureRmHDInsightJobOutput `
            -Clustername $clusterName `
            -JobId $wordCountJob.JobId `
            -DefaultContainer $container `
            -DefaultStorageAccountName $storageAccountName `
            -DefaultStorageAccountKey $storageAccountKey `
            -HttpCredential $creds
            -DisplayOutputType StandardError

在运行作业时，这将返回写入到服务器上的 STDERR 的信息，它可帮助确定该作业失败的原因。

##<a id="summary"></a>摘要

如你所见，Azure PowerShell 提供了简单的方法让你在 HDInsight 群集上运行 MapReduce 作业、监视作业状态，以及检索输出。

##<a id="nextsteps"></a>后续步骤

有关 HDInsight 中的 MapReduce 作业的一般信息：

* [在 HDInsight Hadoop 上使用 MapReduce](/documentation/articles/hdinsight-use-mapreduce)

有关 HDInsight 上的 Hadoop 的其他使用方法的信息：

* [将 Hive 与 HDInsight 上的 Hadoop 配合使用](/documentation/articles/hdinsight-use-hive)

* [将 Pig 与 HDInsight 上的 Hadoop 配合使用](/documentation/articles/hdinsight-use-pig)

<!---HONumber=Mooncake_1207_2015-->