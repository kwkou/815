<properties
    pageTitle="将 MapReduce 和 PowerShell 与 Hadoop 配合使用 | Azure"
    description="了解如何使用 PowerShell 在 HDInsight 的 Hadoop 上远程运行 MapReduce 作业。"
    services="hdinsight"
    documentationcenter=""
    author="Blackmist"
    manager="jhubbard"
    editor="cgronlun"
    tags="azure-portal" />
<tags
    ms.assetid="21b56d32-1785-4d44-8ae8-94467c12cfba"
    ms.service="hdinsight"
    ms.devlang="na"
    ms.topic="article"
    ms.tgt_pltfrm="na"
    ms.workload="big-data"
    ms.date="11/15/2016"
    wacn.date="01/25/2017"
    ms.author="larryfr" />  


# 通过 PowerShell 使用 HDInsight 上的 Hadoop 运行 MapReduce 作业

[AZURE.INCLUDE [mapreduce-selector](../../includes/hdinsight-selector-use-mapreduce.md)]

[AZURE.INCLUDE [azure-sdk-developer-differences](../../includes/azure-sdk-developer-differences.md)]

本文档提供了一个示例，演示了使用 Azure PowerShell 在 HDInsight 的 Hadoop 群集中运行 MapReduce 作业。

## <a id="prereq"></a>先决条件

要完成本文中的步骤，需要：

* **Azure HDInsight（HDInsight 上的 Hadoop）群集（基于 Windows）**
* **配备 Azure PowerShell 的工作站**。
  
[AZURE.INCLUDE [upgrade-powershell](../../includes/hdinsight-use-latest-powershell.md)]

## <a id="powershell"></a>使用 Azure PowerShell 运行 MapReduce 作业

Azure PowerShell 提供 *cmdlet*，可让你在 HDInsight 上远程运行 MapReduce 作业。从内部来讲，完成该操作的方法是使用 REST 调用 HDInsight 群集上运行的 [WebHCat](https://cwiki.apache.org/confluence/display/Hive/WebHCat)（前称 Templeton）。

在远程 HDInsight 群集上运行 MapReduce 作业时，将使用以下 Cmdlet。

* **Login-AzureRmAccount -EnvironmentName AzureChinaCloud**：向 Azure 订阅进行 Azure PowerShell 身份验证。

* **New-AzureRmHDInsightMapReduceJobDefinition**：使用指定的 MapReduce 信息创建新*作业定义*。

* **Start-AzureRmHDInsightJob**：将作业定义发送到 HDInsight，启动作业，然后返回可用来检查作业状态的*作业*对象。

* **Wait-AzureRmHDInsightJob**：使用作业对象来检查作业的状态。它等到作业完成或超出等待时间。

* **Get-AzureRmHDInsightJobOutput**：用于检索作业输出。

以下步骤演示了如何使用这些 Cmdlet 在 HDInsight 群集上运行作业。

1. 使用编辑器将以下代码保存为 **mapreducejob.ps1**。必须将 **CLUSTERNAME** 替换为 HDInsight 群集的名称。

        #Specify the values
        $clusterName = "CLUSTERNAME"

        # Login to your Azure subscription
        # Is there an active Azure subscription?
        $sub = Get-AzureRmSubscription -ErrorAction SilentlyContinue
        if(-not($sub))
        {
            Login-AzureRmAccount -EnvironmentName AzureChinaCloud
        }

        #Get HTTPS/Admin credentials for submitting the job later
        $creds = Get-Credential
        #Get the cluster info so we can get the resource group, storage, etc.
        $clusterInfo = Get-AzureRmHDInsightCluster -ClusterName $clusterName
        $resourceGroup = $clusterInfo.ResourceGroup
        $storageAccountName=$clusterInfo.DefaultStorageAccount.split('.')[0]
        $container=$clusterInfo.DefaultStorageContainer
        $storageAccountKey=(Get-AzureRmStorageAccountKey `
            -Name $storageAccountName `
        -ResourceGroupName $resourceGroup)[0].Value

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
            -JarFile "wasbs:///example/jars/hadoop-mapreduce-examples.jar" `
            -ClassName "wordcount" `
            -Arguments `
                "wasbs:///example/data/gutenberg/davinci.txt", `
                "wasbs:///example/data/WordCountOutput"

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
            -Blob 'example/data/WordCountOutput/part-r-00000' `
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
   
    运行脚本时，系统将提示对 Azure 订阅进行身份验证。还会要求你提供 HDInsight 群集的 HTTPS/Admin 帐户名称和密码。

3. 作业完成后，应显示如下输出：
    
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
    
    > [AZURE.NOTE]
    如果 **ExitCode** 的值不是 0，请参阅[故障排除](#troubleshooting)。
    
    此示例还会将下载的文件存储到从中运行脚本的目录中的 **output.txt** 文件。

### 查看输出

在文本编辑器中打开 **output.txt** 文件，以查看作业生成的单词和计数。

> [AZURE.NOTE]
MapReduce 作业的输出文件是固定不变的。因此，如果重新运行此示例，需要更改输出文件的名称。

## <a id="troubleshooting"></a>故障排除

如果作业完成时未返回任何信息，可能表示处理期间发生错误。若要查看此作业的错误信息，请将以下命令添加到 **mapreducejob.ps1** 文件的末尾，保存，然后重新运行该文件。

    # Print the output of the WordCount job.
    Write-Host "Display the standard output ..." -ForegroundColor Green
    Get-AzureRmHDInsightJobOutput `
            -Clustername $clusterName `
            -JobId $wordCountJob.JobId `
            -DefaultContainer $container `
            -DefaultStorageAccountName $storageAccountName `
            -DefaultStorageAccountKey $storageAccountKey `
            -HttpCredential $creds `
            -DisplayOutputType StandardError

运行作业时，这将返回写入到服务器上的 STDERR 的信息，可用于确定该作业失败的原因。

## <a id="summary"></a>摘要

Azure PowerShell 提供了一种简单方法，可让你在 HDInsight 群集上运行 MapReduce 作业、监视作业状态，以及检索输出。

## <a id="nextsteps"></a>后续步骤

有关 HDInsight 中的 MapReduce 作业的一般信息：

* [在 HDInsight Hadoop 上使用 MapReduce](/documentation/articles/hdinsight-use-mapreduce/)

有关 HDInsight 上 Hadoop 的其他使用方法的信息：

* [将 Hive 与 HDInsight 上的 Hadoop 配合使用](/documentation/articles/hdinsight-use-hive/)
* [将 Pig 与 HDInsight 上的 Hadoop 配合使用](/documentation/articles/hdinsight-use-pig/)

<!---HONumber=Mooncake_0120_2017-->
<!--Update_Description: update from ASM to ARM-->