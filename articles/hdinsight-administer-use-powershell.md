<properties 
	pageTitle="使用 PowerShell 管理 HDInsight 中的 Hadoop 群集 | Azure" 
	description="了解如何使用 Azure PowerShell 执行针对 HDInsight 中 Hadoop 的管理任务。" 
	services="hdinsight" 
	editor="cgronlun" 
	manager="paulettm" 
	authors="mumian" 
	documentationCenter=""/>

<tags 
	ms.service="hdinsight"
	ms.date="07/21/2015" 
	wacn.date="" />

# 使用 Azure PowerShell 管理 HDInsight 中的 Hadoop 群集



Azure PowerShell 是一个功能强大的脚本编写环境，可用于在 Azure 中控制和自动执行工作负荷的部署和管理。在本文中，你将要学习如何通过使用 Windows PowerShell 借助于本地 Azure PowerShell 控制台来管理 Azure HDInsight 中的 Hadoop 群集。有关 HDInsight PowerShell cmdlet 的列表，请参阅 [HDInsight cmdlet 参考][hdinsight-powershell-reference]。
			
##先决条件
	
在开始阅读本文前，你必须具有：

- **一个 Azure 订阅**。请参阅[购买选项][azure-purchase-options]、[试用][azure-trial]。

- **配备 Azure PowerShell 的工作站**。请参阅[安装和使用 Azure PowerShell][Powershell-install-configure]。


##设置 HDInsight 群集
HDInsight 使用 Azure Blob 存储容器作为默认文件系统。你需要先拥有 Azure 存储帐户和存储容器，然后才能创建 HDInsight 群集。

[AZURE.INCLUDE [provisioningnote](../includes/hdinsight-provisioning.md)]

**创建 Azure 存储帐户**

在导入了 publishsettings 文件后，你可以使用以下命令创建一个存储帐户：

	# Create an Azure Storage account
	$storageAccountName = "<StorageAcccountName>"
	$location = "<Microsoft data center>"           # For example, "China East"

	New-AzureStorageAccount -StorageAccountName $storageAccountName -Location $location


[AZURE.INCLUDE [数据中心列表](../includes/hdinsight-pricing-data-centers-clusters.md)]


有关通过使用 Azure 门户创建 Azure 存储帐户的信息，请参阅[创建、管理或删除存储帐户](/documentation/articles/storage-create-storage-account/)。

如果你已有存储帐户但是不知道帐户名称和帐户密钥，可以使用以下命令来检索该信息：

	# List Storage accounts for the current subscription
	Get-AzureStorageAccount
	# List the keys for a Storage account
	Get-AzureStorageKey <StorageAccountName>

有关使用 Azure 门户获取信息的详细信息，请参阅[创建、管理或删除存储帐户](/documentation/articles/storage-create-storage-account/)中的“查看、复制和重新生成存储访问密钥”部分。

**创建 Azure 存储帐户**

Azure PowerShell 无法在 HDInsight 设置过程中创建 Blob 容器。你可以使用以下脚本创建一个容器：

	$storageAccountName = "<StorageAccountName>"
	$storageAccountKey = Get-AzureStorageKey $storageAccountName | %{ $_.Primary }
	$containerName="<ContainerName>"

	# Create a storage context object
	$destContext = New-AzureStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $storageAccountKey  
	
	# Create a Blob storage container
	New-AzureStorageContainer -Name $containerName -Context $destContext

**设置群集**

准备好存储帐户和 Blob 容器后，你就可以创建群集了。
		
	$storageAccountName = "<StorageAccountName>"
	$containerName = "<ContainerName>"

	$clusterName = "<HDInsightClusterName>"
	$location = "<MicrosoftDataCenter>"
	$clusterNodes = <ClusterSizeInNodes>

	# Get the Storage account key
	$storageAccountKey = Get-AzureStorageKey $storageAccountName | %{ $_.Primary }

	# Create a new HDInsight cluster
	New-AzureHDInsightCluster -Name $clusterName -Location $location -DefaultStorageAccountName "$storageAccountName.blob.core.chinacloudapi.cn" -DefaultStorageAccountKey $storageAccountKey -DefaultStorageContainerName $containerName  -ClusterSizeInNodes $clusterNodes


下面的屏幕快照显示脚本执行：

![HDI.PS.Provision][image-hdi-ps-provision]




##列出群集详细信息
使用以下命令可列出当前订阅中的所有群集：

	Get-AzureHDInsightCluster 

使用以下命令可显示当前订阅中特定群集的详细信息：

	Get-AzureHDInsightCluster -Name <ClusterName> 

##删除群集
使用以下命令来删除群集：

	Remove-AzureHDInsightCluster -Name <ClusterName> 

##授予/撤消 HTTP 服务访问权限

HDInsight 群集提供以下 HTTP Web 服务（所有这些服务都有 REST 样式的终结点）：

- ODBC
- JDBC
- Ambari
- Oozie
- Templeton


默认情况下，将授权这些服务进行访问。你可以撤消/授予访问权限。下面是一个示例：

	Revoke-AzureHDInsightHttpServicesAccess -Name hdiv2 -Location "China East"

在此示例中，<i>hdiv2</i> 是 HDInsight 群集名称。

>[AZURE.NOTE]授予/撤消访问权限时，你将重设群集用户的用户名和密码。

也可以使用 Azure 门户完成此操作。请参阅[使用 Azure 门户管理 HDInsight][hdinsight-admin-portal]。

##缩放群集
请参阅[缩放 HDInsight 中的 Hadoop 群集](/documentation/articles/hdinsight-hadoop-cluster-scaling)。

##提交 MapReduce 作业
HDInsight 群集分发附带一些 MapReduce 示例。其中一个示例是计算源文件中的单词频率。

**提交 MapReduce 作业**

以下 Azure PowerShell 脚本将提交单词计数示例作业：
	
	$clusterName = "<HDInsightClusterName>"            
	
	# Define the MapReduce job
	$wordCountJobDefinition = New-AzureHDInsightMapReduceJobDefinition -JarFile "wasb:///example/jars/hadoop-mapreduce-examples.jar" -ClassName "wordcount" -Arguments "wasb:///example/data/gutenberg/davinci.txt", "wasb:///example/data/WordCountOutput"
	
	# Run the job and show the standard error 
	$wordCountJobDefinition | Start-AzureHDInsightJob -Cluster $clusterName | Wait-AzureHDInsightJob -WaitTimeoutInSeconds 3600 | %{ Get-AzureHDInsightJobOutput -Cluster $clusterName -JobId $_.JobId -StandardError}
	
有关 **wasb** 前缀的信息，请参阅[将 Azure Blob 存储用于 HDInsight][hdinsight-storage]。

**下载 MapReduce 作业输出**

以下 Azure PowerShell 脚本将从上一个过程中检索 MapReduce 作业输出：

	$storageAccountName = "<StorageAccountName>"   
	$containerName = "<ContainerName>"             
		
	# Create the Storage account context object
	$storageAccountKey = Get-AzureStorageKey $storageAccountName | %{ $_.Primary }
	$storageContext = New-AzureStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $storageAccountKey  
	
	# Download the output to local computer
	Get-AzureStorageBlobContent -Container $ContainerName -Blob example/data/WordCountOutput/part-r-00000 -Context $storageContext -Force
	
	# Display the output
	cat ./example/data/WordCountOutput/part-r-00000 | findstr "there"

有关开发和运行 MapReduce 作业的详细信息，请参阅 [将 MapReduce 与 HDInsight 配合使用][hdinsight-use-mapreduce]。






































##提交 Hive 作业
HDInsight 群集分发附带称作 *hivesampletable* 的示例 Hive 表。你可以使用 HiveQL **SHOW TABLES** 命令列出群集上的 Hive 表。

**提交 Hive 作业**

以下脚本将提交一个 Hive 作业以列出 Hive 表：
	
	$clusterName = "<HDInsightClusterName>"               
	
	# HiveQL query
	$querystring = @"
		SHOW TABLES;
		SELECT * FROM hivesampletable 
			WHERE Country='United Kingdom'
			LIMIT 10;
	"@

	Use-AzureHDInsightCluster -Name $clusterName
	Invoke-Hive $querystring

该 Hive 作业将首先显示在该群集上创建的 Hive 表，并且显示从 hivesampletable 表返回的数据。

有关使用 Hive 的详细信息，请参阅 [将 Hive 与 HDInsight 配合使用][hdinsight-use-hive]。


##将数据上载到 Azure Blob 存储
请参阅[将数据上载到 HDInsight][hdinsight-upload-data]。

##从 Azure Blob 存储下载作业输出
请参阅本文中的[提交 MapReduce 作业](#mapreduce)部分。

## 另请参阅
* [HDInsight cmdlet 参考文档][hdinsight-powershell-reference]
* [使用 Azure 门户管理 HDInsight][hdinsight-admin-portal]
* [使用命令行界面管理 HDInsight][hdinsight-admin-cli]
* [设置 HDInsight 群集][hdinsight-provision]
* [将数据上载到 HDInsight][hdinsight-upload-data]
* [以编程方式提交 Hadoop 作业][hdinsight-submit-jobs]
* [Azure HDInsight 入门][hdinsight-get-started]


[azure-purchase-options]: /pricing/purchase-options/

[azure-trial]: /pricing/1rmb-trial/

[hdinsight-get-started]: /documentation/articles/hdinsight-get-started/
[hdinsight-provision]: /documentation/articles/hdinsight-provision-clusters/

[hdinsight-submit-jobs]: /documentation/articles/hdinsight-submit-hadoop-jobs-programmatically/

[hdinsight-admin-portal]: /documentation/articles/hdinsight-administer-use-management-portal/
[hdinsight-admin-cli]: /documentation/articles/hdinsight-administer-use-command-line/
[hdinsight-storage]: /documentation/articles/hdinsight-use-blob-storage/
[hdinsight-mapreduce]: /documentation/articles/hdinsight-use-mapreduce/

[hdinsight-hive]: /documentation/articles/hdinsight-use-hive/
[hdinsight-upload-data]: /documentation/articles/hdinsight-upload-data/

[hdinsight-powershell-reference]: http://msdn.microsoft.com/zh-cn/library/azure/dn479228.aspx

[Powershell-install-configure]: /documentation/articles/install-configure-powershell/

[image-hdi-ps-provision]: ./media/hdinsight-administer-use-powershell/HDI.PS.Provision.png

<!---HONumber=66-->