<properties
   pageTitle="在 HDInsight 中将 Hadoop Pig 与 PowerShell 配合使用 | Azure"
   description="了解如何使用 Azure PowerShell 将 Pig 作业提交到 HDInsight 上的 Hadoop 群集。"
   services="hdinsight"
   documentationCenter=""
   authors="Blackmist"
   manager="paulettm"
   editor="cgronlun"
	tags="azure-portal"/>

<tags
	ms.service="hdinsight"
	ms.date="04/26/2016"
	wacn.date="06/29/2016"/>

#使用 PowerShell 运行 Pig 作业

[AZURE.INCLUDE [pig-selector](../includes/hdinsight-selector-use-pig.md)]

本文档提供使用 Azure PowerShell 向 HDInsight 群集上的 Hadoop 提交 Pig 作业的示例。Pig 允许你通过使用可为数据转换建模的语言 (Pig Latin) 编写 MapReduce 作业，而无需使用映射和化简函数。

> [AZURE.NOTE]本文档未详细描述示例中使用的 Pig Latin 语句的作用。有关此示例中使用的 Pig Latin 的信息，请参阅[将 Pig 与 HDInsight 上的 Hadoop 配合使用](/documentation/articles/hdinsight-use-pig/)。

##<a id="prereq"></a>先决条件

若要完成本文中的步骤，你将需要：

- **一个 Azure 订阅**。请参阅[获取 Azure 试用版](/pricing/1rmb-trial/)。

- **配备 Azure PowerShell 的工作站**。

    [AZURE.INCLUDE [upgrade-powershell](../includes/hdinsight-use-latest-powershell.md)]

##<a id="powershell"></a>使用 PowerShell 运行 Pig 作业

Azure PowerShell 提供 *cmdlet*，可让你在 HDInsight 上远程运行 Pig 作业。从内部来讲，这是通过使用 REST 调用 HDInsight 群集上运行的 [WebHCat](https://cwiki.apache.org/confluence/display/Hive/WebHCat)（以前称为 Templeton）实现的。

在远程 HDInsight 群集上运行 Pig 作业时，将使用以下 Cmdlet：

* **Add-AzureAccount**：向 Azure 订阅进行 Azure PowerShell 身份验证

[AZURE.INCLUDE [automation-azurechinacloud-environment-parameter](../includes/automation-azurechinacloud-environment-parameter.md)]

* **New-AzureHDInsightPigJobDefinition**：使用指定的 Pig Latin 语句创建新的*作业定义*

* **Start-AzureHDInsightJob**：将作业定义发送到 HDInsight，启动作业，然后返回可用来检查作业状态的*作业*对象

* **Wait-AzureHDInsightJob**：使用作业对象来检查作业的状态。它将等到作业完成或超出等待时间。

* **Get-AzureHDInsightJobOutput**：用于检索作业的输出

以下步骤演示了如何使用这些 Cmdlet 在 HDInsight 群集上运行作业。

1. 使用编辑器将以下代码保存为 **pigjob.ps1**。必须将 **CLUSTERNAME** 替换为 HDInsight 群集的名称。

		#Specify the cluster name
		$clusterName = "CLUSTERNAME"
		#Where the output will be saved
		$statusFolder = "/tutorial/pig/status"

		#Store the Pig Latin into $QueryString
		$QueryString =  "LOGS = LOAD 'wasb:///example/data/sample.log';" +
		"LEVELS = foreach LOGS generate REGEX_EXTRACT(`$0, '(TRACE|DEBUG|INFO|WARN|ERROR|FATAL)', 1)  as LOGLEVEL;" +
		"FILTEREDLEVELS = FILTER LEVELS by LOGLEVEL is not null;" +
		"GROUPEDLEVELS = GROUP FILTEREDLEVELS by LOGLEVEL;" +
		"FREQUENCIES = foreach GROUPEDLEVELS generate group as LOGLEVEL, COUNT(FILTEREDLEVELS.LOGLEVEL) as COUNT;" +
		"RESULT = order FREQUENCIES by COUNT desc;" +
		"DUMP RESULT;"

		#Create a new HDInsight Pig Job definition
		$pigJobDefinition = New-AzureHDInsightPigJobDefinition -Query $QueryString -StatusFolder $statusFolder

		# Start the Pig job on the HDInsight cluster
		Write-Host "Start the Pig job ..." -ForegroundColor Green
		$pigJob = Start-AzureHDInsightJob -Cluster $clusterName -JobDefinition $pigJobDefinition

		# Wait for the Pig job to complete
		Write-Host "Wait for the Pig job to complete ..." -ForegroundColor Green
		Wait-AzureHDInsightJob -Job $pigJob -WaitTimeoutInSeconds 3600

		# Print the output of the Pig job.
		Write-Host "Display the standard output ..." -ForegroundColor Green
		Get-AzureHDInsightJobOutput -Cluster $clusterName -JobId $pigJob.JobId -StandardOutput

2. 打开一个新的 Windows PowerShell 命令提示符。将目录更改为 **pigjob.ps1** 文件的所在位置，然后使用以下命令来运行脚本：

		.\pigjob.ps1
        
    首先会提示你登录到 Azure 订阅。然后，将要求你输入 HDInsight 群集的 HTTPs/Admin 帐户名称和密码。

7. 在作业完成时，它应返回如下信息：

        Start the Pig job ...
        Wait for the Pig job to complete ...
        Display the standard output ...
        (TRACE,816)
        (DEBUG,434)
        (INFO,96)
        (WARN,11)
        (ERROR,6)
        (FATAL,2)

##<a id="troubleshooting"></a>故障排除

如果在作业完成时未返回任何信息，则可能表示处理期间发生错误。若要查看此作业的错误信息，请将以下命令添加到 **pigjob.ps1** 文件的末尾，保存，然后重新运行该文件。

	# Print the output of the Pig job.
	Write-Host "Display the standard error output ..." -ForegroundColor Green
    Get-AzureHDInsightJobOutput -Cluster $clusterName -JobId $pigJob.JobId -StandardError

这样就会返回运行作业时写入到服务器上的 STDERR 的信息，它可帮助确定该作业失败的原因。

##<a id="summary"></a>摘要

如你所见，Azure PowerShell 提供了简单的方法让你在 HDInsight 群集上运行 Pig 作业、监视作业状态，以及检索输出。

##<a id="nextsteps"></a>后续步骤

有关 HDInsight 中的 Pig 的一般信息：

* [将 Pig 与 HDInsight 上的 Hadoop 配合使用](/documentation/articles/hdinsight-use-pig/)

有关 HDInsight 上的 Hadoop 的其他使用方法的信息：

* [将 Hive 与 HDInsight 上的 Hadoop 配合使用](/documentation/articles/hdinsight-use-hive/)

* [将 MapReduce 与 HDInsight 上的 Hadoop 配合使用](/documentation/articles/hdinsight-use-mapreduce/)

<!---HONumber=Mooncake_1207_2015-->