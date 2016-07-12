<properties
   pageTitle="在 HDInsight 中将 Hadoop Hive 与 PowerShell 配合使用 | Azure"
   description="使用 PowerShell 在 HDInsight 上的 Hadoop 中运行 Hive 查询。"
   services="hdinsight"
   documentationCenter=""
   authors="Blackmist"
   manager="paulettm"
   editor="cgronlun"
	tags="azure-portal"/>

<tags
	ms.service="hdinsight"
	ms.date="04/07/2016"
	wacn.date="05/24/2016"/>

#使用 PowerShell 运行 Hive 查询

[AZURE.INCLUDE [hive-selector](../includes/hdinsight-selector-use-hive.md)]

> [AZURE.NOTE] 本文档未详细描述示例中使用的 HiveQL 语句的作用。有关此示例中使用的 HiveQL 的信息，请参阅[将 Hive 与 HDInsight 上的 Hadoop 配合使用](/documentation/articles/hdinsight-use-hive/)。


**<a id="prereq"></a>先决条件**

若要完成本文中的步骤，你将需要：

- **Azure HDInsight（HDInsight 上的 Hadoop）群集（基于 Windows）** 
- **配备 Azure PowerShell 的工作站**。

    [AZURE.INCLUDE [upgrade-powershell](../includes/hdinsight-use-latest-powershell.md)]

##<a id="powershell"></a>使用 Azure PowerShell 运行 Hive 查询

Azure PowerShell 提供 *cmdlet*，可让你在 HDInsight 上远程运行 Hive 查询。从内部来讲，这是通过使用 REST 调用 HDInsight 群集上运行的 [WebHCat](https://cwiki.apache.org/confluence/display/Hive/WebHCat)（以前称为 Templeton）实现的。

在远程 HDInsight 群集上运行 Hive 查询时，将使用以下 Cmdlet：

* **Add-AzureAccount**：在 Azure 订阅中进行 Azure PowerShell 身份验证

[AZURE.INCLUDE [automation-azurechinacloud-environment-parameter](../includes/automation-azurechinacloud-environment-parameter.md)]

* **New-AzureHDInsightHiveJobDefinition**：使用指定的 HiveQL 语句创建新的*作业定义*

* **Start-AzureHDInsightJob**：将作业定义发送到 HDInsight，启动作业，然后返回可用来检查作业状态的*作业* 对象

* **Wait-AzureHDInsightJob**：使用作业对象来检查作业的状态。它将一直等到作业完成或超出等待时间。

* **Get-AzureHDInsightJobOutput**：用于检索作业的输出

* **Invoke-AzureHDInsightHiveJob**：用于运行 HiveQL 语句。这将阻止查询完成，然后返回结果

* **Use-AzureHDInsightCluster**：设置要用于 **Invoke-Hive** 命令的当前群集

以下步骤演示了如何使用这些 Cmdlet 在 HDInsight 群集上运行作业：

1. 使用编辑器将以下代码保存为 **hivejob.ps1**。必须将 **CLUSTERNAME** 替换为 HDInsight 群集的名称。

		#Specify the values
		$clusterName = "CLUSTERNAME"
		$httpUsername = "HTTPUSERNAME"
		$httpUserPassword  = "HTTPUSERPASSWORD"
		# Login to your Azure subscription
		# Is there an active Azure subscription?
		$sub = Get-AzureSubscription -ErrorAction SilentlyContinue
		if(-not($sub))
		{
		    Add-AzureAccount -Environment AzureChinaCloud
		}

		#HiveQL
		$queryString = "DROP TABLE log4jLogs;" +
				       "CREATE EXTERNAL TABLE log4jLogs(t1 string, t2 string, t3 string, t4 string, t5 string, t6 string, t7 string) ROW FORMAT DELIMITED FIELDS TERMINATED BY ' ' STORED AS TEXTFILE LOCATION 'wasb:///example/data/';" +
				       "SELECT * FROM log4jLogs WHERE t4 = '[ERROR]';"

		#Create an HDInsight Hive job definition
		$hiveJobDefinition = New-AzureHDInsightHiveJobDefinition -Query $queryString 

		#Submit the job to the cluster
		Write-Host "Start the Hive job..." -ForegroundColor Green

		$passwd = ConvertTo-SecureString $httpUserPassword -AsPlainText -Force
		$creds = New-Object System.Management.Automation.PSCredential ($httpUsername, $passwd)
		$hiveJob = Start-AzureHDInsightJob -Cluster $clusterName -JobDefinition $hiveJobDefinition -Credential $creds


		#Wait for the Hive job to complete
		Write-Host "Wait for the job to complete..." -ForegroundColor Green
		Wait-AzureHDInsightJob -Cluster $clusterName -JobId $hiveJob.JobId -Credential $creds

		# Print the output
		Write-Host "Display the standard output..." -ForegroundColor Green
		Get-AzureHDInsightJobOutput -ClusterName $clusterName -JobId $hiveJob.JobId -StandardOutput 

            
2. 打开一个新的 **Azure PowerShell** 命令提示符。将目录更改为 **hivejob.ps1** 文件的所在位置，然后使用以下命令来运行脚本：

		.\hivejob.ps1

    脚本运行时，系统将提示你输入你的群集的 HTTPS/Admin 帐户凭据。还可能会提示你登录到 Azure 订阅。
    
7. 在作业完成时，它应返回如下信息：

        Display the standard output...
        2012-02-03      18:35:34        SampleClass0    [ERROR] incorrect       id
        2012-02-03      18:55:54        SampleClass1    [ERROR] incorrect       id
        2012-02-03      19:25:27        SampleClass4    [ERROR] incorrect       id

4. 如前所述，**Invoke-Hive** 可以用来运行查询，并等待响应。使用以下命令，并将 **CLUSTERNAME** 替换为群集的名称：

        Use-AzureHDInsightCluster CLUSTERNAME
		Invoke-Hive -Query @"
		CREATE TABLE IF NOT EXISTS errorLogs (t1 string, t2 string, t3 string, t4 string, t5 string, t6 string, t7 string) STORED AS ORC;
        INSERT OVERWRITE TABLE errorLogs SELECT t1, t2, t3, t4, t5, t6, t7 FROM log4jLogs WHERE t4 = '[ERROR]';
        SELECT * FROM errorLogs;
        "@

	输出将如下所示：

		2012-02-03	18:35:34	SampleClass0	[ERROR]	incorrect	id
		2012-02-03	18:55:54	SampleClass1	[ERROR]	incorrect	id
		2012-02-03	19:25:27	SampleClass4	[ERROR]	incorrect	id

	> [AZURE.NOTE] 对于较长的 HiveQL 查询，你可以使用 Azure PowerShell **Here-Strings** cmdlet 或 HiveQL 脚本文件。以下代码段显示了如何使用 **Invoke-Hive** cmdlet 来运行 HiveQL 脚本文件。HiveQL 脚本文件必须上载到 wasb://。
	> <p>`Invoke-AzureHDInsightHiveJob -File "wasb://<ContainerName>@<StorageAccountName>/<Path>/query.hql"`
	> <p>有关 **Here-Strings** 的详细信息，请参阅<a href="http://technet.microsoft.com/zh-cn/library/ee692792.aspx" target="_blank">使用 Windows PowerShell Here-Strings</a>。

##<a id="troubleshooting"></a>故障排除

如果在作业完成时未返回任何信息，则可能表示处理期间发生错误。若要查看此作业的错误信息，请将以下内容添加到 **hivejob.ps1** 文件的末尾，保存，然后重新运行该文件。

	# Print the output of the Hive job.
	Write-Host "Display the standard output ..." -ForegroundColor Green
	Get-AzureHDInsightJobOutput -Cluster $clusterName -JobId $hiveJob.JobId -StandardError

在运行作业时，这将返回写入到服务器上的 STDERR 中的信息，该信息可帮助确定该作业失败的原因。

##<a id="summary"></a>摘要

如你所见，Azure PowerShell 提供了简单的方法让你在 HDInsight 群集上运行 Hive 查询，监视作业状态，以及检索输出。

##<a id="nextsteps"></a>后续步骤

有关 HDInsight 中的 Hive 的一般信息：

* [将 Hive 与 HDInsight 上的 Hadoop 配合使用](/documentation/articles/hdinsight-use-hive/)

有关 HDInsight 上的 Hadoop 的其他使用方法的信息：

* [将 Pig 与 HDInsight 上的 Hadoop 配合使用](/documentation/articles/hdinsight-use-pig/)

* [将 MapReduce 与 HDInsight 上的 Hadoop 配合使用](/documentation/articles/hdinsight-use-mapreduce/)

<!---HONumber=Mooncake_0215_2016-->