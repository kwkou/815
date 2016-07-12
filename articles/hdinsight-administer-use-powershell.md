<properties
	pageTitle="使用 PowerShell 管理 HDInsight 中的 Hadoop 群集 | Azure"
	description="了解如何使用 Azure PowerShell 执行针对 HDInsight 中 Hadoop 的管理任务。"
	services="hdinsight"
	editor="cgronlun"
	manager="paulettm"
	tags="azure-portal"
	authors="mumian"
	documentationCenter=""/>

<tags
	ms.service="hdinsight"
	ms.date="04/05/2016"
	wacn.date="05/24/2016"/>

# 使用 Azure PowerShell 管理 HDInsight 中的 Hadoop 群集

[AZURE.INCLUDE [选择器](../includes/hdinsight-portal-management-selector.md)]

Azure PowerShell 是一个功能强大的脚本编写环境，可用于在 Azure 中控制和自动执行工作负荷的部署和管理。在本文中，你将要学习如何通过使用 Windows PowerShell 借助于本地 Azure PowerShell 控制台来管理 Azure HDInsight 中的 Hadoop 群集。有关 HDInsight PowerShell cmdlet 的列表，请参阅 [HDInsight cmdlet 参考][hdinsight-powershell-reference]。



**先决条件**

在开始阅读本文前，你必须具有：

- **一个 Azure 订阅**。请参阅[获取 Azure 试用版](/pricing/1rmb-trial/)。

##<a id="install-azure-powershell-10-and-greater"></a>安装 Azure PowerShell 1.0 和更高版本

首先，必须卸载 0.9x 版本。

若要检查所安装的 PowerShell 版本，请执行以下操作：

	Get-Module *azure*
	
若要卸载旧版本，请运行控制面板中的“程序和功能”。

有两个主要选项用于安装 Azure PowerShell。

- [PowerShell 库](https://www.powershellgallery.com/)。在已提升权限的 PowerShell ISE 或已提升权限的 Windows PowerShell 控制台中运行以下命令：
		
		# Install the Azure Service Management module from PowerShell Gallery
		Install-Module Azure
		
		# Import Azure Service Management module
		Import-Module Azure

	有关详细信息，请参阅 [PowerShell 库](https://www.powershellgallery.com/)。

- [Microsoft Web 平台安装程序 (WebPI)](http://aka.ms/webpi-azps)。如果你已安装 Azure PowerShell 0.9.x，系统将提示你卸载 0.9.x。如果你从 PowerShell 库安装了 Azure PowerShell 模块，安装程序将要求你在安装之前删除该模块，以确保 Azure PowerShell 环境一致。有关说明，请参阅[通过 WebPI 安装 Azure PowerShell 1.0](https://azure.microsoft.com/blog/azps-1-0/)。

WebPI 将每月接收更新。PowerShell 库将持续接收更新。如果你能够熟练地从 PowerShell 库安装，则该库就是在 Azure PowerShell 中获得最新、最完善更新的首要途径。

##创建群集

HDInsight 群集要求在 Azure 存储帐户中创建 Blob 容器：

- HDInsight 使用 Azure 存储帐户的 Blob 容器作为默认文件系统。你需要先拥有 Azure 存储帐户和存储容器，然后才能创建 HDInsight 群集。默认存储帐户和 HDInsight 群集必须位于同一位置。

[AZURE.INCLUDE [provisioningnote](../includes/hdinsight-provisioning.md)]

**连接到 Azure**

[AZURE.INCLUDE [automation-azurechinacloud-environment-parameter](../includes/automation-azurechinacloud-environment-parameter.md)]

	Add-AzureAccount -Environment AzureChinaCloud
	Get-AzureSubscription  # list your subscriptions and get your subscription ID
	Select-AzureSubscription -SubscriptionId "<Your Azure Subscription ID>"

如果你有多个 Azure 订阅，将调用 **Select-AzureSubscription**。

**创建 Azure 存储帐户**

	New-AzureStorageAccount -StorageAccountName <Azure Storage Account Name> -Location "<Azure Location>" -Type <AccountType> # account type example: Standard_LRS for zero redundancy storage
	
[AZURE.INCLUDE [数据中心列表](../includes/hdinsight-pricing-data-centers-clusters.md)]


如果你已有存储帐户但是不知道帐户名称和帐户密钥，可以使用以下命令来检索该信息：

	# List Storage accounts for the current subscription
	Get-AzureStorageAccount
	# List the keys for a Storage account
	Get-AzureStorageKey -StorageAccountName $storageAccountName

有关使用经典管理门户获取信息的详细信息，请参阅[关于 Azure 存储帐户](/documentation/articles/storage-create-storage-account/)的“查看、复制和重新生成存储访问密钥”部分。

**创建 Azure 存储帐户**

Azure PowerShell 无法在 HDInsight 创建过程中创建 Blob 容器。你可以使用以下脚本创建一个容器：

	$storageAccountName = "<Azure Storage Account Name>"
	$storageAccountKey = Get-AzureStorageKey -StorageAccountName $storageAccountName |  %{ $_.Primary }
	$containerName="<AzureBlobContainerName>"

	# Create a storage context object
	$destContext = New-AzureStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $storageAccountKey  

	# Create a Blob storage container
	New-AzureStorageContainer -Name $containerName -Context $destContext

**创建群集**

准备好存储帐户和 Blob 容器后，你就可以创建群集了。

	$storageAccountName = "<Azure Storage Account Name>"
	$containerName = "<AzureBlobContainerName>"

	$clusterName = "<HDInsightClusterName>"
	$location = "<AzureDataCenter>"
	$clusterNodes = <ClusterSizeInNodes>

	# Get the Storage account key
	$storageAccountKey = Get-AzureStorageKey -StorageAccountName $storageAccountName | %{ $_.Primary }

	# Create a new HDInsight cluster
	New-AzureHDInsightCluster -Name $clusterName `
		-Location $location `
		-DefaultStorageAccountName "$storageAccountName.blob.core.chinacloudapi.cn" `
		-DefaultStorageAccountKey $storageAccountKey `
		-DefaultStorageContainerName $containerName  `
		-ClusterSizeInNodes $clusterNodes

##列出群集
使用以下命令可列出当前订阅中的所有群集：

	Get-AzureHDInsightCluster

##显示群集

使用以下命令可显示当前订阅中特定群集的详细信息：

	Get-AzureHDInsightCluster -Name <Cluster Name>

##删除群集
使用以下命令来删除群集：

	Remove-AzureHDInsightCluster -Name <Cluster Name>

##缩放群集
群集缩放功能可让你更改 Azure HDInsight 中运行的群集使用的辅助节点数，而无需重新创建群集。

>[AZURE.NOTE] 只支持使用 HDInsight 3.1.3 或更高版本的群集。如果你不确定群集的版本，可以查看“属性”页。

更改 HDInsight 支持的每种类型的群集所用数据节点数的影响：

- Hadoop

	你可以顺利地增加正在运行的 Hadoop 群集中的辅助节点数，而不会影响任何挂起或运行中的作业。你还可以在操作进行中提交新作业。系统会正常处理失败的缩放操作，让群集始终保持正常运行状态。

	减少数据节点数目以缩减 Hadoop 群集时，系统会重新启动群集中的某些服务。这会导致所有正在运行和挂起的作业在缩放操作完成时失败。但是，你可以在操作完成后重新提交这些作业。

- HBase

	你可以顺利地在 HBase 群集运行时对其添加或删除节点。在完成缩放操作后的几分钟内，区域服务器就能自动平衡。不过，你也可以手动平衡区域服务器，方法是登录到群集的头节点，然后在命令提示符窗口中运行以下命令：

		>pushd %HBASE_HOME%\bin
		>hbase shell
		>balancer

	有关使用 HBase shell 的详细信息，请参阅
- Storm

	你可以顺利地在 Storm 群集运行时对其添加或删除数据节点。但是，在缩放操作成功完成后，你需要重新平衡拓扑。

	可以使用两种方法来完成重新平衡操作：

	* Storm Web UI
	* 命令行界面 (CLI) 工具

	有关更多详细信息，请参阅 [Apache Storm 文档](http://storm.apache.org/documentation/Understanding-the-parallelism-of-a-Storm-topology.html)。

	HDInsight 群集上提供了 Storm Web UI：

	![hdinsight storm 缩放重新平衡](./media/hdinsight-administer-use-management-portal-v1/hdinsight.portal.scale.cluster.storm.rebalance.png)

	以下是有关如何使用 CLI 命令重新平衡 Storm 拓扑的示例：

		## Reconfigure the topology "mytopology" to use 5 worker processes,
		## the spout "blue-spout" to use 3 executors, and
		## the bolt "yellow-bolt" to use 10 executors

		$ storm rebalance mytopology -n 5 -e blue-spout=3 -e yellow-bolt=10

若要使用 Azure PowerShell 更改 Hadoop 群集大小，请从客户端计算机运行以下命令：

	Set-AzureHDInsightClusterSize -Cluster <Cluster Name> -ClusterSizeInNodes <NewSize>
	

##<a name="grant/revoke-access"></a> 授予/撤消访问权限

HDInsight 群集提供以下 HTTP Web 服务（所有这些服务都有 REST 样式的终结点）：

- ODBC
- JDBC
- Oozie
- Templeton


默认情况下，将授权这些服务进行访问。你可以撤消/授予访问权限。若要撤消：

	Revoke-AzureHDInsightHttpServicesAccess -Name <Cluster Name>

若要授予：

	$clusterName = "<HDInsight Cluster Name>"

	# Credential option 1
	$hadoopUserName = "admin"
	$hadoopUserPassword = "<Enter the Password>"
	$hadoopUserPW = ConvertTo-SecureString -String $hadoopUserPassword -AsPlainText -Force
	$credential = New-Object System.Management.Automation.PSCredential($hadoopUserName,$hadoopUserPW)

	# Credential option 2
	#$credential = Get-Credential -Message "Enter the HTTP username and password:" -UserName "admin"
	
	Grant-AzureHDInsightHttpServicesAccess -Name $clusterName -HttpCredential $credential

>[AZURE.NOTE] 授予/撤消访问权限时，你将重设群集用户的用户名和密码。

也可以使用经典管理门户完成此操作。请参阅[使用 Azure 经典管理门户管理 HDInsight][hdinsight-admin-portal]。

##更新 HTTP 用户凭据

这与[授予/撤消 HTTP 访问权限](#grant/revoke-access)是同一过程。如果已授予群集 HTTP 访问权限，必须先撤消该访问权限。然后再使用新的 HTTP 用户凭据授予访问权限。


##查找默认存储帐户

以下 Powershell 脚本演示如何获取群集的默认存储帐户名称和默认存储帐户密钥。

	$clusterName = "<HDInsight Cluster Name>"
	
	$cluster = Get-AzureHDInsightCluster -Name $clusterName
	$defaultStorageAccountName = ($cluster.DefaultStorageAccount).Replace(".blob.core.chinacloudapi.cn", "")
	$defaultBlobContainerName = $cluster.DefaultStorageContainer
	$defaultStorageAccountKey = Get-AzureStorageKey -StorageAccountName $defaultStorageAccountName |  %{ $_.Primary }
	$defaultStorageAccountContext = New-AzureStorageContext -StorageAccountName $defaultStorageAccountName -StorageAccountKey $defaultStorageAccountKey 

##提交作业

**提交 MapReduce 作业**

请参阅[在基于 Windows 的 HDInsight 中运行 Hadoop MapReduce 示例](/documentation/articles/hdinsight-run-samples/)。

**提交 Hive 作业**

请参阅[使用 PowerShell 运行 Hive 查询](/documentation/articles/hdinsight-hadoop-use-hive-powershell/)

**提交 Pig 作业**

请参阅[使用 PowerShell 运行 Pig 作业](/documentation/articles/hdinsight-hadoop-use-pig-powershell/)。

**提交 Sqoop 作业**

请参阅[将 Sqoop 与 HDInsight 配合使用](/documentation/articles/hdinsight-use-sqoop/)。

**提交 Oozie 作业**

请参阅[在 HDInsight 中将 Oozie 与 Hadoop 配合使用以定义和运行工作流](/documentation/articles/hdinsight-use-oozie/)。

##将数据上载到 Azure Blob 存储
请参阅[将数据上载到 HDInsight][hdinsight-upload-data]。


## 另请参阅
* [HDInsight cmdlet 参考文档][hdinsight-powershell-reference]
* [使用 Azure 经典管理门户管理 HDInsight][hdinsight-admin-portal]
* [使用命令行界面管理 HDInsight][hdinsight-admin-cli]
* [创建 HDInsight 群集][hdinsight-provision]
* [将数据上载到 HDInsight][hdinsight-upload-data]
* [以编程方式提交 Hadoop 作业][hdinsight-submit-jobs]
* [Azure HDInsight 入门][hdinsight-get-started]


[hdinsight-hive]: /documentation/articles/hdinsight-use-hive/
[azure-purchase-options]: /pricing/overview/
[azure-member-offers]: /pricing/member-offers/
[azure-trial]: /pricing/1rmb-trial/

[hdinsight-get-started]: /documentation/articles/hdinsight-hadoop-tutorial-get-started-windows-v1/
[hdinsight-provision]: /documentation/articles/hdinsight-provision-clusters-v1/
[hdinsight-provision-custom-options]: /documentation/articles/hdinsight-provision-clusters-v1/#configuration
[hdinsight-submit-jobs]: /documentation/articles/hdinsight-submit-hadoop-jobs-programmatically/

[hdinsight-admin-cli]: /documentation/articles/hdinsight-administer-use-command-line/
[hdinsight-admin-portal]: /documentation/articles/hdinsight-administer-use-management-portal-v1/
[hdinsight-storage]: /documentation/articles/hdinsight-hadoop-use-blob-storage/
[hdinsight-use-hive]: /documentation/articles/hdinsight-use-hive/
[hdinsight-use-mapreduce]: /documentation/articles/hdinsight-use-mapreduce/
[hdinsight-upload-data]: /documentation/articles/hdinsight-upload-data/
[hdinsight-flight]: /documentation/articles/hdinsight-analyze-flight-delay-data/

[hdinsight-powershell-reference]: https://msdn.microsoft.com/zh-cn/library/dn858087.aspx

[powershell-install-configure]: /documentation/articles/powershell-install-configure/

[image-hdi-ps-provision]: ./media/hdinsight-administer-use-powershell/HDI.PS.Provision.png

<!---HONumber=Mooncake_0215_2016-->